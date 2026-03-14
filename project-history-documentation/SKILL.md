---
name: project-history-documentation
description: Maintains PROJECT_HISTORY.md with bug catalog, decisions, and lessons. Use when documenting bugs, decisions, or milestones. Not for READMEs or API docs.
---

# Project History Documentation

## When to Use

- Starting a new project (create the initial document)
- After fixing a bug with a non-obvious root cause (add to bug catalog)
- After making a decision where alternatives were considered (add to decision rationale)
- After a version release (update version history and lessons learned)
- After architecture changes (update architecture and startup/shutdown sections)

Do NOT use for: API docs, user guides, or README files. This is a technical build reference.

## Automatic Documentation Rules

This document is maintained continuously, not written after the fact. Follow these rules during active development:

### After Every Non-Trivial Bug Fix

If the root cause was not immediately obvious, add a bug catalog entry:

```
**[ID]: [Short descriptive name]**
Trigger:  [What causes it]
Symptom:  [What the user or developer sees]
Root:     [The actual underlying cause]
Fix:      [What was changed]
Commit:   [Reference]
Pattern:  [Generalizable lesson, if any]
```

Skip trivial bugs (typos, missing imports, copy-paste errors). Document bugs where the root cause required investigation or where the same mistake could happen in another project.

### After Every Non-Obvious Technical Decision

If you considered alternatives before choosing, add a decision rationale entry:

```
| Decision | Choice | Why |
|----------|--------|-----|
```

Skip obvious decisions. Document decisions where the wrong choice would have caused real problems.

### After Every Version Bump

Update the version history table and review lessons learned for new entries.

## Why This Document Exists

A project history answers questions that code and git history cannot:
- WHY was this decision made? (code shows what, not why)
- What was the root cause of that bug? (git log shows the fix, not the investigation)
- What order should things be built in? (git history is chronological, not logical)
- What lessons transfer to the next project? (buried in commits and conversations)

## Sections

Create a file called PROJECT_HISTORY.md. Start with sections 1-4 on project creation. Add remaining sections as the project grows.

| # | Section | When to create | When to update |
|---|---------|---------------|----------------|
| 1 | Header and Summary | Project start | When scope changes |
| 2 | Table of Contents | Project start | When sections are added |
| 3 | Operational Reference | When system is usable | When user-facing behavior changes |
| 4 | Architecture | Project start | When components are added/changed |
| 5 | Implementation Sequence | After first phase complete | After each build phase |
| 6 | Bug Catalog | After first non-trivial bug | After every non-trivial bug fix |
| 7 | Decision Rationale | After first non-obvious decision | After every non-obvious decision |
| 8 | Startup/Shutdown Sequences | When service has ordering requirements | When ordering changes |
| 9 | Test Architecture | When test suite exists | When test strategy changes |
| 10 | Version History | After first release | After every version bump |
| 11 | Lessons Learned | After first milestone | At every milestone |

Consult `references/section-templates.md` for exact templates and formatting for each section.

## Bug Catalog Rules

- Categorize by type: race conditions, data accuracy, silent failures, timing, etc.
- Use short IDs for cross-referencing: RC-1, DA-2, SF-1
- Always include the root cause. The fix is in the code. The root cause is the lesson.
- Add a "Pattern" line when the bug teaches something reusable beyond this project
- Include embarrassing bugs. Those are often the most valuable.

## Writing Guidelines

1. Write for someone who was not in the room.
2. Code examples: short (under 20 lines), comments explain WHY not WHAT.
3. Tables for structured data. Prose for context and reasoning.
4. Do not duplicate git log. Focus on WHY and LESSONS, not chronological play-by-play.

## Quality Checklist

Before marking the document as complete at a milestone, verify:

- [ ] Every bug entry has a root cause, not just a fix description
- [ ] Decision rationale includes alternatives that were rejected and why
- [ ] Implementation sequence reflects build order, not alphabetical order
- [ ] Code examples are under 20 lines and include WHY comments
- [ ] No project secrets (API keys, IPs, passwords) appear anywhere
- [ ] Lessons learned are written as transferable principles, not project-specific notes
- [ ] Someone unfamiliar with the project could understand every section without follow-up questions

## Common Mistakes

- Writing a narrative diary instead of a reference document. This is not "what happened on Tuesday."
- Omitting the root cause from bug entries. The fix is in the code. The value is understanding WHY.
- Only documenting happy paths. The bug catalog and decision rationale are the most valuable sections.
- Documenting every trivial bug. Only catalog bugs where the root cause was non-obvious or the lesson is transferable.
- Waiting until the end to document. Update continuously during development.
