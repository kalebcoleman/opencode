# Balanced Model Cost Optimization for OMO Config

## TL;DR

> **Quick Summary**: Replace all-premium routing in `oh-my-opencode.json` with a balanced two-tier low-cost strategy using both `opencode/gpt-5-nano` and `opencode/glm-5-free`, while keeping critical orchestration on `openai/gpt-5.3-codex`.
>
> **Deliverables**:
> - Updated agent/category model routing in `oh-my-opencode.json`
> - Preserved `_migrations` and non-target top-level sections
> - Automated evidence for syntax, availability, scope integrity, and smoke behavior
>
> **Estimated Effort**: Short
> **Parallel Execution**: YES - 3 waves
> **Critical Path**: Task 1 -> Task 5 -> Task 6/7 -> Task 9 -> Task 10

---

## Context

### Original Request
User asked to make `oh-my-opencode.json` cost effective because nearly all agents/categories are pinned to `openai/gpt-5.3-codex`.

### Interview Summary
**Key Discussions**:
- User approved a **Balanced** savings strategy (not maximum-aggressive degradation).
- User explicitly requested using **both** `opencode/gpt-5-nano` and `opencode/glm-5-free` by suitability.

**Research Findings**:
- Local config hotspot: `oh-my-opencode.json` maps 9/10 agents and 8/8 categories to `openai/gpt-5.3-codex`.
- Existing known-good cheap lane already present: `atlas` -> `opencode/glm-5-free`.
- Official/community patterns consistently route quick/research lanes to cheaper/free models.
- Model strings are permissive in schema; typos are runtime failures, not schema failures.

### Metis Review
**Identified Gaps (addressed)**:
- Guardrails tightened: single-file routing change only, preserve `_migrations`, no provider/concurrency edits.
- Added runtime model-availability checks (`opencode models`) before/after edits.
- Added acceptance criteria for scope integrity and representative cheap vs premium smoke checks.

---

## Work Objectives

### Core Objective
Reduce recurring model cost in oh-my-opencode routing while keeping reliability for orchestrator/high-complexity lanes.

### Concrete Deliverables
- Updated `agents.*.model` and `categories.*.model` values in `oh-my-opencode.json`.
- Optional variant downtuning on selected non-critical premium categories.
- Verification evidence under `.sisyphus/evidence/`.

### Definition of Done
- [ ] `python -m json.tool "/Users/itzjuztmya/.config/opencode/oh-my-opencode.json" >/dev/null` exits `0`
- [ ] `opencode models opencode` output contains `opencode/gpt-5-nano` and `opencode/glm-5-free`
- [ ] `_migrations` unchanged compared to pre-edit snapshot
- [ ] One cheap lane (`explore`) and one premium lane (`sisyphus`) pass smoke run

### Must Have
- Use both `opencode/gpt-5-nano` and `opencode/glm-5-free` in low-cost routing.
- Keep premium lane on `openai/gpt-5.3-codex` for critical orchestrators/hardest work.

### Must NOT Have (Guardrails)
- No edits outside `oh-my-opencode.json`.
- No changes to `tmux`, `background_task`, provider/auth settings, or plugin wiring.
- No cleanup/refactor of `_migrations` (preserve exact content/order).
- No model pools/round-robin/fallback constructs beyond existing schema usage.

---

## Verification Strategy (MANDATORY)

> **ZERO HUMAN INTERVENTION** — all checks are command/tool executable.

### Test Decision
- **Infrastructure exists**: N/A (config-only task)
- **Automated tests**: None
- **Framework**: CLI command verification
- **Agent-Executed QA**: Mandatory for every task

### QA Policy
Evidence path convention:
- `.sisyphus/evidence/task-{N}-{scenario-slug}.txt`
- `.sisyphus/evidence/task-{N}-{scenario-slug}-error.txt`

Tools:
- `Bash` for JSON validation, model listing, and smoke commands
- `Read` for before/after config assertions

---

## Execution Strategy

### Parallel Execution Waves

