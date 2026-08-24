---
name: seo
description: Optimize for search engine visibility and ranking. Use when asked to "improve SEO", "optimize for search", "fix meta tags", "add structured data", "sitemap optimization", or "search engine optimization".
license: MIT
metadata:
  author: web-quality-skills
  version: "2.0"
---

# SEO optimization

Search engine optimization based on Lighthouse SEO audits and Google Search guidelines. Focus on technical SEO, on-page optimization, and structured data.

## Evidence-led audit workflow

When a rendered page is available:

1. Run live Lighthouse SEO and Agentic Browsing checks when that capability is available; with Chrome DevTools MCP, use `lighthouse_audit`. Use the results to localize rendered-page failures.
2. Inspect signals Lighthouse cannot establish on its own: response headers, redirects, `robots.txt`, sitemap coverage, canonical consistency across page templates, structured-data eligibility, and Search Console evidence when the user provides access.
3. Separate technical crawl/index findings from content quality and authority. Do not invent ranking-factor weights or promise ranking changes.
4. Fix the source and re-run the same checks. For indexation or ranking outcomes, report that search-engine validation remains pending.

If live tools are unavailable, use category-specific Lighthouse CLI output plus direct source and HTTP inspection. A Lighthouse SEO score covers a useful subset of technical checks; it is not a prediction of rankings.

| Area | What this skill can verify |
|------|----------------------------|
| Crawl and index controls | Technical configuration and consistency |
| Rendered metadata and semantics | Presence, validity, and page-template issues |
| Structured data | Syntax and eligibility signals, not guaranteed rich results |
| Core Web Vitals | Link to measured field/lab evidence from the Core Web Vitals skill |
| Content usefulness and authority | Review quality, but do not assign synthetic ranking percentages |

---

## Technical SEO

### Crawlability

**robots.txt:**
```text
# /robots.txt
User-agent: *
Allow: /

# Block admin/private areas
Disallow: /admin/
Disallow: /api/
Disallow: /private/

# Don't block resources needed for rendering
# ❌ Disallow: /static/

Sitemap: https://example.com/sitemap.xml
```

**Meta robots:**
```html
<!-- Default: indexable, followable -->
<meta name="robots" content="index, follow">

<!-- Noindex specific pages -->
<meta name="robots" content="noindex, nofollow">

<!-- Indexable but don't follow links -->
<meta name="robots" content="index, nofollow">

<!-- Control snippets -->
<meta name="robots" content="max-snippet:150, max-image-preview:large">
```

**Canonical URLs:**
```html
<!-- Prevent duplicate content issues -->
<link rel="canonical" href="https://example.com/page">

<!-- Self-referencing canonical (recommended) -->
<link rel="canonical" href="https://example.com/current-page">

<!-- For paginated content -->
<link rel="canonical" href="https://example.com/products">
<!-- Or use rel="prev" / rel="next" for explicit pagination -->
```

### XML sitemap

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/</loc>
    <lastmod>2024-01-15</lastmod>
    <changefreq>daily</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://example.com/products</loc>
    <lastmod>2024-01-14</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.8</priority>
  </url>
</urlset>
```

**Sitemap best practices:**
- Maximum 50,000 URLs or 50MB per sitemap
- Use sitemap index for larger sites
- Include only canonical, indexable URLs
- Update `lastmod` when content changes
- Submit to Google Search Console

### URL structure

```
✅ Good URLs:
https://example.com/products/blue-widget
https://example.com/blog/how-to-use-widgets

❌ Poor URLs:
https://example.com/p?id=12345
https://example.com/products/item/category/subcategory/blue-widget-2024-sale-discount
```

**URL guidelines:**
- Use hyphens, not underscores
- Lowercase only
- Keep short (< 75 characters)
- Include target keywords naturally
- Avoid parameters when possible
- Use HTTPS always

### HTTPS & security

```html
<!-- Ensure all resources use HTTPS -->
<img src="https://example.com/image.jpg">

<!-- Not: -->
<img src="http://example.com/image.jpg">
```

**Security headers for SEO trust signals:**
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
```

---

## On-page SEO

### Title tags

```html
<!-- ❌ Missing or generic -->
<title>Page</title>
<title>Home</title>

<!-- ✅ Descriptive with primary keyword -->
<title>Blue Widgets for Sale | Premium Quality | Example Store</title>
```

