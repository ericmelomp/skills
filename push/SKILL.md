---
name: push
description: >-
  Analyze repository changes, write a commit message grounded in the Conventional
  Commits spec (fetched live from conventionalcommits.org) and the repo's own
  conventions, then push to the current remote on any host — GitHub, GitLab,
  Bitbucket, Azure DevOps, Gitea, or a self-hosted server. Handles multi-repo
  workspaces by asking which project to commit. Use when the user explicitly asks
  to commit and push or invokes /push (e.g. "commit and push", "comita e faz
  push", "haz commit y push"). Does not auto-trigger and never commits or pushes
  proactively — only on an explicit request. Readable in English, Brazilian
  Portuguese, and Spanish (see translations/).
metadata:
  disable-model-invocation: true
---

# /push — Commit and push to any Git remote

> **Languages / Idiomas:** this canonical file is in English. Full translations:
> [Português (BR)](pt-BR.md) · [Español](es.md).
>
> **Version:** 1.0.0 · **Status:** Stable · **Spec:** [Agent Skills v1](https://agentskills.io/specification)

Analyze the changes in a repository, commit them with a message grounded in the
Conventional Commits specification and the repo's existing style, and push to its
remote. Works with any host — GitHub, GitLab, Bitbucket, Azure DevOps, Gitea, or
a self-hosted server — because the push itself is plain `git`. The host only
affects wording and a few optional helpers, never the core flow.

## Scope and safety posture

- Run **only** when the user explicitly asks (`/push`, "commit and push", "ship
  this"). Never commit or push on your own initiative.
- Default target is `origin` on the current branch, unless the user says
  otherwise.
- Pushing publishes work and can be hard to undo. Before any irreversible or
  history-rewriting action (force-push, `--amend`, deleting refs), stop and get
  explicit confirmation. See "Safety checks".

## 1. Find the right repository

A workspace often contains more than one Git repo — nested folders, monorepo
packages, submodules, or worktrees.

### 1a. Discover repos with changes

`.git` is a _directory_ in a normal clone but a _file_ in submodules and
worktrees, so a plain `find ... -type d` silently misses those. Match `.git`
regardless of type, then resolve each real working-tree root:

```bash
# Find every .git (dir or file), don't descend into it, then resolve its root
for g in $(find . -name .git -prune -print 2>/dev/null); do
  git -C "$(dirname "$g")" rev-parse --show-toplevel 2>/dev/null
done | sort -u
```

Treat each distinct toplevel as one repo. If one repo is a **submodule** of
another discovered repo, confirm with the user how to handle it — don't silently
commit a submodule pointer bump as though it were ordinary work.

For each repo, run `git -C <repo> status --short` and note the branch. Build the
set of repos with any change worth committing (staged, unstaged, or relevant
untracked files).

### 1b. Choose the target repo(s)

| Situation                      | Action                                        |
| ------------------------------ | --------------------------------------------- |
| User named a repo or path      | Use that repo only                            |
| Exactly one repo has changes   | Use it — no question needed                   |
| Zero repos have changes        | Stop — report there's nothing to commit       |
| Two or more repos have changes | Ask which to commit **before** doing anything |

### 1c. Ask when multiple repos have changes

Stop after discovery and present each candidate with its folder name, branch,
and a brief summary of changed files from `git status --short`. Then ask which
to commit, for example:

> Changes exist in more than one repository. Which project should I commit and push?
>
> 1. `api` (main) — 3 files changed
> 2. `web` (feat/login) — 1 file changed
> 3. All of the above (one commit + push per repo)

- Prefer an interactive choice prompt (e.g. an AskQuestion-style tool) when one
  is available; allow multi-select only when "all" is a sensible option.
- If no such tool exists, ask in plain text and **wait for the reply** before
  continuing. Don't infer the target from open editor tabs alone.
- After the user answers, run steps 2–7 only for the chosen repo(s). If they
  pick "all", run the full flow once per repo, in list order.

## 2. Detect the host (light touch)

The push is identical everywhere; the host only changes wording and optional
helpers. Read the remote URL and map it:

```bash
git -C <repo> remote get-url origin 2>/dev/null   # or: git remote -v
```

| Host in the remote URL                   | Term for the change request    | Optional CLI |
| ---------------------------------------- | ------------------------------ | ------------ |
| `github.com` or a GitHub Enterprise host | pull request (PR)              | `gh`         |
| `gitlab.*`                               | merge request (MR)             | `glab`       |
| `bitbucket.org`                          | pull request (PR)              | —            |
| `dev.azure.com`, `*.visualstudio.com`    | pull request (PR)              | `az repos`   |
| anything else / self-hosted              | "merge/pull request" (generic) | —            |

Notes:

- SSH remotes may use a `~/.ssh/config` host alias (e.g. `git@my-host:org/repo`)
  that hides the real host. If the host is ambiguous, don't guess — treat it as
  generic and rely on plain `git`.
- Detection is for clearer output only. **This skill does not open PRs/MRs.** It
  just surfaces the "create a PR/MR" URL if the push prints one.

## 3. Inspect the changes (in parallel)

In the target repo, gather context at once:

```bash
git status
git diff
git diff --staged
git log --oneline -10
git branch -vv
```

When working on a feature branch, also compare against the base:

```bash
git diff main...HEAD 2>/dev/null || git diff master...HEAD 2>/dev/null
```

## 4. Write the commit message

### 4a. Fetch the Conventional Commits spec (always)

Always fetch the canonical spec first and treat it as the source of truth for the
message format:

```
https://www.conventionalcommits.org/en/v1.0.0/
```

Fetch it with whatever web tool the environment provides (e.g. a `WebFetch`
tool, or `curl -fsSL <url>`), and ground the type list, the
`<type>[(scope)]: <description>` structure, and the breaking-change rules in what
the spec actually says. The spec is the parameter for how messages are formed.

If the site is unreachable — offline work, CI, or a locked-down network — fall
back to the embedded summary in 4c so committing is never blocked by a missing
network.

### 4b. Layer the repository's own convention on top

The spec defines the _format_; the repository may define the _specifics_. Check,
in order:

- a `commitlint` config (`.commitlintrc*`, `commitlint.config.*`) → use its
  configured types and scopes;
- `CONTRIBUTING.md` or a `.gitmessage` template → follow what's documented;
- recent history (`git log --oneline -20`) → match the prevailing style.

If a repo clearly uses a _different_ convention than Conventional Commits, follow
the repo — consistency within a project beats any external standard.

### General rules

- Read **all** staged and unstaged changes, not just one file.
- Imperative mood ("add handler", not "added handler"), no trailing period,
  summary ≤ ~72 characters.
- One logical change per commit; split unrelated changes.
- Explain _why_ in the body when the change isn't self-evident; wrap at ~72.
- Write in the language the repo's history already uses — don't switch a
  project's commit language.
- Never commit likely secrets (`.env`, credentials, `*.pem`, tokens,
  `secrets.*`). Warn the user if they ask you to.

### 4c. Conventional Commits summary (fallback / quick reference)

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

| Type       | Use for                                          |
| ---------- | ------------------------------------------------ |
| `feat`     | a new feature or capability                      |
| `fix`      | a bug fix                                        |
| `docs`     | documentation only                               |
| `refactor` | a code change that's neither a fix nor a feature |
| `perf`     | a performance improvement                        |
| `test`     | tests only                                       |
| `build`    | the build system or dependencies                 |
| `ci`       | CI/CD configuration                              |
| `chore`    | maintenance, tooling, miscellany                 |
| `style`    | formatting only, no logic change                 |

Breaking changes: add `!` after the type/scope (`feat(api)!: ...`) or a
`BREAKING CHANGE:` footer describing what broke and how to migrate.

**Examples**

Input: added optional pagination to the users-list endpoint
Output: `feat(api): add pagination to users list`

Input: fixed a crash when the config file is missing
Output: `fix(config): handle missing config file gracefully`

Input: removed the deprecated v1 auth flow
Output:

```
feat(auth)!: remove deprecated v1 flow

BREAKING CHANGE: clients must migrate to the v2 token endpoint.
```

## 5. Preview before committing

Show the user the drafted message(s) and the push target (remote + branch), and
get a clear go-ahead before committing. This catches a wrong-scope commit or a
wrong-branch push while they're still cheap to fix.

## 6. Safety checks

Never do any of the following unless the user explicitly asked:

- any `git config` write
- `--no-verify` / `--no-gpg-sign` (skipping hooks or commit signing)
- `push --force` / `-f` — and warn before force-pushing `main`/`master`
- `git commit --amend` — and only when it's an unpushed commit you created

Other guardrails:

- Clean tree / nothing to stage → report it and **stop**. No empty commits.
- If a commit fails on a hook, fix the issue and create a **new** commit; don't
  amend the failed one.
- Respect `.gitignore`; stage only files related to the work. Skip incidental
  artifacts (`__pycache__`, `.terraform`, build output, local binaries) unless
  the user intends to commit them.

## 7. Commit

```bash
git add <relevant paths>
git commit -m "$(cat <<'EOF'
feat(scope): short imperative description

Optional body explaining why.
EOF
)"
git status
```

## 8. Push

Plain `git` works on every host:

```bash
git push -u origin HEAD     # first push of a new branch (sets upstream)
git push                    # when an upstream already exists
```

After pushing, report:

- the branch name and short commit SHA(s),
- the remote URL (from `git remote -v`),
- the "create a PR/MR" URL if the push output printed one — most hosts (GitHub,
  GitLab, Bitbucket, Azure DevOps) print it for a newly pushed branch.

If the matching host CLI is installed (`gh`, `glab`, `az`), it may be used for
read-only extras such as CI/pipeline status — but it is never required, and this
skill does not create PRs/MRs.

## 9. When something goes wrong

| Situation                                  | What to do                                                                                                          |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| Authentication failed                      | Point to the credentials/token or SSH key for that host; don't retry blindly                                        |
| Non-fast-forward (remote is ahead)         | Explain; offer `git pull --rebase` if appropriate; never force without explicit approval                            |
| Branches have diverged                     | Surface both sides and let the user choose rebase vs merge                                                          |
| Push rejected by a protected branch        | Many hosts block direct pushes to the default branch; suggest pushing a feature branch and opening a PR/MR manually |
| Secret-scanning push protection            | Stop; help remove the secret and rewrite the offending commit before retrying                                       |
| Pre-commit / commit-msg hook failed        | Show the hook output, fix the cause, and create a new commit                                                        |
| Not a Git repo / detached HEAD / no remote | Report the exact state and ask how to proceed instead of guessing a target                                          |

## 10. Multi-repo summary

When the user chose "all" (or named several repos), end with a short table:

| Repo  | Branch       | Commit    | Pushed |
| ----- | ------------ | --------- | ------ |
| `api` | `main`       | `abc1234` | yes    |
| `web` | `feat/login` | `def5678` | yes    |
