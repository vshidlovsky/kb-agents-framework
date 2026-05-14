# Doc Type Registry

Reference file read by the KB Generator agent. Defines all 15 built-in doc type packs and the format for custom packs.

For each pack: what to extract, where to look, what to produce, and how it connects to other docs.

---

## How to read this file

The KB Generator receives a doc type name (e.g., `repo-map`) and reads the matching entry below. Each entry has:

- **Extraction strategy** — step-by-step instructions for what to scan and how
- **Scope globs** — file patterns that define this doc's source files (also used by the refresher for staleness detection)
- **Output format** — what the generated doc should contain
- **Load priority** — hot / warm / cold (written into doc header and manifest)
- **Cross-references** — which other docs this one links to
- **Template** — path to the output template in `templates/doc-types/`

Custom packs follow the same structure — see the Custom Pack Format section at the end.

---

## Tier 1: Universal

### repo-map

> Template: `templates/doc-types/repo-map.md`
> Load priority: **hot**

**Extraction strategy:**

1. Find root build/workspace manifest (`melos.yaml`, `nx.json`, `lerna.json`, `pnpm-workspace.yaml`, `Cargo.toml` workspace, root `pom.xml`)
2. `find . -maxdepth 4 -name "pubspec.yaml" -o -name "package.json" -o -name "pom.xml" -o -name "build.gradle*" -o -name "go.mod" -o -name "Cargo.toml" -o -name "pyproject.toml"` (excluding build artifacts, node_modules)
3. For each manifest: parse package name, path, version, internal dependencies
4. Build adjacency list: package → [depends on]
5. Identify hub packages: depended on by >50% of other packages
6. Read build tool configs for versions (Flutter SDK, Node, Java, Go, Rust, Python)
7. For each package, extract a one-line "Key Observations" note — dependency count, notable features, version info from pubspec
8. Inventory scripts directory (if present): list script files with one-line purpose
9. Check for workspace field in root pubspec.yaml (Dart workspaces) — note entry count
10. Check for shared_dependencies.yaml or similar shared config files
11. Generate Mermaid dependency diagram from adjacency list (packages as nodes, internal deps as directed edges)

**Scope globs:**

```
**/pubspec.yaml
**/package.json
**/pom.xml
**/build.gradle*
**/go.mod
**/Cargo.toml
**/pyproject.toml
melos.yaml
nx.json
lerna.json
pnpm-workspace.yaml
```

**Output format:**

1. Tooling table:

| Tool | Version | Config File |
|------|---------|-------------|
| Flutter | 3.24.0 | `.fvmrc` |

2. Package inventory table:

| Package | Path | Description | Key Observations | Depended-On-By |
|---------|------|-------------|------------------|----------------|
| `core` | `packages/core` | Shared domain models | 23 deps, exports domain types | 12 |

3. Hub packages callout (>50% dependents)
4. Mermaid dependency diagram

> Package inventory table MUST include a "Key Observations" column. This is a hot-priority hub document — density matters more than brevity here.

**Cross-references:** Referenced by app-profiles, shared-code, gotchas

---

### app-profiles

> Template: `templates/doc-types/app-profile.md`
> Load priority: **hot**

**Extraction strategy:**

1. For each app/service listed in `kb-context.md` Apps/Services in Scope:
2. Find entry point: `main.dart`, `index.tsx`, `index.ts`, `main.go`, `Application.java`, `Program.cs`
3. Trace DI registration from entry point (look for service locator setup, provider tree, module registration)
4. Trace routing setup (GoRouter, React Router, Express routes, Spring controllers)
5. Identify state management approach from imports (Bloc/Cubit, Redux, MobX, Zustand, ChangeNotifier)
6. Read environment config (flavor/scheme files, .env references, config classes)

**Scope globs:** (varies by stack — examples)

```
apps/*/lib/main.dart
apps/*/src/index.*
apps/*/src/main/java/**/Application.java
apps/*/src/main.go
apps/*/Program.cs
```

**Output format:**

One file per app in `docs/project-kb/02-apps/{app-name}.md`. Sections:

