# Security Notes — Instrument Innovation Agent

This document is the detailed threat model and hardening checklist for maintainers and security reviewers.

---

## Architecture Security Overview

```
Browser (client)
  ├── User input (textarea, checkboxes) — validated, length-capped
  ├── CSP meta header — restricts resource loading
  └── fetch() → ANTHROPIC API or PROXY
                    └── streams SSE → parser → esc() → DOM
```

The application has **no server-side logic by default**. All processing occurs in the browser. The only external communication is a POST to `https://api.anthropic.com/v1/messages`.

---

## Threat Analysis

### T1: Cross-Site Scripting (XSS) via API Response

**Risk**: The Claude API returns strings that could contain `<script>` tags, event handlers, or other HTML if the model is coerced via prompt injection.

**Mitigations implemented**:
1. `esc(s)` — encodes `&`, `<`, `>`, `"`, `'` on every string before `innerHTML`
2. Streaming phase uses `streamPre.textContent` — never `innerHTML`
3. Numeric fields use `safeInt()` — strips non-integer characters
4. `tierColour()` uses an explicit hex allow-list — no dynamic CSS eval
5. CSP `script-src 'self' 'unsafe-inline'` blocks external scripts

**Residual risk**: `'unsafe-inline'` in `script-src` is necessary for the single-file build. Mitigated by extracting JS to `/src/app.js` in production.

**Verification**: Search for `innerHTML` assignments and confirm each interpolated value is wrapped in `esc()` or `safeInt()`.

---

### T2: API Key Exposure

**Risk**: API key hard-coded in client HTML would be visible to anyone with DevTools or access to the source file.

**Mitigations implemented**:
1. No API key is present in source code — `API_ENDPOINT` is a constant pointing to either the Anthropic API (dev) or a proxy (prod)
2. `.gitignore` excludes `.env` and all credential file patterns
3. `.env.example` contains only clearly fake placeholder values
4. `SECURITY.md` warns about immediate rotation if leaked

**Production requirement**: Deploy a server-side proxy that reads `ANTHROPIC_API_KEY` from environment. Never pass the key through to the client.

---

### T3: Prompt Injection via Textarea

**Risk**: A malicious user could craft a query designed to override the system prompt and cause the model to return malicious JSON or exfiltrate data.

**Mitigations implemented**:
1. `maxlength="800"` on the textarea (HTML + JS double-check)
2. User content is appended to the system prompt as a clearly delimited addendum, not interpolated into the structured schema definition
3. The system prompt explicitly specifies JSON-only output, reducing the blast radius of injection

**Residual risk**: Model-level prompt injection cannot be fully prevented client-side. The output sanitisation layer (T1 mitigations) ensures injected HTML/JS cannot execute even if the model is manipulated.

---

### T4: Clickjacking

**Risk**: Embedding this page in an iframe could trick users into interacting with it unknowingly.

**Mitigations implemented**:
1. `<meta http-equiv="X-Frame-Options" content="DENY">`
2. CSP `frame-ancestors 'none'`

**Note**: `meta` CSP is lower-priority than HTTP headers. For production, set the CSP as an HTTP response header from your proxy/CDN.

---

### T5: Information Leakage via Referer

**Risk**: Navigating from this page to Google Fonts could leak the URL in the `Referer` header.

**Mitigation**: `<meta name="referrer" content="strict-origin">` — only sends the origin, not the full path.

---

### T6: Dependency Supply Chain

**Risk**: Google Fonts CDN could serve malicious font files or CSS.

**Mitigations implemented**:
1. CSP `style-src` restricted to `fonts.googleapis.com` only
2. CSP `font-src` restricted to `fonts.gstatic.com` only

**Hardening option**: Self-host fonts and remove all external `font-src` and `style-src` CDN entries.

---

## Hardening Checklist (Production)

- [ ] Move CSP from `<meta>` to HTTP response header
- [ ] Remove `'unsafe-inline'` from `script-src` (requires extracting JS to external file)
- [ ] Deploy server-side proxy; update `API_ENDPOINT` to proxy URL
- [ ] Set `ANTHROPIC_API_KEY` in proxy environment, not in any file
- [ ] Add HTTP `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- [ ] Add HTTP `Permissions-Policy: geolocation=(), camera=(), microphone=()`
- [ ] Enable rate limiting on the proxy (10 req/min per IP recommended)
- [ ] Enable CORS on proxy restricted to your domain only
- [ ] Run `trufflehog` or `gitleaks` in CI on every PR
- [ ] Self-host Google Fonts if operating in high-security environments
- [ ] Review Anthropic API key scopes — limit to `messages:write` only

---

## Secret Scanning Patterns

The following regex patterns are used internally and by CI to detect accidental secret commits:

```
# Anthropic API keys
sk-ant-api\d{2}-[A-Za-z0-9_-]{48,}

# Generic high-entropy strings (base64, 40+ chars)
[A-Za-z0-9+/]{40,}={0,2}

# Bearer tokens
(?i)bearer\s+[A-Za-z0-9._-]{20,}

# Generic "key = value" with long values
(?i)(api[_-]?key|secret|token|password)\s*[=:]\s*['"]?[A-Za-z0-9._-]{16,}
```

---

## Audit Log

| Date | Auditor | Finding | Status |
|------|---------|---------|--------|
| 2025-04-21 | Initial review | `err.message` unescaped in error banner | Fixed — wrapped in `esc()` |
| 2025-04-21 | Initial review | `tierColor` injected directly into style attr | Fixed — replaced with `tierColour()` allow-list |
| 2025-04-21 | Initial review | Numeric fields from API unvalidated in table | Fixed — `safeInt()` applied to all numeric fields |
| 2025-04-21 | Initial review | Stream output used `innerHTML` | Fixed — changed to `textContent` |
| 2025-04-21 | Initial review | `phaseWrapper` used `innerHTML` for static strings | Fixed — replaced with `textContent` DOM API |
