---
name: skill-extraction
description: Extracts reusable skills from project history. Use at version bumps or when asked to update skills.
disable-model-invocation: true
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

Priority order for where to put new knowledge:

1. Add to an existing skill's SKILL.md (if the skill is under 500 words)
2. Add to an existing skill's references/ file (if SKILL.md is already long)
3. Create a new references/ file in an existing skill (if no reference file fits)
4. Create a new skill (last resort — only if no existing skill covers this category)

For updates to existing skills:
- Add the new rule or example in the appropriate section
- If SKILL.md exceeds 500 words after the addition, move detailed content to references/
- Do not duplicate existing content
- Keep descriptions under 130 characters

For new skills (last resort), follow Anthropic's guide:

Frontmatter:
- name: kebab-case only, matches folder name
- description: under 130 characters, [What it does] + [When to use it]
- Under 1024 characters total, no XML angle brackets

SKILL.md body:
- Under 500 words
- Specific and actionable instructions, not vague guidance
- Include common mistakes section
- Heavy reference material goes in references/ folder

### Step 5: Quality Gate

Every candidate rule must pass all three tests before it can be committed. If it fails any test, rewrite it or discard it.

1. Can I give a concrete code example? If not, the rule is too vague. Rewrite with a specific before/after pattern, or discard.

2. Does it say what TO DO, not just what to avoid? "Don't use floats for money" is incomplete. "Use Decimal for money, not float" is actionable. If the rule only says what not to do, add what to do instead.

3. Would a developer who reads this actually change their behavior? If the answer is "they would have done this anyway," the rule is obvious and not worth adding. Skip it.

### Step 6: Quality Check

Before committing, verify each new or updated skill:

- [ ] Every rule passed the quality gate (all 3 tests)
- [ ] Checked all existing skills for overlap first
- [ ] No duplicate rules across skills
- [ ] Description includes trigger phrases and negative triggers
- [ ] No project-specific details leaked (no IPs, keys, repo names, proprietary logic)
- [ ] Instructions are actionable (commands, code patterns, checklists)
- [ ] Common mistakes section exists
- [ ] If over 500 words, heavy content moved to references/
- [ ] Update README.md in skills repo with any new skills

### Step 7: Log the Extraction

Append to ~/.claude/skills/extraction-log.md with what was considered and what was decided. This creates a searchable history across all projects.

Format:
```markdown
## [Project Name], v[Version] ([Date])
- ADDED to [skill-name]: "[rule summary]"
- UPDATED [skill-name]: "[what changed]"
- SKIPPED: "[lesson]" — [reason: already covered by X / too vague / too obvious]
- NEW SKILL: [skill-name] — "[why no existing skill fit]"
```

If the same lesson gets SKIPPED 3+ times across different projects, flag it. Either the existing skill's rule is not clear enough, or it needs to be rewritten.

### Step 8: Commit and Push

```bash
cd ~/.claude/skills
git add -A
git commit -m "Add/update skills from [project name] v[version] lessons"
git push
```

## Skill Count Limit

Before creating a new skill, check the current count:

```bash
ls -d ~/.claude/skills/*/  | wc -l
```

If the count reaches 40, stop and notify the user. Do not create the new skill. Instead, ask the user how they would like to proceed — options include consolidating related skills, removing unused ones, or accepting the higher token cost.

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
