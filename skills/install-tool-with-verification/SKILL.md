---
name: install-tool-with-verification
description: Use when installing any new tool/package from GitHub, Brew, or Cask — includes post-install verification script, automated health check cron, and deliverable report so the user can independently confirm the tool is in active daily use.
version: 1.0.0
tags:
  - devops
  - installation
  - verification
  - cron
  - health-check
  - accountability
  triggers:
    - install tool
    - brew install
    - npm install
    - pip install

---

# Install Tool with Verification

## Core Principle

> Installing a tool is only half the work. The user needs to know it's actually being used, not just sitting in a directory.

Every tool installation must produce **verifiable evidence** of operation and a **persistent monitoring mechanism** so the user can independently confirm the tool is live — without needing to SSH in or check manually.

## The Three-Layer Verification Pyramid

```
         ┌──────────────┐
         │  User-facing  │  ← Daily cron report delivered to user
         │  cron report  │
         ├──────────────┤
         │  Health check │  ← Script that tests each tool independently
         │  script       │
         ├──────────────┤
         │  Immediate    │  ← Right after install: run once, show output
         │  proof of run │
         └──────────────┘
```

## Workflow

### Step 1: Install the Tool

```bash
# Examples:
brew install restic
brew install --cask agentsview
brew install ffmpeg
git clone https://github.com/user/project.git ~/GitHub_Projects/project
pip install sia-agent
```

Always verify the binary is findable right after:

```bash
which restic          # or command -v restic
restic --help | head -3
```

### Step 2: Immediate Proof of Run

Run the tool immediately with a simple command that produces output the user can see:

```bash
# Example for restic:
restic -r ~/Backups/restic-repo init   # creates repo
restic -r ~/Backups/restic-repo backup --verbose /path/to/data

# Example for agentsview:
agentsview health
agentsview sync  # if applicable
```

Show this output to the user so they can see "it works".

### Step 3: Write a Health Check Script

Create `~/.hermes/scripts/health-check-{toolname}.sh` that independently verifies the tool is operational.

**Script pattern:**

```bash
#!/bin/bash
set -euo pipefail

PASS=0; FAIL=0; WARN=0

# Check each tool independently
if command -v restic &>/dev/null; then
  # Run a real check: does it have data?
  SNAPSHOT_COUNT=$(restic -r "$REPO" snapshots --compact 2>/dev/null | grep -cE '^[a-f0-9]{8}' || true)
  if [ "$SNAPSHOT_COUNT" -gt 0 ]; then
    echo "  ✅ restic: installed, $SNAPSHOT_COUNT snapshots"
    PASS=$((PASS+1))
  else
    echo "  ⚠️  restic: installed but no data"
    WARN=$((WARN+1))
  fi
else
  echo "  ❌ restic: not installed"
  FAIL=$((FAIL+1))
fi

# ... repeat for each tool ...

echo "Results: ✅ $PASS | ⚠️  $WARN | ❌ $FAIL"
exit $FAIL
```

### Step 4: Bridge Health Check into a Cron Job

Use the `cronjob` tool to schedule the health check daily. The cron delivers the result directly to the user:

```yaml
schedule: "0 9 * * *"   # daily 9am
name: "[Project] Daily Health Check"
prompt: |
  Run the health check script at ~/.hermes/scripts/health-check.sh
  Report the results to the user with any recommendations.
```

### Step 5: Report to User

Tell the user concretely how they will know the tool is working tomorrow:

> "You'll get a daily message at 9am showing tool status."
> "You can also check manually: `[command]`"

## Pitfalls to Avoid

### ❌ "Installed, trust me" syndrome
Installing and assuming the user knows it works. Always produce visible proof.

### ❌ Only manual verification
If the user has to actively check, they won't. Make it push-based (cron delivery).

### ❌ Health check that requires user interaction
The health check script must run unattended (no `sudo` prompts, no interactive input, no password files). If a tool needs credentials, either:
- Store them as env vars in the cron job config
- Hardcode them in the script (only for low-risk local-only tools)
- Skip that tool's check if credentials aren't available

### ❌ Forgetting to add the tool to the installed-list memory
After install, update memory with the tool name so future GitHub reporting doesn't suggest installing it again.

## Comparison Table

| Stage | What | Who Sees It | Duration |
|-------|------|-------------|----------|
| Step 2 | Immediate proof of run | User right now | Instant |
| Step 3 | Health check script | Agent runs it | Ongoing |
| Step 4 | Daily cron health report | User receives daily | Forever until disabled |
| Memory | Tool added to installed list | Agent refers to it | Cross-session |

## Real Example (from 2026-06-12 session)

```bash
# After installing restic + agentsview + cloning superpowers/sia:
# 1. Restic backup ran immediately: 28.5MB, 1058 files
# 2. agentsview health returned OK
# 3. Health check script created: ~/.hermes/scripts/health-check.sh
# 4. Cron job scheduled: "System Daily Health Check" at 9am daily
# 5. Memory updated with new tools on the installed list
```
