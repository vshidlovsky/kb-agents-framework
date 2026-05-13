# KB Agents Framework

Multi-agent framework for generating and maintaining project knowledge bases that AI coding agents consume. Built for [Claude Code](https://claude.ai/code).

## What It Does

Scans your codebase and generates structured reference documents — repo maps, API registries, screen inventories, navigation graphs, feature flag registries, and more — that AI coding agents read before working on your project. Agents get the "lay of the land" from pre-indexed docs instead of grep-searching from scratch every time.

## Status

**Design phase.** See [docs/architecture.md](docs/architecture.md) for the full architecture design.

## Key Design Decisions

- **Curated over comprehensive** — agents generate draft docs, humans review before they become canonical. Research shows auto-generated verbose docs hurt agent performance.
- **Factual, not prescriptive** — documents what exists (packages, endpoints, screens, flags), not how code should be written. Conventions belong in a separate framework.
- **Incremental refresh** — tracks which source files each document depends on. Only stale docs are regenerated.
- **Citation-grounded** — every claim cites a real file path. The validator checks that every cited path exists.

## Related Projects

- [prd-agents-framework](https://github.com/user/prd-agents-framework) — sibling framework for multi-agent PRD creation. The PRD researcher agent consumes the KB this framework produces.

## License

MIT
