# /push — Commitar e enviar para qualquer remoto Git

> **Tradução (pt-BR).** Este é um documento de leitura em português. O arquivo
> canônico que o agente carrega é [`../SKILL.md`](../SKILL.md) (em inglês). Outras
> traduções: [English](../SKILL.md) · [Español](es.md).

Analisa as mudanças de um repositório, faz o commit com uma mensagem baseada na
especificação Conventional Commits e no estilo já usado pelo repositório, e envia
para o remoto dele. Funciona com qualquer host — GitHub, GitLab, Bitbucket, Azure
DevOps, Gitea ou um servidor self-hosted — porque o push em si é `git` puro. O
host só muda a terminologia e alguns auxiliares opcionais, nunca o fluxo
principal.

## Escopo e postura de segurança

- Execute **somente** quando o usuário pedir explicitamente (`/push`, "commita e
  faz push", "sobe isso"). Nunca commite ou faça push por conta própria.
- O destino padrão é `origin` na branch atual, a menos que o usuário diga o
  contrário.
- Fazer push publica o trabalho e pode ser difícil de desfazer. Antes de qualquer
  ação irreversível ou que reescreva histórico (force-push, `--amend`, apagar
  refs), pare e peça confirmação explícita. Veja "Verificações de segurança".

## 1. Encontrar o repositório certo

Um workspace costuma conter mais de um repositório Git — pastas aninhadas,
pacotes de monorepo, submódulos ou worktrees.

### 1a. Descobrir repositórios com mudanças

`.git` é um *diretório* num clone normal, mas um *arquivo* em submódulos e
worktrees, então um `find ... -type d` simples deixa esses de fora
silenciosamente. Encontre `.git` independente do tipo e resolva a raiz real de
cada working-tree:

```bash
# Encontra todo .git (dir ou arquivo), não entra nele e resolve a raiz
for g in $(find . -name .git -prune -print 2>/dev/null); do
  git -C "$(dirname "$g")" rev-parse --show-toplevel 2>/dev/null
done | sort -u
```

Trate cada raiz distinta como um repositório. Se um repositório for **submódulo**
de outro já descoberto, confirme com o usuário como tratar — não commite
silenciosamente uma atualização de ponteiro de submódulo como se fosse trabalho
comum.

Para cada repositório, rode `git -C <repo> status --short` e anote a branch.
Monte o conjunto de repositórios com qualquer mudança que valha commitar (staged,
unstaged ou untracked relevante).

### 1b. Escolher o(s) repositório(s) alvo

| Situação | Ação |
|----------|------|
| O usuário nomeou um repositório ou caminho | Use só esse repositório |
| Exatamente um repositório tem mudanças | Use ele — sem precisar perguntar |
| Nenhum repositório tem mudanças | Pare — informe que não há nada a commitar |
| Dois ou mais repositórios têm mudanças | Pergunte qual commitar **antes** de fazer qualquer coisa |

### 1c. Perguntar quando há mudanças em vários repositórios

Pare após a descoberta e apresente cada candidato com o nome da pasta, a branch e
um breve resumo dos arquivos alterados a partir do `git status --short`. Em
seguida, pergunte qual commitar, por exemplo:

> Há mudanças em mais de um repositório. Qual projeto devo commitar e enviar?
> 1. `api` (main) — 3 arquivos alterados
> 2. `web` (feat/login) — 1 arquivo alterado
> 3. Todos os listados acima (um commit + push por repositório)

- Prefira um prompt de escolha interativo (ex.: uma ferramenta estilo
  AskQuestion) quando houver; permita seleção múltipla só quando "todos" fizer
  sentido.
- Se não houver essa ferramenta, pergunte em texto e **aguarde a resposta** antes
  de continuar. Não deduza o alvo apenas pelas abas abertas no editor.
- Depois da resposta, rode os passos 2–7 só para o(s) repositório(s) escolhido(s).
  Se o usuário escolher "todos", rode o fluxo completo uma vez por repositório, na
  ordem da lista.

## 2. Detectar o host (de leve)

O push é idêntico em todo lugar; o host só muda a terminologia e auxiliares
opcionais. Leia a URL do remoto e mapeie:

```bash
git -C <repo> remote get-url origin 2>/dev/null   # ou: git remote -v
```

| Host na URL do remoto | Termo para o pedido de mudança | CLI opcional |
|-----------------------|--------------------------------|--------------|
| `github.com` ou um host GitHub Enterprise | pull request (PR) | `gh` |
| `gitlab.*` | merge request (MR) | `glab` |
| `bitbucket.org` | pull request (PR) | — |
| `dev.azure.com`, `*.visualstudio.com` | pull request (PR) | `az repos` |
| qualquer outro / self-hosted | "merge/pull request" (genérico) | — |

Observações:
- Remotos SSH podem usar um alias de host do `~/.ssh/config` (ex.:
  `git@meu-host:org/repo`) que esconde o host real. Se o host for ambíguo, não
  adivinhe — trate como genérico e use `git` puro.
- A detecção serve só para deixar a saída mais clara. **Esta skill não abre
  PRs/MRs.** Ela apenas mostra a URL de "criar PR/MR" se o push imprimir uma.

## 3. Inspecionar as mudanças (em paralelo)

No repositório alvo, reúna o contexto de uma vez:

```bash
git status
git diff
git diff --staged
git log --oneline -10
git branch -vv
```

Ao trabalhar em uma feature branch, compare também com a base:

```bash
git diff main...HEAD 2>/dev/null || git diff master...HEAD 2>/dev/null
```

## 4. Escrever a mensagem de commit

### 4a. Buscar a especificação Conventional Commits (sempre)

Sempre busque primeiro a especificação canônica e trate-a como fonte de verdade
para o formato da mensagem:

```
https://www.conventionalcommits.org/pt-br/v1.0.0/
```

Busque com a ferramenta web disponível no ambiente (ex.: uma ferramenta
`WebFetch`, ou `curl -fsSL <url>`) e baseie a lista de tipos, a estrutura
`<tipo>[(escopo)]: <descrição>` e as regras de breaking change no que a
especificação realmente diz. A especificação é o parâmetro de como as mensagens
são formadas.

Se o site estiver inacessível — trabalho offline, CI ou rede restrita — recorra
ao resumo embutido em 4c, para que o commit nunca seja bloqueado por falta de
rede.

### 4b. Sobrepor a convenção do próprio repositório

A especificação define o *formato*; o repositório pode definir os *detalhes*.
Verifique, nesta ordem:

- uma configuração de `commitlint` (`.commitlintrc*`, `commitlint.config.*`) →
  use os tipos e escopos configurados;
- `CONTRIBUTING.md` ou um template `.gitmessage` → siga o que está documentado;
- o histórico recente (`git log --oneline -20`) → acompanhe o estilo
  predominante.

Se um repositório claramente usa uma convenção *diferente* de Conventional
Commits, siga o repositório — consistência dentro de um projeto vale mais do que
qualquer padrão externo.

### Regras gerais

- Leia **todas** as mudanças staged e unstaged, não apenas um arquivo.
- Modo imperativo ("adiciona handler", não "adicionado handler"), sem ponto final,
  resumo com ≤ ~72 caracteres.
- Uma mudança lógica por commit; separe mudanças não relacionadas.
- Explique o *porquê* no corpo quando a mudança não for óbvia; quebre em ~72.
- Escreva no idioma que o histórico do repositório já usa — não troque o idioma
  dos commits de um projeto.
- Nunca commite prováveis segredos (`.env`, credenciais, `*.pem`, tokens,
  `secrets.*`). Avise o usuário se ele pedir isso.

### 4c. Resumo de Conventional Commits (fallback / referência rápida)

```
<tipo>[escopo opcional]: <descrição>

[corpo opcional]

[rodapé(s) opcional(is)]
```

| Tipo | Quando usar |
|------|-------------|
| `feat` | uma nova funcionalidade ou capacidade |
| `fix` | uma correção de bug |
| `docs` | apenas documentação |
| `refactor` | mudança de código que não é correção nem feature |
| `perf` | melhoria de performance |
| `test` | apenas testes |
| `build` | sistema de build ou dependências |
| `ci` | configuração de CI/CD |
| `chore` | manutenção, tooling, diversos |
| `style` | apenas formatação, sem mudança de lógica |

Breaking changes: adicione `!` após o tipo/escopo (`feat(api)!: ...`) ou um
rodapé `BREAKING CHANGE:` descrevendo o que quebrou e como migrar.

**Exemplos**

Entrada: adicionada paginação opcional ao endpoint de listagem de usuários
Saída: `feat(api): add pagination to users list`

Entrada: corrigido um crash quando o arquivo de config está ausente
Saída: `fix(config): handle missing config file gracefully`

Entrada: removido o fluxo de auth v1 obsoleto
Saída:
```
feat(auth)!: remove deprecated v1 flow

BREAKING CHANGE: clients must migrate to the v2 token endpoint.
```

> Observação: nos exemplos, a *descrição* segue o idioma do repositório. Se o
> projeto escreve commits em português, escreva a descrição em português.

## 5. Pré-visualizar antes de commitar

Mostre ao usuário a(s) mensagem(ns) rascunhada(s) e o destino do push (remoto +
branch) e obtenha um aval claro antes de commitar. Isso pega um commit com escopo
errado ou um push na branch errada enquanto ainda é barato corrigir.

## 6. Verificações de segurança

Nunca faça nenhum dos itens abaixo sem o usuário pedir explicitamente:

- qualquer escrita de `git config`
- `--no-verify` / `--no-gpg-sign` (pular hooks ou assinatura de commit)
- `push --force` / `-f` — e avise antes de fazer force-push em `main`/`master`
- `git commit --amend` — e só quando for um commit não enviado que você criou

Outras salvaguardas:

- Árvore limpa / nada a preparar → informe e **pare**. Sem commits vazios.
- Se um commit falhar em um hook, corrija a causa e crie um **novo** commit; não
  faça amend do commit que falhou.
- Respeite o `.gitignore`; prepare apenas arquivos relacionados ao trabalho. Pule
  artefatos incidentais (`__pycache__`, `.terraform`, saída de build, binários
  locais), a menos que o usuário queira commitá-los.

## 7. Commit

```bash
git add <caminhos relevantes>
git commit -m "$(cat <<'EOF'
feat(escopo): descrição curta no imperativo

Corpo opcional explicando o porquê.
EOF
)"
git status
```

## 8. Push

`git` puro funciona em todo host:

```bash
git push -u origin HEAD     # primeiro push de uma branch nova (define o upstream)
git push                    # quando o upstream já existe
```

Depois do push, informe:

- o nome da branch e o(s) SHA(s) curto(s) do commit;
- a URL do remoto (de `git remote -v`);
- a URL de "criar PR/MR" se a saída do push imprimir uma — a maioria dos hosts
  (GitHub, GitLab, Bitbucket, Azure DevOps) imprime para uma branch recém-enviada.

Se a CLI correspondente ao host estiver instalada (`gh`, `glab`, `az`), ela pode
ser usada para extras somente-leitura, como status de CI/pipeline — mas nunca é
obrigatória, e esta skill não cria PRs/MRs.

## 9. Quando algo dá errado

| Situação | O que fazer |
|----------|-------------|
| Falha de autenticação | Aponte para as credenciais/token ou a chave SSH daquele host; não tente de novo às cegas |
| Non-fast-forward (remoto à frente) | Explique; ofereça `git pull --rebase` se apropriado; nunca force sem aprovação explícita |
| Branches divergiram | Mostre os dois lados e deixe o usuário escolher rebase ou merge |
| Push rejeitado por branch protegida | Muitos hosts bloqueiam push direto na branch padrão; sugira enviar uma feature branch e abrir um PR/MR manualmente |
| Push protection de secret scanning | Pare; ajude a remover o segredo e a reescrever o commit ofensor antes de tentar de novo |
| Hook de pre-commit / commit-msg falhou | Mostre a saída do hook, corrija a causa e crie um novo commit |
| Não é um repositório Git / detached HEAD / sem remoto | Informe o estado exato e pergunte como proceder em vez de adivinhar um alvo |

## 10. Resumo multi-repo

Quando o usuário escolher "todos" (ou nomear vários repositórios), termine com uma
tabela curta:

| Repositório | Branch | Commit | Enviado |
|-------------|--------|--------|---------|
| `api` | `main` | `abc1234` | sim |
| `web` | `feat/login` | `def5678` | sim |
