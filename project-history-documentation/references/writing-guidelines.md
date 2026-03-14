# Writing Guidelines

## Why This Document Exists

A project history answers questions that code and git history cannot:
- WHY was this decision made? (code shows what, not why)
- What was the root cause of that bug? (git log shows the fix, not the investigation)
- What order should things be built in? (git history is chronological, not logical)
- What lessons transfer to the next project? (buried in commits and conversations)

## Guidelines

1. Write for someone who was not in the room.
2. Code examples: short (under 20 lines), comments explain WHY not WHAT.
3. Tables for structured data. Prose for context and reasoning.
4. Do not duplicate git log. Focus on WHY and LESSONS, not chronological play-by-play.

## Bug Catalog Entry Format

```
**[ID]: [Short descriptive name]**
Trigger:  [What causes it]
Symptom:  [What the user or developer sees]
Root:     [The actual underlying cause]
Fix:      [What was changed]
Commit:   [Reference]
Pattern:  [Generalizable lesson, if any]
```

## Decision Rationale Entry Format

```
| Decision | Choice | Why |
|----------|--------|-----|
```

## Common Mistakes

- Writing a narrative diary instead of a reference document. This is not "what happened on Tuesday."
- Omitting the root cause from bug entries. The fix is in the code. The value is understanding WHY.
- Only documenting happy paths. The bug catalog and decision rationale are the most valuable sections.
- Documenting every trivial bug. Only catalog bugs where the root cause was non-obvious or the lesson is transferable.
- Waiting until the end to document. Update continuously during development.