Wave 1 (Start Immediately - planning + validation foundations):
- Task 1: Capture immutable baseline snapshot + target-path allowlist [quick]
- Task 2: Build balanced routing matrix (premium/nano/free) [deep]
- Task 3: Validate runtime model availability for target IDs [quick]
- Task 4: Decide variant policy for non-critical premium lanes [quick]
- Task 5: Produce exact JSON path change list (no scope creep) [deep]

Wave 2 (After Wave 1 - apply config updates in parallel chunks):
- Task 6: Apply agent-level remap (research/helper lanes) [quick]
- Task 7: Apply category-level remap (quick/low/writing lanes) [quick]
- Task 8: Apply variant downshift for selected non-critical premium categories [quick]
- Task 9: Run structural integrity checks (`json`, `_migrations`, out-of-scope) [unspecified-high]
- Task 10: Run representative cheap vs premium smoke checks [unspecified-high]

Wave 3 (After Wave 2 - hardening + rollback readiness):
- Task 11: Create rollback procedure using pre-edit snapshot [writing]
- Task 12: Run lightweight benchmark prompts for low-cost lanes [unspecified-high]
- Task 13: Finalize evidence index and change audit [writing]

Wave FINAL (After ALL tasks - independent review, parallel):
- Task F1: Plan compliance audit [oracle]
- Task F2: Code quality/config sanity review [unspecified-high]
- Task F3: Real command-level QA replay [unspecified-high]
- Task F4: Scope fidelity check [deep]

Critical Path: 1 -> 5 -> 6/7 -> 9 -> 10
Parallel Speedup: ~55% faster than strict sequential
Max Concurrent: 5 (Waves 1 & 2)

### Dependency Matrix
- 1: -- -> 5, 9, 11
- 2: -- -> 6, 7, 8
- 3: -- -> 6, 7, 10
- 4: -- -> 8
- 5: 1,2 -> 6,7,8
- 6: 2,3,5 -> 9,10,12
- 7: 2,3,5 -> 9,10,12
- 8: 2,4,5 -> 9,10
- 9: 6,7,8 -> 13,F1-F4
- 10: 3,6,7,8 -> 12,13,F1-F4
- 11: 1,9 -> 13,F4
- 12: 6,7,10 -> 13,F3
- 13: 9,10,11,12 -> F1-F4

### Agent Dispatch Summary
- Wave 1: T1 quick, T2 deep, T3 quick, T4 quick, T5 deep
- Wave 2: T6 quick, T7 quick, T8 quick, T9 unspecified-high, T10 unspecified-high
- Wave 3: T11 writing, T12 unspecified-high, T13 writing
- Final: F1 oracle, F2 unspecified-high, F3 unspecified-high, F4 deep

---

## TODOs

- [x] 1. Capture baseline snapshot and guardrails

  **What to do**:
  - Save pre-edit snapshot to `/tmp/oh-my-opencode.before.json`.
  - Record allowed edit paths: `agents.*.model`, `agents.*.variant`, `categories.*.model`, `categories.*.variant`.

  **Must NOT do**:
  - Do not mutate source config in this task.

  **Recommended Agent Profile**:
  - **Category**: `quick` (fast deterministic setup)
  - **Skills**: `git-master`, `superpowers/verification-before-completion`

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1
  - **Blocks**: 5, 9, 11
  - **Blocked By**: None

  **References**:
  - `oh-my-opencode.json:18` - current `agents` map to baseline.
  - `oh-my-opencode.json:56` - current `categories` map to baseline.
  - `oh-my-opencode.json:86` - `_migrations` block that must stay unchanged.

  **Acceptance Criteria**:
  - [ ] `/tmp/oh-my-opencode.before.json` exists and matches current file bytes.
  - [ ] Guardrail list recorded in evidence.

  **QA Scenarios**:
  ```
  Scenario: Baseline snapshot written
    Tool: Bash
    Preconditions: Source config exists at /Users/itzjuztmya/.config/opencode/oh-my-opencode.json
    Steps:
      1. Run: cp "/Users/itzjuztmya/.config/opencode/oh-my-opencode.json" "/tmp/oh-my-opencode.before.json"
      2. Run: python -m json.tool "/tmp/oh-my-opencode.before.json" >/dev/null
      3. Run: shasum "/tmp/oh-my-opencode.before.json" "/Users/itzjuztmya/.config/opencode/oh-my-opencode.json"
    Expected Result: Both files hash-equal before edits.
    Failure Indicators: Missing file, non-zero exit code, hash mismatch.
    Evidence: .sisyphus/evidence/task-1-baseline.txt

  Scenario: Missing source file fails clearly
    Tool: Bash
    Preconditions: Use intentionally wrong path
    Steps:
      1. Run: cp "/Users/itzjuztmya/.config/opencode/does-not-exist.json" "/tmp/oh-my-opencode.before.json"
      2. Assert command exits non-zero.
    Expected Result: Error includes "No such file or directory".
    Evidence: .sisyphus/evidence/task-1-baseline-error.txt
  ```

