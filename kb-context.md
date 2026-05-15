# Knowledge Base Context

> Fill this file once per project. The KB agents read it before every run.
> Delete the `> GUIDE` blocks after filling each section.

---

## Project Identity

> GUIDE: One paragraph. What this project is, who uses it, what problem it solves.

- **Name**: [project name]
- **Description**: [one sentence — what it does and for whom]
- **Tech stack**: [language, framework, build tool — e.g., "Flutter 3.24, Dart 3.5, Melos monorepo"]
- **Repo**: [GitHub URL]
- **Repo structure**: [monorepo with multiple apps / single app / multi-service]

---

## Repo Layout

> GUIDE: Map the key directories so agents know where to look.
> The kb-setup agent will fill this automatically during project setup.
> Adapt the entries below to match your project — delete rows that don't apply, add rows that do.

| Path | Contains |
|------|----------|
| `apps/` | Application packages |
| `packages/` | Shared library packages |
| `docs/` | Documentation |
| `docs/project-kb/` | Generated knowledge base (this framework's output) |

---

## KB Configuration

### Output Path

- **KB directory**: `docs/project-kb/`
- **Manifest file**: `docs/project-kb/.manifest.json`
- **Doc templates directory**: `docs/kb-doc-templates/`

### Apps/Services in Scope

> GUIDE: List the apps or services the KB should cover.
> For monorepos, check which apps to include. The kb-setup agent fills this during project setup.

| App/Service | Path | Include |
|-------------|------|---------|
| [app-name] | `apps/[app-name]` | [x] |

### Included Doc Type Packs

> Built-in packs shipped with the framework. The setup agent pre-checks packs
> that match the project's structure. Mirrors PRD framework's section pack pattern.
>
> GUIDE: Check the packs that apply to your project. The kb-setup agent recommends
> which to enable based on what it finds in the project — you can override.

#### Tier 1: Universal (always recommended)

- [x] repo-map — Package/module inventory, internal dependency graph, tooling versions
- [x] app-profiles — Per-app entry points, state management, routing, DI, environment config
- [x] shared-code — Shared packages/modules: directory structure, key exports, heavily-used APIs
- [x] gotchas — Anti-patterns, pitfalls, dead code, version confusion, naming collisions
- [ ] canonical-examples — Best-practice implementation patterns from the newest/cleanest code

#### Tier 2: Frontend/Mobile

- [ ] screen-inventory — Screen name → route → file path → package ownership
- [ ] navigation-graph — Screen-to-screen transition tables
- [ ] dependency-index — Class → screens reverse lookup
- [ ] analytics-registry — Event name → trigger → file path → parameters → domain

#### Tier 3: Backend/API

- [ ] api-registry — Endpoint → method → service file → DTOs → consumers
- [ ] database-schema — Tables, relationships, migration history, ORM mapping
- [ ] service-map — Service-to-service dependencies, protocols, retry patterns
- [ ] log-registry — Log points → severity → file path → structured fields → alerts

#### Tier 4: Cross-Cutting

- [x] project-conventions — Naming patterns for events, flags, routes, l10n keys, log formats
- [ ] feature-flags — Flag name → key → default → what it gates → consumed by
- [ ] l10n-registry — Prefix → feature area → package → key count → example keys
- [ ] env-config — Environment variables, config files, secrets references, per-env differences

### Custom Doc Type Packs

> Doc types discovered by the setup agent or defined by the user. Each custom pack
> is a markdown file following the same format as the built-in packs.
> The setup agent proposes custom packs when it finds project structures worth
> documenting that don't fit any built-in pack.
>
> GUIDE: Add custom pack file paths below, or leave as "none".
> Custom packs are saved to `docs/kb-doc-templates/custom/` and follow the same format
> as built-in pack templates.

Custom doc type pack files (one path per line, or "none"):

- none

---

## Scoping Rules

> GUIDE: Configure what the KB agents scan and what they skip.

### Include

> Directories to scan for KB generation. One glob per line.

- `apps/**`
- `packages/**`
- `lib/**`

### Exclude

> Directories to skip. Build artifacts, generated code, test fixtures.

- `**/build/**`
- `**/node_modules/**`
- `**/.dart_tool/**`
- `**/generated/**`
- `**/*.g.dart`
- `**/*.freezed.dart`

### Budgets

- **Line budget per doc**: 200–700 (target), 1000 (warning threshold)
- **Token budget per doc**: ~5,600 (target, based on line budget × 8)

---

## Conventions Reference

> GUIDE: If your project has a coding conventions doc (from agent-coding-conventions framework
> or manual), reference it here. The KB framework cross-references it but does not generate or
> maintain it.

- **Conventions doc path**: none
- **Canonical examples doc path**: none

---

## Freshness & Loading

### Load Priority Defaults

> Determines which docs agents should read first. Agents use these tags to budget
> their context window — loading hot docs always, warm docs when relevant, cold docs on demand.

| Priority | Doc Types |
|----------|-----------|
| hot (always load) | repo-map, app-profiles, gotchas |
| warm (load when touching related files) | screen-inventory, api-registry, dependency-index, analytics-registry, log-registry, project-conventions, canonical-examples |
| cold (load on demand) | l10n-registry, env-config, database-schema, service-map, feature-flags |

### Staleness Hook

> GUIDE: Enable the pre-commit hook to get warned when committing files
> that match a stale KB doc's scope globs.

- **Enable pre-commit hook**: [x] yes / [ ] no
- Hook warns when committing files that match a stale doc's scope globs
- Hook reads `.manifest.json` to check freshness scores

---

## Integration

### PRD Framework

> GUIDE: If prd-agents-framework is also installed, fill these paths so the KB setup agent
> can auto-configure the integration. The PRD researcher reads the KB before touching source code.

- **project-context.md path**: [e.g., `.claude/project-context.md` — or "not installed"]
- **KB path to set in project-context.md**: `docs/project-kb/` (under Research Configuration > Knowledge base)
