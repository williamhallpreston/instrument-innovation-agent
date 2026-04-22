# 🎵 Instrument Innovation Agent

> A four-phase AI-powered market research engine that identifies meaningful gaps in the global musical instrument landscape — instruments that *should* exist but don't yet.

[![License: MIT](https://img.shields.io/badge/License-MIT-amber.svg)](LICENSE)
[![Security Policy](https://img.shields.io/badge/Security-SECURITY.md-blue)](SECURITY.md)

---

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [How It Works](#how-it-works)
- [Setup & Installation](#setup--installation)
- [Usage](#usage)
- [Security Considerations](#security-considerations)
- [Project Structure](#project-structure)
- [Contributing](#contributing)
- [Code of Conduct](#code-of-conduct)
- [License](#license)

---

## Overview

The Instrument Innovation Agent is a single-page web application that uses the Anthropic Claude API to conduct a rigorous, multi-dimensional analysis of the musical instrument market. It streams results in real time across four structured phases:

| Phase | Description |
|-------|-------------|
| **1 — Market Landscape** | Surveys traditional, hybrid, and emerging instrument categories across demographics and geographies |
| **2 — Gap Identification** | Evaluates unmet needs, tech feasibility, demand signals, and cultural resonance |
| **3 — Instrument Briefs** | Produces structured design briefs for each validated gap |
| **4 — Opportunity Ranking** | Ranks all concepts by Innovation × Market Size × Feasibility × Cultural Timing |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Runtime | Vanilla JavaScript (ES2022, `'use strict'`) |
| Markup | Semantic HTML5 with ARIA attributes |
| Styling | CSS custom properties, no framework |
| Fonts | Google Fonts (Playfair Display, DM Mono, Cormorant Garamond) |
| AI Backend | [Anthropic Claude API](https://docs.anthropic.com) — `claude-sonnet-4-20250514` |
| Streaming | Server-Sent Events (SSE) via `ReadableStream` |
| Build | None required — ships as a single HTML file |
| Proxy (recommended) | Any Node.js/Express or Cloudflare Worker reverse proxy |

**No build step, no npm, no bundler required** for local development. A server-side proxy is strongly recommended for any non-local deployment (see [Security Considerations](#security-considerations)).

---

## How It Works

```
User Browser
    │
    ├── textarea input (max 800 chars, validated)
    ├── focus-area checkboxes → injected into system prompt
    │
    ▼
[Optional: /api/proxy  ←  injects ANTHROPIC_API_KEY from env]
    │
    ▼
Anthropic API  (streaming SSE)
    │
    ▼
SSE parser → accumulated JSON string
    │
    ▼
JSON.parse() → schema validation
    │
    ▼
esc() / safeInt() / tierColour() sanitisers
    │
    ▼
DOM insertion via innerHTML (all values escaped)
```

---

## Setup & Installation

### Prerequisites

- A modern browser (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- An [Anthropic API key](https://console.anthropic.com/)
- For production: a server-side proxy (Node.js 18+ or Cloudflare Workers)

### Local Development (simplest)

1. **Clone the repo**
   ```bash
   git clone https://github.com/your-org/instrument-innovation-agent.git
   cd instrument-innovation-agent
   ```

2. **Copy and configure environment file**
   ```bash
   cp .env.example .env
   # Edit .env — add your ANTHROPIC_API_KEY
   ```

3. **Serve the file locally**

   The app is a static HTML file. You can serve it with any static server:
   ```bash
   # Python (built-in)
   python3 -m http.server 8080 --directory src/

   # Node.js (npx)
   npx serve src/

   # VS Code: use the Live Server extension
   ```

4. **Open** `http://localhost:8080` in your browser.

> ⚠️ **Do not open `index.html` directly as a `file://` URL.** The Fetch API requires an HTTP context; otherwise the streaming request will be blocked by CORS.

### Production Deployment

Direct browser-to-Anthropic API calls expose your API key in network traffic. For production:

1. Deploy a lightweight reverse proxy (see `docs/DEPLOYMENT.md` for examples)
2. Set `ANTHROPIC_API_KEY` as a server-side environment variable
3. Update `API_ENDPOINT` in `src/index.html` to point to your proxy URL
4. Add your domain to your Anthropic API key's allowed-origins list

---

## Usage

### Basic

1. Open the application in your browser
2. Optionally type a **focus directive** (max 800 characters) — e.g.:
   - *"Focus on instruments for players with physical disabilities"*
   - *"Explore gaps in West African musical traditions"*
3. Toggle **Focus Area** checkboxes in the sidebar to narrow the scope
4. Click **▶ Run Full Analysis**
5. Watch the four-phase report stream in

### Presets

Click any preset button to pre-fill the query textarea:

| Preset | Description |
|--------|-------------|
| **Accessibility** | Adaptive instruments, haptic interfaces, motor-impairment design |
| **Non-Western Traditions** | West African, SE Asian, Middle Eastern, Indigenous music |
| **Electronic Frontiers** | Beyond MIDI controllers — pure electronic expression |
| **Children & Education** | Ages 4–12, classroom use, developmental instruments |

### API Response Format

The agent returns a single JSON object matching the schema documented in `docs/API_SCHEMA.md`. All numeric fields are plain integers; all string fields are escaped before DOM insertion.

---

## Security Considerations

See [`SECURITY.md`](SECURITY.md) for the full vulnerability disclosure policy.

### API Key Handling

- **Never** embed a real `ANTHROPIC_API_KEY` in `src/index.html` or any client-side file.
- The `API_ENDPOINT` constant is a placeholder. In production it should point to **your own proxy**, which injects the key server-side.
- Rotate your key immediately if it is ever committed to VCS. See [Anthropic key rotation docs](https://docs.anthropic.com/en/api/getting-started).

### XSS Prevention

All data returned by the Claude API passes through two sanitisation layers before DOM insertion:

- `esc(s)` — HTML-encodes `&`, `<`, `>`, `"`, `'`
- `safeInt(val, min, max)` — coerces numeric fields to bounded integers
- `tierColour(tier)` — maps price tiers to an explicit CSS hex allow-list (no dynamic CSS injection)

Streaming output is written to `textContent` (never `innerHTML`) during the stream phase.

### Content Security Policy

A `<meta>` CSP header is present in `index.html`. For production, move it to an HTTP response header and remove `'unsafe-inline'` by extracting JavaScript to `/src/app.js`.

```
Content-Security-Policy:
  default-src 'none';
  script-src 'self';
  style-src 'self' https://fonts.googleapis.com;
  font-src https://fonts.gstatic.com;
  connect-src https://your-proxy.example.com;
  img-src 'self' data:;
  frame-ancestors 'none';
```

### Input Validation

- User textarea input is capped at **800 characters** (HTML `maxlength` + JS check)
- Input is trimmed and validated before inclusion in the API request
- The API system prompt uses static strings; user input is appended as a clearly delimited optional addendum

### Known Limitations

| Limitation | Mitigation |
|-----------|-----------|
| No server-side rate limiting in the base HTML | Add rate limiting in your proxy layer |
| `'unsafe-inline'` required in single-file build | Extract JS to `/src/app.js` for production |
| Google Fonts loaded from external CDN | Self-host fonts for air-gapped environments |
| No authentication/authorisation layer | Add via proxy or a hosting platform (Cloudflare Access, etc.) |

---

## Project Structure

```
instrument-innovation-agent/
├── src/
│   └── index.html          # Main application (single-file build)
├── docs/
│   ├── DEPLOYMENT.md       # Proxy setup, environment config, hosting guides
│   ├── API_SCHEMA.md       # Full JSON schema for the Claude response
│   └── SECURITY_NOTES.md   # Detailed threat model and hardening checklist
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   └── PULL_REQUEST_TEMPLATE.md
├── .env.example            # Placeholder env vars — never commit a real .env
├── .gitignore
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md               # This file
└── SECURITY.md
```

---

## Contributing

Contributions are welcome! Please read [`CONTRIBUTING.md`](CONTRIBUTING.md) before opening a pull request.

**Quick checklist before submitting a PR:**

- [ ] Run a secret scan locally (see `docs/SECURITY_NOTES.md`)
- [ ] Ensure no API keys, tokens, or credentials are present in any file
- [ ] All user-facing strings from the API pass through `esc()` before `innerHTML`
- [ ] New numeric fields use `safeInt()` before interpolation
- [ ] `CHANGELOG.md` updated

---

## Code of Conduct

This project follows the [Contributor Covenant v2.1](CODE_OF_CONDUCT.md). By participating, you agree to uphold its standards. Report unacceptable behaviour to the maintainers at the address listed in `CODE_OF_CONDUCT.md`.

---

## License

[MIT](LICENSE) © 2026

