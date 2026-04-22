# Contributing to Instrument Innovation Agent

Thank you for your interest in contributing! This document explains how to get involved effectively and safely.

---

## Getting Started

1. **Fork** the repository and clone your fork locally.
2. Create a **feature branch**: `git checkout -b feat/your-feature-name`
3. Make your changes (see guidelines below).
4. **Run the secret scan** before committing (see Security section).
5. Open a **Pull Request** against `main`.

---

## Development Guidelines

### Code Style

- Vanilla JavaScript, `'use strict'`, ES2022 syntax
- Function declarations over anonymous assignments for named logic
- Comment non-obvious code; add `// SECURITY:` comments on all auth/input/DOM-injection paths
- Keep the single-file constraint: CSS in `<style>`, JS in `<script>` — no external bundling required

### Security Requirements for PRs

Every pull request **must** satisfy:

- [ ] No secrets, API keys, tokens, or credentials in any file (including test fixtures)
- [ ] All new API response fields passed through `esc()` before `innerHTML`
- [ ] All new numeric fields from API response passed through `safeInt()`
- [ ] Any new CSS values derived from API data use an explicit allow-list (like `tierColour()`)
- [ ] No `eval()`, `Function()`, `document.write()`, or `innerHTML` with unescaped user input
- [ ] CSP `meta` tag remains intact and not loosened without discussion
- [ ] `CHANGELOG.md` entry added

### Commit Messages

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add export-to-PDF button for phase reports
fix: escape overallScore before DOM insertion
security: replace direct innerHTML with textContent in stream panel
docs: add proxy setup guide to DEPLOYMENT.md
```

### Secret Scanning Before Push

Run a local scan before `git push`:

```bash
# Using truffleHog (recommended)
pip install trufflehog
trufflehog filesystem . --only-verified

# Or gitleaks
gitleaks detect --source . --verbose
```

If any findings are flagged, **do not push**. Remediate and use `git filter-repo` or BFG to remove history if a secret was already committed.

---

## Reporting Bugs

Please use the [Bug Report template](.github/ISSUE_TEMPLATE/bug_report.md).  
For **security vulnerabilities**, follow [`SECURITY.md`](SECURITY.md) instead — do not open a public issue.

---

## Feature Requests

Use the [Feature Request template](.github/ISSUE_TEMPLATE/feature_request.md). Features that expand the attack surface (new external fetches, file uploads, auth systems) will require a short security design note alongside the implementation.

---

## Questions?

Open a [Discussion](https://github.com/your-org/instrument-innovation-agent/discussions) rather than an issue for general questions.
