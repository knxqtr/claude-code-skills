---
name: skill-extraction
description: Extracts reusable Claude Code skills from a project's history, bug catalog, and lessons learned. Use when user says "extract skills", "update skills", "what did we learn", "turn this into a skill", or after a PROJECT_HISTORY.md has been written or updated. Do NOT use during active development — wait until a milestone is reached.
---

# Skill Extraction

## When to Use

- A PROJECT_HISTORY.md has been written or significantly updated
- User asks to extract lessons into reusable skills
- A project reaches a major milestone (v1.0, production deploy, post-mortem)
- User asks "what did we learn that applies to other projects?"

Do NOT use during active development. Wait until there is enough history to extract from.

## Process

### Step 1: Review the Source Material

Read the project's documentation in this order of value:
1. Bug catalog (highest value — every bug with a root cause is a potential skill rule)
2. Lessons learned (already written as transferable principles)
3. Decision rationale (non-obvious choices that should be repeated)
4. Implementation sequence (build order patterns that worked)

If no PROJECT_HISTORY.md exists, suggest creating one first using the project-history-documentation skill.

### Step 2: Identify Transferable Patterns

For each bug, lesson, or decision, ask:

- Would this apply to a different project? If yes, it is a candidate.
- Is this already covered by an existing skill? If yes, update that skill instead of creating a new one.
- Is this obvious to any experienced developer? If yes, skip it. Skills should capture non-obvious lessons.

### Step 3: Check Existing Skills

Before creating anything new, read the current skills in the user's repo:

```bash
ls ~/.claude/skills/
```

For each candidate pattern, check:
- Does an existing skill already cover this? Update it with the new example or rule.
- Does it fit as a new rule in an existing skill? Add it there.
- Is it genuinely a new category? Only then create a new skill.

Updating existing skills is better than creating new ones. More skills means more tokens loaded.

### Step 4: Write or Update Skills

For new skills, follow these rules from the Anthropic guide:

Frontmatter:
- name: kebab-case only, matches folder name
- description: [What it does] + [When to use it] + [Trigger phrases] + [Negative triggers if needed]
- Under 1024 characters total
- No XML angle brackets

SKILL.md body:
- Under 500 words for frequently-triggered skills
- Be specific and actionable ("Use Decimal for money" not "Be careful with numbers")
- Include common mistakes section
- Move heavy reference material to references/ folder

For updated skills:
- Add the new rule or example in the appropriate section
- Do not duplicate existing content
- Update the description if new triggers are needed

### Step 5: Quality Check

Before committing, verify each new or updated skill:

- [ ] Description includes trigger phrases users would actually say
- [ ] Description includes negative triggers to prevent false loading
- [ ] No project-specific details leaked (no IPs, keys, repo names, proprietary logic)
- [ ] Instructions are actionable (commands, code patterns, checklists) not vague ("be careful")
- [ ] Common mistakes section exists
- [ ] If over 500 words, heavy content moved to references/
- [ ] Works alongside existing skills without contradiction

### Step 6: Commit and Push

```bash
cd ~/claude-code-skills  # or wherever the skills repo lives
git add -A
git commit -m "Add/update skills from [project name] lessons"
git push
```

Symlink any new skills:
```bash
ln -s ~/claude-code-skills/new-skill-name/ ~/.claude/skills/new-skill-name
```

## What to Extract vs What to Skip

Extract:
- Bugs where the root cause was non-obvious
- Patterns that appeared 2+ times across the project
- Decisions where the wrong choice would have caused real damage
- Workarounds for platform/API quirks that are not documented elsewhere

Skip:
- Project-specific configuration or business logic
- Bugs caused by simple typos or copy-paste errors
- Patterns already well-documented in official docs
- Lessons that only apply to one specific technology version

## Suggesting Extraction to the User

At project milestones, you may suggest skill extraction by asking:

"We have [N] new bugs cataloged and [N] lessons learned since the last skill update. Would you like me to review them and update your skills repo?"

Do not extract without asking. The user decides when and what to capture.

## Common Mistakes

- Creating a new skill for every bug. Most bugs add a rule to an existing skill, not a new skill.
- Including project-specific details in skills. Skills must be generic. No server IPs, API keys, or proprietary logic.
- Writing vague instructions. "Handle errors properly" is not a skill. "Wrap every user-triggered operation in try/except and notify on the user-facing channel" is a skill.
- Extracting too early. Wait until a milestone. Patterns that seem important during active development often turn out to be one-off issues.
