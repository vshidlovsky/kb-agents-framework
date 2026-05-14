# App Profile Output Template

> This template defines the structure of each generated `02-apps/{app-name}.md` document.
> One file is generated per app/service in scope.
> The KB Generator reads this template and fills it with extracted data.
> Delete the `> TEMPLATE` blocks when generating.

---

```markdown
# {App Name}

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: hot. Load when touching: {app path}.

## Entry Point

> TEMPLATE: The main file that bootstraps this app. Trace what it sets up.

- **File**: `{path to main.dart / index.tsx / Application.java / main.go}`
- **Bootstraps**: {what the entry point initializes — e.g., "DI container → router → root widget"}

## Environment Config

> TEMPLATE: How this app reads environment/flavor/scheme configuration.

| Config Source | Path | What It Controls |
|---------------|------|-----------------|
| {source type — flavor, .env, config class} | `{path}` | {what it configures} |

**Flavors/Schemes** (if applicable):

| Flavor | Config File | API Base URL | Notes |
|--------|-------------|--------------|-------|
| {dev/staging/prod} | `{path}` | {base URL or "from env"} | |

## State Management

> TEMPLATE: What state management approach this app uses. Cite the imports.

- **Approach**: {Bloc/Cubit, Provider, Redux, MobX, Zustand, ChangeNotifier, etc.}
- **Evidence**: `{path}` — {import or registration that proves the approach}

Key state classes:

| State Class | Path | Scope |
|-------------|------|-------|
| `{ClassName}` | `{path}` | {what part of the app it manages} |

## Routing

> TEMPLATE: How screens are navigated in this app.

- **Router type**: {GoRouter, AutoRoute, Navigator 2.0, React Router, etc.}
- **Route file**: `{path to route registration}`
- **Route count**: {N} routes

Top-level route groups:

| Route Prefix | Screen Count | Purpose |
|--------------|-------------|---------|
| `/{prefix}` | {N} | {what this group covers} |

## Dependency Injection

> TEMPLATE: How services and managers are registered and provided.

- **DI mechanism**: {GetIt, Provider, Riverpod, Spring DI, Go wire, etc.}
- **Registration file**: `{path}`
- **Registered services**: {count}

Key registrations:

| Service | Type | Scope |
|---------|------|-------|
| `{ServiceName}` | {singleton/factory/lazy} | {what it provides} |

## Pages/Endpoints Overview

> TEMPLATE: High-level count and grouping. Details are in screen-inventory or api-registry.

- **Total pages/endpoints**: {count}
- **Grouped by**: {feature area, route prefix, package}

| Group | Count | Path Prefix |
|-------|-------|-------------|
| {group name} | {N} | `{path or route prefix}` |

## See Also

- [Repo Map](../01-repo-map.md) — package inventory and dependency graph
- [Shared Code](../03-shared-code.md) — shared packages this app imports
- [Screen Inventory](../05-screen-inventory.md) — full screen list with routes (if generated)
- [API Registry](../{NN}-api-registry.md) — endpoints this app calls (if generated)
```
