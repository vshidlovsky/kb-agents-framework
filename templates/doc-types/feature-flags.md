# Feature Flags Output Template

> Feature flag definitions, provider keys, defaults, and consumption points. The KB Generator
> reads this template and fills it. Delete the `> TEMPLATE` blocks when generating.

---

```markdown
# Feature Flags

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: cold. Load when touching: feature_flags, remote_config, flags, config.

## Overview

> TEMPLATE: One-line summary.

{N} feature flags across {M} domains. Provider: {LaunchDarkly / Firebase Remote Config / Unleash / Flagsmith / custom enum / etc.} (`{config path}`).

## Flags by Domain

> TEMPLATE: Group flags by feature area or domain. Order domains by flag count (descending).
> For each flag: name is the code-side constant, provider key is the remote config key,
> default is what the code uses when the flag is unavailable.
> "What It Gates" should be a one-sentence description of the behavior change.
> "Consumed By" lists screens/classes that check this flag.

### {Domain / Feature Area}

| Flag Name | Provider Key | Default | What It Gates | Consumed By |
|-----------|-------------|---------|---------------|-------------|
| `{ENABLE_FEATURE}` | `{enable_feature}` | `{false}` | {one sentence} | `{Screen1}`, `{Screen2}` |

---

### {Next Domain}

| Flag Name | Provider Key | Default | What It Gates | Consumed By |
|-----------|-------------|---------|---------------|-------------|
| ... | ... | ... | ... | ... |

---

{Repeat for each domain}

## Flag Summary

> TEMPLATE: Flat table for quick lookup, sorted alphabetically.

| Flag Name | Provider Key | Default | Domain | Consumers |
|-----------|-------------|---------|--------|-----------|
| `{ENABLE_FEATURE}` | `{enable_feature}` | `{false}` | {domain} | {count} |

## Flag Definitions

> TEMPLATE: Where flags are defined in code. One row per definition file.
> This helps agents find where to add new flags.

| File | Type | Flag Count |
|------|------|-----------|
| `{path/to/flags_enum}` | {enum / config class / JSON} | {count} |

## Stale Flags

> TEMPLATE: Flags that appear to be permanently enabled or disabled (no runtime variation).
> These are candidates for cleanup. Skip this section if none detected.
>
> Detection heuristic: flag default is the only value ever used, or flag is checked
> but the condition branch is empty/unreachable.

| Flag Name | Evidence | Suggested Action |
|-----------|----------|-----------------|
| `{FLAG_NAME}` | {always true in all envs / dead else-branch} | {remove flag, hardcode true} |

## See Also

- [App Profiles]({NN}-apps/) — per-app flag provider setup
- [Screen Inventory]({NN}-screen-inventory.md) — screens referenced in Consumed By column
- [Project Conventions]({NN}-project-conventions.md) — flag naming convention (if documented)
```
