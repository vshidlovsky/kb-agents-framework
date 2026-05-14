# Screen Inventory Output Template

> One row per screen/page in the app. The KB Generator reads this template and fills it.
> For large projects (500+ lines), use the split-file layout: summary index + per-package detail files.
> Delete the `> TEMPLATE` blocks when generating.

---

## Split Layout: Index File Template

> Use when combined output exceeds 500 lines.
> Target: 50-100 lines. Contains routing infrastructure + package lookup table.

```markdown
# Screen Inventory

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: routes, pages, screens, views.
> Split doc: detail files in `{NN}-screen-inventory/`. Load the one matching your working package.

## Overview

{N} screens across {M} packages. Router: {GoRouter / Navigator 1.0 / etc.} (`{router_file}`).

## Route Registration

- **Router file**: `{path}`
- **Registration pattern**: {description}
- **Total registered routes**: {N}

## Screens by Package

> TEMPLATE: Lookup table — agents find which detail file has the screens they need.

| Package | Screens | Router | Detail |
|---------|---------|--------|--------|
| `{package}` | {count} | {GoRouter/Nav 1.0} | [{package}.md]({NN}-screen-inventory/{package}.md) |

## Unrouted Screens

> TEMPLATE: Screens not registered in any router. Skip if none found.

| Screen Name | File Path | Likely Reason |
|-------------|-----------|---------------|
| `{ScreenName}` | `{path}` | {dialog / bottom sheet / dead code} |

## See Also

- [Navigation Graph]({NN}-navigation-graph.md) — screen-to-screen transitions
- [Dependency Index]({NN}-dependency-index.md) — which managers/services feed which screens
- [App Profiles]({NN}-apps/) — per-app routing overview
```

## Split Layout: Detail File Template

> One file per package. Target: 100-300 lines.
> Agent loads this when working in the matching package.

```markdown
# Screen Inventory: {Package Name}

> Part of [{NN}-screen-inventory.md](../{NN}-screen-inventory.md). Generated {date} at commit {short_sha}.
> Load when working in `packages/{package}/` or on routes owned by this package.

## Screens

| Screen Name | Route | File Path | Notes |
|-------------|-------|-----------|-------|
| `{ScreenName}` | `{/route/path}` | `{path/to/screen}` | {guards, redirects, or blank} |

{Repeat for all screens in this package}
```

## Single-File Template

> Use when combined output is under 500 lines.

```markdown
# Screen Inventory

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: routes, pages, screens, views.

## Overview

{N} screens across {M} packages. Router: {GoRouter / etc.} (`{router_file}`).

## Screens by Package

> TEMPLATE: One table per package. Sort by screen count descending.

### {Package Name} ({N} screens)

| Screen Name | Route | File Path | Notes |
|-------------|-------|-----------|-------|
| `{ScreenName}` | `{/route/path}` | `{path/to/screen}` | |

---

{Repeat for each package}

## Route Registration

- **Router file**: `{path}`
- **Registration pattern**: {description}
- **Total registered routes**: {N}

## Unrouted Screens

| Screen Name | File Path | Likely Reason |
|-------------|-----------|---------------|
| `{ScreenName}` | `{path}` | {dialog / bottom sheet / dead code} |

## See Also

- [Navigation Graph]({NN}-navigation-graph.md) — screen-to-screen transitions
- [Dependency Index]({NN}-dependency-index.md) — which managers/services feed which screens
- [App Profiles]({NN}-apps/) — per-app routing overview
```
