# /push — Commitear y enviar a cualquier remoto Git

> **Traducción (es).** Este es un documento de lectura en español. El archivo
> canónico que carga el agente es [`../SKILL.md`](../SKILL.md) (en inglés). Otras
> traducciones: [English](../SKILL.md) · [Português (BR)](pt-BR.md).

Analiza los cambios de un repositorio, los commitea con un mensaje basado en la
especificación Conventional Commits y en el estilo que ya usa el repositorio, y
los envía a su remoto. Funciona con cualquier host — GitHub, GitLab, Bitbucket,
Azure DevOps, Gitea o un servidor self-hosted — porque el push en sí es `git`
puro. El host solo cambia la terminología y algunos auxiliares opcionales, nunca
el flujo principal.

## Alcance y postura de seguridad

- Ejecuta **solo** cuando el usuario lo pida explícitamente (`/push`, "haz commit
  y push", "sube esto"). Nunca commitees ni hagas push por iniciativa propia.
- El destino por defecto es `origin` en la rama actual, salvo que el usuario
  indique lo contrario.
- Hacer push publica el trabajo y puede ser difícil de deshacer. Antes de
  cualquier acción irreversible o que reescriba historial (force-push, `--amend`,
  borrar refs), detente y pide confirmación explícita. Ver "Comprobaciones de
  seguridad".

## 1. Encontrar el repositorio correcto

Un workspace suele contener más de un repositorio Git — carpetas anidadas,
paquetes de monorepo, submódulos o worktrees.

### 1a. Descubrir repositorios con cambios

`.git` es un *directorio* en un clon normal, pero un *archivo* en submódulos y
worktrees, así que un `find ... -type d` simple los omite silenciosamente.
Encuentra `.git` sin importar el tipo y luego resuelve la raíz real de cada
working-tree:

```bash
# Encuentra todo .git (dir o archivo), no entra en él y resuelve su raíz
for g in $(find . -name .git -prune -print 2>/dev/null); do
  git -C "$(dirname "$g")" rev-parse --show-toplevel 2>/dev/null
done | sort -u
```

Trata cada raíz distinta como un repositorio. Si un repositorio es **submódulo**
de otro ya descubierto, confirma con el usuario cómo tratarlo — no commitees
silenciosamente una actualización de puntero de submódulo como si fuera trabajo
normal.

Para cada repositorio, ejecuta `git -C <repo> status --short` y anota la rama.
Arma el conjunto de repositorios con cualquier cambio que valga la pena commitear
(staged, unstaged o untracked relevante).

### 1b. Elegir el/los repositorio(s) objetivo

| Situación | Acción |
|-----------|--------|
| El usuario nombró un repositorio o ruta | Usa solo ese repositorio |
| Exactamente un repositorio tiene cambios | Úsalo — no hace falta preguntar |
| Ningún repositorio tiene cambios | Detente — informa que no hay nada para commitear |
| Dos o más repositorios tienen cambios | Pregunta cuál commitear **antes** de hacer nada |

### 1c. Preguntar cuando hay cambios en varios repositorios

Detente tras el descubrimiento y presenta cada candidato con el nombre de la
carpeta, la rama y un breve resumen de los archivos modificados a partir de
`git status --short`. Luego pregunta cuál commitear, por ejemplo:

> Hay cambios en más de un repositorio. ¿Qué proyecto debo commitear y enviar?
> 1. `api` (main) — 3 archivos modificados
> 2. `web` (feat/login) — 1 archivo modificado
> 3. Todos los anteriores (un commit + push por repositorio)

- Prefiere un prompt de elección interactivo (p. ej. una herramienta tipo
  AskQuestion) cuando exista; permite selección múltiple solo cuando "todos"
  tenga sentido.
- Si no hay tal herramienta, pregunta en texto y **espera la respuesta** antes de
  continuar. No deduzcas el objetivo solo por las pestañas abiertas del editor.
- Tras la respuesta, ejecuta los pasos 2–7 solo para el/los repositorio(s)
  elegido(s). Si el usuario elige "todos", ejecuta el flujo completo una vez por
  repositorio, en el orden de la lista.

## 2. Detectar el host (de forma ligera)

El push es idéntico en todas partes; el host solo cambia la terminología y los
auxiliares opcionales. Lee la URL del remoto y mapéala:

```bash
git -C <repo> remote get-url origin 2>/dev/null   # o: git remote -v
```

| Host en la URL del remoto | Término para la solicitud de cambio | CLI opcional |
|---------------------------|-------------------------------------|--------------|
| `github.com` o un host GitHub Enterprise | pull request (PR) | `gh` |
| `gitlab.*` | merge request (MR) | `glab` |
| `bitbucket.org` | pull request (PR) | — |
| `dev.azure.com`, `*.visualstudio.com` | pull request (PR) | `az repos` |
| cualquier otro / self-hosted | "merge/pull request" (genérico) | — |

Notas:
- Los remotos SSH pueden usar un alias de host del `~/.ssh/config` (p. ej.
  `git@mi-host:org/repo`) que oculta el host real. Si el host es ambiguo, no
  adivines — trátalo como genérico y usa `git` puro.
- La detección sirve solo para una salida más clara. **Esta skill no abre
  PRs/MRs.** Solo muestra la URL de "crear PR/MR" si el push imprime una.

## 3. Inspeccionar los cambios (en paralelo)

En el repositorio objetivo, reúne el contexto de una vez:

```bash
git status
git diff
git diff --staged
git log --oneline -10
git branch -vv
```

Al trabajar en una feature branch, compara también contra la base:

```bash
git diff main...HEAD 2>/dev/null || git diff master...HEAD 2>/dev/null
```

## 4. Escribir el mensaje de commit

### 4a. Obtener la especificación Conventional Commits (siempre)

Obtén siempre primero la especificación canónica y trátala como fuente de verdad
para el formato del mensaje:

```
https://www.conventionalcommits.org/es/v1.0.0/
```

Obtenla con la herramienta web disponible en el entorno (p. ej. una herramienta
`WebFetch`, o `curl -fsSL <url>`) y basa la lista de tipos, la estructura
`<tipo>[(ámbito)]: <descripción>` y las reglas de breaking change en lo que la
especificación realmente dice. La especificación es el parámetro de cómo se
forman los mensajes.

Si el sitio está inaccesible — trabajo offline, CI o red restringida — recurre al
resumen incrustado en 4c, para que el commit nunca se bloquee por falta de red.

### 4b. Superponer la convención del propio repositorio

La especificación define el *formato*; el repositorio puede definir los
*detalles*. Verifica, en este orden:

- una configuración de `commitlint` (`.commitlintrc*`, `commitlint.config.*`) →
  usa sus tipos y ámbitos configurados;
- `CONTRIBUTING.md` o una plantilla `.gitmessage` → sigue lo documentado;
- el historial reciente (`git log --oneline -20`) → acompaña el estilo
  predominante.

Si un repositorio usa claramente una convención *distinta* de Conventional
Commits, sigue al repositorio — la consistencia dentro de un proyecto vale más
que cualquier estándar externo.

### Reglas generales

- Lee **todos** los cambios staged y unstaged, no solo un archivo.
- Modo imperativo ("agrega handler", no "agregado handler"), sin punto final,
  resumen ≤ ~72 caracteres.
- Un cambio lógico por commit; separa cambios no relacionados.
- Explica el *porqué* en el cuerpo cuando el cambio no sea evidente; ajusta a ~72.
- Escribe en el idioma que ya usa el historial del repositorio — no cambies el
  idioma de los commits de un proyecto.
- Nunca commitees posibles secretos (`.env`, credenciales, `*.pem`, tokens,
  `secrets.*`). Advierte al usuario si lo pide.

### 4c. Resumen de Conventional Commits (fallback / referencia rápida)

```
<tipo>[ámbito opcional]: <descripción>

[cuerpo opcional]

[pie(s) opcional(es)]
```

| Tipo | Cuándo usar |
|------|-------------|
| `feat` | una nueva funcionalidad o capacidad |
| `fix` | una corrección de bug |
| `docs` | solo documentación |
| `refactor` | cambio de código que no es corrección ni feature |
| `perf` | mejora de rendimiento |
| `test` | solo pruebas |
| `build` | sistema de build o dependencias |
| `ci` | configuración de CI/CD |
| `chore` | mantenimiento, tooling, varios |
| `style` | solo formato, sin cambio de lógica |

Breaking changes: agrega `!` tras el tipo/ámbito (`feat(api)!: ...`) o un pie
`BREAKING CHANGE:` que describa qué se rompió y cómo migrar.

**Ejemplos**

Entrada: se agregó paginación opcional al endpoint de listado de usuarios
Salida: `feat(api): add pagination to users list`

Entrada: se corrigió un crash cuando falta el archivo de configuración
Salida: `fix(config): handle missing config file gracefully`

Entrada: se eliminó el flujo de auth v1 obsoleto
Salida:
```
feat(auth)!: remove deprecated v1 flow

BREAKING CHANGE: clients must migrate to the v2 token endpoint.
```

> Nota: en los ejemplos, la *descripción* sigue el idioma del repositorio. Si el
> proyecto escribe commits en español, escribe la descripción en español.

## 5. Previsualizar antes de commitear

Muestra al usuario el/los mensaje(s) redactado(s) y el destino del push (remoto +
rama) y obtén un visto bueno claro antes de commitear. Esto detecta un commit con
ámbito equivocado o un push en la rama equivocada mientras aún es barato
corregirlo.

## 6. Comprobaciones de seguridad

Nunca hagas nada de lo siguiente sin que el usuario lo pida explícitamente:

- cualquier escritura de `git config`
- `--no-verify` / `--no-gpg-sign` (saltar hooks o la firma del commit)
- `push --force` / `-f` — y advierte antes de hacer force-push en `main`/`master`
- `git commit --amend` — y solo cuando sea un commit no enviado que tú creaste

Otras salvaguardas:

- Árbol limpio / nada que preparar → infórmalo y **detente**. Sin commits vacíos.
- Si un commit falla en un hook, corrige la causa y crea un commit **nuevo**; no
  hagas amend del commit que falló.
- Respeta el `.gitignore`; prepara solo archivos relacionados con el trabajo.
  Omite artefactos incidentales (`__pycache__`, `.terraform`, salida de build,
  binarios locales), salvo que el usuario quiera commitearlos.

## 7. Commit

```bash
git add <rutas relevantes>
git commit -m "$(cat <<'EOF'
feat(ámbito): descripción corta en imperativo

Cuerpo opcional explicando el porqué.
EOF
)"
git status
```

## 8. Push

`git` puro funciona en todos los hosts:

```bash
git push -u origin HEAD     # primer push de una rama nueva (fija el upstream)
git push                    # cuando el upstream ya existe
```

Tras el push, informa:

- el nombre de la rama y el/los SHA(s) corto(s) del commit;
- la URL del remoto (de `git remote -v`);
- la URL de "crear PR/MR" si la salida del push imprime una — la mayoría de los
  hosts (GitHub, GitLab, Bitbucket, Azure DevOps) la imprime para una rama recién
  enviada.

Si la CLI correspondiente al host está instalada (`gh`, `glab`, `az`), puede
usarse para extras de solo lectura, como el estado de CI/pipeline — pero nunca es
obligatoria, y esta skill no crea PRs/MRs.

## 9. Cuando algo sale mal

| Situación | Qué hacer |
|-----------|-----------|
| Fallo de autenticación | Apunta a las credenciales/token o a la clave SSH de ese host; no reintentes a ciegas |
| Non-fast-forward (remoto adelantado) | Explica; ofrece `git pull --rebase` si corresponde; nunca fuerces sin aprobación explícita |
| Ramas divergieron | Muestra ambos lados y deja que el usuario elija rebase o merge |
| Push rechazado por rama protegida | Muchos hosts bloquean el push directo a la rama por defecto; sugiere enviar una feature branch y abrir un PR/MR manualmente |
| Push protection de secret scanning | Detente; ayuda a quitar el secreto y a reescribir el commit ofensor antes de reintentar |
| Hook de pre-commit / commit-msg falló | Muestra la salida del hook, corrige la causa y crea un commit nuevo |
| No es un repositorio Git / detached HEAD / sin remoto | Informa el estado exacto y pregunta cómo proceder en vez de adivinar un objetivo |

## 10. Resumen multi-repo

Cuando el usuario elija "todos" (o nombre varios repositorios), termina con una
tabla corta:

| Repositorio | Rama | Commit | Enviado |
|-------------|------|--------|---------|
| `api` | `main` | `abc1234` | sí |
| `web` | `feat/login` | `def5678` | sí |
