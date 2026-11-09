---
name: openspec-archive-change
description: Archive a completed change in the experimental workflow. Use when the user wants to finalize and archive a change after implementation is complete.
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.8.0"
---

Archive a completed change in the experimental workflow.

**Input**: Optionally specify a change name after `openspec-archive-change` (e.g., `openspec-archive-change add-auth`). If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

**Steps**

1. **If no change name provided, prompt for selection**

   Run `openspec list --json` to get available changes. Use the **AskUserQuestion tool** to let the user select.

   Show only active changes (not already archived).
   Include the schema used for each change if available.

   **IMPORTANT**: Do NOT guess or auto-select a change. Always let the user choose.

2. **Check artifact completion status**

   Run `openspec status --change "<name>" --json` to check artifact completion.

   Parse the JSON to understand:
   - `schemaName`: The workflow being used
   - `artifacts`: List of artifacts with their status (`done` or other)

   **If any artifacts are not `done`:**
   - Display warning listing incomplete artifacts
   - Prompt user for confirmation to continue
   - Proceed if user confirms

3. **Check task completion status**

   Read the tasks file (typically `tasks.md`) to check for incomplete tasks.

   Count tasks marked with `- [ ]` (incomplete) vs `- [x]` (complete).

   **If incomplete tasks found:**
   - Display warning showing count of incomplete tasks
   - Prompt user for confirmation to continue
   - Proceed if user confirms

   **If no tasks file exists:** Proceed without task-related warning.

4. **Assess delta spec sync state**

   Assess both main spec integration and affected repos/modules' local `SPEC.md` updates, including pending sync notes in the change's artifacts. Local files remain with their code and are refreshed by `openspec-sync-specs`, which asks the user to approve each `SPEC.md` change.

   Recursively find delta `spec.md` files under `openspec/changes/<name>/specs/`. Use each file's full directory path relative to `specs/` as its capability ID. Proceed without a sync prompt only when there are no delta specs and no local updates needed.

   **If delta specs exist:**
   - Compare each delta spec with its corresponding main spec at `openspec/specs/<capability-id>/spec.md`; preserve nested domain segments in `<capability-id>`.
   - Determine what changes would be applied (adds, modifications, removals, renames)
   - Show a combined summary before prompting

   **Prompt options:**
   - If either main specs or local `SPEC.md` needs updates: "Sync now (recommended)", "Archive without syncing"
   - If both are current: "Archive now", "Sync anyway", "Cancel"

   If the user chooses sync, invoke `openspec-sync-specs` for this change, including local refresh even when main specs are already current. Recheck pending entries afterward. Resolve them before normal archival; if any remain, explain the reasons and proceed only if the user explicitly chooses archive with those warnings. Include outstanding local updates when the user chooses to skip sync.

5. **Perform the archive**

   **Merge GLOSSARY (if present):**

   Check if `openspec/changes/<name>/GLOSSARY.md` exists.

   **If it exists:**
   - If project-level `openspec/GLOSSARY.md` doesn't exist → create it from the change-level content as-is
   - Otherwise, merge each term from the change-level GLOSSARY:
     - Term not present in project-level → append to the corresponding `## section`
     - Term already present → overwrite with the change-level definition
     - Terms only in project-level → keep unchanged
   - After merging, delete `openspec/changes/<name>/GLOSSARY.md`
   - Note any overwrites in the summary, e.g. "overwrote Principal (was: 登入使用者)"

   Create the archive directory if it doesn't exist:
   ```bash
   mkdir -p openspec/changes/archive
   ```

   Generate target name using current date: `YYYY-MM-DD-<change-name>`

   **Check if target already exists:**
   - If yes: Fail with error, suggest renaming existing archive or using different date
   - If no: Move the change directory to archive

   ```bash
   mv openspec/changes/<name> openspec/changes/archive/YYYY-MM-DD-<name>
   ```

6. **Display summary**

   Show archive completion summary including:
   - Change name
   - Schema that was used
   - Archive location
   - Spec sync status (synced / sync skipped / no delta specs)
   - Local `SPEC.md` status and any unresolved pending paths
   - GLOSSARY merge status (merged / no GLOSSARY)
   - Note about any warnings (incomplete artifacts/tasks)

**Output On Success**

```
## Archive Complete

**Change:** <change-name>
**Schema:** <schema-name>
**Archived to:** openspec/changes/archive/YYYY-MM-DD-<name>/
**Specs:** ✓ Synced to main specs (or "No delta specs" or "Sync skipped")
**Glossary:** ✓ Merged to openspec/GLOSSARY.md (N terms) (or "No GLOSSARY")

All artifacts complete. All tasks complete.
```

**Output On Success With Warnings**

```
## Archive Complete (with warnings)

**Change:** <change-name>
**Schema:** <schema-name>
**Archived to:** openspec/changes/archive/YYYY-MM-DD-<name>/
**Specs:** Sync skipped (user chose to skip)
**Glossary:** ✓ Merged to openspec/GLOSSARY.md (N terms) (or "No GLOSSARY")

**Warnings:**
- Archived with 2 incomplete artifacts
- Archived with 3 incomplete tasks
- Delta spec sync was skipped (user chose to skip)

Review the archive if this was not intentional.
```

**Output On Error (Archive Exists)**

```
## Archive Failed

**Change:** <change-name>
**Target:** openspec/changes/archive/YYYY-MM-DD-<name>/

Target archive directory already exists.

**Options:**
1. Rename the existing archive
2. Delete the existing archive if it's a duplicate
3. Wait until a different date to archive
```

**Guardrails**
- Always prompt for change selection if not provided
- Use artifact graph (openspec status --json) for completion checking
- Don't block archive on warnings - just inform and confirm
- Preserve .openspec.yaml when moving to archive (it moves with the directory)
- Show clear summary of what happened
- If sync is requested, use the Skill tool to invoke `openspec-sync-specs` (agent-driven)
- Assess main specs and local `SPEC.md` together before prompting; already-synced main specs do not imply local documentation is current
