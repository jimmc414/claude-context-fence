---
description: "File watching and triggers - manually rerunning is tedious, need automatic rebuild on save, auto-restart server on code change, monitoring file changes and running commands (entr, watchexec, nodemon, inotifywait)"
when_to_use: "Use when: manually rerunning a command after every edit is tedious, need automatic rebuild when files change, want server to auto-restart on code change, need to trigger tests on save, monitoring directories for new files, live reload during development. Triggers: manually rerunning, auto rebuild, auto restart, rerun on save, tedious repetition, watch files, on change, inotify, file monitor, entr, watchexec, nodemon, cargo watch, fswatch, chokidar, live reload, hot reload, file watcher, rebuild on save, rerun on change, auto test, watch directory, file trigger."
when-to-use: "Use when: manually rerunning a command after every edit is tedious, need automatic rebuild when files change, want server to auto-restart on code change, need to trigger tests on save, monitoring directories for new files, live reload during development. Triggers: manually rerunning, auto rebuild, auto restart, rerun on save, tedious repetition, watch files, on change, inotify, file monitor, entr, watchexec, nodemon, cargo watch, fswatch, chokidar, live reload, hot reload, file watcher, rebuild on save, rerun on change, auto test, watch directory, file trigger."
argument-hint: "[describe what files to watch and what to run]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# File Watch Task Router

You have access to the **full conversation context**. Your job is to build a well-formed argument for the file watching recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Analyze the conversation to identify:

### 1. Watch Target
What files/directories to monitor?
- Specific files mentioned in conversation
- File patterns (*.py, *.go, *.ts)
- Directories being worked on
- Exclude patterns (node_modules, .git, __pycache__)

### 2. Action on Change
What to do when files change?
- Run command (build, test, lint)
- Restart process (server, daemon)
- Execute script
- Clear screen and rerun

### 3. Language/Framework Context
What's being developed?
- Python, Go, Rust, Node.js, etc.
- Specific build tool or test runner
- Framework-specific needs (hot reload)

### 4. Debounce Needs
How to handle rapid changes?
- Wait for changes to settle
- Immediate response needed
- Batch multiple changes

---

## Goal-to-Tool Mapping

| Goal | Primary Tool | Notes |
|------|-------------|-------|
| Simple rerun on change | entr | Pipe file list, run command |
| Extension-based filtering | watchexec | Glob patterns, cross-platform |
| Node.js process restart | nodemon | Auto-restart Node apps |
| Rust rebuild | cargo watch | Cargo-integrated watcher |
| Low-level file events | inotifywait | Linux inotify, scripting |
| macOS file monitoring | fswatch | Cross-platform, macOS native |
| Periodic re-execution | watch | Repeat command every N seconds |

---

## Argument Format

Construct a structured argument string:

```
watch <target> and <action> - filter: <patterns>, options: <settings>
```

**Include only relevant components.**

### Examples

**Auto-test Python:**
```
watch *.py files in ./src and run pytest - clear screen on each run
```

**Restart server:**
```
watch ./server.py and restart python server.py - on any .py file change
```

**Build on save:**
```
watch all .go files in ./cmd/ and run go build - exclude vendor/
```

**Monitor directory:**
```
monitor /var/data/ for new files - run processing script on each new file
```

**Node.js development:**
```
watch and restart Node.js server at ./app.js - ignore node_modules
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-watch-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Watch target is unclear
- Command to run is not specified
- Multiple actions could apply

**Use Read tool first if:**
- Need to verify file patterns exist
- Need to check what build system is used
