# Tool Tier Ranking Proposal

**Goal:** Organize ~416 remaining tools into 4 tiers by usefulness for AI agents

---

## Tier Definitions

| Tier | Name | Count | Criteria |
|------|------|-------|----------|
| **1** | Essential | ~45 | Universal, daily use, excellent agent UX |
| **2** | High Value | ~100 | Frequently useful, good UX, core domain tools |
| **3** | Specialized | ~170 | Domain-specific, situational, language-bound |
| **4** | Reference | ~100 | Niche, edge cases, completeness |

---

## Tier 1: Essential (~45 tools)

Tools every agent should have immediate access to.

### Core Operations
| Tool | Why Essential |
|------|---------------|
| **jq** | JSON processing - universal data format |
| **rg (ripgrep)** | Primary search tool |
| **fd** | Find files quickly |
| **git** | Version control - universal |
| **curl** | HTTP requests - API interactions |
| **timeout** | Prevent infinite hangs (critical safety) |
| **xargs** | Batch operations |

### File Operations
| Tool | Why Essential |
|------|---------------|
| **cat/head/tail** | Read files |
| **cp/mv/rm/mkdir** | File management |
| **chmod/chown** | Permissions |
| **ln** | Symlinks |
| **diff** | Compare files |
| **patch** | Apply changes |
| **tar/zip/unzip** | Archives |

### Text Processing
| Tool | Why Essential |
|------|---------------|
| **grep** | Fallback search (ubiquitous) |
| **sed** | Stream editing |
| **awk** | Text processing |
| **sort/uniq** | Sorting/dedup |
| **cut/paste** | Column ops |
| **wc** | Counting |
| **tr** | Character translation |

### System Info
| Tool | Why Essential |
|------|---------------|
| **ps** | Process listing |
| **ls** | Directory listing |
| **pwd** | Current directory |
| **env/printenv** | Environment |
| **which** | Find commands |
| **date** | Date/time |
| **uname** | System info |

### Safety & Control
| Tool | Why Essential |
|------|---------------|
| **sponge** | Safe in-place writes |
| **chronic** | Suppress success output |
| **shellcheck** | Validate bash scripts |
| **flock** | File locking |

### Development
| Tool | Why Essential |
|------|---------------|
| **python** | Universal scripting fallback |
| **node** | JavaScript runtime |
| **gh** | GitHub operations |
| **docker** | Containerization |
| **make** | Build automation |

---

## Tier 2: High Value (~100 tools)

Frequently useful, good UX, core for common domains.

### Enhanced CLI
bat, eza, dust, duf, sd, delta, hyperfine, tree, less, file

### JSON/Data
jc, jo, gron, yq, miller, xsv, qsv, csvkit, dasel

### Databases
sqlite3, sqlite-utils, duckdb, psql, redis-cli, mongosh

### Version Control
git-delta, difftastic, pre-commit, git-extras (subset), glab

### HTTP/Network
httpie, xh, wget, nc, dig, ssh, rsync, mtr, nmap

### Containers
docker-compose, podman, kubectl, helm, minikube

### Cloud
aws, gcloud, az, wrangler, terraform, ansible

### Languages (Core)
pip, npm, cargo, go, pytest, jest

### Linting/Format
eslint, prettier, ruff, black, mypy, hadolint, actionlint

### Build
just, task, cmake, ninja, poetry, uv

### Debugging
strace, lsof, pgrep/pkill, htop-alternatives (procs)

### Safety
ts, parallel, tee, watch, entr

---

## Tier 3: Specialized (~170 tools)

Domain-specific, useful when needed.

### Language Ecosystems
- **Python:** pyenv, pipx, pdm, mypy, pyright, black, ruff, pytest, hypothesis
- **JavaScript:** pnpm, yarn, tsx, vitest, eslint, prettier, nx, turborepo
- **Go:** golangci-lint, gopls, delve, gofmt
- **Rust:** rustup, clippy, rustfmt, cargo-audit, cargo-watch
- **Ruby:** rbenv, bundle, rubocop, rails, rake
- **Java:** maven, gradle, sdkman, kotlin, scala

### DevOps/Infra
- kubectl depth, kustomize, stern, kind, argocd
- opentofu, pulumi, cdktf, crossplane
- serverless, sam
- prometheus, promtool, vector

### Databases (Extended)
flyway, dbmate, alembic, prisma, drizzle-kit, knex, liquibase

### API/Testing
k6, artillery, vegeta, grpcurl, newman, playwright, puppeteer, cypress

### Security
trivy, trufflehog, gitleaks, semgrep, bandit, safety, sops, vault, age

### Media
ffmpeg, ffprobe, imagemagick, pandoc, weasyprint

### Documentation
mkdocs, docusaurus, sphinx, typedoc, jsdoc, redoc

### Misc Specialized
expect, screen, tmux, mitmproxy, wireshark (tshark)

---

## Tier 4: Reference (~100 tools)

Niche, edge cases, kept for completeness.

### Legacy/Compatibility
ifconfig, netstat, route, arp (use iproute2 instead)

### Low-Level System
lsblk, lscpu, lsmem, blkid, dmesg, sysctl, nsenter, unshare
partx, losetup, mount/umount, flock details

### Specialized Networking
tc, bridge, rdma, devlink, socat, tcpdump, tshark, ngrep

### Niche Utilities
- dateutils (dateadd, datediff, etc.)
- num-utils (numaverage, numsum, etc.)
- factor, units, qalc, bc, dc

### Rare but Valid
- comby (structural refactoring)
- ast-grep (AST queries)
- pup, xidel (HTML parsing)
- mlflow, wandb, dvc (ML ops)
- torchserve, vllm, triton (ML serving)

### Language-Specific Niche
- PHP: composer, php
- Scala: scala, sbt
- Many ORM/migration tools
- Specialized profilers

---

## Summary

| Tier | Purpose | Example Use Case |
|------|---------|------------------|
| **1** | Always available | "Search for X in codebase" |
| **2** | Load on demand | "Deploy to Kubernetes" |
| **3** | Domain contexts | "Set up Python ML project" |
| **4** | Rare/reference | "Debug kernel namespace issue" |

---

## Questions for You

1. **Does this tier structure make sense?**
2. **Should any tools move between tiers?**
3. **Want me to be more aggressive (smaller Tier 1)?**
4. **Any tools you'd remove entirely from Tier 4?**

Once approved, I'll update the inventory with tier classifications.
