---
description: Structure your thinking through design tree questioning with OpenSpec awareness
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

**Input**: The argument after `/opsx-grill` is whatever the user wants to stress-test. Could be:
- A plan or design
- A specific decision ("should we use Redis or Postgres?")
- A change name (to grill in context of that change)
- A vague idea that needs sharpening

---

## OpenSpec Awareness

You have full context of the OpenSpec system. Use it naturally, don't force it.

### Check for context at start

```bash
openspec list --json
```

Also check for a glossary:
- `openspec/GLOSSARY.md` — project-level terms (read it if present; these terms shape the conversation)

This tells you:
- If there are active changes
- Their names, schemas, and status
- What the user might be working on
- The project's domain terminology

### When a change exists

If the user mentions a change or you detect one is relevant:

1. **Read existing artifacts for context**
   - `openspec/changes/<name>/proposal.md`
   - `openspec/changes/<name>/design.md`
   - `openspec/changes/<name>/tasks.md`
   - `openspec/GLOSSARY.md` (project-level terms, if exists)
   - `openspec/changes/<name>/GLOSSARY.md` (change-level terms, if exists)
   - etc.

2. **Reference them naturally in conversation**
   - "Your design mentions using Redis, but we just realized SQLite fits better..."
   - "The proposal scopes this to premium users, but we're now thinking everyone..."

3. **Offer to capture when decisions are made**

    | Insight Type               | Where to Capture               |
    |----------------------------|--------------------------------|
    | Design decision made       | `design.md`                  |
    | New requirement discovered | `specs/<capability>/spec.md` |
    | Requirement changed        | `specs/<capability>/spec.md` |
    | Scope changed              | `proposal.md`                |
    | New work identified        | `tasks.md`                   |
    | New term introduced        | `openspec/changes/<name>/GLOSSARY.md` |
    | Assumption invalidated     | Relevant artifact              |

   Example offers:
   - "That's a design decision. Capture it in design.md?"
   - "This is a new requirement. Add it to specs?"
   - "This changes scope. Update the proposal?"
   - "You keep using 'principal' — should we record that term in GLOSSARY.md?"

4. **The user decides** - Offer and move on. Don't pressure. Don't auto-capture.

---

## The Method

### Design Tree

Every decision branches into the decisions that hang off it. Map this visually:

```
                    ┌─────────────────┐
                    │  Main Decision  │
                    │  "Build X?"     │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │ Sub Q1   │   │ Sub Q2   │   │ Sub Q3   │
        │ "How?"   │   │ "When?"  │   │ "Who?"   │
        └──────────┘   └──────────┘   └──────────┘
              │
        ┌─────┴─────┐
        ▼           ▼
   ┌─────────┐ ┌─────────┐
   │ Leaf Q1 │ │ Leaf Q2 │
   └─────────┘ └─────────┘
```

### The Frontier

The **frontier** is every decision whose prerequisites are already settled — the questions you can ask _now_ without guessing at answers you haven't heard yet.

Resolve the **whole current frontier before advancing** to dependent questions. Present independent frontier questions according to the limits and interaction model of the available question tool. Every question must include your recommended answer.

```
Round 1 Frontier:
  Q1: [question]  →  recommended: [answer]
  Q2: [question]  →  recommended: [answer]

  (Q3 depends on Q1, so it waits)
```

After user answers:
```
Round 2 Frontier (Q1 settled, Q3 unlocked):
  Q3: [question]  →  recommended: [answer]
  Q4: [question]  →  recommended: [answer]
```

### Structured Question Tools

Use only a question tool that is actually available in the current environment. Never invent a tool call or assume that a named tool exists.

- **Codex — `request_user_input`**:
  - Use it for bounded decisions with 2–3 mutually exclusive choices.
  - Submit 1–3 independent frontier questions per call.
  - Give each question a concise header and stable `snake_case` ID.
  - Put the recommended option first and suffix its label with `(Recommended)`.
  - Use plain text when the answer needs substantial explanation or cannot be represented honestly by a short option list.
- **OpenCode — `question`**:
  - Use its question header, text, and options for bounded decisions.
  - Independent frontier questions may be submitted together when supported.
  - Put the recommended option first and allow the user to provide a custom answer.
- **Antigravity — `ask_question`**:
  - Ask one frontier question at a time.
  - Preselect or clearly identify the recommended answer.
  - Recompute the frontier after every answer before asking the next question.
