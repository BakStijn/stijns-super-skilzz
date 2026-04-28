# Skill: audit-maintainability

## Metadata

| Field | Value |
|---|---|
| `name` | `audit-maintainability` |
| `version` | `1.0.0` |
| `primary_category` | `assessment` |
| `secondary_category` | (none) |
| `output_mode` | `human_readable` |
| `output_format` | `markdown` |
| `evidence_mode` | `required` |
| `audience` | `technical` |
| `tone` | `neutral` |
| `distribution_mode` | `plugin` |

## Purpose

Evaluates already-written code against a fixed definition of maintainability. The skill accepts a single file, a folder, or the whole solution as input. At runtime it detects which technique(s) are used in the scope, investigates the conventions already established in the codebase, fetches current external best practices via live web research (max. 2 years old), and produces a markdown report of findings — each with file location, severity, actionable recommendation, and confidence — plus a summary for the developer.

## When to use

- The developer wants a maintainability assessment of a file, folder, or solution.
- The developer wants their code measured against both internal consistency and current external best practices for the detected stack.
- The developer wants findings grounded in the 15 maintainability principles defined below.

## When not to use

- For quality dimensions other than maintainability — security, performance, correctness/bugs, accessibility, and test coverage are explicitly **out of scope** and must be ignored, even when observed.
- For analyzing external dependencies (e.g., `node_modules`, vendored libraries, build output, generated code) — only files developers actively work in are in scope.
- When the live web research fails **and** stack detection also yields no recognizable technique — the skill stops in that combined case.

## Required input

| Field | Type | Description |
|---|---|---|
| `target` | string | A path to a single file, a path to a folder, or a marker that the whole solution/context is in scope. |

## Optional input

None.

## Input schema

```yaml
target:
  type: string
  required: true
  one_of:
    - path_to_file        # e.g. "src/components/Cart.vue"
    - path_to_folder      # e.g. "src/features/checkout"
    - whole_solution      # e.g. "." or explicit "solution"
```

## Maintainability definition (the 15 principles)

The skill assesses against these principles. Findings must reference the principle they relate to.

1. **Understandability** — a reader grasps quickly *what* the code does and *why*.
2. **Readability** — names, structure, and layout tell the story.
3. **Predictability** — similar situations are solved similarly; behavior matches naming and context.
4. **Modifiability** — a change in one place does not cascade; impact is bounded and predictable.
5. **Testability** — behavior can be isolated and verified; "correct" is well-defined.
6. **Discoverability** — logic lives where one expects it; conventions guide finding it.
7. **Consistency** — patterns are applied the same way across the codebase; deviations are deliberate.
8. **Simplicity** — solutions are no more complex than the problem requires; no speculative abstraction.
9. **Separation of concerns** — each unit has a clear role; concerns do not bleed across.
10. **Low coupling, high cohesion** — units know little of each other's internals; what belongs together stays together.
11. **Robustness to change** — code tolerates foreseeable change; decisions are reversible; failure is recognizable.
12. **Documentation & context** — non-trivial decisions, assumptions, and constraints are recorded where they apply; the *why* is captured.
13. **Debuggability** — root cause is reachable without reading the whole codebase; the system signals when something is wrong.
14. **Onboarding & transferability** — the codebase is not dependent on one person to be understood.
15. **Longevity** — code is written knowing it will be read more often than written; short-term workarounds are visible, not silently permanent.

## Processing rules

The skill must execute these steps in order.

### 1. Validate input
- If `target` is missing, refers to a path that does not exist, or is empty, **enter interview mode** and ask the user for a valid target. Do not fall through to a default.

### 2. Detect technique(s)
- Inspect manifest files, file extensions, framework markers (e.g. `package.json`, `pom.xml`, `*.csproj`, `Cargo.toml`, `requirements.txt`, configuration files, characteristic file extensions).
- If **no recognizable technique** can be determined, **stop** the run and report:
  ```
  ## Cannot proceed
  **Reason**: No recognizable technique detected in scope.
  **Missing**: A codebase with identifiable framework/language signals.
  **Action**: Provide a target containing recognizable source files.
  ```

### 3. Scope filtering
- Include only developer-authored files in scope.
- Exclude: `node_modules/`, `vendor/`, `dist/`, `build/`, `.next/`, `target/`, `bin/`, `obj/`, `__pycache__/`, lockfiles, generated code, minified output, and any folder marked as external dependencies for the detected stack.

