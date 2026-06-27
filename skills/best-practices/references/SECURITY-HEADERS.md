# Security headers & hardening reference

Full configuration for the security controls summarized in the
[best-practices skill](../SKILL.md#security-headers--hardening): Content Security
Policy, Trusted Types, Subresource Integrity, the standard security-header set, and
known-vulnerable dependency patterns to avoid.

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
  createHTML: (s) => DOMPurify.sanitize(s, { RETURN_TRUSTED_TYPE: true })
});

// ❌ This now throws TypeError under enforcement
element.innerHTML = userInput;

// ✅ Goes through the policy
element.innerHTML = escape.createHTML(userInput);
```

Roll out with `Content-Security-Policy-Report-Only` first to find every sink usage in your app, then flip to enforcement. Angular has built-in Trusted Types support; React 19+ produces TrustedHTML when Trusted Types are enforced; for everything else, [DOMPurify](https://github.com/cure53/DOMPurify) is the de-facto sanitizer.

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
