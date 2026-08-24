# Cumulative Layout Shift (CLS)

Read this reference when field CLS is poor, a performance trace reports layout shifts, or source inspection finds content that changes geometry without reserved space.

CLS scores unexpected shift clusters across a page visit. A layout-shift score is the `impact fraction × distance fraction`. Use the shifted-node and initiator evidence: the element that moved may be the victim of content inserted above it.

## Reserve media space

**Bad:**

```html
<img src="photo.jpg" alt="Photo">
<iframe src="https://video.example/embed/123" title="Demo"></iframe>
```

**Good:**

```html
<img src="photo.jpg" alt="Photo" width="800" height="600">

<div class="video-frame">
  <iframe src="https://video.example/embed/123" title="Demo"></iframe>
</div>
```

```css
.video-frame {
  aspect-ratio: 16 / 9;
}

.video-frame iframe {
  height: 100%;
  width: 100%;
}
```

Reserve a realistic minimum for ads and embeds whose final size can vary. A placeholder that later collapses can also shift content.

## Handle dynamic content deliberately

Do not insert banners, validation summaries, consent UI, or notifications above visible content without reserving space. Prefer an overlay, insert outside the active viewport, or allocate a stable container before the content arrives.

**Bad:**

```javascript
main.prepend(notification);
```

**Good:**

```javascript
const slot = document.querySelector('[data-notification-slot]');
slot.replaceChildren(notification);
```

The corresponding slot must already have appropriate reserved dimensions. Verify that responsive content and localization do not overflow it.

## Stabilize fonts

Use a fallback with similar metrics and tune it with `size-adjust`, `ascent-override`, `descent-override`, and `line-gap-override` when trace evidence attributes shifts to font replacement.

```css
@font-face {
  font-family: "Brand Fallback";
  src: local("Arial");
  size-adjust: 102%;
  ascent-override: 92%;
  descent-override: 24%;
  line-gap-override: 0%;
}
```

Do not copy these values to another font pair; derive them from the actual font metrics and test representative text.

## Animate without layout

Prefer `transform` and `opacity` for visual motion. Animating `height`, `width`, `top`, or `left` can trigger layout, but replacing them mechanically is not enough: confirm the transformed element does not obscure content or change the intended hit area.

```css
.toast {
  inset-block-start: 1rem;
  inset-inline-end: 1rem;
  position: fixed;
  transform: translateY(-150%);
  transition: transform 200ms;
}

.toast.is-visible {
  transform: translateY(0);
}
```

## Inspect one browser session

This observer reports shifts seen during the current page session. It is not the distribution of real visits.

```javascript
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (!entry.hadRecentInput) {
      console.log('Layout shift', entry.value);
      entry.sources?.forEach(source => {
        console.log('Shifted node', source.node);
        console.log('Previous rect', source.previousRect);
        console.log('Current rect', source.currentRect);
      });
    }
  }
}).observe({ type: 'layout-shift', buffered: true });
```

## Verification checklist

- [ ] Images and responsive media reserve intrinsic space
- [ ] Ads, embeds, and async components have stable containers
- [ ] Banners and validation messages do not displace visible content unexpectedly
- [ ] Font swaps use measured fallback metrics when they cause shifts
- [ ] Animations avoid unnecessary layout work
- [ ] The relevant page state and viewport are exercised, not only the initial load
- [ ] Field improvement is claimed only after new RUM or CrUX visits

## Sources

* [Optimize CLS](https://web.dev/articles/optimize-cls)
* [Debug layout shifts](https://developer.chrome.com/docs/devtools/performance/insights#cls-culprits)
* [CSS font metric overrides](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/size-adjust)
