---
name: project-history-documentation
description: Creates and maintains a PROJECT_HISTORY.md technical build reference for any project. Captures architecture, bug catalog with root causes, decision rationale, and lessons learned. Use when user says "document project", "project history", "build log", "write documentation", "capture what we built", or when a project reaches a milestone worth documenting.
---

# Project History Documentation

## When to Use

- A project has reached a working state and the build process should be captured
- User asks to document what was built, how, and why
- After a significant milestone (MVP, v1.0, production deploy)
- Starting a new project where you want to track decisions from day one
- Reviewing an existing project that lacks documentation

Do NOT use for: API docs, user guides, or README files. This is a technical build reference for the developers who built it and anyone who might build something similar.

## What Makes This Document Valuable

A good project history answers questions that code and git history cannot:
- WHY was this decision made? (code shows what, not why)
- What bugs were found and what was the root cause? (git log shows the fix, not the investigation)
- What order should things be built in? (git history shows chronological, not logical order)
- What lessons apply to the next project? (buried in commit messages and conversations)

## Document Structure

Create a file called PROJECT_HISTORY.md with these sections. Not every project needs all sections. Start with what exists and add sections as the project grows.

### Section 1: Header and Summary

```markdown
# [Project Name] — Technical Build Reference

[One-line description of what the project does, end-to-end flow.]

[Timeline and scope summary. Example: "From zero to fully operational in 3 days."]
[Who should read this and when.]
```

### Section 2: Table of Contents

Number every section. This document will grow and readers need to jump to what they need.

### Section 3: Operational Reference

How the system works from the user's perspective. Include:
- Input format (what goes in)
- Output/actions (what comes out)
- Commands and controls available to the user
- Key configuration values in a table
- All notification/alert types the system produces

This section answers: "What does this system do and how do I interact with it?"

### Section 4: Architecture

```markdown
## Architecture

[ASCII diagram showing data flow between components]

### File Responsibilities
| File | Responsibility | Lines |
|------|---------------|-------|
| ... | ... | ... |

### Dependencies
[List with version constraints and what each is for]

### Runtime Requirements
[Python version, env vars, external services needed]
```

### Section 5: Implementation Sequence

Document the build phases in the order they were built, not alphabetically. Each phase should include:
- What was built and why in that order
- Key code patterns introduced (with short inline examples)
- What changed from the original plan, if anything

```markdown
### Phase 1 — [Name] ([Timeline], [commits])
[What was built. Why this order.]
[Code pattern if non-obvious:]
```python
# Short, relevant code example (under 20 lines)
```
```

This section answers: "If I were building this from scratch, what order should I build things in?"

### Section 6: Bug Catalog

The most valuable section. Every significant bug gets a structured entry:

```markdown
### Category: [Race Conditions / Timing / Data Accuracy / etc.]

**[ID]: [Short descriptive name]**
Trigger:  [What causes it]
Symptom:  [What the user or developer sees]
Root:     [The actual underlying cause]
Fix:      [What was changed]
Commit:   [Reference]
Pattern:  [Generalizable lesson, if any]
```

Rules for the bug catalog:
- Categorize bugs by type (race conditions, data accuracy, silent failures, etc.)
- Include the root cause, not just the fix. The root cause is the lesson.
- Add a "Pattern" line when the bug teaches something reusable beyond this project
- Use short IDs (RC-1, DA-2, SF-1) for cross-referencing

### Section 7: Decision Rationale

A table of significant technical decisions:

```markdown
| Decision | Choice | Why |
|----------|--------|-----|
| SL method | Candle close, not tick | Signals are candle-close strategies; tick triggers false stops |
| State persistence | JSON file | Max ~10 concurrent items; human-readable; zero dependencies |
| Concurrency model | asyncio single-thread | I/O-bound workload; no thread-safety complexity |
```

Only include decisions where the "why" is non-obvious or where alternatives were seriously considered.

### Section 8: Startup and Shutdown Sequences

For services and bots, document the exact order things must start and stop, and WHY that order matters.

```markdown
### Startup (Critical Order)
1. Logging         -- available for everything after
2. Lock file       -- prevent duplicate instances
3. External client -- API connection
...

Step 5 before Step 6 is critical because: [reason]
```

### Section 9: Test Architecture

```markdown
## Test Architecture

[Directory tree of test files with one-line descriptions]

Key patterns:
- [How tests are isolated from production data]
- [Shared test utilities and why they exist]
- [Any special test infrastructure (simulation harness, fuzz testing)]
```

### Section 10: Version History

```markdown
| Version | Date | Key Changes |
|---------|------|-------------|
| v1.0.0 | 2026-03-05 | Initial release. Core loop end-to-end. |
| v2.0.0 | 2026-03-09 | Full validation. All commands. 878 tests. |
```

### Section 11: Lessons Learned

Group by category. Write as transferable principles, not project-specific notes.

```markdown
### [Category name]
- [Principle that applies beyond this project]
- [Another principle]
```

Good lessons learned are ones someone building a completely different project could benefit from.

## Writing Guidelines

1. Write for someone who was not in the room. They should understand every decision without asking follow-up questions.

2. Code examples should be short (under 20 lines), show the pattern clearly, and include comments explaining WHY, not WHAT.

3. Use tables for structured data (config values, decisions, versions). Use prose for context and reasoning.

4. Keep the bug catalog honest. Include embarrassing bugs. The root cause is what makes them valuable.

5. Update the document as the project evolves. A stale project history is worse than none because it misleads.

6. Do not duplicate what git log already shows. Focus on the WHY and the LESSONS, not the chronological play-by-play.

## When to Update

- After fixing a significant bug (add to bug catalog)
- After making a non-obvious technical decision (add to decision rationale)
- After a version release (add to version history)
- After a deployment or architecture change (update relevant sections)
- At project milestones (review lessons learned)

## Common Mistakes

- Writing a narrative diary instead of a reference document. This is not "what happened on Tuesday." It is "how this system works and why."
- Omitting the root cause from bug entries. The fix is in the code. The value is in understanding WHY the bug existed.
- Only documenting happy paths. The bug catalog and decision rationale are the most valuable sections.
- Letting the document go stale. Set a rule: every push updates the relevant sections.
