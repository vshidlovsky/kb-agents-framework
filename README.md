# KB Agents Framework

Multi-agent framework for generating and maintaining project knowledge bases that AI coding agents consume. Built for [Claude Code](https://claude.ai/code).

## What It Does

Scans your codebase and generates structured reference documents — repo maps, API registries, screen inventories, navigation graphs, feature flag registries, and more — that AI coding agents read before working on your project. Agents get the "lay of the land" from pre-indexed docs instead of grep-searching from scratch every time.

## Why Not Just Auto-Generate?

Research shows auto-generated comprehensive context files **reduce agent task success by ~3%** while increasing inference cost by 20%+ ([ETH Zurich, Feb 2026](https://arxiv.org/abs/2602.11988)). Human-curated minimal docs improve success by ~4%.

This framework generates **drafts** that humans review before they become canonical. Every document goes through a human gate. That's what separates it from context dump tools.

## How It Works

```
/generate-kb
    │
    ├── Discovery & Scope
    │   Agent scans project, recommends built-in + custom doc type packs
    │   🔵 User confirms which packs to enable
    │
    ├── Tier 1 Generation (repo-map, app-profiles, shared-code, gotchas)
    │   🔵 User reviews
    │
    ├── Tier 2/3 + Custom Generation (project-specific docs)
    │   🔵 User reviews
    │
    ├── Tier 4 Generation (feature-flags, l10n, env-config)
    │   🔵 User reviews
    │
    ├── Convention Discovery (human-confirmed naming patterns)
    │   🔵 User confirms each convention individually
    │
    ├── Validation (file paths exist, routes registered, line budgets)
    │   🔵 User reviews failures
    │
    └── Cross-Reference Pass → Done
```

Incremental refresh via `/refresh-kb` — only regenerates docs whose source files changed.

## Key Design Decisions

- **Curated over comprehensive** — agents generate draft docs, humans review before they become canonical
- **Factual, not prescriptive** — documents what exists (packages, endpoints, screens, flags), not how code should be written
- **Non-obvious only** — don't document what one grep can find; KB value is relationships and reverse lookups
- **Incremental refresh** — tracks which source files each document depends on via scope globs and commit SHAs
- **Citation-grounded** — every claim cites a real file path; the validator checks that every cited path exists
- **Pack-based** — 17 built-in doc type packs as a seed list + open-ended discovery for custom packs

## Doc Type Packs

### Built-in (17 packs in 4 tiers)

| Tier | Packs |
|------|-------|
| **Universal** | repo-map, app-profiles, shared-code, gotchas, canonical-examples |
| **Frontend/Mobile** | screen-inventory, navigation-graph, dependency-index, analytics-registry |
| **Backend/API** | api-registry, database-schema, service-map, log-registry |
| **Cross-Cutting** | project-conventions, feature-flags, l10n-registry, env-config |

### Custom Packs

The setup agent scans beyond the built-in list and proposes custom packs for project structures that don't fit any built-in type (CI/CD pipelines, state machines, codegen registries, workspace graphs, etc.). Custom packs follow the same format and lifecycle as built-in packs.

## Framework Structure

```
kb-agents-framework/
├── agents/
│   ├── kb-setup.md                # One-time project setup agent
│   ├── kb-generator.md            # Document generation agent
│   ├── kb-validator.md            # Accuracy validation agent
│   ├── kb-refresher.md            # Incremental refresh agent
│   ├── doc-type-registry.md       # All 17 doc type packs defined
│   ├── quality-patterns.md        # Anti-patterns and validation checks
│   └── manifest-schema.md         # Canonical .manifest.json schema
├── skills/
│   ├── generate-kb/
│   │   └── SKILL.md               # Full generation pipeline (7 phases)
│   └── refresh-kb/
│       └── SKILL.md               # Incremental refresh pipeline (4 phases)
├── templates/
│   └── doc-types/                 # Output templates per doc type
│       ├── repo-map.md            # Tier 1
│       ├── app-profile.md
│       ├── shared-code.md
│       ├── gotchas.md
│       ├── canonical-examples.md
│       ├── screen-inventory.md    # Tier 2
│       ├── navigation-graph.md
│       ├── dependency-index.md
│       ├── analytics-registry.md
│       ├── api-registry.md        # Tier 3
│       ├── database-schema.md
│       ├── service-map.md
│       ├── log-registry.md
│       ├── project-conventions.md # Tier 4
│       ├── feature-flags.md
│       ├── l10n-registry.md
│       └── env-config.md
├── kb-context.md                  # Config template (copied to target project)
├── docs/
│   └── architecture.md            # Full architecture design
├── README.md
├── LICENSE
└── .gitignore
```

## Status

**Phase 5: Polish** — complete. All 4 agents, both skills, all 17 templates, manifest schema, and cross-reference resolution are built. See [docs/architecture.md](docs/architecture.md) for the full design.

### Implementation Phases

1. **Foundation** — kb-context.md, doc-type-registry, quality-patterns, repo-map template, kb-generator ✓
2. **Setup and Orchestration** — kb-setup agent, generate-kb skill, remaining Tier 1 templates ✓
3. **Validation and Refresh** — kb-validator, kb-refresher, refresh-kb skill ✓
4. **Extended Doc Types** — Tier 2/3/4 templates (all 16) ✓
5. **Polish** — cross-reference resolution, manifest schema, template consistency ✓

## Research Basis

Design decisions are informed by:
- [Evaluating AGENTS.md](https://arxiv.org/abs/2602.11988) (ETH Zurich, 2026) — auto-generated context hurts; curated minimal context helps
- [Codified Context](https://arxiv.org/html/2602.20478v1) (2026) — three-tier knowledge hierarchy, 24% knowledge-to-code ratio
- [Repowise](https://github.com/repowise-dev/repowise) — freshness scoring, hotspot detection, co-change analysis
- [Aider repo-map](https://aider.chat/docs/repomap.html) — tree-sitter symbol ranking, token budget awareness
- [GitNexus](https://github.com/abhigyanpatwari/GitNexus) — blast radius analysis, pre-commit staleness hooks

See [Section 14 of the architecture doc](docs/architecture.md) for the full research basis.

## Related Projects

- [prd-agents-framework](https://github.com/vshidlovsky/prd-agents-framework) — sibling framework for multi-agent PRD creation. The PRD researcher agent consumes the KB this framework produces.

## License

MIT
