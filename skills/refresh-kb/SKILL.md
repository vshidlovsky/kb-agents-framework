---
name: refresh-kb
description: Incremental KB refresh pipeline — detects stale docs, regenerates only what changed, checks for new conventions and custom packs, validates refreshed docs. Human gates between phases.
argument-hint: [doc-type]
---

# KB Refresh Pipeline

Incrementally refresh the project knowledge base. Optional `{argument}` targets a specific doc type (e.g., `api-registry`).

## Pre-flight

1. Verify `.claude/kb-context.md` exists. If not, STOP: "No KB configuration found. Run `/generate-kb` first."

2. Verify `docs/project-kb/.manifest.json` exists and has at least one document entry. If not, STOP: "No KB has been generated yet. Run `/generate-kb` first."

3. Read `.claude/kb-context.md` — extract enabled packs, scoping rules, conventions reference.

4. Read `docs/project-kb/.manifest.json` — extract current document state.

5. If `{argument}` is provided and matches a doc type name, narrow scope to only that doc type.

## Phase 1: Change Detection

Spawn an Agent using `.claude/agents/kb-refresher.md`:
- Let it analyze git changes since last generation
- Let it compute freshness scores and hotspot analysis
- Let it produce the refresh plan

The refresher will update the manifest with new freshness scores and insert staleness warnings into docs below 0.8 freshness.

Present the refresh plan to the user:

```
## KB Refresh Analysis

**Changes since last generation**: {N} source files changed
**Baseline commit**: {sha} ({date})
**Current commit**: {sha}

### Docs Needing Refresh

| Doc | Freshness | Strategy | Changed Files | Hotspot |
|-----|-----------|----------|---------------|---------|
| {name} | {score} | {patch/regenerate} | {N} | {yes/no} |

### Docs Current (no action needed)

| Doc | Freshness |
|-----|-----------|
| {name} | {score} |

{If argument was provided}: Narrowing to: {argument}
```

### Gate 1: User confirms which docs to refresh

Ask: **"Which docs should I refresh? Options: 'all' to refresh everything marked above, list specific doc names, or 'skip' to skip refresh and just update freshness scores."**

If "skip": update manifest freshness scores, insert staleness warnings, and go to Completion.

## Phase 2: Regeneration

For each doc the user confirmed for refresh, spawn the kb-generator agent:

```
Spawn Agent using `.claude/agents/kb-generator.md`:
  - {doc_type} = {doc type name}
  - {sequence_number} = {existing sequence number from manifest}
  - {is_custom} = {true if custom pack}
```

**For targeted patches** (freshness ≥ 0.90, strategy = patch):
- Instead of full regeneration, tell the generator to read the existing doc and only update the rows/sections affected by the changed files
- Pass the list of changed files from the refresh plan so the generator knows what to focus on

**For full regeneration** (freshness < 0.90):
- Standard full generation — the generator replaces the doc entirely

Present summary after regeneration:

```
## Regeneration Complete

| Doc | Strategy | Lines Before | Lines After | Change |
|-----|----------|-------------|-------------|--------|
| {name} | {patch/regen} | {old} | {new} | {+/-N} |
```

### Gate 2: User reviews refreshed docs

Ask: **"Review the refreshed docs. Say 'continue' to proceed to convention check, or provide feedback on specific docs."**

If feedback is given, regenerate the affected doc(s). Repeat until "continue."

## Phase 3: Convention Check + Pack Discovery

### Convention Check

If `project-conventions` is enabled and the conventions doc exists:

1. Read `docs/project-kb/{NN}-project-conventions.md` — extract existing confirmed conventions
2. Look at the changed files from the refresh plan
3. Scan for new naming patterns in the changed code that don't match any existing convention
4. If new patterns found with ≥10 examples and ≥80% consistency:
   - Present each candidate individually:
     ```
     New pattern detected in recently changed files:
     I found log messages follow "[ClassName] message" format.
     Examples: [WalletManager] balance fetched, [AuthService] token expired.
     Confidence: 87% (13 of 15 log calls match).
     This is not covered by any existing convention.
     Confirm / edit / skip?
     ```
5. Append confirmed conventions to the conventions doc (continue numbering: C-005, C-006, etc.)

### Pack Discovery

Check if the changed files reveal project structures that might warrant a new custom pack:

1. Look at the directories containing changed files
2. Check if any directory patterns suggest a structure not covered by existing docs
   - New CI/CD configs added? → propose `build-pipeline` pack
   - New protobuf/GraphQL schemas? → propose `graphql-registry` or `protobuf-registry` pack
   - New scheduled jobs? → propose `scheduled-jobs` pack
3. If a new pack opportunity is found, present it with rationale

### Gate 3: User confirms new conventions + any new custom packs

If no new conventions or packs were found: skip this gate, tell the user "No new conventions or custom pack opportunities detected" and proceed.

If new conventions were proposed: wait for user to confirm/edit/skip each one.

If new custom packs were proposed: ask user to confirm. If confirmed:
1. Write the custom pack definition to `docs/kb-doc-templates/custom/{pack-name}.md`
2. Add it to `kb-context.md` under Custom Doc Type Packs
3. Run the generator for the new pack
4. Present the generated doc for review

## Phase 4: Validation

Spawn an Agent using `.claude/agents/kb-validator.md`:
- Validate all refreshed docs (not the entire KB — only what was regenerated)
- `{docs_to_validate}` = comma-separated list of refreshed doc types

Present the validation report:

```
## Validation Report (Refreshed Docs Only)

| Doc | File Paths | Completeness | Line Budget | Cross-Refs | Verdict |
|-----|-----------|-------------|-------------|------------|---------|
| {name} | {PASS/FAIL} | {PASS/WARN/FAIL} | {PASS/WARN} | {PASS/WARN} | {verdict} |
```

### Gate 4: User confirms

Ask: **"Validation complete. Say 'done' to finalize, or 'fix' to regenerate failing docs."**

If "fix": regenerate failing docs, re-validate. Repeat until "done."

## Completion

1. Update `docs/project-kb/.manifest.json`:
   - Set `lastRefresh` to current timestamp
   - Update refreshed docs: new `commitSha`, `generatedAt`, `lineCount`, `tokenEstimate`, `freshness` = 1.0, `status` = "current"
   - Remove `staleSince` and `staleFiles` from refreshed docs
   - Remove staleness warning headers from refreshed docs

2. Present final summary:

```
## Refresh Complete

| Metric | Value |
|--------|-------|
| Docs refreshed | {N} |
| Docs unchanged | {M} |
| New conventions added | {C} |
| New custom packs | {P} |
| Validation | {passed/warnings/failures} |

**Next refresh**: Run `/refresh-kb` after your next round of code changes.
```

3. Commit:
   ```bash
   git add docs/project-kb/ .claude/kb-context.md
   git commit -m "docs(kb): refresh {list of refreshed doc names}"
   ```

   If custom packs were added:
   ```bash
   git add docs/kb-doc-templates/custom/
   ```

---

## Error Handling

- **No changes detected**: Report "KB is current — no source files changed since last generation" and exit cleanly.
- **Manifest missing docs**: If a doc is enabled in kb-context.md but missing from the manifest, treat it as needing full generation (not refresh).
- **Generator fails**: Report the error, skip that doc, continue with others.
- **Validator not built**: Skip Phase 4 with a warning.
- **Argument doesn't match**: If `{argument}` doesn't match any known doc type, list available doc types and ask the user to choose.