- [x] 2. Define balanced routing matrix (premium vs nano vs glm-free)

  **What to do**:
  - Create explicit mapping table before edits.
  - Keep premium set: `sisyphus`, `hephaestus`, `oracle`, `prometheus`, `metis`, `momus`, plus `ultrabrain`, `deep`, `artistry`, `unspecified-high`.
  - Assign low-cost split:
    - `opencode/gpt-5-nano`: `explore`, `librarian`, `quick`, `visual-engineering`
    - `opencode/glm-5-free`: `multimodal-looker`, `writing`, `unspecified-low` (and keep `atlas`)

  **Must NOT do**:
  - Do not invent new categories or agent keys.

  **Recommended Agent Profile**:
  - **Category**: `deep` (policy and tradeoff precision)
  - **Skills**: `create-plan`, `superpowers/brainstorming`

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1
  - **Blocks**: 5, 6, 7, 8
  - **Blocked By**: None

  **References**:
  - `oh-my-opencode.json:19` - current premium-heavy agent assignments.
  - `oh-my-opencode.json:57` - current premium-heavy category assignments.
  - `oh-my-opencode.json.bak.2026-02-18T04-38-37-104Z:63` - precedent mixed-cost routing.
  - `https://github.com/code-yeongyu/oh-my-opencode/blob/dev/docs/configurations.md` - official mixed override examples.

  **Acceptance Criteria**:
  - [ ] Mapping table lists all edited lanes and target model IDs.
  - [ ] Table uses both `opencode/gpt-5-nano` and `opencode/glm-5-free`.

  **QA Scenarios**:
  ```
  Scenario: Mapping table complete
    Tool: Bash
    Preconditions: Plan file exists
    Steps:
      1. Search plan for "opencode/gpt-5-nano"
      2. Search plan for "opencode/glm-5-free"
      3. Verify listed premium keep-set names are present
    Expected Result: Both low-cost models and premium keep-set are documented.
    Failure Indicators: Missing either model or missing keep-set lane.
    Evidence: .sisyphus/evidence/task-2-matrix.txt

  Scenario: Invalid model id detected
    Tool: Bash
    Preconditions: Introduce temp bad ID in scratch note (not config)
    Steps:
      1. Compare bad ID against `opencode models opencode` output
      2. Assert no exact match found
    Expected Result: Invalid IDs are rejected before config edit.
    Evidence: .sisyphus/evidence/task-2-matrix-error.txt
  ```

