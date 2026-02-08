# Proposed Task-Based Skill Categories

Reference document for the hierarchical CLI toolkit skill system.

---

## 1. task-json

**Description:** `"JSON/YAML/XML processing - parsing, querying, transforming, and validating structured data"`

**When to use:** `"Use when: extracting fields from JSON, filtering JSON arrays, transforming YAML configs, querying XML documents, validating JSON schema, converting between JSON/YAML/XML. Triggers: json, yaml, xml, jq, yq, parse config, api response, structured data."`

**Tools:**
```
jq, yq, xq, jo, jd, gron, dasel, fx, jless, xmlstarlet, jsonlint, yamllint, dyff
```

**Tool count:** 13

---

## 2. task-tabular

**Description:** `"Tabular data processing - CSV/TSV parsing, column extraction, sorting, aggregation, and statistics"`

**When to use:** `"Use when: extracting columns from CSV/TSV, summing or averaging columns, filtering rows by condition, sorting tabular data, joining files by key, computing statistics, deduplicating data. Triggers: csv, tsv, column, sum, average, sort by field, group by, deduplicate, delimited."`

**Tools:**
```
awk, cut, sort, uniq, column, paste, join, comm, tsort,
miller (mlr), datamash, qsv, xsv, q, textql,
numaverage, numbound, numgrep, numinterval, numnormalize,
numprocess, numrandom, numrange, numround, numsum
```

**Tool count:** 25

---

## 3. task-text

**Description:** `"Text search and transformation - pattern matching, filtering lines, find/replace, and text reformatting"`

**When to use:** `"Use when: searching for text patterns, filtering log lines, replacing text in files, extracting matches with regex, counting lines or words, reformatting text, converting encodings. Triggers: grep, search, find text, replace, regex, filter lines, word count, encoding."`

**Tools:**
```
grep, rg (ripgrep), ag, ast-grep, sed, tr, sd,
head, tail, tac, rev, nl, wc, fmt, fold, expand, unexpand,
split, csplit, pr, iconv, recode, strings
```

**Tool count:** 24

---

## 4. task-files

**Description:** `"File operations - finding files, comparing directories, syncing, copying, and managing disk space"`

**When to use:** `"Use when: finding files by name/date/size, comparing two files or directories, syncing folders, checking disk usage, copying with progress, creating symlinks, batch renaming files. Triggers: find files, compare files, diff, rsync, sync folders, disk usage, du, df, copy, rename."`

**Tools:**
```
find, fd (fdfind), locate, updatedb, bfs, which, whereis,
file, stat, readlink, realpath, namei,
ls, tree,
diff, diff3, sdiff, cmp, difft, difftastic, wiggle, patch,
cp, mv, rm, ln, mkdir, rmdir, touch, install, shred, rename,
rsync, scp, rclone, sshfs,
du, df, ncdu, dust, duf,
lsblk, findmnt, mount, umount, blkid, losetup, fallocate, wipefs
```

**Tool count:** 50

---

## 5. task-http

**Description:** `"HTTP requests and downloads - API calls, file downloads, request debugging, and web testing"`

**When to use:** `"Use when: making GET/POST requests, calling REST APIs, downloading files, sending JSON payloads, debugging HTTP headers, testing endpoints, uploading files. Triggers: curl, wget, http request, api call, download, post json, rest api, http headers."`

**Tools:**
```
curl, wget, xh, httpie, aria2c,
hurl, prism, json-server
```

**Tool count:** 8

---

## 6. task-media

**Description:** `"Media processing - video/audio conversion, image manipulation, metadata extraction, and document conversion"`

**When to use:** `"Use when: converting video formats, extracting audio from video, resizing images, reading EXIF metadata, creating thumbnails, converting documents to PDF, generating diagrams. Triggers: ffmpeg, video convert, audio extract, image resize, thumbnail, exif, pdf convert, diagram."`

**Tools:**
```
ffmpeg, ffprobe, yt-dlp,
convert, identify, mogrify, composite (imagemagick),
exiftool, gifsicle, optipng, jpegoptim,
sox, whisper,
pandoc, weasyprint, unoconv,
graphviz (dot), mermaid-cli
```

**Tool count:** 16

---

## 7. task-archives

**Description:** `"Archive and compression - creating/extracting archives, compressing files, and handling various archive formats"`

**When to use:** `"Use when: creating tar archives, extracting zip files, compressing with gzip/zstd, handling 7z or rar files, batch compression, extracting specific files from archives. Triggers: tar, zip, unzip, gzip, compress, extract, archive, 7z, zstd."`

**Tools:**
```
tar, gzip, gunzip, bzip2, bunzip2, xz, unxz, zstd,
zip, unzip, 7z, rar, unrar,
pigz, cpio, ar, zrun
```

**Tool count:** 17

---

## 8. task-process

**Description:** `"Process management - listing processes, job control, killing processes, resource limits, and parallel execution"`

**When to use:** `"Use when: listing running processes, killing processes by name or PID, running commands in background, setting process priority, limiting CPU/memory, running commands in parallel, batch processing with xargs. Triggers: ps, kill, process list, background job, parallel, xargs, nice, priority, timeout."`

