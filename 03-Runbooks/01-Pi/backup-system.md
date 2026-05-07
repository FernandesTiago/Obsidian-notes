# Pi Backup System

Automated incremental backup from Pi to notebook over SSH/rsync.

## Objective

Daily backup of critical Pi data (configs, home, package list, crontabs) to the notebook. Survives SD card death without losing WireGuard keys, Pi-hole config, or service state.

## Architecture

Three-layer plan, layer 1 active:

- **Layer 1 — rsync to notebook** ✅ active
- **Layer 2 — Mega.nz weekly** ⏳ planned
- **Layer 3 — SD card USB biweekly** ⏳ planned

Layer 1 details:
- Script runs on Pi as **root** (needs read access to `/etc/wireguard`, `/etc/pihole`, etc)
- SSH from Pi root → notebook user via key auth (no password)
- rsync with `--link-dest` for hardlink-based incremental snapshots
- One snapshot per day, located at `~/backups/pi/YYYY-MM-DD` on the notebook
- Cron triggers hourly; script skips if notebook offline or backup already done today
- Snapshots older than 30 days are deleted automatically

## State

- Script: `/root/scripts/backup-to-notebook.sh` (mode 700, owner root)
- Log: `/root/scripts/backup.log`
- SSH key: `/root/.ssh/id_ed25519` (root → notebook)
- Cron: `sudo crontab -l` → runs hourly at minute 0
- Destination: `<NOTEBOOK_USER>@<NOTEBOOK_IP>:~/backups/pi/`

See [[IPs]] for real values.

## Setup

### 1. Generate SSH key for root on Pi

```bash
sudo ssh-keygen -t ed25519 -C "pi-root-to-notebook" -f /root/.ssh/id_ed25519 -N ""
```

`-N ""` skips passphrase (required for unattended cron execution).

### 2. Copy public key to notebook

```bash
sudo ssh-copy-id -i /root/.ssh/id_ed25519.pub <NOTEBOOK_USER>@<NOTEBOOK_IP>
```

Notebook IP via Tailscale (`100.x.x.x`) so it works regardless of which network the notebook is on.

### 3. Test root SSH

```bash
sudo ssh <NOTEBOOK_USER>@<NOTEBOOK_IP> "echo ok"
```

Must return `ok` without prompting for password.

### 4. Install script

```bash
sudo mkdir -p /root/scripts
sudo mv ~/scripts/backup-to-notebook.sh /root/scripts/
sudo chown root:root /root/scripts/backup-to-notebook.sh
sudo chmod 700 /root/scripts/backup-to-notebook.sh
```

### 5. Schedule via root crontab

```bash
sudo crontab -e
```

```cron
0 * * * * /root/scripts/backup-to-notebook.sh
```

User crontab (`crontab -e` without sudo) is for bots and speedtest. Backup lives in root crontab because it needs root privileges.

## Script

Located at `/root/scripts/backup-to-notebook.sh`:

```bash
#!/bin/bash
set -euo pipefail

REMOTE="<NOTEBOOK_USER>@<NOTEBOOK_IP>"
DEST="~/backups/pi"
DATE=$(date +%Y-%m-%d)
LOG="/root/scripts/backup.log"

# Skip if notebook unreachable
if ! ssh -o ConnectTimeout=5 "$REMOTE" "exit" 2>/dev/null; then
    echo "$(date) — Notebook offline, pulando backup" >> "$LOG"
    exit 0
fi

# Skip if today's backup already exists
if ssh "$REMOTE" "[ -d $DEST/$DATE ]" 2>/dev/null; then
    echo "$(date) — Backup de hoje já existe, pulando" >> "$LOG"
    exit 0
fi

ssh "$REMOTE" "mkdir -p $DEST/$DATE"

rsync -aHAX --delete \
    --link-dest="$DEST/latest" \
    --exclude='.cache' \
    --exclude='node_modules' \
    --exclude='__pycache__' \
    /etc /home/<PI_USER> \
    "$REMOTE:$DEST/$DATE/" 2>> "$LOG"

# Package list
dpkg --get-selections > /tmp/pi-packages.txt
scp /tmp/pi-packages.txt "$REMOTE:$DEST/$DATE/" 2>> "$LOG"
rm /tmp/pi-packages.txt

# Crontabs (user + root)
sudo -u <PI_USER> crontab -l > /tmp/pi-crontab-user.txt 2>/dev/null || true
crontab -l > /tmp/pi-crontab-root.txt 2>/dev/null || true
scp /tmp/pi-crontab-user.txt /tmp/pi-crontab-root.txt "$REMOTE:$DEST/$DATE/" 2>> "$LOG"
rm /tmp/pi-crontab-*.txt

# Update "latest" symlink for next run's --link-dest
ssh "$REMOTE" "cd $DEST && rm -f latest && ln -s $DATE latest"

# Prune snapshots older than 30 days
ssh "$REMOTE" "find $DEST -maxdepth 1 -type d -mtime +30 -name '20*' -exec rm -rf {} +"

echo "$(date) — Backup OK em $REMOTE:$DEST/$DATE" >> "$LOG"
```

