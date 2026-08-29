# OpenSpec (Fork / Enhanced)

> AI-assisted specification-driven development workflow. **Based on official skills with key improvements.**

**English** | **[繁體中文](README.zh.md)**

## Differences from Official Version

| Area                               | Official                           | This Fork                                     |
| ---------------------------------- | ---------------------------------- | --------------------------------------------- |
| **Reverse Questioning & Q&A Tree** | N/A                                | **Added openspec-grill with Q&A integration** |
| **Custom Verification**            | ❌ Not supported                    | **`VERIFY.md`** — declarative, scope-aware    |
| **Glossary**                       | ❌ Not supported                    | **`GLOSSARY.md`** — project + change-level domain terms |
| **Command Prefix**                 | `/openspec-*`                      | `/opsx:*` (shorter, namespace-friendly)       |
| **Store/Registry Support**         | Full (`--store`, `openspec store`) | ❌ Removed — single-repo simplicity            |

---

## Core Improvements

### 1. Openspec-Grill (Integration & Enhancement of grill-me)

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

### 2. VERIFY.md — Declarative Custom Verification

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

### 3. GLOSSARY.md — Domain Terminology Registry

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

---

## 🚀 Installation & Setup (Agent-Assisted Installation)

This Fork is specifically designed and optimized for AI Coding Agents. Simply copy and paste the **single-line command** matching your Agent tool, and your Agent will fetch this guide and follow the exact steps automatically:

| Tool | Single-Line Command for Agent |
| :--- | :--- |
| **Antigravity** | `Fetch and follow instructions from https://github.com/s16777216/openspec-improve#antigravity` |
| **Codex** | `Fetch and follow instructions from https://github.com/s16777216/openspec-improve#codex` |
| **OpenCode** | `Fetch and follow instructions from https://github.com/s16777216/openspec-improve#opencode` |

---

### <a id="antigravity"></a> Google Antigravity

When the Agent receives the `#antigravity` installation instruction, execute the following steps:

1. **Environment Check**:
   - Run `openspec --version` to verify CLI installation. If missing, run `npm install -g openspec@latest`.
2. **Initialize Project**:
   - Run `openspec init --no-animation --tools antigravity` in the project root to establish OpenSpec structure and configuration.
3. **Overwrite Enhanced Skills & Workflows**:
   - Copy `skills/` from this repo into `<project-root>/.agent/skills/` (or `.agents/skills/`), overwriting existing files.
   - Copy `commands/` from this repo into `<project-root>/.agent/workflows/` (enabling `/opsx-*` slash commands), overwriting existing files.
   - *(Optional Global)*: If global installation is requested, copy into `~/.gemini/config/skills/` and `~/.gemini/config/global_workflows/`.
   - If not present, create project-level starter templates for `openspec/VERIFY.md` and `openspec/GLOSSARY.md` following the [Starter Template Specifications](#templates) below.

---

### <a id="codex"></a> Codex

When the Agent receives the `#codex` installation instruction, execute the following steps:

1. **Environment Check**:
   - Run `openspec --version` to verify CLI installation. If missing, run `npm install -g openspec@latest`.
2. **Initialize Project**:
   - Run `openspec init --no-animation --tools codex` in the project root to establish OpenSpec structure and configuration.
3. **Overwrite Enhanced Skills**:
   - Copy `skills/` from this repo into `<project-root>/.agents/skills/`, overwriting existing files.
   - Codex operates primarily via Skills (e.g. `$openspec-propose`).
   - If not present, create project-level starter templates for `openspec/VERIFY.md` and `openspec/GLOSSARY.md` following the [Starter Template Specifications](#templates) below.

---

### <a id="opencode"></a> OpenCode

When the Agent receives the `#opencode` installation instruction, execute the following steps:

1. **Environment Check**:
   - Run `openspec --version` to verify CLI installation. If missing, run `npm install -g openspec@latest`.
2. **Initialize Project**:
   - Run `openspec init --no-animation --tools opencode` in the project root to establish OpenSpec structure and configuration.
3. **Overwrite Enhanced Commands & Skills**:
   - Copy `commands/` from this repo into `<project-root>/.opencode/commands/` (enabling `/opsx-*` slash commands), overwriting existing files.
   - Copy `skills/` from this repo into `<project-root>/.opencode/skills/`, overwriting existing files.
   - If not present, create project-level starter templates for `openspec/VERIFY.md` and `openspec/GLOSSARY.md` following the [Starter Template Specifications](#templates) below.

---

### <a id="templates"></a> 📄 Starter Template Specifications (VERIFY.md & GLOSSARY.md)

When initializing, the Agent should create the following files under `openspec/` if they do not exist:

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

---

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

## License

MIT