**Tools:**
```
ps, top, htop, btop, ctop,
kill, pkill, killall, skill, snice,
pgrep, pidof, pidwait,
jobs, bg, fg, nohup, disown,
timeout, watch, xargs, parallel,
nice, renice, ionice, chrt, taskset, prlimit,
lsof, fuser, pmap, pwdx, pstree
```

**Tool count:** 34

---

## 9. task-system

**Description:** `"System information and administration - hardware info, services, logs, scheduling, and date/time operations"`

**When to use:** `"Use when: checking system specs, viewing memory/CPU usage, reading system logs, managing systemd services, scheduling cron jobs, working with dates and timestamps, checking hardware sensors. Triggers: system info, memory usage, cpu info, journalctl, systemctl, cron, logs, date arithmetic, uptime."`

**Tools:**
```
uname, hostname, uptime, whoami, id, groups, users, who, w,
free, vmstat, iostat, mpstat, sar, tload,
lscpu, lsmem, lsns, lshw, lspci, lsusb, sensors, nvidia-smi,
dmesg, journalctl, logger, ccze, logrotate, angle-grinder,
systemctl, service, sysctl,
cron, crontab, at,
date, cal, timedatectl, hwclock,
dateadd, datediff, dateround, dateseq, datesort, datetest, datezone, strptime
```

**Tool count:** 48

---

## 10. task-network

**Description:** `"Network diagnostics - connectivity testing, DNS queries, port scanning, traffic analysis, and firewall rules"`

**When to use:** `"Use when: testing network connectivity, resolving DNS names, checking open ports, analyzing network traffic, configuring IP addresses, debugging connection issues, setting firewall rules. Triggers: ping, traceroute, dns, dig, nmap, ports, netstat, ss, ip address, tcpdump, firewall."`

**Tools:**
```
ping, traceroute, tracepath, mtr,
dig, nslookup, host, whois,
ss, netstat, ip, ifconfig, route, arp, arpd,
nmap, nc (netcat), ngrep, tcpdump, tshark,
tc, bridge, nstat, devlink, lnstat, rtmon, ctstat, routel, genl, rdma,
iptables, nft,
ssh, autossh, mosh, sshuttle, bore
```

**Tool count:** 40

---

## 11. task-permissions

**Description:** `"Users and permissions - file permissions, ownership, user management, and access control"`

**When to use:** `"Use when: changing file permissions, setting ownership, creating users, managing groups, configuring sudo access, setting up ACLs, checking user identity. Triggers: chmod, chown, permissions, user, group, sudo, acl, ownership, 755, rwx."`

**Tools:**
```
chmod, chown, chgrp, chroot,
id, whoami, groups,
su, sudo, passwd,
useradd, usermod, userdel, groupadd, groupmod,
getfacl, setfacl, umask
```

**Tool count:** 18

---

## 12. task-git

**Description:** `"Version control - Git operations, GitHub/GitLab CLI, diffs, history, and repository management"`

**When to use:** `"Use when: committing changes, branching, merging, viewing git history, creating pull requests, managing GitHub issues, comparing commits, cleaning up branches. Triggers: git, commit, branch, merge, pull request, gh, github, gitlab, diff, history, rebase."`

**Tools:**
```
git, gh, glab,
delta, difftastic, tig, lazygit, gitui,
git-lfs, git-crypt, pre-commit, gitleaks,
git-extras (60+ subcommands including):
  git-summary, git-authors, git-changelog, git-effort,
  git-delete-merged-branches, git-squash, git-undo
```

**Tool count:** 13 core + 60 git-extras

---

## 13. task-dev

**Description:** `"Development tools - build systems, compilers, linters, formatters, and language toolchains"`

**When to use:** `"Use when: building projects with make/cmake, compiling code, linting shell scripts, formatting code, managing Python/Node environments, running linters, checking types. Triggers: make, build, compile, lint, format, shellcheck, eslint, ruff, prettier, gcc, npm, pip."`

**Tools:**
```
make, cmake, ninja, meson, just, invoke,
gcc, g++, clang, ldd,
shellcheck, shfmt,
eslint, prettier, ruff, black, pyright, mypy, bandit, semgrep,
hadolint, actionlint,
python, pip, pipx, uv, poetry, pdm, pyenv, virtualenv,
node, npm, pnpm, yarn, npx, tsx, tsc, bun, deno, nvm, volta,
esbuild, vite,
cargo, go
```

**Tool count:** 48

---

## 14. task-crypto

**Description:** `"Cryptography and encoding - hashing, checksums, encryption, key management, and base64 encoding"`

**When to use:** `"Use when: computing file checksums, encoding/decoding base64, encrypting files, generating SSH keys, managing GPG keys, creating certificates, hashing passwords. Triggers: hash, md5, sha256, checksum, base64, encrypt, gpg, ssh key, certificate, openssl."`

**Tools:**
```
md5sum, sha1sum, sha256sum, sha512sum, b2sum, cksum, rhash,
base64, base32, basenc, xxd, od, hexdump, uuencode,
openssl, gpg, gnupg, age, sops, pass, vault,
ssh-keygen, ssh-agent, mkcert, certbot
```

