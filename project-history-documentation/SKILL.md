---
name: project-history-documentation
description: Maintains PROJECT_HISTORY.md with structured bug catalog, decision rationale, and lessons learned. Use when documenting bugs, technical decisions, version milestones, or architecture changes. Covers automatic documentation rules during development. Not for READMEs, API docs, or user guides.
disable-model-invocation: true
---

# Project History Documentation

## When to Use

- Starting a new project (create initial document)
- After fixing a bug with a non-obvious root cause
- After making a decision where alternatives were considered
- After a version release or architecture change
- After receiving a user correction that reveals an instruction gap or repeated mistake

Do NOT use for: API docs, user guides, or README files.

## Automatic Documentation Rules

### After Every Non-Trivial Bug Fix
Add a bug catalog entry if the root cause was not immediately obvious. Skip trivial bugs (typos, missing imports, copy-paste errors).

### After Every Non-Obvious Technical Decision
Add a decision rationale entry if you considered alternatives before choosing.

### After Every User Correction
Add a corrections log entry if the user corrects Claude's behavior and the root cause is a gap in instructions, a conflicting rule, or a repeated mistake. Skip one-off misunderstandings or preference clarifications that won't recur.

### After Every Version Bump
Update the version history table and review lessons learned for new entries.

## Sections

Create PROJECT_HISTORY.md with sections 1-4 at project start. Add others as needed.

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
| 12 | Corrections Log | After first instruction gap correction | After every non-trivial correction |

Consult `references/section-templates.md` for exact templates and formatting for each section.

## Bug Catalog Rules

- Categorize by type: race conditions, data accuracy, silent failures, timing, etc.
- Use short IDs for cross-referencing: RC-1, DA-2, SF-1
- Always include the root cause. The fix is in the code. The root cause is the lesson.
- Add a "Pattern" line when the bug teaches something reusable beyond this project
- Include embarrassing bugs. Those are often the most valuable.

## Corrections Log Rules

- Use short IDs: CL-1, CL-2, etc.
- Always include the root cause (why Claude made the mistake, not just what the mistake was)
- Add a "Prevention" line describing what was changed to stop it from recurring
- Only log corrections that reveal systemic issues -- skip one-off misunderstandings

## Quality Checklist

- [ ] Every bug entry has a root cause, not just a fix
- [ ] Every correction entry has a root cause and a prevention action
- [ ] Decision rationale includes rejected alternatives and why
- [ ] Implementation sequence reflects build order, not alphabetical
- [ ] Code examples under 20 lines with WHY comments
- [ ] No project secrets (API keys, IPs, passwords)
- [ ] Lessons written as transferable principles
- [ ] Understandable without follow-up questions

For writing guidelines and common mistakes, see `references/writing-guidelines.md`.
