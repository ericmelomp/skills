# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] — 2025-06-18

### Added

- **push** skill — Automated commit message generation and push workflow
  - Conventional Commits compliance with live spec fetching
  - Multi-repo workspace discovery (submodules, worktrees, monorepos)
  - Universal host support (GitHub, GitLab, Bitbucket, Azure DevOps, Gitea, self-hosted)
  - Safety guardrails (force-push protection, secret detection, hook respect)
  - Preview-before-action workflow
  - Error recovery guidance for common Git scenarios
- Full translation: Brazilian Portuguese (`push/pt-BR.md`)
- Full translation: Spanish (`push/es.md`)
- Repository documentation
  - `README.md` — Project overview and quick start
  - `SECURITY.md` — Security policy and vulnerability reporting
  - `CONTRIBUTING.md` — Contribution guidelines and standards
  - `CHANGELOG.md` — Version history

---

## [Unreleased]

_No unreleased changes._

---

### Version History Format

Each release entry includes:

- **Added** — New features or skills
- **Changed** — Changes to existing functionality
- **Deprecated** — Features to be removed in future versions
- **Removed** — Features removed in this version
- **Fixed** — Bug fixes
- **Security** — Vulnerability fixes

---

[1.0.0]: https://github.com/ericmelomp/skills/releases/tag/v1.0.0
[Unreleased]: https://github.com/ericmelomp/skills/compare/v1.0.0...HEAD