**Tool count:** 25

---

## 15. task-database

**Description:** `"Database operations - SQL queries, SQLite management, Redis commands, and data import/export"`

**When to use:** `"Use when: running SQL queries, managing SQLite databases, importing CSV to database, querying with DuckDB, using Redis for caching, exploring data with datasette. Triggers: sqlite, sql query, database, psql, redis, duckdb, import csv, export data."`

**Tools:**
```
sqlite3, sqlite-utils,
psql, mysql, mariadb,
redis-cli,
duckdb, datasette, meilisearch, dolt
```

**Tool count:** 10

---

## 16. task-containers

**Description:** `"Containers and environments - Docker operations, terminal multiplexing, namespace isolation, and environment management"`

**When to use:** `"Use when: running Docker containers, using docker-compose, managing tmux sessions, isolating processes with namespaces, setting up development environments, inspecting container images. Triggers: docker, container, docker-compose, tmux, screen, namespace, devcontainer, environment."`

**Tools:**
```
docker, docker-compose, dive, ctop,
direnv, mise,
screen, tmux, zellij,
unshare, nsenter, chroot, setsid,
lima, devcontainer
```

**Tool count:** 15

---

## 17. task-debug

**Description:** `"Debugging and profiling - system call tracing, performance profiling, memory analysis, and benchmarking"`

**When to use:** `"Use when: tracing system calls, profiling CPU usage, finding memory leaks, benchmarking commands, debugging process hangs, analyzing performance bottlenecks. Triggers: strace, trace, profile, perf, valgrind, memory leak, benchmark, gdb, debug, flamegraph."`

**Tools:**
```
strace, ltrace, dtrace, bpftrace,
perf, sysprof, flamegraph,
py-spy, scalene, memray, austin,
clinic, 0x,
valgrind, massif, heaptrack,
gdb, lldb,
time, hyperfine
```

**Tool count:** 20

---

## 18. task-watch

**Description:** `"File watching and triggers - monitoring file changes and running commands on change"`

**When to use:** `"Use when: watching files for changes, auto-rebuilding on save, triggering commands when files change, monitoring directories for new files. Triggers: watch files, on change, auto rebuild, inotify, file monitor, entr."`

**Tools:**
```
inotifywait, fswatch, watchman, entr
```

**Tool count:** 4

---

## 19. task-api-test

**Description:** `"API and load testing - HTTP benchmarking, load testing, API validation, and performance testing"`

**When to use:** `"Use when: load testing APIs, benchmarking HTTP endpoints, stress testing servers, validating API responses, testing OpenAPI specs, measuring request latency. Triggers: load test, benchmark, stress test, api test, k6, wrk, vegeta, requests per second."`

**Tools:**
```
k6, artillery, vegeta, hey, ab, wrk, wrk2,
hurl, schemathesis
```

**Tool count:** 9

---

## Summary Table

| # | Category | Tool Count | Priority | Status |
|---|----------|-----------|----------|--------|
| 1 | task-json | 13 | High | Implemented |
| 2 | task-tabular | 25 | High | Implemented |
| 3 | task-text | 24 | High | Implemented (as task-text-search) |
| 4 | task-files | 50 | High | Implemented |
| 5 | task-http | 8 | High | Implemented |
| 6 | task-media | 16 | Medium | Planned |
| 7 | task-archives | 17 | Medium | Planned |
| 8 | task-process | 34 | Medium | Planned |
| 9 | task-system | 48 | Medium | Planned |
| 10 | task-network | 40 | Medium | Planned |
| 11 | task-permissions | 18 | Low | Planned |
| 12 | task-git | 73 | Medium | Planned |
| 13 | task-dev | 48 | Low | Planned |
| 14 | task-crypto | 25 | Low | Planned |
| 15 | task-database | 10 | Medium | Planned |
| 16 | task-containers | 15 | Medium | Planned |
| 17 | task-debug | 20 | Medium | Planned |
| 18 | task-watch | 4 | Low | Planned |
| 19 | task-api-test | 9 | Low | Planned |

**Total unique tools covered:** ~245 (some overlap between categories)

---

## Excluded from Skills

These tools are intentionally not included in task skills:

### Cloud CLIs (require account setup, have extensive documentation)
```
aws, az, gcloud, wrangler
```

### AI/LLM Tools (rapidly evolving, model-specific)
```
ollama, llm, openai, anthropic, huggingface-hub, mlflow, dvc, vllm, etc.
```

### Package Managers (distro-specific, well-documented)
```
apt, dpkg, dnf, yum, pacman, brew, nix
```

### Testing Frameworks (language-specific)
```
pytest, jest, vitest, hypothesis, playwright
```

### Trivial/Self-Explanatory
```
echo, printf, yes, seq, true, false, sleep, env, pwd, cat, less, tee, cd
```

### Interactive/TUI Only
```
vim, ed, fzf, pet, glow, cheat
```

### Platform-Specific Clipboard
```
xclip, pbcopy, pbpaste, wl-copy, wl-paste
```
