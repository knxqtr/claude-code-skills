---
name: project-claude-template
description: Generates a minimal project CLAUDE.md with auto-detected build/test/lint commands. Use when creating a new project or when an existing project is missing a CLAUDE.md. Do not use if CLAUDE.md already exists.
disable-model-invocation: true
---

# Project CLAUDE.md Template

Generate a CLAUDE.md for the current project. This file should contain ONLY project-specific content. Everything in the global ~/.claude/CLAUDE.md applies automatically and must NOT be duplicated.

## Pre-Check

If a CLAUDE.md or .claude/CLAUDE.md already exists in the project root, STOP. Do not overwrite or merge. Tell the user the file already exists.

## Step 1: Detect Project Name

Use the current folder name as the project title. Convert hyphens and underscores to spaces, title case. For example: `trading-bot` becomes `Trading Bot`, `vault_app` becomes `Vault App`.

## Step 2: Detect Build, Test, and Lint Commands

Scan the project root for config files in this order. Check ALL of them, not just the first match. Only include command lines that were actually found.

### Python
- pyproject.toml exists:
  - Test: look for [tool.pytest] section. If found, use `pytest`
  - Lint: look for [tool.ruff] section. If found, use `ruff check .`
  - Build: look for [build-system] or [project.scripts]. If found, use `pip install -e .`
- setup.py or setup.cfg exists (and no pyproject.toml):
  - Test: if tests/ directory exists, use `pytest`
  - Build: use `pip install -e .`
- requirements.txt exists (and no pyproject.toml, setup.py, or setup.cfg):
  - Test: if tests/ directory exists, use `pytest`

### JavaScript / TypeScript
- package.json exists:
  - Read the "scripts" object
  - Build: use value of scripts.build if it exists, run with `npm run build`
  - Test: use value of scripts.test if it exists, run with `npm test`
  - Lint: use value of scripts.lint if it exists, run with `npm run lint`

### Rust
- Cargo.toml exists:
  - Build: `cargo build`
  - Test: `cargo test`
  - Lint: `cargo clippy`

### Go
- go.mod exists:
  - Build: `go build ./...`
  - Test: `go test ./...`
  - Lint: `golangci-lint run`

### Makefile
- Makefile exists:
  - Read targets. If a `build` target exists, use `make build`
  - If a `test` target exists, use `make test`
  - If a `lint` target exists, use `make lint`

If multiple config files are found (e.g., pyproject.toml AND package.json in a full-stack project), include commands from all of them, grouped under subheadings by language.

## Step 3: Generate the CLAUDE.md

Write the file to `./CLAUDE.md` in the project root.

### If build/test/lint commands were detected (single language):

```
# [Project Name]

## Build and Test

- Build: `command`
- Test: `command`
- Lint: `command`
```

Only include lines where a command was actually detected. If build was found but lint was not, only list build and test.

### If build/test/lint commands were detected (multiple languages):

```
# [Project Name]

## Build and Test

### Python
- Test: `pytest`
- Lint: `ruff check .`

### JavaScript
- Build: `npm run build`
- Test: `npm test`
```

### If NO config files were found:

```
# [Project Name]
```

Just the title. No empty sections, no placeholders, no instructions to fill in later.

## What This File Must NEVER Contain

These are handled by the global ~/.claude/CLAUDE.md:

- Communication style rules (plain English, no bold/italic)
- Safety rules (no pushing, no code changes without asking, no reading secrets)
- Core Principles (simplicity, no laziness, minimal impact)
- Verification Before Done
- Documentation rules
- Skills/workflow rules
- Venv activation

If it is in the global file, it does not go in the project file.
