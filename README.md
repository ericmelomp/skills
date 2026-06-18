<p align="center">
  <h1 align="center">Skills</h1>
  <p align="center">
    Agent Skills for AI-Assisted Development — Enterprise-Grade Automation
  </p>
  <p align="center">
    <a href="https://agentskills.io/specification">
      <img src="https://img.shields.io/badge/spec-Agent%20Skills%20v1-blue" alt="Agent Skills Specification" />
    </a>
    <a href="https://skills.sh">
      <img src="https://img.shields.io/badge/registry-skills.sh-green" alt="Skills Registry" />
    </a>
    <a href="#license">
      <img src="https://img.shields.io/badge/license-MIT-yellow" alt="License" />
    </a>
  </p>
</p>

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Quick Start](#quick-start)
- [Available Skills](#available-skills)
- [Installation Options](#installation-options)
- [Repository Structure](#repository-structure)
- [Development Guide](#development-guide)
- [Validation & Quality Assurance](#validation--quality-assurance)
- [Publishing & Distribution](#publishing--distribution)
- [Security Policy](#security-policy)
- [Contributing](#contributing)
- [Support & Communication](#support--communication)
- [License](#license)

---

## Overview

A curated collection of agent skills designed for AI coding assistants. Built on the open [Agent Skills](https://agentskills.io/specification) specification, these skills provide deterministic, auditable automation workflows that integrate seamlessly with modern development environments.

### Key Principles

| Principle               | Description                                                                        |
| ----------------------- | ---------------------------------------------------------------------------------- |
| **Explicit Invocation** | Skills execute only on direct user command — never auto-triggered                  |
| **Safety-First**        | Destructive operations require explicit confirmation before execution              |
| **Host-Agnostic**       | Works with GitHub, GitLab, Bitbucket, Azure DevOps, Gitea, and self-hosted servers |
| **Multi-Language**      | Full i18n support with canonical English and verified translations                 |
| **Standards-Driven**    | Grounded in industry specifications (Conventional Commits, Agent Skills spec)      |

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      Agent Skills                         │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────┐    ┌──────────────┐    ┌───────────────┐  │
│  │  Skills  │───▶│  Agent Skills │───▶│  AI Assistant │  │
│  │  (YAML)  │    │  Runtime      │    │  (Cursor, etc)│  │
│  └──────────┘    └──────────────┘    └───────────────┘  │
│       │                                                  │
│       ▼                                                  │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Execution Layer                                  │   │
│  │  • Git operations (commit, push, branch mgmt)    │   │
│  │  • Host detection (GitHub, GitLab, Bitbucket...) │   │
│  │  • Convention enforcement (Conventional Commits) │   │
│  │  • Safety validation (secrets, force-push, etc.) │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Quick Start

### Prerequisites

- **Node.js** ≥ 18.x
- **Git** ≥ 2.30
- A compatible AI coding assistant (Cursor, Claude Code, or any Agent Skills-compatible tool)

### Installation

Install a skill into the current project:

```bash
npx skills add ericmelomp/skills --skill push
```

Install globally (available across all projects):

```bash
npx skills add ericmelomp/skills --skill push -g
```

Install into specific agents only:

```bash
npx skills add ericmelomp/skills --skill push --agent cursor claude-code
```

> **Verify installation:** After installing, invoke `/push` in your AI assistant to confirm the skill is loaded.

---

## Available Skills

| Skill                   | Trigger | Description                                                                                | Status    |
| ----------------------- | ------- | ------------------------------------------------------------------------------------------ | --------- |
| [`push`](push/SKILL.md) | `/push` | Automated commit message generation and push workflow with Conventional Commits compliance | ✅ Stable |

### push

A complete commit-and-push workflow that activates exclusively on explicit user request.

**Trigger phrases:** `/push`, "commit and push", "comita e faz push", "haz commit y push"

**Capabilities:**

- 🔍 **Multi-repo discovery** — Detects all Git repositories in a workspace (including submodules and worktrees)
- 📝 **Conventional Commits** — Fetches the live spec and layers repo-specific conventions
- 👀 **Preview before action** — Shows the draft message and push target for confirmation
- 🌐 **Universal host support** — GitHub, GitLab, Bitbucket, Azure DevOps, Gitea, self-hosted
- 🔒 **Safety guardrails** — Blocks force-push, amend, hook bypass, and secret leakage without explicit approval
- 🌍 **Multilingual** — English, Portuguese (BR), Spanish

**Requirements:**

| Dependency | Required    | Purpose                                     |
| ---------- | ----------- | ------------------------------------------- |
| `git`      | ✅ Yes      | Core VCS operations                         |
| `gh`       | ❌ Optional | GitHub push-link reporting, CI status       |
| `glab`     | ❌ Optional | GitLab push-link reporting, pipeline status |
| `az repos` | ❌ Optional | Azure DevOps integration                    |

**Documentation:**

| Language            | File                             | Status      |
| ------------------- | -------------------------------- | ----------- |
| English (canonical) | [`push/SKILL.md`](push/SKILL.md) | ✅ Complete |
| Português (BR)      | [`push/pt-BR.md`](push/pt-BR.md) | ✅ Complete |
| Español             | [`push/es.md`](push/es.md)       | ✅ Complete |

---

## Installation Options

### Per-Project (Recommended for Teams)

```bash
npx skills add ericmelomp/skills --skill push
```

This installs the skill in the current project directory, ensuring all team members using AI assistants have access to the same workflow.

### Global

```bash
npx skills add ericmelomp/skills --skill push -g
```

Available in all projects for the current user. Best for individual developers who want consistent tooling.

### Agent-Specific

```bash
npx skills add ericmelomp/skills --skill push --agent cursor claude-code
```

Restricts the skill to specific AI assistants. Useful when different tools require different configurations.

---

## Repository Structure

```
skills/
├── README.md                   # This file — project overview and documentation
├── SECURITY.md                 # Security policy and vulnerability reporting
├── CONTRIBUTING.md             # Contribution guidelines
├── CHANGELOG.md                # Version history
├── .claude/
│   └── settings.local.json     # Local agent configuration
└── push/                       # Skill: push
    ├── SKILL.md                # Canonical skill definition (English)
    ├── pt-BR.md                # Translation: Brazilian Portuguese
    ├── es.md                   # Translation: Spanish
    ├── scripts/                # (Reserved) Executable helpers
    ├── references/             # (Reserved) On-demand reference docs
    └── assets/                 # (Reserved) Templates and static resources
```

### Naming Conventions

| Item            | Convention             | Example               |
| --------------- | ---------------------- | --------------------- |
| Skill directory | `lowercase-hyphenated` | `push`, `auto-review` |
| Main definition | `SKILL.md` (required)  | `push/SKILL.md`       |
| Translations    | `<locale>.md`          | `pt-BR.md`, `es.md`   |
| Scripts         | `<verb>-<noun>.sh`     | `validate-message.sh` |

---

## Development Guide

### Adding a New Skill

1. **Create the skill directory:**

   ```bash
   mkdir my-skill
   ```

2. **Define `SKILL.md`** with required YAML frontmatter:

   ```yaml
   ---
   name: my-skill
   description: >-
     Brief description of what the skill does and when to trigger it.
     Max 1024 characters.
   disable-model-invocation: true
   ---
   ```

3. **Write step-by-step instructions** in the markdown body (max ~500 lines / ~5000 tokens)

4. **Add translations** (optional but recommended):
   - `pt-BR.md` — Brazilian Portuguese
   - `es.md` — Spanish
   - Additional locales as needed

5. **Validate:**

   ```bash
   npx skills-ref validate ./my-skill
   ```

6. **Update `README.md`** — Add an entry to the Available Skills table

7. **Submit for review** — Open a pull request following the contribution guidelines

### Skill Design Guidelines

- **Single responsibility** — One skill, one well-defined workflow
- **Explicit triggers only** — Never auto-activate; always require user intent
- **Graceful degradation** — Handle offline scenarios, missing tools, and edge cases
- **Safety by default** — Confirm before irreversible actions
- **Idempotent when possible** — Running the skill twice should not cause issues

### Configuration: `disable-model-invocation`

Skills in this repository include `disable-model-invocation: true` in their frontmatter. This instructs Cursor to load the skill only when the user explicitly invokes it, rather than injecting it into every context window.

> **Note:** This field is Cursor-specific and is not part of the [Agent Skills specification](https://agentskills.io/specification). The `skills-ref validate` command will flag it as an unexpected field — this warning is expected and harmless. The skill installs and runs correctly via `npx skills add`.

---

## Validation & Quality Assurance

### Local Validation

Run the official validator before pushing changes:

```bash
npx skills-ref validate ./push
```

### Checklist Before Submission

- [ ] `SKILL.md` has valid YAML frontmatter (`name`, `description`)
- [ ] `name` matches the parent directory name
- [ ] `name` uses only lowercase letters, numbers, and hyphens (max 64 chars)
- [ ] `description` is ≤ 1024 characters
- [ ] Main body is under ~500 lines / ~5000 tokens
- [ ] Translations are complete and consistent with the canonical file
- [ ] `npx skills-ref validate` passes (ignoring `disable-model-invocation` warning)
- [ ] README.md is updated with the new skill entry

---

## Publishing & Distribution

[skills.sh](https://skills.sh) indexes public GitHub repositories automatically. No separate upload step is required.

### Publishing Workflow

1. Ensure the repository is public at `https://github.com/ericmelomp/skills`
2. Push changes to the `main` branch
3. The registry indexes new and updated skills automatically
4. Users install with:
   ```bash
   npx skills add ericmelomp/skills --skill <name>
   ```

### Versioning

This project follows [Semantic Versioning](https://semver.org/):

- **MAJOR** — Breaking changes to skill behavior or interface
- **MINOR** — New skills or non-breaking feature additions
- **PATCH** — Bug fixes, documentation improvements, translation updates

---

## Security Policy

### Reporting Vulnerabilities

If you discover a security vulnerability in any skill, **do not** open a public issue. Instead, report it privately:

- **Email:** [security contact to be defined]
- **Response time:** Within 48 hours for initial acknowledgment

### Security Principles

All skills in this repository follow these security principles:

1. **No secret exposure** — Skills refuse to commit files likely containing secrets (`.env`, `*.pem`, credentials, tokens)
2. **No force-push without consent** — Destructive Git operations require explicit user confirmation
3. **No hook bypass** — `--no-verify` and `--no-gpg-sign` are never used without explicit approval
4. **No configuration mutation** — Skills never write to `git config` without explicit request
5. **Minimal permissions** — Skills operate with the least privilege necessary

---

## Contributing

We welcome contributions from the community. Please review the following before submitting:

### How to Contribute

1. **Fork** this repository
2. **Create a feature branch** from `main`
3. **Make your changes** following the Development Guide above
4. **Validate** with `npx skills-ref validate`
5. **Submit a Pull Request** with a clear description of what and why

### Contribution Standards

- All skills must pass validation
- Translations must be complete (not partial)
- Commit messages must follow [Conventional Commits](https://www.conventionalcommits.org/)
- Documentation changes must be reflected in all affected languages

### Code of Conduct

Contributors are expected to maintain a respectful and inclusive environment. Harassment, discrimination, and toxic behavior are not tolerated.

---

## Support & Communication

| Channel                                                                | Purpose                                 |
| ---------------------------------------------------------------------- | --------------------------------------- |
| [GitHub Issues](https://github.com/ericmelomp/skills/issues)           | Bug reports, feature requests           |
| [GitHub Discussions](https://github.com/ericmelomp/skills/discussions) | Questions, ideas, community interaction |
| [skills.sh](https://skills.sh)                                         | Browse the full skills directory        |

---

## License

Unless noted in a skill's own frontmatter, content in this repository is provided under the **MIT License** for use with AI coding assistants.

```
MIT License

Copyright (c) 2025 ericmelomp

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---
