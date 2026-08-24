# Security reference

## HTTPS everywhere

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

## Content Security Policy (CSP)

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

## Trusted Types (modern DOM-XSS defense)

A strict CSP blocks loading untrusted *script files*, but it doesn't stop a string from reaching `innerHTML`, `eval`, or other DOM-XSS sinks. Trusted Types — Baseline across all major browsers since early 2026 — closes that hole by making sinks reject raw strings and accept only typed objects produced by a named policy.

```
Content-Security-Policy: require-trusted-types-for 'script'; trusted-types default;
```

```javascript
// One central policy that does the sanitization
const escape = trustedTypes.createPolicy('default', {
  createHTML: (s) => DOMPurify.sanitize(s, { RETURN_TRUSTED_TYPE: false })
});

// ❌ This now throws TypeError under enforcement
element.innerHTML = userInput;

// ✅ Goes through the policy
element.innerHTML = escape.createHTML(userInput);
```

Roll out with `Content-Security-Policy-Report-Only` first to find every sink usage, then flip to enforcement. Keep the sanitizer current.

Framework integration is not interchangeable:

* **Angular** documents built-in sanitization plus the `angular` Trusted Types policy required for enforcement; lazy loading, JIT, upgrades, and explicit sanitizer bypasses need additional named policies. Follow [Angular's maintained security guide](https://angular.dev/best-practices/security#enforcing-trusted-types).
* **React** does not sanitize `dangerouslySetInnerHTML`. Keep untrusted input out of it or pass content sanitized by an application-owned policy, and verify the deployed React version under report-only enforcement. Do not assume the framework creates `TrustedHTML`; see [React's security warning](https://react.dev/reference/react-dom/components/common#dangerously-setting-the-inner-html).

## Subresource Integrity (SRI) for third-party scripts

Pin every `<script>` and `<link rel="stylesheet">` you load from a CDN you don't control. If the CDN is compromised — as happened to polyfill.io in 2024 — the browser refuses to execute a file whose hash doesn't match.

```html
<script src="https://cdn.example.com/lib@1.2.3/dist/lib.js"
        integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
        crossorigin="anonymous"></script>
```

`integrity` accepts space-separated hashes; include the next version's hash before rotating to avoid downtime. Generate with `openssl dgst -sha384 -binary file.js | openssl base64 -A`. SRI requires `crossorigin` and an `Access-Control-Allow-Origin` response header from the CDN.

## Security headers

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

## No vulnerable libraries

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

## Input sanitization

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

## Secure cookies

```javascript
// ❌ Insecure cookie
document.cookie = "session=abc123";

// ✅ Secure cookie (server-side)
Set-Cookie: session=abc123; Secure; HttpOnly; SameSite=Strict; Path=/
```

---

## Sources

- [MDN web security](https://developer.mozilla.org/en-US/docs/Web/Security)
- [OWASP Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [DOMPurify](https://github.com/cure53/DOMPurify)
