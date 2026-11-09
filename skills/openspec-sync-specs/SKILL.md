---
name: openspec-sync-specs
description: Sync a change's delta specs to main specs and refresh local SPEC.md files from implemented code. Use to integrate specs or finish pending local documentation updates without archiving the change.
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.8.0"
---

Sync delta specs from a change to main specs and refresh the affected directories' local `SPEC.md` folder guides (with user approval).

This is an **agent-driven** operation - you will read delta specs and directly edit main specs to apply the changes. This allows intelligent merging (e.g., adding a scenario without copying the entire requirement).

**Input**: Optionally specify a change name after `openspec-sync-specs` (e.g., `openspec-sync-specs add-auth`). If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

**Steps**

1. **If no change name provided, prompt for selection**

   Run `openspec list --json` to get available changes. Use the **AskUserQuestion tool** to let the user select.

   Show changes that have delta specs (under `specs/` directory) or pending local `SPEC.md` updates recorded in their artifacts.

   **IMPORTANT**: Do NOT guess or auto-select a change. Always let the user choose.

2. **Find delta specs**

   Recursively find `spec.md` files under `openspec/changes/<name>/specs/`. Include both `specs/<capability>/spec.md` and `specs/<domain>/<capability>/spec.md` (and deeper domain paths). Ignore a `spec.md` directly in the `specs/` root. The capability ID is the full relative directory path, such as `identity/authentication`, not just `authentication`.

   Each delta spec file contains sections like:
   - `## ADDED Requirements` - New requirements to add
   - `## MODIFIED Requirements` - Changes to existing requirements
   - `## REMOVED Requirements` - Requirements to remove
   - `## RENAMED Requirements` - Requirements to rename (FROM:/TO: format)

   Also read the change's design/tasks artifacts for affected repos/modules and pending local `SPEC.md` paths. If no delta specs exist, continue with local documentation updates when needed; stop only when neither kind of update is needed.

