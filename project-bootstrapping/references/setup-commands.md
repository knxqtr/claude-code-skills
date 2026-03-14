# Project Bootstrapping — Setup Commands

## 1. Create Folder and Git Repo

```bash
mkdir my-project && cd my-project
git init
```

## 2. Create Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate
```

## 3. .gitignore Contents

```
venv/
.env
__pycache__/
*.pyc
.DS_Store
*.log
data/
```

## 4. .env Template

```
API_KEY=xxx
BOT_TOKEN=xxx
```

Load with `python-dotenv`. Never commit this file.

## 5. config.py Template

```python
import os
from dotenv import load_dotenv

load_dotenv()

API_KEY = os.getenv("API_KEY")
POSITION_SIZE = 200
MAX_RETRIES = 3
```

## 6. First Commit

```bash
git add .gitignore config.py PROGRESS.md
git commit -m "Initial project setup"
```

## 7. conftest.py — Test I/O Isolation

```python
# Redirect all file I/O to temp directories
@pytest.fixture(autouse=True)
def isolate_io(tmp_path, monkeypatch):
    monkeypatch.setattr(module, "DATA_PATH", str(tmp_path / "data.json"))
```
