# Contributing to Linkumori (Clean URLs) RULES



 Adding or Updating URL Rules

URL cleaning rules tell Linkumori which tracking parameters to strip, which domains to apply them to, and how to handle redirects. All custom rules live in `custom-rules.json` at the root of this repository.




### Rule File Structure


```json
{
  "providers": {
    "providerName": {
      ...provider fields...
    },
    "anotherProvider": {
      ...provider fields...
    }
  }
}
```

Each key inside `providers` is a unique name for that provider (website or service). Provider names are arbitrary but should be lowercase and descriptive.

---

### All Provider Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `urlPattern` | string (regex) | One of `urlPattern` or `domainPatterns` required | Regex matched against the full URL to identify this provider |
| `domainPatterns` | array of strings | One of `urlPattern` or `domainPatterns` required | AdBlock-style domain patterns (see Domain Patterns below) |
| `completeProvider` | boolean | Yes | If `true`, blocks all requests to this provider entirely (domain blocking) |
| `forceRedirection` | boolean | No | If `true`, forces following redirects even without a matching redirection rule |
| `rules` | array of strings (regex) | No | Query parameter names to strip from the URL |
| `rawRules` | array of strings (regex) | No | Regex patterns applied directly to the raw URL string before parameter parsing |
| `referralMarketing` | array of strings (regex) | No | Affiliate/referral parameters — stripped separately and can be toggled by the user |
| `exceptions` | array of strings (regex) | No | Full URL regex patterns — matching URLs are skipped even if they match `urlPattern` |
| `domainExceptions` | array of strings | No | AdBlock-style domain patterns — matching domains are skipped |
| `redirections` | array of strings (regex) | No | Regex patterns to unwrap redirect URLs; must capture the real destination URL |
| `domainRedirections` | array of strings | No | AdBlock-style domain patterns that trigger redirect unwrapping |
| `methods` | array of strings | No | HTTP methods to apply rules to (e.g. `"GET"`, `"POST"`). If omitted, applies to all |
| `resourceTypes` | array of strings | No | Browser resource types to apply rules to (e.g. `"main_frame"`, `"sub_frame"`, `"xmlhttprequest"`) |

---

### Choosing: `urlPattern` vs `domainPatterns`

The provider requires **one** of these two fields to identify which URLs it applies to:

**`urlPattern`** — a standard JavaScript regex matched against the full URL string:
```json
"urlPattern": "^https?://([a-z0-9-]+\\.)?example\\.com/"
```

**`domainPatterns`** — an array of AdBlock-style domain patterns (simpler and preferred for most cases):
```json
"domainPatterns": ["||example.com^", "||example.co.uk^"]
```

---

### Domain Patterns Syntax

Domain patterns use AdBlock-style notation. The following formats are supported:

| Pattern | Matches |
|---------|---------|
| `\|\|example.com^` | `example.com` and all subdomains (e.g. `www.example.com`, `sub.example.com`) |
| `\|\|example.*^` | `example.com`, `example.co.uk`, `example.de`, etc. — any TLD. Only matches root domain and `www.` prefix |
| `\|\|*.example.com^` | All subdomains of `example.com` but not `example.com` itself |
| `\|\|*.example.*^` | All subdomains of `example` across any TLD |
| `\|\|example.com/path` | `example.com` and subdomains, but only on URLs starting with `/path` |

Examples:

```json
"domainPatterns": [
  "||example.com^",
  "||example.*^",
  "||cdn.example.com^",
  "||example.com/redirect"
]
```

---

### `rules` — Stripping Query Parameters

Each rule is a regex pattern matched against **query parameter keys** (not values). Parameters whose keys match are removed from the URL.

```json
"rules": [
  "utm_source",
  "utm_medium",
  "utm_campaign",
  "utm_term",
  "utm_content",
  "utm_[a-z]+",
  "fbclid",
  "gclid",
  "mc_eid",
  "tracking_id"
]
```

Rules use anchored matching (`^rule$`), so `"ref"` will only match a parameter named exactly `ref`, not `referral`. To match both, use `"ref(erral)?"`.

---

### `rawRules` — Raw URL String Replacement

Raw rules are regex patterns applied directly to the entire URL string via `String.replace()` **before** query parameters are parsed. Use these for tracking tokens embedded in the URL path rather than as query parameters.

