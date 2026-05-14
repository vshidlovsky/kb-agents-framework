# L10n Registry Output Template

> Localization key inventory grouped by prefix/feature area. The KB Generator reads this
> template and fills it. Delete the `> TEMPLATE` blocks when generating.

---

```markdown
# L10n Registry

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: cold. Load when touching: arb, locales, i18n, translations, messages.

## Overview

> TEMPLATE: One-line summary.

{N} translation keys across {M} prefixes. Primary locale: `{locale code}` (`{path to primary locale file}`). {K} supported locales.

## Keys by Prefix

> TEMPLATE: One row per key prefix (first segment before `_` or `.`).
> Ranked by key count (most keys first).
> Primary package is the package/module that owns this feature area.
> Example keys are 3 representative keys from this prefix.

| Prefix | Feature Area | Primary Package | Key Count | Example Keys |
|--------|-------------|-----------------|-----------|--------------|
| `{mt_}` | {Money Transfer} | `{packages/money_transfer}` | {87} | `{mt_send_button}`, `{mt_amount_label}`, `{mt_confirm_title}` |
| `{auth_}` | {Authentication} | `{packages/auth}` | {42} | `{auth_login_title}`, `{auth_password_hint}`, `{auth_forgot_link}` |

## Locale Files

> TEMPLATE: All translation files by locale. One row per file.

| Locale | File Path | Key Count | Complete |
|--------|-----------|-----------|----------|
| `{en}` | `{path/to/intl_en.arb}` | {N} | {yes — primary} |
| `{es}` | `{path/to/intl_es.arb}` | {M} | {yes / no — missing {K} keys} |

## Missing Translations

> TEMPLATE: Keys present in the primary locale but missing in other locales.
> Skip this section if all locales are complete.
> Only list locales with missing keys, not fully translated ones.

| Locale | Missing Keys | Example Missing |
|--------|-------------|-----------------|
| `{es}` | {12} | `{mt_fee_disclosure}`, `{auth_biometric_prompt}` |

## Key Generation

> TEMPLATE: How translation keys are generated or managed.
> Skip this section if no code generation is detected.

| Tool | Config | Command |
|------|--------|---------|
| `{flutter gen-l10n / i18next / formatjs}` | `{config path}` | `{generation command}` |

## See Also

- [Screen Inventory]({NN}-screen-inventory.md) — which screens use which key prefixes
- [Project Conventions]({NN}-project-conventions.md) — l10n key naming convention (if documented)
- [App Profiles]({NN}-apps/) — per-app locale configuration
```
