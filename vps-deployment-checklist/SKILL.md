---
name: vps-deployment-checklist
description: VPS deployment checklist for Python services including systemd, rsync, and remote setup. Use when deploying a bot or service to a Linux server, setting up auto-restart, or debugging why a service fails on the server but works locally. Covers run scripts, venv portability, lock files, and heartbeats.
---

# VPS Deployment Checklist

## When to Use

- Deploying a Python bot or service to a Linux server
- Setting up auto-restart with systemd
- Syncing code from local machine to remote server
- Debugging why a service fails on the server but works locally

## Pre-Deployment

### 1. Run script must activate venv

systemd does not source your shell profile. The venv must be activated explicitly in your run script.

```bash
#!/bin/bash
cd /home/botuser/my-project
source venv/bin/activate
python main.py
```

### 2. rsync must exclude local-only files

```bash
rsync -avz --exclude='venv' \
           --exclude='*.log' \
           --exclude='data/' \
           --exclude='.env' \
           --exclude='__pycache__' \
           --exclude='.git' \
           -e "ssh -p 2222" \
           . user@server:/home/user/my-project/
```

Never sync venv (Python versions may differ), logs, or runtime data. Create venv on the server separately.

### 3. Server Python version may differ

Your local Mac may run Python 3.11, the server may run 3.12. Venvs are NOT portable across Python versions. Always create a fresh venv on the server:

```bash
ssh user@server
cd /home/user/my-project
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## systemd Service

```ini
[Unit]
Description=My Bot Service
After=network.target

[Service]
Type=simple
User=botuser
WorkingDirectory=/home/botuser/my-project
ExecStart=/home/botuser/my-project/run.sh
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Key settings:
- Restart=always: auto-restart on crash
- RestartSec=5: wait 5 seconds before restart (prevents tight crash loops)
- User: run as a non-root user

### Commands
```bash
sudo systemctl enable my-bot      # start on boot
sudo systemctl start my-bot       # start now
sudo systemctl restart my-bot     # restart
sudo systemctl stop my-bot        # stop
sudo journalctl -u my-bot -f     # view logs
```

### 4. Secrets must not be CLI arguments

Never pass secrets (API tokens, passwords) as command-line arguments. They are visible to every user on the server via `ps aux`.

Instead, write secrets to a config file with restricted permissions:
```bash
cat > ~/my-project/credentials.conf << 'EOF'
api_token=secret_value
EOF
chmod 600 ~/my-project/credentials.conf
```

Then pass the file path: `--config-file ~/my-project/credentials.conf`

### 5. Uploaded scripts must live inside the package

If your deployment uploads helper scripts (watchdog, health check) to the server, keep them inside the Python package (e.g., `src/myapp/data/script.py`), not in a top-level `scripts/` directory. Relative path traversal from `__file__` breaks in wheel installs because the package tree is different from the source tree.

## Safety Mechanisms

### Lock file
Prevents two instances from running simultaneously (see async-safety-checklist skill).

### Heartbeat
Send a periodic "I'm alive" message (every 12-24 hours). If you stop receiving heartbeats, something is wrong.

### Graceful vs hard restart
- Graceful: process saves state, exits cleanly, systemd restarts it
- Hard kill: SIGKILL, no cleanup. Use only when graceful restart hangs.
- Consider separate commands: /restart (graceful) and /kill (hard)

## Common Mistakes

- Forgetting to activate venv in run.sh. The service starts with system Python and missing packages.
- Syncing local venv to server. It was built for a different Python version and architecture.
- Running as root. Use a dedicated service user with minimal permissions.
- No RestartSec. Service crashes, restarts instantly, crashes again, thousands of times per minute.
- Syncing .env with rsync. Server credentials differ from local ones. Create .env on the server manually.
