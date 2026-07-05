---
name: best-practices
description: Apply modern web development best practices for security, compatibility, and code quality. Use when asked to "apply best practices", "security audit", "modernize code", "code quality review", or "check for vulnerabilities".
license: MIT
metadata:
  author: web-quality-skills
  version: "1.0"
---

# Best practices

Modern web development standards based on Lighthouse best practices audits. Covers security, browser compatibility, and code quality patterns.

## Security

### HTTPS everywhere

**Enforce HTTPS:**
```html
<!-- ❌ Mixed content -->
<img src="http://example.com/image.jpg">
<script src="http://cdn.example.com/script.js"></script>

<!-- ✅ HTTPS only -->
<img src="https://example.com/image.jpg">
<script src="https://cdn.example.com/script.js"></script>
```

Avoid protocol-relative URLs (`//example.com/...`) — they're an HTTP-era pattern with no benefit on HTTPS-only sites and hide the actual scheme from reviewers.

**HSTS Header:**
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

### Content Security Policy (CSP)

```html
<!-- Basic CSP via meta tag -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' https://trusted-cdn.com; 
               style-src 'self' 'unsafe-inline';
               img-src 'self' data: https:;
               connect-src 'self' https://api.example.com;">

<!-- Better: HTTP header -->
```

**CSP Header (recommended):**
```
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' 'nonce-abc123' https://trusted.com;
  style-src 'self' 'nonce-abc123';
  img-src 'self' data: https:;
  connect-src 'self' https://api.example.com;
  frame-ancestors 'self';
  base-uri 'self';
  form-action 'self';
```

**Using nonces for inline scripts:**
```html
<script nonce="abc123">
  // This inline script is allowed
</script>
```

### Trusted Types (modern DOM-XSS defense)

A strict CSP blocks loading untrusted *script files*, but it doesn't stop a string from reaching `innerHTML`, `eval`, or other DOM-XSS sinks. Trusted Types — Baseline across all major browsers since early 2026 — closes that hole by making sinks reject raw strings and accept only typed objects produced by a named policy.

```
Content-Security-Policy: require-trusted-types-for 'script'; trusted-types default;
```

```javascript
// One central policy that does the sanitization
const escape = trustedTypes.createPolicy('default', {
  createHTML: (s) => DOMPurify.sanitize(s, { RETURN_TRUSTED_TYPE: true })
});

// ❌ This now throws TypeError under enforcement
element.innerHTML = userInput;

// ✅ Goes through the policy
element.innerHTML = escape.createHTML(userInput);
```

