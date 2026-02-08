---
description: "Backup and restore recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-backup with source, destination, and context included"
when-to-use: "Receives prepared arguments from task-backup with source, destination, and context included"
context: fork
model: inherit
allowed-tools: Read
---

# Backup and Restore Recipes

**Tools:** restic, borg, rsync, rclone, duplicity

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Tool | Command |
|------|------|---------|
| Init backup repo | restic | `restic init --repo /backup` |
| Create backup | restic | `restic backup /data --repo /backup` |
| List snapshots | restic | `restic snapshots --repo /backup` |
| Restore latest | restic | `restic restore latest --target /restore --repo /backup` |
| Init borg repo | borg | `borg init --encryption=repokey /backup` |
| Create borg backup | borg | `borg create /backup::name /data` |
| Mirror with rsync | rsync | `rsync -av --delete /data/ /backup/` |
| Cloud sync | rclone | `rclone sync /data remote:backup` |

---

## When to Use What

| Tool | Best For | Limitation |
|------|----------|------------|
| restic | Fast encrypted dedup backups, cloud support | No compression options (built-in) |
| borg | Encrypted dedup with compression control | Slower remote, no native cloud |
| rsync | Simple file mirroring and sync | No dedup, no encryption, no versioning |
| rclone | Cloud storage sync (40+ providers) | Not a true backup tool (no versions) |
| duplicity | GPG-encrypted incremental to cloud | Slower, older codebase |
| tar + cron | Simple scheduled archive backups | No dedup, no incremental |
| pg_dump / mysqldump | Database-specific logical backups | Not for file backups |

---

## restic

### Initialize Repository
```bash
# Local repository
restic init --repo /mnt/backup/restic-repo

# SFTP repository
restic init --repo sftp:user@host:/backup/restic-repo

# S3 repository
export AWS_ACCESS_KEY_ID="key"
export AWS_SECRET_ACCESS_KEY="secret"
restic init --repo s3:s3.amazonaws.com/bucket-name

# Backblaze B2
export B2_ACCOUNT_ID="id"
export B2_ACCOUNT_KEY="key"
restic init --repo b2:bucket-name:/restic

# REST server
restic init --repo rest:http://host:8000/
```

### Create Backup
```bash
# Basic backup
restic backup /home/user/data --repo /mnt/backup/restic-repo

# With excludes
restic backup /home/user --repo /mnt/backup/restic-repo \
  --exclude="node_modules" --exclude=".cache" --exclude="*.tmp"

# Exclude from file
restic backup /home/user --repo /mnt/backup/restic-repo \
  --exclude-file=/home/user/.restic-excludes

# With tags
restic backup /home/user --repo /mnt/backup/restic-repo --tag daily --tag automated

# Backup multiple paths
restic backup /home/user/docs /home/user/photos --repo /mnt/backup/restic-repo

# Dry run (show what would be backed up)
restic backup /home/user --repo /mnt/backup/restic-repo --dry-run -v

# Using password file (for scripts)
restic backup /data --repo /mnt/backup/restic-repo --password-file /root/.restic-pw
```

### Browse and Restore
```bash
# List snapshots
restic snapshots --repo /mnt/backup/restic-repo

# List snapshots with tags
restic snapshots --repo /mnt/backup/restic-repo --tag daily

# Browse snapshot contents
restic ls latest --repo /mnt/backup/restic-repo
restic ls latest --repo /mnt/backup/restic-repo /home/user/docs/

# Restore latest snapshot
restic restore latest --target /tmp/restore --repo /mnt/backup/restic-repo

# Restore specific snapshot
restic restore abc12345 --target /tmp/restore --repo /mnt/backup/restic-repo

# Restore specific path from snapshot
restic restore latest --target /tmp/restore --include "/home/user/docs" --repo /mnt/backup/restic-repo

# Mount snapshots as filesystem (browse interactively)
mkdir /mnt/restic-mount
restic mount /mnt/restic-mount --repo /mnt/backup/restic-repo
# Browse at /mnt/restic-mount/snapshots/
fusermount -u /mnt/restic-mount   # Unmount when done

# Diff between snapshots
restic diff snap1 snap2 --repo /mnt/backup/restic-repo
```

