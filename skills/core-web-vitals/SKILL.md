---
name: core-web-vitals
description: Optimize Core Web Vitals (LCP, INP, CLS) for better page experience and search ranking. Use when asked to "improve Core Web Vitals", "fix LCP", "reduce CLS", "optimize INP", "page experience optimization", or "fix layout shifts".
license: MIT
metadata:
  author: web-quality-skills
  version: "2.0"
---

# Core Web Vitals optimization

Targeted optimization for the three Core Web Vitals using field data to identify user impact and browser traces to diagnose causes.

## Measure before optimizing

When a runnable URL is available, read [the performance measurement workflow](../performance/references/MEASUREMENT.md). Prefer this sequence:

1. Check page-level CrUX p75 data, with a clearly labeled origin fallback when page data is unavailable.
2. Record a Chrome DevTools performance trace under stated conditions. Current Chrome DevTools MCP trace summaries can include CrUX alongside the observed lab metrics.
3. Analyze only the insights associated with the failing metric, then inspect the implicated code and resources.
4. Re-run equivalent lab measurements after the fix. Do not claim an immediate field improvement; CrUX and first-party RUM need new user visits.

If only source code is available, identify likely causes but do not claim that LCP, INP, or CLS is failing without runtime evidence.

## The three metrics

| Metric | Measures | Good | Needs work | Poor |
|--------|----------|------|------------|------|
| **LCP** | Loading | ≤ 2.5s | 2.5s – 4s | > 4s |
| **INP** | Interactivity | ≤ 200ms | 200ms – 500ms | > 500ms |
| **CLS** | Visual Stability | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |

Google measures at the **75th percentile** — 75% of page visits must meet "Good" thresholds.

---

## LCP: Largest Contentful Paint

LCP measures when the largest visible content element renders. Usually this is:
- Hero image or video
- Large text block
- Background image
- `<svg>` element

### Common LCP issues

**1. Slow server response (TTFB > 800ms)**
```
Fix: CDN, caching, optimized backend, edge rendering
```

**2. Render-blocking resources**
```html
<!-- ❌ Blocks rendering -->
<link rel="stylesheet" href="/all-styles.css">

<!-- ✅ Critical CSS inlined, rest deferred -->
<style>/* Critical above-fold CSS */</style>
<link rel="preload" href="/styles.css" as="style" 
      onload="this.onload=null;this.rel='stylesheet'">
```

**3. Slow resource load times**
```html
<!-- ❌ LCP image is discovered only after a stylesheet loads -->
<div class="hero"></div>

<!-- ✅ Discoverable in initial HTML and prioritized -->
<link rel="preload" href="/hero.webp" as="image" fetchpriority="high">
<img src="/hero.webp" alt="Hero" fetchpriority="high">
```

Prefer a discoverable `<img>` with `fetchpriority="high"`. Add the preload only when the trace shows that the resource would otherwise be discovered late; duplicate or speculative preloads can compete for bandwidth.

**4. Client-side rendering delays**
```javascript
// ❌ Content loads after JavaScript
useEffect(() => {
  fetch('/api/hero-text').then(r => r.json()).then(setHeroText);
}, []);

// ✅ Server-side or static rendering
// Use SSR, SSG, or streaming to send HTML with content
export async function getServerSideProps() {
  const heroText = await fetchHeroText();
  return { props: { heroText } };
}
```

**5. Make navigations instant with the Speculation Rules API**

For sites with predictable same-origin journeys, prerendering a likely next page can make a successful subsequent navigation much faster. Treat this as a measured navigation optimization, not a substitute for fixing the current page's LCP.

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

`eagerness` settings range from stronger intent signals (`conservative`, `moderate`) to earlier speculation (`eager`, `immediate`). Start conservatively and measure prediction hit rate, transferred bytes, server load, and navigation improvement before expanding the rules.