- [x] 3. Validate runtime model availability

  **What to do**:
  - Run model listing commands and confirm target IDs are currently available.
  - Capture exact output snippets for audit.

  **Must NOT do**:
  - Do not proceed to edits if either low-cost model is unavailable.

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: `superpowers/verification-before-completion`

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1
  - **Blocks**: 6, 7, 10
  - **Blocked By**: None

  **References**:
  - `https://opencode.ai/docs/models/` - provider/model ID format.
  - `oh-my-opencode.json:53` - known good `opencode/glm-5-free` local usage.

  **Acceptance Criteria**:
  - [ ] `opencode models opencode` contains `opencode/gpt-5-nano` and `opencode/glm-5-free`.
  - [ ] `opencode models openai` contains `openai/gpt-5.3-codex`.

  **QA Scenarios**:
  ```
  Scenario: All target model IDs are available
    Tool: Bash
    Preconditions: OpenCode CLI authenticated
    Steps:
      1. Run: opencode models opencode
      2. Assert output contains exact strings `opencode/gpt-5-nano` and `opencode/glm-5-free`
      3. Run: opencode models openai
      4. Assert output contains `openai/gpt-5.3-codex`
    Expected Result: All target IDs present.
    Failure Indicators: Missing any required model ID.
    Evidence: .sisyphus/evidence/task-3-model-availability.txt

  Scenario: Provider unavailable error is surfaced
    Tool: Bash
    Preconditions: Temporarily run against unknown provider
    Steps:
      1. Run: opencode models not-a-provider
      2. Assert non-zero exit and clear error message
    Expected Result: Failure is explicit and blocks edits.
    Evidence: .sisyphus/evidence/task-3-model-availability-error.txt
  ```

- [x] 4. Set variant policy for balanced savings

  **What to do**:
  - Keep `xhigh` only for `ultrabrain`.
  - Keep `high` for critical orchestration agents.
  - Downshift `deep` and `artistry` from `high` to default (remove explicit variant) unless later smoke check fails.

  **Must NOT do**:
  - Do not remove variants from `prometheus`, `metis`, `momus`, or `ultrabrain` in this wave.

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: `create-plan`

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1
  - **Blocks**: 8
  - **Blocked By**: None

  **References**:
  - `oh-my-opencode.json:61` - `ultrabrain` currently `xhigh`.
  - `oh-my-opencode.json:65` - `deep` currently `high`.
  - `oh-my-opencode.json:69` - `artistry` currently `high`.

  **Acceptance Criteria**:
  - [ ] Variant keep/drop table exists in evidence.
  - [ ] Keep/drop aligns with balanced profile.

  **QA Scenarios**:
  ```
  Scenario: Variant policy aligns with balanced objective
    Tool: Bash
    Preconditions: Policy note prepared
    Steps:
      1. Check policy includes keep-set: ultrabrain/prometheus/metis/momus
      2. Check policy includes downshift candidates: deep/artistry
    Expected Result: Policy is explicit and conflict-free.
    Failure Indicators: Missing keep-set or no downshift candidates.
    Evidence: .sisyphus/evidence/task-4-variant-policy.txt

  Scenario: Contradictory policy is rejected
    Tool: Bash
    Preconditions: Create conflicting temp table
    Steps:
      1. Put same lane into both keep and downshift groups
      2. Run validation grep for duplicates
    Expected Result: Duplicate lane detected and corrected.
    Evidence: .sisyphus/evidence/task-4-variant-policy-error.txt
  ```

- [x] 5. Build exact JSON path change list

  **What to do**:
  - Enumerate each field to edit before touching file.
  - Include before->after values for every target lane.

  **Must NOT do**:
  - No wildcard edits.

  **Recommended Agent Profile**:
  - **Category**: `deep`
  - **Skills**: `superpowers/writing-plans`

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 1 (finisher)
  - **Blocks**: 6, 7, 8
  - **Blocked By**: 1, 2

  **References**:
  - `oh-my-opencode.json:19` - agent keys and current values.
  - `oh-my-opencode.json:57` - category keys and current values.
  - `oh-my-opencode.json:86` - `_migrations` exclude list.

  **Acceptance Criteria**:
  - [ ] Change list includes only whitelisted paths.
  - [ ] No out-of-scope key appears.

  **QA Scenarios**:
  ```
  Scenario: Change list is complete and scoped
    Tool: Bash
    Preconditions: Path list saved in evidence file
    Steps:
      1. Verify each listed path starts with `agents.` or `categories.`
      2. Verify no `tmux`, `background_task`, `_migrations` listed
    Expected Result: 100% of edits are in scope.
    Failure Indicators: Any forbidden top-level key appears.
    Evidence: .sisyphus/evidence/task-5-pathlist.txt

  Scenario: Out-of-scope path is blocked
    Tool: Bash
    Preconditions: Inject one fake path `background_task.defaultConcurrency`
    Steps:
      1. Run path allowlist checker
      2. Assert checker exits non-zero
    Expected Result: Checker rejects out-of-scope path.
    Evidence: .sisyphus/evidence/task-5-pathlist-error.txt
  ```