### Forget and Prune (Retention)
```bash
# Remove specific snapshot
restic forget abc12345 --repo /mnt/backup/restic-repo

# Apply retention policy (keep 7 daily, 4 weekly, 12 monthly)
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 12 \
  --repo /mnt/backup/restic-repo

# Forget + prune in one step (reclaim disk space)
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune \
  --repo /mnt/backup/restic-repo

# Prune unreferenced data
restic prune --repo /mnt/backup/restic-repo

# Check repository integrity
restic check --repo /mnt/backup/restic-repo
restic check --read-data --repo /mnt/backup/restic-repo   # Verify all data (slow)
```

### Repository Statistics
```bash
restic stats --repo /mnt/backup/restic-repo
restic stats --mode raw-data --repo /mnt/backup/restic-repo
restic cat config --repo /mnt/backup/restic-repo
```

---

## borg

### Initialize Repository
```bash
# Encrypted (recommended)
borg init --encryption=repokey /mnt/backup/borg-repo

# Encrypted with key file
borg init --encryption=keyfile /mnt/backup/borg-repo

# No encryption (fastest)
borg init --encryption=none /mnt/backup/borg-repo

# Remote repository
borg init --encryption=repokey ssh://user@host/path/to/repo
```

### Create Backup
```bash
# Basic backup with timestamp name
borg create /mnt/backup/borg-repo::$(date +%Y-%m-%d) /home/user

# With compression
borg create --compression lz4 /mnt/backup/borg-repo::daily-$(date +%Y-%m-%d) /home/user
borg create --compression zstd,3 /mnt/backup/borg-repo::daily-$(date +%Y-%m-%d) /home/user

# With excludes
borg create /mnt/backup/borg-repo::backup-$(date +%Y-%m-%d) /home/user \
  --exclude '*.cache' --exclude 'node_modules' --exclude '.git'

# Show progress
borg create --progress --stats /mnt/backup/borg-repo::backup /home/user

# Dry run
borg create --dry-run --list /mnt/backup/borg-repo::test /home/user
```

### List and Extract
```bash
# List all archives
borg list /mnt/backup/borg-repo

# List contents of archive
borg list /mnt/backup/borg-repo::daily-2024-01-15

# Extract full archive
borg extract /mnt/backup/borg-repo::daily-2024-01-15

# Extract specific path
borg extract /mnt/backup/borg-repo::daily-2024-01-15 home/user/docs

# Extract to specific directory
cd /tmp/restore && borg extract /mnt/backup/borg-repo::daily-2024-01-15

# Show archive info
borg info /mnt/backup/borg-repo::daily-2024-01-15
```

### Prune (Retention)
```bash
# Apply retention policy
borg prune --keep-daily=7 --keep-weekly=4 --keep-monthly=12 /mnt/backup/borg-repo

# Dry run first
borg prune --dry-run --list --keep-daily=7 --keep-weekly=4 /mnt/backup/borg-repo

# Compact after pruning (borg 1.2+)
borg compact /mnt/backup/borg-repo

# Verify integrity
borg check /mnt/backup/borg-repo
borg check --verify-data /mnt/backup/borg-repo   # Thorough check
```

---

## rsync Backup Strategies

### Basic Mirror
```bash
# Mirror source to destination (delete extras)
rsync -av --delete /home/user/ /mnt/backup/home/

# Dry run first
rsync -avn --delete /home/user/ /mnt/backup/home/

# With excludes
rsync -av --delete --exclude='node_modules' --exclude='.cache' /home/user/ /mnt/backup/home/
```

### Incremental with Hard Links
```bash
# Create incremental backup using --link-dest (space efficient)
DEST="/mnt/backup/$(date +%Y-%m-%d)"
LATEST="/mnt/backup/latest"
rsync -av --link-dest="$LATEST" /home/user/ "$DEST/"
ln -snf "$DEST" "$LATEST"

# This creates date-stamped directories where unchanged files
# are hard-linked to previous backup (uses no extra space)
```

### Remote Backup
```bash
# Push to remote
rsync -avz /home/user/ user@backup-server:/backup/home/

# Pull from remote
rsync -avz user@remote:/data/ /mnt/backup/data/

# With bandwidth limit
rsync -avz --bwlimit=5000 /data/ user@host:/backup/   # 5MB/s limit

# Over specific SSH port
rsync -avz -e "ssh -p 2222" /data/ user@host:/backup/
```

---

## rclone (Cloud Sync)

