# Project Conventions Output Template

> Naming conventions confirmed by the user. Unlike other doc types, conventions are individually
> proposed and confirmed — never auto-generated in bulk. The KB Generator reads this template
> and fills it. Delete the `> TEMPLATE` blocks when generating.

---

```markdown
# Project Conventions

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: any feature, PRD, analytics, routing, naming.
>
> These conventions are confirmed by a human — agents must follow them as hard rules.

## Overview

> TEMPLATE: One-line summary.

{N} confirmed conventions across {M} domains.

## Conventions

> TEMPLATE: One entry per confirmed convention, numbered sequentially (C-001, C-002, …).
> Each convention was individually proposed to the user and confirmed.
> Never auto-populate — every entry here has explicit user approval.
>
> Domains to look for:
> - Analytics events — event tracking calls (`trackEvent`, `logEvent`, `analytics.track`)
> - Feature flags — flag definitions (enum values, config keys)
> - Routes — route registrations (path strings)
> - L10n keys — translation key prefixes (ARB/JSON keys)
> - Log messages — log/print calls (structured or unstructured)
> - API paths — endpoint definitions (controller annotations, route strings)
>
> But any domain with a discoverable pattern qualifies.

### C-001: {Convention Name}

- **Domain**: {Analytics events / Feature flags / Routes / L10n keys / Log messages / API paths / Other}
- **Pattern**: `{pattern description with format}`
- **Examples**: `{example_1}`, `{example_2}`, `{example_3}`
- **Confidence**: {percentage of codebase names matching this pattern}
- **Writer rule**: {one sentence — how a PRD writer or developer should apply this convention}
- **Reviewer check**: {one sentence — how a reviewer should verify compliance}
- **Confirmed**: {date}

---

### C-002: {Convention Name}

- **Domain**: {domain}
- **Pattern**: `{pattern}`
- **Examples**: `{example_1}`, `{example_2}`, `{example_3}`
- **Confidence**: {percentage}
- **Writer rule**: {rule}
- **Reviewer check**: {check}
- **Confirmed**: {date}

---

{Repeat for each confirmed convention}

## Conventions by Domain

> TEMPLATE: Summary table for quick lookup.

| ID | Domain | Pattern | Confidence |
|----|--------|---------|------------|
| C-001 | {domain} | `{pattern summary}` | {percentage} |

## Rejected Conventions

> TEMPLATE: Conventions that were proposed but rejected by the user.
> Keeping these prevents re-proposing the same pattern during refresh.
> Skip this section if none were rejected.

| Domain | Proposed Pattern | Reason Rejected |
|--------|-----------------|-----------------|
| {domain} | `{pattern}` | {user's reason or "no reason given"} |

## See Also

- [Feature Flags]({NN}-feature-flags.md) — flag naming conventions reference this doc
- [L10n Registry]({NN}-l10n-registry.md) — key prefix conventions reference this doc
- [API Registry]({NN}-api-registry.md) — endpoint naming conventions reference this doc
- [Screen Inventory]({NN}-screen-inventory.md) — route naming conventions reference this doc
```
