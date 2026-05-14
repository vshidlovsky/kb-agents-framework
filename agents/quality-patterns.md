# Quality Patterns

Reference file read by the KB Generator and KB Validator agents. Defines anti-patterns to avoid during generation and validation checks to run after generation.

---

## Anti-Patterns

The generator MUST avoid these. The validator flags them as failures.

### AP-1: Verbose dump

Reproducing file contents instead of summarizing structure. The KB is an index, not a mirror.

**Rule:** Max 3 lines of code citation per entry. If you need more, cite the file path and let the agent read it.

**Bad:**
```
The WalletManager class has the following methods:
  Future<Balance> getBalance() async {
    final response = await _apiClient.get('/wallet/balance');
    final data = json.decode(response.body);
    return Balance.fromJson(data);
  }
  // ... 50 more lines
```

**Good:**
```
| Class | Path | Key Methods | Consumers |
|-------|------|-------------|-----------|
| `WalletManager` | `packages/wallet/lib/src/wallet_manager.dart` | `getBalance()`, `sendMoney()`, `getHistory()` | 12 screens |
```

### AP-2: Ungrounded claim

Any statement not backed by a file path citation. Every fact in the KB must be verifiable.

**Bad:** "The app uses Provider for state management."
**Good:** "The app uses Provider for state management (`apps/boss_money/lib/main.dart:23` — `MultiProvider` wrapping `MaterialApp`)."

### AP-3: Stale reference

Citing a file that doesn't exist, a route that isn't registered, or a class that was renamed.

**Rule:** The validator runs `[ -f "{path}" ]` for every cited path. Stale references are hard failures.

### AP-4: Scope creep

Including information about packages, apps, or services not listed in `kb-context.md` Apps/Services in Scope.

**Rule:** Check the scope before writing. If a package isn't in scope, don't document it — even if it's interesting.

### AP-5: Coding convention leakage

Documenting HOW to structure code (state management patterns, DI registration approaches, file organization). KB docs document naming patterns and what exists, not implementation patterns.

**Boundary test:** If a PRD author needs it to name something correctly → belongs in KB (project convention). If a developer needs it to structure code correctly → belongs in coding conventions framework (separate).

**Bad:** "Always use Cubit for new features. Use ChangeNotifier only in money_transfer_v2 for legacy reasons."
**Good (in project-conventions):** "Feature flag names follow `SCREAMING_SNAKE_CASE` with `ENABLE_` prefix. Examples: `ENABLE_GOOGLE_PAY`, `ENABLE_DARK_MODE`."

### AP-6: Redundancy across docs

Repeating information in multiple docs. Use cross-references instead.

**Bad:** Listing all WalletManager consumers in both the dependency-index AND the shared-code doc.
**Good:** dependency-index has the full consumer list; shared-code says "See dependency-index for consumer list."

### AP-7: Missing cross-reference

A screen inventory entry without a nav graph link. An API registry entry without a dependency index link. If two docs cover related aspects of the same entity, they must link to each other.

### AP-8: Documenting the obvious

Including information agents can derive from a single file read or grep. KB value comes from relationships, reverse lookups, and cross-cutting concerns that are expensive to derive from source.

**Test:** If removing this entry wouldn't cost an agent more than one grep, it doesn't belong in the KB.

**Bad:** Listing every file in a package directory (agents can `ls`).
**Good:** Showing which 47 screens import WalletManager (agents can't cheaply grep + collate this).

---

## Validation Checks

The validator runs these checks on every generated or refreshed doc.

### VC-1: File path existence

Every cited file path must exist on disk.

```bash
while IFS= read -r path; do
  [ -f "$path" ] || echo "FAIL: $path does not exist"
done < <(grep -oP '`[^`]+\.(dart|ts|tsx|js|jsx|java|kt|go|py|rs|cs|yaml|json|xml|sql|md)`' "$doc")
```

**Result:** FAIL if any cited path doesn't exist. Indicates stale reference — needs regeneration.

### VC-2: Route accuracy

Every route string in screen-inventory or navigation-graph must exist in a route registration file.

**Method:** For each route string, grep in the routing files identified during extraction. Flag routes that appear in the KB but not in code.

**Result:** FAIL if >0 missing routes. May indicate renamed routes or deleted screens.

### VC-3: API endpoint accuracy

Every endpoint in api-registry must still exist in its cited service file.

**Method:** For each endpoint row, verify the service file contains the method + path combination.

**Result:** FAIL if >0 missing endpoints. May indicate API changes.

### VC-4: Completeness

Count items in the doc vs items found by a fresh scan. Flag if >10% missing.

**Method:** Run the extraction strategy's discovery step (find routes, find endpoints, find packages) and compare count against the doc's row count.

**Result:** WARN if 5-10% missing, FAIL if >10% missing. New items were added to the codebase since last generation.

### VC-5: Line budget

Warn if any doc exceeds 1,000 lines.

**Method:** `wc -l "$doc"`

**Result:** WARN at >1,000 lines. Suggest splitting into sub-documents or summarizing verbose sections.

### VC-6: Cross-reference bidirectionality

If doc A references doc B entry X, doc B should reference doc A.

**Method:** Parse "See also" links and cross-reference mentions. For each outgoing link, verify the target doc has a corresponding incoming link.

**Result:** WARN if unidirectional. Not a hard failure — the cross-reference pass in the generate-kb pipeline fixes these.

### VC-7: Token budget

Warn if any doc exceeds the token budget from `kb-context.md`.

**Method:** `lines × 8` as rough token estimate. Compare against configured budget.

**Result:** WARN if estimated tokens exceed budget. Agents consuming this doc may exceed their context window.

### VC-8: Freshness header accuracy

The freshness score in the doc header must match the manifest.

**Method:** Parse header `Freshness: {score}`, compare against `.manifest.json` freshness value for this doc.

**Result:** WARN if mismatch. Indicates the header wasn't updated after a refresh.

---

## Output Format Rules

Every generated document MUST follow these rules. The validator checks compliance.

1. **Header line** (first line after title):
   ```
   > Generated {date} at commit {short_sha}. Freshness: {score}. Every claim cites real files.
   ```

2. **Loading hint** (second metadata line):
   ```
   > Load priority: {hot|warm|cold}. Load when touching: {scope summary}.
   ```

3. **Staleness warning** (auto-inserted by refresher when freshness < 0.8):
   ```
   > ⚠ STALE since {date} — {N} scope files changed. Run /refresh-kb to update.
   ```

4. **Tables for inventories and registries**, not prose. One real snippet beats three paragraphs.

5. **Local file paths, not permalinks.** Agents consume paths via the Read tool. The commit SHA in the header provides traceability.

6. **"See also" cross-references** at the end of each doc, linking to related docs.

7. **No verbatim code dumps** — summarize structure, cite paths. Max 3 lines of inline code per entry.

8. **Sequential numbering** — docs are numbered `01-repo-map.md`, `02-apps/`, `03-shared-code.md`, etc. Custom packs continue the sequence.
