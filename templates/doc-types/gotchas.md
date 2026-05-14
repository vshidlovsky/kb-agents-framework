# Gotchas Output Template

> This template defines the structure of the generated `04-gotchas.md` document.
> The KB Generator reads this template and fills it with extracted data.
> Delete the `> TEMPLATE` blocks when generating.

---

```markdown
# Gotchas

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: hot. Load when touching: all source directories.

> TEMPLATE: Each gotcha is a non-obvious pitfall that could trip up an agent (or developer)
> working in this codebase. Only include things that aren't apparent from reading the code —
> the "why" behind confusing structures, naming traps, dead code that looks alive, etc.
>
> Categories:
> - Version confusion (v1/v2 coexistence, legacy packages kept for compatibility)
> - Dead code (packages/files that exist but aren't used)
> - Naming traps (misleading names, inconsistent patterns)
> - Hidden coupling (files that must change together but have no import link)
> - Stale dependencies (packages with no recent activity that are still depended on)
> - Orphaned config (workspace entries with no matching directory, or vice versa)

## Summary

{N} gotchas found across {M} categories.

| Category | Count |
|----------|-------|
| Version confusion | {N} |
| Dead code | {N} |
| Naming traps | {N} |
| Hidden coupling | {N} |
| Stale dependencies | {N} |
| Orphaned config | {N} |

## Gotchas

### G-001: {Short descriptive title}

- **Category**: {version confusion / dead code / naming trap / hidden coupling / stale dependency / orphaned config}
- **Problem**: {What's confusing or dangerous — one or two sentences}
- **Rule**: {What agents should do about it — concrete, actionable}
- **Citation**: `{file path}` — {specific evidence}

---

### G-002: {Short descriptive title}

- **Category**: {category}
- **Problem**: {description}
- **Rule**: {action}
- **Citation**: `{file path}` — {evidence}

---

{Continue with G-003, G-004, etc.}

## Hidden Coupling (Co-Change Pairs)

> TEMPLATE: Files that frequently change together in git history but have no
> import/dependency link. These represent hidden coupling — changing one without
> the other is likely a bug.
>
> Only include pairs with strong co-change signal (changed together in >60% of commits
> that touch either file). Skip pairs where the coupling is obvious from imports.

| File A | File B | Co-Change Rate | Likely Reason |
|--------|--------|---------------|---------------|
| `{path_a}` | `{path_b}` | {N}% ({M} of {K} commits) | {brief explanation} |

## See Also

- [Repo Map]({NN}-repo-map.md) — package inventory (gotchas reference package names)
- [Shared Code]({NN}-shared-code.md) — shared package details (dead code may be in shared packages)
```
