---
name: skill-loader
description: Checks for applicable domain skills before writing implementation code. Use before writing any new implementation code to ensure relevant domain skills from ~/.claude/skills/ are loaded into the project. Not for documentation, config changes, or minor edits.
---

# Skill Loader

## When to Use

Before writing any new implementation code (new files, new functions, new modules). This includes:

- Starting a new feature or component
- Adding error handling, retries, or resilience patterns
- Writing tests with mocks or fakes
- Setting up deployment, monitoring, or alerting
- Working with external APIs, async code, or shared state

Do NOT trigger for: documentation edits, config changes, renaming, formatting, or single-line fixes.

## Process

1. Read `~/.claude/skills/SKILL_INDEX.md` -- a keyword-to-skill mapping
2. Check if any keywords match the code you're about to write
3. If a matching skill exists and is not already in the project's .claude/skills/ directory, copy it in:
   `cp -r ~/.claude/skills/<skill-name> <project>/.claude/skills/`
4. Read the skill before writing the code

Skip skills already present in the project's .claude/skills/ directory.

## What Counts as a Match

A skill matches if the code you're about to write falls within the skill's described domain. Examples:

- About to write a retry loop or API client -> check api-integration-patterns
- About to use asyncio.Lock or background tasks -> check async-safety-checklist
- About to write test mocks -> check mock-fidelity-standards
- About to calculate money or fees -> check financial-math-rules

When uncertain, read the skill's description. A false positive (reading a skill you don't need) costs a few seconds. A false negative (missing a skill you do need) costs bugs.

## Common Mistakes

- Skipping the check because "this is simple code." Simple code is where most preventable bugs live.
- Only checking skills you remember by name. Always check SKILL_INDEX.md -- new skills and keywords may have been added since your last conversation.
- Checking but not copying into the project. If you don't copy it in, it won't be loaded next conversation.
