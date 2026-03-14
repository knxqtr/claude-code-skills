---
name: skill-extraction
description: Extracts reusable Claude Code skills from a project's history, bug catalog, and lessons learned. Runs automatically after version bumps. Checks existing skills before creating new ones. Use when a version is released, user says "extract skills" or "update skills", or after PROJECT_HISTORY.md has significant new entries. Do NOT use during active development — wait until a version milestone.
---

# Skill Extraction

## When to Use

- Automatically after a version bump (triggered by global CLAUDE.md rules)
- User asks to extract lessons into reusable skills
- PROJECT_HISTORY.md has 3+ new bug catalog entries or lessons since last extraction

Do NOT use during active development. Wait until a version milestone.

## Process

### Step 1: Review the Source Material

Read the project's PROJECT_HISTORY.md in this order of value:
1. Bug catalog (highest value — every bug with a root cause is a potential skill rule)
2. Lessons learned (already written as transferable principles)
3. Decision rationale (non-obvious choices that should be repeated)
4. Implementation sequence (build order patterns that worked)

If no PROJECT_HISTORY.md exists, create one first using the project-history-documentation skill.

### Step 2: Check Existing Skills First

This is the most important step. Before creating anything new:

```bash
ls ~/.claude/skills/
```

Read the SKILL.md of every existing skill that could be related. For each candidate pattern from the project history, ask:

1. Does an existing skill already have a rule covering this exact scenario? If yes, skip.
2. Does an existing skill cover the same category but is missing this specific case? If yes, add the new rule/example to that skill.
3. Is this a genuinely new category not covered by any existing skill? Only then create a new skill.

Priority order: skip (already covered) > update existing > create new.

Creating new skills increases token load across all conversations. Only create when no existing skill fits.

### Step 3: Identify Transferable Patterns

For each unmatched bug, lesson, or decision, ask:

- Would this apply to a different project built with different technology? If yes, strong candidate.
- Would this apply to a similar project? If yes, candidate.
- Is this obvious to any experienced developer? If yes, skip.
- Was the root cause non-obvious? If yes, strong candidate.
- Did this pattern appear 2+ times in this project? If yes, strong candidate.

### Step 4: Write or Update Skills

For updates to existing skills:
- Add the new rule or example in the appropriate section
- Do not duplicate existing content
- Update the description if new trigger phrases are needed
- Keep the skill under its word limit

For new skills, follow Anthropic's guide:

Frontmatter:
- name: kebab-case only, matches folder name
- description: [What it does] + [When to use it] + [Trigger phrases] + [Negative triggers]
- Under 1024 characters, no XML angle brackets

SKILL.md body:
- Under 500 words for frequently-triggered skills
- Specific and actionable instructions, not vague guidance
- Include common mistakes section
- Move heavy reference to references/ folder

### Step 5: Quality Check

Before committing, verify each new or updated skill:

- [ ] Checked all existing skills for overlap first
- [ ] No duplicate rules across skills
- [ ] Description includes trigger phrases and negative triggers
- [ ] No project-specific details leaked (no IPs, keys, repo names, proprietary logic)
- [ ] Instructions are actionable (commands, code patterns, checklists)
- [ ] Common mistakes section exists
- [ ] If over 500 words, heavy content moved to references/
- [ ] Update README.md in skills repo with any new skills

### Step 6: Commit, Push, and Symlink

```bash
cd ~/claude-code-skills
git add -A
git commit -m "Add/update skills from [project name] v[version] lessons"
git push
```

Symlink any new skills so they are immediately active:
```bash
ln -s ~/claude-code-skills/new-skill-name/ ~/.claude/skills/new-skill-name
```

## What to Extract vs What to Skip

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

## Common Mistakes

- Not checking existing skills first. This is the most common error. Always read existing skills before creating new ones.
- Creating a new skill for every bug. Most bugs add a rule to an existing skill, not a new skill.
- Including project-specific details. Skills must be generic. No server IPs, API keys, or proprietary logic.
- Writing vague instructions. "Handle errors properly" is not a skill. "Wrap every user-triggered operation in try/except and notify on the user-facing channel" is a skill.
- Extracting too early. Wait until a version milestone. Patterns that seem important mid-development often turn out to be one-off issues.
