# Changelog

## Unreleased

### Fixed

- best-practices 2.0.1: show `Permissions-Policy` as an HTTP response header with double-quoted origins. Browsers ignore it in `<meta http-equiv>`, so the old example restricted nothing.
- best-practices 2.0.1: in the security reference, mark the CSP nonce as a per-response random value, name the Trusted Types policy `app-html` so raw `innerHTML` strings are rejected as the comment says (a `default` policy would sanitize them instead), and say that `structuredClone` throws on functions rather than dropping them.

## 2.0.0

Version 2 moves the skills from broad source-first advice to evidence-led audits.

### Changed

- Prefer browser performance traces for performance diagnosis and live Lighthouse audits for Accessibility, SEO, Best Practices, and Agentic Browsing when those capabilities are available.
- Separate CrUX field data, first-party RUM, repeatable lab measurements, single-session browser observations, and source-code hypotheses.
- Add low-friction fallbacks for environments without Chrome DevTools MCP.
- Move detailed measurement, RUM, security, structured-data, INP, and CLS guidance into focused references.
- Remove unsupported ranking percentages, hard metadata limits, TTI guidance, and generic performance-savings estimates while retaining sourced implementation detail.
- Correct the DOMPurify Trusted Types policy example so the policy callback returns a string (`RETURN_TRUSTED_TYPE: false`).

### Compatibility

The six skill names, trigger phrases, installation paths, package layouts, and framework-agnostic defaults remain compatible with version 1.

Intentional behavior changes are evidence-related: one `PerformanceObserver` session is no longer described as field data, performance no longer routes through the Lighthouse-for-agents audit when a trace is available, and ranking or field improvements are not claimed without the corresponding evidence.
