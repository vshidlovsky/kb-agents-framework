# API Registry Output Template

> Full HTTP contract extraction. The KB Generator reads this template and fills it.
> For large projects (500+ lines), use the split-file layout: summary index + per-domain detail files.
> Delete the `> TEMPLATE` blocks when generating.

---

## Split Layout: Index File Template

> Use when combined output exceeds 500 lines.
> Target: 50-100 lines. Contains shared infrastructure (HTTP stack, auth, errors) + service lookup table.

```markdown
# API Registry

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: services, api, clients, controllers, handlers.
> Split doc: detail files in `{NN}-api-registry/`. Load the one matching your working domain.

## Overview

{N} endpoints across {M} services. {K} DTOs. {J} service domains.

## HTTP Stack Architecture

> TEMPLATE: How requests flow from service class through middleware/interceptors to the network.
> This is shared across all services — keep it in the index so every agent gets it.

{HTTP client classes, interceptor chain, retry policies, base URL config}

## Authentication

| Mechanism | Header/Token | Config |
|-----------|-------------|--------|
| {Bearer JWT / API key / OAuth2 / session} | `{header name}` | `{config file or class}` |

## Error Handling

| HTTP Status | Error Shape | Example |
|-------------|------------|---------|
| {400/401/404/500} | `{error response DTO or shape}` | `{brief example}` |

## Services by Domain

> TEMPLATE: Lookup table — agents find which detail file has the service they need.

| Service | Package | Endpoints | Detail |
|---------|---------|-----------|--------|
| `{ServiceName}` | `{package}` | {count} | [{domain}.md]({NN}-api-registry/{domain}.md) |

## See Also

- [Dependency Index]({NN}-dependency-index.md) — which screens consume these services
- [App Profiles]({NN}-apps/) — per-app API client setup
```

## Split Layout: Detail File Template

> One file per service domain. Target: 100-300 lines.
> Agent loads this when working with services in the matching domain.

```markdown
# API Registry: {Domain Name}

> Part of [{NN}-api-registry.md](../{NN}-api-registry.md). Generated {date} at commit {short_sha}.
> Load when working with {domain} services or their consumers.

## Endpoints

> TEMPLATE: One table per service class. Same 8-column format as single-file output.

### {Service Name} (`{service_file_path}`)

| Method | Path | Service Method | Repository | Consumer | Request DTO | Response DTO |
|--------|------|----------------|------------|----------|-------------|--------------|
| {GET/POST/...} | `{/v1/resource}` | `{methodName()}` | `{RepoClass}` | `{ConsumerClass}` | `{RequestDto}` | `{ResponseDto}` |

#### DTO Models

> TEMPLATE: Field tables for DTOs specific to this service. Skip generic wrappers.

**{RequestDto}** (`{dto_file_path}`)

| Field | Type | Notes |
|-------|------|-------|
| `{fieldName}` | `{type}` | {constraints, if notable} |

**{ResponseDto}** (`{dto_file_path}`)

| Field | Type | Notes |
|-------|------|-------|
| `{fieldName}` | `{type}` | |

---

{Repeat for each service in this domain}
```

## Single-File Template

> Use when combined output is under 500 lines.

```markdown
# API Registry

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: services, api, clients, controllers, handlers.

## Overview

{N} endpoints across {M} services. {K} DTOs.

## HTTP Stack Architecture

{HTTP client classes, interceptor chain, retry policies}

## Endpoints by Service

> TEMPLATE: One table per service class, same 8-column format.

### {Service Name} (`{service_file_path}`)

| Method | Path | Service Method | Repository | Consumer | Request DTO | Response DTO |
|--------|------|----------------|------------|----------|-------------|--------------|
| ... | ... | ... | ... | ... | ... | ... |

#### DTO Models

{Same DTO format as detail file template}

---

{Repeat for each service}

## Authentication

| Mechanism | Header/Token | Config |
|-----------|-------------|--------|
| ... | ... | ... |

## Error Handling

| HTTP Status | Error Shape | Example |
|-------------|------------|---------|
| ... | ... | ... |

## See Also

- [Dependency Index]({NN}-dependency-index.md) — which screens consume these services
- [App Profiles]({NN}-apps/) — per-app API client setup
```
