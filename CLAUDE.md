# CLAUDE.md — AI Assistant Guide for Linkumori Rules

## Repository Overview

This repository is the **community rules database** for the [Linkumori](https://github.com/Linkumori/Linkumori-Addon-MV3-Firefox) Firefox browser extension (MV3). It contains a single large JSON file (`custom-rules.json`) that defines site-specific URL cleaning rules. These rules strip tracking parameters, analytics tokens, affiliate codes, and other noise from URLs as users browse the web.

The extension merges these community rules with ClearURLs rules at runtime. This repository is solely the rule set — no extension source code lives here.

**License:** BSD-style — Copyright (c) 2026 Subham Mahesh
**Related repo:** https://github.com/Linkumori/Linkumori-Addon-MV3-Firefox

---

## Repository Structure

```
custom-rules.json   — All URL cleaning rules (the only code artifact)
README.MD           — High-level overview of the repository
contributing.md     — Detailed guide for adding/editing rules
License.md          — BSD-style license text
CLAUDE.md           — This file
```

There is no build system, no package manager, no test runner, and no compiled output. The entire project is a single JSON data file.

---

## The Rules File: `custom-rules.json`

The file is a flat JSON object (no wrapping `providers` key at the top level in the actual file — entries are direct top-level keys). Each key is a unique **provider name** and maps to a provider object.

### Provider Naming Convention

Provider names follow the pattern: `<domain-identifier>_<index>`

Examples:
- `amazon._1` — matches all Amazon TLDs
- `amazon.*_1` — wildcard TLD variant
- `www.amazon._1` — specific subdomain variant
- `google._1` — Google across all TLDs
- `instagram.com_1` — specific TLD

Rules for the same site are split into multiple entries (e.g., `amazon._1`, `amazon.*_1`, `www.amazon._1`) when different subdomains or URL patterns require different parameter sets.

### Provider Object Fields

| Field | Type | Required | Purpose |
|---|---|---|---|
| `urlPattern` | string (regex) | One of `urlPattern` or `domainPatterns` | Regex matched against full URL |
| `domainPatterns` | array of strings | One of `urlPattern` or `domainPatterns` | AdBlock-style domain patterns |
| `completeProvider` | boolean | No (defaults false) | If `true`, blocks all requests to this provider |
| `forceRedirection` | boolean | No | Forces redirect-following even without a redirection rule |
| `rules` | array of strings (regex) | No | Query parameter keys to strip unconditionally |
| `rawRules` | array of strings (regex) | No | Regex applied directly to the raw URL string (before query parsing) |
| `referralMarketing` | array of strings (regex) | No | Affiliate/referral parameters — user-toggleable |
| `exceptions` | array of strings (regex) | No | Full URL patterns that bypass this provider's rules |
| `domainExceptions` | array of strings | No | AdBlock-style domain patterns that bypass rules |
| `redirections` | array of strings (regex) | No | Patterns to unwrap redirect URLs (first capture group = real URL) |
| `domainRedirections` | array of strings | No | AdBlock-style domain patterns that trigger redirect unwrapping |
| `methods` | array of strings | No | HTTP methods to apply rules to (e.g. `"GET"`) |
| `resourceTypes` | array of strings | No | Browser resource types to apply rules to |

### `urlPattern` Regex Conventions

The dominant pattern used across entries:

```
^https?:\/\/(?:[a-z0-9-]+\.)*?<domain>\.<tld-or-wildcard>
```

- `\\.` escapes dots in JSON regex strings
- `(?:[a-z0-9-]+\\.)*?` handles arbitrary subdomains lazily
- TLD wildcards use `\\.` with no specific extension (e.g., `amazon\\.` matches all Amazon TLDs)
- Wildcard entries use `\\.\\*` literally (e.g., `amazon\\.\\*`)

### `domainPatterns` AdBlock Syntax

| Pattern | Matches |
|---|---|
| `\|\|example.com^` | `example.com` and all subdomains |
| `\|\|example.*^` | `example.com`, `example.co.uk`, etc. (any TLD, root + `www.` only) |
| `\|\|*.example.com^` | All subdomains of `example.com`, not root |
| `\|\|*.example.*^` | All subdomains of `example` across any TLD |
| `\|\|example.com/path` | `example.com` and subdomains at that path prefix |

### `rules` Matching Behavior

- Rules are anchored: `"ref"` matches only the parameter named exactly `ref`
- Regex is supported: `"utm_[a-z]+"` strips all `utm_*` parameters
- Complex patterns: `"^[a-z_]{1,20}=[a-zA-Z0-9._-]{80,}$"` strips long opaque token parameters

### `rawRules` Caution

`rawRules` run before query parsing and modify the raw URL string via regex replacement. Use sparingly — broken patterns corrupt URLs.

### `referralMarketing` vs `rules`

- `rules` — always stripped
- `referralMarketing` — stripped only when the user enables "Strip referral marketing" in the extension settings. Affiliate/referral parameters (e.g., `tag`, `ref`, `affiliate_id`) belong here, not in `rules`.

---

## Key Conventions

1. **Separate providers per domain variant** — Do not combine Amazon.com, Amazon.*, and gaming.amazon.com into one entry. Use separate keyed entries.

2. **Regex in rules are partial patterns anchored automatically** — No need to add `^` or `$` to simple parameter names. Only use regex features when matching multiple variants (e.g., `"utm_[a-z]+"`).

3. **Affiliate/referral parameters go in `referralMarketing`** — Never in `rules`, because users may want to keep them.

4. **Do not strip functional parameters** — Only strip parameters that have no effect on page content, search results, or core functionality.

5. **`resourceTypes` for scoped rules** — When a rule should only apply to XHR or background requests (not page loads), specify `"resourceTypes": ["xmlhttprequest"]` (see: `amazon._1`).

6. **JSON must be valid** — Validate with `jq . custom-rules.json` before committing.

---

## Development Workflow

### Adding a New Provider

1. Identify the domain and what tracking parameters it uses
2. Choose a provider key: `<site>_1` (or `_2`, `_3` for additional entries)
3. Write the `urlPattern` following the established regex convention
4. Determine which parameters go in `rules` vs `referralMarketing`
5. Add the entry to `custom-rules.json`
6. Validate: `jq . custom-rules.json`

### Branch Naming

For rule-only contributions, use the `rules/` prefix:

```bash
git checkout -b rules/site-name
git checkout -b rules/amazon-affiliate-params
git checkout -b rules/new-provider-example-com
```

### Pull Request Requirements

- State the site being targeted
- List what parameters are being stripped and why
- Provide a sample URL (with tracking params) demonstrating the issue
- One site or closely related group of sites per PR

### JSON Validation

```bash
jq . custom-rules.json
```

This is the only "test" available — the file must be valid JSON.

---

## Common Parameter Categories

When evaluating parameters to strip, these are common tracking parameter types:

| Category | Examples |
|---|---|
| UTM analytics | `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term` |
| Ad click IDs | `fbclid`, `gclid`, `msclkid`, `twclid`, `ttclid` |
| Session/visit IDs | `sessionId`, `visitId`, `sid` |
| Internal tracking | Site-specific opaque tokens, A/B test flags |
| Referral marketing | `ref`, `tag`, `affiliate`, `partner_id`, `ascsubtag` |

---

## What NOT to Do

- Do not strip parameters that affect page content (e.g., search query `q=`, product `id=`, page `page=`)
- Do not use `completeProvider: true` unless the entire domain is a tracking/analytics service with no legitimate user-facing content
- Do not use `rawRules` when a standard `rules` entry suffices
- Do not combine unrelated domains into one provider entry
- Do not commit invalid JSON

---

## Extension Integration

This file is consumed by the Linkumori Firefox extension. The extension merges `custom-rules.json` with the ClearURLs rule set at runtime. Changes to this repository take effect in the extension on its next rules update cycle.

Related extension repository: https://github.com/Linkumori/Linkumori-Addon-MV3-Firefox
