# Section Templates for PROJECT_HISTORY.md

Use these templates when writing each section. Copy and adapt to fit your project.

---

## Header and Summary

```markdown
# [Project Name] — Technical Build Reference

[One-line description of what the project does, end-to-end flow.]

[Timeline and scope summary. Example: "From zero to fully operational in 3 days."]
[Who should read this and when.]
```

---

## Architecture

```markdown
## Architecture

[ASCII diagram showing data flow between components]

### File Responsibilities
| File | Responsibility | Lines |
|------|---------------|-------|
| config.py | All settings centralized | 86 |
| main.py | Startup sequence, lock file, schedulers | 491 |

### Dependencies
[List with version constraints and what each is for]

### Runtime Requirements
[Python version, env vars, external services needed]
```

---

## Implementation Phase

```markdown
### Phase 1 — [Name] ([Timeline], [commits])
[What was built. Why this order.]
[Code pattern if non-obvious:]
```python
# Short, relevant code example (under 20 lines)
```
```

---

## Bug Catalog Entry

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

Bug ID format: two-letter category code + number (RC-1, DA-2, SF-1).

---

## Decision Rationale

```markdown
| Decision | Choice | Why |
|----------|--------|-----|
| State persistence | JSON file | Max ~10 concurrent items; human-readable; zero dependencies |
| Concurrency model | asyncio single-thread | I/O-bound workload; no thread-safety complexity |
| Auth storage | httpOnly cookies, not localStorage | XSS cannot exfiltrate tokens from cookies |
```

---

## Startup and Shutdown Sequences

```markdown
### Startup (Critical Order)
1. Logging         -- available for everything after
2. Lock file       -- prevent duplicate instances
3. External client -- API connection
...

Step 5 before Step 6 is critical because: [reason]

### Shutdown (Critical Order)
1. Stop workers    -- may send final notifications during cleanup
2. Stop schedulers -- heartbeat, summaries
3. Stop comms      -- disconnect last
```

---

## Test Architecture

```markdown
## Test Architecture

[Directory tree of test files with one-line descriptions]

Key patterns:
- [How tests are isolated from production data]
- [Shared test utilities and why they exist]
- [Any special test infrastructure (simulation harness, fuzz testing)]
```

---

## Version History

```markdown
| Version | Date | Key Changes |
|---------|------|-------------|
| v1.0.0 | 2026-03-05 | Initial release. Core loop end-to-end. |
| v2.0.0 | 2026-03-09 | Full validation. All commands. 878 tests. |
```

---

## Lessons Learned

```markdown
### [Category name]
- [Principle that applies beyond this project]
- [Another principle]
```

Good lessons learned are ones someone building a completely different project could benefit from.
