---
description: Verify implementation matches change artifacts before archiving
---

Verify that an implementation matches the change artifacts (specs, tasks, design).

**Input**: Optionally specify a change name after `/opsx:verify` (e.g., `/opsx:verify add-auth`). If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

**Steps**

1. **If no change name provided, prompt for selection**

   Run `openspec list --json` to get available changes. Use the **AskUserQuestion tool** to let the user select.

   Show changes that have implementation tasks (tasks artifact exists).
   Include the schema used for each change if available.
   Mark changes with incomplete tasks as "(In Progress)".

   **IMPORTANT**: Do NOT guess or auto-select a change. Always let the user choose.

2. **Check status to understand the schema**
   ```bash
   openspec status --change "<name>" --json
   ```
   Parse the JSON to understand:
   - `schemaName`: The workflow being used (e.g., "spec-driven")
   - Which artifacts exist for this change

3. **Get the change directory and load artifacts**

   ```bash
   openspec instructions apply --change "<name>" --json
   ```

   This returns the change directory and `contextFiles` (artifact ID -> array of concrete file paths). Read all available artifacts from `contextFiles`.

   Also read the glossaries if present — delta specs may reference terms defined there:
   - `openspec/GLOSSARY.md` (project-level terms, if exists)
   - `openspec/changes/<name>/GLOSSARY.md` (change-level terms, if exists)

4. **Initialize verification report structure**

   Create a report structure with three dimensions:
   - **Completeness**: Track tasks and spec coverage
   - **Correctness**: Track requirement implementation and scenario coverage
   - **Coherence**: Track design adherence and pattern consistency

   Each dimension can have CRITICAL, WARNING, or SUGGESTION issues.

5. **Verify Completeness**

   **Task Completion**:
   - If `contextFiles.tasks` exists, read every file path in it
   - Parse checkboxes: `- [ ]` (incomplete) vs `- [x]` (complete)
   - Count complete vs total tasks
   - If incomplete tasks exist:
     - Add CRITICAL issue for each incomplete task
     - Recommendation: "Complete task: <description>" or "Mark as done if already implemented"

   **Spec Coverage**:
   - If delta specs exist in `openspec/changes/<name>/specs/`:
     - Extract all requirements (marked with "### Requirement:")
     - For each requirement:
       - Search codebase for keywords related to the requirement
       - Assess if implementation likely exists
     - If requirements appear unimplemented:
       - Add CRITICAL issue: "Requirement not found: <requirement name>"
       - Recommendation: "Implement requirement X: <description>"

6. **Verify Correctness**

   **Requirement Implementation Mapping**:
   - For each requirement from delta specs:
     - Search codebase for implementation evidence
     - If found, note file paths and line ranges
     - Assess if implementation matches requirement intent
     - If divergence detected:
       - Add WARNING: "Implementation may diverge from spec: <details>"
       - Recommendation: "Review <file>:<lines> against requirement X"

   **Scenario Coverage**:
   - For each scenario in delta specs (marked with "#### Scenario:"):
     - Check if conditions are handled in code
     - Check if tests exist covering the scenario
     - If scenario appears uncovered:
       - Add WARNING: "Scenario not covered: <scenario name>"
       - Recommendation: "Add test or implementation for scenario: <description>"

7. **Verify Coherence**

   **Design Adherence**:
   - If `contextFiles.design` exists:
     - Extract key decisions (look for sections like "Decision:", "Approach:", "Architecture:")
     - Verify implementation follows those decisions
     - If contradiction detected:
       - Add WARNING: "Design decision not followed: <decision>"
       - Recommendation: "Update implementation or revise design.md to match reality"
   - If no design.md: Skip design adherence check, note "No design.md to verify against"

   **Code Pattern Consistency**:
   - Review new code for consistency with project patterns
   - Check file naming, directory structure, coding style
   - If significant deviations found:
     - Add SUGGESTION: "Code pattern deviation: <details>"
     - Recommendation: "Consider following project pattern: <example>"

