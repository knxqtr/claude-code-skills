---
name: project-bootstrapping
description: Python project setup from scratch including folder structure, config centralization, venv, git, and testing. Use when starting a new bot, service, CLI tool, or any Python application. Covers build order, .gitignore, .env secrets, and test isolation.
disable-model-invocation: true
---

# Project Bootstrapping

## When to Use

- Starting a new Python project from zero
- Setting up a bot, service, CLI tool, or any Python application
- Reviewing whether an existing project has the basics in place

## Setup Order

Do these in order. Each step depends on the previous one.

1. Create folder and git repo
2. Create virtual environment
3. Create .gitignore (before first commit) -- include venv/, .env, __pycache__/, *.pyc, .DS_Store, *.log, data/
4. Create .env for secrets -- load with python-dotenv, never commit this file
5. Create config.py -- centralize all constants and settings in one file, do not scatter magic numbers
6. Create a tracking document (PROGRESS.md) -- tracks what is done and what is left
7. First commit -- add .gitignore, config.py, PROGRESS.md

## Build Order for Services/Bots

Build in dependency order -- each layer only depends on layers above it:

```
1. config.py          -- all settings centralized
2. Data layer         -- mappers, parsers, data models
3. External clients   -- API wrappers, database connections
4. Notifications      -- separate module, not scattered in business logic
5. Business logic     -- the core coordinator
6. User interface     -- commands, handlers, web routes
7. Orchestration      -- main.py, startup sequence, schedulers
```

## Testing from Day One

- Isolate test I/O from production data using tmp_path
- Start with happy-path tests, add edge cases as you find real bugs
- Deduplicate test utilities into tests/helpers.py when 3+ files share the same mock

## Folder Structure

```
my-project/
  config.py
  main.py
  src/              # or flat files for small projects
  tests/
    conftest.py
    helpers.py
    test_*.py
  data/             # runtime data (gitignored)
  .env              # secrets (gitignored)
  .gitignore
  requirements.txt
  PROGRESS.md
```

For small projects (under 10 files), flat structure is fine. Do not over-organize before you have code.

## Common Mistakes

- Installing packages without a venv. Dependencies leak into the system Python.
- Scattering config values across multiple files. When a value needs changing, you hunt for it everywhere.
- No .gitignore before first commit. Secrets or venv end up in git history permanently.
- Over-engineering folder structure for a 3-file project. Start flat, split when a file gets hard to navigate.

For setup commands and config code, see `references/setup-commands.md`.