### Configuration
```bash
# Interactive configuration
rclone config

# List configured remotes
rclone listremotes

# Test connection
rclone lsd remote:

# List contents
rclone ls remote:bucket/path
rclone lsl remote:bucket/path     # With size and date
```

### Sync and Copy
```bash
# Sync (make destination match source, deletes extras)
rclone sync /home/user/docs remote:backup/docs

# Copy (add new/changed files, don't delete)
rclone copy /home/user/docs remote:backup/docs

# Dry run
rclone sync /home/user/docs remote:backup/docs --dry-run

# With progress
rclone sync /home/user/docs remote:backup/docs --progress

# With bandwidth limit
rclone sync /home/user/docs remote:backup/docs --bwlimit 10M

# Exclude patterns
rclone sync /home/user remote:backup --exclude "node_modules/**" --exclude ".cache/**"
```

### Cloud Providers
```bash
# Google Drive
rclone sync /data gdrive:backup/data

# Backblaze B2
rclone sync /data b2:mybucket/data

# AWS S3
rclone sync /data s3:mybucket/data

# SFTP
rclone sync /data sftp:backup/data

# Encrypt remote (wrap any remote with encryption)
# Configure with: rclone config → name: encrypted → type: crypt → remote: remote:path
rclone sync /sensitive-data encrypted:backup
```

---

## duplicity (GPG-Encrypted Backups)

```bash
# Full backup to local path
duplicity full /home/user file:///mnt/backup/duplicity

# Incremental backup (automatic if previous full exists)
duplicity /home/user file:///mnt/backup/duplicity

# Backup to S3
duplicity /home/user s3://s3.amazonaws.com/mybucket/backup

# Backup to SFTP
duplicity /home/user sftp://user@host//backup/duplicity

# With GPG encryption (specific key)
duplicity --encrypt-key ABCD1234 /home/user file:///mnt/backup/duplicity

# Symmetric encryption (passphrase only)
PASSPHRASE="secret" duplicity /home/user file:///mnt/backup/duplicity

# List files in backup
duplicity list-current-files file:///mnt/backup/duplicity

# Restore full backup
duplicity restore file:///mnt/backup/duplicity /home/user/restored

# Restore specific file
duplicity restore --file-to-restore path/to/file file:///mnt/backup/duplicity /home/user/restored-file

# Verify backup
duplicity verify file:///mnt/backup/duplicity /home/user

# Remove old backups (keep last 30 days)
duplicity remove-older-than 30D file:///mnt/backup/duplicity --force

# Remove all but last N full backups
duplicity remove-all-but-n-full 3 file:///mnt/backup/duplicity --force
```

---

## Database Backup

### PostgreSQL
```bash
# Custom format (compressed, supports parallel restore)
pg_dump -Fc mydb > mydb.dump
pg_dump -Fc -h localhost -U postgres mydb > mydb.dump

# SQL format (human-readable)
pg_dump mydb > mydb.sql

# Compressed SQL
pg_dump mydb | gzip > mydb.sql.gz

# All databases
pg_dumpall -U postgres > all_databases.sql

# Restore custom format
pg_restore -d mydb mydb.dump
pg_restore -C -d postgres mydb.dump      # Create database and restore

# Pipe to backup tool
pg_dump -Fc mydb | restic backup --stdin --stdin-filename mydb.dump --repo /backup
```

### MySQL
```bash
# Single database
mysqldump --single-transaction mydb > mydb.sql
mysqldump --single-transaction mydb | gzip > mydb.sql.gz

# All databases
mysqldump --all-databases --single-transaction > all_dbs.sql

# With routines and triggers
mysqldump --single-transaction --routines --triggers mydb > mydb.sql

# Restore
mysql mydb < mydb.sql
gunzip < mydb.sql.gz | mysql mydb
```

### SQLite
```bash
# Online backup (safe while database is in use)
sqlite3 mydb.db ".backup mydb.bak"

# SQL dump
sqlite3 mydb.db ".dump" > mydb.sql
sqlite3 mydb.db ".dump" | gzip > mydb.sql.gz

# Copy file (only safe if no writers)
cp mydb.db mydb-$(date +%Y%m%d).db
```

