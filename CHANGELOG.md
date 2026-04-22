# Changelog

All notable changes to this project will be documented in this file.  
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).  
This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Security
- Initial security audit completed (2025-04-21)
- Fixed: `err.message` was interpolated unescaped into error banner `innerHTML` — now wrapped in `esc()`
- Fixed: `tierColor` was injected directly into a `style` attribute from API data — replaced with `tierColour()` allow-list
- Fixed: Numeric fields from API response (`rank`, `innovationImpact`, etc.) were interpolated without validation — now use `safeInt()`
- Fixed: Streaming panel used `innerHTML` to render live API output — changed to `textContent` assignment
- Fixed: `phaseWrapper` used `innerHTML` with static strings — replaced with DOM API (`textContent`) for defence in depth
- Added: `maxlength="800"` on textarea plus JS-side length check
- Added: CSP `<meta>` header with `frame-ancestors 'none'`, locked `connect-src`, `font-src`
- Added: `X-Content-Type-Options`, `X-Frame-Options`, `referrer` meta headers
- Added: ARIA attributes for accessibility (`aria-live`, `role="meter"`, `scope` on table headers)

### Added
- Initial project release: four-phase musical instrument innovation agent
- Streaming SSE support for real-time response rendering
- Focus-area checkboxes integrated into system prompt
- Four preset query templates
- `README.md`, `SECURITY.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`
- `docs/DEPLOYMENT.md`, `docs/API_SCHEMA.md`, `docs/SECURITY_NOTES.md`
- `.gitignore` with comprehensive secret and credential exclusions
- `.env.example` with documented placeholders