1. Entry Point (file path, what it bootstraps)
2. Environment Config (flavors/schemes, config sources)
3. State Management (approach, key state classes)
4. Routing (router type, route count, route file path)
5. Dependency Injection (DI mechanism, registration file)
6. Pages/Endpoints overview (count, top-level grouping)

**Cross-references:** References repo-map for package context

---

### shared-code

> Template: `templates/doc-types/shared-code.md`
> Load priority: **hot**

**Extraction strategy:**

1. For each shared package/module (from repo-map, excluding apps):
2. Read directory structure (`find {pkg}/lib -maxdepth 2 -type d` or equivalent)
3. Parse barrel files for public exports (e.g., `lib/{pkg}.dart`, `src/index.ts`)
4. Count files per directory to show package shape
5. Identify key classes by import frequency — grep for each exported symbol across the project, rank by reference count (most-referenced first)
6. For top-N symbols: note what they do (class/function/mixin/type, one line)

**Scope globs:**

```
packages/*/lib/**
modules/*/src/**
libs/**
shared/**
```

**Output format:**

Per-package sections:

| Directory | Files | Purpose |
|-----------|-------|---------|
| `lib/src/models/` | 12 | Domain models |

Key exports (ranked by usage):

| Export | Type | Used By (count) |
|--------|------|-----------------|
| `WalletManager` | class | 47 files |

**Cross-references:** References repo-map for package list

---

### gotchas

> Template: `templates/doc-types/gotchas.md`
> Load priority: **hot**

**Extraction strategy:**

1. Scan for version confusion: `find . -maxdepth 3 -type d` looking for v1/v2 directories, `_old` suffixes, duplicate package names with different paths
2. Detect dead code: packages listed in workspace config but not imported by any app
3. Identify naming inconsistencies: grep for patterns that break established naming (e.g., mixed camelCase and snake_case in same domain)
4. Find stale dependencies: packages with no commits in last 6 months that are still depended on
5. Check for orphaned packages: listed in workspace config but missing from disk (or vice versa)
6. Detect co-change pairs: `git log --format= --name-only` on last 500 commits, find files that always change together without import links (hidden coupling)
7. Check for misleading names: packages/classes whose name suggests one thing but code does another

> When citing files as evidence for a gotcha, include the line number where the evidence is found (e.g., `pubspec.yaml:135`). Use `grep -n` instead of `grep` for all evidence-gathering commands. The citation field should be formatted as `path/to/file:line — evidence description`.

**Scope globs:** All source directories (uses repo-map output as primary input)

**Output format:**

Per-gotcha entries:

```
### G-001: {Short title}

**Problem:** {What's confusing or dangerous}
**Rule:** {What agents should do about it}
**Citation:** `{file path}` — {evidence}
```

**Cross-references:** References repo-map, shared-code

---

### canonical-examples

> Template: `templates/doc-types/canonical-examples.md`
> Load priority: **warm**

Unlike auto-generated doc types, canonical examples follow a **human-confirmation pattern** (same as project-conventions): agents propose candidate patterns, users confirm each one before it enters the doc. This ensures the KB only recommends patterns the team actually endorses.

**Extraction strategy:**

1. Identify newest packages: use `git log --diff-filter=A --format=%ai --name-only` to find most recently added source files, then rank packages by recency of new file creation. Also flag packages with explicit "v2" or "new" naming.
2. For each candidate package, find the cleanest feature module: look for highest test coverage (count test files relative to source files), most consistent naming patterns, and most complete file structure (barrel exports, dedicated model/service/state directories).
3. Extract the structural pattern: how files are organized (directory layout), how state flows (creation → exposure → consumption), how API calls are wired (service → DTO → consumer), how tests are structured (setup → act → assert, mocking approach).
4. Compare against older packages to identify what the newer approach does differently — this becomes the "anti-pattern to avoid" for each entry.
5. Present each candidate pattern to user for confirmation (same human-gate as conventions). Only confirmed patterns enter the document.

**Scope globs:** Broad — all source packages (uses repo-map output as primary input to identify packages)

