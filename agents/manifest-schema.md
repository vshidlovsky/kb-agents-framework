# Manifest Schema

Canonical reference for the `.manifest.json` file structure. All agents that read or write the manifest must conform to this schema.

The manifest lives at `docs/project-kb/.manifest.json` in the target project.

---

## Schema (v2)

```json
{
  "version": 2,
  "projectName": "string — project name from kb-context.md",
  "lastFullGeneration": "ISO 8601 timestamp | null",
  "lastRefresh": "ISO 8601 timestamp | null",
  "documents": {
    "{doc-type}": {
      "file": "string — output filename, e.g. '01-repo-map.md'",
      "generatedAt": "ISO 8601 timestamp",
      "commitSha": "string — short git SHA at generation time",
      "sourceFileCount": "number — files scanned during extraction",
      "lineCount": "number — lines in generated doc",
      "tokenEstimate": "number — lineCount × 8",
      "freshness": "number — 0.0 to 1.0, starts at 1.0",
      "loadPriority": "hot | warm | cold",
      "status": "current | stale | regenerating",
      "scopeGlobs": ["array of glob patterns from pack definition"],
      "staleSince": "ISO 8601 timestamp | null — set when freshness drops below threshold",
      "staleFiles": ["array of changed file paths that triggered staleness | null"]
    }
  }
}
```

## Field Details

### Top-level

| Field | Required | Set By | Description |
|-------|----------|--------|-------------|
| `version` | yes | kb-setup | Schema version. Always `2`. |
| `projectName` | yes | kb-setup | Project name from kb-context.md. |
| `lastFullGeneration` | yes | generate-kb | Timestamp of last full `/generate-kb` run. Null until first run. |
| `lastRefresh` | yes | refresh-kb | Timestamp of last `/refresh-kb` run. Null until first refresh. |
| `documents` | yes | all agents | Map of doc type name → document metadata. |

### Per-document

| Field | Required | Set By | Description |
|-------|----------|--------|-------------|
| `file` | yes | kb-generator | Output filename with sequence prefix (e.g., `05-screen-inventory.md`). |
| `generatedAt` | yes | kb-generator | When this doc was last generated or regenerated. |
| `commitSha` | yes | kb-generator | Git short SHA at generation time. Used for diff-based freshness. |
| `sourceFileCount` | yes | kb-generator | How many source files were scanned. |
| `lineCount` | yes | kb-generator | Lines in the generated doc. Used for budget checks. |
| `tokenEstimate` | yes | kb-generator | Approximate token count (`lineCount × 8`). Used for context window budgeting. |
| `freshness` | yes | kb-refresher | Continuous score 0.0–1.0. Starts at 1.0, decays based on source file changes. |
| `loadPriority` | yes | kb-generator | `hot` (always load), `warm` (load when relevant), `cold` (load on demand). |
| `status` | yes | kb-refresher | `current` (freshness >= 0.8), `stale` (freshness < 0.8), `regenerating` (in progress). |
| `scopeGlobs` | yes | kb-generator | Glob patterns defining which source files this doc covers. Used by refresher for change detection. |
| `staleSince` | no | kb-refresher | When this doc first became stale. Null if current. |
| `staleFiles` | no | kb-refresher | Source files that changed since last generation, triggering staleness. Null if current. |

## Freshness Scoring

The refresher computes freshness using:

```
freshness = 1.0 - (changed_scope_files / total_scope_files)
```

Where:
- `changed_scope_files` = files matching `scopeGlobs` that changed since `commitSha`
- `total_scope_files` = all files matching `scopeGlobs`

Thresholds:
- `>= 0.8` → status `current`, no action needed
- `0.5 – 0.79` → status `stale`, patch update (targeted edits)
- `< 0.5` → status `stale`, full regeneration

## Example

```json
{
  "version": 2,
  "projectName": "boss-money",
  "lastFullGeneration": "2026-05-13T10:30:00Z",
  "lastRefresh": "2026-05-13T14:15:00Z",
  "documents": {
    "repo-map": {
      "file": "01-repo-map.md",
      "generatedAt": "2026-05-13T10:30:00Z",
      "commitSha": "a1b2c3d",
      "sourceFileCount": 47,
      "lineCount": 342,
      "tokenEstimate": 2736,
      "freshness": 0.95,
      "loadPriority": "hot",
      "status": "current",
      "scopeGlobs": ["**/pubspec.yaml", "**/package.json", "melos.yaml"],
      "staleSince": null,
      "staleFiles": null
    },
    "screen-inventory": {
      "file": "05-screen-inventory.md",
      "generatedAt": "2026-05-13T10:45:00Z",
      "commitSha": "a1b2c3d",
      "sourceFileCount": 128,
      "lineCount": 510,
      "tokenEstimate": 4080,
      "freshness": 0.62,
      "loadPriority": "warm",
      "status": "stale",
      "scopeGlobs": ["**/pages/**", "**/screens/**", "**/views/**", "**/router*"],
      "staleSince": "2026-05-13T12:00:00Z",
      "staleFiles": ["packages/money_transfer/lib/pages/new_recipient_page.dart"]
    }
  }
}
```
