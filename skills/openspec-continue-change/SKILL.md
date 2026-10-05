---
name: openspec-continue-change
description: Continue working on an OpenSpec change by creating the next artifact. Use when the user wants to progress their change, create the next artifact, or continue their workflow.
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.8.0"
---

Continue working on a change by creating the next artifact.

**Input**: Optionally specify a change name after `/opsx-continue` (e.g., `/opsx-continue add-auth`). If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

**Steps**

1. **If no change name provided, prompt for selection**

   Run `openspec list --json` to get available changes sorted by most recently modified. Then use the **AskUserQuestion tool** to let the user select which change to work on.

   Present the top 3-4 most recently modified changes as options, showing:
   - Change name
   - Schema (from `schema` field if present, otherwise "spec-driven")
   - Status (e.g., "0/5 tasks", "complete", "no tasks")
   - How recently it was modified (from `lastModified` field)

   Mark the most recently modified change as "(Recommended)" since it's likely what the user wants to continue.

   **IMPORTANT**: Do NOT guess or auto-select a change. Always let the user choose.

2. **Check current status**
   ```bash
   openspec status --change "<name>" --json
   ```
   Parse the JSON to understand current state. The response includes:
   - `schemaName`: The workflow schema being used (e.g., "spec-driven")
   - `artifacts`: Array of artifacts with their status ("done", "ready", "blocked")
   - `isComplete`: Boolean indicating if all artifacts are complete

3. **Act based on status**:

   ---

   **If all artifacts are complete (`isComplete: true`)**:
   - Congratulate the user
   - Show final status including the schema used
   - Suggest: "All artifacts created! You can now implement this change with `/opsx-apply` or archive it with `/opsx-archive`."
   - STOP

   ---

   **If artifacts are ready to create** (status shows artifacts with `status: "ready"`):
   - Pick the FIRST artifact with `status: "ready"` from the status output
   - Get its instructions:
     ```bash
     openspec instructions <artifact-id> --change "<name>" --json
     ```
   - Parse the JSON. The key fields are:
     - `context`: Project background (constraints for you - do NOT include in output)
     - `rules`: Artifact-specific rules (constraints for you - do NOT include in output)
     - `template`: The structure to use for your output file
     - `instruction`: Schema-specific guidance
     - `outputPath`: Where to write the artifact
     - `dependencies`: Completed artifacts to read for context
   - **Create the artifact file**:
     - Read any completed dependency files for context
     - Read the glossaries before you write: project-level `openspec/GLOSSARY.md` and change-level `openspec/changes/<name>/GLOSSARY.md`. Skip a file that does not exist. Use the defined terms with their exact spelling and do NOT use an alias as the main term. Do NOT invent a definition for a new term; use the rule for terms confirmed with the user below.
     - Use `template` as the structure - fill in its sections
     - Apply `context` and `rules` as constraints when writing - but do NOT copy them into the file
     - Write to the output path specified in instructions
   - **If the created artifact is the proposal** (the change-describing artifact):
     - Check the conversation for **confirmed project-specific terms** (from a previous grill/explore "Terms to Record" output or confirmed during this session)
     - If any exist and are NOT already in project-level `openspec/GLOSSARY.md`, write them to `openspec/changes/<name>/GLOSSARY.md`:
       - Grouped by capability: `## <capability>`
       - Format: `- **Term** — Definition. Aliases: alias1、alias2。`
     - Only record terms confirmed with the user — never fabricate; skip if none
   - Show what was created and what's now unlocked
   - STOP after creating ONE artifact

   ---

   **If no artifacts are ready (all blocked)**:
   - This shouldn't happen with a valid schema
   - Show status and suggest checking for issues

4. **After creating an artifact, show progress**
   ```bash
   openspec status --change "<name>"
   ```

**Output**

After each invocation, show:
- Which artifact was created
- Schema workflow being used
- Current progress (N/M complete)
- What artifacts are now unlocked
- Prompt: "Run `/opsx-continue` to create the next artifact"

