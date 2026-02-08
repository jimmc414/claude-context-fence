---
description: "Backup and restore - lost my files, accidentally deleted important data, need to restore previous version, protect against data loss, incremental backups, snapshots, cloud sync, and disaster recovery (restic, borg, rsync, rclone, pg_dump)"
when_to_use: "Use when: lost files and need to recover, accidentally deleted important data, need to restore a previous version, want to protect against data loss, setting up incremental or automated backups, syncing to cloud storage, creating database dumps, managing backup retention policies, verifying backup integrity. Triggers: lost files, accidentally deleted, restore previous version, protect data, data loss prevention, backup, restore, restic, borg, rsync backup, incremental, snapshot, rclone, disaster recovery, cloud sync, backup schedule, encrypted backup, duplicity, pg_dump, mysqldump, database backup, tar backup, lvm, zfs, btrfs, selective restore, schedule backup, cron backup, automated backup, cloud backup, offsite backup, backup verify, backup rotate, retention policy, point in time recovery."
when-to-use: "Use when: lost files and need to recover, accidentally deleted important data, need to restore a previous version, want to protect against data loss, setting up incremental or automated backups, syncing to cloud storage, creating database dumps, managing backup retention policies, verifying backup integrity. Triggers: lost files, accidentally deleted, restore previous version, protect data, data loss prevention, backup, restore, restic, borg, rsync backup, incremental, snapshot, rclone, disaster recovery, cloud sync, backup schedule, encrypted backup, duplicity, pg_dump, mysqldump, database backup, tar backup, lvm, zfs, btrfs, selective restore, schedule backup, cron backup, automated backup, cloud backup, offsite backup, backup verify, backup rotate, retention policy, point in time recovery."
argument-hint: "[describe what you want to backup or restore]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Backup and Restore Task Router

You have access to the **full conversation context**. Your job is to build a well-formed argument for the backup recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Analyze the conversation to identify:

### 1. Goal
What backup operation is needed?
- Create backup, restore from backup, list snapshots, prune old backups
- Set up backup schedule, verify backup integrity, sync to cloud
- Configure encryption, manage backup repositories

### 2. What to Backup
Source data?
- Directories, files, databases
- Specific paths mentioned in conversation
- Exclude patterns (caches, temp files, node_modules)

### 3. Destination
Where should backups go?
- Local path, external drive
- Remote server (SSH/SFTP)
- Cloud storage (S3, B2, Google Drive, etc.)
- Backup repository path

### 4. Strategy
What backup approach?
- Full vs incremental vs differential
- Encryption requirements
- Retention policy (keep N daily, weekly, monthly)
- Deduplication needs

### 5. Schedule
Automation needs?
- One-time backup
- Cron schedule
- Systemd timer

---

## Goal-to-Tool Mapping

| Goal | Primary Tool | Notes |
|------|-------------|-------|
| Incremental + encrypted | restic | Dedup, fast, cloud-native |
| Incremental + compressed | borg | Best compression ratio |
| Simple directory sync | rsync | Efficient incremental copy |
| Cloud storage sync | rclone | 40+ cloud providers |
| One-time archive | tar | Simple full backup to file |
| PostgreSQL backup | pg_dump | Logical database dump |
| MySQL backup | mysqldump | Logical database dump |
| GPG-encrypted backup | duplicity | Encrypted incremental to remote |
| Filesystem snapshots | LVM/ZFS/Btrfs | Block-level instant snapshots |

---

## Argument Format

Construct a structured argument string:

```
<goal> <source> to <destination> - strategy: <approach>, encryption: <yes/no>, retention: <policy>
```

**Include only relevant components.**

### Examples

**Initialize backup repo:**
```
initialize restic repo at /mnt/backup - encrypted
```

**Create backup:**
```
backup /home/user/projects to restic repo at /mnt/backup - exclude: node_modules, .git
```

**Restore from backup:**
```
restore latest snapshot from borg repo at /mnt/backup to /home/user/restored
```

**Cloud sync:**
```
sync /home/user/documents to google-drive:backup/ using rclone
```

**Setup schedule:**
```
create daily backup schedule for /home/user with restic to s3:mybucket - keep 7 daily, 4 weekly, 12 monthly
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-backup-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Backup source is unclear
- Destination type is ambiguous
- Encryption preference unknown
- Retention policy not specified

**Use Read tool first if:**
- Need to verify source paths exist
- Need to check existing backup configuration
