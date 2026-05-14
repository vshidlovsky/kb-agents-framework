# Env Config Output Template

> Environment variables, config sources, per-environment overrides, and secret references.
> The KB Generator reads this template and fills it. Delete the `> TEMPLATE` blocks when generating.

---

```markdown
# Environment Config

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: cold. Load when touching: env, config, environment, settings.

## Overview

> TEMPLATE: One-line summary.

{N} environment variables, {M} secrets, {K} environments. Config source: {.env files / config classes / environment modules / etc.}.

## Variables by Category

> TEMPLATE: Group variables by category (API, Database, Auth, Feature, Logging, etc.).
> Order categories by variable count (descending).
> NEVER include actual secret values — only document that a secret exists.
> Default column shows the development/local default, or "—" if none.

### {Category}

| Variable | Source | Default | Per-Env Override | Consumed By | Secret |
|----------|--------|---------|-----------------|-------------|--------|
| `{API_BASE_URL}` | `{.env}` | `{http://localhost:3000}` | staging: `{value}`, prod: `{value}` | `{ApiClient}` | no |
| `{API_KEY}` | `{.env}` | — | all envs | `{ApiClient}` | **yes** |

---

### {Next Category}

| Variable | Source | Default | Per-Env Override | Consumed By | Secret |
|----------|--------|---------|-----------------|-------------|--------|
| ... | ... | ... | ... | ... | ... |

---

{Repeat for each category}

## Config Files

> TEMPLATE: All config/environment files in the project.

| File | Environment | Variables | Notes |
|------|------------|-----------|-------|
| `{.env}` | {local/default} | {count} | {committed / gitignored} |
| `{.env.example}` | {template} | {count} | {committed — lists required vars without values} |
| `{.env.staging}` | {staging} | {count} | {committed / gitignored} |

## Config Loading

> TEMPLATE: How environment config is loaded into the application.
> Skip this section if it's just standard dotenv loading.

| Mechanism | Config Class/File | Priority |
|-----------|------------------|----------|
| `{dotenv / Spring profiles / Django settings / custom}` | `{path/to/config_loader}` | {env file → system env → defaults} |

## Secrets Summary

> TEMPLATE: List of all secret variables (no values!). Documents which secrets
> need to be provisioned for each environment.
> Skip this section if no secrets detected.

| Secret | Required Environments | Provider | Notes |
|--------|----------------------|----------|-------|
| `{DB_PASSWORD}` | {all} | {env var / AWS Secrets Manager / Vault} | |
| `{STRIPE_SECRET_KEY}` | {staging, prod} | {env var} | {not needed in local — uses test mode} |

## See Also

- [App Profiles](02-apps/) — per-app environment setup
- [Service Map]({NN}-service-map.md) — which services consume which variables
- [Database Schema]({NN}-database-schema.md) — database connection config variables
```
