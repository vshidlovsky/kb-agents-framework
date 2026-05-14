# KB Agents Framework — Architecture

A multi-agent framework for generating and maintaining project knowledge bases that AI coding agents consume. Sibling to [prd-agents-framework](https://github.com/user/prd-agents-framework) — same structural patterns, different purpose.

---

## 1. Problem Statement

AI coding agents increasingly work with large, complex codebases. Before an agent can write code, review a spec, or research an initiative, it needs to understand what exists in the project: what packages are there, how they relate, what APIs are exposed, how screens connect, what feature flags gate behavior.

Today, agents discover this by grep-searching and tracing imports — every time, from scratch. This is slow, expensive, and error-prone. A project knowledge base (KB) provides this understanding as pre-indexed, structured reference documents that agents read before touching source code.

### Why existing tools don't solve this

The ecosystem has converged on two categories, neither of which covers the full problem:

**Context file generators** (Repowise, Repomix, RepoAgent) auto-generate documentation from code. But ETH Zurich research ("Evaluating AGENTS.md", Feb 2026) found that auto-generated comprehensive context files **reduce task success rates by ~3%** while increasing inference cost by 20%+. The problem: they produce verbose dumps that drown agents in marginally relevant information. Human-curated files improve success by ~4%, but only when kept minimal.

**Context file formats** (AGENTS.md, CLAUDE.md, Cursor Rules, Windsurf Rules) define where to put instructions. They say nothing about what a knowledge base should contain, how to generate it, or how to keep it current.

**Code intelligence engines** (GitNexus, Sourcegraph, CodeGraphContext) build knowledge graphs with MCP tools for live queries. These are powerful but require infrastructure (graph databases, indexing pipelines) and deliver context via API calls rather than pre-indexed documents. They solve "query the codebase" but not "give agents a curated lay of the land before they start working."

The gap is a framework that covers the full lifecycle: **what to generate → how to curate it → how to keep it fresh** — with the insight that curated and minimal beats comprehensive and auto-generated.

### Evidence from practice

A hand-built knowledge base for a 415K-line Flutter monorepo (12 documents, ~6,500 lines, ~360KB) demonstrates what works:

- Structured by concern: separate docs for repo topology, API registry, screen inventory, navigation graph, feature flags
- Tables for lookup, prose for explanation, citations for every claim
- Cross-references between docs (screen inventory → navigation graph, API registry → dependency index)
- Line-budgeted: average ~540 lines per doc, enough to navigate without overwhelming

This KB reduced the PRD researcher agent's work from full codebase discovery to targeted lookups. The framework automates producing knowledge bases like this one.

---

## 2. Design Principles

### Curated over comprehensive

The framework generates draft documents. Humans review and approve them before they become canonical. Auto-generated docs are proposals, not facts. This is the single most important design decision — it's what separates this framework from existing generators that produce verbose, uncurated dumps.

### Factual, not prescriptive

The KB documents **what exists** — packages, endpoints, screens, flags, dependencies. It does not document **how code should be written** — conventions, patterns, style rules. That boundary is deliberate: prescriptive rules require deep judgment about what to enforce versus tolerate. A separate framework ([agent-coding-conventions](https://github.com/user/agent-coding-conventions)) handles convention extraction with a different methodology (human-led curation, not automated extraction).

### Incremental refresh

A KB generated once and never updated becomes stale within weeks. The framework tracks which source files each document depends on. When those files change, only the affected documents are regenerated. Full regeneration is available but never required after initial setup.

### Cross-referenced, not redundant

Documents reference each other instead of duplicating information. The screen inventory links to the navigation graph. The API registry links to the dependency index. This keeps documents small and prevents staleness from duplicated data getting out of sync.

### Line-budgeted

Each document targets 200–700 lines. Documents exceeding 1,000 lines trigger a warning to split or summarize. The reference implementation averages ~540 lines per doc — enough to provide navigation value without becoming the verbose dumps that research shows hurt agent performance.

### Citation-grounded

Every claim in a generated document must cite a real file path. No inferences, no guesses. The validator agent checks that every cited path exists on disk. This makes the KB trustworthy — agents can follow citations to source code without risk of hallucinated references.

### Non-obvious only

Don't document what the code already says. Agents can read a `package.json` themselves — what they can't do cheaply is trace which 47 screens import `WalletManager` or which endpoints return `TransferDTO`. The KB captures relationships, reverse lookups, and cross-cutting concerns that are expensive to derive from source but trivial to consult from a table. If removing an entry wouldn't cost the agent more than one grep, it doesn't belong in the KB. (Informed by ETH Zurich's finding that redundant context increases inference cost without improving task success.)

---

## 3. Document Type Packs

Following the same pack architecture as the PRD framework's section packs. Each doc type is a self-contained pack with extraction strategy, output template, scope globs, and quality checks. Packs come in two flavors:

**Built-in packs** — 14 doc type packs shipped with the framework, organized into four tiers. These are the seed list — the discovery agent knows about them and recommends which to enable based on what it finds in the project.

**Custom packs** — doc types the discovery agent proposes after scanning the project, or that users define manually. If the agent finds something worth documenting that doesn't fit any built-in pack (e.g., a complex build pipeline, a mono-repo workspace layout, a domain-specific state machine, a test fixture registry), it proposes a custom pack with a name, description, scope globs, and extraction strategy. Custom packs follow the same format as built-in packs and are generated, validated, and refreshed identically.

The built-in list is a floor, not a ceiling. The discovery agent's real job is to understand the project and propose whatever KB documents would help agents work in it.

### Built-in Packs

#### Tier 1: Universal (any project type)

| Doc Type | What It Captures | Reference Equivalent |
|----------|-----------------|---------------------|
| **Repo Map** | Package/module inventory, internal dependency graph, tooling versions, build system, workspace config | `01-monorepo-map.md` (342 lines) |
| **App Profiles** | Per-app/service entry points, state management, routing, DI, environment config. One file per app. | `02-apps/*.md` (~200 lines each) |
| **Shared Code Catalog** | Shared packages/modules: directory structure, key exports, heavily-used APIs, barrel files | `03-shared-code.md` (696 lines) |
| **Gotchas** | Anti-patterns, pitfalls, dead code, version confusion, naming collisions, misleading structures | `06-gotchas.md` (527 lines) |

Every project gets Tier 1. These four documents form the foundation that higher-tier docs build on.

#### Tier 2: Frontend/Mobile (UI-bearing projects)

| Doc Type | What It Captures | Reference Equivalent |
|----------|-----------------|---------------------|
| **Screen Inventory** | Screen name → route → file path → package ownership tables | `07-boss-money-screens.md` (411 lines) |
| **Navigation Graph** | From → trigger → to → type transition tables for all screen-to-screen flows | `09-boss-money-nav-graph.md` (460 lines) |
| **Dependency Index** | Class → screens reverse lookup (which managers/controllers/services feed which pages) | `10-boss-money-dependency-index.md` (501 lines) |

Recommended for: Flutter, React Native, Swift, Kotlin mobile apps; React, Vue, Angular, Next.js web apps.

#### Tier 3: Backend/API (services, APIs, data layers)

| Doc Type | What It Captures | Reference Equivalent |
|----------|-----------------|---------------------|
| **API Registry** | Endpoint → method → service file → DTOs → consumers. Full HTTP contract extraction. | `08-boss-money-api-registry.md` (1,140 lines) |
| **Database Schema Map** | Tables/collections, relationships, migration history, ORM model mapping | (new) |
| **Service Map** | Service-to-service dependencies, protocols, retry/circuit-breaker patterns | (new) |

Recommended for: Spring Boot, Express, NestJS, Django, FastAPI, Go services. Also for frontend projects that consume APIs (API Registry is useful for both).

#### Tier 3 note: API Registry applies broadly

The API Registry is listed under Tier 3 but is valuable for any project that consumes HTTP APIs — including mobile and web apps. The setup agent recommends it whenever HTTP client code or API documentation is detected, regardless of project type.

#### Tier 4: Cross-Cutting (configuration, i18n, flags, conventions)

| Doc Type | What It Captures | Reference Equivalent |
|----------|-----------------|---------------------|
| **Project Conventions** | Naming patterns for events, flags, routes, API paths, l10n keys, log formats. Extracted from existing names in the codebase — verifiable, not opinion. | `04-conventions.md` (subset — naming patterns only) |
| **Feature Flags** | Flag name → key → default → what it gates → consumed by | `11-boss-money-rc-flags.md` (387 lines) |
| **L10n Registry** | Prefix → feature area → package → key count → example keys | `12-boss-money-l10n-prefixes.md` (269 lines) |
| **Environment Config** | Environment variables, config files, secrets references, per-environment differences | (new) |

Recommended based on detection: feature flag enums/configs → Feature Flags; ARB/JSON/PO translation files → L10n Registry; .env files or config classes → Environment Config. Project Conventions is always recommended — PRDs reference it to name events, flags, routes, and keys correctly.

### Custom Packs

The discovery agent proposes custom doc type packs when it finds project structures worth documenting that don't fit any built-in pack. Examples of what the agent might discover and propose:

| Discovery | Proposed Custom Pack |
|-----------|---------------------|
| Complex multi-stage CI/CD pipeline | `build-pipeline` — stages, triggers, artifact flow |
| Domain-specific state machines (payment flows, order lifecycle) | `state-machines` — states, transitions, guard conditions per entity |
| Monorepo workspace orchestration (Nx, Turborepo, Melos) | `workspace-graph` — task dependencies, build order, cache keys |
| Extensive test fixture/factory system | `test-fixtures` — factory → model mapping, shared seed data |
| Custom code generation (build_runner, OpenAPI, protobuf) | `codegen-registry` — generator → input → output → regeneration commands |
| GraphQL schema with resolvers | `graphql-registry` — types, queries, mutations, resolver files |

Custom packs follow the same structure as built-in packs:

```markdown
# Custom Doc Type Pack: {name}

## Description
What this document captures and why it's valuable for agents.

## Extraction Strategy
How to find and extract the information from the codebase.

## Scope Globs
Which files/directories to scan.

## Output Template
Table schemas and section structure for the generated document.

## Cross-References
Which other docs this one links to/from.

## Quality Checks
Pack-specific validation rules.
```

Custom pack definitions are saved to `templates/doc-types/custom/` in the target project and registered in `kb-context.md` under Custom Doc Type Packs. They persist across refreshes — the refresher tracks their scope globs in the manifest just like built-in packs.

### Project Conventions vs Coding Conventions

Two kinds of conventions exist. This framework handles one; a separate framework handles the other.

**Project conventions** (this framework) — naming patterns for things that appear in PRDs: analytics events (`mt_send_money_tapped` → `{domain}_{action}_{target}`), feature flags (`ENABLE_GOOGLE_PAY` → `SCREAMING_SNAKE_CASE`), route paths, l10n key prefixes, log message formats, API path patterns. These are factual and verifiable — scan existing names and the pattern is self-evident. The PRD writer needs these to write correct specs; the PRD reviewer checks against them.

**Coding conventions** (separate `agent-coding-conventions` framework) — how to structure code: state management patterns per package, error handling approaches, DI registration, file organization, test patterns. These require deep judgment about what to enforce versus tolerate and need human-led curation rather than automated extraction.

The boundary: if a PRD author needs it to write a correct spec, it's a project convention. If a developer needs it to write correct code, it's a coding convention. Some overlap exists — the project conventions doc may note "events use snake_case" while the coding conventions doc explains the full analytics event implementation pattern. Cross-reference, don't duplicate.

The KB framework detects if a conventions doc exists and cross-references it in `kb-context.md`, but does not generate or maintain it.

---

## 4. Agent Architecture

Four executable agents plus two reference files.

### Agents

| Agent | File | Role |
|-------|------|------|
| **KB Setup** | `agents/kb-setup.md` | One-time setup: detects project type, maps repo, recommends doc types, drafts `kb-context.md` |
| **KB Generator** | `agents/kb-generator.md` | Generates KB documents from source code. Reads doc type registry for extraction strategy per type. |
| **KB Validator** | `agents/kb-validator.md` | Validates generated docs against live code: file existence, route accuracy, completeness, cross-references |
| **KB Refresher** | `agents/kb-refresher.md` | Detects what changed since last generation, identifies stale docs, triggers targeted regeneration |

### Reference Files

| File | Role |
|------|------|
| `agents/doc-type-registry.md` | Defines all 14 built-in doc type packs: extraction strategy, scope globs, output template reference, cross-references, quality checks. Custom packs extend this at runtime. |
| `agents/quality-patterns.md` | Quality anti-patterns (verbose dumps, ungrounded claims, convention leakage) and validation checks. Read by generator and validator. |

### Why one generator, not one per pack

The generator is a single agent parameterized by the doc type pack, not one agent per doc type. The extraction logic varies by type, but the surrounding workflow (read context → read pack definition → scan code → apply template → write output → update manifest → commit) is identical. This avoids duplicate agent files and makes adding new doc types trivial — whether built-in or custom, the generator treats every pack the same way. Custom packs discovered by the setup agent work without any framework changes.

### Agent detail: KB Setup

Mirrors `prd-agents-framework/agents/project-setup.md`.

| Step | Action |
|------|--------|
| 0 | Read the blank `kb-context.md` template |
| 1 | Detect project type (pubspec.yaml → Flutter, package.json → JS/TS, pom.xml → Java, etc.) |
| 2 | Map repo layout (`find . -maxdepth 3 -type d`, excluding build artifacts) |
| 3 | Detect monorepo vs single-app vs multi-service |
| 4 | Identify apps/services (count entry points, app manifests) |
| 5 | Detect existing knowledge bases (`docs/project-kb/`, `docs/kb/`, embedded CLAUDE.md architecture) |
| 6 | **Built-in pack matching**: Check each built-in doc type pack against the project. Recommend enabling packs where the project has matching structures (Flutter pages → screen-inventory, HTTP clients → api-registry, .env files → env-config, etc.) |
| 7 | **Open-ended discovery**: Scan beyond the built-in list. Look for project structures that would be valuable for agents to know about but aren't covered by any built-in pack. Propose custom doc type packs with name, description, extraction strategy, and scope globs. Examples: CI/CD pipelines, state machines, codegen registries, workspace orchestration graphs. |
| 8 | **Present full pack proposal**: Show the user all recommended packs — built-in (with checkboxes, pre-checked for matches) + custom (each with description and rationale). User confirms, edits, or skips each. Same pattern as PRD section packs. |
| 9 | Ask user: scope (which apps/services if monorepo), conventions doc path |
| 10 | Draft `kb-context.md` with all fields filled, `[TODO]` for unknowns |
| 11 | Save custom pack definitions to `templates/doc-types/custom/` |
| 12 | Create `docs/project-kb/` directory and `docs/project-kb/.manifest.json` |
| 13 | Present summary, user reviews and confirms |

### Agent detail: KB Generator

The core extraction agent. Invoked per doc type (or batch of types) by the skill.

| Step | Action |
|------|--------|
| 0 | Read `.claude/kb-context.md` for project config, scope, exclusions |
| 1 | Read the doc type pack definition: built-in from registry, or custom from `docs/kb-doc-templates/custom/` |
| 2 | Read `docs/project-kb/.manifest.json` to check what already exists |
| 3 | Capture HEAD commit SHA for file reference grounding |
| 4 | Execute the extraction strategy from the registry (scan directories, read manifests, trace imports, build tables) |
| 5 | Apply the output template, generating markdown with citation-grounded tables and cross-references |
| 6 | Enforce quality guardrails: line budget check, citation check, no convention leakage |
| 7 | Write output to `docs/project-kb/{NN}-{doc-name}.md` with sequential numbering |
| 8 | Update `docs/project-kb/.manifest.json` with generation metadata |
| 9 | Commit: `docs(kb): generate {doc-name}` |

### Agent detail: KB Validator

Accuracy checker. Runs after generation or on demand.

| Step | Action |
|------|--------|
| 0 | Read `.claude/kb-context.md` and `docs/project-kb/.manifest.json` |
| 1 | For each doc, run checks from `quality-patterns.md`: |
|   | — File existence: every cited path → `[ -f "{path}" ]` |
|   | — Route accuracy: every route string → grep in route registration files |
|   | — API accuracy: every endpoint → verify service file still contains method+path |
|   | — Completeness: items in doc vs fresh scan count. Flag if >10% missing |
|   | — Cross-reference integrity: bidirectional links between related docs |
|   | — Line budget: warn if any doc exceeds 1,000 lines |
| 2 | Produce validation report per doc: PASS/FAIL per check |
| 3 | For FAILs: indicate full regeneration needed vs targeted fix |
| 4 | Commit validation report |

### Agent detail: KB Refresher

Incremental update agent. Detects what changed and regenerates only stale docs.

| Step | Action |
|------|--------|
| 0 | Read `.claude/kb-context.md` and `docs/project-kb/.manifest.json` |
| 1 | `git diff --name-only {last_commit_sha}..HEAD -- {source_dirs}` |
| 2 | Map changed files → affected doc types using scope globs from manifest |
| 3 | Compute freshness scores: `(unchanged scope files) / (total scope files)` per doc |
| 4 | **Hotspot analysis**: `git log --format= --name-only` on last 500 commits to identify high-churn files. Docs covering hotspot files get priority — they go stale faster. |
| 5 | For each stale doc: determine if full regeneration or targeted patch |
|   | — freshness ≥ 0.9 → attempt targeted patch (update specific rows/sections) |
|   | — freshness < 0.9 → full regeneration of that doc |
| 6 | Invoke kb-generator for each doc needing regeneration |
| 7 | Run kb-validator on all refreshed docs |
| 8 | Update manifest: new timestamps, commit SHAs, freshness scores, token estimates |
| 9 | Insert staleness warning headers into docs with freshness < 0.8 that weren't refreshed |
| 10 | Present summary: refreshed, untouched, new validation failures, hotspot docs flagged |
| 11 | Commit: `docs(kb): refresh {list of refreshed docs}` |

---

## 5. Skill Pipelines

Two skills orchestrate the agents with human gates between phases.

### `/generate-kb` — Initial Generation

```
/generate-kb [scope]
    │
    ├── Pre-flight
    │   Verify kb-context.md exists and is filled
    │
    ├── Phase 0: Discovery & Scope
    │   If monorepo: confirm which apps/services to cover
    │   Match project against built-in packs (pre-check matches)
    │   Open-ended discovery: scan for structures not covered by built-in packs
    │   Propose custom packs with name, description, rationale
    │   Present full pack list: built-in (checkboxes) + custom (each with description)
    │   🔵 Gate 0: User confirms which packs to enable (built-in + custom)
    │
    ├── Phase 1: Tier 1 Generation (universal docs)
    │   Spawn kb-generator for: repo-map, app-profiles, shared-code, gotchas
    │   🔵 Gate 1: User reviews Tier 1 docs
    │   "These are the factual foundation. Review for accuracy."
    │
    ├── Phase 2: Tier 2/3 + Custom Pack Generation (project-specific docs)
    │   Spawn kb-generator for enabled Tier 2/3 doc types + custom packs
    │   🔵 Gate 2: User reviews Tier 2/3 + custom docs
    │
    ├── Phase 3: Tier 4 Generation (cross-cutting docs, except conventions)
    │   Spawn kb-generator for enabled Tier 4 doc types (feature-flags, l10n, env-config)
    │   🔵 Gate 3: User reviews Tier 4 docs
    │
    ├── Phase 4: Convention Discovery
    │   Scan codebase for naming patterns across all domains
    │   (uses Tier 2/3/4 outputs as input where available)
    │   Present each candidate convention individually:
    │     "I found analytics events follow {domain}_{action}_{target}.
    │      Examples: mt_send_money_tapped, auth_login_succeeded.
    │      Confirm / edit / skip?"
    │   🔵 Gate 4: User confirms, edits, or skips each convention
    │   Write confirmed conventions to docs/project-kb/project-conventions.md
    │
    ├── Phase 5: Validation
    │   Spawn kb-validator on all generated docs
    │   Present validation report
    │   🔵 Gate 5: User reviews failures, decides to fix or accept
    │
    ├── Phase 6: Cross-Reference Pass
    │   Verify cross-references between docs are correct
    │   Add "See also" links where docs reference each other
    │   🔵 Gate 6: User confirms final KB
    │
    └── Completion
        Summary: generated docs, file paths, total lines, validation status
```

**Why tier-by-tier, not all at once:** Tier 1 docs inform later tiers — the generator uses the repo map to navigate when building screen inventories and API registries. Human review between tiers catches scope issues before investing in detailed extraction.

### `/refresh-kb` — Incremental Refresh

```
/refresh-kb
    │
    ├── Pre-flight
    │   Verify kb-context.md and .manifest.json exist
    │
    ├── Phase 1: Change Detection
    │   Spawn kb-refresher to detect stale docs
    │   Present: which docs are stale, why, scope of changes
    │   🔵 Gate 1: User confirms which docs to refresh
    │
    ├── Phase 2: Regeneration
    │   Spawn kb-generator for each doc to refresh
    │   🔵 Gate 2: User reviews refreshed docs
    │
    ├── Phase 3: Convention Check + Pack Discovery
    │   Scan refreshed docs + changed code for new naming patterns
    │   Compare against existing conventions in project-conventions.md
    │   If new patterns found: propose each individually
    │   Also check: do changed files reveal project structures that
    │     warrant a new custom pack not yet in the KB?
    │   🔵 Gate 3: User confirms new conventions + any new custom packs
    │
    ├── Phase 4: Validation
    │   Spawn kb-validator on refreshed docs
    │   Present validation report
    │   🔵 Gate 4: User confirms
    │
    └── Completion
        Summary: refreshed, untouched, new conventions added, validation status
```

---

## 6. Configuration

### `kb-context.md` — Project Configuration Template

Equivalent of `prd-agents-framework/project-context.md`. Copied into target project at `.claude/kb-context.md` during setup. Every agent reads it as Step 0.

```
# Knowledge Base Context

## Project Identity
- Name, description, tech stack, repo URL, repo structure (monorepo/single/multi-service)

## Repo Layout
- Path → contents table

## KB Configuration

### Output Path
- KB directory: `docs/project-kb/`
- Manifest file: `docs/project-kb/.manifest.json`

### Apps/Services in Scope
- Table: app name → path → include [x]/[ ]

### Included Doc Type Packs

> Built-in packs shipped with the framework. The setup agent pre-checks packs
> that match the project's structure. Mirrors PRD framework's section pack pattern.

- [x] repo-map — Package/module inventory, internal dependency graph, tooling versions
- [x] app-profiles — Per-app entry points, state management, routing, DI, environment config
- [x] shared-code — Shared packages/modules: directory structure, key exports, heavily-used APIs
- [x] gotchas — Anti-patterns, pitfalls, dead code, version confusion, naming collisions
- [ ] screen-inventory — Screen name → route → file path → package ownership (frontend/mobile)
- [ ] navigation-graph — Screen-to-screen transition tables (frontend/mobile)
- [ ] dependency-index — Class → screens reverse lookup (frontend/mobile)
- [ ] api-registry — Endpoint → method → service file → DTOs → consumers (backend, or any API consumer)
- [ ] database-schema — Tables, relationships, migration history, ORM mapping (backend)
- [ ] service-map — Service-to-service dependencies, protocols, retry patterns (backend)
- [x] project-conventions — Naming patterns for events, flags, routes, l10n keys, log formats
- [ ] feature-flags — Flag name → key → default → what it gates → consumed by
- [ ] l10n-registry — Prefix → feature area → package → key count → example keys
- [ ] env-config — Environment variables, config files, secrets references, per-env differences

### Custom Doc Type Packs

> Doc types discovered by the setup agent or defined by the user. Each custom pack
> is a markdown file following the same format as the built-in packs.
> The setup agent proposes custom packs when it finds project structures worth
> documenting that don't fit any built-in pack.

Custom doc type pack files (one path per line, or "none"):
- none

### Scoping Rules
- Include: [directories to scan]
- Exclude: [directories to skip]
- Line budget: [default 200-700, max 1000]

### Conventions Reference
- Conventions doc path (or "none") — cross-referenced, not duplicated
- Canonical examples doc path (or "none")

### Freshness & Loading

#### Load Priority Defaults
- hot (always load): repo-map, app-profiles, gotchas
- warm (load when touching related files): screen-inventory, api-registry, dependency-index, project-conventions
- cold (load on demand): l10n-registry, env-config, database-schema, service-map, feature-flags

#### Staleness Hook
- Enable pre-commit hook: [x] yes / [ ] no
- Hook warns when committing files that match a stale doc's scope globs

## Integration

### PRD Framework
- project-context.md path (if PRD framework is installed)
- KB path to set in project-context.md Research Configuration > Knowledge base
```

### `.manifest.json` — Generation State

Tracks per-document generation metadata for incremental refresh.

```json
{
  "version": 2,
  "projectName": "project-name",
  "lastFullGeneration": "2026-05-12T10:30:00Z",
  "lastRefresh": "2026-05-12T14:00:00Z",
  "documents": {
    "repo-map": {
      "file": "01-repo-map.md",
      "generatedAt": "2026-05-12T10:30:00Z",
      "commitSha": "abc1234",
      "sourceFileCount": 120,
      "lineCount": 342,
      "tokenEstimate": 2800,
      "freshness": 1.0,
      "loadPriority": "hot",
      "status": "current",
      "scopeGlobs": ["**/pubspec.yaml", "**/package.json", "melos.yaml"]
    },
    "api-registry": {
      "file": "08-api-registry.md",
      "generatedAt": "2026-05-10T09:00:00Z",
      "commitSha": "def5678",
      "sourceFileCount": 45,
      "lineCount": 1140,
      "tokenEstimate": 9200,
      "freshness": 0.72,
      "loadPriority": "warm",
      "status": "stale",
      "scopeGlobs": ["**/services/**", "**/api/**", "**/clients/**"],
      "staleSince": "2026-05-11T15:00:00Z",
      "staleFiles": ["lib/services/transfer_service.dart", "lib/api/auth_client.dart"]
    }
  }
}
```

**Key fields:**

- **`scopeGlobs`** — tells the refresher which files to watch. When `git diff` returns files matching a doc's scope globs, that doc is marked stale.
- **`freshness`** — 0.0 to 1.0 score. Calculated as `(unchanged scope files) / (total scope files)` since last generation. 1.0 = fully current, 0.0 = every scope file changed. More nuanced than binary current/stale — a doc at 0.95 probably just needs a patch, while 0.3 needs full regeneration. Inspired by Repowise's freshness scoring.
- **`loadPriority`** — `hot` (always load before any task), `warm` (load when working in related files), `cold` (load on demand). Inspired by the Codified Context paper's three-tier knowledge hierarchy. Agents use this to decide what to read first without consuming their full context window.
- **`tokenEstimate`** — approximate token count for the doc. Helps agents budget their context window. Calculated at generation time using a simple `lines × 8` heuristic (refinable per project).
- **`staleSince`** / **`staleFiles`** — when the doc became stale and which specific files triggered it. Helps the refresher decide between targeted patch and full regeneration.

---

## 7. Quality Guardrails

Defined in `agents/quality-patterns.md`, enforced by the generator and validator.

### Anti-patterns the generator must avoid

| Anti-Pattern | Description |
|-------------|-------------|
| **Verbose dump** | Reproducing file contents instead of summarizing structure. Max 3 lines of code citation per entry. |
| **Ungrounded claim** | Any statement not backed by a file path citation. |
| **Stale reference** | Citing a file that doesn't exist or a route that isn't registered. |
| **Scope creep** | Including information about packages/apps not in the KB scope. |
| **Coding convention leakage** | Documenting HOW to structure code (state management, DI, file organization). KB docs document naming patterns and what exists, not implementation patterns. |
| **Redundancy across docs** | Repeating information in multiple docs. Use cross-references. |
| **Missing cross-reference** | A screen inventory entry without a nav graph link, or API registry entry without a dependency index link. |
| **Documenting the obvious** | Including information agents can derive from a single file read or grep. KB value comes from relationships and reverse lookups, not restating what the code says. (ETH Zurich: redundant context increases cost without improving success.) |

### Validation checks

| Check | Method |
|-------|--------|
| File path exists | `[ -f "{path}" ]` for every cited path |
| Route registered | Grep route string in route registration files |
| API endpoint exists | Verify service file still contains method + path |
| Completeness | Count items in doc vs items found by fresh scan; flag if >10% missing |
| Line budget | Warn if any doc exceeds 1,000 lines |
| Cross-reference bidirectionality | If doc A references doc B entry X, doc B should reference doc A |

### Output format standards

Every generated document follows these rules:

- Header line: `> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.`
- Staleness warning (auto-inserted by refresher when freshness < 0.8): `> ⚠ STALE since {date} — {N} scope files changed. Run /refresh-kb to update.`
- Tables for inventories and registries, not prose
- **Local file paths, not permalinks.** Agents consume paths via the Read tool. Permalinks add noise to dense tables. The commit SHA in the header and manifest provides traceability. Staleness is handled by the refresh pipeline regenerating docs, not by permalink archaeology. (This differs from the PRD researcher, which uses permalinks because research docs are read weeks later when code may have changed.)
- Loading hint line: `> Load priority: {hot|warm|cold}. Load when touching: {scope globs summary}.`
- "See also" cross-references to related docs
- No verbatim code dumps — summarize structure, cite paths

---

## 8. Integration with PRD Framework

The PRD framework's researcher agent already supports knowledge bases. The integration requires no changes to the PRD framework:

1. **KB Setup** writes the KB path (`docs/project-kb/`) into `project-context.md` under `Research Configuration > Knowledge base`
2. **PRD Researcher** (Step 0) reads `docs/project-kb/` before touching source code — this behavior already exists
3. **Benefits**: Researcher uses repo map to navigate, screen inventory to find relevant screens, API registry for endpoint contracts, dependency index for class usage. Research time drops from full codebase discovery to targeted lookups.

The `kb-context.md` includes a field pointing to `project-context.md` so the KB setup agent can auto-configure this integration when both frameworks are installed.

---

## 9. Framework File Structure

### The framework repo

```
kb-agents-framework/
├── agents/
│   ├── kb-setup.md                # One-time setup agent
│   ├── kb-generator.md            # Document generation agent
│   ├── kb-validator.md            # Validation agent
│   ├── kb-refresher.md            # Incremental refresh agent
│   ├── doc-type-registry.md       # Reference: all doc types and strategies
│   └── quality-patterns.md        # Reference: anti-patterns and checks
├── skills/
│   ├── generate-kb/
│   │   └── SKILL.md               # Full generation pipeline
│   └── refresh-kb/
│       └── SKILL.md               # Incremental refresh pipeline
├── templates/
│   └── doc-types/                 # Built-in doc type pack templates
│       ├── repo-map.md
│       ├── app-profile.md
│       ├── shared-code.md
│       ├── gotchas.md
│       ├── screen-inventory.md
│       ├── navigation-graph.md
│       ├── dependency-index.md
│       ├── api-registry.md
│       ├── database-schema.md
│       ├── service-map.md
│       ├── project-conventions.md
│       ├── feature-flags.md
│       ├── l10n-registry.md
│       └── env-config.md
│       # Custom packs are created per-project at install time
├── kb-context.md                  # Config template (copied to target project)
├── docs/
│   └── architecture.md            # This document
├── README.md
├── LICENSE
└── .gitignore
```

### What gets installed in a target project

```
target-project/
├── .claude/
│   ├── kb-context.md              # Filled by kb-setup
│   ├── agents/
│   │   ├── kb-setup.md
│   │   ├── kb-generator.md
│   │   ├── kb-validator.md
│   │   ├── kb-refresher.md
│   │   ├── doc-type-registry.md
│   │   └── quality-patterns.md
│   └── skills/
│       ├── generate-kb/
│       │   └── SKILL.md
│       └── refresh-kb/
│           └── SKILL.md
├── docs/project-kb/                      # Generated KB (created by kb-setup)
│   ├── .manifest.json
│   ├── 01-repo-map.md
│   ├── 02-apps/
│   │   └── {app-name}.md
│   ├── 03-shared-code.md
│   ├── 04-gotchas.md
│   ├── 05-screen-inventory.md     # (if enabled)
│   ├── 06-navigation-graph.md     # (if enabled)
│   └── ...                        # remaining enabled doc types
└── docs/
    └── kb-doc-templates/          # Output templates (copied, customizable)
        ├── repo-map.md
        ├── ...                    # other built-in pack templates
        └── custom/                # Custom pack definitions (created by setup agent)
            └── {custom-pack-name}.md
```

Doc type templates are copied to `docs/kb-doc-templates/` (not `.claude/`) because they are project artifacts that teams may customize. Custom packs discovered by the setup agent are saved to the `custom/` subdirectory and registered in `kb-context.md` under Custom Doc Type Packs.

---

## 10. Implementation Sequence

### Phase 1: Foundation

| Priority | File | Purpose |
|----------|------|---------|
| 1 | `kb-context.md` | Configuration template |
| 2 | `agents/doc-type-registry.md` | All 14 built-in doc type packs defined + custom pack format spec |
| 3 | `agents/quality-patterns.md` | Quality guardrails |
| 4 | `templates/doc-types/repo-map.md` | First output template |
| 5 | `agents/kb-generator.md` | Extraction agent (Tier 1 only) |
| 6 | `README.md` | Pipeline overview, getting started |

### Phase 2: Setup and Orchestration

| Priority | File | Purpose |
|----------|------|---------|
| 7 | `agents/kb-setup.md` | Project detection and config |
| 8 | `skills/generate-kb/SKILL.md` | Full generation pipeline |
| 9 | Remaining Tier 1 templates | app-profile, shared-code, gotchas |

### Phase 3: Validation and Refresh

| Priority | File | Purpose |
|----------|------|---------|
| 10 | `agents/kb-validator.md` | Accuracy checking |
| 11 | `agents/kb-refresher.md` | Incremental refresh |
| 12 | `skills/refresh-kb/SKILL.md` | Refresh pipeline |

### Phase 4: Extended Doc Types

| Priority | File | Purpose |
|----------|------|---------|
| 13 | Tier 2 templates | screen-inventory, navigation-graph, dependency-index |
| 14 | Tier 3 templates | api-registry, database-schema, service-map |
| 15 | Tier 4 templates | feature-flags, l10n-registry, env-config |

### Phase 5: Polish

| Priority | File | Purpose |
|----------|------|---------|
| 16 | Custom pack discovery tuning | Improve setup agent's open-ended discovery heuristics |
| 17 | Cross-reference pass logic | In generate-kb skill |
| 18 | Integration test | Run against money-app-2 reference project |

---

## 11. Reference Implementation Analysis

The hand-built `docs/project-kb/` in money-app-2 provides the empirical basis for this framework's design decisions. Key observations:

### What makes it effective for agents

**Structured for lookup, not reading.** The docs use tables with consistent column schemas. An agent needing an endpoint contract looks up a row in the API registry table — it doesn't parse prose. This is the fundamental difference between agent-oriented docs and human-oriented docs.

**Citation-grounded.** Every claim cites a real file path. An agent can follow citations to read source code without guessing. The KB is a reliable index, not a summary that might be wrong.

**Separation of concerns by document scope.** Each doc owns one dimension of the codebase: topology (repo map), behavior (screens, navigation), contracts (API registry), configuration (flags, l10n). An agent working on a specific task pulls only the relevant docs, not the whole KB.

**Cross-referencing over duplication.** The screen inventory references the navigation graph. The API registry references the dependency index. Information appears once and is linked, not repeated.

**Inverse indices.** The dependency index (class → screens) and API registry (endpoint → consumers) are reverse lookups that are expensive to derive from code but trivial to consult from a table. These are the highest-value docs for agents.

### What the framework must automate

| Manual Step | Framework Equivalent |
|-------------|---------------------|
| Read every pubspec.yaml, count packages, build dependency list | Repo Map generator: `find` + manifest parsing |
| Trace each app's entry point, DI, routing, state management | App Profile generator: entry point detection + import tracing |
| Catalog shared packages, barrel exports, key classes | Shared Code generator: barrel file parsing + import frequency |
| Document all screens with routes and file paths | Screen Inventory generator: route registration scanning |
| Map screen-to-screen transitions | Navigation Graph generator: navigation call tracing |
| Extract all HTTP endpoints with DTOs and consumers | API Registry generator: service class scanning + consumer grep |
| Identify feature flag enums and consumption points | Feature Flags generator: flag definition + usage scanning |
| Discover anti-patterns, version confusion, dead code | Gotchas generator: naming analysis + orphan detection |
| Extract naming patterns for events, flags, routes, keys | Project Conventions generator: scan existing names → derive patterns |

### What the framework cannot automate

**Judgment calls.** The gotchas doc includes explanations like "this naming is preserved for backwards compatibility" — that requires institutional knowledge. The framework flags naming confusion but cannot explain *why* the confusion exists.

**Coding convention rules.** "Use Cubit in wallet, ChangeNotifier in money_transfer_v2" is a prescriptive rule derived from team decisions, not code analysis. This belongs in the coding conventions framework.

**Canonical examples.** "This file is the gold standard for a Scope widget" is a human judgment. The KB framework can identify frequently-imported patterns but cannot certify them as canonical.

These gaps are why human curation is a deliberate step in the pipeline, not an afterthought.

---

## 12. Built-in Doc Type Packs — Detailed Entries

Each entry defines what the generator extracts, how it extracts it, and what the output looks like. Custom packs follow the same structure but are defined per-project by the setup agent's discovery pass.

### Tier 1

#### repo-map

- **Extraction**: Read root manifest → find all package manifests → parse names, paths, internal dependencies → build adjacency list → identify hub packages (depended on by >50% of others) → read build tool configs for versions → generate Mermaid dependency diagram from adjacency list
- **Scope globs**: `**/pubspec.yaml`, `**/package.json`, `**/pom.xml`, `**/build.gradle*`, `**/go.mod`, `**/Cargo.toml`, `**/pyproject.toml`, `melos.yaml`, `nx.json`, `lerna.json`
- **Output**: Tooling table (tool → version → config file) + Package inventory table (name → path → description → depended-on-by count) + Hub packages callout + Mermaid dependency diagram (auto-generated from the adjacency list, showing packages as nodes and internal dependencies as edges)
- **Load priority**: `hot` — agents should always load the repo map first
- **Cross-references**: Referenced by app-profiles, shared-code, gotchas

#### app-profiles

- **Extraction**: For each app in scope → find entry point (main.dart, index.tsx, Application.java, main.go) → trace DI registration → trace routing setup → identify state management imports → read env config
- **Scope globs**: `apps/*/lib/main.dart`, `apps/*/src/index.*`, `apps/*/src/main/java/**/Application.java` (varies by stack)
- **Output**: One file per app. Sections: Entry Point, Environment Config, State Management, Routing, DI, Pages/Endpoints overview
- **Cross-references**: References repo-map for package context

#### shared-code

- **Extraction**: For each shared package/module → read directory structure → parse barrel files for exports → count files per directory → identify key classes by import frequency across the project → rank symbols by reference count (most-referenced first, inspired by Aider's PageRank approach)
- **Scope globs**: `packages/*/lib/**`, `modules/*/src/**`, `libs/**` (varies by stack)
- **Output**: Per-package sections with directory → contents tables + key exports list + heavily-used APIs
- **Cross-references**: References repo-map for package list

#### gotchas

- **Extraction**: Scan for version confusion (v1/v2 directories, duplicate package names) → detect dead code (packages not imported by any app) → identify naming inconsistencies → find stale dependencies → check for orphaned packages in workspace config → detect co-change pairs from git history (files that always change together without import links — hidden coupling, inspired by Repowise's git analytics)
- **Scope globs**: All source directories (uses repo-map output as input)
- **Output**: Per-gotcha entries: Problem statement → Rule → Citation
- **Cross-references**: References repo-map, shared-code

### Tier 2

#### screen-inventory

- **Extraction**: Find all route registrations (GoRouter, AutoRoute, Navigator, React Router, Vue Router, etc.) → extract route string + component/page file → map to package ownership
- **Scope globs**: `**/router/**`, `**/routes/**`, `**/pages/**`, `**/screens/**` (varies by stack)
- **Output**: Per-package tables: Screen Name → Route → File Path → Notes
- **Cross-references**: Links to navigation-graph, dependency-index

#### navigation-graph

- **Extraction**: Trace all navigation calls (pushNamed, go, navigate, etc.) from each screen → extract from → trigger → to → transition type
- **Scope globs**: Same as screen-inventory plus `**/navigation/**`
- **Output**: Per-flow transition tables: From → Trigger → To → Type → Notes
- **Cross-references**: Links to screen-inventory

#### dependency-index

- **Extraction**: Identify key classes (managers, controllers, services, view models) → for each, grep for usage in page/screen files → build reverse lookup
- **Scope globs**: `**/managers/**`, `**/controllers/**`, `**/services/**`, `**/view_models/**`, `**/viewmodels/**`, `**/hooks/**`
- **Output**: Per-class entries: Class definition path → List of consuming screens with file paths
- **Cross-references**: Links to screen-inventory, api-registry

### Tier 3

#### api-registry

- **Extraction**: Find all HTTP service/client classes → extract method + path + request/response shapes → trace to consumer classes → map to DTOs
- **Scope globs**: `**/services/**`, `**/api/**`, `**/clients/**`, `**/controllers/**`, `**/handlers/**`
- **Output**: Per-service tables: Method → Path → Service File → Service Method → Repository → Consumer → Request DTO → Response DTO
- **Cross-references**: Links to dependency-index, app-profiles

#### database-schema

- **Extraction**: Find migration files → read ORM model definitions → extract table names, columns, relationships → trace migration history
- **Scope globs**: `**/migrations/**`, `**/models/**`, `**/entities/**`, `**/schema/**`
- **Output**: Per-table entries: Table → Columns → Relationships → Migration file → ORM model file
- **Cross-references**: Links to api-registry (which endpoints read/write which tables)

#### service-map

- **Extraction**: Read Docker Compose / Kubernetes manifests → find inter-service HTTP/gRPC clients → extract service dependencies → document protocols and patterns
- **Scope globs**: `docker-compose*.yml`, `**/k8s/**`, `**/deploy/**`, `**/clients/**`
- **Output**: Service dependency table: Service → Depends On → Protocol → Client File → Retry/Circuit Breaker
- **Cross-references**: Links to api-registry, env-config

### Tier 4

#### project-conventions

Unlike other doc types, project conventions are **not auto-generated and dumped**. They follow the same human-confirmation pattern as PRD lessons: agents propose conventions, users confirm them one by one, confirmed conventions become canonical. This prevents noisy or wrong patterns from entering the inventory.

**Lifecycle:**

- **KB setup (initial)**: Scan existing names across multiple domains → propose candidate conventions → user confirms each → write confirmed conventions to `docs/project-kb/project-conventions.md`
- **KB refresh (ongoing)**: Detect new naming patterns not covered by existing conventions → propose additions → user confirms → append to conventions file
- **PRD writer**: Reads conventions file, follows the rules when naming new events, flags, routes, keys
- **PRD reviewer**: Reads conventions file, checks that proposed names follow the patterns

**Convention format** (mirrors PRD lessons structure):

```markdown
## C-001: Analytics Event Naming
- **Domain**: Analytics events
- **Pattern**: `{domain}_{action}_{target}` in snake_case
- **Examples**: `mt_send_money_tapped`, `auth_login_succeeded`, `wallet_balance_refreshed`
- **Writer rule**: Name all new analytics events following this pattern
- **Reviewer check**: Verify every event name in the PRD matches `{domain}_{action}_{target}` format
- **Confirmed**: 2026-05-12
```

**Domains to scan:**

| Domain | What to scan | Example pattern |
|--------|-------------|-----------------|
| Analytics events | Event tracking calls | `{domain}_{action}_{target}` snake_case |
| Feature flags | Flag definitions | `SCREAMING_SNAKE_CASE` with `ENABLE_` prefix |
| Routes | Route registrations | `/feature-name/action` kebab-case |
| L10n keys | Translation key prefixes | `{feature}_` prefix in snake_case |
| Log messages | Log/print calls | `[ServiceName] message` or structured JSON |
| API paths | Endpoint definitions | `/v1/{domain}/{resource}` versioned kebab-case |

- **Scope globs**: Broad — touches analytics, flag, route, l10n, logging, and API files. Uses other doc type outputs (feature-flags, l10n-registry, api-registry) as input where available.
- **Output**: `docs/project-kb/project-conventions.md` — numbered entries (C-001, C-002, ...) with pattern, examples, writer rule, reviewer check
- **Cross-references**: Links to feature-flags, l10n-registry, api-registry, screen-inventory. Referenced by PRD writer and reviewer for naming correctness.

#### feature-flags

- **Extraction**: Find flag definitions (enums, config classes, LaunchDarkly/Firebase RC/Unleash configs) → extract flag name + key + default → grep for consumption points
- **Scope globs**: `**/feature_flags/**`, `**/remote_config/**`, `**/config/**` + known flag provider config files
- **Output**: Per-domain tables: Flag Enum/Name → Provider Key → Default → What It Gates → Consumed By
- **Cross-references**: Links to app-profiles, screen-inventory

#### l10n-registry

- **Extraction**: Read ARB/JSON/PO/XLIFF translation files → group keys by prefix → count per prefix → map prefix to feature package → extract example keys
- **Scope globs**: `**/*.arb`, `**/locales/**`, `**/i18n/**`, `**/translations/**`
- **Output**: Tiered tables: Prefix → Feature Area → Primary Package → Key Count → Example Keys
- **Cross-references**: Links to screen-inventory (which screens use which prefixes)

#### env-config

- **Extraction**: Read .env files → read config classes → identify per-environment overrides → document secrets references (not values) → map config to consumers
- **Scope globs**: `.env*`, `**/config/**`, `**/environment/**`
- **Output**: Config table: Variable/Key → Source → Default → Per-Env Override → Consumed By
- **Cross-references**: Links to app-profiles, service-map

---

## 13. Design Decisions and Trade-offs

| Decision | Rationale | Alternative Considered |
|----------|-----------|----------------------|
| Single generator agent | Extraction varies by type but workflow is identical. Avoids duplicate agents. Custom packs work without framework changes. | One agent per doc type — more isolated but massive duplication |
| Pack-based doc types | Built-in packs as seed list + open-ended discovery for custom packs. Mirrors PRD section packs. Doesn't limit the agent to a predefined menu. | Fixed doc type list — simpler but misses project-specific structures |
| Tier-by-tier generation | Tier 1 informs later tiers. Human review between tiers catches scope issues early. | All-at-once — faster but riskier |
| JSON manifest for refresh | Enables precise per-doc staleness detection via scope globs + commit SHA. | Git notes or branch-based tracking — more complex, less transparent |
| Conventions excluded | Prescriptive rules require human judgment. Automated extraction produces verbose dumps. | Include conventions — would blur the factual/prescriptive boundary |
| Line budgets (200-700, warn at 1000) | Reference implementation averages ~540 lines per doc. Stays in the sweet spot for agent consumption. | No limit — leads to verbose dumps; strict limit — may truncate valuable content |
| Human curation gates | Research shows auto-generated docs hurt performance without curation. | Fully automated — faster but lower quality |
| Cross-references over redundancy | Prevents staleness from duplicated data; keeps docs small. | Self-contained docs — easier to read in isolation but data rots |
| Templates copied to target project | Teams may customize templates for project-specific needs. | Templates stay in framework — less flexibility |
| Freshness scoring (0-1) | More granular than binary stale/current. Helps refresher decide patch vs full regen. Inspired by Repowise's per-page freshness. | Binary status — simpler but loses nuance for prioritization |
| Load priority tags (hot/warm/cold) | Agents can't load the full KB in one context window. Tags tell them what to read first. Inspired by Codified Context paper's three-tier hierarchy. | Load everything — exceeds context budgets; let agent decide — wastes turns discovering relevance |
| Token estimates per doc | Agents need to budget their context window. Token count alongside line count gives actionable sizing. | Line count only — doesn't reflect actual context cost |
| Staleness warning headers | Auto-inserted in doc headers when manifest detects drift. Agents see warnings at read time without checking the manifest. | Manifest-only tracking — agents must check manifest before reading each doc |
| Pre-commit staleness hook | Warns at commit time when changes affect KB docs. Catches drift early. Inspired by GitNexus's pre-commit hooks. | Manual refresh only — drift accumulates silently |
| Non-obvious content only | Don't document what one grep can find. KB value comes from relationships and reverse lookups. ETH Zurich: redundant context increases cost without improving success. | Comprehensive documentation — produces verbose dumps that hurt agent performance |
| Hotspot-aware refresh priority | High-churn files go stale faster. Docs covering hotspots should be refreshed first. Inspired by Repowise's git analytics layer. | FIFO refresh — treats all docs equally regardless of code velocity |

---

## 14. Research Basis

Design decisions in this framework are informed by academic research, open-source tool analysis, and practitioner experience reports. This section documents the key sources and what was borrowed from each.

### Academic Research

**"Evaluating AGENTS.md" (ETH Zurich, Feb 2026)** — Tested context files across Claude Code, Codex, and Qwen Code on AGENTbench (138 tasks) and SWE-bench Lite (300 tasks). Key findings that shaped this framework:
- LLM-generated context files **reduce** task success by ~3% while increasing inference cost by 20%+
- Human-written files improve success by ~4% but only when minimal — "unnecessary requirements from context files make tasks harder"
- Architectural overviews don't help agents find files faster — agents discover README independently
- Agents follow instructions faithfully, so verbose instructions cause unnecessary exploration

*Framework response:* Curated-over-comprehensive principle, human gates, non-obvious-only principle, line budgets.

**"Codified Context: Infrastructure for AI Agents" (Feb 2026)** — Case study of 283 development sessions on a 108K-line C# system. Measured a 24.2% knowledge-to-code ratio across three tiers of context.
- Three-tier knowledge hierarchy: Hot (~660 lines, always loaded), Specialist (domain-specific), Cold (on-demand retrieval)
- Knowledge duplication outperformed retrieval — agents with pre-loaded context made fewer errors than those relying on lookup
- Specification staleness was the #1 failure mode — outdated docs misled agents into conflicting code
- Maintenance cost: ~1-2 hours weekly, ~5 minutes per session

*Framework response:* Load priority tags (hot/warm/cold), freshness scoring, staleness warning headers, incremental refresh pipeline.

**"Context Engineering for Coding Agents" (Martin Fowler, 2025-2026)** — Practitioner guide emphasizing "curating what the model sees" over feeding everything. Key insight: scope rules strategically — load guidance only when relevant files are accessed, not all at once.

*Framework response:* Scope-based loading hints in doc headers, scope globs in manifest for selective loading.

### Open-Source Tools Analyzed

| Tool | What We Borrowed | What We Didn't |
|------|-----------------|----------------|
| **Repowise** — codebase intelligence with 4 layers (graph, git, docs, decisions) | Freshness scoring per doc, hotspot detection from git analytics, co-change detection for gotchas | MCP-as-delivery (we use curated docs, not live queries), auto-generated wikis (violates curated-over-comprehensive) |
| **Aider repo-map** — tree-sitter symbol extraction with PageRank ranking | Graph-ranked symbol importance for shared-code/dependency-index, token budget awareness per doc | Dynamic per-session context selection (our docs are pre-generated, not session-scoped) |
| **GitNexus** — knowledge graph engine with MCP tools (28K stars) | Blast radius concept for refresh impact analysis, pre-commit hooks for staleness detection, Mermaid diagram generation | Full knowledge graph infrastructure (over-engineered for curated doc use case), PolyForm license |
| **Repomix** — context packing into AI-friendly formats (22K stars) | Confirmed that XML-structured output works well for AI consumption — validates our structured table format | Whole-repo packing (violates line budgets and selective loading) |
| **RepoAgent** — LLM-powered documentation generation | Hierarchical doc generation (symbols → files → modules → repo) — informed our tier-by-tier generation order | Fully automated generation without human gates (violates curated-over-comprehensive) |

### Practitioner Guides

**Augment Code "How to Build Your AGENTS.md" (2026)** — Key insights adopted:
- "One real snippet beats three paragraphs" → conventions pack uses concrete examples
- Three-tier permission boundaries (Always / Ask First / Never) → gotchas doc format
- Non-obvious patterns only — skip what agents can infer from training data
- Keep context files under 150-200 lines → our line budgets

**Packmind ContextOps (2026)** — Four-phase lifecycle (Build → Distribute → Govern → Maintain) that validates our generate → curate → refresh pipeline design. Key insight: only a small fraction of repos have adopted any AI config files — early adopters gain significant advantage.
