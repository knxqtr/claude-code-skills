---
name: project-bootstrapping
description: Python project setup from scratch — folder structure, config, venv, git, testing, and tracking docs. Use when starting a new Python project or bot from zero. Use when user says "new project", "start from scratch", "set up project", "project structure", or "bootstrap".
---

# Project Bootstrapping

## When to Use

- Starting a new Python project from zero
- Setting up a bot, service, CLI tool, or any Python application
- Reviewing whether an existing project has the basics in place

## Setup Order

Do these in order. Each step depends on the previous one.

### 1. Create folder and git repo
```bash
mkdir my-project && cd my-project
git init
```

### 2. Create virtual environment
```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Create .gitignore (before first commit)
```
venv/
.env
__pycache__/
*.pyc
.DS_Store
*.log
data/
```

### 4. Create .env for secrets
```
API_KEY=xxx
BOT_TOKEN=xxx
```
Load with `python-dotenv`. Never commit this file.

### 5. Create config.py
Centralize all constants and settings in one file. Do not scatter magic numbers through the codebase.

```python
import os
from dotenv import load_dotenv

load_dotenv()

API_KEY = os.getenv("API_KEY")
POSITION_SIZE = 200
MAX_RETRIES = 3
```

### 6. Create a tracking document
A PROGRESS.md or similar file that tracks what is done and what is left. Update it as you build. This is your north star when things get complex.

### 7. First commit
```bash
git add .gitignore config.py PROGRESS.md
git commit -m "Initial project setup"
```

## Build Order for Services/Bots

Build in dependency order — each layer only depends on layers above it:

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

```python
# conftest.py — redirect all file I/O to temp directories
@pytest.fixture(autouse=True)
def isolate_io(tmp_path, monkeypatch):
    monkeypatch.setattr(module, "DATA_PATH", str(tmp_path / "data.json"))
```

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