- [x] 6. Apply agent-level remap

  **What to do**:
  - Update agent routes:
    - `librarian` -> `opencode/gpt-5-nano`
    - `explore` -> `opencode/gpt-5-nano`
    - `multimodal-looker` -> `opencode/glm-5-free`
  - Keep critical orchestration agents on `openai/gpt-5.3-codex`.

  **Must NOT do**:
  - Do not change agent names, categories, or permissions.

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: `superpowers/verification-before-completion`

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2
  - **Blocks**: 9, 10, 12
  - **Blocked By**: 2, 3, 5

  **References**:
  - `oh-my-opencode.json:31` - `librarian` model field.
  - `oh-my-opencode.json:34` - `explore` model field.
  - `oh-my-opencode.json:37` - `multimodal-looker` model field.

  **Acceptance Criteria**:
  - [ ] Three target agents remapped exactly as matrix states.
  - [ ] `sisyphus`, `oracle`, `prometheus`, `metis`, `momus` unchanged model IDs.

  **QA Scenarios**:
  ```
  Scenario: Agent remap applied exactly
    Tool: Bash
    Preconditions: Config edited
    Steps:
      1. Parse JSON and print agents.librarian.model, agents.explore.model, agents.multimodal-looker.model
      2. Assert expected IDs exactly match matrix
    Expected Result: All three values match target IDs.
    Failure Indicators: Any mismatch or missing key.
    Evidence: .sisyphus/evidence/task-6-agent-remap.txt

  Scenario: Premium keep-set accidentally changed
    Tool: Bash
    Preconditions: Compare against baseline snapshot
    Steps:
      1. Diff premium agent model fields between baseline and edited file
      2. Assert no differences
    Expected Result: Keep-set unchanged.
    Evidence: .sisyphus/evidence/task-6-agent-remap-error.txt
  ```

- [x] 7. Apply category-level remap

  **What to do**:
  - Update category routes:
    - `quick` -> `opencode/gpt-5-nano`
    - `visual-engineering` -> `opencode/gpt-5-nano`
    - `writing` -> `opencode/glm-5-free`
    - `unspecified-low` -> `opencode/glm-5-free`
  - Keep `ultrabrain`, `deep`, `artistry`, `unspecified-high` on premium.

  **Must NOT do**:
  - Do not disable categories.

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: `superpowers/verification-before-completion`

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2
  - **Blocks**: 9, 10, 12
  - **Blocked By**: 2, 3, 5

  **References**:
  - `oh-my-opencode.json:57` - category mapping block.
  - `https://github.com/code-yeongyu/oh-my-opencode/blob/dev/docs/configurations.md` - mixed category model pattern.

  **Acceptance Criteria**:
  - [ ] Target categories mapped to exact IDs.
  - [ ] Premium category keep-set preserved.

  **QA Scenarios**:
  ```
  Scenario: Category remap applied
    Tool: Bash
    Preconditions: Config edited
    Steps:
      1. Parse and print categories.quick/model, visual-engineering/model, writing/model, unspecified-low/model
      2. Assert exact expected IDs
    Expected Result: All four mappings correct.
    Failure Indicators: Any mismatch.
    Evidence: .sisyphus/evidence/task-7-category-remap.txt

  Scenario: Category accidentally disabled
    Tool: Bash
    Preconditions: Config edited
    Steps:
      1. Search category objects for `"disable": true`
      2. Assert none found in targeted categories
    Expected Result: No disable flags introduced.
    Evidence: .sisyphus/evidence/task-7-category-remap-error.txt
  ```

