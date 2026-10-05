# OpenSpec (Fork / Enhanced)

> AI-assisted specification-driven development workflow. **Based on official skills with key improvements.**

**English** | **[繁體中文](README.zh.md)**

## Differences from Official Version

| Area                               | Official                           | This Fork                                               |
| ---------------------------------- | ---------------------------------- | ------------------------------------------------------- |
| **Reverse Questioning & Q&A Tree** | N/A                                | **Added openspec-grill with Q&A integration**           |
| **Custom Verification**            | ❌ Not supported                    | **`VERIFY.md`** — declarative, scope-aware              |
| **Glossary**                       | ❌ Not supported                    | **`GLOSSARY.md`** — project + change-level domain terms |
| **Store/Registry Support**         | Full (`--store`, `openspec store`) | ❌ Removed — single-repo simplicity                      |

---

## Core Improvements

### 1. Openspec-Status

**Quick status overview** — `/opsx-status`

A **concise overview** of all active OpenSpec changes, showing:
- Each change's stage (ideation/in-progress/ready-for-review)
- Task progress (e.g. 3/7 tasks)
- Next action and blockers
- Single recommended next step

**Example output:**
```
## OpenSpec Status

3 active changes:

### add-auth-flow — "OAuth login for the API"
- Stage: in-progress  •  Progress: 3/7 tasks
- Next: task 4 "Wire up refresh token rotation"

### fix-db-migration — "Repair flaky migration ordering"
- Stage: ready-for-review  •  Progress: 7/7 tasks
- Next: run `/opsx-verify fix-db-migration`
```

Perfect for your morning "what's in flight?" check.

---

### 2. Openspec-Grill (Integration & Enhancement of grill-me)

#### New Features

- **Reverse Questioning** — proactively clarifies requirements before starting
- **Q&A Tree** — structured, traceable discussion flow
- **Specs Awareness** — directly references and discusses spec files
- **Design Tree Integration** — auto-generates `design.md`
- **State Tracking** — records Q&A state to avoid repetition

#### Output Results

After discussion completes, automatically generates:
- ✅ `design.md` — structured design decisions
- ✅ `tasks.md` — executable task list
- ✅ `specs/` — spec file updates (if needed)

---

### 3. VERIFY.md — Declarative Custom Verification

````markdown
# Verification

## my-repo

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
````

**How it works:**
- Agent reads `VERIFY.md` (project-level + change-level, **additive**)
- Cross-references change's affected files to determine **which repo sections are in scope**
- Executes only in-scope sections
- Skipped sections noted in report: `"Skipped shared-lib (not in change scope)"`

### Additive Change-Level Config

```
Project VERIFY.md:     lint + typecheck + test for all repos
Change VERIFY.md:      + security-scan for auth repo only
Result:                Both run, merged in report
```

---

### 4. GLOSSARY.md — Domain Terminology Registry

Central registry for project-specific terms (jargon, abbreviations, domain vocabulary).

**Format** — `term + definition + aliases` in a pure list (no manual ADDED/MODIFIED flags; the diff happens at archive time):

```markdown
# Glossary

## auth

- **Principal** — Authenticated request principal. Aliases: user、account。
```

**How it works:**
- **Create (grill)**: `/opsx-grill` detects project-specific terms during the conversation, confirms definitions + aliases with the user, then writes them to `openspec/changes/<name>/GLOSSARY.md`
- **Archive**: `/opsx-archive` merges the change-level `GLOSSARY.md` into project-level `openspec/GLOSSARY.md` — new terms appended, existing terms overwritten — then deletes the change-level file

Project-level `openspec/GLOSSARY.md` is the single source of truth; change-level files are additive.

> **⚠️ Warning: `openspec init` overwrites fork extensions.**
> The official `openspec init` does **not** understand this fork's extensions (GLOSSARY, VERIFY). Running it with a tool selection rewrites the skills back to the official versions, wiping out GLOSSARY/VERIFY integrations. `/opsx-setup` avoids this by using `--tools none`.
> **If you ever run `openspec init` with a tool selection, restore the skills immediately by running `npx skills add s16777216/openspec-improve` again.**

