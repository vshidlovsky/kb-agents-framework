# Canonical Examples Output Template

> This template defines the structure of the generated `{NN}-canonical-examples.md` document.
> The KB Generator reads this template and fills it with extracted data.
> Delete the `> TEMPLATE` blocks when generating.

---

```markdown
# Canonical Examples

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: new features, unfamiliar packages.

> TEMPLATE: Each canonical pattern is a "copy-paste-this" reference from the newest/cleanest
> parts of the codebase. The goal is NOT to dump code — it's to point agents at the right
> source file, explain what makes it the gold standard, and list the structural elements
> to replicate. One sentence on what makes it canonical, bullet list of what to copy,
> and one anti-pattern to avoid.
>
> Categories:
> - Feature Module (end-to-end feature structure: files, folders, wiring)
> - State Management (how state is created, exposed, consumed)
> - API Integration (service class, DTOs, error handling, retry)
> - Testing (test file structure, mocking approach, assertion patterns)
> - UI Component (reusable widget/component structure, props, composition)

## Overview

{N} canonical patterns across {M} categories.

| Category | Count |
|----------|-------|
| Feature Module | {N} |
| State Management | {N} |
| API Integration | {N} |
| Testing | {N} |
| UI Component | {N} |

## Canonical Patterns

### CP-001: {Short descriptive title}

- **Category**: {Feature Module / State Management / API Integration / Testing / UI Component}
- **Source**: `{package name}` — `{file path}`
- **Why canonical**: {One sentence — what makes this the best current example}
- **Key structural elements** (replicate these):
  - {Element 1 — e.g., "Barrel file re-exports all public API from src/"}
  - {Element 2 — e.g., "State class is immutable with copyWith, created via factory"}
  - {Element 3 — e.g., "Error handling wraps all API calls in Result type"}
  - {Element 4 — ...}
- **Anti-pattern to avoid**: {What the older/worse approach looks like — e.g., "Older packages use mutable state classes with direct field assignment (see `packages/legacy_wallet/`)"}

---

### CP-002: {Short descriptive title}

- **Category**: {category}
- **Source**: `{package name}` — `{file path}`
- **Why canonical**: {one sentence}
- **Key structural elements** (replicate these):
  - {element}
  - {element}
  - {element}
- **Anti-pattern to avoid**: {older approach}

---

{Continue with CP-003, CP-004, etc.}

## Patterns by Category

> TEMPLATE: Summary table grouping all patterns by category for quick lookup.
> Agents scanning for "how do I build a new feature module?" jump straight to this table.

| Category | Patterns | Source Packages |
|----------|----------|-----------------|
| Feature Module | CP-001, CP-00{X} | `{package_a}`, `{package_b}` |
| State Management | CP-00{X} | `{package_c}` |
| API Integration | CP-00{X} | `{package_d}` |
| Testing | CP-00{X} | `{package_e}` |
| UI Component | CP-00{X} | `{package_f}` |

## See Also

- [Shared Code]({NN}-shared-code.md) — shared package APIs (canonical examples often live in shared packages)
- [Project Conventions]({NN}-project-conventions.md) — naming conventions these patterns follow
- [Gotchas]({NN}-gotchas.md) — anti-patterns and version confusion (the "don't copy this" counterpart)
```
