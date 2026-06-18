# Contributing

Thank you for your interest in contributing to this project. This document provides guidelines and processes for contributing effectively.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Skill Authoring Standards](#skill-authoring-standards)
- [Translation Guidelines](#translation-guidelines)
- [Commit Conventions](#commit-conventions)
- [Pull Request Process](#pull-request-process)
- [Review Criteria](#review-criteria)

---

## Code of Conduct

By participating in this project, you agree to maintain a respectful, inclusive, and collaborative environment. We do not tolerate harassment, discrimination, or toxic behavior in any form.

---

## Getting Started

### Prerequisites

- Git ≥ 2.30
- Node.js ≥ 18.x (for validation tooling)
- Familiarity with the [Agent Skills specification](https://agentskills.io/specification)

### Setup

```bash
# Clone the repository
git clone https://github.com/ericmelomp/skills.git
cd skills

# Validate existing skills
npx skills-ref validate ./push
```

---

## Development Workflow

1. **Fork** the repository on GitHub
2. **Clone** your fork locally
3. **Create a feature branch** from `main`:
   ```bash
   git checkout -b feat/my-new-skill
   ```
4. **Make your changes** following the standards below
5. **Validate** your skill:
   ```bash
   npx skills-ref validate ./my-skill
   ```
6. **Commit** with a Conventional Commits message
7. **Push** to your fork and open a Pull Request

---

## Skill Authoring Standards

### Directory Structure

Every skill must live in its own directory with a name matching the `name` field in `SKILL.md`:

```
my-skill/
├── SKILL.md          # Required — YAML frontmatter + instructions
├── pt-BR.md          # Recommended — Brazilian Portuguese translation
├── es.md             # Recommended — Spanish translation
├── scripts/          # Optional — Executable helpers
├── references/       # Optional — Extended documentation
└── assets/           # Optional — Templates and static resources
```

### SKILL.md Requirements

#### Frontmatter (Required)

```yaml
---
name: my-skill
description: >-
  Clear description of what the skill does and when to trigger it.
  Maximum 1024 characters. Include trigger phrases.
disable-model-invocation: true
---
```

#### Field Rules

| Field                      | Constraint                                                                         |
| -------------------------- | ---------------------------------------------------------------------------------- |
| `name`                     | Lowercase letters, numbers, hyphens only. Max 64 chars. Must match directory name. |
| `description`              | Max 1024 chars. Describe purpose AND trigger conditions.                           |
| `disable-model-invocation` | Set to `true` (Cursor-specific, prevents auto-injection).                          |

#### Body Guidelines

- Maximum ~500 lines / ~5000 tokens
- Use numbered steps for the workflow
- Include safety checks and error handling
- Move verbose reference material to `references/`
- Use clear section headers for navigation

### Design Principles

When authoring a skill, follow these principles:

1. **Single Responsibility** — One skill solves one well-defined problem
2. **Explicit Invocation Only** — Never auto-trigger; always require user intent
3. **Safety by Default** — Confirm before any irreversible action
4. **Graceful Degradation** — Handle missing tools, offline scenarios, and edge cases
5. **Idempotent Behavior** — Running the skill twice should not cause issues
6. **Host Agnostic** — Support all major Git hosting platforms
7. **Minimal Assumptions** — Don't assume directory structure, branch naming, or tooling

---

## Translation Guidelines

### When to Translate

- All user-facing skills should have at minimum: English (canonical), Portuguese (BR), and Spanish
- Translations are full documents, not partial snippets

### Translation Rules

1. **Canonical file is always `SKILL.md` in English** — translations reference it
2. **Include a header note** pointing to the canonical file and other translations
3. **Translate all user-facing text** — headings, descriptions, examples, comments
4. **Keep code blocks in English** — commands, file names, and variable names stay as-is
5. **Localize the Conventional Commits spec URL** when available (e.g., `/pt-br/` or `/es/`)
6. **Maintain structural parity** — same sections, same order, same tables

### Translation Header Template

```markdown
> **Tradução (pt-BR).** Este é um documento de leitura em português. O arquivo
> canônico que o agente carrega é [`../SKILL.md`](../SKILL.md) (em inglês). Outras
> traduções: [English](../SKILL.md) · [Español](es.md).
```

---

## Commit Conventions

This project follows [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types

| Type       | Use For                                            |
| ---------- | -------------------------------------------------- |
| `feat`     | New skill or significant capability addition       |
| `fix`      | Bug fix in skill logic or documentation error      |
| `docs`     | Documentation changes (README, CONTRIBUTING, etc.) |
| `refactor` | Restructuring without changing behavior            |
| `chore`    | Maintenance, tooling, CI configuration             |
| `style`    | Formatting only, no logic change                   |

### Scopes

| Scope          | Use For                                     |
| -------------- | ------------------------------------------- |
| `push`         | Changes to the push skill                   |
| `repo`         | Repository-level changes (README, CI, etc.) |
| `i18n`         | Translation additions or updates            |
| `<skill-name>` | Changes to a specific skill                 |

### Examples

```
feat(push): add Azure DevOps pipeline status reporting
fix(push): handle detached HEAD state gracefully
docs(repo): add CONTRIBUTING.md with contribution guidelines
chore(repo): add CI validation workflow
feat(i18n): add French translation for push skill
```

---

## Pull Request Process

### Before Submitting

- [ ] All modified skills pass `npx skills-ref validate`
- [ ] Translations are complete and consistent
- [ ] README.md is updated (if adding a new skill)
- [ ] Commit messages follow Conventional Commits
- [ ] No secrets, credentials, or sensitive data in any file

### PR Template

```markdown
## Summary

Brief description of what this PR does and why.

## Changes

- List of changes made

## Validation

- [ ] `npx skills-ref validate ./skill-name` passes
- [ ] Tested with [AI assistant name]
- [ ] Translations updated (if applicable)

## Notes

Any additional context for reviewers.
```

### Review Process

1. Submit your PR against `main`
2. Maintainer reviews within 5 business days
3. Address feedback through additional commits (no force-push during review)
4. Once approved, maintainer merges via squash-and-merge

---

## Review Criteria

Pull requests are evaluated on:

| Criterion    | Weight | Description                                     |
| ------------ | ------ | ----------------------------------------------- |
| Correctness  | High   | Does the skill work as documented?              |
| Safety       | High   | Are destructive actions properly guarded?       |
| Completeness | High   | Are translations and docs updated?              |
| Clarity      | Medium | Is the skill instruction clear and unambiguous? |
| Consistency  | Medium | Does it follow existing patterns in the repo?   |
| Scope        | Low    | Is the change appropriately sized?              |

### Grounds for Rejection

- Skills that auto-trigger without explicit user invocation
- Missing safety guards for destructive operations
- Incomplete translations (partial translations are worse than none)
- Skills that require undocumented dependencies
- Violations of the security policy

---

## Questions?

Open a [GitHub Discussion](https://github.com/ericmelomp/skills/discussions) for questions about contributing. For security concerns, see [SECURITY.md](SECURITY.md).

---

_Thank you for helping make this project better for everyone._