```
apps/*/lib/**
apps/*/src/**
packages/*/lib/**
packages/*/src/**
modules/*/src/**
libs/**
```

**Output format:**

Per-pattern entries:

```
### CP-001: {Short title}

**Source:** `{package}` — `{file path}`
**Why canonical:** {one sentence}
**Key structural elements:** {bullet list of what to replicate}
**Anti-pattern to avoid:** {what the older approach looks like}
```

Plus a Patterns by Category summary table.

**Cross-references:** References shared-code, project-conventions, gotchas

---

## Tier 2: Frontend/Mobile

### screen-inventory

> Template: `templates/doc-types/screen-inventory.md`
> Load priority: **warm**

**Extraction strategy:**

1. Detect routing framework from imports (GoRouter, AutoRoute, Navigator 2.0, React Router, Vue Router, Angular Router, Next.js pages/app)
2. Find route registration files: grep for `GoRoute`, `@RoutePage`, `createBrowserRouter`, `Route path=`, etc.
3. For each route: extract route string, component/page file path, route name
4. Map each page file to its package/module ownership
5. Count total screens per package

**Scope globs:**

```
**/router/**
**/routes/**
**/pages/**
**/screens/**
**/views/**
```

**Output format:**

Per-package tables:

| Screen Name | Route | File Path | Notes |
|-------------|-------|-----------|-------|
| SendMoneyPage | `/send-money` | `packages/mt/lib/pages/send_money_page.dart` | |

**Cross-references:** Links to navigation-graph, dependency-index

---

### navigation-graph

> Template: `templates/doc-types/navigation-graph.md`
> Load priority: **warm**

**Extraction strategy:**

1. Using screen-inventory output as base, trace all navigation calls from each screen file
2. Grep for: `context.go(`, `context.push(`, `Navigator.push`, `router.push`, `navigate(`, `Link to=`
3. For each call: extract source screen, trigger (button tap, swipe, auto-redirect), destination screen, transition type (push, replace, pop)
4. Build transition table per feature flow

**Scope globs:** Same as screen-inventory plus `**/navigation/**`

**Output format:**

Per-flow transition tables:

| From | Trigger | To | Type | Notes |
|------|---------|-----|------|-------|
| HomePage | "Send Money" tap | SendMoneyPage | push | |
| SendMoneyPage | success | ConfirmationPage | replace | clears back stack |

**Cross-references:** Links to screen-inventory

---

### dependency-index

> Template: `templates/doc-types/dependency-index.md`
> Load priority: **warm**

**Extraction strategy:**

1. Identify key classes: scan for files in managers/, controllers/, services/, view_models/, viewmodels/, hooks/, providers/, blocs/, cubits/, repositories/ directories
2. For each key class: grep for its import/usage across all page/screen/component files
3. Build reverse lookup: class → [list of screens that consume it]
4. For transitive consumers: if a class is consumed by a bloc/cubit/view-model that is then consumed by a screen, trace the full chain and note the intermediary (e.g., "via HomeViewModel")
5. Rank by consumer count (most-consumed first)

**Important: full file paths for all multi-consumer classes.** Every class with 2+ consumers must list every consumer with its full file path. Do not abbreviate paths or truncate consumer lists. This is the core value of the doc — agents need exact file paths for impact analysis, not summaries. If a class has 108 consumers, list all 108. Group consumers by package for readability.

**Scope globs:**

```
**/managers/**
**/controllers/**
**/services/**
**/view_models/**
**/viewmodels/**
**/hooks/**
**/providers/**
**/blocs/**
**/cubits/**
**/repositories/**
```

**Output format:**

Per-class entries with full consumer lists:

```
### NavigationManager (Manager) — 108 consumers

Source: `packages/core/lib/navigation/navigation_manager.dart`

- **auth_v2** (6): biometric_welcome (`packages/auth_v2/lib/pages/biometric_welcome_page.dart`), email_input (`packages/auth_v2/lib/pages/email_input_page.dart`), ...
- **egifts** (5): brand_search (`packages/egifts/lib/views/brand_search_page.dart`), ...
```

