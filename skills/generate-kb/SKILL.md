---
name: generate-kb
description: Full knowledge base generation pipeline — discovers project structure, recommends doc type packs, generates KB documents tier by tier with human gates between phases. Chains kb-setup → kb-generator → kb-validator agents.
argument-hint: [scope]
---

# KB Generation Pipeline

Generate a project knowledge base. Optional `{argument}` narrows scope (e.g., a specific app name or doc type).

## Pre-flight

1. Check if `.claude/kb-context.md` exists.
   - **If it does NOT exist**: run setup first. Spawn an Agent using the prompt from `.claude/agents/kb-setup.md`. After setup completes, continue to Phase 0.
   - **If it exists**: read it and extract project config, enabled packs, scoping rules.

2. Verify `docs/project-kb/` directory exists. If not, create it.
3. Read `docs/project-kb/.manifest.json` if it exists — note what's already been generated.
4. Read `agents/doc-type-registry.md` — you need this to know extraction strategies.
5. Read `agents/quality-patterns.md` — you need this to enforce quality.

## Phase 0: Discovery & Scope

If `kb-context.md` was just created by setup, skip to scope confirmation — packs were already chosen during setup.

If `kb-context.md` already existed, present the current pack configuration:

```
## Current KB Configuration

Enabled packs:
{list enabled packs from kb-context.md}

Scope:
{list apps/services in scope}

{If argument was provided}: Narrowing to: {argument}
```

Ask: **"Ready to generate with this configuration? Say 'continue' to proceed, or tell me what to change."**

If the user wants changes, update `kb-context.md` accordingly.

### Gate 0: User confirms scope

Wait for user confirmation before proceeding.

## Phase 1: Tier 1 Generation (Universal)

Generate the foundational docs that all other tiers depend on. Spawn the kb-generator agent for each enabled Tier 1 pack.

**Sequence matters** — generate in this order because later docs reference earlier ones:

1. **repo-map** (sequence: `01`)
   Spawn Agent using `.claude/agents/kb-generator.md`:
   - `{doc_type}` = `repo-map`
   - `{sequence_number}` = `01`
   - `{is_custom}` = `false`

   After each doc generation, verify `docs/project-kb/.manifest.json` was updated with the new document entry. If not, read the generated doc's line count and update the manifest manually.

2. **app-profiles** (sequence: `02`)
   Spawn Agent using `.claude/agents/kb-generator.md`:
   - `{doc_type}` = `app-profiles`
   - `{sequence_number}` = `02`
   - `{is_custom}` = `false`

   After each doc generation, verify `docs/project-kb/.manifest.json` was updated with the new document entry. If not, read the generated doc's line count and update the manifest manually.

3. **shared-code** (sequence: `03`)
   Spawn Agent using `.claude/agents/kb-generator.md`:
   - `{doc_type}` = `shared-code`
   - `{sequence_number}` = `03`
   - `{is_custom}` = `false`

   After each doc generation, verify `docs/project-kb/.manifest.json` was updated with the new document entry. If not, read the generated doc's line count and update the manifest manually.

4. **gotchas** (sequence: `04`)
   Spawn Agent using `.claude/agents/kb-generator.md`:
   - `{doc_type}` = `gotchas`
   - `{sequence_number}` = `04`
   - `{is_custom}` = `false`

   After each doc generation, verify `docs/project-kb/.manifest.json` was updated with the new document entry. If not, read the generated doc's line count and update the manifest manually.

After all Tier 1 docs are generated, present a summary:

```
## Tier 1 Generation Complete

| Doc | File | Lines | Tokens (~) |
|-----|------|-------|------------|
| Repo Map | docs/project-kb/01-repo-map.md | {N} | {N×8} |
| App Profiles | docs/project-kb/02-apps/{name}.md | {N} | {N×8} |
| Shared Code | docs/project-kb/03-shared-code.md | {N} | {N×8} |
| Gotchas | docs/project-kb/04-gotchas.md | {N} | {N×8} |

These are the factual foundation. Review for accuracy before I generate project-specific docs.
```

