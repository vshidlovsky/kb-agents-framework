# Database Schema Output Template

> Tables, relationships, migrations, ORM models. The KB Generator reads this template and fills it.
> Delete the `> TEMPLATE` blocks when generating.

---

```markdown
# Database Schema Map

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: cold. Load when touching: migrations, models, entities, schema.

## Overview

> TEMPLATE: One-line summary.

{N} tables/collections, {M} migrations, ORM: {ActiveRecord / Prisma / SQLAlchemy / Hibernate / TypeORM / etc.} (`{model directory}`).

## Tables

> TEMPLATE: One section per table/collection. Order by importance (core domain tables first,
> then supporting tables, then join tables).
> Don't reproduce every column — focus on PKs, FKs, notable constraints, and non-obvious columns.

### {table_name}

- **ORM model**: `{path/to/model_file}`
- **Created by migration**: `{migration_file_name}` (`{path}`)

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `{column}` | `{type}` | {PK / FK → table.col / NOT NULL / UNIQUE / INDEX} | {only if non-obvious} |

---

### {next_table}

- **ORM model**: `{path}`
- **Created by migration**: `{migration}` (`{path}`)

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| ... | ... | ... | ... |

---

{Repeat for each table}

## Relationships

> TEMPLATE: Entity relationships derived from foreign keys and ORM associations.

| From | To | Type | FK Column | ORM Declaration |
|------|----|------|-----------|-----------------|
| `{table_a}` | `{table_b}` | {1:N / N:1 / N:M / 1:1} | `{fk_column}` | `{has_many / belongs_to / etc.}` |

## Migration History

> TEMPLATE: Migrations in chronological order. Only include migrations that made
> structural changes (new tables, column additions, index changes).
> Skip data-only migrations.

| # | Migration | Date | Change |
|---|-----------|------|--------|
| 1 | `{migration_name}` | {date or "from filename"} | {one-line summary — e.g., "create users table"} |
| 2 | `{migration_name}` | {date} | {summary} |

**Migration directory**: `{path}`
**Total migrations**: {N}

## See Also

- [API Registry]({NN}-api-registry.md) — which endpoints read/write these tables
- [Service Map]({NN}-service-map.md) — which services own which tables (if generated)
```
