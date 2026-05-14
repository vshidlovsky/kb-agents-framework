# API Registry Output Template

> Full HTTP contract extraction. The KB Generator reads this template and fills it.
> Delete the `> TEMPLATE` blocks when generating.

---

```markdown
# API Registry

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: services, api, clients, controllers, handlers.

## Overview

> TEMPLATE: One-line summary.

{N} endpoints across {M} services. {K} DTOs.

## Endpoints by Service

> TEMPLATE: One table per service/client class. Group endpoints by the service that defines them.
> Each row is one HTTP endpoint.
> Sort services by endpoint count (descending).
>
> For frontend/mobile projects: services are HTTP client classes that call backend APIs.
> For backend projects: services are controllers/handlers that expose APIs.

### {Service Name} (`{service_file_path}`)

| Method | Path | Service Method | Repository | Consumer | Request DTO | Response DTO |
|--------|------|----------------|------------|----------|-------------|--------------|
| {GET/POST/PUT/DELETE/PATCH} | `{/v1/resource}` | `{methodName()}` | `{RepoClass}` | `{ConsumerClass}` | `{RequestDto}` | `{ResponseDto}` |

**Base URL**: {base URL config reference or "per-environment — see env-config"}

#### DTO Models

> TEMPLATE: Document request/response DTO shapes for endpoints in this service.
> Only include DTOs specific to this service. Skip generic wrappers (ApiResponse<T>, Result<T, E>) — note the inner type.
> One field table per DTO. Keep it compact — agents need field names and types, not full docs.

**{RequestDto}** (`{dto_file_path}`)

| Field | Type | Notes |
|-------|------|-------|
| `{fieldName}` | `{type}` | {constraints or enum values, if notable} |

**{ResponseDto}** (`{dto_file_path}`)

| Field | Type | Notes |
|-------|------|-------|
| `{fieldName}` | `{type}` | |

---

### {Next Service} (`{path}`)

| Method | Path | Service Method | Repository | Consumer | Request DTO | Response DTO |
|--------|------|----------------|------------|----------|-------------|--------------|
| ... | ... | ... | ... | ... | ... | ... |

#### DTO Models

{Same pattern as above}

---

{Repeat for each service}

## Authentication

> TEMPLATE: How API authentication works. One line per auth mechanism.
> Skip this section if no auth is detected.

| Mechanism | Header/Token | Config |
|-----------|-------------|--------|
| {Bearer JWT / API key / OAuth2 / session} | `{header name}` | `{config file or class}` |

## Error Handling

> TEMPLATE: Common error response patterns across services.
> Skip this section if no consistent pattern is detected.

| HTTP Status | Error Shape | Example |
|-------------|------------|---------|
| {400/401/404/500} | `{error response DTO or shape}` | `{brief example}` |

## See Also

- [Dependency Index]({NN}-dependency-index.md) — which screens consume these services
- [App Profiles]({NN}-apps/) — per-app API client setup
- [Database Schema]({NN}-database-schema.md) — tables behind these endpoints (if generated)
- [Service Map]({NN}-service-map.md) — inter-service dependencies (if generated)
```