- [x] 8. Apply variant downshift on selected non-critical premium lanes

  **What to do**:
  - Remove explicit `variant: high` from `deep` and `artistry` only.
  - Keep `ultrabrain` as `xhigh`; keep orchestrator high variants unchanged.

  **Must NOT do**:
  - No variant edits outside approved list.

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: `superpowers/verification-before-completion`

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2
  - **Blocks**: 9, 10
  - **Blocked By**: 2, 4, 5

  **References**:
  - `oh-my-opencode.json:61` - `ultrabrain` `xhigh` keep.
  - `oh-my-opencode.json:65` - `deep` currently high.
  - `oh-my-opencode.json:69` - `artistry` currently high.

  **Acceptance Criteria**:
  - [ ] `deep` and `artistry` no longer explicitly `high`.
  - [ ] `ultrabrain` remains `xhigh`.

  **QA Scenarios**:
  ```
  Scenario: Variant downshift applied correctly
    Tool: Bash
    Preconditions: Config edited
    Steps:
      1. Print categories.deep.variant and categories.artistry.variant
      2. Assert both are absent/null
      3. Print categories.ultrabrain.variant and assert `xhigh`
    Expected Result: Exactly approved variant changes applied.
    Failure Indicators: Ultrabrain changed, or deep/artistry still high.
    Evidence: .sisyphus/evidence/task-8-variant-downshift.txt

  Scenario: Critical variants changed accidentally
    Tool: Bash
    Preconditions: Baseline snapshot exists
    Steps:
      1. Compare orchestrator variants with baseline
      2. Assert no diff for prometheus/metis/momus
    Expected Result: Critical variants unchanged.
    Evidence: .sisyphus/evidence/task-8-variant-downshift-error.txt
  ```

- [x] 9. Run structural integrity checks

  **What to do**:
  - Validate JSON syntax.
  - Verify `_migrations` unchanged from `/tmp/oh-my-opencode.before.json`.
  - Verify top-level out-of-scope sections unchanged (`$schema`, `tmux`, `background_task`).

  **Must NOT do**:
  - Do not "fix" unrelated drift in this task.

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
  - **Skills**: `superpowers/verification-before-completion`

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 2 finisher
  - **Blocks**: 13, F1-F4
  - **Blocked By**: 6, 7, 8

  **References**:
  - `oh-my-opencode.json:1` - full JSON validity boundary.
  - `oh-my-opencode.json:86` - `_migrations` integrity target.

  **Acceptance Criteria**:
  - [ ] JSON parse command exits 0.
  - [ ] `_migrations` arrays are byte-equivalent pre/post.
  - [ ] Top-level out-of-scope sections exactly unchanged.

  **QA Scenarios**:
  ```
  Scenario: Integrity checks pass
    Tool: Bash
    Preconditions: Tasks 6-8 complete
    Steps:
      1. Run: python -m json.tool "/Users/itzjuztmya/.config/opencode/oh-my-opencode.json" >/dev/null
      2. Run python assert script comparing `_migrations` with /tmp baseline
      3. Run python assert script comparing `$schema`, `tmux`, `background_task`
    Expected Result: All assertions pass.
    Failure Indicators: Non-zero exit or assertion failure.
    Evidence: .sisyphus/evidence/task-9-integrity.txt

  Scenario: Malformed JSON blocks completion
    Tool: Bash
    Preconditions: Use temporary malformed copy only
    Steps:
      1. Validate malformed file with `python -m json.tool`
      2. Assert non-zero exit
    Expected Result: Parser fails and task cannot pass.
    Evidence: .sisyphus/evidence/task-9-integrity-error.txt
  ```