---

### 5. ASD-STE100 Writing Style

`/opsx-propose`, `/opsx-continue`, `/opsx-ff`, and `/opsx-update` write artifact prose in [Simplified Technical English (ASD-STE100)](https://www.asd-ste100.org/) style, so that each sentence has only one reading.

- Short sentences, active voice, commands for tasks
- One word, one meaning — terms follow project- and change-level `GLOSSARY.md`
- `MUST` / `MUST NOT` for obligations; measurable values instead of vague words
- Technical names, templates, and OpenSpec structure stay unchanged

**Languages:** the artifact keeps the language of the request. English follows ASD-STE100 directly. For other languages the language-neutral rules apply, which is not formal ASD-STE100. The rules are prompt instructions only; nothing checks compliance automatically.

---

## 🚀 Installation & Setup

Install with the [`skills`](https://github.com/vercel-labs/skills) CLI. It copies every skill under `skills/` into each agent's own skills directory.

1. **Install the enhanced skills**:
   ```bash
   npx skills add s16777216/openspec-improve
   ```
   - Add `-g` to install globally instead of per-project; add `-a <agent>` (for example `-a claude-code -a codex`) to skip the interactive agent selection.
2. **Ask your agent to run the setup skill**: `/opsx-setup` (Codex: `$openspec-setup`). It is safe to run again at any time. It:
   - checks the OpenSpec CLI and offers to install it (`npm install -g openspec@latest`);
   - runs `openspec init --no-animation --tools none` if `openspec/` does not exist;
   - creates `openspec/VERIFY.md` and `openspec/GLOSSARY.md` from the [Starter Template Specifications](#templates) if they are missing.

   It asks for confirmation before each step and never overwrites an existing file. `/opsx-status`, `/opsx-new`, `/opsx-propose`, and `/opsx-onboard` also suggest `/opsx-setup` when one of these files is missing.
   - **Windows users**: If PowerShell blocks `openspec` due to execution policy, the agent uses `openspec.cmd` instead.

### Invoking skills

Each workflow is a skill named `openspec-*`. The `/opsx-*` names used in this document are shorthand; see the **Skill** column in the [command reference](#command-reference) for the actual names.

| Tool | Invocation |
| :--- | :--- |
| **Claude Code** | `/openspec-propose` |
| **Codex** | `$openspec-propose` |
| **Antigravity / OpenCode** | Mention the skill by name, or let the agent pick it up from its description |

---

### <a id="templates"></a> 📄 Starter Template Specifications (VERIFY.md & GLOSSARY.md)

`/opsx-setup` creates the following files under `openspec/` if they do not exist (the templates are embedded in `skills/openspec-setup/SKILL.md`; keep both in sync):

1. **`openspec/VERIFY.md`** (Declarative Custom Verification)
   ````markdown
   # Verification

   ## <project-or-module-name>

   ```bash
   # Add project-specific lint, typecheck, or test commands (e.g. npm test / cargo test)
   npm test
   ```
   ````

2. **`openspec/GLOSSARY.md`** (Domain Terminology Registry)
   ````markdown
   # Glossary

   ## core

   - **ExampleTerm** — Definition of example domain term. Aliases: alias1, alias2.
   ````

---

## Skills (Agent Workflows)

```
     |
    ●-- explore / grill : requirements discussion, design tree Q&A
     |
     |
    ●-- propose : generate proposal from discussion
     |
     |
    ●-- apply : implement based on proposal
     |
     |
    ●-- verify : generate verification report from proposal & implementation
     |
     |
    ●-- archive : archive the proposal
     |
     v
```

### <a id="command-reference"></a> Complete Command Reference

| Command | Skill | Description |
| :--- | :--- | :--- |
| `/opsx-setup` | `openspec-setup` | Set up OpenSpec in a project: CLI check, `openspec init`, starter `VERIFY.md` and `GLOSSARY.md` |
| `/opsx-status` | `openspec-status` | Quick overview of all active changes and next steps |
| `/opsx-explore` | `openspec-explore` | Think through problems before/during work |
| `/opsx-grill` | `openspec-grill` | Design tree questioning to sharpen decisions |
| `/opsx-new` | `openspec-new-change` | Start a new change, step through artifacts one at a time |
| `/opsx-continue` | `openspec-continue-change` | Continue working on an existing change |
| `/opsx-ff` | `openspec-ff-change` | Fast-forward: create all artifacts at once |
| `/opsx-propose` | `openspec-propose` | Create a change and generate all artifacts |
| `/opsx-update` | `openspec-update-change` | Revise existing planning artifacts and keep them consistent; no code edits |
| `/opsx-apply` | `openspec-apply-change` | Implement tasks from a change |
| `/opsx-verify` | `openspec-verify-change` | Verify implementation matches artifacts |
| `/opsx-sync` | `openspec-sync-specs` | Sync delta specs from a change to main specs |
| `/opsx-archive` | `openspec-archive-change` | Archive a completed change |
| `/opsx-bulk-archive` | `openspec-bulk-archive-change` | Archive multiple completed changes at once |
| `/opsx-onboard` | `openspec-onboard` | Guided onboarding through a complete workflow cycle |

---

### Revising an Existing Plan

Use `/opsx-update <change-name>` (or `$openspec-update-change` in Codex) when requirements change, grill/explore yields new decisions, or existing planning artifacts contradict one another. It proposes revisions to existing artifacts for user confirmation; it does not create missing artifacts or edit implementation code.

Fork integration includes confirmed grill/explore decisions and terminology checks against project- and change-level `GLOSSARY.md`. It can propose edits to an existing change-level glossary and `VERIFY.md` when the revised plan affects terms or verification scope. Project-level files remain read-only; verification settings are additive, and checks run through `/opsx-verify`, not update. Missing extension files are reported as deferred setup rather than created automatically.

Use `/opsx-continue` or `/opsx-ff` for missing artifacts, `/opsx-apply` to implement the revised plan, and `/opsx-sync` to merge delta specs into main specs. The terminal command `openspec update` separately refreshes generated skills from the installed CLI.

## Directory Structure

```
openspec/
├── changes/
│   └── add-user-auth/
│       ├── .openspec.yaml
│       ├── proposal.md
│       ├── specs/
│       │   └── auth/spec.md
│       ├── design.md
│       ├── tasks.md
│       ├── VERIFY.md          # change-level (additive)
│       └── GLOSSARY.md        # change-level terms (merged at archive)
├── specs/
│   └── auth/spec.md           # main specs
├── VERIFY.md                  # project-level
└── GLOSSARY.md                # project-level (single source of truth)
```

---

## Compatibility

- **OpenSpec CLI**: Tested against version **1.8.0**. Minimum supported version may differ; check with `openspec --version`.
- **Platforms**: macOS, Linux, Windows (use `openspec.cmd` on Windows if PowerShell execution policy blocks `.ps1` shims).

---

## Maintenance

### Adding a New Workflow

To add a new workflow skill:

1. Create `skills/openspec-<action>/SKILL.md` with the workflow content.
2. Ensure steps, guardrails, and output contracts are consistent with the other skills.
3. Add the skill to the reference tables in both README.md and README.zh.md.
4. Run the consistency checks (see below) (currently 15 skills).

### Consistency Checks

Run these read-only checks to verify project health:

```bash
# Scan for inconsistent /opsx: references
grep -r '/opsx:' skills/ README.md README.zh.md

# Verify YAML frontmatter in all skills
grep -l 'generatedBy' skills/*/SKILL.md

# Check Markdown fence validity
# (use a markdown linter or manual review)
```

---

## License

MIT