## What's backed up

- `/etc` (entire directory — includes WireGuard keys, Pi-hole config, sudoers, SSH host keys, UFW rules, NetworkManager connections)
- `/home/<PI_USER>` (excluding `.cache`, `node_modules`, `__pycache__`)
- `dpkg --get-selections` (full package list)
- User and root crontabs

## What's NOT backed up

- `/var/log` — logrotate handles rotation
- Caches and node_modules — recoverable via reinstall
- Secrets in Bitwarden — already managed there

## Verify

```bash
sudo cat /root/scripts/backup.log              # recent activity
ssh <NOTEBOOK_USER>@<NOTEBOOK_IP> "ls ~/backups/pi/"          # snapshot dates
ssh <NOTEBOOK_USER>@<NOTEBOOK_IP> "du -sh ~/backups/pi/*"     # size per snapshot
```

First snapshot will be ~180MB. Subsequent snapshots use hardlinks via `--link-dest`, so they take only the delta — typically a few MB.

## Restore

If the SD card dies:

1. Flash a new SD with the same OS version
2. Boot, restore network access (SSH or HDMI)
3. Pull the most recent snapshot:
```bash
   rsync -aHAX <NOTEBOOK_USER>@<NOTEBOOK_IP>:~/backups/pi/latest/ /tmp/restore/
```
4. Restore configs:
```bash
   sudo cp -r /tmp/restore/etc/* /etc/
   sudo cp -r /tmp/restore/<PI_USER>/* /home/<PI_USER>/
```
5. Reinstall packages:
```bash
   sudo dpkg --set-selections < /tmp/restore/pi-packages.txt
   sudo apt-get dselect-upgrade
```
6. Restore crontabs:
```bash
   crontab /tmp/restore/pi-crontab-user.txt
   sudo crontab /tmp/restore/pi-crontab-root.txt
```
7. Restart services: `pihole-FTL`, `wg-quick@wg0`, `ssh`, etc.

**This restore procedure has not been tested yet.** Untested backup is hope, not backup. Run a dry restore on a spare SD before relying on it.

## Troubleshoot

### Permission denied on `/etc/wireguard`, `/etc/pihole`, `/etc/sudoers`, etc

**Symptom:** rsync log filled with `send_files failed: Permission denied (13)`. Critical configs missing from snapshot.

**Cause:** script running as regular user (`<PI_USER>`) instead of root. Files like `/etc/wireguard/*.conf`, `/etc/shadow`, `/etc/sudoers` are root-readable only.

**Fix:** migrate to root execution:
- Generate SSH key in `/root/.ssh/`
- Copy public key to notebook (`sudo ssh-copy-id`)
- Move script to `/root/scripts/`
- Schedule via `sudo crontab -e`, not user crontab
- Remove the corresponding line from user crontab to avoid duplicate runs

This was the original failure mode before the migration to root.

### `--link-dest arg does not exist: ~/backups/pi/latest`

**Symptom:** warning on first execution.

**Cause:** the `latest` symlink doesn't exist yet on the very first run. Harmless.

**Fix:** ignore. From the second run onward, the symlink is created and incremental hardlinks work correctly.

### `IO error encountered -- skipping file deletion`

**Symptom:** rsync prints this near the end.

**Cause:** rsync hit a transient I/O error during sync and refused to delete files in `--delete` mode (safety mechanism — won't delete if the source side had errors).

**Fix:** usually harmless if the snapshot itself completed. Replace `--delete` with `--delete-after` to silence it (deletes only after sync finishes successfully).

## Revert

To disable the backup:

```bash
sudo crontab -e   # remove the backup line
```

To remove entirely:

```bash
sudo rm -rf /root/scripts/backup-to-notebook.sh /root/scripts/backup.log
sudo rm /root/.ssh/id_ed25519 /root/.ssh/id_ed25519.pub
# Optionally remove the public key from the notebook's ~/.ssh/authorized_keys
```

## References

- [[IPs]] — real IPs, usernames, paths
- [[ssh-config]] — SSH alias setup (planned note)
- [[duckdns]] — separate cron jobs (user crontab)
- [[speedtest]] — separate cron jobs (user crontab)