- [x] 10. Run cheap-vs-premium smoke checks

  **What to do**:
  - Run command smoke prompt through `explore` (cheap lane) and `sisyphus` (premium lane).
  - Confirm no provider/model resolution errors.

  **Must NOT do**:
  - Do not treat partial output as pass unless command exits 0.

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
  - **Skills**: `superpowers/verification-before-completion`

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 2 finisher
  - **Blocks**: 12, 13, F1-F4
  - **Blocked By**: 3, 6, 7, 8

  **References**:
  - `oh-my-opencode.json:34` - `explore` low-cost lane.
  - `oh-my-opencode.json:19` - `sisyphus` premium lane.

  **Acceptance Criteria**:
  - [ ] `opencode run "Reply with EXACTLY: OK" --agent explore` succeeds and outputs `OK`.
  - [ ] `opencode run "Reply with EXACTLY: OK" --agent sisyphus` succeeds and outputs `OK`.

  **QA Scenarios**:
  ```
  Scenario: Cheap lane smoke pass
    Tool: Bash
    Preconditions: Config updated
    Steps:
      1. Run explore smoke command
      2. Assert exit code 0 and output contains exact `OK`
    Expected Result: Command succeeds without ProviderModelNotFoundError.
    Failure Indicators: timeout, provider error, missing `OK`.
    Evidence: .sisyphus/evidence/task-10-explore-smoke.txt

  Scenario: Invalid agent invocation fails cleanly
    Tool: Bash
    Preconditions: Config updated
    Steps:
      1. Run: opencode run "Reply with EXACTLY: OK" --agent not-a-real-agent
      2. Assert non-zero exit code
      3. Assert error message clearly indicates unknown/invalid agent
    Expected Result: Failure is explicit and does not mutate config.
    Evidence: .sisyphus/evidence/task-10-sisyphus-smoke-error.txt
  ```

- [x] 11. Create rollback procedure

  **What to do**:
  - Document exact rollback command from `/tmp/oh-my-opencode.before.json`.
  - Define rollback triggers (model-not-found, repeated timeout, severe quality drop).

  **Must NOT do**:
  - Do not execute rollback unless smoke checks fail.

  **Recommended Agent Profile**:
  - **Category**: `writing`
  - **Skills**: `code-documentation`

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 3
  - **Blocks**: 13, F4
  - **Blocked By**: 1, 9

  **References**:
  - `/tmp/oh-my-opencode.before.json` - rollback source.
  - `oh-my-opencode.json` - rollback target.

  **Acceptance Criteria**:
  - [ ] Rollback command is executable and path-correct.
  - [ ] Trigger list is explicit and binary.

  **QA Scenarios**:
  ```
  Scenario: Rollback command validated (dry run)
    Tool: Bash
    Preconditions: Baseline snapshot exists
    Steps:
      1. Print rollback command in evidence
      2. Run command as dry-run equivalent (compare paths, not execute copy)
    Expected Result: Command is syntactically valid and points to real source.
    Failure Indicators: Missing source path or malformed command.
    Evidence: .sisyphus/evidence/task-11-rollback.txt

  Scenario: Missing baseline detected
    Tool: Bash
    Preconditions: Simulate missing snapshot path
    Steps:
      1. Check test path existence
      2. Assert non-zero and explanatory error
    Expected Result: Procedure blocks unsafe rollback.
    Evidence: .sisyphus/evidence/task-11-rollback-error.txt
  ```