### Gate 1: User reviews Tier 1 docs

Ask: **"Review the Tier 1 docs above. These are the foundation — later docs build on them. Say 'continue' to proceed, or provide feedback."**

If feedback is given, regenerate the affected doc(s). Repeat until user says "continue."

## Phase 2: Tier 2/3 + Custom Pack Generation (Project-Specific)

Generate all enabled Tier 2, Tier 3, and custom packs. These use Tier 1 outputs as input.

Assign sequence numbers continuing from Tier 1:
- Start at `05` and increment for each enabled pack

For each enabled Tier 2/3 pack and each custom pack, spawn the kb-generator agent:

```
Spawn Agent using `.claude/agents/kb-generator.md`:
  - {doc_type} = {pack name}
  - {sequence_number} = {next number}
  - {is_custom} = {true if custom pack, false if built-in}
```

After each doc generation, verify `docs/project-kb/.manifest.json` was updated with the new document entry. If not, read the generated doc's line count and update the manifest manually.

**Parallelization note**: Tier 2/3 packs within the same tier can be generated in parallel — they don't depend on each other. Custom packs may depend on built-in outputs, so generate them after built-in Tier 2/3 packs.

After all are generated, present summary:

```
## Tier 2/3 + Custom Generation Complete

| Doc | File | Lines | Tokens (~) |
|-----|------|-------|------------|
| {name} | {path} | {N} | {N×8} |
| ... | ... | ... | ... |
```

### Gate 2: User reviews Tier 2/3 + custom docs

Ask: **"Review the project-specific docs above. Say 'continue' to proceed to cross-cutting docs, or provide feedback."**

If feedback is given, regenerate affected doc(s). Repeat until user says "continue."

## Phase 3: Tier 4 Generation (Cross-Cutting, except conventions)

Generate enabled Tier 4 packs **except** project-conventions (which has its own phase).

For each enabled Tier 4 pack (feature-flags, l10n-registry, env-config), spawn the kb-generator agent with the next sequence number.

After each doc generation, verify `docs/project-kb/.manifest.json` was updated with the new document entry. If not, read the generated doc's line count and update the manifest manually.

Present summary after generation.

### Gate 3: User reviews Tier 4 docs

Ask: **"Review the cross-cutting docs. Say 'continue' to proceed to convention discovery, or provide feedback."**

## Phase 4: Convention Discovery

If project-conventions is enabled in kb-context.md, run convention discovery.

Spawn Agent using `.claude/agents/kb-generator.md`:
- `{doc_type}` = `project-conventions`
- `{sequence_number}` = next number
- `{is_custom}` = `false`

The generator agent handles the human-confirmation flow for conventions — it presents each candidate convention individually and waits for user confirmation.

**Important**: Do NOT proceed past this phase until the user has confirmed or skipped each proposed convention. Conventions that are skipped do not enter the KB.

### Gate 4: User confirms conventions

The generator handles this gate internally. When it's done, it will have written `docs/project-kb/{NN}-project-conventions.md` with only confirmed conventions.

## Phase 5: Validation