```json
"rawRules": [
  "/ref=[^&]*",
  ";jsessionid=[^?]*"
]
```

> Use `rawRules` sparingly — incorrect patterns can corrupt the URL.

---

### `referralMarketing` — Affiliate Parameters

Same syntax as `rules`, but these are treated as a separate category. Users can choose to keep or strip affiliate/referral parameters independently of standard tracking parameters.

```json
"referralMarketing": [
  "ref",
  "tag",
  "affiliate_id",
  "partner"
]
```

---

### `exceptions` — Skip Specific URLs

Regex patterns matched against the full URL. If a URL matches an exception, the provider's rules are not applied to it even if the URL matches `urlPattern` or `domainPatterns`.

```json
"exceptions": [
  "^https?://example\\.com/checkout",
  "^https?://api\\.example\\.com/"
]
```

---

### `domainExceptions` — Skip Specific Domains

AdBlock-style domain patterns (same syntax as `domainPatterns`) for domains that should be excluded from this provider's rules.

```json
"domainExceptions": [
  "||safe.example.com^"
]
```

---

### `redirections` — Unwrap Redirect URLs

Regex patterns matched against the full URL. The **first capture group** must capture the real destination URL. Linkumori will navigate to the captured URL instead.

```json
"redirections": [
  "^https?://example\\.com/redirect\\?url=([^&]*)",
  "^https?://out\\.example\\.com/\\?link=(.*)"
]
```

The captured value is automatically decoded before navigation.

---

### `domainRedirections`

AdBlock-style domain patterns that flag a domain as a redirect wrapper, triggering redirect unwrapping logic.

```json
"domainRedirections": [
  "||out.example.com^"
]
```

---

### `methods` — Limit by HTTP Method

By default rules apply to all HTTP methods. Use `methods` to restrict to specific ones:

```json
"methods": ["GET"]
```

---

### `resourceTypes` — Limit by Resource Type

By default rules apply to all resource types. Use `resourceTypes` to restrict to specific browser resource types:

```json
"resourceTypes": ["main_frame", "sub_frame"]
```

Common values: `main_frame`, `sub_frame`, `stylesheet`, `script`, `image`, `font`, `object`, `xmlhttprequest`, `ping`, `media`, `websocket`, `other`.

---

### Complete Example

```json
"example-shop_1": {
  "urlPattern": "^https?:\\/\\/(?:[a-z0-9-]+\\.)*?example\\.com",
  "completeProvider": false,
  "forceRedirection": false,
  "rules": [
    "utm_[a-z]+",
    "fbclid",
    "gclid",
    "tracking_id",
    "session_id"
  ],
  "rawRules": [
    ";jsessionid=[^?#]*"
  ],
  "referralMarketing": [
    "ref",
    "tag",
    "affiliate"
  ],
  "exceptions": [
    "^https?://api\\.example\\.com/"
  ],
  "domainExceptions": [
    "||payments.example.com^"
  ],
  "redirections": [
    "^https?://([a-z0-9-]+\\.)?example\\.com/out\\?url=([^&]*)"
  ],
  "methods": ["GET"],
  "resourceTypes": ["main_frame"]
}
```

---

### Branch Naming for Rule Changes

Use the `rules/` prefix when branching for rule-only contributions:

```bash
git checkout -b rules/amazon-affiliate-params
git checkout -b rules/new-provider-example-com
```

---


---

## Submitting a Pull Request

1. Fork this repository and create a branch: `git checkout -b rules/site-name`
2. Add or update your entry in `custom-rules.json`
3. Validate the JSON: `jq . custom-rules.json`
4. Open a pull request with:
   - The site being targeted
   - What parameters are being stripped and why
   - A sample URL (with tracking params) that demonstrates the issue

**Guidelines:**
- Only strip parameters that serve no functional purpose for the user
- Put affiliate/referral parameters in `referralMarketing`, not `rules`
- Do not strip parameters that affect page content, search results, or functionality
- One site or closely related group of sites per pull request

---

## License

[BSD-style](License.md) — Copyright (c) 2026 Subham Mahesh

---

*Thank you for helping make Linkumori better for everyone.*
