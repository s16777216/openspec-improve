# OpenSpec (Fork / Enhanced)

> AI-assisted specification-driven development workflow. **Based on official skills with key improvements.**

**English** | **[繁體中文](README.zh.md)**

## Differences from Official Version

| Area                               | Official                           | This Fork                                     |
| ---------------------------------- | ---------------------------------- | --------------------------------------------- |
| **Reverse Questioning & Q&A Tree** | N/A                                | **Added openspec-grill with Q&A integration** |
| **Custom Verification**            | ❌ Not supported                    | **`VERIFY.md`** — declarative, scope-aware    |
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
│       └── VERIFY.md          # change-level (additive)
├── specs/
│   └── auth/spec.md           # main specs
└── VERIFY.md                  # project-level
```

---

## License

MIT