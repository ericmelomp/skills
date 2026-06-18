# Security Policy

## Supported Versions

| Version           | Supported              |
| ----------------- | ---------------------- |
| Latest (main)     | ✅ Yes                 |
| Previous releases | ⚠️ Critical fixes only |

## Reporting a Vulnerability

If you discover a security vulnerability in any skill within this repository, **do not** open a public GitHub issue.

### Private Disclosure Process

1. **Email** your report to the maintainer with the subject line: `[SECURITY] CoreScale Skills — <brief description>`
2. Include:
   - Affected skill(s) and file(s)
   - Steps to reproduce
   - Potential impact assessment
   - Suggested fix (if available)
3. **Response timeline:**
   - Initial acknowledgment: within **48 hours**
   - Assessment and triage: within **5 business days**
   - Fix or mitigation: within **30 days** for critical issues

### What Qualifies as a Security Issue

- Skills that could commit or expose secrets (API keys, tokens, private keys)
- Bypass of safety guardrails (e.g., force-push without consent)
- Command injection through crafted repository state
- Privilege escalation in skill execution context
- Unintended data exfiltration through Git operations

### What Does NOT Qualify

- Feature requests or general bugs (use GitHub Issues)
- Issues in third-party tools (Git, `gh`, `glab`) — report to their maintainers
- Theoretical attacks requiring pre-existing system compromise

## Security Design Principles

All skills in this repository are designed with the following principles:

### 1. Explicit Consent for Destructive Actions

No skill will perform irreversible operations without explicit user confirmation:

- `git push --force`
- `git commit --amend`
- `--no-verify` (hook bypass)
- `git config` writes
- Deletion of branches or refs

### 2. Secret Detection

Skills actively refuse to stage or commit files matching known secret patterns:

- `.env`, `.env.*`
- `*.pem`, `*.key`
- `credentials.*`, `secrets.*`
- Token files and API key stores

### 3. No Proactive Execution

Skills never auto-trigger. They execute only on direct, explicit user invocation (e.g., `/push`). This eliminates an entire class of supply-chain attack vectors.

### 4. Minimal Surface Area

- Skills do not install dependencies
- Skills do not execute arbitrary code
- Skills do not modify system configuration
- Skills operate within the Git repository scope only

### 5. Auditability

Every action a skill takes is visible in the AI assistant's conversation context. Users can review, approve, or reject any operation before it executes.

## Dependency Security

This project has **zero runtime dependencies**. Skills are plain Markdown files parsed by the AI assistant runtime. There are no `node_modules`, no compiled binaries, and no package dependencies to audit.

Development tools (e.g., `npx skills-ref validate`) are fetched on-demand and are not bundled.

## Responsible Disclosure

We follow a coordinated disclosure model:

1. Reporter submits vulnerability privately
2. Maintainers acknowledge and assess
3. Fix is developed and tested
4. Fix is released with a security advisory
5. Reporter is credited (unless they prefer anonymity)

We will not pursue legal action against security researchers who act in good faith and follow this disclosure process.

---

_Last updated: June 2025_