- [x] 12. Run lightweight benchmark prompts on low-cost lanes

  **What to do**:
  - Run 3 short prompts through `explore` and `librarian` to sanity-check response quality and latency.
  - Compare against pre-change expected behavior (not exact wording).

  **Must NOT do**:
  - Do not expand into full benchmark suite; keep scope to smoke-level comparison.

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
  - **Skills**: `superpowers/verification-before-completion`

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 3
  - **Blocks**: 13, F3
  - **Blocked By**: 6, 7, 10

  **References**:
  - `oh-my-opencode.json:31` - `librarian` cheap lane.
  - `oh-my-opencode.json:34` - `explore` cheap lane.

  **Acceptance Criteria**:
  - [ ] All benchmark prompts return successful outputs.
  - [ ] No critical regressions (empty output, repeated tool-call failure, provider errors).

  **QA Scenarios**:
  ```
  Scenario: Low-cost lane benchmark passes
    Tool: Bash
    Preconditions: Cheap-lane remap complete
    Steps:
      1. Run three predefined prompts on explore
      2. Run three predefined prompts on librarian
      3. Record latency and success/fail
    Expected Result: >=5/6 successful responses, 0 provider/model errors.
    Failure Indicators: Provider errors, repeated timeouts, empty output.
    Evidence: .sisyphus/evidence/task-12-benchmark.txt

  Scenario: Stress prompt exposes instability
    Tool: Bash
    Preconditions: Use intentionally long prompt
    Steps:
      1. Send large prompt once to cheap lane
      2. Assert failure is graceful with explicit error (if any)
    Expected Result: No crash; error is explicit if request fails.
    Evidence: .sisyphus/evidence/task-12-benchmark-error.txt
  ```

- [x] 13. Finalize evidence index and change audit

  **What to do**:
  - Produce a concise evidence index linking each task to its artifact.
  - Record exact changed JSON paths and final model distribution counts.

  **Must NOT do**:
  - No further config edits in this task.

  **Recommended Agent Profile**:
  - **Category**: `writing`
  - **Skills**: `code-documentation`

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 3 finisher
  - **Blocks**: F1-F4
  - **Blocked By**: 9, 10, 11, 12

  **References**:
  - `.sisyphus/evidence/` - required proof artifacts.
  - `oh-my-opencode.json` - final audited config state.

  **Acceptance Criteria**:
  - [ ] Evidence index includes all task artifacts.
  - [ ] Audit includes path-level diff summary and lane distribution summary.

  **QA Scenarios**:
  ```
  Scenario: Evidence index complete
    Tool: Bash
    Preconditions: Evidence files created by prior tasks
    Steps:
      1. List expected evidence files for tasks 1-12
      2. Verify each file exists
    Expected Result: No missing evidence files.
    Failure Indicators: Any missing task artifact.
    Evidence: .sisyphus/evidence/task-13-evidence-index.txt

  Scenario: Missing evidence blocks closeout
    Tool: Bash
    Preconditions: Temporarily remove one expected file path from list
    Steps:
      1. Run existence checker
      2. Assert non-zero and clear missing filename output
    Expected Result: Closeout fails until evidence is complete.
    Evidence: .sisyphus/evidence/task-13-evidence-index-error.txt
  ```

---

## Final Verification Wave (MANDATORY)

- [x] F1. **Plan Compliance Audit** - `oracle`
  Validate each Must Have / Must NOT Have against final file and evidence artifacts.

- [x] F2. **Code Quality Review** - `unspecified-high`
  Confirm JSON validity, no unrelated key changes, and command pass/fail logs are complete.

- [x] F3. **Real Command QA Replay** - `unspecified-high`
  Re-run all task QA scenarios from clean shell and verify evidence files exist.

- [x] F4. **Scope Fidelity Check** - `deep`
  Compare actual changes to planned JSON paths and detect any contamination.

---

## Commit Strategy

- Commit once after all checks pass.
- Message: `chore(config): rebalance oh-my-opencode model routing for lower cost`
- Files: `oh-my-opencode.json` and optional `.sisyphus/evidence/*` index
- Pre-commit checks: JSON validation + model-list + smoke commands

---

## Success Criteria

### Verification Commands
```bash
python -m json.tool "/Users/itzjuztmya/.config/opencode/oh-my-opencode.json" >/dev/null
opencode models opencode
opencode models openai
opencode run "Reply with EXACTLY: OK" --agent explore
opencode run "Reply with EXACTLY: OK" --agent sisyphus
```

### Final Checklist
- [x] All Must Have items satisfied
- [x] All Must NOT Have violations absent
- [x] `_migrations` preserved exactly
- [x] Low-cost lanes use both nano and glm-free
- [x] Premium critical lanes remain on gpt-5.3-codex