8. **Execute Custom Verification from VERIFY.md (if any)**

   After the built-in three-dimension checks, look for and execute user-defined verification via `VERIFY.md`. This is a **supplement** — built-in checks always run first, custom verification adds domain-specific validation on top.

   **Detection**:

   Check for VERIFY.md in this order:

   a. **Project-level** — `openspec/VERIFY.md`
   b. **Change-level** — `openspec/changes/<name>/VERIFY.md`

   Both are read if present. Change-level **appends** to project-level — it does not replace.

   **VERIFY.md Format**:

   The file is markdown with one section per repo/capability. Each section heading is the repo name, and the body is a fenced code block containing shell commands to run:

   ```markdown
   # Verification

   ## PIC_圖資管理系統

   ```bash
   npm run lint
   npm run typecheck
   npm test
   ```

   ## shared-lib

   ```bash
   cargo build
   cargo test
   ```
   ```

   **Scope-Aware Execution**:

   The agent determines which sections are relevant by checking the change's scope (which repos/capabilities the change touches):

   1. Read the VERIFY.md sections
   2. Cross-reference with the change's affected files/repo to determine which sections are in scope
   3. Execute only the in-scope sections' code blocks
   4. Report skipped sections with reason: `"Skipped <repo> (not in change scope)"`

   **Execution**:

   For each in-scope section:

   1. Run the code block commands using the `bash` tool
   2. Capture stdout, stderr, and exit code
   3. Interpret results: determine which commands passed/failed, severity of failures
   4. Report findings in the Custom Verification section of the report

   **If no VERIFY.md is found**:

   Add a note to the report:
   ```
   > No custom verification defined.
   > Create `openspec/VERIFY.md` to add domain-specific checks.
   ```

   **VERIFY.md contract**:
   - **Structure**: Markdown with `## <repo-name>` headings and fenced bash code blocks
   - **Scope**: Agent decides which sections to run based on change scope
   - **Commands**: Standard shell commands. Agent runs them and captures output
   - **No special format required inside code blocks**: Any valid shell commands work

9. **Generate Verification Report**

   **Summary Scorecard**:
   ```
   ## Verification Report: <change-name>

   ### Summary
   | Dimension    | Status           |
   |--------------|------------------|
   | Completeness | X/Y tasks, N reqs|
   | Correctness  | M/N reqs covered |
   | Coherence    | Followed/Issues  |

   ### Custom Verification
   [See below for custom script results]

   ### Issues
   [See below for issues by priority]
   ```

   **Custom Verification Section** (placed after Summary, before Issues):

   If VERIFY.md was found and sections were executed, output a section for each:

   ```
   ### Custom Verification

   #### PIC_圖資管理系統
   [Agent's interpretation — which commands ran, which passed/failed]

   <details>
   <summary>Command Output</summary>

   ```
   [Raw output from the commands]
   ```

   </details>

   #### shared-lib (Skipped — not in change scope)

   #### web-frontend
   [Only if in scope and present in VERIFY.md]

   <details>
   <summary>Command Output</summary>

   ```
   [Raw output from the commands]
   ```

   </details>
   ```

   If no VERIFY.md was found, display the guidance message instead.

   **Issues by Priority**:

   1. **CRITICAL** (Must fix before archive):
      - Incomplete tasks
      - Missing requirement implementations
      - Each with specific, actionable recommendation

   2. **WARNING** (Should fix):
      - Spec/design divergences
      - Missing scenario coverage
      - Each with specific recommendation

   3. **SUGGESTION** (Nice to fix):
      - Pattern inconsistencies
      - Minor improvements
      - Each with specific recommendation

   **Final Assessment**:
   - If CRITICAL issues: "X critical issue(s) found. Fix before archiving."
   - If only warnings: "No critical issues. Y warning(s) to consider. Ready for archive (with noted improvements)."
   - If all clear: "All checks passed. Ready for archive."

**Verification Heuristics**

- **Completeness**: Focus on objective checklist items (checkboxes, requirements list)
- **Correctness**: Use keyword search, file path analysis, reasonable inference - don't require perfect certainty
- **Coherence**: Look for glaring inconsistencies, don't nitpick style
- **Custom Verification**: Interpret command output holistically — look for error messages, failure counts, tool-specific output (e.g., eslint, tsc, pytest). When the output is ambiguous, prefer WARNING over CRITICAL. When uncertain whether a line is an error or a warning, check context (exit code, surrounding lines). Skipped sections (not in change scope) should not count against the verification result.
- **False Positives**: When uncertain, prefer SUGGESTION over WARNING, WARNING over CRITICAL
- **Actionability**: Every issue must have a specific recommendation with file/line references where applicable

**Graceful Degradation**

- If only tasks.md exists: verify task completion only, skip spec/design checks
- If tasks + specs exist: verify completeness and correctness, skip design
- If full artifacts: verify all three dimensions
- Custom verification via VERIFY.md is always checked regardless of artifact availability — it may validate things outside the artifact system (lint, typecheck, security scans, etc.)
- If a command fails to execute (syntax error, missing dependency), report it as a WARNING with the error output, not as a verification check failure
- Always note which built-in checks were skipped and why

**Output Format**

Use clear markdown with:
- Table for summary scorecard
- Grouped lists for issues (CRITICAL/WARNING/SUGGESTION)
- Code references in format: `file.ts:123`
- Specific, actionable recommendations
- No vague suggestions like "consider reviewing"