# Linkumori Rules

Custom URL cleaning rules for the [Linkumori](https://github.com/Linkumori/Linkumori_Addon_MV3-Firefox_RULES.git) Firefox extension (Manifest V3).

This repository contains `custom-rules.json` — a community-maintained list of site-specific rules that strip tracking parameters, analytics tokens, and other noise from URLs.

## What This Does

Each rule entry targets a specific domain and defines:

- **`urlPattern`** — regex pattern matching the site's URLs
- **`rules`** — list of query parameters (or regex patterns) to remove
- **`referralMarketing`** — parameters kept only when referral marketing is allowed in the extension settings
- **`resourceTypes`** — (optional) limit rule application to specific request types (e.g. `xmlhttprequest`)

**Example entry:**

```json
"amazon._1": {
  "urlPattern": "^https?:\\/\\/(?:[a-z0-9-]+\\.)*?amazon\\.",
  "rules": ["th", "linkCode", "qid", "crid"],
  "referralMarketing": ["ascsubtag", "tag"],
  "resourceTypes": ["xmlhttprequest"]
}
```

## File Structure

```
custom-rules.json   — All site-specific URL cleaning rules
License.md          — BSD-style license
contributing.md     — How to contribute new rules
```

## Usage

This file is consumed directly by the Linkumori Firefox extension. If you maintain your own fork of the extension, point it at your fork of this rules file.

## License

[BSD-style](License.md) — Copyright (c) 2026 Subham Mahesh
