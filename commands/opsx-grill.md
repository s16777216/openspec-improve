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

This tells you:
- If there are active changes
- Their names, schemas, and status
- What the user might be working on

### When a change exists

If the user mentions a change or you detect one is relevant:

1. **Read existing artifacts for context**
   - `openspec/changes/<name>/proposal.md`
   - `openspec/changes/<name>/design.md`
   - `openspec/changes/<name>/tasks.md`
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
    | Assumption invalidated     | Relevant artifact              |

   Example offers:
   - "That's a design decision. Capture it in design.md?"
   - "This is a new requirement. Add it to specs?"
   - "This changes scope. Update the proposal?"

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

Ask the **whole frontier in one round**: number each question and give your recommended answer. Then wait for the user's answers before the next round.

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

Each question should be formatted like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

---

## Ending the Session

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed.

**Do not act on it until the user confirms you have reached a shared understanding.**

When the frontier is empty, summarize:

```
## Design Tree Complete

**Topic**: [what was grilled]

**Decisions Made**:
- Q1: [answer]
- Q2: [answer]
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
