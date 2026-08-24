---
name: performance
description: Optimize web performance for faster loading and better user experience. Use when asked to "speed up my site", "optimize performance", "reduce load time", "fix slow loading", "improve page speed", or "performance audit".
license: MIT
metadata:
  author: web-quality-skills
  version: "2.0"
---

# Performance optimization

Evidence-led performance optimization using real-user signals for prioritization and browser traces for diagnosis. Focuses on loading speed, runtime responsiveness, and resource delivery.

## How it works

1. If a page can run, read [the measurement workflow](references/MEASUREMENT.md) and establish a field-plus-lab baseline before editing.
2. Prioritize poor real-user Core Web Vitals. Use a DevTools performance trace and its focused insights to find the cause.
3. Inspect and change only the code or assets connected to measured bottlenecks.
4. Re-run equivalent lab measurements and report before/after values, conditions, and uncertainty. Field verification remains pending until enough new user data arrives.

When no runnable page exists, perform static inspection but call findings **hypotheses**, not measured regressions. Include the command or browser workflow that can verify each high-impact hypothesis.

Prefer a browser tool that records a performance trace and exposes focused insights. With Chrome DevTools MCP, use `performance_start_trace` and `performance_analyze_insight`; do not route performance through `lighthouse_audit`, which covers non-performance Lighthouse categories.

## Starting performance budget

Budgets must reflect the product's target devices, networks, page types, and user journeys. The values below are initial guardrails for a typical content or commerce page, not universal pass/fail criteria. Preserve an existing project budget when one is already defined.

| Resource | Budget | Rationale |
|----------|--------|-----------|
| Total page weight | < 1.5 MB | Bounds transfer time and data cost on constrained target networks; calibrate with representative pages |
| JavaScript (compressed) | < 300 KB | Protect parse and execution cost |
| CSS (compressed) | < 100 KB | Limit render-blocking work |
| Images (above-fold) | < 500 KB | Protect likely LCP resources |
| Fonts | < 100 KB | Limit critical font transfer |
| Third-party | < 200 KB | Bound code outside product control |

## Critical rendering path

