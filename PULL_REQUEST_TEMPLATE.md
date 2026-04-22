## Summary

<!-- What does this PR do? -->

## Type of Change

- [ ] Bug fix
- [ ] New feature
- [ ] Security fix
- [ ] Documentation
- [ ] Refactor

## Security Checklist

- [ ] No secrets, API keys, or credentials in any file
- [ ] All new API response strings pass through `esc()` before `innerHTML`
- [ ] All new numeric fields use `safeInt()`
- [ ] No `eval()`, `Function()`, or unescaped `innerHTML` introduced
- [ ] CSP header not loosened
- [ ] Secret scan run locally (truffleHog / gitleaks)
- [ ] `CHANGELOG.md` updated

## Testing

<!-- How was this tested? -->
