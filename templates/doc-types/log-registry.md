# Log Registry Output Template

> Inventory of structured logging across the project. The KB Generator reads this template and fills it.
> For large projects (500+ lines), use the split-file layout: summary index + per-domain detail files.
> Delete the `> TEMPLATE` blocks when generating.

---

## Split Layout: Index File Template

> Use when combined output exceeds 500 lines.
> Target: 50-100 lines. Contains logging infrastructure + domain lookup table.

```markdown
# Log Registry

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: logging, observability, error handling, alerting.
> Split doc: detail files in `{NN}-log-registry/`. Load the one matching your working domain.

## Overview

{N} log points across {M} domains. Logging framework: {framework name} (`{config_file}`).

## Logging Infrastructure

> TEMPLATE: How logs are emitted — framework setup, formatters, transports, correlation.

- **Framework**: `{name}` — configured in `{config_file}`
- **Log levels used**: `{DEBUG, INFO, WARN, ERROR, FATAL}`
- **Structured format**: {JSON / key-value / unstructured}
- **Correlation**: {correlation ID / trace ID / request ID — how it propagates}
- **Transports**: {stdout, file, ELK, Datadog, CloudWatch, etc.}
- **Alerting integration**: {which log patterns trigger alerts, PagerDuty/Slack/etc.}

## Severity Distribution

| Level | Count | Domains |
|-------|-------|---------|
| ERROR | {N} | {domain1, domain2} |
| WARN | {N} | {domain1, domain3} |
| INFO | {N} | {domain1, domain2, domain3} |
| DEBUG | {N} | {domain2} |

## Log Points by Domain

> TEMPLATE: Lookup table — agents find which detail file has the log points they need.

| Domain | Log Points | Primary Level | Detail |
|--------|-----------|---------------|--------|
| `{domain}` | {count} | {INFO/ERROR/etc.} | [{domain}.md]({NN}-log-registry/{domain}.md) |

## See Also

- [API Registry]({NN}-api-registry.md) — request/response logging per endpoint
- [Service Map]({NN}-service-map.md) — inter-service log correlation
- [Project Conventions]({NN}-project-conventions.md) — log message naming rules
```

## Split Layout: Detail File Template

> One file per domain. Target: 100-300 lines.
> Agent loads this when working with services in the matching domain.

```markdown
# Log Registry: {Domain Name}

> Part of [{NN}-log-registry.md](../{NN}-log-registry.md). Generated {date} at commit {short_sha}.
> Load when adding or modifying logging in the {domain} domain.

## Log Points

| Log Message / Pattern | Level | File Path | Structured Fields | Alert |
|----------------------|-------|-----------|-------------------|-------|
| `{message or pattern}` | `{ERROR}` | `{path/to/file:line}` | `{field1, field2}` | {yes/no — links to alert rule if yes} |

{Repeat for all log points in this domain}
```

## Single-File Template

> Use when combined output is under 500 lines.

```markdown
# Log Registry

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: warm. Load when touching: logging, observability, error handling, alerting.

## Overview

{N} log points across {M} domains. Logging framework: {framework name} (`{config_file}`).

## Logging Infrastructure

- **Framework**: `{name}` — configured in `{config_file}`
- **Log levels used**: `{list}`
- **Structured format**: {JSON / key-value / unstructured}
- **Correlation**: {correlation/trace/request ID propagation}
- **Transports**: {list}
- **Alerting integration**: {description}

## Severity Distribution

| Level | Count | Domains |
|-------|-------|---------|
| ERROR | {N} | {domains} |
| WARN | {N} | {domains} |
| INFO | {N} | {domains} |
| DEBUG | {N} | {domains} |

## Log Points by Domain

> TEMPLATE: One table per domain. Sort by log point count descending.

### {Domain Name} ({N} log points)

| Log Message / Pattern | Level | File Path | Structured Fields | Alert |
|----------------------|-------|-----------|-------------------|-------|
| `{message}` | `{level}` | `{path:line}` | `{fields}` | {yes/no} |

---

{Repeat for each domain}

## Unstructured Log Calls

> TEMPLATE: Log calls using print/console.log/debugPrint instead of the structured logger. Skip if none.

| Call | File Path | Suggested Migration |
|------|-----------|-------------------|
| `{print('...')}` | `{path:line}` | `{logger.info('...', {fields})}` |

## See Also

- [API Registry]({NN}-api-registry.md) — request/response logging per endpoint
- [Service Map]({NN}-service-map.md) — inter-service log correlation
- [Project Conventions]({NN}-project-conventions.md) — log message naming rules
```