### Redis
```bash
# Trigger background save
redis-cli BGSAVE

# Wait for save to complete
redis-cli LASTSAVE

# Copy RDB file
cp /var/lib/redis/dump.rdb /backup/redis-$(date +%Y%m%d).rdb

# Export as append-only file
redis-cli BGREWRITEAOF
```

---

## tar-Based Backups

```bash
# Compressed tar backup with date stamp
tar czf backup-$(date +%Y%m%d).tar.gz /data

# Exclude patterns
tar czf backup.tar.gz /data --exclude='node_modules' --exclude='.git' --exclude='*.log'

# Bzip2 compression (smaller but slower)
tar cjf backup.tar.bz2 /data

# XZ compression (smallest but slowest)
tar cJf backup.tar.xz /data

# Incremental backup with snapshot file
tar czf backup-full.tar.gz --listed-incremental=/var/backup/snapshot.snar /data
tar czf backup-incr1.tar.gz --listed-incremental=/var/backup/snapshot.snar /data

# Encrypted tar backup
tar czf - /data | gpg -c -o backup.tar.gz.gpg
gpg -d backup.tar.gz.gpg | tar xzf -     # Decrypt and extract

# Backup from file list
find /data -name "*.py" -newer /var/backup/last_backup > /tmp/filelist.txt
tar czf backup-python.tar.gz -T /tmp/filelist.txt

# Verify tar archive
tar tzf backup.tar.gz > /dev/null && echo "Archive OK"
```

---

## Snapshot-Based Backups

### LVM Snapshots
```bash
# Create snapshot
lvcreate --snapshot --size 1G --name data-snap /dev/vg0/data

# Mount and backup snapshot
mkdir -p /mnt/snap
mount /dev/vg0/data-snap /mnt/snap -o ro
rsync -av /mnt/snap/ /backup/data/

# Cleanup
umount /mnt/snap
lvremove -f /dev/vg0/data-snap
```

### Btrfs Snapshots
```bash
# Create read-only snapshot
btrfs subvolume snapshot -r /data /snapshots/data-$(date +%Y%m%d)

# List snapshots
btrfs subvolume list /snapshots

# Send snapshot to remote (incremental)
btrfs send /snapshots/data-20240116 -p /snapshots/data-20240115 | ssh remote btrfs receive /backup/
```

### ZFS Snapshots
```bash
# Create snapshot
zfs snapshot pool/data@$(date +%Y%m%d)

# List snapshots
zfs list -t snapshot

# Send snapshot to remote
zfs send pool/data@20240116 | ssh remote zfs receive backup/data

# Incremental send
zfs send -i pool/data@20240115 pool/data@20240116 | ssh remote zfs receive backup/data

# Rollback to snapshot
zfs rollback pool/data@20240115
```

---

## Backup Monitoring

```bash
# Check restic last backup time
restic snapshots --json --repo /backup | jq -r '.[-1].time'

# Alert if no backup in last 24 hours
LAST=$(restic snapshots --json --repo /backup | jq -r '.[-1].time')
HOURS=$(( ($(date +%s) - $(date -d "$LAST" +%s)) / 3600 ))
[ "$HOURS" -gt 24 ] && echo "WARNING: Last backup was $HOURS hours ago"

# Borg last archive info
borg info --last 1 /backup/borg-repo

# Backup size trend
restic snapshots --json --repo /backup | jq -r '.[] | "\(.time)\t\(.short_id)"' | while read ts id; do
  echo "$ts $(restic stats "$id" --json --repo /backup | jq '.total_size')"
done
```

---

## Pre/Post Backup Hooks

```bash
# Stop service, backup, restart
systemctl stop myapp
restic backup /var/lib/myapp --repo /backup
systemctl start myapp

# Database quiesce pattern
pg_dump -Fc mydb > /tmp/mydb.dump
restic backup /tmp/mydb.dump /var/lib/myapp --repo /backup
rm /tmp/mydb.dump

# Lock file approach for application-aware backup
touch /var/lock/backup.lock
# Application checks for lock file and pauses writes
restic backup /data --repo /backup
rm /var/lock/backup.lock
```

---

## Selective Restore

```bash
# Find a file in backup
restic find "config.yaml" --repo /backup
restic find --long "*.conf" --repo /backup

# Restore single file from specific snapshot
restic restore abc123 --target /tmp/restore --include "/etc/app/config.yaml" --repo /backup

# Borg: extract specific path
borg extract /backup/borg-repo::archive-name path/to/file

# Borg: diff before restore
borg diff /backup/borg-repo::archive1 archive2

# Restore and compare before overwriting
restic restore latest --target /tmp/restore --include "/etc/app/" --repo /backup
diff -r /tmp/restore/etc/app/ /etc/app/
```