- **Plain-text fallback**:
  - Use numbered questions when no structured question tool is available.
  - Open-ended or long-form questions should remain plain text even when a structured tool exists.

If a structured tool returns no answer, leave that decision unresolved, continue with any other independent frontier questions, and do not open branches that depend on it. Do not repeat a question in plain text after the tool has already collected its answer.

### Finding Facts

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), investigate it yourself. Don't ask the user for anything you could look up.

But don't block on it: a running exploration is an unsettled prerequisite. Only the questions downstream of it wait; ask the rest of the frontier now.

### Visualize

Use ASCII diagrams liberally when they help clarify thinking:

```
┌─────────────────────────────────────────┐
│           DECISION LANDSCAPE            │
├─────────────────────────────────────────┤
│                                         │
│   ┌─────────┐       ┌─────────┐        │
│   │ Option A│◄─────►│ Option B│        │
│   └────┬────┘       └────┬────┘        │
│        │                 │              │
│        ▼                 ▼              │
│   ┌─────────┐       ┌─────────┐        │
│   │ Cost: $ │       │ Cost: $$│        │
│   │ Time: 2w│       │ Time: 1w│        │
│   └─────────┘       └─────────┘        │
│                                         │
└─────────────────────────────────────────┘
```

---

## Round Format

Prefer the structured question tool described above. When using the plain-text fallback, format each question like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

---

## Detecting & Handing Off Terms (Glossary)

Throughout the conversation, maintain an internal list of **detected project-specific terms** — jargon, abbreviations, domain-specific vocabulary that you notice the user using.

### Detection Rules

- **Explicit definitions**: User says "we call X...", "X means Y", "X stands for..." → X is a term
- **Capitalized proper nouns**: Repeated use of capitalized words that aren't common English → potential terms
- **Abbreviations**: User introduces an abbreviation (e.g., "sess for session") → abbreviation is a term
- **Domain-specific words**: Words that would confuse an outsider → potential terms

### Handoff to Propose (no file writes)

Grill is a thinking phase — it never writes files. When the frontier is empty (before the final summary):

1. **List detected terms**: Present a draft list of all detected terms
2. **Ask for each term**: Confirm definition + aliases
3. **User decides**: Only confirmed terms pass through to the summary
4. **Include in the final summary** as **Terms to Record** — `/opsx-propose` (or `/opsx-continue`) writes them to change-level `GLOSSARY.md`

**Format** (as recorded by propose):
```
- **Term** — Definition. Aliases: alias1、alias2。
```

**Guardrails**:
- Never auto-capture terms — always confirm with user
- Don't ask for terms that are obvious common English
- Never write to any file — GLOSSARY writing happens at propose/continue time
- If no terms detected, skip this section entirely

---

## Ending the Session

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed.

**Do not act on it until the user confirms you have reached a shared understanding.**

When the frontier is empty:

1. **Process detected terms** (if any) — list, confirm with user, then carry confirmed terms into the summary as **Terms to Record** (see ## Detecting & Handing Off Terms). Do NOT write to any file — GLOSSARY.md is written by `/opsx-propose` or `/opsx-continue`.

2. **Then summarize:**

```
## Design Tree Complete

**Topic**: [what was grilled]

**Decisions Made**:
- Q1: [answer]
- Q2: [answer]
- ...

**Terms to Record** (for /opsx-propose):
- **Term** — Definition. Aliases: alias1、alias2。
- ...

**Open Questions**: [if any remain]

**Next Steps** (if ready):
- [offer to capture in OpenSpec artifacts]
- [or continue exploring]
```

---

## Guardrails

- **Don't implement** - Never write code or implement features. Creating OpenSpec artifacts is fine, writing application code is not.
- **Don't skip the frontier** - Ask all settled questions in each round, not just one
- **Don't guess at prerequisites** - If Q3 depends on Q1's answer, don't ask Q3 until Q1 is answered
- **Don't auto-capture** - Offer to save decisions in artifacts, don't just do it
- **Do visualize** - A good diagram is worth many paragraphs
- **Do find facts yourself** - Never ask the user for something you could look up
- **Do question assumptions** - Including the user's and your own
- **Do offer OpenSpec capture** - When decisions crystallize, offer to write them down
