---
name: project-history-documentation
description: Creates and maintains a PROJECT_HISTORY.md technical build reference for any project. Captures architecture, bug catalog with root causes, decision rationale, and lessons learned. Use when user says "document project", "project history", "build log", "capture what we built", or when a project reaches a milestone worth documenting. Do NOT use for README files, API docs, or user guides.
---

# Project History Documentation

## When to Use

- A project has reached a working state and the build process should be captured
- User asks to document what was built, how, and why
- After a significant milestone (MVP, v1.0, production deploy)
- Starting a new project where you want to track decisions from day one

Do NOT use for: API docs, user guides, or README files. This is a technical build reference.

## Why This Document Exists

A project history answers questions that code and git history cannot:
- WHY was this decision made? (code shows what, not why)
- What was the root cause of that bug? (git log shows the fix, not the investigation)
- What order should things be built in? (git history is chronological, not logical)
- What lessons transfer to the next project? (buried in commits and conversations)

## Sections

Create a file called PROJECT_HISTORY.md. Start with the sections you have content for. Add sections as the project grows. Do not try to fill every section on day one.

| # | Section | What it answers |
|---|---------|----------------|
| 1 | Header and Summary | What is this project? Who should read this? |
| 2 | Table of Contents | Navigation (number every section) |
| 3 | Operational Reference | What does the system do? How do I interact with it? |
| 4 | Architecture | How are the components connected? File responsibilities? |
| 5 | Implementation Sequence | What order should things be built in and why? |
| 6 | Bug Catalog | What broke, why, and what is the generalizable lesson? |
| 7 | Decision Rationale | Why this choice over the alternatives? |
| 8 | Startup/Shutdown Sequences | What order must things start/stop and why? |
| 9 | Test Architecture | How are tests organized and isolated? |
| 10 | Version History | What changed in each release? |
| 11 | Lessons Learned | What transfers to the next project? |

Consult `references/section-templates.md` for exact templates and formatting for each section.

## Bug Catalog Rules

The bug catalog is the most valuable section. Every significant bug gets a structured entry with: Trigger, Symptom, Root cause, Fix, Commit, and Pattern (generalizable lesson).

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
5. Update as the project evolves. A stale project history misleads.

## Iterative Approach

Do not try to write the perfect document in one pass.

- First pass: header, architecture, and whatever sections you have content for
- After bugs are found: add to bug catalog
- After non-obvious decisions: add to decision rationale
- After a release: add to version history
- At milestones: review and update lessons learned

The document grows alongside the project. Sections that do not apply yet can be added later.

## Quality Checklist

Before marking the document as complete, verify:

- [ ] Every bug entry has a root cause, not just a fix description
- [ ] Decision rationale includes alternatives that were rejected and why
- [ ] Implementation sequence reflects build order, not alphabetical order
- [ ] Code examples are under 20 lines and include WHY comments
- [ ] No project secrets (API keys, IPs, passwords) appear anywhere
- [ ] Lessons learned are written as transferable principles, not project-specific notes
- [ ] Someone unfamiliar with the project could understand every section without follow-up questions

## When to Update

- After fixing a significant bug (add to bug catalog)
- After making a non-obvious technical decision (add to decision rationale)
- After a version release (add to version history)
- After a deployment or architecture change (update relevant sections)
- At project milestones (review lessons learned)

## Common Mistakes

- Writing a narrative diary instead of a reference document. This is not "what happened on Tuesday."
- Omitting the root cause from bug entries. The fix is in the code. The value is understanding WHY.
- Only documenting happy paths. The bug catalog and decision rationale are the most valuable sections.
- Letting the document go stale. Rule: every push updates the relevant sections.
- Trying to write all sections at once. Start with what you know, add as the project grows.
