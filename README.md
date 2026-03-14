# Claude Code Skills

Personal skills for Claude Code — reusable patterns and best practices extracted from building production systems.

These skills teach Claude how to avoid common pitfalls in async programming, crash recovery, financial math, testing, deployment, and more. They are general-purpose and apply across projects.

## Installation

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/knxqtr/claude-code-skills ~/.claude/skills
```

Or symlink if you want to keep the repo elsewhere:

```bash
git clone https://github.com/knxqtr/claude-code-skills ~/claude-code-skills
ln -s ~/claude-code-skills/* ~/.claude/skills/
```

## Skills

### High Value

| Skill | What it does |
|-------|-------------|
| crash-recovery-design | State persistence and startup recovery for long-running services |
| silent-failure-prevention | Ensure every user action gets visible feedback, with real fallbacks |
| async-safety-checklist | Race condition prevention, lock patterns, shutdown ordering |
| financial-math-rules | Decimal math, fill prices, fee handling, balance accuracy |
| mock-fidelity-standards | Mocks that reproduce real error behavior, not just happy paths |
| defense-in-depth-monitoring | Multi-layer event detection with timing guards |

### Nice to Have

| Skill | What it does |
|-------|-------------|
| api-integration-patterns | External API retry, caching, fuzzy matching, error handling |
| project-bootstrapping | Python project setup from scratch — structure, config, testing |
| vps-deployment-checklist | systemd, rsync, venv activation, lock files, heartbeats |
| order-management-edge-cases | Partial fills, cancel races, resize windows, size calculations |
| property-based-testing | Hypothesis-based fuzz testing with invariant checking |
| project-history-documentation | Creates and maintains a PROJECT_HISTORY.md technical build reference |
| skill-extraction | Extracts reusable skills from project history, bug catalogs, and lessons learned |

## Origin

These patterns were extracted from building production systems with real users and real consequences. Every rule exists because ignoring it caused a real bug — 32 bugs cataloged with root causes across 1,463 tests.
