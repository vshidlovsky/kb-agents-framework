# Analytics Registry Output Template

> Event-level inventory of all analytics/tracking calls. The KB Generator reads this template and fills it.
> For large projects (500+ lines), use the split-file layout: summary index + per-domain detail files.
> Delete the `> TEMPLATE` blocks when generating.

---

## Split Layout: Index File Template

> Use when combined output exceeds 500 lines.
> Target: 50-100 lines. Contains tracking infrastructure + domain lookup table.

```markdown
# Analytics Registry

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: analytics, tracking, events, metrics.
> Split doc: detail files in `{NN}-analytics-registry/`. Load the one matching your working domain.

## Overview

{N} tracked events across {M} domains. Tracking SDK: {SDK name} (`{config_file}`).

## Tracking Infrastructure

> TEMPLATE: How events are fired — SDK setup, middleware/interceptors, global properties.
> This is shared across all domains — keep it in the index.

- **SDK/library**: `{name}` — initialized in `{init_file}`
- **Track call pattern**: `{e.g., analytics.track('event_name', properties)}` or `{trackEvent(EventName.value, params)}`
- **Global properties**: {properties attached to every event, e.g., user_id, app_version, platform}
- **Middleware/interceptors**: {any transform or enrichment layers}

## Events by Domain

> TEMPLATE: Lookup table — agents find which detail file has the events they need.

| Domain | Events | Detail |
|--------|--------|--------|
| `{domain}` | {count} | [{domain}.md]({NN}-analytics-registry/{domain}.md) |

## See Also

- [Screen Inventory]({NN}-screen-inventory.md) — which screens fire which events
- [Project Conventions]({NN}-project-conventions.md) — event naming rules
- [Feature Flags]({NN}-feature-flags.md) — flag-gated events
```

## Split Layout: Detail File Template

> One file per domain. Target: 100-300 lines.
> Agent loads this when working with features in the matching domain.

```markdown
# Analytics Registry: {Domain Name}

> Part of [{NN}-analytics-registry.md](../{NN}-analytics-registry.md). Generated {date} at commit {short_sha}.
> Load when adding or modifying analytics events in the {domain} domain.

## Events

| Event Name | Trigger | File Path | Parameters | Notes |
|------------|---------|-----------|------------|-------|
| `{event_name}` | `{user tap / page load / API response / etc.}` | `{path/to/file.dart:line}` | `{param1, param2}` | {conditional, A/B test, etc.} |

{Repeat for all events in this domain}
```

## Single-File Template

> Use when combined output is under 500 lines.

```markdown
# Analytics Registry

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: analytics, tracking, events, metrics.

## Overview

{N} tracked events across {M} domains. Tracking SDK: {SDK name} (`{config_file}`).

## Tracking Infrastructure

- **SDK/library**: `{name}` — initialized in `{init_file}`
- **Track call pattern**: `{description}`
- **Global properties**: {list}
- **Middleware/interceptors**: {if any}

## Events by Domain

> TEMPLATE: One table per domain. Sort by event count descending.

### {Domain Name} ({N} events)

| Event Name | Trigger | File Path | Parameters | Notes |
|------------|---------|-----------|------------|-------|
| `{event_name}` | `{trigger}` | `{path}` | `{params}` | |

---

{Repeat for each domain}

## Orphaned Events

> TEMPLATE: Events that don't match the naming convention or appear unreachable. Skip if none.

| Event Name | File Path | Issue |
|------------|-----------|-------|
| `{event_name}` | `{path}` | {naming mismatch / dead code / no trigger found} |

## See Also

- [Screen Inventory]({NN}-screen-inventory.md) — which screens fire which events
- [Project Conventions]({NN}-project-conventions.md) — event naming rules
- [Feature Flags]({NN}-feature-flags.md) — flag-gated events
```
