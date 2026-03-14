---
name: skill-extraction
description: Extracts reusable skills from project history. Use at version bumps or when asked to update skills.
disable-model-invocation: true
---

# Skill Extraction

## When to Use

- Automatically after a version bump (triggered by global CLAUDE.md rules)
- User asks to extract lessons into reusable skills
- PROJECT_HISTORY.md has 3+ new bug catalog entries or lessons since last extraction

Do NOT use during active development. Wait until a version milestone.

## Process Summary

1. Read the project's PROJECT_HISTORY.md (bug catalog first, then lessons, decisions, implementation sequence)
2. Check ALL existing skills for overlap before creating anything new
3. Identify transferable patterns (non-obvious, appeared 2+ times, applicable beyond this project)
4. Write or update skills (priority: update existing > add reference file > new skill as last resort)
5. Run quality gate on every rule
6. Log the extraction
7. Commit and push

For the full step-by-step process with commands and templates, see `references/process-details.md`.

## Priority Order

skip (already covered) > update existing skill > create new skill

Creating new skills increases token load across all conversations. Only create when no existing skill fits.

## What to Extract vs Skip

Extract:
- Bugs where the root cause was non-obvious
- Patterns that appeared 2+ times across the project
- Decisions where the wrong choice would have caused real damage
- Workarounds for platform/API quirks not documented elsewhere

Skip:
- Anything already covered by an existing skill
- Project-specific configuration or business logic
- Bugs caused by simple typos or copy-paste errors
- Patterns well-documented in official docs
- Lessons that only apply to one specific technology version

## Quality Gate

Every candidate rule must pass all three tests:

1. Can I give a concrete code example? If not, the rule is too vague.
2. Does it say what TO DO, not just what to avoid? Add the alternative.
3. Would a developer actually change their behavior? If they would have done this anyway, skip it.

## Skill Count Limit

If count reaches 40, stop and notify the user. Do not create new skills without user guidance.

## Quality Checklist

- [ ] Every rule passed the quality gate
- [ ] Checked all existing skills for overlap
- [ ] No duplicate rules across skills
- [ ] No project-specific details leaked
- [ ] Instructions are actionable (commands, code patterns, checklists)
- [ ] Common mistakes section exists
- [ ] SKILL.md under 500 words; heavy content in references/

## Common Mistakes

- Not checking existing skills first. Always read existing skills before creating new ones.
- Creating a new skill for every bug. Most bugs add a rule to an existing skill.
- Including project-specific details. Skills must be generic.
- Writing vague instructions. "Handle errors properly" is not a skill. "Wrap every user-triggered operation in try/except and notify on the user-facing channel" is.
- Extracting too early. Wait until a version milestone.
