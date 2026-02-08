# Examples: CLI Toolkit Skills

This directory contains 22 two-tier skill pairs implementing the context fence pattern for CLI tool domains. Each pair consists of a **router** (context-aware, conversation-reading dispatcher) and a **recipes store** (isolated, dense command reference). Together they cover approximately 195 CLI utilities across 22 domains. These are production-tested Claude Code skills extracted from active use, not toy examples.

One skill (task-containers) ships as router-only: it is a fully functional router that invokes tools directly rather than delegating to a recipes file.

---

## Installation

To deploy these skills in your own project:

1. Copy all `task-*.md` files into your project's `.claude/commands/` directory.
2. Optionally copy `toolkit-master.md` as a browsable index skill.
3. Add the routing guidance from the [CLAUDE.md Configuration](#claudemd-configuration) section below to your project's `CLAUDE.md`.
4. Ensure the CLI tools referenced by each skill are installed on the system. Skills degrade gracefully when tools are missing but will suggest installation commands.

```bash
mkdir -p .claude/commands
cp examples/task-*.md .claude/commands/
cp examples/toolkit-master.md .claude/commands/
```

---

## Skill Catalog

### Data Processing

| Domain | Router | Recipes | Router Lines | Recipes Lines | Key Tools | Problem Description |
|--------|--------|---------|:------------:|:-------------:|-----------|-------------------|
| JSON | `task-json.md` | `task-json-recipes.md` | 161 | 774 | jq, gron, dasel, yq, xq, fx, jless | Parse, query, transform JSON/YAML/XML; extract nested fields; filter arrays |
| Tabular | `task-tabular.md` | `task-tabular-recipes.md` | 128 | 806 | awk, cut, sort, miller, datamash | CSV/TSV processing, column operations, aggregation, delimiter conversion |
| Text Search | `task-text-search.md` | `task-text-search-recipes.md` | 134 | 879 | ripgrep, grep, sed, ast-grep | Search patterns in files, filter lines, batch find/replace |

### File Operations

| Domain | Router | Recipes | Router Lines | Recipes Lines | Key Tools | Problem Description |
|--------|--------|---------|:------------:|:-------------:|-----------|-------------------|
| Files | `task-files.md` | `task-files-recipes.md` | 137 | 834 | fd, find, rsync, ncdu, diff, tree | Find files by name/date/size, compare directories, sync, disk usage |
| Archives | `task-archives.md` | `task-archives-recipes.md` | 128 | 92 | tar, gzip, zip, 7z, zstd | Create/extract archives, compress files, handle corrupted archives |
| Watch | `task-watch.md` | `task-watch-recipes.md` | 122 | 160 | entr, watchexec, nodemon, inotifywait | Watch files for changes, auto-rebuild on save, auto-restart on edit |

### Networking

| Domain | Router | Recipes | Router Lines | Recipes Lines | Key Tools | Problem Description |
|--------|--------|---------|:------------:|:-------------:|-----------|-------------------|
| HTTP | `task-http.md` | `task-http-recipes.md` | 142 | 1,191 | curl, wget, httpie | API calls, downloads, HTTP debugging, authentication, SSL errors |
| Network | `task-network.md` | `task-network-recipes.md` | 134 | 1,524 | nmap, dig, tcpdump, mtr, ssh, ss | Connectivity, DNS resolution, port scanning, traffic capture |
| API Test | `task-api-test.md` | `task-api-test-recipes.md` | 124 | 1,091 | ab, wrk, hey, k6, vegeta, hurl | Load testing, HTTP benchmarking, performance analysis |

### System Administration

| Domain | Router | Recipes | Router Lines | Recipes Lines | Key Tools | Problem Description |
|--------|--------|---------|:------------:|:-------------:|-----------|-------------------|
| Process | `task-process.md` | `task-process-recipes.md` | 129 | 811 | ps, kill, lsof, xargs, parallel | Process management, port-in-use, zombie processes, parallel execution |
| System | `task-system.md` | `task-system-recipes.md` | 124 | 1,269 | systemctl, journalctl, free, iostat | System monitoring, services, memory/CPU, cron scheduling |
| Permissions | `task-permissions.md` | `task-permissions-recipes.md` | 123 | 87 | chmod, chown, setfacl, sudo | File permissions, ownership, ACLs, permission denied errors |
| Containers | `task-containers.md` | -- | 134 | -- | docker, podman, docker-compose, tmux | Container lifecycle, image optimization, compose, namespace isolation |

### Development

| Domain | Router | Recipes | Router Lines | Recipes Lines | Key Tools | Problem Description |
|--------|--------|---------|:------------:|:-------------:|-----------|-------------------|
| Git | `task-git.md` | `task-git-recipes.md` | 145 | 850 | git, gh, glab, delta | Version control, branching, merging, pull requests, GitHub CLI |
| Dev | `task-dev.md` | `task-dev-recipes.md` | 133 | 1,581 | gcc, npm, pip, ruff, eslint, cargo | Build systems, linting, formatting, dependency management |
| Debug | `task-debug.md` | `task-debug-recipes.md` | 146 | 1,829 | strace, perf, valgrind, gdb, hyperfine, py-spy | CPU/memory profiling, tracing, benchmarking, segfault analysis |
| Database | `task-database.md` | `task-database-recipes.md` | 130 | 1,292 | sqlite3, duckdb, psql, mysql, redis-cli | SQL queries, schema inspection, data import/export, connection issues |

### Security and Analysis

| Domain | Router | Recipes | Router Lines | Recipes Lines | Key Tools | Problem Description |
|--------|--------|---------|:------------:|:-------------:|-----------|-------------------|
| Crypto | `task-crypto.md` | `task-crypto-recipes.md` | 116 | 724 | openssl, gpg, ssh-keygen, age, sha256sum | Hashing, checksums, encryption, certificates, SSL debugging |
| Reverse Eng. | `task-reverse-engineer.md` | `task-reverse-engineer-recipes.md` | 139 | 827 | readelf, objdump, strings, strace, binwalk, yara | Binary analysis, file inspection, protocol decoding, forensics |

### Media

| Domain | Router | Recipes | Router Lines | Recipes Lines | Key Tools | Problem Description |
|--------|--------|---------|:------------:|:-------------:|-----------|-------------------|
| Media | `task-media.md` | `task-media-recipes.md` | 151 | 1,038 | ffmpeg, imagemagick, yt-dlp, whisper, pandoc | Video/audio conversion, image manipulation, document conversion |

### Logs and Backup

| Domain | Router | Recipes | Router Lines | Recipes Lines | Key Tools | Problem Description |
|--------|--------|---------|:------------:|:-------------:|-----------|-------------------|
| Logs | `task-logs.md` | `task-logs-recipes.md` | 130 | 685 | journalctl, lnav, goaccess | Log analysis, filtering by severity/time, access log parsing |
| Backup | `task-backup.md` | `task-backup-recipes.md` | 131 | 748 | restic, borg, rsync, rclone, pg_dump | Incremental backups, snapshots, cloud sync, database dumps |

---

## Anatomy of a Skill Pair

Using `task-json` / `task-json-recipes` as the canonical example.

### Router (`task-json.md` -- 161 lines)

The router's frontmatter declares it as context-aware and able to invoke other skills:

```yaml
---
description: "JSON processing - can't read nested API response, ..."
context: inherit          # Sees full conversation history
allowed-tools: Skill, Read  # Can invoke recipes skill and read files
---
```

The body has three responsibilities:

1. **Context extraction** -- Structured analysis sections (Goal, File Paths, Structure Hints, Filter Conditions, Output Format, Scope Limits, Error Context) that guide the model to pull relevant details from the conversation.

2. **Argument construction** -- A format template and examples showing how to compress extracted context into a single structured string: `<goal> from <file_path> - structure: <hints>, filter: <conditions>, output: <format>`.

3. **Recipes invocation** -- A final instruction to call `task-json-recipes` via the Skill tool with the constructed argument. Includes guidance on when to ask the user instead.

### Recipes (`task-json-recipes.md` -- 774 lines)

The recipes store's frontmatter isolates it from conversation context:

```yaml
---
description: "JSON recipes reference (internal)"
context: fork             # Cannot see conversation -- clean context
allowed-tools: Read       # Can only read files, not invoke skills
---
```

The body is a dense reference organized as:

1. **Tools list** -- All tools covered: jq, yq, xq, jo, jd, gron, dasel, fx, jless, etc.
2. **Quick reference table** -- One-line command for each common operation.
3. **When to use what** -- Tool selection guidance with trade-offs.
4. **Categorized recipes** -- Detailed command examples grouped by operation type (extraction, filtering, transformation, aggregation, format conversion, streaming, multi-format).

### Critical Differences

| Aspect | Router | Recipes |
|--------|--------|---------|
| `context` | `inherit` (sees conversation) | `fork` (isolated) |
| `allowed-tools` | `Skill, Read` | `Read` only |
| Size | 50--160 lines | 87--1,829 lines |
| Purpose | Extract context, build arguments | Return commands |
| `description` | Long, trigger-rich (SEO for routing) | Short, "(internal)" |

---

## Statistics

**Files:** 44 skill files (22 routers + 21 recipes + 1 master index). One domain (containers) is router-only.

**Routers (22 files):**
- Total: 2,945 lines
- Average: 134 lines
- Range: 116 (task-crypto) to 161 (task-json)

**Recipes (21 files):**
- Total: 18,092 lines
- Average: 862 lines
- Range: 87 (task-permissions-recipes) to 1,829 (task-debug-recipes)

**Combined:** 21,037 lines across 43 skill files (excluding toolkit-master.md at 121 lines).

**Largest pair:** task-debug at 146 + 1,829 = 1,975 lines.
**Smallest complete pair:** task-permissions at 123 + 87 = 210 lines.

**Estimated tokens:** Routers average ~500 tokens each (~11,000 total). Recipes average ~3,500 tokens each (~70,000 total). A single skill invocation loads one router (~500 tokens) plus one recipes file (~3,500 tokens) for approximately 4,000 tokens per use.

---

## Disambiguation Guide

When domains overlap, a request may plausibly match multiple skills. This table resolves ambiguity by mapping common request patterns to the correct skill.

| User Request Pattern | Use This Skill | Not This |
|---------------------|---------------|----------|
| "search for text/pattern in files" | task-text-search | task-files |
| "find files by name/size/date" | task-files | task-text-search |
| "extract data from JSON" | task-json | task-tabular |
| "process CSV/TSV columns" | task-tabular | task-json |
| "Docker container issues" | task-containers | task-process |
| "system process management" | task-process | task-containers |
| "parse structured log fields" | task-logs | task-text-search |
| "search text in log files" | task-text-search | task-logs |
| "encrypt/decrypt files" | task-crypto | task-files |
| "file checksums for verification" | task-crypto | task-files |
| "database backup (pg_dump)" | task-backup | task-database |
| "database queries/schema" | task-database | task-backup |
| "HTTP API testing (load/perf)" | task-api-test | task-http |
| "HTTP requests (curl/wget)" | task-http | task-api-test |
| "security scanning containers" | task-containers | task-dev |
| "code vulnerability scanning" | task-dev | task-containers |
| "create/extract archives" | task-archives | task-files |
| "watch files and rerun" | task-watch | task-files, task-dev |
| "chmod/chown permissions" | task-permissions | task-files, task-system |
| "strace for debugging perf" | task-debug | task-reverse-engineer |
| "strace to understand binary" | task-reverse-engineer | task-debug |
| "check what's on local port" | task-process (lsof) | task-network |
| "scan remote ports" | task-network (nmap) | task-process |
| "view/search system logs" | task-logs | task-system |
| "benchmark CLI command" | task-debug (hyperfine) | task-api-test |
| "benchmark HTTP endpoint" | task-api-test | task-debug |
| "download video/audio" | task-media (yt-dlp) | task-http |

Disambiguation matters because the routing model sees skill descriptions and picks the best match. When two skills cover overlapping tools (e.g., strace appears in both task-debug and task-reverse-engineer), the user's intent determines the correct choice. Adding this table to CLAUDE.md gives the model an explicit tiebreaker.

---

## CLAUDE.md Configuration

When deploying these skills, add the following to your project's `CLAUDE.md` to enable correct routing:

````markdown
## Skill Routing Guidance

When invoking `task-*` skills, **prefer the more descriptive skill** over internal/recipes variants:

1. **Prefer the descriptive skill** - It has `context: inherit` and sees the full conversation
2. **Skills with "(internal)" are helpers** - Let the parent skill invoke them
3. **The descriptive skill builds proper arguments** from conversation context

### Two-Tier Skills Reference

| Descriptive (use this) | Internal (auto-called) |
|------------------------|------------------------|
| `task-json` | `task-json-recipes` |
| `task-tabular` | `task-tabular-recipes` |
| `task-text-search` | `task-text-search-recipes` |
| `task-files` | `task-files-recipes` |
| `task-reverse-engineer` | `task-reverse-engineer-recipes` |
| `task-media` | `task-media-recipes` |
| `task-git` | `task-git-recipes` |
| `task-debug` | `task-debug-recipes` |
| `task-network` | `task-network-recipes` |
| `task-http` | `task-http-recipes` |
| `task-database` | `task-database-recipes` |
| `task-process` | `task-process-recipes` |
| `task-system` | `task-system-recipes` |
| `task-dev` | `task-dev-recipes` |
| `task-api-test` | `task-api-test-recipes` |
| `task-crypto` | `task-crypto-recipes` |
| `task-logs` | `task-logs-recipes` |
| `task-backup` | `task-backup-recipes` |
| `task-archives` | `task-archives-recipes` |
| `task-permissions` | `task-permissions-recipes` |
| `task-containers` | (router-only) |
| `task-watch` | `task-watch-recipes` |

**Why this works:** The router skill extracts context (file paths, constraints, output format) from the conversation and constructs well-formed arguments. The recipes skill receives these prepared arguments and returns the appropriate command.

### How It Works

```
User: "extract emails from that json file"
         |
   task-json (router)
   - Sees full context
   - Finds file path from earlier in conversation
   - Identifies nested structure discussed
   - Builds: "extract emails from /tmp/data.json - path: .users[].email"
         |
   task-json-recipes (isolated)
   - Receives prepared argument
   - Returns: jq -r '.users[].email' /tmp/data.json
```
````

For the full disambiguation table, see the [Disambiguation Guide](#disambiguation-guide) section above and copy the relevant entries into your CLAUDE.md.

---

## Creating Your Own Pair

See [SPEC.md](../SPEC.md) for the full specification including component specs, the argument protocol, routing rules, and anti-patterns.

In brief: write a router (50--160 lines, `context: inherit`, `allowed-tools: Skill, Read`) that extracts conversation context and constructs a structured argument string. Write a recipes store (87--1,800+ lines, `context: fork`, `allowed-tools: Read`) containing dense command references organized by operation type. Add a routing preference entry to your project's CLAUDE.md so the model knows to invoke the router first.