**Artifact Creation Guidelines**

The artifact types and their purpose depend on the schema. Use the `instruction` field from the instructions output to understand what to create.

Common artifact patterns:

**spec-driven schema** (proposal → specs → design → tasks):
- **proposal.md**: Ask user about the change if not clear. Fill in Why, What Changes, Capabilities, Impact.
  - The Capabilities section is critical - each capability listed will need a spec file.
- **specs/<capability>/spec.md**: Create one spec per capability listed in the proposal's Capabilities section (use the capability name, not the change name).
- **design.md**: Document technical decisions, architecture, and implementation approach.
- **tasks.md**: Break down implementation into checkboxed tasks.

For other schemas, follow the `instruction` field from the CLI output.

Common artifact patterns:

**spec-driven schema** (proposal → specs → design → tasks):
- **proposal.md**: Ask user about the change if not clear. Fill in Why, What Changes, Capabilities, Impact.
  - The Capabilities section is critical - each capability listed will need a spec file.
- **specs/<capability>/spec.md**: Create one spec per capability listed in the proposal's Capabilities section (use the capability name, not the change name).
- **design.md**: Document technical decisions, architecture, and implementation approach.
- **tasks.md**: Break down implementation into checkboxed tasks.

For other schemas, follow the `instruction` field from the CLI output.

**Writing Style (ASD-STE100)**

Write the prose of every artifact that you create in Simplified Technical English (ASD-STE100). The goal is text that has only one reading.

Language:
- Write in the language of the user's request and of the existing artifacts. Do NOT translate.
- English: follow ASD-STE100 directly.
- Other languages: ASD-STE100 has no official version. Apply the language-neutral rules below and keep the same intent.

Rules:
- **One idea per sentence.** Keep requirement and description sentences short (English: 25 words or fewer). Keep task and procedure sentences shorter (English: 20 words or fewer). Keep paragraphs to 6 sentences or fewer.
- **Use the active voice.** Name the actor: "The API returns an error", not "An error is returned". Write tasks as commands: "Add the field", not "The field should be added".
- **One word, one meaning.** Use one term for one concept. Do NOT use synonyms for variety. Use the exact spelling of the glossary terms that you read (project-level and change-level `GLOSSARY.md`).
- **Use plain words.** Avoid idioms, phrasal verbs, and words with many meanings. Keep articles (English: "the", "a").
- **State obligation clearly.** Use `MUST` for a mandatory requirement and `MUST NOT` for a prohibition. Do NOT use "should", "could", "might", or "may" in a requirement. Keep the `SHALL`/`MUST` keyword that OpenSpec requires.
- **Be measurable.** Replace vague words ("fast", "many", "soon") with a number and a unit, or with a condition that a test can check.
- **Limit noun strings.** Do NOT stack more than 3 nouns in a row. Rewrite with a verb or a preposition.
- **Keep the same sentence form for the same kind of statement.** Write every scenario with the same `WHEN`/`THEN` pattern.

Scope:
- Technical names (API names, commands, file paths, identifiers, error codes) stay as they are. Write them in code format and spell them the same way each time.
- The template structure, headings, and the artifact rules from `openspec instructions` have priority over these style rules. Apply the style rules to the prose only.
- Do NOT write "ASD-STE100" or "STE" in the artifact. Do NOT claim the text is certified or compliant.

**Guardrails**
- Write the artifact prose in the style defined by **Writing Style (ASD-STE100)**
- Create ONE artifact per invocation
- Always read dependency artifacts before creating a new one
- Never skip artifacts or create out of order
- If context is unclear, ask the user before creating
- Verify the artifact file exists after writing before marking progress
- Use the schema's artifact sequence, don't assume specific artifact names
- **IMPORTANT**: `context` and `rules` are constraints for YOU, not content for the file
  - Do NOT copy `<context>`, `<rules>`, `<project_context>` blocks into the artifact
  - These guide what you write, but should never appear in the output