3. **For each delta spec, apply changes to main specs**

   For each capability ID `<capability-id>` with a delta spec at `openspec/changes/<name>/specs/<capability-id>/spec.md`:

   a. **Read the delta spec** to understand the intended changes

   b. **Read the main spec** at `openspec/specs/<capability-id>/spec.md` (may not exist yet). Preserve the full relative path; never match by the last path segment alone.

   c. **Apply changes intelligently**:

      **ADDED Requirements:**
      - If requirement doesn't exist in main spec → add it
      - If requirement already exists → update it to match (treat as implicit MODIFIED)

      **MODIFIED Requirements:**
      - Find the requirement in main spec
      - Apply the changes - this can be:
        - Adding new scenarios (don't need to copy existing ones)
        - Modifying existing scenarios
        - Changing the requirement description
      - Preserve scenarios/content not mentioned in the delta

      **REMOVED Requirements:**
      - Remove the entire requirement block from main spec

      **RENAMED Requirements:**
      - Find the FROM requirement, rename to TO

   d. **Create new main spec** if capability doesn't exist yet:
      - Create `openspec/specs/<capability-id>/spec.md` (including parent directories)
      - Add Purpose section (can be brief, mark as TBD)
      - Add Requirements section with the ADDED requirements

4. **Refresh local folder guides (`SPEC.md`)**

   Perform this step even when main specs already match the delta. Use the change's artifacts and actual code to identify the directories whose files this change added, removed, renamed, or re-purposed, and the capability IDs implemented there.

   A `SPEC.md` is a **folder guide**: it lets an agent quickly learn what a directory contains and what it is for. Each file describes only its own directory, stays short (about 30 lines at most), and has two required sections and one optional section:
   - **Purpose** - 1-2 sentences on what this directory is responsible for.
   - **Contents** - one line per file or subfolder stating its responsibility. For a subfolder that has its own `SPEC.md`, write one line that points to it; do not expand it.
   - **Capabilities** (optional) - include only when this directory is primarily responsible for specific capabilities: the full capability ID with a link or stable path to the central spec, one line each. Do not enumerate every capability the directory or module touches; if the list would be long (for example at a module root), omit the section and let the subdirectories that own those capabilities list them. Omit the section entirely when nothing applies; never write an empty section or "N/A".

   Never put these in a `SPEC.md`: flows or step-by-step behavior, locking/rollback details, numeric limits, dependency versions, test inventories, or copies of requirement/scenario text. They go stale and pollute agent context; they belong in code, the central spec, or design.

   **Where to create one**: a repo/module root gets one. For any other directory, propose a new `SPEC.md` only when it has a distinct responsibility that the parent's one-line Contents entry cannot convey, and say why in the approval request; the user decides. Otherwise the parent's Contents covers it. Do not create a file in every directory.

   **What to update**: only the `SPEC.md` of directories touched by this change. Leave every other `SPEC.md` alone and preserve unrelated content.

   **User approval is required** before creating or modifying any `SPEC.md`. For each file, show its path, whether it is a create or an update, and the proposed content (full text for a new file, a diff for an existing one). Then use the **AskUserQuestion tool** to let the user approve, request edits, or skip. Write only approved content. A skipped file stays unchanged and is reported as "skipped by user", not as a failure. This applies on every run, including when invoked from archive, and never write `SPEC.md` without approval.

   - Describe only the implementation that exists. If a repo is inaccessible or a portion is not implemented, leave that local content unchanged and record its path, capability ID, and reason as pending in the change's design artifact (or a separate notes section in the tasks artifact when no design exists). Do not turn these notes into incomplete implementation task checkboxes.
   - Resolve recorded pending entries when their local updates are completed. A later sync must revisit them even if the main spec needs no further edits. Report main-spec integration and local documentation completion separately.

5. **Show summary**

   After applying all changes, summarize:
   - Which capabilities were updated
   - What changes were made (requirements added/modified/removed/renamed)
   - Local `SPEC.md` files updated or already current, and pending paths with reasons

**Delta Spec Format Reference**

```markdown
## ADDED Requirements

### Requirement: New Feature
The system SHALL do something new.

#### Scenario: Basic case
- **WHEN** user does X
- **THEN** system does Y

## MODIFIED Requirements

### Requirement: Existing Feature
#### Scenario: New scenario to add
- **WHEN** user does A
- **THEN** system does B

## REMOVED Requirements

### Requirement: Deprecated Feature

## RENAMED Requirements

- FROM: `### Requirement: Old Name`
- TO: `### Requirement: New Name`
```

**Key Principle: Intelligent Merging**

Unlike programmatic merging, you can apply **partial updates**:
- To add a scenario, just include that scenario under MODIFIED - don't copy existing scenarios
- The delta represents *intent*, not a wholesale replacement
- Use your judgment to merge changes sensibly

**Output On Success**

```
## Specs Synced: <change-name>

Updated main specs:

**<capability-1>**:
- Added requirement: "New Feature"
- Modified requirement: "Existing Feature" (added 1 scenario)

**<capability-2>**:
- Created new spec file
- Added requirement: "Another Feature"

Local implementation specs:
- Updated (approved): api/SPEC.md (identity/authentication)
- Skipped by user: api/src/auth/SPEC.md
- Pending: web/SPEC.md (identity/authentication; frontend not implemented)

Main specs are now updated. The change remains active - finish implementation and pending local updates before archive.
```

**Guardrails**
- Read both delta and main specs before making changes
- Preserve existing content not mentioned in delta
- Main specs express intended behavior; local `SPEC.md` is a short folder guide of actual code. Never document planned code as implemented.
- Never create or modify a `SPEC.md` without showing the proposed content and getting user approval first
- If something is unclear, ask for clarification
- Show what you're changing as you go
- The operation should be idempotent - running twice should give same result
