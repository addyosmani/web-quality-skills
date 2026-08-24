# First-party real-user monitoring

Read this only when the user wants to add, review, or improve production RUM.

## Before changing the site

Telemetry changes affect production data collection and privacy. Reuse an existing analytics or RUM pipeline when possible. If adding a new endpoint, vendor, cookie, or consent behavior was not requested, propose the change and get authorization before implementing it.

Prefer the `web-vitals` library over hand-written `PerformanceObserver` code. It follows the metric lifecycle and browser edge cases used by Google's tooling. Switch to the attribution build only when the extra diagnostic context will be reviewed and deliberately allowlisted.

## Minimal collection pattern

```javascript
import {onCLS, onINP, onLCP} from 'web-vitals';

function sendToRum({name, value, rating, id, navigationType}) {
  const body = JSON.stringify({
    name,
    value,
    rating,
    id,
    navigationType,
    path: location.pathname,
    release: window.APP_RELEASE
  });

  if (!navigator.sendBeacon?.('/rum', body)) {
    fetch('/rum', {method: 'POST', body, keepalive: true});
  }
}

onCLS(sendToRum);
onINP(sendToRum);
onLCP(sendToRum);
```

Adapt the payload to the existing backend. Do not include query strings, user-entered text, full DOM fragments, or other personal data. The `web-vitals/attribution` build can add element or script details; review them and send only explicit, low-cardinality fields that fit the site's privacy model.

## Collection rules

* Record a stable release or experiment identifier so regressions can be attributed to a change.
* Group by route template rather than creating a high-cardinality bucket for every URL.
* Retain device/form factor, navigation type, and coarse connection context when the privacy model allows it.
* Sample deliberately and record the sampling rate. Do not compare cohorts collected with different sampling rules as if they were equal.
* Let the library report final metric values. `reportAllChanges` is useful for local debugging but usually creates noisy production telemetry.
* Handle consent and regional privacy requirements through the site's existing policy.

## Aggregation and reporting

For each route or product journey, report:

* p75 for LCP, INP, and CLS
* percentage of visits in good, needs-improvement, and poor buckets
* sample count and time window
* important segments such as form factor, release, and navigation type

Do not use an average as the pass/fail signal. Assess each Core Web Vital at p75; all three p75 values must meet their good thresholds for the route or origin to pass the combined assessment.

CrUX and first-party RUM can disagree because they cover different users, browsers, routes, sampling rules, and time windows. Document those differences before treating either source as wrong.

## Sources

* [web-vitals library](https://github.com/GoogleChrome/web-vitals)
* [Best practices for measuring Web Vitals in the field](https://web.dev/articles/vitals-field-measurement-best-practices)
* [Find slow interactions in the field](https://web.dev/articles/find-slow-interactions-in-the-field)
