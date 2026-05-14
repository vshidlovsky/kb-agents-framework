# Repo Map Output Template

> This template defines the structure of the generated `01-repo-map.md` document.
> The KB Generator reads this template and fills it with extracted data.
> Delete the `> TEMPLATE` blocks when generating — they are instructions, not output.

---

```markdown
# Repo Map

> Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
> Load priority: hot. Load when touching: workspace configs, package manifests.

## Tooling

> TEMPLATE: One row per build tool, runtime, or workspace orchestrator.
> Only include tools with version-pinned configs — skip globally installed tools.

| Tool | Version | Config File |
|------|---------|-------------|
| {tool} | {version} | `{config_path}` |

## Package Inventory

> TEMPLATE: One row per package/module in the workspace.
> Sort by depended-on-by count (descending) so hub packages appear first.
> Description is one phrase — what the package provides, not how it works.

| Package | Path | Description | Depended-On-By |
|---------|------|-------------|----------------|
| `{name}` | `{path}` | {one-phrase description} | {count} |

### Hub Packages

> TEMPLATE: Call out packages depended on by >50% of other packages.
> These are the foundational packages — changes here ripple widely.

{List hub packages with their dependent count and a one-line note about what they provide.}

## Dependency Diagram

> TEMPLATE: Auto-generated Mermaid diagram from the adjacency list.
> Packages are nodes. Internal dependencies are directed edges.
> For readability, omit edges to hub packages if they would create a hairball —
> note "all packages depend on {hub}" instead.

```mermaid
graph TD
    {package_a} --> {package_b}
    {package_a} --> {package_c}
    {package_b} --> {package_d}
```

## See Also

- [App Profiles](02-apps/) — per-app entry points, state management, routing
- [Shared Code](03-shared-code.md) — detailed contents of shared packages listed above
- [Gotchas](04-gotchas.md) — anti-patterns and pitfalls found in this repo structure
```
