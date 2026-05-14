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

> TEMPLATE: One entry per key class (manager, controller, service, view model, bloc, cubit, hook, provider, repository).
> Ranked by consumer count (most-consumed first).
> Only include classes consumed by 2+ screens — single-use classes aren't useful as an index.
>
> "Consumers" are screens/pages/views that import or inject this class.
> This is the reverse lookup that's expensive to derive from code but trivial from a table.
>
> IMPORTANT: List EVERY consumer with its FULL file path. Do not abbreviate paths or truncate
> consumer lists. Agents need exact paths for impact analysis. If a class has 50 consumers, list all 50.
> For transitive consumers (class → bloc → screen), note the intermediary with "via {BlocName}".

> TEMPLATE (10+ consumers): Group consumers by package for readability.

### {ClassName} ({Type}) — {N} consumers

Source: `{path/to/class_file}`

- **{package_name}** ({count}): {ScreenName} (`{full/path/to/screen.ext}`), {ScreenName} (`{full/path}`), ...
- **{package_name}** ({count}): {ScreenName} (`{full/path}`) — via {IntermediaryClass}, ...

---

> TEMPLATE (under 10 consumers): Flat list, no package grouping needed.

### {ClassName} ({Type}) — {N} consumers

Source: `{path/to/class_file}`

- {ScreenName} (`{full/path/to/screen.ext}`)
- {ScreenName} (`{full/path}`) — via {IntermediaryClass}

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
- [Shared Code]({NN}-shared-code.md) — package-level view of these classes
```