For classes with fewer than 10 consumers, use a flat list instead of package grouping:

```
### PaymentMethodManager (Manager) — 4 consumers

Source: `packages/credit_card/lib/managers/payment_method_manager.dart`

- CashInPage (`packages/wallet/lib/cash_in/presentation/pages/cash_in_page.dart`) — via CashInBloc
- CashOutPage (`packages/wallet/lib/cash_out/presentation/pages/cash_out_page.dart`) — via CashOutBloc
- ...
```

Plus a summary table at the end for quick lookup (class, type, defined in, consumer count).

**Cross-references:** Links to screen-inventory, api-registry

---

## Tier 3: Backend/API

### api-registry

> Template: `templates/doc-types/api-registry.md`
> Load priority: **warm**

**Extraction strategy:**

1. Find all HTTP service/client classes: grep for `@GET`, `@POST`, `@RestController`, `dio.get`, `fetch(`, `axios.`, `http.Client`, `HttpClient`
2. For each service: extract method (GET/POST/PUT/DELETE), path, request body shape, response shape
3. For each endpoint, also extract the response type/shape: look for return type annotations, `fromJson` calls, or response mapper usage. Document the response DTO or shape in the Response DTO column. If the response is a generic Result<T, E> wrapper, note the inner type T.
4. Trace to repository classes: find which repository/data-layer class wraps each service call. Grep for the service method name in repository files (`**/repositories/**`, `**/repos/**`, `**/data/**`). Record the repository class name in the Repository column.
5. Trace to consumer classes: from the repository (or service if no repository layer), find the UI-layer consumer — manager, cubit, bloc, view-model, controller, or page that invokes the call. Record in the Consumer column.
6. Map to DTOs: what data classes are used for request/response?
7. Document DTO shapes inline: after each service's endpoint table, add a **DTO Models** subsection listing the request and response DTOs used by that service. For each DTO, document field names and types in a compact table. This is the highest-value addition — agents need DTO shapes to write correct API calls without reading every model file.
8. Document the HTTP stack architecture: how requests flow from service class through middleware/interceptors to the network layer. This is high-value context for agents writing new API calls.
9. Group by service/domain

**Important: completeness target.** Every service class found in step 1 must appear as a section. Do not stop after a subset. If a project has 99 service classes, the output must have 99 sections. The difference between a useful api-registry and a mediocre one is exhaustive coverage.

**Scope globs:**

```
**/services/**
**/api/**
**/clients/**
**/controllers/**
**/handlers/**
**/routes/**
**/repositories/**
**/repos/**
**/data/**
```

**Output format:**

Per-service tables with 8 columns:

| Method | Path | Service File | Service Method | Repository | Consumer | Request DTO | Response DTO |
|--------|------|-------------|----------------|------------|----------|-------------|--------------|
| POST | `/v1/transfers` | `transfer_service.dart` | `createTransfer()` | `TransferRepository` | `SendMoneyCubit` | `TransferRequest` | `TransferResponse` |

After each service's endpoint table, add a DTO Models subsection:

```
#### DTO Models

**TransferRequest** (`packages/mt/lib/models/transfer_request.dart`)

| Field | Type | Notes |
|-------|------|-------|
| `amount` | `double` | |
| `currency` | `String` | ISO 4217 |
| `recipientId` | `String` | |

**TransferResponse** (`packages/mt/lib/models/transfer_response.dart`)

| Field | Type | Notes |
|-------|------|-------|
| `transferId` | `String` | |
| `status` | `TransferStatus` | enum |
| `estimatedArrival` | `DateTime` | |
```

Only document DTOs that are specific to the service. Skip generic wrappers (ApiResponse<T>, Result<T, E>) — just note the inner type.

**Cross-references:** Links to dependency-index, app-profiles

---

### database-schema

> Template: `templates/doc-types/database-schema.md`
> Load priority: **cold**

**Extraction strategy:**

