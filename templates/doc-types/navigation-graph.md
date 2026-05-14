# Navigation Graph Output Template

> Maps every screen-to-screen transition in the app. The KB Generator reads this template and fills it.
> Delete the `> TEMPLATE` blocks when generating.

---

```markdown
# Navigation Graph

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: routes, navigation, pages, screens.

## Overview

> TEMPLATE: One-line summary.

{N} transitions across {M} screens. {K} distinct navigation flows.

## Transitions by Flow

> TEMPLATE: Group transitions by user-facing flow (e.g., "Send Money", "Login", "Onboarding").
> Each table covers one logical flow. Order flows by frequency of use or importance.
>
> From/To should match screen names from the screen-inventory.
> Trigger is what the user does (tap, swipe, auto-redirect, deep link).
> Type is the navigation action (push, replace, pop, reset, dialog, bottom sheet).

### {Flow Name}

| From | Trigger | To | Type | Notes |
|------|---------|-----|------|-------|
| `{ScreenA}` | {user action} | `{ScreenB}` | {push/replace/pop} | |
| `{ScreenB}` | {user action} | `{ScreenC}` | {push} | |
| `{ScreenC}` | success | `{ScreenD}` | replace | clears back stack |
| `{ScreenC}` | cancel/back | `{ScreenB}` | pop | |

---

### {Next Flow}

| From | Trigger | To | Type | Notes |
|------|---------|-----|------|-------|
| ... | ... | ... | ... | ... |

---

{Repeat for each flow}

## Entry Points

> TEMPLATE: How users arrive at the first screen of each flow.
> Entry points include: app launch, deep links, push notifications, tab switches.

| Entry Point | Target Screen | Source |
|-------------|---------------|--------|
| App launch | `{HomeScreen}` | `{main.dart / index.tsx}` |
| Deep link `{pattern}` | `{TargetScreen}` | `{deep link handler file}` |
| Push notification `{type}` | `{TargetScreen}` | `{notification handler file}` |

## Dead-End Screens

> TEMPLATE: Screens with no outgoing navigation (user can only go back or close the app).
> These may be intentional (confirmation pages) or bugs (missing navigation).
> Skip this section if none found.

| Screen | Reason |
|--------|--------|
| `{ScreenName}` | {intentional terminal / missing navigation} |

## See Also

- [Screen Inventory]({NN}-screen-inventory.md) — full screen list with routes and file paths
- [Dependency Index]({NN}-dependency-index.md) — which services feed which screens
```