Caveats:
- **Bandwidth/CPU cost.** Each prerender is roughly a full page load. Scope `where` carefully (`href_matches` patterns, exclude logout/checkout) and avoid `immediate` outside small sites.
- **Side effects fire early.** Analytics, ads, and any code that runs on load will fire when the prerender starts, not when the user navigates. Gate side effects on the [`prerenderingchange` event](https://developer.chrome.com/docs/web-platform/prerender-pages#detect_when_a_page_is_prerendered_or_used_for_a_full_navigation) or `document.prerendering`.
- **Chromium-only.** Safari and Firefox ignore the script — it's a progressive enhancement, never a regression.

### LCP optimization checklist

```markdown
- [ ] TTFB < 800ms (use CDN, edge caching)
- [ ] LCP resource is discoverable in initial HTML and prioritized; preload only if the trace shows late discovery
- [ ] LCP image optimized (WebP/AVIF, correct size)
- [ ] Critical CSS inlined (< 14KB)
- [ ] No render-blocking JavaScript in <head>
- [ ] Fonts don't block text rendering (font-display: swap)
- [ ] LCP element in initial HTML (not JS-rendered)
- [ ] Speculation Rules added for likely-next navigations (moderate eagerness)
```

### LCP element identification

This snippet diagnoses the current page session. It is not field data.

```javascript
// Find your LCP element
new PerformanceObserver((list) => {
  const entries = list.getEntries();
  const lastEntry = entries[entries.length - 1];
  console.log('LCP element:', lastEntry.element);
  console.log('LCP time:', lastEntry.startTime);
}).observe({ type: 'largest-contentful-paint', buffered: true });
```

---

## INP: Interaction to Next Paint

INP measures responsiveness across ALL interactions (clicks, taps, key presses) during a page visit. It reports the worst interaction (at 98th percentile for high-traffic pages).

### INP breakdown

Total INP = **Input Delay** + **Processing Time** + **Presentation Delay**

| Phase | Target | Optimization |
|-------|--------|--------------|
| Input Delay | < 50ms | Reduce main thread blocking |
| Processing | < 100ms | Optimize event handlers |
| Presentation | < 50ms | Minimize rendering work |

### Common INP issues

**1. Long tasks blocking main thread**
```javascript
// ❌ Long synchronous task
function processLargeArray(items) {
  items.forEach(item => expensiveOperation(item));
}

// ✅ Break into chunks and yield to the scheduler. scheduler.yield() is the
//    recommended modern API — its continuation is queued at a boosted
//    priority so the rest of your work resumes ahead of unrelated tasks,
//    while still letting the browser handle pending input first.
async function processLargeArray(items) {
  const CHUNK_SIZE = 100;
  for (let i = 0; i < items.length; i += CHUNK_SIZE) {
    items.slice(i, i + CHUNK_SIZE).forEach(expensiveOperation);

    if ('scheduler' in window && 'yield' in scheduler) {
      await scheduler.yield();
    } else {
      // Fallback for browsers without scheduler.yield (Safari, older Firefox).
      // setTimeout(0) yields but loses priority — your continuation may run
      // after unrelated tasks the browser picked up in between.
      await new Promise(r => setTimeout(r, 0));
    }
  }
}
```

**2. Heavy event handlers**
```javascript
// ❌ All work in handler
button.addEventListener('click', () => {
  // Heavy computation
  const result = calculateComplexThing();
  // DOM updates
  updateUI(result);
  // Analytics
  trackEvent('click');
});

// ✅ Prioritize visual feedback, then yield before doing the heavy work
button.addEventListener('click', async () => {
  // 1. Immediate visual feedback (cheap DOM update)
  button.classList.add('loading');

  // 2. Yield so the browser can paint the loading state before we block
  if ('scheduler' in window && 'yield' in scheduler) {
    await scheduler.yield();
  }

  // 3. Now do the heavy work — the user already saw the click register
  const result = calculateComplexThing();
  updateUI(result);

  // 4. Lowest-priority work last, when the main thread is idle
  if ('requestIdleCallback' in window) {
    requestIdleCallback(() => trackEvent('click'));
  } else {
    setTimeout(() => trackEvent('click'), 0);
  }
});
```

**3. Third-party scripts**
```javascript
// ❌ Eagerly loaded, blocks interactions
<script src="https://heavy-widget.com/widget.js"></script>

// ✅ Lazy loaded on interaction or visibility
const loadWidget = () => {
  import('https://heavy-widget.com/widget.js')
    .then(widget => widget.init());
};
button.addEventListener('click', loadWidget, { once: true });
```

**4. Excessive re-renders (React/Vue)**
```javascript
// ❌ Re-renders entire tree
function App() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <Counter count={count} />
      <ExpensiveComponent /> {/* Re-renders on every count change */}
    </div>
  );
}

// ✅ Memoized expensive components
const MemoizedExpensive = React.memo(ExpensiveComponent);

function App() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <Counter count={count} />
      <MemoizedExpensive />
    </div>
  );
}
```

### INP optimization checklist

```markdown
- [ ] No tasks > 50ms on main thread
- [ ] Event handlers complete quickly (< 100ms)
- [ ] Visual feedback provided immediately
- [ ] Heavy work deferred with requestIdleCallback
- [ ] Third-party scripts don't block interactions
- [ ] Debounced input handlers where appropriate
- [ ] Web Workers for CPU-intensive operations
```

### INP debugging

This snippet diagnoses interactions performed in the current page session. It is not a substitute for field INP.

```javascript
// Identify slow interactions. durationThreshold: 40 matches what the
// web-vitals library uses — 16 (one frame) fires on nearly every interaction
// and drowns the console; 40 surfaces interactions that are starting to feel
// sluggish without spamming.
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 200) {
      console.warn('Slow interaction:', {
        type: entry.name,
        duration: entry.duration,
        processingStart: entry.processingStart,
        processingEnd: entry.processingEnd,
        target: entry.target
      });
    }
  }
}).observe({ type: 'event', buffered: true, durationThreshold: 40 });
```

For field debugging across real users, prefer the `web-vitals/attribution` build of the [web-vitals library](https://github.com/GoogleChrome/web-vitals) — `onINP()` from that build attaches a `LoAF` (Long Animation Frame) breakdown identifying the longest script and the input/processing/presentation phase that ate the budget.

---

## CLS: Cumulative Layout Shift

CLS measures unexpected layout shifts. A shift occurs when a visible element changes position between frames without user interaction.

**CLS Formula:** `impact fraction × distance fraction`

### Common CLS causes

**1. Images without dimensions**
```html
<!-- ❌ Causes layout shift when loaded -->
<img src="photo.jpg" alt="Photo">

<!-- ✅ Space reserved -->
<img src="photo.jpg" alt="Photo" width="800" height="600">

<!-- ✅ Or use aspect-ratio -->
<img src="photo.jpg" alt="Photo" style="aspect-ratio: 4/3; width: 100%;">
```

**2. Ads, embeds, and iframes**
```html
<!-- ❌ Unknown size until loaded -->
<iframe src="https://ad-network.com/ad"></iframe>

<!-- ✅ Reserve space with min-height -->
<div style="min-height: 250px;">
  <iframe src="https://ad-network.com/ad" height="250"></iframe>
</div>

<!-- ✅ Or use aspect-ratio container -->
<div style="aspect-ratio: 16/9;">
  <iframe src="https://youtube.com/embed/..." 
          style="width: 100%; height: 100%;"></iframe>
</div>
```

**3. Dynamically injected content**
```javascript
// ❌ Inserts content above viewport
notifications.prepend(newNotification);

// ✅ Insert below viewport or use transform
const insertBelow = viewport.bottom < newNotification.top;
if (insertBelow) {
  notifications.prepend(newNotification);
} else {
  // Animate in without shifting
  newNotification.style.transform = 'translateY(-100%)';
  notifications.prepend(newNotification);
  requestAnimationFrame(() => {
    newNotification.style.transform = '';
  });
}
```

**4. Web fonts causing FOUT**
```css
/* ❌ Font swap shifts text */
@font-face {
  font-family: 'Custom';
  src: url('custom.woff2') format('woff2');
}

/* ✅ Optional font (no shift if slow) */
@font-face {
  font-family: 'Custom';
  src: url('custom.woff2') format('woff2');
  font-display: optional;
}

/* ✅ Or match fallback metrics */
@font-face {
  font-family: 'Custom';
  src: url('custom.woff2') format('woff2');
  font-display: swap;
  size-adjust: 105%; /* Match fallback size */
  ascent-override: 95%;
  descent-override: 20%;
}
```

**5. Animations triggering layout**
```css
/* ❌ Animates layout properties */
.animate {
  transition: height 0.3s, width 0.3s;
}

/* ✅ Use transform instead */
.animate {
  transition: transform 0.3s;
}
.animate.expanded {
  transform: scale(1.2);
}
```

### CLS optimization checklist

```markdown
- [ ] All images have width/height or aspect-ratio
- [ ] All videos/embeds have reserved space
- [ ] Ads have min-height containers
- [ ] Fonts use font-display: optional or matched metrics
- [ ] Dynamic content inserted below viewport
- [ ] Animations use transform/opacity only
- [ ] No content injected above existing content
```

### CLS debugging

This snippet diagnoses shifts observed in the current page session. It does not represent the distribution of real visits.

```javascript
// Track layout shifts
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (!entry.hadRecentInput) {
      console.log('Layout shift:', entry.value);
      entry.sources?.forEach(source => {
        console.log('  Shifted element:', source.node);
        console.log('  Previous rect:', source.previousRect);
        console.log('  Current rect:', source.currentRect);
      });
    }
  }
}).observe({ type: 'layout-shift', buffered: true });
```

---

## Measurement sources

| Source | Use |
|--------|-----|
| Chrome DevTools MCP `performance_start_trace` | Observe one load or interaction and diagnose Performance Insights; use the included CrUX context when available |
| CrUX or Search Console | Prioritize aggregated real-user outcomes at p75 |
| Lighthouse CLI or PageSpeed Insights | Controlled lab fallback when DevTools tools are unavailable |
| First-party RUM | Segment current production experience by route, device, release, and attribution |
| Raw `PerformanceObserver` | Inspect one page session during debugging |

Do not use `lighthouse_audit` for performance when working through Chrome DevTools MCP; that tool intentionally covers non-performance Lighthouse categories. Do not compare a single lab value directly with a field p75 as if they were equivalent samples.

When adding or reviewing production collection, read [the first-party RUM reference](../performance/references/RUM.md). Prefer the `web-vitals` library because raw browser APIs do not by themselves implement every Core Web Vital's lifecycle and reporting rules.

---

## Framework quick fixes

### Next.js
```jsx
// LCP: Use next/image with priority
import Image from 'next/image';
<Image src="/hero.jpg" priority fill alt="Hero" />

// INP: Use dynamic imports
const HeavyComponent = dynamic(() => import('./Heavy'), { ssr: false });

// CLS: Image component handles dimensions automatically
```

### React
```jsx
// LCP: Preload in head
<link rel="preload" href="/hero.jpg" as="image" fetchpriority="high" />

// INP: Memoize and useTransition
const [isPending, startTransition] = useTransition();
startTransition(() => setExpensiveState(newValue));

// CLS: Always specify dimensions in img tags
```

### Vue/Nuxt
```vue
<!-- LCP: Use nuxt/image with preload -->
<NuxtImg src="/hero.jpg" preload loading="eager" />

<!-- INP: Use async components -->
<component :is="() => import('./Heavy.vue')" />

<!-- CLS: Use aspect-ratio CSS -->
<img :style="{ aspectRatio: '16/9' }" />
```

## References

- [Detailed LCP optimization](references/LCP.md) — read when an LCP trace points to discovery, loading, or render delay
- [web.dev LCP](https://web.dev/articles/lcp)
- [web.dev INP](https://web.dev/articles/inp)
- [web.dev CLS](https://web.dev/articles/cls)
- [Performance skill](../performance/SKILL.md)