### Server response
* **TTFB < 800ms.** Time to First Byte should be fast. Use CDN, caching, and efficient backends.
* **Enable compression.** Gzip or Brotli for text assets. Brotli preferred (15-20% smaller).
* **HTTP/2 or HTTP/3.** Multiplexing reduces connection overhead.
* **Edge caching.** Cache HTML at CDN edge when possible.
* **Consider Early Hints (HTTP 103) for measured document latency.** If a trace shows slow HTML generation and stable critical subresources, send an interim `103` with `Link` headers before the normal final response from the same request. Use HTTP/2 or later. A CDN may synthesize the `103` from `Link` headers on an earlier `200`, or the origin/edge handler can emit it directly. Unsupported clients continue to the final response, but confirm current browser and infrastructure support. Limit hints to proven critical preloads or preconnects: inaccurate hints waste bandwidth. Cloudflare reported a 20–30% LCP improvement in an artificial, image-heavy test; treat that as a vendor case study, not an expected saving, and measure your result. See [MDN's 103 implementation example](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/103) and [the Cloudflare study](https://blog.cloudflare.com/early-hints-performance/).

### Resource loading

**Preconnect to required origins:**
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://cdn.example.com" crossorigin>
```

**Preload critical resources:**

Preload only resources whose late discovery is visible in the trace. Each preload competes for bandwidth and an unnecessary high-priority request can delay LCP.

```html
<!-- LCP image -->
<link rel="preload" href="/hero.webp" as="image" fetchpriority="high">

<!-- Critical font -->
<link rel="preload" href="/font.woff2" as="font" type="font/woff2" crossorigin>
```

**Prerender likely-next navigations** with the [Speculation Rules API](https://developer.chrome.com/docs/web-platform/prerender-pages):
```html
<script type="speculationrules">
{
  "prerender": [{
    "where": { "href_matches": "/*" },
    "eagerness": "moderate"
  }]
}
</script>
```
`moderate` waits for a stronger intent signal than eager modes. Measure prediction hit rate, transferred bytes, and server cost; a wrong prerender is roughly an unused navigation. See [core-web-vitals → LCP](../core-web-vitals/SKILL.md#lcp-largest-contentful-paint) for the tradeoffs and the `prerenderingchange` gating needed for analytics.

**Defer non-critical CSS:**
```html
<!-- Critical CSS inlined -->
<style>/* Above-fold styles */</style>

<!-- Non-critical CSS -->
<link rel="preload" href="/styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="/styles.css"></noscript>
```

### JavaScript optimization

**Defer non-essential scripts:**
```html
<!-- Parser-blocking (avoid) -->
<script src="/critical.js"></script>

<!-- Deferred (preferred) -->
<script defer src="/app.js"></script>

<!-- Async (for independent scripts) -->
<script async src="/analytics.js"></script>

<!-- Module (deferred by default) -->
<script type="module" src="/app.mjs"></script>
```

**Code splitting patterns:**
```javascript
// Route-based splitting
const Dashboard = lazy(() => import('./Dashboard'));

// Component-based splitting
const HeavyChart = lazy(() => import('./HeavyChart'));

// Feature-based splitting
if (user.isPremium) {
  const PremiumFeatures = await import('./PremiumFeatures');
}
```

**Tree shaking best practices:**
```javascript
// ❌ Imports entire library
import _ from 'lodash';
_.debounce(fn, 300);

// ✅ Imports only what's needed
import debounce from 'lodash/debounce';
debounce(fn, 300);
```

## Image optimization

### Format selection
| Format | Use case | Browser support |
|--------|----------|-----------------|
| AVIF | Photos, best compression | 92%+ |
| WebP | Photos, good fallback | 97%+ |
| PNG | Graphics with transparency | Universal |
| SVG | Icons, logos, illustrations | Universal |

### Responsive images
```html
<picture>
  <!-- AVIF for modern browsers -->
  <source 
    type="image/avif"
    srcset="hero-400.avif 400w,
            hero-800.avif 800w,
            hero-1200.avif 1200w"
    sizes="(max-width: 600px) 100vw, 50vw">
  
  <!-- WebP fallback -->
  <source 
    type="image/webp"
    srcset="hero-400.webp 400w,
            hero-800.webp 800w,
            hero-1200.webp 1200w"
    sizes="(max-width: 600px) 100vw, 50vw">
  
  <!-- JPEG fallback -->
  <img 
    src="hero-800.jpg"
    srcset="hero-400.jpg 400w,
            hero-800.jpg 800w,
            hero-1200.jpg 1200w"
    sizes="(max-width: 600px) 100vw, 50vw"
    width="1200" 
    height="600"
    alt="Hero image"
    loading="lazy"
    decoding="async">
</picture>
```

### LCP image priority
```html
<!-- Above-fold LCP image: eager loading, high priority -->
<img 
  src="hero.webp" 
  fetchpriority="high"
  loading="eager"
  decoding="sync"
  alt="Hero">

<!-- Below-fold images: lazy loading -->
<img 
  src="product.webp" 
  loading="lazy"
  decoding="async"
  alt="Product">
```

## Font optimization

### Loading strategy
```css
/* System font stack as fallback */
body {
  font-family: 'Custom Font', -apple-system, BlinkMacSystemFont, 
               'Segoe UI', Roboto, sans-serif;
}

/* Prevent invisible text */
@font-face {
  font-family: 'Custom Font';
  src: url('/fonts/custom.woff2') format('woff2');
  font-display: swap; /* or optional for non-critical */
  font-weight: 400;
  font-style: normal;
  unicode-range: U+0000-00FF; /* Subset to Latin */
}
```

### Preloading critical fonts
```html
<link rel="preload" href="/fonts/heading.woff2" as="font" type="font/woff2" crossorigin>
```

### Variable fonts
```css
/* One file instead of multiple weights */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/Inter-Variable.woff2') format('woff2-variations');
  font-weight: 100 900;
  font-display: swap;
}
```

## Caching strategy

### Cache-Control headers
```
# HTML (short or no cache)
Cache-Control: no-cache, must-revalidate

# Static assets with hash (immutable)
Cache-Control: public, max-age=31536000, immutable

# Static assets without hash
Cache-Control: public, max-age=86400, stale-while-revalidate=604800

# API responses
Cache-Control: private, max-age=0, must-revalidate
```

### Service worker caching
```javascript
// Cache-first for static assets
self.addEventListener('fetch', (event) => {
  if (event.request.destination === 'image' ||
      event.request.destination === 'style' ||
      event.request.destination === 'script') {
    event.respondWith(
      caches.match(event.request).then((cached) => {
        return cached || fetch(event.request).then((response) => {
          const clone = response.clone();
          caches.open('static-v1').then((cache) => cache.put(event.request, clone));
          return response;
        });
      })
    );
  }
});
```

## Runtime performance

### Avoid layout thrashing
```javascript
// ❌ Forces multiple reflows
elements.forEach(el => {
  const height = el.offsetHeight; // Read
  el.style.height = height + 10 + 'px'; // Write
});

