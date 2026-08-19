# OpenSpec

> AI-assisted specification-driven development workflow. Artifacts pipeline + agent skills.

## Overview

OpenSpec is a structured workflow system for AI-assisted development. It enforces a clear pipeline:

```
new → continue → apply → verify → archive
```

Each step produces explicit artifacts (proposal, specs, design, tasks) that serve as both documentation and implementation contracts.

### Workflow Timeline

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         OPEN SPEC WORKFLOW TIMELINE                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐         │
│   │   NEW    │────▶│ CONTINUE │────▶│  APPLY   │────▶│  VERIFY  │────▶ARCHIVE│
│   └──────────┘     └──────────┘     └──────────┘     └──────────┘         │
│        │               │               │               │                   │
│        ▼               ▼               ▼               ▼                   │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                        ARTIFACTS PRODUCED                            │   │
│   ├──────────────┬──────────────┬──────────────┬────────────────────────┤   │
│   │  proposal.md │  specs/*.md  │  design.md   │  tasks.md              │   │
│   │  (what/why)  │  (require-   │  (how/arch)  │  (checklist)           │   │
│   │              │   ments)     │              │                        │   │
│   └──────────────┴──────────────┴──────────────┴────────────────────────┘   │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                        OPTIONAL / PARALLEL                           │   │
│   ├──────────────────────┬──────────────────────┬──────────────────────┤   │
│   │ /opsx:propose        │ /opsx:explore        │ /opsx:grill          │   │
│   │ (all artifacts       │ (thinking partner    │ (design tree         │   │
│   │  at once)            │  mode)               │  questioning)        │   │
│   └──────────────────────┴──────────────────────┴──────────────────────┘   │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                        SYNC / MAINTAIN                               │   │
│   ├──────────────────────┬──────────────────────┬──────────────────────┤   │
│   │ /opsx:sync           │ /opsx:verify         │ /opsx:ff             │   │
│   │ (delta→main spec)    │ (custom VERIFY.md)   │ (fast-forward)       │   │
│   └──────────────────────┴──────────────────────┴──────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Core Concepts

| Concept      | Description                                                                                |
| ------------ | ------------------------------------------------------------------------------------------ |
| **Change**   | A unit of work with its own directory at `openspec/changes/<name>/`                        |
| **Artifact** | Structured markdown files: `proposal.md`, `specs/`, `design.md`, `tasks.md`                |
| **Schema**   | Workflow definition (e.g., `spec-driven`) that dictates artifact sequence and dependencies |
| **CLI**      | `openspec` command manages state, generates templates, tracks progress                     |

## Quick Start

```bash
# Initialize OpenSpec in your project
openspec init

# Start a new change
/opsx:new add-user-auth

# Continue building artifacts (proposal → specs → design → tasks)
/opsx:continue

# Implement the tasks
/opsx:apply

# Verify before archive
/opsx:verify

# Archive completed change
/opsx:archive
```

## Skills (Agent Workflows)

Located in `openspec-src/skills/` — these are the agent-executable workflows:

| Skill                      | Command          | Purpose                            |
| -------------------------- | ---------------- | ---------------------------------- |
| `openspec-new-change`      | `/opsx:new`      | Create new change directory        |
| `openspec-continue-change` | `/opsx:continue` | Create next artifact               |
| `openspec-propose`         | `/opsx:propose`  | Generate all artifacts in one step |
| `openspec-apply-change`    | `/opsx:apply`    | Implement tasks                    |
| `openspec-verify-change`   | `/opsx:verify`   | Three-dimension verification       |
| `openspec-archive-change`  | `/opsx:archive`  | Archive completed change           |
| `openspec-sync-specs`      | `/opsx:sync`     | Delta spec → main spec             |
| `openspec-explore`         | `/opsx:explore`  | Thinking partner mode              |
| `openspec-grill`           | `/opsx:grill`    | Design tree questioning            |
| `openspec-onboard`         | `/opsx:onboard`  | Project onboarding                 |
| `openspec-ff-change`       | `/opsx:ff`       | Fast-forward through artifacts     |

## Verification System

### Built-in Three Dimensions

1. **Completeness** — Task completion rate + spec coverage
2. **Correctness** — Requirements implemented + scenarios covered
3. **Coherence** — Design adherence + code pattern consistency

### Custom Verification (VERIFY.md)

Add project-specific checks via `VERIFY.md`:

```markdown
# Verification

## my-repo

```bash
npm run lint
npm run typecheck
npm test
```
```

**Location**: `openspec/VERIFY.md` (project) + `openspec/changes/<name>/VERIFY.md` (change, additive)

**Execution**: Agent reads VERIFY.md, determines which sections are in scope based on the change's affected files, runs only relevant commands.

## Directory Structure

```
openspec/
├── changes/
│   └── add-user-auth/
│       ├── .openspec.yaml
│       ├── proposal.md
│       ├── specs/
│       │   └── auth/
│       │       └── spec.md
│       ├── design.md
│       ├── tasks.md
│       └── VERIFY.md          # change-level verification
├── specs/
│   └── auth/
│       └── spec.md            # main specs (synced via /opsx:sync)
└── VERIFY.md                  # project-level verification
```

## Comparison

See [openspec-vs-mattpocock.md](openspec-vs-mattpocock.md) for detailed comparison with Matt Pocock's workflow system.

## Installation

```bash
# Install openspec CLI
# (see openspec-src/ for source)

# Install skills for your agent
# Copy openspec-src/skills/ to your agent's skill directory
```

## License

MIT