### 4. Internal research
- Identify conventions already established in the codebase: naming style, folder structure, state management approach, component composition patterns, error handling patterns, test layout, etc.
- Record each observed convention as a baseline against which deviations within the same codebase will be flagged.

### 5. External research (live web)
- Query for current best practices for the detected technique(s). Sources must be no older than 2 years.
- If web research fails (no connectivity, no usable sources), **continue** with internal research + the 15 principles only, and add this disclaimer to the report:
  ```
  > **Disclaimer**: External best-practice research was unavailable for this run. Findings are based on the codebase's own conventions and the 15 maintainability principles only.
  ```

### 6. Time-budget control
- The full run must complete within 1 hour.
- If the elapsed time approaches the budget before analysis is complete, **interrupt** and ask the user:
  ```
  Time budget of 1 hour is approaching. Estimated remaining duration: <X> minutes.
  Continue? (yes / no)
  ```

### 7. Generate findings
For each issue identified:
- Map the finding to one of the 15 principles (this is the `criterion`).
- Assign **severity** strictly per the assessment extension:
  - `Critical` — must fix; immediate risk or release blocker (rare for pure maintainability).
  - `Warning` — should fix; creates technical debt or future risk.
  - `Info` — consider; opportunity for improvement (style, minor convention deviation).
- Provide an **actionable recommendation** (concrete, not "consider improving").
- Cite **evidence** as `path:line` (or `path:line-line` for ranges).
- Assign **confidence** — `high` / `medium` / `low`.
- If the finding originates from a **conflict** between internal convention and external best practice: propose the external pattern, explain *why*, and include an **impact analysis** describing how applying the change would propagate through the solution.

### 8. Sensitivity
- Lean toward **sensitivity over specificity**. Include findings of `medium` or `low` confidence, but mark confidence accurately. Do not fabricate findings; if nothing is found, say so.

### 9. Privacy & data leak prevention
- Never include the contents of secret-bearing files (`.env*`, `*.pem`, `*.key`, files matching common secret patterns) in the report or in any web query.
- Never send file contents, file names, or identifiable strings (variable names, comments containing project-specific info, customer names) as part of external web queries. External queries must be **abstracted** to the technique and pattern level only (e.g. "Vue 3 component composition best practices 2024").
- If a file appears to contain credentials or personal data, skip it for analysis and note it as `**Note:** [path] skipped — appears to contain sensitive data.`

### 10. Strict scope
- If issues are observed that fall outside maintainability (security, performance, correctness, accessibility, test coverage), **do not assess** them. Add a single `**Note:**` line per category if observed, with no further detail.

### 11. Output
- Produce the markdown report per the output contract below.
- Show the **Summary** section to the user as the conversational reply, in addition to writing the full report to a markdown file.

## Output contract

The skill produces a markdown report with this exact structure:

```markdown
## Maintainability Audit Report

### Scope
- **Target**: <file/folder/solution path>
- **Detected technique(s)**: <list>
- **Files analyzed**: <count>
- **Files skipped**: <count> (<reason summary>)
- **Criteria**: 15 maintainability principles; internal conventions; external best practices (max. 2 years old)

### Summary
<one sentence: X Critical, Y Warning, Z Info — N findings total>

### Findings

| # | Criterion | Severity | Location | Finding | Recommendation | Confidence | Impact (if conflict) |
|---|---|---|---|---|---|---|---|
| 1 | <principle> | Critical/Warning/Info | `path:line` | <reason> | <concrete action> | high/medium/low | <impact analysis or empty> |

### Priority Actions
1. **[Critical]** <action>
2. **[Warning]** <action>
...

### Out-of-scope observations (if any)
**Note:** <category> issues observed but not assessed (out of scope).
```

**Override of standard assessment contract:**
The default assessment contract includes a per-criterion **Scores** table (`Pass` / `Needs Work` / `Fail`). This skill **omits** the Scores section by explicit user requirement: the goal is actionable findings, not a dashboard. Justification is recorded here and in the metadata.

## Self-check

Before delivering output, the agent verifies:

