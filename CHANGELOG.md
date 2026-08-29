# Changelog

## 2.0.1

### Fixed

- Package the Codex plugin from the repository root so its real `skills/` directory is included in the installed plugin cache.
- Remove the cross-root `codex/skills` symlink that could produce a successful Codex install containing zero skills.
- Document both required Codex CLI steps: registering the marketplace and installing the plugin.

### Compatibility

The six skill names, trigger phrases, and framework-agnostic behavior are unchanged. The patch release changes only Codex packaging and install metadata.

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
