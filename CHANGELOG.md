# Changelog

## 2.0.1

### Changed

- `seo`: the ✅ meta description example no longer claims "Rated 4.9/5 by 10,000+ customers" or a 20% discount. An agent that copies the example's pattern may put a rating, a review count or an offer into pages that show none. The example keeps the page's own facts (shipping, returns, what you can compare), and the guidelines add one line: state only ratings, review counts, prices or discounts the page itself shows.

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