```
□ Every finding cites specific evidence (path:line).
□ Every finding has a severity from the fixed set (Critical / Warning / Info).
□ Every finding has an actionable recommendation.
□ Every finding has a confidence value (high / medium / low).
□ Every conflict-driven finding has an impact analysis filled in.
□ No issues were fabricated; "No issues found" is used when applicable.
□ No out-of-scope dimensions were assessed in depth (only flagged as Notes).
□ No secret-bearing files appear in the report or were sent to external queries.
□ Disclaimer present if external research failed.
□ Summary line is correct and matches the count of findings by severity.
□ Output matches the exact contract structure (no missing or extra sections).
```

## Failure behavior

| Situation | Behavior |
|---|---|
| `target` missing or path does not exist | Enter interview mode; ask the user to provide a valid path. |
| Live web research fails | Continue with internal research + 15 principles; include disclaimer in report. |
| No recognizable technique detected | Stop the run; produce structured `Cannot proceed` block. |
| 1-hour time budget threatens to be exceeded | Pause; ask the user whether to continue, with an estimate of remaining duration. |
| Input refers to external dependencies only (e.g. only `node_modules/`) | Reject with reason: scope contains no developer-authored files. |
| Input contains only secret-bearing files | Reject with reason: nothing analyzable in scope. |
| User asks the skill to assess security/performance/etc. | Refuse: state the skill's scope is maintainability only; suggest a different skill. |

## Quality checks

- Findings are perceived as **plausible** by a developer familiar with the codebase.
- Sensitivity is favored over specificity; lower-confidence findings are included but clearly labeled.
- All findings are **deterministic** for the same input.
- No fabrication; "No issues found" is a valid outcome.
- All evidence is precisely addressed.
- All recommendations are concrete and immediately actionable.
- No content from secret-bearing files leaks into the report or external queries.
- Out-of-scope categories are at most flagged as a Note, never assessed.

## Examples

### Normal cases

**Example 1 — Single Vue component**
- Input: `src/components/Cart.vue`
- Detected: Vue 3 SFC.
- Internal: codebase consistently uses `<script setup>` and Pinia stores.
- External: best practice — extract complex template logic to computed properties.
- Finding: `Cart.vue:48-72` mixes data transformation in template; severity `Warning`; recommendation: extract into a `computed` named `displayedItems`; confidence `high`.

**Example 2 — Backend folder (Node + Express)**
- Input: `src/api/orders/`
- Detected: Node.js, Express, TypeScript.
- Finding: `src/api/orders/handler.ts:120-145` — single function handling validation, persistence, and notification; severity `Warning`; principle: Separation of concerns; recommendation: split into `validateOrder`, `persistOrder`, `notifySubscribers`.

**Example 3 — Whole solution scan**
- Input: `.`
- Detected: Vue + .NET backend.
- 12 findings across both subprojects, grouped by criterion. Summary: `0 Critical, 7 Warning, 5 Info`.

**Example 4 — Conflict between internal and external**
- Internal: codebase consistently uses Options API in Vue.
- External best practice: Composition API for new components.
- Finding: severity `Info`; recommendation: adopt Composition API for new components; impact: existing 40 components remain Options API; migration is incremental, no immediate refactor required.

**Example 5 — Clean code, no findings**
- Input: a small, well-structured utility module.
- Output: `Summary: 0 Critical, 0 Warning, 0 Info — No issues found.`

### Edge cases

**Example 6 — Mixed-tech monorepo**
- Detected: Vue, .NET, Python.
- Skill runs internal + external research per detected stack and aggregates findings under a single report.

**Example 7 — Very large solution, time budget threatens**
- After 50 minutes, remaining estimated 25 min for full coverage.
- Skill pauses, asks user, user replies `no`. Skill produces a partial report flagging which paths were not analyzed.

**Example 8 — Web research returns no recent sources**
- Disclaimer added to report; findings come from internal conventions + 15 principles only.

### Failure cases

**Example 9 — Path does not exist**
- Input: `src/featurz/`.
- Skill enters interview mode: "The path `src/featurz/` was not found. Please provide a valid file or folder path, or `.` for the whole solution."

**Example 10 — Unrecognizable technique**
- Input: a folder containing only plain text files.
- Skill emits the `Cannot proceed` block.

## Test requirements

In addition to the cases above, validation must confirm:
- **False positives**: clean input produces "No issues found", not fabricated items.
- **False negatives**: input with a known SoC violation produces a Warning under the Separation-of-concerns principle.
- **Severity accuracy**: a stylistic deviation is `Info`, never `Warning` or `Critical`.
- **Privacy**: an input folder containing a `.env` file produces a report that never quotes its contents and never includes its contents in external queries.