1. Find migration files: `**/migrations/**`, `**/db/migrate/**`, `**/alembic/**`
2. Read ORM model definitions: `**/models/**`, `**/entities/**`
3. For each table/collection: extract name, columns/fields, types, relationships (FK, belongs_to, has_many)
4. Trace migration history: list migrations in order with summary of what each changed
5. Map tables to ORM model files

**Scope globs:**

```
**/migrations/**
**/models/**
**/entities/**
**/schema/**
**/db/**
```

**Output format:**

Per-table entries:

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | `bigint` | PK, auto-increment | |
| `user_id` | `bigint` | FK → users.id, NOT NULL | |

Relationships section: Table → Related Table → Type (1:N, N:M) → FK Column

**Cross-references:** Links to api-registry (which endpoints read/write which tables)

---

### service-map

> Template: `templates/doc-types/service-map.md`
> Load priority: **cold**

**Extraction strategy:**

1. Read orchestration files: `docker-compose*.yml`, `**/k8s/**`, `**/deploy/**`, `**/helm/**`
2. Find inter-service clients: HTTP clients pointing to other internal services, gRPC stubs
3. For each dependency: extract service name, protocol (HTTP/gRPC/AMQP), client file
4. Check for retry/circuit-breaker patterns in client code
5. Generate Mermaid service dependency diagram

**Scope globs:**

```
docker-compose*.yml
**/k8s/**
**/deploy/**
**/helm/**
**/clients/**
```

**Output format:**

| Service | Depends On | Protocol | Client File | Retry/CB |
|---------|-----------|----------|-------------|----------|
| `api-gateway` | `auth-service` | HTTP | `clients/auth_client.go` | retry 3x, CB threshold 5 |

Plus Mermaid service topology diagram.

**Cross-references:** Links to api-registry, env-config

---

## Tier 4: Cross-Cutting

### project-conventions

> Template: `templates/doc-types/project-conventions.md`
> Load priority: **warm**

Unlike other doc types, project conventions are **not auto-generated and dumped**. They follow the same human-confirmation pattern as PRD lessons: agents propose conventions, users confirm them one by one, confirmed conventions become canonical.

**Discovery strategy:**

1. Scan each domain below for existing naming patterns in the codebase
2. For each domain: collect 10+ examples, derive the pattern, calculate confidence (% of names matching)
3. Present each candidate convention individually to user for confirmation
4. Only confirmed conventions enter the doc

**Domains to scan:**

| Domain | What to scan | Example pattern |
|--------|-------------|-----------------|
| Analytics events | Event tracking calls (`trackEvent`, `logEvent`, `analytics.track`) | `{domain}_{action}_{target}` snake_case |
| Feature flags | Flag definitions (enum values, config keys) | `SCREAMING_SNAKE_CASE` with `ENABLE_` prefix |
| Routes | Route registrations (path strings) | `/feature-name/action` kebab-case |
| L10n keys | Translation key prefixes (ARB/JSON keys) | `{feature}_` prefix in snake_case |
| Log messages | Log/print calls (structured or unstructured) | `[ServiceName] message` or structured JSON |
| API paths | Endpoint definitions (controller annotations, route strings) | `/v1/{domain}/{resource}` versioned kebab-case |

**Scope globs:** Broad — touches analytics, flag, route, l10n, logging, and API files. Uses other doc type outputs (feature-flags, l10n-registry, api-registry) as input where available.

**Output format:**

Numbered entries in `docs/project-kb/project-conventions.md`:

```markdown
## C-001: Analytics Event Naming
- **Domain**: Analytics events
- **Pattern**: `{domain}_{action}_{target}` in snake_case
- **Examples**: `mt_send_money_tapped`, `auth_login_succeeded`, `wallet_balance_refreshed`
- **Writer rule**: Name all new analytics events following this pattern
- **Reviewer check**: Verify every event name in the PRD matches `{domain}_{action}_{target}` format
- **Confirmed**: 2026-05-12
```

**Cross-references:** Links to feature-flags, l10n-registry, api-registry, screen-inventory. Referenced by PRD writer and reviewer for naming correctness.

---

### feature-flags

