# Dependency Index Output Template

> Reverse lookup: class → screens that consume it. The KB Generator reads this template and fills it.
> Delete the `> TEMPLATE` blocks when generating.

---

```markdown
# Dependency Index

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: managers, controllers, services, view models.

## Overview

> TEMPLATE: One-line summary.

{N} key classes consumed by {M} screens. Top consumer: `{ClassName}` ({K} screens).

## Index by Class

> TEMPLATE: One entry per key class (manager, controller, service, view model, hook, provider).
> Ranked by consumer count (most-consumed first).
> Only include classes consumed by 2+ screens — single-use classes aren't useful as an index.
>
> "Consumers" are screens/pages/views that import or inject this class.
> This is the reverse lookup that's expensive to derive from code but trivial from a table.

### `{ClassName}` — {N} consumers

- **Defined in**: `{path/to/class_file}`
- **Type**: {manager / controller / service / view model / hook / provider}
- **Purpose**: {one sentence — what this class does}

| Consumer Screen | File Path | Usage |
|----------------|-----------|-------|
| `{ScreenName}` | `{path/to/screen}` | {how it uses the class — reads data / triggers action / listens to state} |
| `{ScreenName}` | `{path/to/screen}` | {usage} |

---

### `{NextClassName}` — {N} consumers

- **Defined in**: `{path}`
- **Type**: {type}
- **Purpose**: {purpose}

| Consumer Screen | File Path | Usage |
|----------------|-----------|-------|
| ... | ... | ... |

---

{Repeat for each key class, ranked by consumer count}

## Classes by Consumer Count

> TEMPLATE: Summary table for quick lookup. Same data as above, condensed.

| Class | Type | Defined In | Consumers |
|-------|------|-----------|-----------|
| `{ClassName}` | {type} | `{path}` | {count} |

## Orphaned Classes

> TEMPLATE: Key classes (in managers/, services/, etc. directories) with zero screen consumers.
> These may be consumed only by other services, or they may be dead code.
> Skip this section if none found.

| Class | Defined In | Notes |
|-------|-----------|-------|
| `{ClassName}` | `{path}` | {consumed by other services / likely dead code} |

## See Also

- [Screen Inventory]({NN}-screen-inventory.md) — screen names and routes referenced above
- [API Registry]({NN}-api-registry.md) — endpoints these classes call
- [Shared Code](03-shared-code.md) — package-level view of these classes
```
