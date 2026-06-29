---
name: macos-backup-overwrite-only
description: Use when setting up daily macOS backup that overwrites rather than accumulates.
tags: [macos, backup, launchd, shell-script]
  triggers:
    - backup
    - macos backup
    - daily backup

---

# macOS Daily Backup — Overwrite Mode (No Accumulation)

Uses macOS native launchd scheduling to daily backup `~/.hermes` and directories to a fixed location, overwriting each time without keeping history.

## Architecture

```
~/Library/LaunchAgents/com.hermes.dailybackup.plist   ← launchd schedule
~/.hermes/scripts/backup.sh                            ← backup script
~/Backups/Hermes-latest/                               ← target location (overwrite each time)
```

## launchd plist

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.hermes.dailybackup</string>
    <key>ProgramArguments</key>
    <array>
        <string>/path/to/backup.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>0</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    <key>RunAtLoad</key>
    <false/>
    <key>StandardOutPath</key>
    <string>/path/to/log.log</string>
    <key>StandardErrorPath</key>
    <string>/path/to/log.log</string>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/usr/local/bin:/usr/bin:/bin:/opt/homebrew/bin</string>
    </dict>
</dict>
</plist>
```

## Backup Script Template

```bash
#!/bin/bash
# Daily backup, overwrite a fixed location each time

BACKUP_ROOT="/path/to/backup_root"
BACKUP_DIR="${BACKUP_ROOT}/BackupName-latest"
LOG_FILE="${BACKUP_ROOT}/backup.log"

echo "[$(date '+%Y-%m-%d %H:%M:%S')] === Starting Backup ===" >> "$LOG_FILE"

# 1. Clear old backup (overwrite each time)
if [ -d "$BACKUP_DIR" ]; then
    rm -rf "$BACKUP_DIR"
fi

# 2. Create directory structure
mkdir -p "${BACKUP_DIR}/subdir"

# 3. rsync backup (exclude large directories)
rsync -a --delete \
    --exclude='venv/' \
    --exclude='__pycache__/' \
    --exclude='node_modules/' \
    --exclude='.git/' \
    /source/path/ \
    "${BACKUP_DIR}/subdir/"

# 4. Log size
DU_SIZE=$(du -sh "$BACKUP_DIR" 2>/dev/null | cut -f1)
echo "[$(date '+%Y-%m-%d %H:%M:%S')] ✅ Backup complete — Size: ${DU_SIZE}" >> "$LOG_FILE"
echo "---" >> "$LOG_FILE"
```

## Load / Reload launchd
```bash
launchctl unload ~/Library/LaunchAgents/com.hermes.dailybackup.plist 2>/dev/null
launchctl load ~/Library/LaunchAgents/com.hermes.dailybackup.plist
```

## Testing
```bash
bash /path/to/backup.sh
```
Check target folder and log.