Spawn an Agent using `.claude/agents/kb-validator.md` (if the file exists — skip this phase if the validator agent hasn't been built yet):
- Validate all generated docs
- Run all checks from `agents/quality-patterns.md`

Present the validation report:

```
## Validation Report

| Doc | File Paths | Routes | APIs | Completeness | Line Budget | Cross-Refs |
|-----|-----------|--------|------|-------------|-------------|------------|
| Repo Map | ✅ PASS | — | — | ✅ PASS | ✅ 342 lines | ✅ PASS |
| API Registry | ✅ PASS | — | ⚠ 2 stale | ✅ PASS | ⚠ 1140 lines | ✅ PASS |
| ... | ... | ... | ... | ... | ... | ... |

{N} docs validated. {P} passed. {W} warnings. {F} failures.
```

### Gate 5: User reviews validation

Ask: **"Review the validation report. Failures indicate stale references or missing data. Say 'continue' to proceed, 'fix' to regenerate failing docs, or provide specific feedback."**

If "fix" is chosen, regenerate failing docs and re-validate.

## Phase 6: Cross-Reference Pass

This phase is MANDATORY — do not skip it. Unresolved `{NN}-` placeholders in generated docs break markdown links and confuse agents.

Resolve all `{NN}-` placeholder links and verify cross-references.

### Step 1: Build File Map

Read `docs/project-kb/.manifest.json`. Build a lookup table mapping doc type names to their actual filenames:

```
repo-map       → 01-repo-map.md
app-profiles   → 02-apps/
shared-code    → 03-shared-code.md
gotchas        → 04-gotchas.md
...
```

### Step 2: Resolve Placeholders

For each generated doc in `docs/project-kb/` (including detail files in subdirectories):

1. Read the file
2. Find all `{NN}-` placeholder patterns in "See Also" sections and anywhere else in the doc
3. Replace each `{NN}-{doc-name}.md` with the actual filename from the file map
4. Replace each `{NN}-apps/` with the actual apps directory prefix
5. For split docs: also resolve placeholders in all detail files within `{NN}-{doc-name}/` subdirectories. Detail files may reference other doc types in their back-links.
6. If a referenced doc wasn't generated (not in manifest), **remove the See Also line** and add a comment: `<!-- {doc-type} not generated -->`
7. Write the updated file

### Step 3: Verify Bidirectional Links

For each cross-reference A → B, check that B → A also exists. Add missing backlinks.

### Step 4: Present Summary

```
## Cross-Reference Pass

{N} placeholder links resolved.
{M} missing backlinks added.
{K} dangling references removed (doc not generated).
{J} docs updated.
```

### Step 5: Verify No Unresolved Placeholders

After resolution, grep all docs in `docs/project-kb/` for remaining `{NN}-` patterns. If any remain, resolve them or remove the broken link. Report count of unresolved placeholders.

### Gate 6: User confirms final KB

Present the final KB summary:

```
## Knowledge Base Complete

| # | Doc | File | Lines (index) | Lines (total) | Tokens (index) | Priority | Freshness |
|---|-----|------|---------------|---------------|----------------|----------|-----------|
| 01 | Repo Map | 01-repo-map.md | 342 | 342 | 2,736 | hot | 1.0 |
| 02 | App: {name} | 02-apps/{name}.md | 200 | 200 | 1,600 | hot | 1.0 |
| 07 | Dependency Index | 07-dependency-index.md | 65 | 1,265 (5 files) | 520 | warm | 1.0 |
| ... | ... | ... | ... | ... | ... | ... | ... |

**Total**: {N} docs, {L} index lines (agents load), {T_total} total lines, ~{T} tokens
**Validation**: {P} passed, {W} warnings, {F} failures
**Conventions**: {C} confirmed

Output: docs/project-kb/
Manifest: docs/project-kb/.manifest.json
```

## Completion

After user confirms:

1. Update `.manifest.json` with `lastFullGeneration` timestamp
2. If PRD framework is installed (per kb-context.md Integration section):
   - Update `.claude/project-context.md` to set `Knowledge base` path to `docs/project-kb/`
   - Tell the user: "Updated project-context.md so the PRD researcher will use this KB."

3. Commit all generated docs:
   ```bash
   git add docs/project-kb/
   git commit -m "docs(kb): generate full knowledge base ({N} docs, {L} lines)"
   ```

4. Tell the user: **"Knowledge base generated. Run `/refresh-kb` anytime to update stale docs after code changes."**

---

## Error Handling

- **Generator agent fails**: Report the error, skip that doc type, continue with others. Present the failure in the summary.
- **Validator not built yet**: Skip Phase 5, warn: "Validator agent not yet implemented — skipping validation. Run manually later."
- **No docs to generate**: If no packs are enabled, STOP: "No doc type packs are enabled in kb-context.md. Edit the file to enable packs, or run kb-setup again."
- **Scope argument**: If `{argument}` matches a specific doc type name, only generate that one doc (skip tier structure). If it matches an app name, only generate app-profile for that app.
