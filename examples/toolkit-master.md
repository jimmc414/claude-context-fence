---
description: "CLI Toolkit Master Index - browse all available task-based command-line skills (human use)"
disable-model-invocation: true
argument-hint: "[optional: specific task or tool query]"
context: fork
model: inherit
allowed-tools: Read, Skill
---

# CLI Toolkit Master Index

A task-oriented skill system for command-line operations. Skills are organized by **what you want to accomplish**, not by individual tools.

Use `/toolkit-master` to browse available skills. Agents auto-route to appropriate skills based on what they're trying to do.

---

**Query**: $ARGUMENTS

---

## Available Task Skills (22)

### Data Processing
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-json` | Parse, query, transform JSON/YAML/XML | jq, yq, xq |
| `/task-tabular` | CSV/TSV processing, column operations, aggregation | awk, cut, sort, miller |
| `/task-text-search` | Search patterns, filter lines, find/replace | grep, rg, sed |

### File Operations
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-files` | Find files, compare, sync, disk usage | find, fd, diff, rsync, du |
| `/task-archives` | Create/extract archives, compress files | tar, gzip, zip, 7z, zstd |
| `/task-watch` | Watch files, auto-run on change | entr, inotifywait, fswatch |

### Networking
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-http` | API calls, downloads, HTTP debugging | curl, wget, httpie |
| `/task-network` | Connectivity, DNS, ports, traffic | ping, dig, nmap, ss, tcpdump |
| `/task-api-test` | Load testing, benchmarking, API validation | k6, wrk, vegeta, hurl |

### System Administration
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-process` | Manage processes, parallel execution | ps, kill, xargs, parallel |
| `/task-system` | System info, services, logs, scheduling | systemctl, journalctl, cron |
| `/task-permissions` | Users, groups, file permissions | chmod, chown, useradd |
| `/task-containers` | Docker, tmux, environments | docker, docker-compose, tmux |

### Development
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-git` | Version control, GitHub, diffs | git, gh, delta |
| `/task-dev` | Build, lint, format, language tools | make, shellcheck, ruff, prettier |
| `/task-debug` | Tracing, profiling, benchmarking | strace, perf, valgrind, hyperfine |
| `/task-database` | SQL queries, SQLite, Redis | sqlite3, psql, duckdb, redis-cli |

### Security & Analysis
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-crypto` | Hashing, encoding, encryption | sha256sum, base64, gpg, openssl |
| `/task-reverse-engineer` | Binary analysis, file inspection, protocol decode | strings, binwalk, readelf, objdump |

### Media
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-media` | Video, audio, images, documents | ffmpeg, imagemagick, pandoc |

### Logs & Backup
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-logs` | Log analysis, journalctl, syslog, access logs | journalctl, lnav, goaccess |
| `/task-backup` | Backups, snapshots, cloud sync, restore | restic, borg, rclone, rsync |

---

## Quick Decision Guide

| I want to... | Use |
|--------------|-----|
| Parse JSON / extract fields | `/task-json` |
| Process CSV / sum columns | `/task-tabular` |
| Search text / filter lines | `/task-text-search` |
| Find files / compare dirs | `/task-files` |
| Make API requests | `/task-http` |
| Convert video / resize image | `/task-media` |
| Compress / extract archives | `/task-archives` |
| Debug network issues | `/task-network` |
| Manage processes | `/task-process` |
| View system logs | `/task-system` |
| Run Docker containers | `/task-containers` |
| Git operations / PRs | `/task-git` |
| Lint / format code | `/task-dev` |
| Profile performance | `/task-debug` |
| Run SQL queries | `/task-database` |
| Load test an API | `/task-api-test` |
| Watch files for changes | `/task-watch` |
| Hash files / encrypt | `/task-crypto` |
| Set file permissions | `/task-permissions` |
| Analyze unknown binary | `/task-reverse-engineer` |
| Analyze application logs | `/task-logs` |
| Backup files / restore | `/task-backup` |

---

## How This System Works

1. **Agents auto-route**: Based on task triggers, agents automatically load the appropriate skill
2. **Human browsing**: Use `/toolkit-master` to see all available skills
3. **Direct access**: Use `/task-{name}` directly if you know what you need
4. **Lazy loading**: Skills use `context: fork` - content only loads when invoked

### Design Principles

- **Task-oriented**: Skills organized by what you're trying to DO
- **~195 tools**: Covered across 22 skills
- **~2-4K tokens each**: Detailed enough to be useful
- **Duplicates OK**: Some tools appear in multiple skills by design
