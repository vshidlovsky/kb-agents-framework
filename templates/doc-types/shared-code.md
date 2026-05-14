# Shared Code Output Template

> This template defines the structure of the generated `03-shared-code.md` document.
> The KB Generator reads this template and fills it with extracted data.
> Delete the `> TEMPLATE` blocks when generating.

---

```markdown
# Shared Code Catalog

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: hot. Load when touching: packages/*, modules/*, libs/*, shared/*.

## Overview

> TEMPLATE: One-line summary of the shared code landscape.

{N} shared packages, {M} total exported symbols, {K} hub symbols (used by 10+ files).

## Packages

> TEMPLATE: One section per shared package. Only include packages that are actually
> shared (imported by at least one app or other package). Skip app-internal packages.
> Order by depended-on-by count (most-used first).

### {Package Name}

> TEMPLATE: Path, shape, and key exports. Don't reproduce file contents — cite paths.

- **Path**: `{package path}`
- **Depended on by**: {count} packages
- **Files**: {count}

#### Directory Structure

| Directory | Files | Purpose |
|-----------|-------|---------|
| `lib/src/{dir}/` | {N} | {one-phrase description} |

#### Key Exports (ranked by usage)

> TEMPLATE: List the top-N most-referenced symbols from this package.
> Ranked by how many files across the project import them.
> "Used by" count comes from grep across the full project.

| Export | Type | Used By (files) | Defined In |
|--------|------|-----------------|------------|
| `{SymbolName}` | {class/function/mixin/typedef/enum} | {count} | `{path}` |

#### Barrel File

> TEMPLATE: If the package has a barrel file (re-exports), note it.
> If not, skip this section.

- **Barrel**: `{path}` — re-exports {N} symbols

---

{Repeat ### section for each shared package}

## Cross-Package Dependencies

> TEMPLATE: Which shared packages depend on other shared packages.
> Only include internal dependencies, not external packages.

| Package | Depends On |
|---------|-----------|
| `{package_a}` | `{package_b}`, `{package_c}` |

## See Also

- [Repo Map](01-repo-map.md) — package inventory and dependency diagram
- [Dependency Index]({NN}-dependency-index.md) — which screens consume these classes (if generated)
- [Gotchas](04-gotchas.md) — anti-patterns found in shared code
```
