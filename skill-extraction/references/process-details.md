# Skill Extraction Process Details

## Step 1: Review the Source Material

Read the project's PROJECT_HISTORY.md in this order of value:
1. Bug catalog (highest value -- every bug with a root cause is a potential skill rule)
2. Lessons learned (already written as transferable principles)
3. Decision rationale (non-obvious choices that should be repeated)
4. Implementation sequence (build order patterns that worked)

If no PROJECT_HISTORY.md exists, create one first using the project-history-documentation skill.

## Step 2: Check Existing Skills First

```bash
ls ~/.claude/skills/
```

Read the SKILL.md of every existing skill that could be related. For each candidate pattern from the project history, ask:

1. Does an existing skill already have a rule covering this exact scenario? If yes, skip.
2. Does an existing skill cover the same category but is missing this specific case? If yes, add the new rule/example to that skill.
3. Is this a genuinely new category not covered by any existing skill? Only then create a new skill.

## Step 3: Identify Transferable Patterns

For each unmatched bug, lesson, or decision, ask:

- Would this apply to a different project built with different technology? If yes, strong candidate.
- Would this apply to a similar project? If yes, candidate.
- Is this obvious to any experienced developer? If yes, skip.
- Was the root cause non-obvious? If yes, strong candidate.
- Did this pattern appear 2+ times in this project? If yes, strong candidate.

## Step 4: Write or Update Skills

Priority order for where to put new knowledge:

1. Add to an existing skill's SKILL.md (if the skill is under 500 words)
2. Add to an existing skill's references/ file (if SKILL.md is already long)
3. Create a new references/ file in an existing skill (if no reference file fits)
4. Create a new skill (last resort -- only if no existing skill covers this category)

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

## Step 5: Check Skill Count

```bash
ls -d ~/.claude/skills/*/  | wc -l
```

If the count reaches 40, stop and notify the user.

## Step 6: Log the Extraction

Append to ~/.claude/skills/extraction-log.md with what was considered and what was decided.

Format:
```markdown
## [Project Name], v[Version] ([Date])
- ADDED to [skill-name]: "[rule summary]"
- UPDATED [skill-name]: "[what changed]"
- SKIPPED: "[lesson]" -- [reason: already covered by X / too vague / too obvious]
- NEW SKILL: [skill-name] -- "[why no existing skill fit]"
```

If the same lesson gets SKIPPED 3+ times across different projects, flag it. Either the existing skill's rule is not clear enough, or it needs to be rewritten.

## Step 7: Commit and Push

```bash
cd ~/.claude/skills
git add -A
git commit -m "Add/update skills from [project name] v[version] lessons"
git push
```
