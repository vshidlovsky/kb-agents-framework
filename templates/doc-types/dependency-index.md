# Dependency Index Output Template

> Reverse lookup: class → screens that consume it. The KB Generator reads this template and fills it.
> For large projects (500+ lines), use the split-file layout: summary index + per-package detail files.
> Delete the `> TEMPLATE` blocks when generating.

---

## Split Layout: Index File Template

> Use when combined output exceeds 500 lines.
> Target: 50-100 lines. This is what agents hot-load to decide which detail file to open.

```markdown
# Dependency Index

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: managers, controllers, services, view models.
> Split doc: detail files in `{NN}-dependency-index/`. Load the one matching your working package.

## Overview

{N} key classes with 2+ consumers across {K} packages. {O} orphaned classes (0 consumers).

## Classes by Consumer Count

> TEMPLATE: Full summary table — one row per multi-consumer class.
> Agents scan this to find which detail file has the class they need.

| Class | Type | Package | Consumers | Detail |
|-------|------|---------|-----------|--------|
| `{ClassName}` | {type} | `{package}` | {count} | [{package}.md]({NN}-dependency-index/{package}.md) |

## Detail Files

| Package | Classes | Total Consumers | File |
|---------|---------|-----------------|------|
| `{package}` | {N} | {M} | [{package}.md]({NN}-dependency-index/{package}.md) |

## See Also

- [Screen Inventory]({NN}-screen-inventory.md) — screen names and routes referenced in detail files
- [API Registry]({NN}-api-registry.md) — endpoints these classes call
- [Shared Code]({NN}-shared-code.md) — package-level view of these classes
```

## Split Layout: Detail File Template

> One file per source package. Target: 100-300 lines.
> Agent loads this when working in the matching package.

```markdown
# Dependency Index: {Package Name}

> Part of [{NN}-dependency-index.md](../{NN}-dependency-index.md). Generated {date} at commit {short_sha}.
> Load when working in `packages/{package}/` or on screens that consume classes from this package.

## Classes

> TEMPLATE: Full per-class entries for all classes DEFINED in this package.
> Ranked by consumer count. Full file paths for every consumer.

> TEMPLATE (10+ consumers): Group by consuming package.

### {ClassName} ({Type}) — {N} consumers

Source: `{path/to/class_file}`

- **{package_name}** ({count}): {ScreenName} (`{full/path/to/screen.ext}`), {ScreenName} (`{full/path}`), ...

---

> TEMPLATE (under 10 consumers): Flat list.

### {ClassName} ({Type}) — {N} consumers

Source: `{path/to/class_file}`

- {ScreenName} (`{full/path/to/screen.ext}`)
- {ScreenName} (`{full/path}`) — via {IntermediaryClass}

---

## Summary

| Class | Type | Defined In | Consumers |
|-------|------|-----------|-----------|
| `{ClassName}` | {type} | `{path}` | {count} |

## Orphaned Classes

> TEMPLATE: Key classes in this package with zero screen consumers. Skip if none.

| Class | Defined In | Notes |
|-------|-----------|-------|
| `{ClassName}` | `{path}` | {consumed by other services / likely dead code} |
```

## Single-File Template

> Use when combined output is under 500 lines. Same as split index + all detail content in one file.

```markdown
# Dependency Index

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: managers, controllers, services, view models.

## Overview

{N} key classes consumed by {M} screens. Top consumer: `{ClassName}` ({K} screens).

## Index by Class

> TEMPLATE: Same per-class entry format as the detail file template above.
> All classes in one file, ranked by consumer count.

{per-class entries}

## Classes by Consumer Count

| Class | Type | Defined In | Consumers |
|-------|------|-----------|-----------|
| `{ClassName}` | {type} | `{path}` | {count} |

## Orphaned Classes

| Class | Defined In | Notes |
|-------|-----------|-------|
| `{ClassName}` | `{path}` | {consumed by other services / likely dead code} |

## See Also

- [Screen Inventory]({NN}-screen-inventory.md) — screen names and routes referenced above
- [API Registry]({NN}-api-registry.md) — endpoints these classes call
- [Shared Code]({NN}-shared-code.md) — package-level view of these classes
```