**Title tag guidelines:**
- Use 50–60 characters only as a rough linting proxy, not a pass/fail limit. Google truncates title links to fit the rendered device width, so preview width when tooling supports it.
- Describe the page topic naturally near the beginning
- Unique for every page
- Add the brand when it helps users distinguish the result
- Action-oriented when appropriate

Treat title-link rewriting separately from truncation. Google may build a different title link from the visible page title, headings, anchor text, and other sources even when the `<title>` is short; investigate accuracy and consistency rather than shortening it automatically. See [Google's title-link guidance](https://developers.google.com/search/docs/appearance/title-link).

### Meta descriptions

```html
<!-- ❌ Missing or duplicate -->
<meta name="description" content="">

<!-- ✅ Compelling and unique -->
<meta name="description" content="Shop premium blue widgets with free shipping. 30-day returns. Rated 4.9/5 by 10,000+ customers. Order today and save 20%.">
```

**Meta description guidelines:**
- Use roughly 150–160 characters only as a linting proxy. Snippets are query- and device-dependent, and Google may select page content instead of the meta description.
- Use the page topic naturally
- Compelling call-to-action
- Unique for every page
- Matches page content

### Heading structure

```html
<!-- ❌ Poor structure -->
<h2>Welcome to Our Store</h2>
<h4>Products</h4>
<h1>Contact Us</h1>

<!-- ✅ Proper hierarchy -->
<h1>Blue Widgets - Premium Quality</h1>
  <h2>Product Features</h2>
    <h3>Durability</h3>
    <h3>Design</h3>
  <h2>Customer Reviews</h2>
  <h2>Pricing</h2>
```

**Heading guidelines:**
- Make the primary page heading descriptive and the hierarchy unambiguous; do not fail a page solely because valid HTML contains more than one `<h1>`
- Logical hierarchy (don't skip levels)
- Include keywords naturally
- Descriptive, not generic

### Image SEO

```html
<!-- ❌ Poor image SEO -->
<img src="IMG_12345.jpg">

<!-- ✅ Optimized image -->
<img src="blue-widget-product-photo.webp"
     alt="Blue widget with chrome finish, side view showing control panel"
     width="800"
     height="600"
     loading="lazy">
```

**Image guidelines:**
- Descriptive filenames with keywords
- Alt text describes the image content
- Compressed and properly sized
- WebP/AVIF with fallbacks
- Lazy load below-fold images

### Internal linking

```html
<!-- ❌ Non-descriptive -->
<a href="/products">Click here</a>
<a href="/widgets">Read more</a>

<!-- ✅ Descriptive anchor text -->
<a href="/products/blue-widgets">Browse our blue widget collection</a>
<a href="/guides/widget-maintenance">Learn how to maintain your widgets</a>
```

**Linking guidelines:**
- Descriptive anchor text with keywords
- Link to relevant internal pages
- Reasonable number of links per page
- Fix broken links promptly
- Use breadcrumbs for hierarchy

---

## Structured data (JSON-LD)

Read [the structured data reference](references/STRUCTURED-DATA.md) when the user requests schema markup or an audit surfaces a structured-data issue. It contains Organization, Article, Product, FAQ, and Breadcrumb examples plus validation links.

* **Describe visible, accurate content.** Do not add a type or claim solely to obtain a rich result.
* **Use the most specific applicable type.** Keep identifiers and absolute URLs stable across renders.
* **Validate rendered output.** Passing syntax does not guarantee search-engine eligibility or display.

## Agentic browsing and AI discoverability

Keep these concepts separate:

* **Lighthouse Agentic Browsing** measures technical signals that help an assistant understand and interact with the rendered page. Current checks include the agent-facing accessibility tree, optional `llms.txt`, and WebMCP registrations, schemas, and form coverage when present.
* **Search indexing and ranking** depend on search-engine systems and cannot be inferred from the Agentic Browsing score.
* **AI ingestion or citation** is product-specific. A technically browsable page or valid `llms.txt` file does not prove that an AI product will ingest, rank, or cite it.

Prioritize semantic HTML, descriptive labels, crawlable content, accurate metadata, and clear page structure because they benefit people, search engines, and agents. Add WebMCP tools only when the application has useful actions to expose and the user wants that integration; validate tool names, descriptions, schemas, and form annotations with Lighthouse.

### Crawler controls are product-specific

Audit each documented user agent separately instead of applying a blanket "AI bot" rule:

| Control | Documented purpose | Effect of blocking |
|---------|--------------------|--------------------|
| `OAI-SearchBot` | ChatGPT search discovery | Prevents page content from being included in ChatGPT summaries and snippets; a link and title may still surface through third-party discovery |
| `PerplexityBot` | Perplexity search indexing | Prevents that crawler from indexing the blocked content for search results |
| `Claude-SearchBot` / `Claude-User` | Claude search indexing / user-directed retrieval | May reduce search visibility / prevents retrieval for user-directed requests |
| `Google-Extended` | Controls certain Gemini training and grounding uses of content Google crawls | Does not affect Google Search inclusion or ranking |

Training controls such as `GPTBot` and `ClaudeBot` are distinct from search and user-fetch controls. `GoogleOther` is a generic crawler, not an AI-search visibility switch. Verify current names and consequences in the vendors' maintained documentation: [OpenAI](https://help.openai.com/en/articles/12627856-publishers-and-developers-faq), [Perplexity](https://docs.perplexity.ai/docs/resources/perplexity-crawlers), [Anthropic](https://privacy.anthropic.com/en/articles/8896518-does-anthropic-crawl-data-from-the-web-and-how-can-site-owners-block-the-crawler), and [Google](https://developers.google.com/crawling/docs/crawlers-fetchers/google-common-crawlers).

### `llms.txt` is optional

`llms.txt` is an experimental proposal, not a cross-vendor discovery standard. Lighthouse can validate the availability and shape of `/llms.txt`, but that does not show that a target product reads it. Add one only when the user requests it or a documented consumer supports it; do not recommend it ahead of crawlability, semantic HTML, accurate metadata, and useful content. Never treat it as a ranking or citation factor, duplicate the sitemap, or reorganize content solely to raise this audit.

---

## Mobile SEO

### Responsive design

```html
<!-- ❌ Not mobile-friendly -->
<meta name="viewport" content="width=1024">

<!-- ✅ Responsive viewport -->
<meta name="viewport" content="width=device-width, initial-scale=1">
```

### Tap targets

```css
/* ❌ Too small for mobile */
.small-link {
  padding: 4px;
  font-size: 12px;
}

/* ✅ Adequate tap target */
.mobile-friendly-link {
  padding: 12px;
  font-size: 16px;
  min-height: 48px;
  min-width: 48px;
}
```

### Font sizes

```css
/* ❌ Too small on mobile */
body {
  font-size: 10px;
}

/* ✅ Readable without zooming */
body {
  font-size: 16px;
  line-height: 1.5;
}
```

---

## International SEO

### Hreflang tags

```html
<!-- For multi-language sites -->
<link rel="alternate" hreflang="en" href="https://example.com/page">
<link rel="alternate" hreflang="es" href="https://example.com/es/page">
<link rel="alternate" hreflang="fr" href="https://example.com/fr/page">
<link rel="alternate" hreflang="x-default" href="https://example.com/page">
```

### Language declaration

```html
<html lang="en">
<!-- or -->
<html lang="es-MX">
```

---

## SEO audit checklist

### Critical
- [ ] HTTPS enabled
- [ ] robots.txt allows crawling
- [ ] No `noindex` on important pages
- [ ] Title tags present and unique
- [ ] Primary page heading is descriptive and the hierarchy is logical

### High priority
- [ ] Meta descriptions present
- [ ] Sitemap submitted
- [ ] Canonical URLs set
- [ ] Mobile-responsive
- [ ] Core Web Vitals passing

### Medium priority
- [ ] Structured data implemented
- [ ] Internal linking strategy
- [ ] Image alt text
- [ ] Descriptive URLs
- [ ] Breadcrumb navigation
- [ ] Agentic Browsing failures reviewed when agent access matters

### Ongoing
- [ ] Fix crawl errors in Search Console
- [ ] Update sitemap when content changes
- [ ] Monitor ranking changes
- [ ] Check for broken links
- [ ] Review Search Console insights

---

## Tools

| Tool | Use |
|------|-----|
| Google Search Console | Monitor indexing, fix issues |
| Google PageSpeed Insights | Performance + Core Web Vitals |
| Rich Results Test | Validate structured data |
| Live Lighthouse audit (Chrome DevTools MCP: `lighthouse_audit`) | Rendered SEO and Agentic Browsing checks for agents |
| Lighthouse CLI | SEO audit fallback |
| Screaming Frog | Crawl analysis |

## References

- [Google Search Central](https://developers.google.com/search)
- [Schema.org](https://schema.org/)
- [Core Web Vitals](../core-web-vitals/SKILL.md)
- [Web Quality Audit](../web-quality-audit/SKILL.md)