// ✅ Batch reads, then batch writes
const heights = elements.map(el => el.offsetHeight); // All reads
elements.forEach((el, i) => {
  el.style.height = heights[i] + 10 + 'px'; // All writes
});
```

### Debounce expensive operations
```javascript
function debounce(fn, delay) {
  let timeout;
  return (...args) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => fn(...args), delay);
  };
}

// Debounce scroll/resize handlers
window.addEventListener('scroll', debounce(handleScroll, 100));
```

### Use requestAnimationFrame
```javascript
// ❌ May cause jank
setInterval(animate, 16);

// ✅ Synced with display refresh
function animate() {
  // Animation logic
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

### Virtualize long lists
```javascript
// For lists > 100 items, render only visible items
// Use libraries like react-window, vue-virtual-scroller, or native CSS:
.virtual-list {
  content-visibility: auto;
  contain-intrinsic-size: 0 50px; /* Estimated item height */
}
```

### Smooth navigations with View Transitions

The [View Transitions API](https://developer.chrome.com/docs/web-platform/view-transitions) lets the browser cross-fade (or custom-animate) between two DOM states using a single GPU-composited snapshot — no double-render, no layout thrash, and the snapshot doesn't count toward CLS.

**Same-document (SPA-style) — Baseline 2026:**
```javascript
// Wrap the DOM mutation that swaps the view
function navigate(newView) {
  if (!document.startViewTransition) return swapDOM(newView);
  document.startViewTransition(() => swapDOM(newView));
}
```

**Cross-document (MPA-style) — Chromium-stable, progressive enhancement elsewhere:**
```css
/* On both source and destination pages */
@view-transition { navigation: auto; }
```
That's the entire integration — same-origin navigations now fade automatically. To opt specific elements into shared-element transitions (e.g. a thumbnail expanding into a hero), give them a matching `view-transition-name`:
```css
.product-thumb[data-id="42"], .product-hero { view-transition-name: product-42; }
```

Pair this with Speculation Rules (above) for instant + animated navigations.

## Third-party scripts

### Load strategies
```javascript
// ❌ Blocks main thread
<script src="https://analytics.example.com/script.js"></script>

// ✅ Async loading
<script async src="https://analytics.example.com/script.js"></script>

// ✅ Delay until interaction
<script>
document.addEventListener('DOMContentLoaded', () => {
  const observer = new IntersectionObserver((entries) => {
    if (entries[0].isIntersecting) {
      const script = document.createElement('script');
      script.src = 'https://widget.example.com/embed.js';
      document.body.appendChild(script);
      observer.disconnect();
    }
  });
  observer.observe(document.querySelector('#widget-container'));
});
</script>
```

### Facade pattern
```html
<!-- Show static placeholder until interaction -->
<div class="youtube-facade" 
     data-video-id="abc123" 
     onclick="loadYouTube(this)">
  <img src="/thumbnails/abc123.jpg" alt="Video title">
  <button aria-label="Play video">▶</button>
</div>
```

## Measurement

Use [the measurement workflow](references/MEASUREMENT.md) whenever a URL is runnable. It defines Chrome DevTools MCP routing, CrUX and fallback sources, repeatable lab conditions, and a compact evidence format.

| Metric | Kind | Interpretation |
|--------|------|----------------|
| LCP, INP, CLS at p75 | Field | User-outcome Core Web Vitals; use for pass/fail prioritization |
| LCP, CLS in a trace | Lab | Reproducible diagnostic values for one navigation |
| TBT | Lab | Main-thread blocking diagnostic and a rough INP proxy, not field INP |
| FCP, Speed Index | Lab | Loading diagnostics, not Core Web Vitals |

Raw `PerformanceObserver` snippets are useful for the current browser session but are not real-user data by themselves. When the user wants production telemetry, read [the first-party RUM reference](references/RUM.md) and prefer `web-vitals` over a hand-rolled metric implementation.

## References

For Core Web Vitals specific optimizations, see [Core Web Vitals](../core-web-vitals/SKILL.md).
