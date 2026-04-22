# Security Policy

## Supported Versions

| Version | Supported |
|---------|-----------|
| `main` (latest) | ✅ Active support |
| Older tagged releases | ❌ No security patches |

We only issue security fixes against the current `main` branch. If you are running a pinned older release, please upgrade.

---

## Reporting a Vulnerability

**Please do not open a public GitHub issue for security vulnerabilities.**

We follow a responsible disclosure model. To report a vulnerability:

1. **Email**: Send a description to `security@your-org.example.com`  
   *(Replace with your actual security contact before publishing)*

2. **Subject line**: `[SECURITY] instrument-innovation-agent — <brief description>`

3. **Include in your report**:
   - A clear description of the vulnerability
   - Steps to reproduce (proof-of-concept code if applicable)
   - Affected file(s) and line numbers
   - Potential impact and suggested severity (Critical / High / Medium / Low)
   - Whether you have already developed a fix

4. **Encryption** (optional but encouraged): Our PGP public key is available at  
   `https://your-org.example.com/.well-known/security.txt`

---

## Response Timeline

| Milestone | Target |
|-----------|--------|
| Acknowledgement of report | **Within 48 hours** |
| Initial triage and severity assessment | **Within 5 business days** |
| Fix or mitigation for Critical / High issues | **Within 14 days** |
| Fix or mitigation for Medium / Low issues | **Within 45 days** |
| Public disclosure (coordinated) | **After fix is released** |

We will keep you informed throughout the process and credit you in our `CHANGELOG.md` unless you prefer to remain anonymous.

---

## Scope

### In Scope

- Cross-site scripting (XSS) via API response injection
- API key exposure or leakage (client-side, build artifacts, git history)
- Content Security Policy bypasses
- Server-side request forgery via the proxy layer
- Prompt injection that causes the agent to exfiltrate data or take unintended actions
- Any vulnerability that allows an attacker to run arbitrary code in a user's browser

### Out of Scope

- Vulnerabilities in the Anthropic API itself — report those to [security@anthropic.com](mailto:security@anthropic.com)
- Vulnerabilities in Google Fonts CDN infrastructure
- Social engineering of maintainers
- Denial-of-service attacks that require a large number of authenticated API requests
- Issues only reproducible on end-of-life browsers (IE11, etc.)
- UI/UX issues that are not security relevant

---

## Threat Model Summary

This application is a **single-page static web app** that calls the Anthropic Claude API. The primary threat surface is:

| Threat | Mitigation |
|--------|-----------|
| API key committed to VCS | `.gitignore` excludes `.env*`; CI secret scanning enforced |
| XSS via API response | All API strings pass through `esc()` before `innerHTML`; streaming uses `textContent` |
| CSS injection via `tierColour` | Allow-list of hex values; no dynamic CSS evaluation |
| Prompt injection via textarea | Input capped at 800 chars; user content clearly delimited in system prompt |
| Clickjacking | `X-Frame-Options: DENY` + `frame-ancestors 'none'` in CSP |
| MIME sniffing | `X-Content-Type-Options: nosniff` header |
| Open redirect | No redirect logic in this application |

For a full threat model, see [`docs/SECURITY_NOTES.md`](docs/SECURITY_NOTES.md).

---

## Security Hall of Fame

We gratefully acknowledge researchers who have responsibly disclosed vulnerabilities:

*(None yet — be the first!)*

---

## Contact

Security disclosures: `security@your-org.example.com`  
General contact: `hello@your-org.example.com`  
Anthropic security (for API-level issues): `security@anthropic.com`
