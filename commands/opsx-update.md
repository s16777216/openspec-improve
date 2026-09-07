---
description: Revise existing change artifacts and keep the plan consistent without editing code
---

Revise a change's existing planning artifacts and keep them coherent. Never edit code.

**CLI compatibility**: On Windows, use `openspec.cmd` in place of `openspec` if PowerShell execution policy blocks the `.ps1` shim. This fork uses the nearest local `openspec/` root; store/registry selection is not supported.

**Input**: Optionally specify a change name. If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

`/opsx-continue` is an expanded-profile workflow and may not be installed. Before suggesting it anywhere below, verify that it is available. If it is unavailable, `openspec status --change "<name>" --json` shows the next artifact and `openspec instructions "<artifact-id>" --change "<name>" --json` explains how to create it.

**Steps**

1. **Select the change**

   If a name is provided, use it. Otherwise:
   - Infer from conversation context if the user mentioned a change
   - Auto-select if only one active change exists
   - If ambiguous, run `openspec list --json` to get available changes sorted by most recently modified, and ask the user to select one

   When prompting, present the top 3-4 most recently modified changes as options, showing:
   - Change name
   - Schema (from `schema` field if present, otherwise "spec-driven")
   - Status (e.g., "0/5 tasks", "complete", "no tasks")
   - How recently it was modified (from `lastModified` field)

   Mark the most recently modified change as "(Recommended)" since it's likely what the user wants to update.

   Always announce: "Using change: <name>" and how to override (e.g., `/opsx-update <other>`).

2. **Get the change's artifacts**
   ```bash
   openspec status --change "<name>" --json
   ```
   Parse the JSON to understand current state. The response includes:
   - `schemaName`: The workflow schema being used (e.g., "spec-driven")
   - `artifacts`: Array of artifacts with their status ("done", "skipped", "ready", "blocked")
   - `isPlanningComplete`: Boolean indicating if all planning artifacts are complete. Older CLI versions expose the same value as `isComplete`.
   - `planningHome`, `changeRoot`, `artifactPaths`, and `actionContext`: path and scope context. Use these instead of assuming repo-local paths.

   The artifact ids and paths come from the active schema - do NOT assume them, and do NOT branch on hardcoded artifact names. Custom schemas must work unchanged.

   Schema artifact edits are limited to `artifactPaths.<id>.existingOutputPaths` - the concrete files that exist on disk, already glob-expanded for glob artifacts (e.g. `specs/**/*.md`). Do NOT write to `resolvedOutputPath`: for a glob artifact it is still the glob pattern, not a real file.

   **Fork context and additional editable files:**
   - Read project-level `openspec/GLOSSARY.md` and `openspec/VERIFY.md` if present. These are read-only context in this workflow.
   - Read `GLOSSARY.md` and `VERIFY.md` directly under the selected `changeRoot` if present. These two existing change-level files are explicit additions to the editable set, even when the schema does not list them. Do not create them or treat them as schema artifact ids.
   - Use the CLI-reported change root, not a guessed path. Missing extension files are valid; report a missing file only when the requested revision needs it.

3. **Understand the request**
   - Read available grill/explore decisions and Q&A summaries from the conversation or user-referenced notes. Carry only user-confirmed decisions and confirmed "Terms to Record" into revisions. Do not infer approval from an unanswered question or overwrite Q&A history; surface unresolved conflicts for clarification.
   - If the user asked for a specific revision ("the design now uses X"), that is the starting edit.
   - If they only said "update" / "make this coherent", treat it as a coherence review: read the existing artifacts and check them against each other for contradictions, gaps, and duplication.

4. **Read and reconcile**
   - Read the artifact(s) the request touches and the change's other existing artifacts.
   - Draft the requested edit for confirmation in step 5. Then check every other existing artifact against it - in ANY direction: an edit to a later artifact may require revising an earlier one, not only the other way around. Build order is a useful reading order, not a constraint on which artifacts may be revised.
   - Note everything that is now inconsistent, missing, or contradictory.
   - Revise only existing schema artifacts and the two existing change-level extension files identified in step 2. Do NOT create artifacts that don't exist yet, and do NOT invent new files under a glob artifact - note them and point the user to `/opsx-continue` to create schema artifacts.
   - **Terminology:** Check revised artifacts against both glossary levels. For confirmed new definitions or aliases, propose updates to the existing change-level glossary, grouped by capability using `- **Term** — Definition. Aliases: alias1, alias2.` Preserve unrelated terms. Explicitly identify conflicts with project-level definitions and resolve them with the user; do not silently choose a meaning. Leave project-level merging to `/opsx-archive`.
   - **Verification planning:** Check whether revised requirements, affected modules, or acceptance criteria change the verification scope or commands. Treat project- and change-level `VERIFY.md` sections as additive. Propose only justified edits to the existing change-level file, preserving its module headings and fenced command blocks. Do not duplicate inherited checks, use change-level edits to cancel project-level checks, execute commands from these files, or claim verification passed. Report needed project-level changes separately.
   - If a required change-level extension file is absent, report the proposed content as deferred setup; do not create it or suggest that `/opsx-continue` necessarily generates arbitrary extension files.
   - If the change is already coherent, say so and make no edits.

5. **Confirm and apply, one file at a time**
   - Show each proposed revision and why. Write only after the user confirms.
   - If the user rejects a revision, do not write it - leave that artifact unchanged.
   - For a substantial schema artifact rewrite, get that artifact's rules and template first (extension files use the fork conventions above, not invented artifact ids):
     ```bash
     openspec instructions "<artifact-id>" --change "<name>" --json
     ```

6. **Point to the next step (guidance only - NEVER act on it)**
   - Artifacts still missing -> suggest `/opsx-continue` to create them.
   - Change already implemented (tasks checked off / already applied) -> the code may no longer match the revised plan; suggest `/opsx-apply` to carry the delta into code.
   - Implementation is aligned but verification is affected or has not been rerun after relevant changes -> suggest `/opsx-verify`; do not run it here.
   - Everything done, implemented, and verified with no unresolved revision gaps -> suggest `/opsx-archive`.

**Output**

After each invocation, show:
- Which artifacts were revised (and which proposed revisions were rejected)
- Which confirmed grill/explore decisions were incorporated, glossary and verification-setting revisions, and any unresolved conflicts or deferred extension setup
- Anything deferred to `/opsx-continue` (not-yet-created artifacts or files)
- Where the change stands and the recommended next command

**Guardrails**
- Planning artifacts only - NEVER edit implementation code. If the revised plan implies code changes, stop and point to `/opsx-apply`.
- Use the artifact ids and paths reported by `openspec status`; never branch on hardcoded artifact names.
- Edit only the concrete files in `existingOutputPaths` plus the selected change's existing `GLOSSARY.md` and `VERIFY.md`; never write to a glob `resolvedOutputPath`. Project-level extension files and Q&A history remain read-only.
- Never create extension files, run verification commands, or automatically invoke apply, verify, sync, or archive. Recommend the next workflow only.
- Do not advance the build frontier: no new artifacts, no new files under glob artifacts - that is `/opsx-continue`'s job.
- Confirm every edit with the user before writing.
- If the request changes the change's *intent* rather than refining it, first verify whether the expanded-profile `/opsx-new` workflow is available. If it is, recommend starting fresh with `/opsx-new` (the "Update vs. Start Fresh" heuristic). If it is unavailable, ask for a distinct unused change name and recommend `openspec new change "<new-change-name>"` instead.
