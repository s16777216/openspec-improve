# Contributing

## Adding a New Workflow

To add a new workflow skill:

1. Create `skills/openspec-<action>/SKILL.md` with the workflow content.
2. Ensure steps, guardrails, and output contracts are consistent with the other skills.
3. Add the skill to the reference tables in both README.md and README.zh.md.
4. Run the consistency checks (see below) (currently 15 skills).

## Consistency Checks

Run these read-only checks to verify project health:

```bash
# Scan for leftover /opsx-style aliases (all commands use the real skill name now)
grep -ri 'opsx' skills/ README.md README.zh.md

# Verify YAML frontmatter in all skills
grep -l 'generatedBy' skills/*/SKILL.md

# Check Markdown fence validity
# (use a markdown linter or manual review)
```
