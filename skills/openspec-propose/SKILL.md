---
name: openspec-propose
description: Propose a new change with all artifacts generated in one step. Use when the user wants to quickly describe what they want to build and get a complete proposal with design, specs, and tasks ready for implementation.
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.8.0"
---

Propose a new change - create the change and generate all artifacts in one step.

I'll create a change with artifacts:
- proposal.md (what & why)
- design.md (how)
- tasks.md (implementation steps)

When ready to implement, run /opsx-apply

---

**Input**: The argument after `/opsx-propose` is the change name (kebab-case), OR a description of what the user wants to build.

**Steps**

1. **If no input provided, ask what they want to build**

   Use the **AskUserQuestion tool** (open-ended, no preset options) to ask:
   > "What change do you want to work on? Describe what you want to build or fix."

   From their description, derive a kebab-case name (e.g., "add user authentication" → `add-user-auth`).

   **IMPORTANT**: Do NOT proceed without understanding what the user wants to build.

2. **Create the change directory**
   ```bash
   openspec new change "<name>"
   ```
   This creates a scaffolded change at `openspec/changes/<name>/` with `.openspec.yaml`.

3. **Get the artifact build order**
   ```bash
   openspec status --change "<name>" --json
   ```
   Parse the JSON to get:
   - `applyRequires`: array of artifact IDs needed before implementation (e.g., `["tasks"]`)
   - `artifacts`: list of all artifacts with their status and dependencies

4. **Create artifacts in sequence until apply-ready**

   **Read the glossaries first.** Read the glossaries once, before you create the first artifact: project-level `openspec/GLOSSARY.md` and change-level `openspec/changes/<name>/GLOSSARY.md`. Skip a file that does not exist. Use the defined terms with their exact spelling. Do NOT use an alias as the main term. If you find a project-specific term that is not in the glossaries, do NOT invent a definition; use the existing rule for terms confirmed with the user.

   Use the **TodoWrite tool** to track progress through the artifacts.

   Loop through artifacts in dependency order (artifacts with no pending dependencies first):

   a. **For each artifact that is `ready` (dependencies satisfied)**:
      - Get instructions:
        ```bash
        openspec instructions <artifact-id> --change "<name>" --json
        ```
      - The instructions JSON includes:
        - `context`: Project background (constraints for you - do NOT include in output)
        - `rules`: Artifact-specific rules (constraints for you - do NOT include in output)
        - `template`: The structure to use for your output file
        - `instruction`: Schema-specific guidance for this artifact type
        - `outputPath`: Where to write the artifact
        - `dependencies`: Completed artifacts to read for context
      - Read any completed dependency files for context
      - Create the artifact file using `template` as the structure
      - Apply `context` and `rules` as constraints - but do NOT copy them into the file
      - Show brief progress: "Created <artifact-id>"

   b. **Continue until all `applyRequires` artifacts are complete**
      - After creating each artifact, re-run `openspec status --change "<name>" --json`
      - Check if every artifact ID in `applyRequires` has `status: "done"` in the artifacts array
      - Stop when all `applyRequires` artifacts are done

   c. **If an artifact requires user input** (unclear context):
      - Use **AskUserQuestion tool** to clarify
      - Then continue with creation

   d. **Record confirmed terms to GLOSSARY (if any)**
      - After all `applyRequires` artifacts are complete, check the conversation for **confirmed project-specific terms**:
        - From a previous grill/explore session's "Terms to Record" output
        - Or terms confirmed during artifact creation
      - If any exist and are NOT already in project-level `openspec/GLOSSARY.md`, write them to `openspec/changes/<name>/GLOSSARY.md`:
        - Grouped by capability: `## <capability>`
        - Format: `- **Term** — Definition. Aliases: alias1、alias2。`
      - Only record terms confirmed with the user — never fabricate
      - If no confirmed terms, skip this step entirely

5. **Show final status**
   ```bash
   openspec status --change "<name>"
   ```

**Output**

After completing all artifacts, summarize:
- Change name and location
- List of artifacts created with brief descriptions
- What's ready: "All artifacts created! Ready for implementation."
- Terms recorded to GLOSSARY.md (N terms) or "(no terms recorded)"
- Prompt: "Run `/opsx-apply` to start implementing."
- Tip: Mention that custom verification is available via `openspec/VERIFY.md` — it defines per-repo checks that run during `/opsx-verify`.

**Artifact Creation Guidelines**

- Follow the `instruction` field from `openspec instructions` for each artifact type
- The schema defines what each artifact should contain - follow it
- Read dependency artifacts for context before creating new ones
- Use `template` as the structure for your output file - fill in its sections
- **IMPORTANT**: `context` and `rules` are constraints for YOU, not content for the file
  - Do NOT copy `<context>`, `<rules>`, `<project_context>` blocks into the artifact
  - These guide what you write, but should never appear in the output

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
- Write all artifact prose in the style defined by **Writing Style (ASD-STE100)**
- Create ALL artifacts needed for implementation (as defined by schema's `apply.requires`)
- Always read dependency artifacts before creating a new one
- If context is critically unclear, ask the user - but prefer making reasonable decisions to keep momentum
- If a change with that name already exists, ask if user wants to continue it or create a new one
- Verify each artifact file exists after writing before proceeding to next
