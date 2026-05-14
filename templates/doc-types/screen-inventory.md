# Screen Inventory Output Template

> One row per screen/page in the app. The KB Generator reads this template and fills it.
> Delete the `> TEMPLATE` blocks when generating.

---

```markdown
# Screen Inventory

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: routes, pages, screens, views.

## Overview

> TEMPLATE: One-line summary of screen landscape.

{N} screens across {M} packages/modules. Router: {GoRouter / React Router / etc.} (`{router_file}`).

## Screens by Package

> TEMPLATE: One table per package that owns screens. Sort packages by screen count (descending).
> Each row is one screen/page/view. Route is the string used in navigation.
> File path is the component/widget file, not the route registration.

### {Package Name} ({N} screens)

| Screen Name | Route | File Path | Notes |
|-------------|-------|-----------|-------|
| `{ScreenName}` | `{/route/path}` | `{path/to/screen.dart}` | {guards, redirects, or blank} |

---

### {Next Package} ({N} screens)

| Screen Name | Route | File Path | Notes |
|-------------|-------|-----------|-------|
| `{ScreenName}` | `{/route/path}` | `{path/to/screen.tsx}` | |

---

{Repeat for each package}

## Route Registration

> TEMPLATE: Where routes are registered and how to find them.

- **Router file**: `{path}`
- **Registration pattern**: {description — e.g., "GoRoute objects in router.dart", "file-based routing in pages/"}
- **Total registered routes**: {N}

## Unrouted Screens

> TEMPLATE: Screens/pages that exist as files but aren't registered in the router.
> These may be dialogs, bottom sheets, or dead code. Skip this section if none found.

| Screen Name | File Path | Likely Reason |
|-------------|-----------|---------------|
| `{ScreenName}` | `{path}` | {dialog / bottom sheet / dead code / embedded widget} |

## See Also

- [Navigation Graph]({NN}-navigation-graph.md) — screen-to-screen transitions
- [Dependency Index]({NN}-dependency-index.md) — which managers/services feed which screens
- [App Profiles](02-apps/) — per-app routing overview
```
