---
name: openspec-status
description: Show a concise overview of all active OpenSpec changes and their execution status, plus recommended next steps. Use when the user wants a quick status check of what's in flight, how far along each change is, and what to do next.
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.8.0"
---

Give the user a **concise battlefield overview** of all active OpenSpec changes — what's in flight, how far along each is, and what to do next. This is a *status report*, not explore mode: be quick, structured, and actionable.

**This is a report-only command.** You read and summarize. NEVER implement, apply, archive, or modify any artifact. You may only *recommend* actions.

---

## Steps

### 1. Gather all active changes

Run:
```bash
openspec list --json
```

If that errors (no `openspec/` dir / no changes), fall back to scanning:
```bash
openspec/changes/*/   # any directory here = an active change
```

Also check project-level context files if present:
- `openspec/GLOSSARY.md` — project terms
- `openspec/VERIFY.md` — project verification config

### 2. No changes → report that and stop

If there are zero active changes, say so plainly and stop. Example:

```
## OpenSpec Status

No active changes right now.

The workspace is clean. Use `/opsx-propose` to start something new.
```

Do NOT invent work or force suggestions.

### 3. For each active change, read its artifacts

For every change found, read:
- `openspec/changes/<name>/proposal.md` — the *why* and *what*
- `openspec/changes/<name>/tasks.md` — task checklist → compute progress
- `openspec/changes/<name>/design.md` — skim for decisions (optional, if quick)
- `openspec/changes/<name>/VERIFY.md` — change-level verification (optional)
- `openspec/changes/<name>/GLOSSARY.md` — change-level terms (optional)

### 4. Compute a lightweight status per change

For each change, determine:
- **Name** (kebab-case) — and a one-line gist from `proposal.md`
- **Progress** — count checked vs total tasks from `tasks.md` (e.g. `3/7`)
- **Stage** — infer from artifacts present & task state:
  - `ideation` — proposal/design exist, tasks empty → still planning
  - `in-progress` — some tasks checked, some unchecked → actively being implemented
  - `ready-for-review` — all tasks checked, not yet verified/archived
  - `stalled` — tasks partial but looks untouched for a while (use your judgment / git recency if helpful)
- **Blocker / next** — the single most important thing this change needs

### 5. Render the concise overview (report + recommended next step)

Keep it **scan-friendly**. One section per change, each condensed. Suggested shape:

```
## OpenSpec Status

3 active changes:

### add-auth-flow — "OAuth login for the API"
- Stage: in-progress  •  Progress: 3/7 tasks
- Next: task 4 "Wire up refresh token rotation"
- Blocker: none detected

### fix-db-migration — "Repair flaky migration ordering"
- Stage: ready-for-review  •  Progress: 7/7 tasks
- Next: run `/opsx-verify fix-db-migration` then archive

### dark-mode — "UI theme toggle"
- Stage: ideation  •  Progress: 0/4 tasks
- Next: `/opsx-grill` to sharpen requirements before proposing
```

Finish with a single **"Recommended next action"** line — pick the ONE highest-leverage thing across all changes (don't enumerate everything).

---

## Recommended-next-action logic

Use this rough priority to pick the single best next move:

| If a change is...                | Recommended next action                       |
| -------------------------------- | --------------------------------------------- |
| `ready-for-review`               | verify then archive (`/opsx-verify <name>`)   |
| `in-progress` + no blocker       | continue applying (`/opsx-apply <name>`)      |
| `in-progress` + blocker found    | surface the blocker, suggest `/opsx-explore`  |
| `ideation`                       | grill to firm up (`/opsx-grill`)              |
| multiple changes ready           | archive ready ones first to reduce queue      |

If nothing is clearly highest-leverage, say so honestly instead of forcing a pick.

---

## Guardrails

- **Report only** — never apply, verify, archive, or create/update artifacts yourself. You surface and recommend; the user decides.
- **Concise** — this is the "get up to speed in 5 seconds" command. No walls of text per change.
- **Don't guess** — if `tasks.md` is missing or unclear, say "no task data" rather than invent progress.
- **Ground in reality** — read the actual artifacts; don't summarize from memory or assumption.
- **Honest about blockers** — if something's stuck, say it. Don't sugarcoat.
- **Handle empty gracefully** — zero changes is a valid, complete answer.
- **Use `/opsx-explore`** for anything that needs deep thinking; this command is the overview, not the investigation.
