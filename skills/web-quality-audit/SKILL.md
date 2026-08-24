---
name: web-quality-audit
description: Run an evidence-led web quality audit covering performance, accessibility, SEO, best practices, and agentic browsing. Use when asked to "audit my site", "review web quality", "run lighthouse audit", "check page quality", or "optimize my website".
license: MIT
metadata:
  author: web-quality-skills
  version: "2.0"
---

# Web quality audit

Comprehensive quality review that combines live browser evidence with source inspection. Covers Performance, Accessibility, SEO, Best Practices, and Agentic Browsing without treating an aggregate score as proof of quality.

> **Lighthouse 13+.** The Performance category now uses shared **Performance Insights** across Lighthouse and the DevTools Performance panel ([announcement](https://developer.chrome.com/blog/moving-lighthouse-to-insights)). Follow current insight names and evidence. Do not require removed audit IDs or automatically recreate their recommendations; some were retired because they were noisy, inactionable, or easy to over-recommend.

## How it works

1. Establish the audit target: representative URLs, important states and journeys, public versus authenticated access, and mobile/desktop scope.
2. If a page can run, read [the measurement workflow](../performance/references/MEASUREMENT.md) and collect a minimal live baseline before searching the codebase broadly.
3. Use runtime failures to localize source inspection. Keep measured findings separate from hypotheses found only in code.
4. Categorize by user impact and confidence, then make or recommend specific fixes.
5. Re-run equivalent automated checks and the affected manual flows. Report what is verified and what still needs field or human validation.

## Tool routing

Use the best capability already available; do not block the audit on optional setup.

| Need | Preferred route | Fallback |
|------|-----------------|----------|
| Performance and Core Web Vitals | Record a browser performance trace and analyze focused insights; with Chrome DevTools MCP, use `performance_start_trace` then `performance_analyze_insight` | Lighthouse CLI or PageSpeed Insights lab data |
| Real-user performance | CrUX values included in current DevTools trace summaries | PageSpeed Insights/CrUX Vis; direct CrUX API only when a key is already available or automation is requested |
| Accessibility, SEO, Best Practices, Agentic Browsing | Run a live Lighthouse audit; with Chrome DevTools MCP, use `lighthouse_audit` | Category-specific Lighthouse CLI audits plus manual checks |
| Rendered semantics and interaction | Inspect the accessibility tree and exercise the UI; with Chrome DevTools MCP, use `take_snapshot` and focused `evaluate_script` | Browser/manual testing |
| Source smoke test | `scripts/analyze.sh <path>` | Direct source inspection |

Chrome DevTools MCP's `lighthouse_audit` intentionally excludes performance. Its navigation mode reloads the page; use snapshot mode when preserving the current authenticated or user-created state matters. The static analyzer is a fast smoke test, not a substitute for a rendered-page audit.

## Audit categories

### Performance

**Core Web Vitals** — Must pass for good page experience:
* **LCP (Largest Contentful Paint) < 2.5s.** The largest visible element must render quickly. Optimize images, fonts, and server response time.
* **INP (Interaction to Next Paint) < 200ms.** User interactions must feel instant. Reduce JavaScript execution time and break up long tasks.
* **CLS (Cumulative Layout Shift) < 0.1.** Content must not jump around. Set explicit dimensions on images, embeds, and ads.

**Resource Optimization:**
* **Compress images.** Use WebP/AVIF with fallbacks. Serve correctly sized images via `srcset`.
* **Minimize JavaScript.** Remove unused code. Use code splitting. Defer non-critical scripts.
* **Optimize CSS.** Extract critical CSS. Remove unused styles. Avoid `@import`.
* **Efficient fonts.** Use `font-display: swap`. Preload critical fonts. Subset to needed characters.

**Loading Strategy:**
* **Preconnect to origins.** Add `<link rel="preconnect">` for third-party domains.
* **Preload critical assets.** LCP images, fonts, and above-fold CSS.
* **Lazy load below-fold content.** Images, iframes, and heavy components.
* **Cache effectively.** Long cache TTLs for static assets. Immutable caching for hashed files.

### Accessibility

**Perceivable:**
* **Text alternatives.** Every `<img>` has meaningful `alt` text. Decorative images use `alt=""`.
* **Color contrast.** Minimum 4.5:1 for normal text, 3:1 for large text (WCAG AA).
* **Don't rely on color alone.** Use icons, patterns, or text alongside color indicators.
* **Captions and transcripts.** Video has captions. Audio has transcripts.

**Operable:**
* **Keyboard accessible.** All functionality available via keyboard. No keyboard traps.
* **Focus visible.** Clear focus indicators on all interactive elements.
* **Skip links.** Provide "Skip to main content" for keyboard users.
* **Sufficient time.** Users can extend time limits. No auto-advancing content without controls.

**Understandable:**
* **Page language.** Set `lang` attribute on `<html>`.
* **Consistent navigation.** Same navigation structure across pages.
* **Error identification.** Form errors clearly described and associated with fields.
* **Labels and instructions.** All form inputs have associated labels.

**Robust:**
* **Valid HTML.** No duplicate IDs. Properly nested elements.
* **ARIA used correctly.** Prefer native elements. ARIA roles match behavior.
* **Name, role, value.** Interactive elements have accessible names and correct roles.

### SEO

**Crawlability:**
* **Valid robots.txt.** Doesn't block important resources.
* **XML sitemap.** Lists all important pages. Submitted to Search Console.
* **Canonical URLs.** Prevent duplicate content issues.
* **No noindex on important pages.** Check meta robots and headers.

**On-Page SEO:**
* **Unique title tags.** Make each title descriptive and concise; display truncation varies by device and result type.
* **Meta descriptions.** Write useful, page-specific summaries; search engines may choose a different snippet.
* **Heading hierarchy.** The primary heading is descriptive and the structure is logical; do not fail valid HTML solely for using more than one `<h1>`.
* **Descriptive link text.** Not "click here" or "read more".

**Technical SEO:**
* **Mobile-friendly.** Responsive design. Tap targets ≥ 48px.
* **HTTPS.** Secure connection required.
* **Page experience signals.** Use field Core Web Vitals as evidence, without promising a ranking change.
* **Structured data.** JSON-LD for rich snippets (Article, Product, FAQ, etc.).

### Best practices

**Security:**
* **HTTPS everywhere.** No mixed content. HSTS enabled.
* **No vulnerable libraries.** Keep dependencies updated.
* **CSP headers.** Content Security Policy to prevent XSS.
* **No exposed source maps.** In production builds.

**Modern Standards:**
* **No deprecated APIs.** Replace `document.write`, synchronous XHR, etc.
* **Valid doctype.** Use `<!DOCTYPE html>`.
* **Charset declared.** `<meta charset="UTF-8">` as first element in `<head>`.
* **No browser errors.** Clean console. No CORS issues.

**UX Patterns:**
* **No intrusive interstitials.** Especially on mobile.
* **Clear permission requests.** Only ask when needed, with context.
* **No misleading buttons.** Buttons do what they say.

### Agentic browsing

Use the Lighthouse Agentic Browsing results as technical signals for how well assistants can understand and interact with the rendered page.

* **Accessible interaction surface.** Semantic HTML, labels, names, roles, and states must expose meaningful controls in the accessibility tree.
* **WebMCP integrations are valid when present.** Review registered tools, schemas, and form coverage; do not add WebMCP solely to raise an audit score.
* **`llms.txt` is optional.** A valid file may help compatible tools discover curated content, but a Lighthouse pass does not prove that search or AI products will ingest, rank, or cite it.
* **Keep this category separate from SEO claims.** Agentic browsability is not evidence of search ranking or AI visibility.

## Severity levels

| Level | Description | Action |
|-------|-------------|--------|
| **Critical** | Security vulnerabilities, complete failures | Fix immediately |
| **High** | Core Web Vitals failures, major a11y barriers | Fix before launch |
| **Medium** | Performance opportunities, SEO improvements | Fix within sprint |
| **Low** | Minor optimizations, code quality | Fix when convenient |

## Audit output format

When performing an audit, structure findings as:

```markdown
## Audit results

### Evidence
| Signal | Scope/conditions | Result | Source |
|--------|------------------|--------|--------|
| LCP | URL, phone, p75/28 days | 3.1s (needs improvement) | CrUX |
| Accessibility | URL, mobile navigation | 92 | Lighthouse |

### Critical issues (X found)
- **[Category]** Issue description. File: `path/to/file.js:123`
  - **Impact:** Why this matters
  - **Evidence:** Measured failure, runtime observation, or source hypothesis
  - **Fix:** Specific code change or recommendation

### High priority (X found)
...

### Summary
- Performance: measured status and X findings
- Accessibility: automated status, X findings, manual checks pending/passed
- SEO: X findings
- Best Practices: X findings
- Agentic Browsing: X findings or not available

### Recommended priority
1. First fix this because...
2. Then address...
3. Finally optimize...

### Verification
- Re-run results under the same conditions
- Manual checks completed
- Field validation still pending
```

## Quick checklist

### Before every deploy
- [ ] Core Web Vitals passing
- [ ] No accessibility errors (axe/Lighthouse)
- [ ] No console errors
- [ ] HTTPS working
- [ ] Meta tags present

### Weekly review
- [ ] Check Search Console for issues
- [ ] Review Core Web Vitals trends
- [ ] Update dependencies
- [ ] Test with screen reader

### Monthly deep dive
- [ ] Full Lighthouse audit
- [ ] Performance profiling
- [ ] Accessibility audit with real users
- [ ] SEO keyword review

## References

For detailed guidelines on specific areas:
- [Performance Optimization](../performance/SKILL.md)
- [Core Web Vitals](../core-web-vitals/SKILL.md)
- [Accessibility](../accessibility/SKILL.md)
- [SEO](../seo/SKILL.md)
- [Best Practices](../best-practices/SKILL.md)