Roll out with `Content-Security-Policy-Report-Only` first to find every sink usage in your app, then flip to enforcement. Angular has built-in Trusted Types support; React 19+ produces TrustedHTML when Trusted Types are enforced; for everything else, [DOMPurify](https://github.com/cure53/DOMPurify) is the de-facto sanitizer.

### Subresource Integrity (SRI) for third-party scripts

Pin every `<script>` and `<link rel="stylesheet">` you load from a CDN you don't control. If the CDN is compromised — as happened to polyfill.io in 2024 — the browser refuses to execute a file whose hash doesn't match.

```html
<script src="https://cdn.example.com/lib@1.2.3/dist/lib.js"
        integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
        crossorigin="anonymous"></script>
```

`integrity` accepts space-separated hashes; include the next version's hash before rotating to avoid downtime. Generate with `openssl dgst -sha384 -binary file.js | openssl base64 -A`. SRI requires `crossorigin` and an `Access-Control-Allow-Origin` response header from the CDN.

### Security headers

```
# Prevent clickjacking — prefer CSP `frame-ancestors` (above); X-Frame-Options
# is the legacy fallback for older browsers.
X-Frame-Options: DENY

# Prevent MIME type sniffing
X-Content-Type-Options: nosniff

# Do NOT send X-XSS-Protection. The legacy browser XSS auditor was deprecated
# and removed (Chrome 78, Edge 17), and in some cases it introduced its own
# vulnerabilities. Use a strict CSP + Trusted Types (below) instead.

# Control referrer information
Referrer-Policy: strict-origin-when-cross-origin

# Permissions policy (formerly Feature-Policy)
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

### No vulnerable libraries

```bash
# Check for vulnerabilities
npm audit
yarn audit

# Auto-fix when possible
npm audit fix

# Check specific package
npm ls lodash
```

**Keep dependencies updated:**
```json
// package.json
{
  "scripts": {
    "audit": "npm audit --audit-level=moderate",
    "update": "npm update && npm audit fix"
  }
}
```

**Known vulnerable patterns to avoid:**
```javascript
// ❌ Recursive merges of untrusted input can pollute Object.prototype
//    via __proto__, constructor, or prototype keys.
_.merge(target, userInput);          // lodash <4.17.20
$.extend(true, {}, target, userInput); // jQuery deep extend
Object.assign(target, ...userInputs); // safe by itself (shallow), but unsafe
                                      // when target IS Object.prototype-derived
                                      // and userInput contains __proto__

// ✅ For untrusted bags, use a null-prototype object so __proto__ is just a key
const safe = Object.create(null);
Object.assign(safe, userInput); // shallow, no recursion → safe by construction

// ✅ For deep copies, structuredClone drops __proto__ and functions
const deepSafe = structuredClone(userInput);

// ✅ For deep merges, use a library that explicitly blocks dangerous keys
//    (e.g. lodash ≥4.17.21 _.mergeWith with a customizer, or deepmerge-ts).
```

### Input sanitization

```javascript
// ❌ XSS vulnerable
element.innerHTML = userInput;
document.write(userInput);

// ✅ Safe text content
element.textContent = userInput;

// ✅ If HTML needed, sanitize
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

### Secure cookies

```javascript
// ❌ Insecure cookie
document.cookie = "session=abc123";

// ✅ Secure cookie (server-side)
Set-Cookie: session=abc123; Secure; HttpOnly; SameSite=Strict; Path=/
```

---

## Browser compatibility

### Doctype declaration

```html
<!-- ❌ Missing or invalid doctype -->
<HTML>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">

<!-- ✅ HTML5 doctype -->
<!DOCTYPE html>
<html lang="en">
```

### Character encoding

```html
<!-- ❌ Missing or late charset -->
<html>
<head>
  <title>Page</title>
  <meta charset="UTF-8">
</head>

<!-- ✅ Charset as first element in head -->
<html>
<head>
  <meta charset="UTF-8">
  <title>Page</title>
</head>
```

### Viewport meta tag

```html
<!-- ❌ Missing viewport -->
<head>
  <title>Page</title>
</head>

<!-- ✅ Responsive viewport -->
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Page</title>
</head>
```

### Feature detection

```javascript
// ❌ Browser detection (brittle)
if (navigator.userAgent.includes('Chrome')) {
  // Chrome-specific code
}

// ✅ Feature detection
if ('IntersectionObserver' in window) {
  // Use IntersectionObserver
} else {
  // Fallback
}

// ✅ Using @supports in CSS
@supports (display: grid) {
  .container {
    display: grid;
  }
}

@supports not (display: grid) {
  .container {
    display: flex;
  }
}
```

### Polyfills (when needed)

Prefer **bundling polyfills at build time** (Babel/SWC + `core-js`, or `@vitejs/plugin-legacy`) targeted by your supported-browsers list. This eliminates the runtime check entirely and avoids shipping polyfill bytes to modern browsers.

If you must load a polyfill at runtime, append a script element — never use `document.write` (it blocks the parser and is broken in async/deferred contexts):

```html
<script>
  if (!('fetch' in window)) {
    const s = document.createElement('script');
    s.src = '/polyfills/fetch.js';
    s.defer = true;
    document.head.appendChild(s);
  }
</script>
```

**Never load polyfills from a third-party CDN you don't control.** The `polyfill.io` service was [compromised in mid-2024](https://sansec.io/research/polyfill-supply-chain-attack) in a supply-chain attack and used to serve malware to ~100k sites. Self-host, or use a vetted mirror (e.g. [Cloudflare's `cdnjs` polyfill build](https://blog.cloudflare.com/polyfill-io-now-available-on-cdnjs-reduce-your-supply-chain-risk/)) — and pin the version with [Subresource Integrity](#subresource-integrity-sri-for-third-party-scripts).

---

## Deprecated APIs

### Avoid these

```javascript
// ❌ document.write (blocks parsing)
document.write('<script src="..."></script>');

// ✅ Dynamic script loading
const script = document.createElement('script');
script.src = '...';
document.head.appendChild(script);

// ❌ Synchronous XHR (blocks main thread)
const xhr = new XMLHttpRequest();
xhr.open('GET', url, false); // false = synchronous

// ✅ Async fetch
const response = await fetch(url);

// ❌ Application Cache (deprecated)
<html manifest="cache.manifest">

// ✅ Service Workers
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js');
}
```

### Event listener passive

```javascript
// ❌ Non-passive touch/wheel (may block scrolling)
element.addEventListener('touchstart', handler);
element.addEventListener('wheel', handler);

// ✅ Passive listeners (allows smooth scrolling)
element.addEventListener('touchstart', handler, { passive: true });
element.addEventListener('wheel', handler, { passive: true });

// ✅ If you need preventDefault, be explicit
element.addEventListener('touchstart', handler, { passive: false });
```

---

## Console & errors

### No console errors

```javascript
// ❌ Errors in production
console.log('Debug info'); // Remove in production
throw new Error('Unhandled'); // Catch all errors

// ✅ Proper error handling
try {
  riskyOperation();
} catch (error) {
  // Log to error tracking service
  errorTracker.captureException(error);
  // Show user-friendly message
  showErrorMessage('Something went wrong. Please try again.');
}
```

### Error boundaries (React)

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, info) {
    errorTracker.captureException(error, { extra: info });
  }
  
  render() {
    if (this.state.hasError) {
      return <FallbackUI />;
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <App />
</ErrorBoundary>
```

### Global error handler

```javascript
// Catch unhandled errors
window.addEventListener('error', (event) => {
  errorTracker.captureException(event.error);
});

// Catch unhandled promise rejections
window.addEventListener('unhandledrejection', (event) => {
  errorTracker.captureException(event.reason);
});
```

---

## Source maps

### Production configuration

```javascript
// ❌ Source maps exposed in production
// webpack.config.js
module.exports = {
  devtool: 'source-map', // Exposes source code
};

// ✅ Hidden source maps (uploaded to error tracker)
module.exports = {
  devtool: 'hidden-source-map',
};

// ✅ Or no source maps in production
module.exports = {
  devtool: process.env.NODE_ENV === 'production' ? false : 'source-map',
};
```

**Strip `sourcesContent` from production maps** when uploading to your error tracker. By default, bundlers embed the full original source inside the `.map` file — anyone who obtains the map (including via a misconfigured upload step) gets your unminified code. Configure your bundler to omit `sourcesContent`, or use a Sentry/Bugsnag CLI flag that does so when uploading.

For Vite, prefer `sourcemap: 'hidden'` over `'true'` so the `//# sourceMappingURL=` comment isn't emitted into the bundle.

---

## Performance best practices

### Avoid blocking patterns

```javascript
// ❌ Blocking script
<script src="heavy-library.js"></script>

// ✅ Deferred script
<script defer src="heavy-library.js"></script>

// ❌ Blocking CSS import
@import url('other-styles.css');

// ✅ Link tags (parallel loading)
<link rel="stylesheet" href="styles.css">
<link rel="stylesheet" href="other-styles.css">
```

### Efficient event handlers

```javascript
// ❌ Handler on every element
items.forEach(item => {
  item.addEventListener('click', handleClick);
});

// ✅ Event delegation
container.addEventListener('click', (e) => {
  if (e.target.matches('.item')) {
    handleClick(e);
  }
});
```

### Memory management

```javascript
// ❌ Memory leak (never removed)
const handler = () => { /* ... */ };
window.addEventListener('resize', handler);

// ✅ Cleanup when done
const handler = () => { /* ... */ };
window.addEventListener('resize', handler);

// Later, when component unmounts:
window.removeEventListener('resize', handler);

// ✅ Using AbortController
const controller = new AbortController();
window.addEventListener('resize', handler, { signal: controller.signal });

// Cleanup:
controller.abort();
```

---

## Code quality

### Valid HTML

```html
<!-- ❌ Invalid HTML -->
<div id="header">
<div id="header"> <!-- Duplicate ID -->

<ul>
  <div>Item</div> <!-- Invalid child -->
</ul>

<a href="/"><button>Click</button></a> <!-- Invalid nesting -->

<!-- ✅ Valid HTML -->
<header id="site-header">
</header>

<ul>
  <li>Item</li>
</ul>

<a href="/" class="button">Click</a>
```

### Semantic HTML

```html
<!-- ❌ Non-semantic -->
<div class="header">
  <div class="nav">
    <div class="nav-item">Home</div>
  </div>
</div>
<div class="main">
  <div class="article">
    <div class="title">Headline</div>
  </div>
</div>

<!-- ✅ Semantic HTML5 -->
<header>
  <nav>
    <a href="/">Home</a>
  </nav>
</header>
<main>
  <article>
    <h1>Headline</h1>
  </article>
</main>
```

### Image aspect ratios

```html
<!-- ❌ Distorted images -->
<img src="photo.jpg" width="300" height="100">
<!-- If actual ratio is 4:3, this squishes the image -->

<!-- ✅ Preserve aspect ratio -->
<img src="photo.jpg" width="300" height="225">
<!-- Actual 4:3 dimensions -->

<!-- ✅ CSS object-fit for flexibility -->
<img src="photo.jpg" style="width: 300px; height: 200px; object-fit: cover;">
```

---

## Permissions & privacy

### Request permissions properly

```javascript
// ❌ Request on page load (bad UX, often denied)
navigator.geolocation.getCurrentPosition(success, error);

// ✅ Request in context, after user action
findNearbyButton.addEventListener('click', async () => {
  // Explain why you need it
  if (await showPermissionExplanation()) {
    navigator.geolocation.getCurrentPosition(success, error);
  }
});
```

### Permissions policy

```html
<!-- Restrict powerful features -->
<meta http-equiv="Permissions-Policy" 
      content="geolocation=(), camera=(), microphone=()">

<!-- Or allow for specific origins -->
<meta http-equiv="Permissions-Policy" 
      content="geolocation=(self 'https://maps.example.com')">
```

---

## Audit checklist

### Security (critical)
- [ ] HTTPS enabled, no mixed content
- [ ] No vulnerable dependencies (`npm audit`)
- [ ] CSP headers configured (with `frame-ancestors`, `base-uri`, `form-action`)
- [ ] `require-trusted-types-for 'script'` enforced (or report-only during rollout)
- [ ] Third-party `<script>`/`<link rel="stylesheet">` pinned with SRI hashes
- [ ] Security headers present (HSTS, X-Content-Type-Options, Referrer-Policy)
- [ ] No exposed source maps (and `sourcesContent` stripped from uploaded ones)

### Compatibility
- [ ] Valid HTML5 doctype
- [ ] Charset declared first in head
- [ ] Viewport meta tag present
- [ ] No deprecated APIs used
- [ ] Passive event listeners for scroll/touch

### Code quality
- [ ] No console errors
- [ ] Valid HTML (no duplicate IDs)
- [ ] Semantic HTML elements used
- [ ] Proper error handling
- [ ] Memory cleanup in components

### UX
- [ ] No intrusive interstitials
- [ ] Permission requests in context
- [ ] Clear error messages
- [ ] Appropriate image aspect ratios

## Tools

| Tool | Purpose |
|------|---------|
| `npm audit` | Dependency vulnerabilities |
| [SecurityHeaders.com](https://securityheaders.com) | Header analysis |
| [W3C Validator](https://validator.w3.org) | HTML validation |
| Lighthouse | Best practices audit |
| [Observatory](https://observatory.mozilla.org) | Security scan |

## References

- [MDN Web Security](https://developer.mozilla.org/en-US/docs/Web/Security)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Web Quality Audit](../web-quality-audit/SKILL.md)
