---
description: "File operations - disk is full, which files are taking up space, need to cleanup old files, organize messy directory, finding files by name/date/size, comparing directories, and syncing with rsync (fd, find, rsync, ncdu, tree)"
when_to_use: "Use when: disk is full and need to free space, want to know which files or directories are taking up space, need to cleanup old or temporary files, organizing a messy directory, finding files by name or date, comparing directories, syncing files between locations, checking disk usage. Triggers: disk full, out of space, files taking up space, cleanup old files, organize directory, which folder is biggest, find files, file search, compare files, diff, rsync, disk usage, directory tree, large files, fd, fdfind, locate, sync files, compare directories, file age, file type, recursive copy, bulk rename, directory size, ncdu, du, duf, tree."
when-to-use: "Use when: disk is full and need to free space, want to know which files or directories are taking up space, need to cleanup old or temporary files, organizing a messy directory, finding files by name or date, comparing directories, syncing files between locations, checking disk usage. Triggers: disk full, out of space, files taking up space, cleanup old files, organize directory, which folder is biggest, find files, file search, compare files, diff, rsync, disk usage, directory tree, large files, fd, fdfind, locate, sync files, compare directories, file age, file type, recursive copy, bulk rename, directory size, ncdu, du, duf, tree."
argument-hint: "[describe what you want to do with files]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# File Operations Task Router

You have access to the **full conversation context**. Your job is to build a well-formed argument for the file operations recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Analyze the conversation to identify:

### 1. Goal
What file operation is needed?
- Find files, compare directories, sync/copy, check disk usage, list structure

### 2. Path(s)
What location(s)?
- Source directory/file
- Destination (for copy/sync)
- Search root directory

### 3. Find Criteria (if searching)
- **By name**: pattern, extension, prefix
- **By time**: modified within N days, older than N days
- **By size**: larger than, smaller than
- **By type**: files only, directories only, symlinks

### 4. Action
What to do with matches?
- List them
- Delete them
- Execute command on each
- Copy/move them

### 5. Output Format
- Simple list
- Detailed (with sizes, dates)
- Tree structure
- Summary totals

---

## Goal-to-Tool Mapping

| Goal | Primary Tool | Notes |
|------|-------------|-------|
| Find files by name | fd (fdfind) | Fast, simple syntax |
| Complex find criteria | find | Most flexible (exec, permissions, etc.) |
| Compare files | diff | Unified, side-by-side, or context diff |
| Structural diff | difftastic | Syntax-aware diff |
| Sync directories | rsync | Efficient incremental copy |
| Check disk usage | du / ncdu | du for scripting, ncdu interactive |
| Free disk space | df / duf | df standard, duf prettier |
| Directory tree | tree | Visual directory listing |
| Identify file type | file | Magic byte identification |

---

## Argument Format

Construct a structured argument string:

```
<goal> in <path> - criteria: <what>, action: <do what>, output: <format>
```

**Include only relevant components.**

### Examples

**Find by name:**
```
find Python files in /home/user/projects - extension: .py, recurse into subdirectories
```

**Find by time:**
```
find files modified in last 24 hours in ./src - type: files only, output: with modification times
```

**Find large files:**
```
find files larger than 100MB in /home - output: sorted by size descending
```

**Compare directories:**
```
compare directories /backup/old and /backup/new - show differences only
```

**Sync:**
```
sync from /home/user/docs to /mnt/backup/docs - preserve timestamps, delete extras in destination
```

**Disk usage:**
```
show disk usage in /var - depth: 1 level, sorted by size
```

**Clean up:**
```
find and delete files older than 30 days in /tmp - extension: .log
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-files-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Delete operation needs confirmation
- Destination path for sync is unclear
- Search criteria are ambiguous

**Use Read tool first if:**
- Need to verify path exists
- Need to check current directory structure
