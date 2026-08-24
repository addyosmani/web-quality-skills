# Interaction to Next Paint (INP)

Read this reference when field INP is poor, a trace identifies a slow interaction, or source inspection finds interaction work that needs runtime verification.

## Diagnose the interaction phases

INP spans three phases. Do not optimize the event handler until the trace shows which phase dominates.

| Phase | Evidence to inspect | Typical fixes |
|-------|---------------------|---------------|
| Input delay | Long tasks already occupying the main thread before the event callback starts | Reduce startup work, split long tasks, delay third parties |
| Processing time | Event callbacks and synchronous work attached to the interaction | Remove unnecessary work, simplify handlers, use workers for CPU-heavy computation |
| Presentation delay | Style, layout, paint, or later main-thread work before the next frame | Reduce DOM scope, rendering cost, and layout invalidation |

## Yield long work

**Bad:**

```javascript
function processLargeArray(items) {
  items.forEach(item => expensiveOperation(item));
}
```

**Good:**

```javascript
async function processLargeArray(items) {
  const chunkSize = 100;

  for (let i = 0; i < items.length; i += chunkSize) {
    items.slice(i, i + chunkSize).forEach(expensiveOperation);

    if ('scheduler' in window && 'yield' in scheduler) {
      await scheduler.yield();
    } else {
      await new Promise(resolve => setTimeout(resolve, 0));
    }
  }
}
```

Choose chunk boundaries from trace evidence. A fixed item count does not guarantee an acceptable task duration on representative devices.

## Prioritize visible feedback

**Bad:**

```javascript
button.addEventListener('click', () => {
  const result = calculateComplexThing();
  updateUI(result);
  trackEvent('click');
});
```

**Good:**

```javascript
button.addEventListener('click', async () => {
  button.classList.add('loading');

  if ('scheduler' in window && 'yield' in scheduler) {
    await scheduler.yield();
  }

  const result = calculateComplexThing();
  updateUI(result);

  if ('requestIdleCallback' in window) {
    requestIdleCallback(() => trackEvent('click'));
  } else {
    setTimeout(() => trackEvent('click'), 0);
  }
});
```

Yielding helps only when the UI update can paint before the remaining work. Confirm the frame in the trace.

## Check common causes

* **Third-party code.** Attribute long tasks to their script URLs. Delay nonessential widgets until interaction or visibility, but avoid making the first user interaction pay the full initialization cost without feedback.
* **Framework rendering.** Profile the affected state transition. Memoization is useful only when it removes measured repeated work; do not apply it indiscriminately.
* **Large DOM updates.** Reduce the number of invalidated nodes and avoid forced synchronous layout caused by interleaved reads and writes.
* **CPU-heavy computation.** Move suitable work to a Web Worker and measure serialization overhead.

## Inspect one browser session

This observer reports interactions seen in the current page session. It is not field INP.

```javascript
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 200) {
      console.warn('Slow interaction', {
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

For production attribution, prefer the `web-vitals/attribution` build. Its `onINP()` attribution can identify the interaction target, event type, and Long Animation Frame or script evidence available for real visits.

## Verification checklist

- [ ] Reproduce the important interaction on a representative device or CPU profile
- [ ] Identify the dominant input, processing, or presentation phase
- [ ] Confirm which first- or third-party task owns the delay
- [ ] Provide visible feedback before deferred work where appropriate
- [ ] Re-run the same interaction and conditions after the fix
- [ ] Wait for new first-party RUM or CrUX visits before claiming field improvement

## Sources

* [Optimize INP](https://web.dev/articles/optimize-inp)
* [`scheduler.yield()`](https://web.dev/articles/optimize-long-tasks#scheduler-yield)
* [web-vitals attribution](https://github.com/GoogleChrome/web-vitals#attribution-build)