---

## Scheduling Backups

### Cron
```bash
# Edit crontab
crontab -e

# Daily backup at 2 AM
0 2 * * * /usr/local/bin/restic backup /home/user --repo /mnt/backup --password-file /root/.restic-pw --quiet

# Weekly prune on Sundays at 3 AM
0 3 * * 0 /usr/local/bin/restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune --repo /mnt/backup --password-file /root/.restic-pw
```

### Systemd Timer
```bash
# /etc/systemd/system/backup.service
# [Unit]
# Description=Restic Backup
# [Service]
# Type=oneshot
# ExecStart=/usr/local/bin/restic backup /home/user --repo /mnt/backup --password-file /root/.restic-pw
# Environment=RESTIC_REPOSITORY=/mnt/backup
# Environment=RESTIC_PASSWORD_FILE=/root/.restic-pw

# /etc/systemd/system/backup.timer
# [Unit]
# Description=Daily Backup Timer
# [Timer]
# OnCalendar=*-*-* 02:00:00
# Persistent=true
# [Install]
# WantedBy=timers.target

# Enable and start
sudo systemctl enable --now backup.timer
sudo systemctl list-timers   # Verify
```

---

## Verification

```bash
# restic: verify backup integrity
restic check --repo /mnt/backup/restic-repo
restic check --read-data --repo /mnt/backup/restic-repo

# borg: verify backup integrity
borg check /mnt/backup/borg-repo
borg check --verify-data /mnt/backup/borg-repo

# rsync: compare source and destination
rsync -avn /source/ /backup/    # Dry run shows differences

# rclone: check for differences
rclone check /local remote:backup
```

---

## 3-2-1 Backup Strategy Workflow

```bash
# 3 copies, 2 different media, 1 offsite

# Copy 1: Local restic repo (primary backup)
restic backup /home/user --repo /mnt/external/restic-repo

# Copy 2: Remote server (different media)
restic backup /home/user --repo sftp:user@backup-server:/restic-repo

# Copy 3: Cloud storage (offsite)
rclone sync /mnt/external/restic-repo b2:mybucket/restic-repo

# Verify all three
restic check --repo /mnt/external/restic-repo
restic check --repo sftp:user@backup-server:/restic-repo
rclone check /mnt/external/restic-repo b2:mybucket/restic-repo
```

---

## Gotchas

```bash
# restic: ALWAYS store your password securely
# If you lose the password, you CANNOT access your backups
# Use --password-file or RESTIC_PASSWORD environment variable

# borg: Export your key! Without it, encrypted repos are unrecoverable
borg key export /mnt/backup/borg-repo /safe/location/borg-key-backup

# rsync: trailing slash matters!
rsync -av /data  /backup/   # Creates /backup/data/
rsync -av /data/ /backup/   # Copies contents into /backup/

# rclone sync: DELETES files in destination not in source
# Use rclone copy if you don't want deletions

# Test restores regularly! A backup you can't restore is useless
```

---

## Combined Workflows

```bash
# Full database + files backup pipeline
pg_dump -Fc mydb | restic backup --stdin --stdin-filename mydb.dump --repo /backup && \
restic backup /var/lib/myapp --repo /backup --tag daily

# Encrypted cloud backup
tar czf - /sensitive | gpg -c --batch --passphrase-file /root/.pw | rclone rcat remote:backup/$(date +%Y%m%d).tar.gz.gpg

# Backup, verify, and prune in one script
restic backup /data --repo /backup && \
restic check --repo /backup && \
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune --repo /backup

# Parallel database + file backup
pg_dump -Fc mydb > /tmp/mydb.dump &
restic backup /data --repo /backup &
wait
restic backup /tmp/mydb.dump --repo /backup && rm /tmp/mydb.dump
```

---

## Installation

```bash
# restic
sudo apt install restic

# borg
sudo apt install borgbackup

# rsync (usually pre-installed)
sudo apt install rsync

# rclone
curl https://rclone.org/install.sh | sudo bash

# duplicity
sudo apt install duplicity
```