> Template: `templates/doc-types/feature-flags.md`
> Load priority: **cold**

**Extraction strategy:**

1. Find flag definitions: grep for flag enums, `RemoteConfig` keys, LaunchDarkly/Unleash/Flagsmith client usage, Firebase Remote Config references
2. For each flag: extract name, provider key, default value, description (from comments or enum docs)
3. Grep for consumption points: where is each flag checked in the code?
4. Group by domain/feature area

**Scope globs:**

```
**/feature_flags/**
**/remote_config/**
**/config/**
**/flags/**
```

Plus known provider config files (`.launchdarkly/**`, `firebase_remote_config*`)

**Output format:**

| Flag Name | Provider Key | Default | What It Gates | Consumed By |
|-----------|-------------|---------|---------------|-------------|
| `ENABLE_GOOGLE_PAY` | `enable_google_pay` | `false` | Google Pay payment method | `PaymentMethodPage`, `CheckoutPage` |

**Cross-references:** Links to app-profiles, screen-inventory

---

### l10n-registry

> Template: `templates/doc-types/l10n-registry.md`
> Load priority: **cold**

**Extraction strategy:**

1. Find translation files: `**/*.arb`, `**/locales/**/*.json`, `**/*.po`, `**/*.xliff`
2. Parse all keys from the primary locale file
3. Group keys by prefix (first segment before `_` or `.`)
4. For each prefix: count keys, identify primary package owner, extract 3 example keys
5. Build tiered table (most keys first)

**Scope globs:**

```
**/*.arb
**/locales/**
**/i18n/**
**/translations/**
**/messages/**
```

**Output format:**

| Prefix | Feature Area | Primary Package | Key Count | Example Keys |
|--------|-------------|-----------------|-----------|--------------|
| `mt_` | Money Transfer | `packages/money_transfer` | 87 | `mt_send_button`, `mt_amount_label`, `mt_confirm_title` |

**Cross-references:** Links to screen-inventory (which screens use which prefixes)

---

### env-config

> Template: `templates/doc-types/env-config.md`
> Load priority: **cold**

**Extraction strategy:**

1. Find .env files: `.env`, `.env.example`, `.env.development`, `.env.production`, `.env.staging`
2. Read config/environment classes: grep for `process.env`, `Environment.`, `@Value("${`, config providers
3. For each variable: extract name, source, default value, per-environment override
4. Document secrets references (names only, never values): `API_KEY`, `DB_PASSWORD`, etc.
5. Map config to consumers: which service/module reads each variable?

**Scope globs:**

```
.env*
**/config/**
**/environment/**
**/settings/**
```

**Output format:**

| Variable | Source | Default | Per-Env Override | Consumed By | Secret |
|----------|-------|---------|-----------------|-------------|--------|
| `API_BASE_URL` | `.env` | `http://localhost:3000` | staging: `api.staging.example.com` | `ApiClient` | no |
| `DB_PASSWORD` | `.env` | — | all envs | `DatabaseConfig` | yes |

**Cross-references:** Links to app-profiles, service-map

---

## Custom Pack Format

Custom packs discovered by the setup agent or defined by users follow this structure.
Saved to `docs/kb-doc-templates/custom/{pack-name}.md` in the target project.

```markdown
# Custom Doc Type Pack: {pack-name}

> Load priority: {hot|warm|cold}

## Description

What this document captures and why it's valuable for agents working in this project.

## Extraction Strategy

Step-by-step instructions for what to scan and how to extract the information.
Be specific — the KB Generator follows these literally.

1. [Step 1]
2. [Step 2]
3. ...

## Scope Globs

File patterns that define this doc's source files (used for staleness detection).

```
glob/pattern/one
glob/pattern/two
```

## Output Format

Table schemas and section structure for the generated document.
Include example tables with column headers.

## Cross-References

Which other KB docs this one links to/from.

## Quality Checks

Pack-specific validation rules beyond the universal quality checks.
```

The KB Generator treats custom packs identically to built-in packs — same workflow, same quality guardrails, same manifest tracking.
