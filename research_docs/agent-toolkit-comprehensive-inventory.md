# Agent Toolkit: Comprehensive Tool Inventory

**Status:** Curated Reference Document (v9 - added RE category)
**Date:** 2026-02-04
**Purpose:** Catalog of agent-friendly CLI tools optimized for AI agent use
**Total Tools:** ~285 (after removing 255 unsuitable tools, added Category 36: Binary Analysis & RE, added backup/log tools)

---

## Table of Contents

1. [Part 1: Tool Suites & Packages](#part-1-tool-suites--packages)
2. [Part 2: Tools by Category (Expanded)](#part-2-tools-by-category-expanded)
3. [Part 3: Cross-Reference Tables](#part-3-cross-reference-tables)
4. [Part 4: Raw Lists for Later Filtering](#part-4-raw-lists-for-later-filtering)

---

# Part 1: Tool Suites & Packages

## 1.1 moreutils (13 tools)

Critical utilities for agent workflows, especially for safe file operations and output control.

| Tool | Description | Agent Use Case | Agent UX | Priority |
|------|-------------|----------------|----------|----------|
| **sponge** | Soak stdin, write to file safely | In-place file modification without race conditions | Excellent | Critical |
| **chronic** | Run command, show output only on failure | Prevent context flooding with success logs | Excellent | Critical |
| **ts** | Timestamp lines of stdin | Add timestamps to any command output | Excellent | High |
| **ifne** | Run command only if stdin is non-empty | Conditional execution in pipelines | Excellent | Medium |
| **pee** | Tee stdin to multiple commands | Split output to parallel processors | Excellent | Medium |
| **combine** | Combine lines from files using set operations | and/or/not/xor on file contents | Good | Medium |
| **isutf8** | Check if input is valid UTF-8 | Encoding validation before processing | Excellent | Medium |
| **lckdo** | Run command with a lock held | Prevent concurrent execution | Good | High |
| **mispipe** | Return exit code of first command in pipe | Better error handling in pipelines | Good | Medium |
| **parallel** | Run commands in parallel (simpler than GNU) | Batch parallel processing | Good | High |
| **zrun** | Run command with decompressed input | Process compressed files transparently | Excellent | Medium |
| **errno** | Look up errno names and descriptions | Debug system call failures | Excellent | Low |
| **ifdata** | Get network interface info | Network configuration queries | Good | Low |

---

## 1.2 util-linux (40+ tools)

System administration utilities, many with JSON output support.

| Tool | Description | Agent Use Case | Agent UX | JSON Support |
|------|-------------|----------------|----------|--------------|
| **lsblk** | List block devices | Disk detection and inventory | Excellent | `-J` |
| **lscpu** | CPU information | Capability detection | Excellent | `-J` |
| **lsmem** | Memory information | Resource awareness | Excellent | `-J` |
| **lsns** | List namespaces | Container introspection | Excellent | `-J` |
| **findmnt** | Find mounted filesystems | Mount point discovery | Excellent | `-J` |
| **flock** | File locking | Prevent race conditions | Excellent | - |
| **uuidgen** | Generate UUIDs | Create unique identifiers | Excellent | - |
| **column** | Columnate output | Table formatting | Good | `-J` |
| **getopt** | Parse command options | Argument parsing in scripts | Good | - |
| **hexdump** | Hex dump of files | Binary analysis | Good | - |
| **rename** | Batch rename files | Pattern-based renaming | Good | - |
| **rev** | Reverse lines character-wise | Text manipulation | Excellent | - |
| **script** | Record terminal session | Session capture and replay | Good | - |
| **scriptreplay** | Replay terminal session | Debugging, demos | Good | - |
| **setsid** | Run in new session | Process isolation | Good | - |
| **taskset** | Set CPU affinity | Performance tuning | Good | - |
| **timeout** | Limit execution time | **Critical**: Prevent infinite hangs | Excellent | - |
| **cal** | Calendar display | Date reference | Excellent | - |
| **chrt** | Set scheduling policy | Process priority | Moderate | - |
| **dmesg** | Kernel messages | Hardware debugging | Excellent | `--json` |
| **eject** | Eject removable media | Device control | Excellent | - |
| **fallocate** | Preallocate file space | Disk management | Good | - |
| **ionice** | Set I/O scheduling class | Resource management | Good | - |
| **logger** | Write to system log | Logging integration | Good | - |
| **look** | Binary search in sorted file | Fast dictionary lookup | Excellent | - |
| **mcookie** | Generate random cookie | Token generation | Excellent | - |
| **namei** | Follow pathname to terminal | Path resolution debugging | Good | - |
| **nsenter** | Enter namespaces | Container debugging | Moderate | - |
| **partx** | Partition table management | Disk operations | Moderate | - |
| **prlimit** | Get/set process resource limits | Resource control | Good | - |
| **renice** | Alter process priority | Process management | Good | - |
| **setterm** | Set terminal attributes | Display control | Moderate | - |
| **swapon/swapoff** | Enable/disable swap | Memory management | Good | - |
| **unshare** | Run in new namespaces | Process isolation | Moderate | - |
| **wdctl** | Watchdog control | System monitoring | Good | - |
| **wipefs** | Wipe filesystem signatures | Disk preparation | Good | - |
| **blkid** | Block device attributes | Device identification | Good | - |
| **losetup** | Loop device management | Disk image handling | Good | - |
| **mount/umount** | Filesystem mounting | Storage management | Good | - |
| **hwclock** | Hardware clock management | Time synchronization | Good | - |

---

## 1.3 GNU coreutils (100+ tools)

Fundamental Unix utilities - the bedrock of shell scripting.

### File Operations
| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **cat** | Concatenate files | File content display | Excellent |
| **cp** | Copy files | File duplication | Excellent |
| **mv** | Move/rename files | File relocation | Excellent |
| **rm** | Remove files | File deletion | Excellent |
| **ln** | Create links | Symbolic/hard links | Good |
| **mkdir** | Make directories | Directory creation | Excellent |
| **rmdir** | Remove directories | Empty directory removal | Excellent |
| **touch** | Update timestamps | File creation, timestamp update | Excellent |
| **chmod** | Change permissions | Access control | Good |
| **chown** | Change ownership | Ownership management | Good |
| **chgrp** | Change group | Group management | Good |
| **install** | Copy with attributes | Deployment | Good |
| **shred** | Secure delete | Sensitive file removal | Good |

### Text Processing
| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **head** | First N lines | Preview files | Excellent |
| **tail** | Last N lines | Log monitoring | Excellent |
| **cut** | Extract columns | Field extraction | Excellent |
| **paste** | Merge lines | Column combining | Good |
| **sort** | Sort lines | Ordering data | Excellent |
| **uniq** | Filter duplicates | Deduplication (requires sort) | Excellent |
| **wc** | Count lines/words/chars | Size metrics | Excellent |
| **tr** | Translate characters | Character transformation | Good |
| **fold** | Wrap lines | Text formatting | Good |
| **fmt** | Reformat paragraphs | Text wrapping | Good |
| **pr** | Paginate files | Print formatting | Good |
| **nl** | Number lines | Line numbering | Excellent |
| **expand/unexpand** | Tab/space conversion | Whitespace normalization | Excellent |
| **split** | Split files by size | Chunking large files | Good |
| **csplit** | Split by pattern | Context-aware splitting | Good |
| **join** | SQL-like join | Relational operations | Good |
| **comm** | Compare sorted files | Set operations | Good |
| **tac** | Reverse file lines | Bottom-to-top processing | Excellent |

### File Information
| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **ls** | List directory | Directory contents | Excellent |
| **stat** | File status | Detailed file metadata | Excellent |
| **file** | Determine file type | Content type detection | Excellent |
| **du** | Disk usage | Space analysis | Excellent |
| **df** | Filesystem usage | Capacity monitoring | Excellent |
| **readlink** | Read symbolic link | Resolve links | Excellent |
| **realpath** | Canonical path | Path normalization | Excellent |

### Output Generation
| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **echo** | Print text | Simple output | Excellent |
| **printf** | Formatted output | Precise formatting | Good |
| **yes** | Repeat string | Auto-confirm (use cautiously) | Excellent |
| **seq** | Generate sequences | Number iteration | Excellent |

### Hashing & Integrity
| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **md5sum** | MD5 hash | Quick checksums | Excellent |
| **sha1sum** | SHA-1 hash | Legacy verification | Excellent |
| **sha256sum** | SHA-256 hash | Secure verification | Excellent |
| **sha512sum** | SHA-512 hash | High-security verification | Excellent |
| **b2sum** | BLAKE2 hash | Fast secure hash | Excellent |
| **cksum** | CRC checksum | Simple integrity | Excellent |

### Encoding
| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **base64** | Base64 encode/decode | Binary-to-text | Excellent |
| **base32** | Base32 encode/decode | Alternative encoding | Excellent |
| **basenc** | Various encodings | Flexible encoding | Good |

### Path Operations
| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **basename** | Strip directory | Filename extraction | Excellent |
| **dirname** | Strip filename | Directory extraction | Excellent |
| **pwd** | Print working directory | Location awareness | Excellent |

### System Information
| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **date** | Display/set date | Timestamp generation | Excellent |
| **sleep** | Delay execution | Timing control | Excellent |
| **env** | Environment variables | Configuration | Excellent |
| **printenv** | Print environment | Debug configuration | Excellent |
| **uname** | System information | Platform detection | Excellent |
| **hostname** | System hostname | Identity | Excellent |
| **whoami** | Current user | Identity | Excellent |
| **id** | User/group IDs | Permission context | Excellent |
| **groups** | User groups | Group membership | Excellent |
| **users** | Logged-in users | Session info | Excellent |
| **who** | Who is logged in | User activity | Excellent |
| **uptime** | System uptime | Health check | Excellent |

### Utilities
| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **tee** | Duplicate output | Save while piping | Excellent |
| **xargs** | Build command lines | Batch operations | Good |
| **nohup** | Run immune to hangups | Background processes | Excellent |
| **nice** | Set priority | Resource management | Excellent |
| **stdbuf** | Modify buffering | Pipeline control | Good |
| **numfmt** | Number formatting | Human-readable sizes | Excellent |
| **factor** | Prime factorization | Math operations | Excellent |
| **shuf** | Shuffle/random | Randomization | Excellent |
| **test** / **[** | Conditional testing | Shell conditionals | Good |
| **true/false** | Return status | Script control flow | Excellent |

---

## 1.4 procps-ng (19 tools)

Process and system monitoring utilities.

| Tool | Description | Agent Use Case | Agent UX | JSON Support |
|------|-------------|----------------|----------|--------------|
| **ps** | Process status | Process enumeration | Excellent | Use with `jc` |
| **top** | Process viewer | Resource monitoring | Moderate | `-b` batch mode |
| **free** | Memory usage | Resource awareness | Excellent | Use with `jc` |
| **vmstat** | Virtual memory stats | System health | Good | Use with `jc` |
| **uptime** | System uptime | Status check | Excellent | Use with `jc` |
| **w** | Who is logged in | User activity | Good | Use with `jc` |
| **pgrep** | Find processes by pattern | Process discovery | Good | - |
| **pkill** | Kill processes by pattern | Process control | Good | - |
| **pidof** | Find PID by name | Process lookup | Excellent | - |
| **pidwait** | Wait for process | Synchronization | Good | - |
| **pmap** | Process memory map | Memory analysis | Good | - |
| **pwdx** | Process working directory | Context discovery | Excellent | - |
| **slabtop** | Kernel slab cache | Memory debugging | Moderate | - |
| **sysctl** | Kernel parameters | System tuning | Good | - |
| **watch** | Repeat command | Polling/monitoring | Good | - |
| **skill/snice** | Send signal/renice by criteria | Process control | Good | - |
| **tload** | Load average graph | Visual load monitoring | Moderate | - |

---

## 1.5 iproute2 (18 tools)

Modern Linux networking utilities (replace net-tools).

| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **ip** | Network configuration | Interface/route/address management | Good |
| **ss** | Socket statistics | Connection monitoring (replaces netstat) | Good |
| **tc** | Traffic control | QoS and traffic shaping | Moderate |
| **bridge** | Bridge management | Network bridge configuration | Moderate |
| **nstat** | Network statistics | Traffic analysis | Good |
| **devlink** | Device link control | NIC management | Moderate |
| **rdma** | RDMA configuration | High-performance networking | Moderate |
| **lnstat** | Linux networking statistics | Link-level stats | Good |
| **rtmon** | Route monitoring | Route change notifications | Good |
| **ctstat** | Connection tracking stats | Firewall monitoring | Good |
| **routel** | Route listing | Human-readable routes | Good |
| **arpd** | ARP daemon | ARP management | Moderate |
| **genl** | Generic netlink | Kernel communication | Moderate |

### Legacy net-tools (for compatibility)
| Tool | Description | Note | Agent UX |
|------|-------------|------|----------|
| **ifconfig** | Interface config | Use `ip addr` instead | Good |
| **netstat** | Network stats | Use `ss` instead | Good |
| **route** | Routing table | Use `ip route` instead | Good |
| **arp** | ARP cache | Use `ip neigh` instead | Good |

---

## 1.6 GNU findutils/diffutils

File finding and comparison utilities.

| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **find** | Find files | Recursive file search | Moderate |
| **xargs** | Build commands from stdin | Batch operations | Good |
| **locate** | Fast file lookup | Index-based search | Excellent |
| **updatedb** | Update locate database | Index maintenance | Good |
| **diff** | Compare files line by line | Change detection | Excellent |
| **diff3** | Three-way comparison | Merge conflicts | Good |
| **sdiff** | Side-by-side diff | Visual comparison | Good |
| **cmp** | Byte comparison | Binary comparison | Excellent |
| **patch** | Apply diffs | Code patching | Good |

---

## 1.7 dateutils (8 tools)

Date/time arithmetic and manipulation.

| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **dateadd** | Add duration to date | Date arithmetic | Excellent |
| **datediff** | Compute date difference | Duration calculation | Excellent |
| **dateseq** | Generate date sequence | Date iteration | Excellent |
| **dateround** | Round date to unit | Date normalization | Excellent |
| **datesort** | Sort by date | Chronological ordering | Excellent |
| **datetest** | Compare dates | Date conditionals | Excellent |
| **datezone** | Timezone conversion | TZ handling | Excellent |
| **strptime** | Parse date strings | Date parsing | Excellent |

---

## 1.8 num-utils (10 tools)

Numeric processing utilities.

| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **numaverage** | Calculate average | Statistics | Excellent |
| **numbound** | Find min/max | Range detection | Excellent |
| **numgrep** | Filter by numeric range | Numeric filtering | Excellent |
| **numinterval** | Show intervals | Difference analysis | Excellent |
| **numnormalize** | Normalize to 0-1 | Data normalization | Excellent |
| **numprocess** | Math expressions | Numeric transformation | Excellent |
| **numrandom** | Generate random numbers | Test data generation | Excellent |
| **numrange** | Generate number sequences | Iteration | Excellent |
| **numround** | Round numbers | Precision control | Excellent |
| **numsum** | Sum numbers | Aggregation | Excellent |

---

## 1.9 Modern CLI Tools (Rust-based)

High-performance rewrites of classic Unix utilities, written in Rust.

| Tool | Replaces | Key Feature | Agent UX |
|------|----------|-------------|----------|
| **ripgrep (rg)** | grep | 10x faster, gitignore-aware | Excellent |
| **fd** | find | Intuitive syntax, parallel | Excellent |
| **dust** | du | Visual disk usage | Good |
| **duf** | df | Beautiful output | Good |
| **sd** | sed | No escaping needed | Excellent |
| **difftastic** | diff | AST-aware structural diff | Good |
| **hyperfine** | time | Statistical benchmarking | Excellent |
| **tokei** | cloc | Fast code counting | Excellent |
| **xh** | httpie | Fast HTTP client | Good |
| **ast-grep** | grep | AST-aware code search | Excellent |

---

## 1.10 Modern CLI Tools (Go-based)

| Tool | Purpose | Key Feature | Agent UX |
|------|---------|-------------|----------|
| **duf** | df replacement | Beautiful output | Good |
| **fzf** | Fuzzy finder | Interactive selection | Good |
| **miller (mlr)** | Data processing | Multi-format tabular | Good |
| **yq** | YAML processor | jq for YAML | Good |
| **gh** | GitHub CLI | GitHub operations | Excellent |
| **glab** | GitLab CLI | GitLab operations | Excellent |

---

## 1.11 git-extras (60+ tools)

Extended git commands for productivity.

| Tool | Description | Agent Use Case | Agent UX |
|------|-------------|----------------|----------|
| **git-summary** | Repository summary | Quick stats | Good |
| **git-effort** | File change frequency | Hot spot detection | Good |
| **git-authors** | List authors | Contribution analysis | Good |
| **git-changelog** | Generate changelog | Release notes | Good |
| **git-commits-since** | Commits since date | Activity tracking | Good |
| **git-count** | Commit counts | Statistics | Good |
| **git-create-branch** | Create and switch | Branch workflow | Good |
| **git-delete-branch** | Delete with safety | Cleanup | Good |
| **git-delete-merged-branches** | Clean merged | Maintenance | Good |
| **git-delta** | List changed files | Change summary | Good |
| **git-ignore** | Manage .gitignore | Ignore patterns | Good |
| **git-info** | Repository info | Overview | Good |
| **git-fork** | Fork repository | Collaboration | Good |
| **git-fresh-branch** | Empty branch | Clean slate | Good |
| **git-line-summary** | Lines per author | Code ownership | Good |
| **git-lock/unlock** | Prevent changes | Protection | Good |
| **git-merge-into** | Merge current into other | Workflow | Good |
| **git-missing** | Show missing commits | Divergence | Good |
| **git-obliterate** | Remove file from history | Security cleanup | Moderate |
| **git-pr** | Pull request helpers | PR workflow | Good |
| **git-release** | Create releases | Versioning | Good |
| **git-rename-branch** | Rename branch | Reorganization | Good |
| **git-rename-tag** | Rename tag | Versioning | Good |
| **git-repl** | Interactive mode | Exploration | Moderate |
| **git-root** | Show root directory | Navigation | Excellent |
| **git-setup** | Initialize with defaults | New repos | Good |
| **git-show-tree** | Visual tree | History visualization | Good |
| **git-squash** | Squash commits | History cleanup | Good |
| **git-standup** | Recent activity | Daily standup | Good |
| **git-touch** | Touch files | Trigger rebuilds | Excellent |
| **git-undo** | Undo last operation | Recovery | Good |

---

# Part 2: Tools by Category (Expanded)

## Category 1: Text Search & Processing

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **ripgrep (rg)** | Fast regex search | Primary code search | Excellent | Different from grep flags |
| **ast-grep (sg)** | Structural code search | AST-aware refactoring | Moderate | Learning curve |
| **awk** | Text processing language | Complex transformations | Good | Syntax complexity |
| **sed** | Stream editor | In-line edits | Moderate | Hallucination-prone |
| **tr** | Character translation | Simple transforms | Excellent | Single characters |
| **cut** | Column extraction | Field extraction | Excellent | Delimiter-based only |
| **paste** | Merge files/lines | Column combining | Good | Simple merging |
| **sort** | Sort lines | Ordering | Excellent | Memory for large files |
| **uniq** | Deduplicate | Remove duplicates | Excellent | Requires sorted input |
| **wc** | Count lines/words/chars | Size metrics | Excellent | Basic counting |
| **head/tail** | First/last lines | Previews | Excellent | - |
| **sd** | Simple find/replace | No escaping needed | Excellent | Not full sed |

---

## Category 2: JSON/Structured Data

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **jq** | JSON query/transform | Complex JSON ops | Moderate | Syntax hallucinations |
| **jc** | CLI output → JSON | Modernize any tool | Excellent | Depends on tool |
| **jo** | Create JSON from args | Safe JSON construction | Good | Simple structures |
| **jd** | JSON diff | Semantic comparison | Good | JSON only |
| **miller (mlr)** | Tabular processing | CSV/JSON/records | Good | Record-oriented |
| **yq** | YAML processor | jq for YAML | Moderate | YAML complexity |
| **qsv** | Ultra-fast CSV | Large CSV files | Good | CSV only |
| **q** | SQL on CSV | Query CSV with SQL | Good | SQL syntax |

---

## Category 3: Database & Data

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **sqlite3** | SQLite CLI | Embedded database | Excellent | Single-file |
| **sqlite-utils** | JSON/CSV → SQLite | File to SQL queries | Excellent | SQLite-specific |
| **DuckDB** | Analytical database | Query CSV/Parquet | Excellent | Analytics-focused |
| **psql** | PostgreSQL CLI | Full Postgres access | Excellent | Requires server |
| **mysql/mariadb** | MySQL CLI | MySQL access | Good | Requires server |
| **datasette** | SQLite → JSON API | Instant read-only API | Good | Read-only |
| **redis-cli** | Redis CLI | Key-value operations | Good | Redis-specific |
| **Dolt** | Git for databases | Version-controlled data | Good | Learning curve |
| **Meilisearch** | Full-text search | Search engine | Excellent | Simple, fast |

---

## Category 4: Version Control

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **git** | Version control | Universal VCS | Excellent | Complexity |
| **gh** | GitHub CLI | GitHub operations | Excellent | GitHub only |
| **glab** | GitLab CLI | GitLab operations | Excellent | GitLab only |
| **difftastic** | Structural diff | AST-aware diffs | Good | Slower |
| **pre-commit** | Git hooks | Quality gates | Good | Config needed |
| **git-extras** | Extended commands | Productivity helpers | Good | Many commands |
| **git-crypt** | Encrypted files | Secrets in git | Good | GPG setup |
| **git-lfs** | Large file storage | Binary files | Good | Server needed |

---

## Category 5: Schema & Validation

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **Pydantic** | Python validation | Data validation | Excellent | Python only |
| **Zod** | TypeScript validation | Schema validation | Excellent | TypeScript only |
| **JSON Schema** | Universal schemas | Cross-language | Good | Verbose |
| **ajv** | JSON Schema validator | Fast validation | Good | Node.js |
| **shellcheck** | Shell linting | **Critical**: bash validation | Excellent | Shell only |
| **shfmt** | Shell formatting | Consistent style | Good | Formatting only |
| **yamllint** | YAML linting | Config validation | Good | YAML only |
| **jsonlint** | JSON linting | JSON validation | Good | Simple validation |
| **file** | Type detection | Magic-based detection | Good | File types only |

---

## Category 6: Task Runners & Build

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **just** | Simple task runner | No tab issues | Excellent | Simple but limited |
| **make** | Build automation | Universal, powerful | Moderate | Arcane syntax |
| **invoke** | Python task runner | Pythonic | Good | Python only |
| **npm/pnpm/yarn** | Node package managers | JavaScript ecosystem | Good | Node.js only |
| **poetry** | Python packaging | Deterministic deps | Good | Python only |
| **uv** | Fast Python packaging | pip replacement | Good | Python only |
| **pdm** | Python packaging | PEP 582 support | Good | Python only |
| **nx** | Monorepo builds | JS/TS monorepos | Moderate | Complex setup |
| **turborepo** | Monorepo builds | Vercel ecosystem | Moderate | JS/TS focused |
| **cmake** | Build generator | C/C++ projects | Moderate | CMakeLists syntax |
| **meson** | Build system | Modern C/C++ | Good | Simpler than cmake |
| **ninja** | Build executor | Fast builds | Good | Low-level |

---

## Category 7: HTTP & Networking

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **curl** | HTTP client | Universal HTTP | Good | Arcane flags |
| **xh** | Fast httpie | Rust implementation | Good | HTTPie-compatible |
| **hurl** | HTTP sequences | File-based API tests | Good | Learning format |
| **wget** | File downloads | Recursive downloads | Moderate | Download-focused |
| **nc (netcat)** | Raw TCP/UDP | Connectivity tests | Good | Low-level |
| **nmap** | Port scanning | Service discovery | Moderate | Security tool |
| **dig** | DNS queries | DNS debugging | Good | DNS only |
| **mitmproxy** | HTTP interception | Traffic debugging | Good | MITM setup |
| **rsync** | File sync | Efficient transfer | Good | SSH-based |
| **ssh** | Secure shell | Remote access | Excellent | Key management |
| **mosh** | Mobile shell | Unreliable networks | Good | Server needed |

---

## Category 8: Containers & Environments

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **docker** | Containerization | Standard containers | Good | Daemon required |
| **docker-compose** | Multi-container | Service orchestration | Good | YAML config |
| **direnv** | Per-directory env | Automatic switching | Good | Shell setup |
| **mise** | Version manager | Multi-language, fast | Good | Rust-based |
| **pyenv** | Python versions | Python management | Good | Python only |
| **nvm** | Node versions | Node management | Good | Node only |
| **virtualenv/venv** | Python isolation | Project isolation | Good | Python only |
| **mamba** | Data science env | Scientific Python | Good | Faster conda |
| **devcontainer** | Dev environments | VS Code containers | Good | VS Code focused |
| **lima** | Linux VMs | macOS Linux dev | Good | macOS focused |

---

## Category 9: Static Analysis & Linting

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **ruff** | Python linting | Fast, comprehensive | Excellent | Python only |
| **eslint** | JS/TS linting | Standard, plugins | Good | Config complexity |
| **pyright** | Python types | Fast, VS Code | Good | Strict |
| **semgrep** | Pattern scanning | Security patterns | Good | Learning rules |
| **bandit** | Python security | Security scanning | Good | Python only |
| **safety** | Python deps | Vulnerability check | Good | Python deps |
| **npm audit** | Node deps | Vulnerability check | Good | Node deps |
| **trivy** | Container scanning | Comprehensive | Good | Container focus |
| **gitleaks** | Secret detection | Leaked credentials | Good | Git focused |
| **hadolint** | Dockerfile linting | Docker best practices | Good | Dockerfile only |
| **actionlint** | GitHub Actions | Workflow linting | Good | Actions only |

---


---

## Category 11: Media Processing

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **ffmpeg** | Video/audio | Powerful transcoding | Poor | Arcane syntax |
| **ffprobe** | Media info | Metadata extraction | Moderate | ffmpeg ecosystem |
| **yt-dlp** | Video downloads | Media fetching | Good | Site-dependent |
| **ImageMagick** | Image processing | Powerful transforms | Moderate | Complex flags |
| **Pillow** | Python images | Python standard | Excellent | Python only |
| **exiftool** | Metadata | EXIF manipulation | Good | Metadata only |

---

## Category 12: Document Generation

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **pandoc** | Universal converter | Any format to any | Good | Complex for edge cases |
| **mermaid-cli** | Diagram rendering | Mermaid → images | Good | Mermaid syntax |
| **graphviz (dot)** | Graph rendering | Directed graphs | Good | DOT syntax |
| **weasyprint** | HTML → PDF | CSS-based PDF | Good | CSS limitations |
| **mkdocs** | Markdown docs | Project docs | Good | Markdown-based |

---

## Category 13: Security & Secrets

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **sops** | Encrypted secrets | Secrets in version control | Good | Key management |
| **age** | File encryption | Modern, minimal | Good | Simple encryption |
| **gpg** | Encryption/signing | Standard crypto | Moderate | Complex |
| **pass** | Password store | GPG-based | Moderate | GPG required |
| **vault** | Secrets management | Enterprise secrets | Moderate | Server needed |
| **mkcert** | Local TLS certs | Dev certificates | Good | Local only |
| **certbot** | Let's Encrypt | Production TLS | Good | Domain needed |
| **openssl** | Crypto toolkit | Swiss army knife | Moderate | Complex flags |
| **ssh-keygen** | SSH keys | Key generation | Good | - |
| **ssh-agent** | Key management | Key caching | Good | Session-based |

---

## Category 14: Testing & Mocking

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **pytest** | Python testing | Test execution | Excellent | Python only |
| **jest** | JavaScript testing | JS/TS tests | Good | Node.js only |
| **json-server** | REST from JSON | Instant mock API | Good | Node.js |
| **prism** | OpenAPI mock | Generate from spec | Good | OpenAPI needed |
| **diff** | File comparison | Change detection | Excellent | Basic |
| **difft** | Structural diff | AST-aware | Good | Slower |
| **jd** | JSON diff | Semantic JSON | Good | JSON only |
| **dyff** | YAML diff | Semantic YAML | Good | YAML only |
| **hypothesis** | Property testing | Edge case generation | Good | Python only |

---

## Category 15: Unix Primitives & Safety

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **timeout** | Limit execution | **Critical**: prevent hangs | Excellent | - |
| **sponge** | Safe in-place write | Prevent race conditions | Excellent | Needs moreutils |
| **chronic** | Output on failure | Prevent context flooding | Excellent | Needs moreutils |
| **watch** | Repeat command | Monitor changes | Good | Interactive |
| **expect** | Script interactive | Automate prompts | Good | Learning curve |
| **xargs** | Build command lines | Batch operations | Good | Argument limits |
| **parallel** | GNU Parallel | Advanced parallel | Good | Complex options |
| **tee** | Split output | Save while piping | Good | - |
| **pv** | Pipe viewer | Progress monitoring | Moderate | Adds overhead |
| **time** | Measure execution | Performance timing | Good | Basic stats |
| **ts** | Timestamp output | Add timestamps | Good | Needs moreutils |
| **ifne** | Conditional run | Run if input exists | Moderate | Needs moreutils |
| **tree** | Directory tree | Visual structure | Good | - |
| **fd** | Fast file find | Better than find | Good | Different syntax |
| **flock** | File locking | Prevent race conditions | Good | - |
| **nohup** | Ignore hangup | Background processes | Good | - |
| **disown** | Remove from shell | Detach processes | Good | - |
| **screen** | Terminal mux | Session persistence | Moderate | Learning curve |
| **tmux** | Terminal mux | Session management | Moderate | Learning curve |
| **zellij** | Modern mux | Plugin system | Moderate | Newer |

---

## Category 16: Debugging & Observability

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **strace** | Syscall tracing | Debug failures | Good | Linux only |
| **ltrace** | Library tracing | API debugging | Good | Less used |
| **lsof** | Open files | Resource debugging | Good | Needs privileges |
| **fuser** | File users | Who has file open | Good | - |
| **pstree** | Process tree | Hierarchy view | Good | - |
| **perf** | Linux profiler | CPU profiling | Moderate | Linux only |
| **bpftrace** | eBPF tracing | Advanced debugging | Moderate | Linux only |
| **py-spy** | Python profiler | Python performance | Good | Python only |
| **valgrind** | Memory checker | Memory debugging | Moderate | Slow |
| **gdb** | Debugger | Binary debugging | Moderate | Complex |
| **lldb** | Debugger | macOS debugging | Moderate | Complex |
| **tcpdump** | Packet capture | Network debugging | Moderate | Needs privileges |
| **tshark** | Wireshark CLI | Protocol analysis | Moderate | Complex filters |
| **ngrep** | Network grep | Pattern in traffic | Moderate | Needs privileges |
| **inotifywait** | File events | Watch for changes | Good | Linux only |
| **fswatch** | Cross-platform watch | File monitoring | Good | - |
| **entr** | Run on change | Auto-rebuild | Good | - |
| **watchman** | Facebook watcher | Large codebase | Good | Setup required |

---

## Category 17: Math & Calculation

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **bc** | Calculator | Arbitrary precision | Excellent | Basic interface |
| **dc** | RPN calculator | Stack-based | Moderate | RPN learning |
| **qalc** | Unit-aware calc | Conversions included | Good | Less common |
| **units** | Unit conversion | Comprehensive database | Good | - |
| **numfmt** | Human numbers | SI/IEC formatting | Good | - |
| **datamash** | Tabular statistics | Quick stats | Good | Column-based |
| **factor** | Prime factorization | Math utility | Good | Limited |
| **shuf** | Random numbers | Randomization | Good | - |
| **uuidgen** | UUID generation | Unique identifiers | Good | - |

---

## Category 18: Cloud CLIs (NEW)

| Tool | Cloud | Description | Agent UX | JSON Support |
|------|-------|-------------|----------|--------------|
| **aws** | AWS | Full AWS access | Good | `--output json` |
| **gcloud** | GCP | Full GCP access | Good | `--format json` |
| **az** | Azure | Full Azure access | Good | `--output json` |
| **wrangler** | Cloudflare | Workers/Pages | Good | `--json` |

---



## Category 21: Message Queues (NEW)

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **redis-cli** | Redis CLI | Pub/sub, streams | Good | - |
| **sqs (aws)** | AWS SQS | SQS operations | Good | AWS CLI |
| **sns (aws)** | AWS SNS | SNS operations | Good | AWS CLI |
| **pubsub (gcloud)** | GCP Pub/Sub | Pub/Sub operations | Good | gcloud CLI |
| **servicebus (az)** | Azure Service Bus | Service Bus ops | Moderate | az CLI |

---


## Category 23: Database Migrations (NEW)

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **dbmate** | Simple migrations | SQL migrations | Good | Simple |
| **sqitch** | Database changes | Dependency-based | Moderate | Different model |
| **atlas** | Schema management | Declarative | Good | Newer |
| **skeema** | MySQL schema | MySQL-focused | Good | MySQL only |

---

## Category 24: Deployment & CD (NEW)

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **github-actions** | GitHub CI/CD | GitHub workflows | Good | GitHub only |
| **semantic-release** | Auto versioning | Semantic versioning | Good | Config needed |

---

## Category 25: AI/LLM Tools (NEW)

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **ollama** | Local LLMs | Run local models | Good | Resources needed |
| **llm** | LLM CLI | Simon Willison's CLI | Good | - |
| **openai** | OpenAI CLI | OpenAI API | Good | API key needed |
| **anthropic** | Anthropic CLI | Claude API | Good | API key needed |
| **huggingface-hub** | HuggingFace CLI | Model management | Good | - |
| **transformers-cli** | Transformers CLI | Model operations | Good | Python |
| **mlflow** | ML lifecycle | Experiment tracking | Moderate | Setup required |
| **dvc** | Data versioning | ML data management | Good | Git-like |
| **bentoml** | Model serving | Deploy models | Good | - |
| **torchserve** | PyTorch serving | PyTorch models | Moderate | PyTorch only |
| **triton** | Inference server | NVIDIA inference | Moderate | GPU required |
| **vllm** | LLM serving | Fast inference | Good | GPU required |
| **text-generation-inference** | HuggingFace serving | LLM inference | Good | GPU required |
| **gpt4all** | Local LLMs | Run on CPU | Good | Slower |
| **llamacpp** | llama.cpp | Efficient inference | Moderate | Build required |
| **whisper** | Speech to text | Audio transcription | Good | Resources needed |
| **coqui-tts** | Text to speech | Speech synthesis | Moderate | - |

---


## Category 27: API Testing (NEW)

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **k6** | Load testing | Performance testing | Good | JavaScript |
| **artillery** | Load testing | YAML-based tests | Good | Node.js |
| **vegeta** | HTTP load testing | Go-based load | Good | Simple |
| **hey** | HTTP load testing | Benchmarking | Good | Simple |
| **ab** | Apache Bench | Basic benchmarking | Moderate | Basic |
| **wrk** | HTTP benchmarking | High performance | Good | - |
| **wrk2** | Latency benchmarking | Coordinated omission | Good | - |
| **hurl** | HTTP testing | File-based tests | Good | - |
| **schemathesis** | API fuzzing | OpenAPI fuzzing | Good | - |

---

## Category 28: Code Generation (NEW)

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **quicktype** | Types from JSON | Type generation | Good | - |
| **openapi-generator** | OpenAPI code gen | SDK generation | Moderate | Config needed |
| **protoc** | Protocol Buffers | gRPC services | Moderate | Build setup |
| **buf** | Modern protobuf | Better protoc | Good | - |
| **graphql-codegen** | GraphQL types | Type generation | Good | - |
| **cookiecutter** | Project templates | Project generation | Good | Python |

---

## Category 29: Documentation (NEW)

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **mkdocs** | Markdown docs | Project docs | Good | - |
| **redoc** | OpenAPI docs | API reference | Good | - |
| **swagger-ui** | OpenAPI docs | Interactive API | Good | - |

---

## Category 30: Browser Automation (NEW)

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **playwright** | Browser automation | E2E testing | Good | - |

---

## Category 31: Package Management (NEW)

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **apt** | Debian packages | System packages | Good | Debian/Ubuntu |
| **dnf/yum** | RPM packages | System packages | Good | RHEL/Fedora |
| **pacman** | Arch packages | System packages | Good | Arch |
| **brew** | Homebrew | macOS/Linux packages | Good | - |
| **nix** | Nix packages | Reproducible packages | Moderate | Learning curve |
| **pip** | Python packages | Python dependencies | Good | Python only |
| **npm** | Node packages | Node dependencies | Good | Node only |
| **pnpm** | Node packages | Fast npm alternative | Good | Node only |

---

## Category 32: Build & Compilation (NEW)

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **gcc** | GNU compiler | C/C++ compilation | Moderate | Complex flags |
| **tsc** | TypeScript compiler | TypeScript build | Good | - |
| **esbuild** | Fast bundler | JS/TS bundling | Good | - |
| **vite** | Build tool | Modern frontend | Good | - |
| **cmake** | Build generator | C/C++ builds | Moderate | CMake syntax |
| **ninja** | Build executor | Fast builds | Good | Low-level |
| **make** | Build system | Universal builds | Moderate | Makefile syntax |

---

## Category 33: Log Management (NEW)

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **angle-grinder** | Log aggregation | SQL-like queries | Good | - |
| **journalctl** | Systemd logs | System logs | Good | `-o json` |
| **logrotate** | Log rotation | Log management | Good | Config needed |
| **ccze** | Log colorizer | Colored logs | Good | - |

---

## Category 34: Performance & Profiling (NEW)

| Tool | Description | Agent Use Case | Agent UX | Limitations |
|------|-------------|----------------|----------|-------------|
| **perf** | Linux profiler | CPU profiling | Moderate | Linux only |
| **flamegraph** | Flame graphs | Visualization | Good | - |
| **py-spy** | Python profiler | Python performance | Good | Python |
| **scalene** | Python profiler | Memory + CPU | Good | Python |
| **memray** | Python memory | Memory profiling | Good | Python |
| **austin** | Python profiler | Sampling profiler | Good | Python |
| **clinic** | Node profiler | Node performance | Good | Node.js |
| **0x** | Node flamegraph | Node visualization | Good | Node.js |
| **valgrind** | Memory checker | Memory debugging | Moderate | Slow |
| **heaptrack** | Heap profiler | Memory usage | Good | C++ |
| **massif** | Heap profiler | Valgrind tool | Moderate | Slow |
| **sysprof** | System profiler | Full system | Moderate | Linux |
| **bpftrace** | eBPF tracing | Advanced tracing | Moderate | Linux |
| **strace** | Syscall tracing | Debug failures | Good | Linux |
| **dtrace** | Dynamic tracing | Comprehensive | Moderate | macOS/BSD |

---

## Category 35: Language-Specific Tools (NEW)

### Python
| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **python** | Python interpreter | Script execution |
| **pip** | Package installer | Dependencies |
| **uv** | Fast pip | Faster installs |
| **poetry** | Project management | Full lifecycle |
| **pipx** | Install apps | CLI tools |
| **pyenv** | Version management | Python versions |
| **virtualenv** | Environments | Isolation |
| **pytest** | Testing | Test execution |
| **ruff** | Linting | Fast linting |
| **black** | Formatting | Code formatting |
| **mypy** | Type checking | Static types |
| **pyright** | Type checking | Fast types |

### JavaScript/TypeScript
| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **node** | Node.js runtime | Script execution |
| **npm** | Package manager | Dependencies |
| **pnpm** | Fast npm | Better npm |
| **yarn** | Package manager | Alternative |
| **bun** | Runtime + package | All-in-one |
| **deno** | Secure runtime | Modern JS |
| **npx** | Package runner | Run packages |
| **tsx** | TypeScript exec | Direct TS run |
| **tsc** | TypeScript compiler | Compilation |
| **eslint** | Linting | Code quality |
| **prettier** | Formatting | Code formatting |
| **jest** | Testing | Test execution |
| **vitest** | Testing | Fast tests |

---

## Category 36: Binary Analysis & Reverse Engineering (NEW)

Tools for analyzing binaries, firmware, protocols, and unknown files. All selected for agent-friendly output (parseable, non-interactive).

### File Triage & Identification
| Tool | Description | Agent Use Case | Agent UX | Package |
|------|-------------|----------------|----------|---------|
| **file** | Identify file type via magic | First step in unknown file analysis | Excellent | file |
| **strings** | Extract printable text | Find hardcoded secrets, URLs, paths | Excellent | binutils |
| **hexdump** | Hex dump of files | Binary inspection | Good | util-linux |
| **xxd** | Hex dump with ASCII | Better hex view, reversible | Excellent | xxd |
| **binwalk** | Firmware analysis | Extract embedded files, entropy analysis | Excellent | binwalk |

### ELF Binary Analysis
| Tool | Description | Agent Use Case | Agent UX | Package |
|------|-------------|----------------|----------|---------|
| **readelf** | ELF headers/sections/symbols | Examine binary structure | Excellent | binutils |
| **objdump** | Disassembly, relocations | Code analysis, section dump | Excellent | binutils |
| **nm** | Symbol table listing | Find functions, variables | Excellent | binutils |
| **ldd** | Shared library dependencies | Identify required libraries | Excellent | libc-bin |
| **size** | Section sizes | Binary layout overview | Excellent | binutils |
| **objcopy** | Extract/convert sections | Pull out specific data | Good | binutils |

### Dynamic Analysis
| Tool | Description | Agent Use Case | Agent UX | Package |
|------|-------------|----------------|----------|---------|
| **strace** | System call tracing | See file/network/process ops | Excellent | strace |
| **ltrace** | Library call tracing | API usage analysis | Excellent | ltrace |
| **gdb** | Debugger (batch mode) | Breakpoints, memory inspection | Good | gdb |

### Network & Protocol Analysis
| Tool | Description | Agent Use Case | Agent UX | Package |
|------|-------------|----------------|----------|---------|
| **tcpdump** | Packet capture | Network traffic recording | Good | tcpdump |
| **tshark** | Protocol decode (JSON) | Structured packet analysis | Excellent | tshark |
| **ngrep** | Network pattern match | Find patterns in traffic | Good | ngrep |
| **socat** | Protocol relay/intercept | MITM, protocol testing | Good | socat |

### Format-Specific Analysis
| Tool | Description | Agent Use Case | Agent UX | Package |
|------|-------------|----------------|----------|---------|
| **pdfinfo** | PDF metadata | Document analysis | Excellent | poppler-utils |
| **pdftotext** | PDF text extraction | Content extraction | Excellent | poppler-utils |
| **zipinfo** | Archive structure | Inspect without extracting | Excellent | zip |
| **exiftool** | Metadata extraction | EXIF, XMP, IPTC data | Excellent | exiftool |
| **mediainfo** | Media file details | Video/audio metadata (JSON) | Excellent | mediainfo |

### Encoding & Crypto Detection
| Tool | Description | Agent Use Case | Agent UX | Package |
|------|-------------|----------------|----------|---------|
| **base64** | Base64 encode/decode | Common encoding | Excellent | coreutils |
| **hashid** | Identify hash types | Determine hash algorithm | Excellent | hashid |
| **openssl** | Crypto operations | Certificate/key inspection | Good | openssl |

### Pattern Matching & Advanced
| Tool | Description | Agent Use Case | Agent UX | Package |
|------|-------------|----------------|----------|---------|
| **yara** | Signature matching | Malware/pattern detection | Excellent | yara |
| **radare2** | RE framework (batch) | Full disassembly, scripted analysis | Good | radare2 |

**Installation:**
```bash
# Core (usually pre-installed)
sudo apt install binutils file xxd

# Analysis
sudo apt install binwalk strace ltrace gdb

# Network
sudo apt install tcpdump tshark ngrep socat

# Format tools
sudo apt install poppler-utils exiftool mediainfo

# RE tools
sudo apt install radare2 yara hashid
```

---

# Part 3: Cross-Reference Tables

## 3.1 Tools by Primary Survival Lever

### Lever 1: Insight Compression (Crystallized Knowledge)
| Tool | Why High Insight | Value |
|------|------------------|-------|
| git | Version control knowledge | Critical |
| Temporal | Durable execution patterns | High |
| Pydantic/Zod | Validation patterns | High |
| shellcheck | Shell best practices | High |
| ruff/eslint | Code quality rules | High |
| semgrep | Security patterns | High |

### Lever 2: Substrate Efficiency (CPU vs GPU)
| Tool | Why Efficient | Savings |
|------|---------------|---------|
| ripgrep | Compiled search | 1000x vs regex in LLM |
| DuckDB | Analytical queries | Massive for data |
| bc | Arbitrary precision | Cheaper than LLM math |
| sqlite | SQL operations | Faster than reasoning |
| jq | JSON processing | Structured transforms |
| awk | Text processing | Pattern matching |
| ffmpeg | Media processing | Specialized hardware |

### Lever 3: Broad Utility (Amortized Awareness)
| Tool | Breadth of Use | Frequency |
|------|----------------|-----------|
| curl | Universal HTTP | Very High |
| git | Universal VCS | Very High |
| jq | JSON processing | Very High |
| sqlite | Embedded database | High |
| docker | Containerization | High |
| just | Task running | High |

### Lever 5: Friction Reduction
| Tool | Friction Reduced | Impact |
|------|------------------|--------|
| jc | CLI → JSON | Modernizes 100+ tools |
| timeout | Prevents hangs | Critical safety |
| sponge | Safe file writes | Prevents race conditions |
| chronic | Output on failure | Reduces noise |
| expect | Automates prompts | Solves blockers |
| sd | Simple sed | No escaping |

---

## 3.2 Tools by Language Ecosystem

### Python Ecosystem
| Tool | Purpose | Priority |
|------|---------|----------|
| python | Runtime | Core |
| pip/uv | Packages | Core |
| pytest | Testing | Core |
| ruff | Linting | High |
| mypy | Types | High |
| black | Formatting | Medium |
| poetry | Project mgmt | Medium |

### Node.js Ecosystem
| Tool | Purpose | Priority |
|------|---------|----------|
| node | Runtime | Core |
| npm/pnpm | Packages | Core |
| jest/vitest | Testing | Core |
| eslint | Linting | High |
| prettier | Formatting | High |
| typescript | Types | High |

---

## 3.3 Tools with Excellent Agent UX

These tools have characteristics that make them particularly agent-friendly:

| Tool | Why Excellent | Key Feature |
|------|---------------|-------------|
| ripgrep | Simple flags, JSON output | `--json` |
| jc | Converts anything to JSON | Universal |
| sqlite3 | Agents know SQL well | Query language |
| just | No tab issues, simple | Clean syntax |
| timeout | Single purpose, clear | Safety |
| sponge | Solves common problem | Safety |
| shellcheck | Clear output, fixes | Actionable |
| diff | Canonical, simple | Universal |
| bc | Deterministic math | Accuracy |
| curl | Well-trained | Ubiquitous |

---

## 3.4 Tools Requiring Recipes (Complex Syntax)

These tools are powerful but need wrapper recipes for reliable agent use:

| Tool | Complexity Source | Recipe Needed |
|------|-------------------|---------------|
| ffmpeg | Arcane flags | Video/audio recipes |
| jq | Syntax hallucinations | Common transforms |
| awk | Programming language | Text processing recipes |
| sed | Escaping rules | Replace patterns |
| find | Many flags | Common searches |
| curl | Flag combinations | API call recipes |
| git | Many subcommands | Workflow recipes |
| docker | Complex builds | Dockerfile patterns |

---

# Part 4: Raw Lists for Later Filtering

## 4.1 All Tools Alphabetically (~245)

```
0x, actionlint, age, arp, artillery, ast-grep, atlas, autossh, ausearch, aws, az

b2sum, bandit, base32, base64, basenc, bc, bentoml, blkid, bore, bpftrace, bridge, bfs, buf

cal, cat, ccze, cd, certbot, cheat, chmod, chown, chroot, chrt, chronic, cksum,
cmake, cmp, column, combine, comm, cookiecutter, cp, cpio, csplit, ctop, ctstat, curl, cut

datamash, datasette, date, dateadd, datediff, dateround, dateseq, datesort, datetest, datezone,
dbmate, dc, dd, deno, depcheck, devlink, df, diff, diff3, difft, difftastic, dig,
direnv, disown, dive, dmesg, dnsutils, docker, docker-compose, dolt, dpkg, du, duckdb, duf, dust, dvc, dyff

ed, eject, entr, env, envsubst, errno, esbuild, eslint, exiftool, expect, expand

factor, fallocate, fastfetch, fd, fdfind, ffmpeg, ffprobe, file, find,
findmnt, flock, fmt, fold, free, fswatch, fuser, fzf

gcc, gcloud, gdb, genl, getopt, gh, git, git-crypt,
git-extras, git-lfs, github-actions, gitleaks,
glab, glow, gnupg, gpg, gpt4all, graphql-codegen, graphviz, groups

hadolint, head, hexdump, hey, hostname, hurl, huggingface-hub, hwclock, hyperfine

iconv, id, ifconfig, ifdata, ifne, imagemagick, inotifywait, install,
invoke, ionice, iostat, ip, iptables

j2cli, jest, jd, jo, join, journalctl, jq, json-server, jsonlint, just

k6

ldd, less, libcst, lima, lldb, llm, ln, lnstat, locate, logger, logrotate, look, losetup,
ls, lsblk, lscpu, lshw, lsmem, lsns, lsof, lspci, lsusb, ltrace

make, man, mamba, massif, mcookie, meilisearch, miller, mispipe, mise, mitmproxy, mkdir, mkcert, mkdocs, mlflow,
mo, mosh, mount, mpstat, mv

namei, nc, netstat, ngrep, nice, ninja, nl, nmap, node, nohup, npm, nstat, nsenter, ntfy, nvm,
numaverage, numbound, numfmt, numgrep, numinterval, numnormalize, numprocess, numrandom, numrange, numround, numsum, nvidia-smi

ollama, openapi-generator, openssl

pacman, pandoc, parallel, partx, pass, paste, patch, pbcopy, pbpaste, pdm,
pee, perf, pet, pgrep, pidof, pidwait, pigz, ping, pip, pipdeptree,
pip-tools, pipx, pkill, playwright, pmap, pnpm, poetry,
postgresql, pr, pre-commit, prettier, printf, printenv,
prlimit, prism, protoc, ps, pstree, psql, pv, pwd, pwdx, py-spy, pyenv, pyright, pytest, python

q, qalc, qsv, quicktype

rdma, read, readlink, realpath, recode, redis-cli, redoc,
rename, renice, rev, rg, rhash, rm, rmdir, route, rpc, rsync, rtmon, ruff

safety, sar, scalene, schemathesis, screen, script, scriptreplay, sd, sdiff, sed, semantic-release, semgrep,
sensors, seq, setsid, setterm, sha1sum, sha256sum, sha512sum, shfmt, shred, shuf, skill, skeema, slabtop, sleep, snice,
sops, sort, split, sponge, sqlite3, sqlite-utils, sqitch, sqlx, ss, ssh, ssh-agent, ssh-keygen,
sshfs, sshuttle, stat, stdbuf, strace, strptime, swagger-cli, swagger-ui, swapon, swapoff, sysctl, sysprof

tac, tail, tar, taskset, tc, tcpdump, tee,
text-generation-inference, textql, time,
timeout, tload, tmux, tokei, torchserve, touch, tr,
tree, triton, trivy, ts, tsc, tshark, tsort, tsx, typescript

umount, uname, unexpand, uniq, units, unshare, unoconv, unzip, updatedb,
uptime, uuidgen, uuencode, uv

valgrind, vault, vegeta, vim, vite, vllm, vmstat, volta

w, watch, watchman, wc, wdctl, weasyprint, wget, whisper, which, who,
whoami, wiggle, wipefs, wl-copy, wl-paste, wrangler, wrk, wrk2

xargs, xclip, xh, xidel, xmake, xmlstarlet, xsv, xxd, xz

yamllint, yes, yt-dlp, yq

z, zellij, zip, zrun, zsh, zstd
```

---

## 4.2 Tools by Package/Suite Origin

### moreutils
```
sponge, chronic, ts, ifne, pee, combine, isutf8, lckdo, mispipe,
parallel, zrun, errno, ifdata
```

### util-linux
```
lsblk, lscpu, lsmem, lsns, findmnt, flock, uuidgen, column, getopt, hexdump,
rename, rev, script, scriptreplay, setsid, taskset, timeout, cal, chrt, dmesg,
eject, fallocate, ionice, logger, look, mcookie, namei, nsenter, partx, prlimit,
renice, setterm, swapon, swapoff, unshare, wdctl, wipefs, blkid, losetup, mount,
umount, hwclock
```

### GNU coreutils
```
cat, cp, mv, rm, ln, mkdir, rmdir, touch, chmod, chown, chgrp, install, shred,
head, tail, cut, paste, sort, uniq, wc, tr, fold, fmt, pr, nl, expand, unexpand,
split, csplit, join, comm, tac, ls, stat, file, du, df, readlink, realpath,
echo, printf, yes, seq, md5sum, sha1sum, sha256sum, sha512sum, b2sum, cksum,
base64, base32, basenc, basename, dirname, pwd, date, sleep, env, printenv,
uname, hostname, whoami, id, groups, users, who, uptime, tee, xargs, nohup,
nice, stdbuf, numfmt, factor, shuf, test, true, false
```

### procps-ng
```
ps, top, free, vmstat, uptime, w, pgrep, pkill, pidof, pidwait, pmap, pwdx,
slabtop, sysctl, watch, skill, snice, tload
```

### iproute2
```
ip, ss, tc, bridge, nstat, devlink, rdma, lnstat, rtmon, ctstat, routel, arpd, genl
```

### findutils/diffutils
```
find, xargs, locate, updatedb, diff, diff3, sdiff, cmp, patch
```

### Modern CLI Tools (Rust-based)
```
ripgrep, fd, dust, duf, sd, difftastic,
hyperfine, tokei, xh, ast-grep
```

### Modern CLI Tools (Go-based)
```
duf, fzf, miller, yq, gh, glab
```

---

## 4.3 Installation Requirements Analysis

This section categorizes all tools by their installation method: pre-installed, single package, or individual install.

### Pre-Installed on Most Linux Systems (~150 tools)

These tools come with standard Linux distributions and require no installation.

| Package | Tools | Size | Install Check |
|---------|-------|------|---------------|
| **GNU coreutils** | cat, cp, mv, rm, ln, mkdir, rmdir, touch, chmod, chown, chgrp, install, shred, head, tail, cut, paste, sort, uniq, wc, tr, fold, fmt, pr, nl, expand, unexpand, split, csplit, join, comm, tac, ls, stat, file, du, df, readlink, realpath, echo, printf, yes, seq, md5sum, sha1sum, sha256sum, sha512sum, b2sum, cksum, base64, base32, basenc, basename, dirname, pwd, date, sleep, env, printenv, uname, hostname, whoami, id, groups, users, who, uptime, tee, xargs, nohup, nice, stdbuf, numfmt, factor, shuf, test, true, false (~70 tools) | ~7 MB | `cat --version` |
| **util-linux** | lsblk, lscpu, lsmem, lsns, findmnt, flock, uuidgen, column, getopt, hexdump, rename, rev, script, scriptreplay, setsid, taskset, timeout, cal, chrt, dmesg, eject, fallocate, ionice, logger, look, mcookie, namei, nsenter, partx, prlimit, renice, setterm, swapon, swapoff, unshare, wdctl, wipefs, blkid, losetup, mount, umount, hwclock (~40 tools) | ~3.5 MB | `lsblk --version` |
| **procps-ng** | ps, top, free, vmstat, uptime, w, pgrep, pkill, pidof, pidwait, pmap, pwdx, slabtop, sysctl, watch, skill, snice, tload (~17 tools) | ~2 MB | `ps --version` |
| **iproute2** | ip, ss, tc, bridge, nstat, devlink, rdma, lnstat, rtmon, ctstat, routel, arpd, genl (~13 tools) | ~3 MB | `ip -V` |
| **net-tools** (legacy) | ifconfig, netstat, route, arp (4 tools) | ~0.2 MB | `ifconfig --version` |
| **findutils** | find, xargs, locate, updatedb (4 tools) | ~0.6 MB | `find --version` |
| **diffutils** | diff, diff3, sdiff, cmp, patch (5 tools) | ~0.5 MB | `diff --version` |
| **Other standard** | awk, sed, ssh, ssh-keygen, ssh-agent, curl, wget, git, make, tar, gzip, gunzip (12+ tools) | ~25 MB | Various |

**Total pre-installed: ~150 tools (~42 MB already on system)**

---

### Single Package Install (~75 tools)

These tools are bundled in packages - one `apt install` gets multiple tools.

| Package | Tools Included | Count | Size | Ubuntu/Debian Install |
|---------|----------------|-------|------|----------------------|
| **moreutils** | sponge, chronic, ts, ifne, pee, combine, isutf8, lckdo, mispipe, parallel, zrun, errno, ifdata | 13 | ~0.2 MB | `sudo apt install moreutils` |
| **dateutils** | dateadd, datediff, dateseq, dateround, datesort, datetest, datezone, strptime | 8 | ~1 MB | `sudo apt install dateutils` |
| **num-utils** | numaverage, numbound, numgrep, numinterval, numnormalize, numprocess, numrandom, numrange, numround, numsum | 10 | ~0.1 MB | `sudo apt install num-utils` |
| **git-extras** | git-summary, git-effort, git-authors, git-changelog, git-commits-since, git-count, git-create-branch, git-delete-branch, git-delete-merged-branches, git-delta, git-ignore, git-info, git-fork, git-fresh-branch, git-line-summary, git-lock, git-unlock, git-merge-into, git-missing, git-obliterate, git-pr, git-release, git-rename-branch, git-rename-tag, git-repl, git-root, git-setup, git-show-tree, git-squash, git-standup, git-touch, git-undo | 31 | ~0.5 MB | `sudo apt install git-extras` |
| **build-essential** | gcc, g++, make, libc-dev | 4 | ~250 MB | `sudo apt install build-essential` |
| **dnsutils** | dig, nslookup, host | 3 | ~0.3 MB | `sudo apt install dnsutils` |
| **netcat** | nc | 1 | ~0.1 MB | `sudo apt install netcat-openbsd` |
| **tree** | tree | 1 | ~0.1 MB | `sudo apt install tree` |
| **bc** | bc, dc | 2 | ~0.2 MB | `sudo apt install bc` |
| **file** | file | 1 | ~0.1 MB | `sudo apt install file` |
| **rsync** | rsync | 1 | ~0.8 MB | `sudo apt install rsync` |

**Total from packages: ~75 tools (~253 MB, mostly build-essential)**

---

### Individual Installation Required (~70 tools)

These tools must be installed individually via various package managers.

#### Modern CLI Tools (Rust-based) - Individual Install
| Tool | Size | Install Method | Command |
|------|------|---------------|---------|
| **ripgrep** | ~5 MB | apt/cargo | `sudo apt install ripgrep` or `cargo install ripgrep` |
| **fd** | ~3 MB | apt/cargo | `sudo apt install fd-find` (binary: `fdfind`) |
| **dust** | ~2 MB | cargo | `cargo install du-dust` |
| **duf** | ~3 MB | apt/go | `sudo apt install duf` |
| **sd** | ~2 MB | cargo | `cargo install sd` |
| **difftastic** | ~12 MB | cargo | `cargo install difftastic` |
| **hyperfine** | ~2 MB | apt/cargo | `sudo apt install hyperfine` |
| **tokei** | ~3 MB | cargo | `cargo install tokei` |
| **xh** | ~5 MB | cargo | `cargo install xh` |
| **ast-grep** | ~7 MB | cargo/npm | `cargo install ast-grep` or `npm i -g @ast-grep/cli` |

**Rust tools subtotal: ~44 MB**

#### Modern CLI Tools (Go-based) - Individual Install
| Tool | Size | Install Method | Command |
|------|------|---------------|---------|
| **fzf** | ~4 MB | apt/go | `sudo apt install fzf` |
| **miller** | ~27 MB | apt | `sudo apt install miller` |
| **yq** | ~10 MB | pip/go | `pip install yq` or `go install github.com/mikefarah/yq/v4@latest` |
| **gh** | ~54 MB | apt | `sudo apt install gh` |
| **glab** | ~43 MB | apt | `sudo apt install glab` |

**Go tools subtotal: ~138 MB**

#### JSON/Data Tools - Individual Install
| Tool | Size | Install Method | Command |
|------|------|---------------|---------|
| **jq** | ~0.1 MB | apt | `sudo apt install jq` |
| **jc** | ~3 MB | pip | `pip install jc` |
| **jo** | ~0.1 MB | apt | `sudo apt install jo` |
| **jd** | ~5 MB | go | `go install github.com/josephburnett/jd@latest` |
| **qsv** | ~15 MB | cargo | `cargo install qsv` |
| **q** | ~2 MB | pip | `pip install q-text-as-data` |

**JSON/Data tools subtotal: ~25 MB**

#### Database Tools - Individual Install
| Tool | Size | Install Method | Command |
|------|------|---------------|---------|
| **sqlite3** | ~2 MB | apt | `sudo apt install sqlite3` |
| **sqlite-utils** | ~2 MB | pip | `pip install sqlite-utils` |
| **DuckDB** | ~40 MB | apt/pip | `pip install duckdb` |
| **psql** | ~5 MB | apt | `sudo apt install postgresql-client` |
| **mysql** | ~10 MB | apt | `sudo apt install mysql-client` |
| **redis-cli** | ~1 MB | apt | `sudo apt install redis-tools` |
| **datasette** | ~5 MB | pip | `pip install datasette` |

**Database tools subtotal: ~65 MB**

#### Linting & Validation - Individual Install
| Tool | Size | Install Method | Command |
|------|------|---------------|---------|
| **shellcheck** | ~8 MB | apt | `sudo apt install shellcheck` |
| **shfmt** | ~3 MB | go | `go install mvdan.cc/sh/v3/cmd/shfmt@latest` |
| **ruff** | ~18 MB | pip | `pip install ruff` |
| **eslint** | ~5 MB | npm | `npm install -g eslint` |
| **pyright** | ~22 MB | pip/npm | `pip install pyright` |
| **yamllint** | ~1 MB | pip | `pip install yamllint` |
| **jsonlint** | ~0.5 MB | npm | `npm install -g jsonlint` |
| **hadolint** | ~8 MB | apt | Download from GitHub releases |
| **actionlint** | ~5 MB | go | `go install github.com/rhysd/actionlint/cmd/actionlint@latest` |
| **semgrep** | ~50 MB | pip | `pip install semgrep` |

**Linting tools subtotal: ~121 MB**

#### Container & Environment - Individual Install
| Tool | Size | Install Method | Command |
|------|------|---------------|---------|
| **docker** | ~400 MB | apt | Follow Docker's official install guide |
| **docker-compose** | ~50 MB | apt/pip | `sudo apt install docker-compose-plugin` |
| **direnv** | ~2 MB | apt | `sudo apt install direnv` |
| **mise** | ~15 MB | curl | `curl https://mise.run \| sh` |
| **pyenv** | ~3 MB | curl | `curl https://pyenv.run \| bash` |
| **nvm** | ~0.5 MB | curl | `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/HEAD/install.sh \| bash` |

**Container/Env tools subtotal: ~471 MB (mostly Docker)**

#### Security Tools - Individual Install
| Tool | Size | Install Method | Command |
|------|------|---------------|---------|
| **sops** | ~12 MB | apt/go | `go install github.com/getsops/sops/v3/cmd/sops@latest` |
| **age** | ~3 MB | apt | `sudo apt install age` |
| **gpg** | ~5 MB | apt | `sudo apt install gnupg` |
| **pass** | ~0.1 MB | apt | `sudo apt install pass` |
| **mkcert** | ~5 MB | go | `go install filippo.io/mkcert@latest` |
| **gitleaks** | ~15 MB | go | `go install github.com/gitleaks/gitleaks/v8@latest` |
| **trivy** | ~50 MB | apt | Follow Aqua's install guide |

**Security tools subtotal: ~90 MB**

#### Media & Documents - Individual Install
| Tool | Size | Install Method | Command |
|------|------|---------------|---------|
| **ffmpeg** | ~80 MB | apt | `sudo apt install ffmpeg` |
| **ImageMagick** | ~20 MB | apt | `sudo apt install imagemagick` |
| **exiftool** | ~25 MB | apt | `sudo apt install libimage-exiftool-perl` |
| **pandoc** | ~195 MB | apt | `sudo apt install pandoc` |
| **mermaid-cli** | ~150 MB | npm | `npm install -g @mermaid-js/mermaid-cli` |
| **graphviz** | ~5 MB | apt | `sudo apt install graphviz` |
| **weasyprint** | ~30 MB | pip | `pip install weasyprint` |

**Media/Docs tools subtotal: ~505 MB (pandoc + mermaid are heavy)**

#### Task Runners & Build - Individual Install
| Tool | Size | Install Method | Command |
|------|------|---------------|---------|
| **just** | ~3 MB | cargo | `cargo install just` |
| **invoke** | ~2 MB | pip | `pip install invoke` |
| **poetry** | ~18 MB | pip | `pip install poetry` |
| **uv** | ~12 MB | pip/curl | `pip install uv` or `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| **cmake** | ~36 MB | apt | `sudo apt install cmake` |
| **ninja** | ~0.2 MB | apt | `sudo apt install ninja-build` |

**Task/Build tools subtotal: ~71 MB**

#### Testing - Individual Install
| Tool | Size | Install Method | Command |
|------|------|---------------|---------|
| **pytest** | ~8 MB | pip | `pip install pytest` |
| **jest** | ~25 MB | npm | `npm install -g jest` |
| **json-server** | ~5 MB | npm | `npm install -g json-server` |
| **hurl** | ~8 MB | apt/cargo | `cargo install hurl` |

**Testing tools subtotal: ~46 MB**

---

### Summary by Installation Method

| Category | Tool Count | Disk Size | Installation Effort |
|----------|------------|-----------|---------------------|
| **Pre-installed** | ~150 | ~42 MB | None (already on system) |
| **Single apt package** | ~75 | ~253 MB | 6-8 commands |
| **Individual install** | ~70 | ~1.6 GB | 50+ commands |
| **Total** | ~295 | **~1.9 GB** | - |

#### Individual Install Breakdown by Category

| Category | Size | Notes |
|----------|------|-------|
| Rust CLI tools | ~44 MB | Fast, single binaries |
| Go CLI tools | ~138 MB | gh/glab are large |
| JSON/Data tools | ~25 MB | Lightweight |
| Database tools | ~65 MB | DuckDB is largest |
| Linting tools | ~121 MB | semgrep is heavy |
| Container/Env | ~471 MB | Docker dominates |
| Security tools | ~90 MB | trivy is largest |
| Media/Docs | ~505 MB | pandoc + mermaid |
| Task/Build | ~71 MB | cmake is largest |
| Testing | ~46 MB | jest is largest |

### Recommended Bootstrap Script (Ubuntu/Debian)

```bash
#!/bin/bash
# Essential packages (covers ~230 tools)

# Package bundles
sudo apt update && sudo apt install -y \
    moreutils dateutils num-utils git-extras \
    build-essential dnsutils netcat-openbsd \
    tree bc file rsync

# Core modern tools
sudo apt install -y \
    ripgrep fd-find fzf jq jo miller \
    shellcheck sqlite3 git curl wget \
    hyperfine duf gh glab age

# Python tools (in a venv or with pipx)
pip install jc sqlite-utils ruff pyright yamllint \
    pytest invoke poetry uv

# Node tools
npm install -g eslint prettier json-server

# Rust tools (optional, for latest versions)
cargo install sd difftastic tokei xh ast-grep just
```

---

## 4.4 Installation Commands Quick Reference

### Ubuntu/Debian Core
```bash
# Core tools
sudo apt install ripgrep fd-find jq sqlite3 git curl shellcheck tree

# moreutils
sudo apt install moreutils

# Build tools
sudo apt install build-essential cmake

# Note: fd is 'fdfind' on Debian/Ubuntu
```

### Homebrew (macOS/Linux)
```bash
# Core modern tools
brew install ripgrep fd jq fzf dust duf

# Additional utilities
brew install moreutils shellcheck hyperfine tokei

# Cloud CLIs
brew install awscli
```

### Cargo (Rust)
```bash
cargo install ripgrep fd-find sd difftastic \
    tokei hyperfine ast-grep xh
```

### pip (Python)
```bash
pip install jc sqlite-utils yq
```

### npm (Node.js)
```bash
npm install -g json-server pnpm yarn
```

---

# Appendix: Discarded Tools (255)

The following tools were removed from the inventory as unsuitable for agent use.

---

## Round 1: TUI/Interactive-Only (14 tools)

These tools require cursor navigation or visual interaction that agents cannot perform.

| Tool | Category | Reason | Alternative |
|------|----------|--------|-------------|
| **htop** | Process viewer | Curses TUI | `ps` + `jc` for JSON |
| **btop** | Process viewer | Curses TUI | `ps` + `jc` for JSON |
| **bottom (btm)** | Process viewer | TUI with graphs | `ps`, `free` + `jc` |
| **zenith** | Process viewer | TUI, less maintained | `ps`, `free` + `jc` |
| **ncdu** | Disk usage | Interactive-only | `gdu` (has non-interactive mode) |
| **broot** | File browser | TUI-focused | `fd`, `tree`, `eza` |
| **k9s** | Kubernetes | TUI | `kubectl -o json` |
| **lazygit** | Git | TUI | `git` CLI directly |
| **gitui** | Git | TUI | `git` CLI directly |
| **tig** | Git history | TUI, development slowed | `git log`, `delta` |
| **jless** | JSON viewer | Interactive, read-only | `jq` (scriptable) |
| **fx** | JSON viewer | Interactive, JavaScript syntax | `jq` (standard) |
| **vipe** | Pipeline editor | Requires interactive editor | Remove human from loop |
| **vidir** | Directory editor | Requires interactive editor | `rename`, scripted `mv` |

## Round 1: Superseded/Deprecated (6 tools)

These tools have been replaced by better alternatives.

| Tool | Category | Reason | Better Alternative |
|------|----------|--------|-------------------|
| **neofetch** | System info | Archived April 2024 | `fastfetch` |
| **ag (silver searcher)** | Search | Maintenance mode, slower | `ripgrep` (3x faster) |
| **autojump** | Navigation | Python-based, slower | `zoxide` (Rust, faster) |
| **diff-so-fancy** | Diff | Perl, fewer features | `delta` (Rust, syntax highlighting) |
| **most** | Pager | Obscure, rarely used | `less` or `bat` |
| **nat** | ls replacement | Less maintained | `eza` (active community) |

## Round 1: Excessive Complexity (5 tools)

These tools have steep learning curves that cause more agent failures than successes.

| Tool | Category | Reason | Better Alternative |
|------|----------|--------|-------------------|
| **bazel** | Build | Steep learning curve, agents struggle | `just`, `make`, `task` |
| **airflow** | Workflow | Terrible agent UX, DAG files | `Prefect`, `Dagster`, `Temporal` |
| **spinnaker** | CD | Enterprise complexity, massive setup | `argocd`, `flux` |
| **chef** | Config mgmt | Ruby DSL, complex, declining | `ansible` (YAML, simpler) |
| **puppet** | Config mgmt | Custom DSL, steep learning curve | `ansible` (more intuitive) |
| **sad** | Search/replace | Requires fzf, interactive preview | `sd` |

---

## Round 2: Niche Cloud CLIs (14 tools)

Kept: aws, gcloud, az, flyctl, vercel, wrangler

| Tool | Reason | Alternative |
|------|--------|-------------|
| **doctl** | DigitalOcean declining market share | `aws`, `gcloud` |
| **netlify** | Limited JSON output, vercel more popular | `vercel` |
| **railway** | Niche platform, limited adoption | `flyctl`, `vercel` |
| **render** | Niche, limited JSON support | `flyctl` |
| **heroku** | Platform declining significantly | `flyctl`, `railway` |
| **supabase** | Niche BaaS, limited CLI capabilities | `aws`, `gcloud` |
| **firebase** | Limited JSON, use gcloud instead | `gcloud` |
| **amplify** | Limited JSON, use aws instead | `aws` |
| **oci** | Oracle Cloud is very niche | `aws`, `az`, `gcloud` |
| **linode-cli** | Niche provider | `aws`, `gcloud` |
| **vultr-cli** | Niche provider | `aws`, `gcloud` |
| **hcloud** | Niche (EU-focused Hetzner) | `aws`, `gcloud` |
| **exo** | Very niche (Exoscale) | `gcloud` |
| **scaleway** | Very niche (EU) | `gcloud` |

## Round 2: Specialized Kubernetes Tools (10 tools)

Kept: kubectl, helm, kustomize, stern, minikube, kind, argocd

| Tool | Reason | Alternative |
|------|--------|-------------|
| **k3d** | kind is more popular | `kind` |
| **k3s** | Edge-focused, niche use case | `minikube`, `kind` |
| **skaffold** | Google tool, less adoption | `tilt` → `docker-compose` |
| **tilt** | Niche dev workflow tool | `docker-compose` |
| **telepresence** | Complex setup required | Local k8s cluster |
| **flux** | argocd more popular for GitOps | `argocd` |
| **krew** | Plugin manager, not core | Direct install |
| **kubecolor** | Just a wrapper | `kubectl` |
| **velero** | Specialized backup tool | Cloud-native backups |
| **istio** | Very complex service mesh | Simpler networking |
| **linkerd** | Service mesh, complex | Simpler networking |
| **kubectx/kubens** | Convenience wrappers | `kubectl config` |
| **kubeseal** | Specialized for sealed-secrets | `sops`, `vault` |

## Round 2: Platform-Specific Serverless (8 tools)

Kept: serverless, sam

| Tool | Reason | Alternative |
|------|--------|-------------|
| **chalice** | Python-specific, AWS-only | `serverless`, `sam` |
| **zappa** | Python-specific, less maintained | `serverless` |
| **func** | Azure Functions specific | `serverless` |
| **functions-framework** | GCP-specific | `serverless` |
| **fn** | Oracle's tool, very niche | `serverless` |
| **openfaas-cli** | Self-hosted, complex setup | `serverless` |
| **kubeless** | K8s-specific, declining | `serverless` |

## Round 2: Redundant Browser Automation (9 tools)

Kept: playwright, puppeteer, cypress

| Tool | Reason | Alternative |
|------|--------|-------------|
| **selenium** | Older, playwright better | `playwright` |
| **webdriver** | Low-level, use playwright | `playwright` |
| **chromedp** | Go-specific | `playwright` |
| **rod** | Go-specific, chromedp alternative | `playwright` |
| **splash** | Specialized rendering service | `playwright` |
| **browserless** | Account/service required | `playwright` |
| **nightwatch** | Less popular than cypress | `cypress`, `playwright` |
| **testcafe** | Less popular than cypress | `cypress`, `playwright` |
| **taiko** | Less popular | `playwright` |

## Round 2: Account-Required Monitoring (12 tools)

Kept: prometheus, promtool, vector

| Tool | Reason | Alternative |
|------|--------|-------------|
| **alertmanager** | Requires Prometheus server | Cloud alerts |
| **grafana-cli** | Requires Grafana server | Cloud dashboards |
| **logcli** | Requires Loki server | `vector`, `rg` |
| **telegraf** | InfluxDB ecosystem lock-in | `vector` |
| **influx** | Requires InfluxDB server | `prometheus` |
| **datadog-agent** | Paid account required | `prometheus` |
| **newrelic** | Paid account required | `prometheus` |
| **dd-trace** | Paid account (Datadog) | OpenTelemetry |
| **otel-cli** | Limited standalone use | `prometheus` |
| **jaeger** | Server setup required | Cloud tracing |
| **uptimerobot** | Account required | Cloud monitoring |
| **pingdom** | Account required | Cloud monitoring |

## Round 2: Log Management (10 tools)

Kept: journalctl, logrotate, ccze, angle-grinder

| Tool | Reason | Alternative |
|------|--------|-------------|
| **lnav** | TUI-based (missed in round 1) | `rg`, `angle-grinder` |
| **goaccess** | TUI/web dashboard | `angle-grinder` |
| **kubetail** | K8s-specific, stern better | `stern` |
| **fluentbit** | Requires setup/config | `vector` |
| **fluentd** | Requires setup/config | `vector` |
| **filebeat** | Requires Elastic Stack | `vector` |
| **promtail** | Requires Loki | `vector` |
| **syslog-ng** | Enterprise syslog | `journalctl` |
| **rsyslog** | Enterprise syslog | `journalctl` |

## Round 2: Deployment & CD (8 tools)

Kept: semantic-release

| Tool | Reason | Alternative |
|------|--------|-------------|
| **jenkins-cli** | Jenkins terrible for agents | GitHub Actions, `argocd` |
| **travis** | Platform declining | GitHub Actions |
| **tekton** | K8s-specific, complex | GitHub Actions |
| **lerna** | Monorepo, npm workspaces preferred | `npm workspaces`, `nx` |
| **rush** | Microsoft-specific monorepo | `nx`, `turborepo` |
| **release-it** | npm alternative, less popular | `semantic-release` |

## Round 2: Code Generation (2 tools)

Kept: openapi-generator, plop, hygen, cookiecutter

| Tool | Reason | Alternative |
|------|--------|-------------|
| **swagger-codegen** | Legacy, openapi-generator is newer | `openapi-generator` |
| **yeoman** | Less maintained, complex | `plop`, `hygen` |

---

## Round 3: Account-Required Tools (9 tools)

Removed tools that require external accounts/subscriptions with limited local utility.

### Cloud Platforms (2 tools)

| Tool | Reason | Alternative |
|------|--------|-------------|
| **flyctl** | Fly.io account required | `aws`, `gcloud`, `wrangler` |
| **vercel** | Vercel account required | `wrangler`, direct deploy |

### CI/CD Platforms (2 tools)

| Tool | Reason | Alternative |
|------|--------|-------------|
| **circleci** | CircleCI account required | `github-actions` |
| **gitlab-ci** | GitLab account required | `github-actions` |

### Documentation Platforms (3 tools)

| Tool | Reason | Alternative |
|------|--------|-------------|
| **gitbook** | GitBook account required, limited free | `mkdocs`, `docusaurus` |
| **readme** | ReadMe paid service required | `redoc`, `swagger-ui` |
| **stoplight** | Stoplight account required | `redoc`, `swagger-ui` |

### Secrets Management (2 tools)

| Tool | Reason | Alternative |
|------|--------|-------------|
| **op (1Password)** | 1Password subscription required | `pass`, `gopass`, `sops` |
| **bitwarden-cli** | Bitwarden account required | `pass`, `gopass`, `sops` |

---

## Round 4: Language-Specific & DevOps/Infrastructure Tools (~76 tools)

Removed tools specific to languages outside Python/JavaScript, plus Kubernetes and DevOps/Infrastructure tools. The toolkit now focuses on language-agnostic utilities and Python/JS ecosystems.

### Rust-Specific (12 tools)

| Tool | Reason | Alternative |
|------|--------|-------------|
| **rustc** | Rust-specific compiler | Use language-agnostic tools |
| **cargo** | Rust package/build tool | `make`, `just` for builds |
| **rustup** | Rust version manager | N/A |
| **clippy** | Rust linter | `semgrep` for patterns |
| **rustfmt** | Rust formatter | N/A |
| **cargo-audit** | Rust dependency audit | `trivy` |
| **cargo-watch** | Rust file watcher | `entr`, `watchman` |
| **cargo-flamegraph** | Rust profiler | `perf`, `flamegraph` |
| **rust-analyzer** | Rust language server | N/A |
| **samply** | Rust sampling profiler | `perf` |
| **rustdoc** | Rust documentation | `doxygen` |

### Go-Specific (15 tools)

| Tool | Reason | Alternative |
|------|--------|-------------|
| **go** | Go toolchain | Use language-agnostic tools |
| **gofmt** | Go formatter | N/A |
| **gopls** | Go language server | N/A |
| **golangci-lint** | Go meta-linter | `semgrep` |
| **delve** | Go debugger | `gdb` |
| **goreleaser** | Go release tool | `semantic-release` |
| **goose** | Go migrations | `dbmate`, `alembic` |
| **gojq** | jq in Go | `jq` |
| **gron** | Flatten JSON | `jq` |
| **gdu** | Disk usage | `dust`, `duf` |
| **curlie** | curl wrapper | `curl`, `xh` |
| **pup** | HTML parser | `xidel`, `xmlstarlet` |
| **task (go-task)** | Task runner | `just`, `make` |
| **gum** | Shell UX | Shell scripts |
| **pprof** | Go profiler | `perf` |
| **godoc** | Go documentation | `doxygen` |

### Java/Scala (8 tools)

| Tool | Reason | Alternative |
|------|--------|-------------|
| **java** | Java runtime | Use language-agnostic tools |
| **javac** | Java compiler | N/A |
| **maven** | Java build | `make` |
| **gradle** | Java build | `make` |
| **sdkman** | JVM version manager | N/A |
| **kotlin** | Kotlin compiler | N/A |
| **scala** | Scala compiler | N/A |
| **async-profiler** | JVM profiler | `perf` |

### Ruby (7 tools)

| Tool | Reason | Alternative |
|------|--------|-------------|
| **ruby** | Ruby interpreter | Use Python/JS |
| **gem** | Ruby packages | `pip`, `npm` |
| **bundle** | Ruby bundler | `poetry`, `pnpm` |
| **rbenv** | Ruby versions | `pyenv`, `nvm` |
| **rubocop** | Ruby linter | `ruff`, `eslint` |
| **rails** | Rails CLI | Django, Express |
| **rake** | Ruby tasks | `just`, `make` |
| **rails migrations** | Rails migrations | `dbmate` |

### PHP (2 tools)

| Tool | Reason | Alternative |
|------|--------|-------------|
| **php** | PHP interpreter | Use Python/JS |
| **composer** | PHP packages | `pip`, `npm` |

### Kubernetes Ecosystem (8 tools)

| Tool | Reason | Alternative |
|------|--------|-------------|
| **kubectl** | K8s CLI | Docker for local dev |
| **helm** | K8s package manager | Docker Compose |
| **kustomize** | K8s config | Docker Compose |
| **stern** | K8s log aggregation | `docker logs` |
| **minikube** | Local K8s | Docker Desktop |
| **kind** | K8s in Docker | Docker Compose |
| **argocd** | GitOps CD | GitHub Actions |
| **crictl** | Container runtime | `docker`, `podman` |

### DevOps/Infrastructure (24 tools)

| Tool | Reason | Alternative |
|------|--------|-------------|
| **terraform** | IaC | Cloud CLIs directly |
| **opentofu** | IaC | Cloud CLIs directly |
| **pulumi** | IaC | Cloud CLIs directly |
| **cdktf** | Terraform CDK | Cloud CLIs directly |
| **crossplane** | K8s IaC | Cloud CLIs directly |
| **ansible** | Config management | Shell scripts |
| **ansible-navigator** | Ansible UI | Shell scripts |
| **salt** | Config management | Shell scripts |
| **cloudformation** | AWS IaC | `aws` CLI |
| **bicep** | Azure IaC | `az` CLI |
| **deployment-manager** | GCP IaC | `gcloud` CLI |
| **serverless** | Serverless framework | Cloud CLIs |
| **sam** | AWS SAM | `aws` CLI |
| **prometheus** | Metrics | Cloud monitoring |
| **promtool** | Prometheus tools | Cloud monitoring |
| **vector** | Log pipeline | `journalctl`, cloud logs |
| **vagrant** | VM management | Docker, cloud VMs |
| **earthly** | Containerized builds | Docker, GitHub Actions |
| **dagger** | CI/CD engine | GitHub Actions |
| **n8n** | Workflow automation | Code-based automation |
| **prefect** | Data pipelines | Python scripts |
| **dagster** | Data orchestration | Python scripts |
| **flyway** | DB migrations (JVM) | `dbmate` |
| **liquibase** | DB migrations (JVM) | `dbmate` |
| **migrate** | Go migrations | `dbmate` |
| **mockserver** | HTTP mock (Java) | `json-server` |
| **wiremock** | HTTP stubbing (Java) | `json-server` |
| **quickcheck** | Property testing (Rust/Haskell) | `hypothesis` |
| **staticcheck** | Go analysis | `semgrep` |
| **sqlc** | SQL to Go | Raw SQL |
| **sqlboiler** | Go ORM | Raw SQL |
| **gqlgen** | GraphQL Go | `graphql-codegen` |
| **slate** | API docs (Ruby) | `redoc` |

---

## Round 5: ORM Tools (9 tools)

Removed ORM and ORM-related migration tools. Agents should use raw SQL or simple migration tools like dbmate.

| Tool | Reason | Alternative |
|------|--------|-------------|
| **alembic** | SQLAlchemy ORM migrations | `dbmate`, raw SQL |
| **prisma** | TypeScript ORM | Raw SQL, `sqlite-utils` |
| **prisma migrate** | Prisma migrations | `dbmate` |
| **drizzle** | TypeScript ORM | Raw SQL |
| **drizzle-kit** | Drizzle migrations | `dbmate` |
| **knex** | Node.js query builder/ORM | Raw SQL |
| **sequelize-cli** | Sequelize ORM migrations | `dbmate` |
| **typeorm** | TypeORM migrations | `dbmate` |
| **django migrations** | Django ORM migrations | `dbmate`, raw SQL |

---

## Round 6: Redundant Tools (~51 tools)

Removed tools where better alternatives exist or multiple tools serve the same purpose.

### Text Search & Processing (5 tools)
| Tool | Reason | Better Alternative |
|------|--------|-------------------|
| **grep** | Slower, no .gitignore | `ripgrep` |
| **ugrep** | Less known than ripgrep | `ripgrep` |
| **choose** | Niche cut alternative | `cut` |
| **hck** | Niche cut alternative | `cut` |
| **comby** | Less active than ast-grep | `ast-grep` |

### JSON/Structured Data (6 tools)
| Tool | Reason | Better Alternative |
|------|--------|-------------------|
| **gojq** | jq clone, minor differences | `jq` |
| **jaq** | jq clone, missing features | `jq` |
| **gron** | Niche use case | `jq` |
| **dasel** | Less popular than yq | `yq` |
| **xsv** | Unmaintained | `qsv` |
| **csvkit** | Slower than qsv | `qsv` |

### Database (7 tools)
| Tool | Reason | Better Alternative |
|------|--------|-------------------|
| **pgcli** | Enhanced wrapper | `psql` |
| **mycli** | Enhanced wrapper | `mysql` |
| **litecli** | Enhanced wrapper | `sqlite3` |
| **usql** | Multi-db complexity | Native clients |
| **mongosh** | Niche (MongoDB) | Native clients |
| **Typesense** | Similar to Meilisearch | `Meilisearch` |
| **Elasticsearch** | Complex setup | `Meilisearch` |

### Version Control (4 tools)
| Tool | Reason | Better Alternative |
|------|--------|-------------------|
| **git-delta** | Syntax only, not AST | `difftastic` |
| **git-secret** | More complex than git-crypt | `git-crypt` |
| **jj (jujutsu)** | Low adoption | `git` |
| **sapling** | Low adoption | `git` |

### Schema & Validation (3 tools)
| Tool | Reason | Better Alternative |
|------|--------|-------------------|
| **cerberus** | Less popular | `Pydantic` |
| **marshmallow** | Less popular | `Pydantic` |
| **typebox** | Niche | `ajv` |

### HTTP & Networking (7 tools)
| Tool | Reason | Better Alternative |
|------|--------|-------------------|
| **httpie** | Slower than xh | `xh` |
| **curlie** | Extra wrapper | `curl` or `xh` |
| **dog** | dig is standard | `dig` |
| **socat** | Complex | `nc` |
| **scp** | rsync more powerful | `rsync` |
| **localtunnel** | Less reliable | `ngrok` |
| **cloudflared** | Cloudflare-specific | `ngrok` |

### Containers & Environments (6 tools)
| Tool | Reason | Better Alternative |
|------|--------|-------------------|
| **podman** | Docker is standard | `docker` |
| **nerdctl** | Docker is standard | `docker` |
| **colima** | Docker Desktop/lima | `docker` |
| **asdf** | Slower than mise | `mise` |
| **conda** | Slower than mamba | `mamba` |
| **multipass** | Ubuntu-only | `lima` |

### Static Analysis (4 tools)
| Tool | Reason | Better Alternative |
|------|--------|-------------------|
| **biome** | eslint has ecosystem | `eslint` |
| **oxlint** | eslint has ecosystem | `eslint` |
| **mypy** | pyright is faster | `pyright` |
| **trufflehog** | gitleaks is faster | `gitleaks` |

### Workflow & Orchestration (5 tools)
| Tool | Reason | Better Alternative |
|------|--------|-------------------|
| **Inngest** | Temporal is standard | `Temporal` |
| **Hatchet** | Temporal is standard | `Temporal` |
| **Trigger.dev** | Temporal is standard | `Temporal` |
| **Windmill** | UI-focused | `Temporal` |
| **Zapier CLI** | Zapier-specific | `Temporal` |

### Media & Documents (9 tools)
| Tool | Reason | Better Alternative |
|------|--------|-------------------|
| **GraphicsMagick** | ImageMagick is standard | `ImageMagick` |
| **libvips** | Less known | `ImageMagick` |
| **sharp** | Node.js only | `ImageMagick`, `Pillow` |
| **wkhtmltopdf** | Deprecated | `weasyprint` |
| **plantuml** | Java required | `mermaid-cli` |
| **d2** | Less adopted | `mermaid-cli`, `graphviz` |
| **sphinx** | RST complexity | `mkdocs` |
| **typedoc** | TS-specific | `mkdocs` |
| **doxygen** | Complex config | `mkdocs` |

### Other Categories (varies)
| Tool | Reason | Better Alternative |
|------|--------|-------------------|
| **gopass** | pass is simpler | `pass` |
| **step** | Complex PKI | `openssl` |
| **cfssl** | Complex PKI | `openssl` |
| **vitest** | jest is standard | `jest` |
| **httpbin** | Limited utility | `json-server` |
| **puppeteer** | playwright is better | `playwright` |
| **cypress** | playwright is better | `playwright` |
| **yarn** | npm/pnpm sufficient | `npm`, `pnpm` |
| **webpack** | esbuild/vite faster | `esbuild`, `vite` |
| **rollup** | esbuild/vite faster | `esbuild`, `vite` |
| **parcel** | vite is modern standard | `vite` |
| **turbopack** | vite is standard | `vite` |
| **clang** | gcc is more common | `gcc` |
| **swc** | tsc is standard | `tsc` |
| **meson** | cmake more common | `cmake` |

---

## Round 7: Human-Focused & Agent-Unfriendly Tools (~35 tools)

Removed tools designed for human UX (syntax highlighting, autocomplete, TUI), require servers/accounts, or have limited agent utility.

### Human UX Tools (10 tools)
| Tool | Reason | Alternative |
|------|--------|-------------|
| **bat** | Syntax highlighting for humans | `cat` |
| **eza** | Colored ls for humans | `ls` |
| **procs** | Pretty ps for humans | `ps` + `jc` |
| **hexyl** | Visual hex viewer | `xxd` |
| **zoxide** | Interactive directory jumper | `cd` |
| **glances** | System monitoring TUI | `ps`, `free`, `df` |
| **tldr** | Simplified man pages | `man`, `--help` |
| **tracy** | Profiler TUI | Standard profilers |
| **terminal-notifier** | macOS desktop notifications | N/A |
| **spectral** | OpenAPI linting (niche) | JSON Schema |

### Server/Account Required (12 tools)
| Tool | Reason | Alternative |
|------|--------|-------------|
| **kafka** | Requires Kafka cluster | Cloud pub/sub |
| **kcat** | Requires Kafka cluster | Cloud pub/sub |
| **nats** | Requires NATS server | Cloud pub/sub |
| **natscli** | Requires NATS server | Cloud pub/sub |
| **mosquitto_pub** | Requires MQTT broker | Cloud pub/sub |
| **mosquitto_sub** | Requires MQTT broker | Cloud pub/sub |
| **amqp-tools** | Requires AMQP server | Cloud pub/sub |
| **wandb** | Requires W&B account | `mlflow` |
| **clearml** | Requires ClearML server | `mlflow` |
| **lm-studio** | GUI application | `ollama` |
| **jan** | GUI application | `ollama` |
| **mtr** | Requires network privileges | `ping`, `traceroute` |

### GUI/Desktop Focused (3 tools)
| Tool | Reason | Alternative |
|------|--------|-------------|
| **ngrok** | Account required | Direct port forwarding |
| **bruno** | GUI-focused API client | `curl`, `xh` |
| **insomnia** | GUI-focused API client | `curl`, `xh` |

### Limited Agent Utility (10 tools)
| Tool | Reason | Alternative |
|------|--------|-------------|
| **grpcurl** | gRPC niche | HTTP/REST |
| **ghz** | gRPC benchmarking niche | `hey`, `wrk` |
| **newman** | Postman collections only | `hurl` |
| **Temporal** | Workflow orchestration server | Direct code |
| **gifsicle** | GIF optimization niche | `ImageMagick` |
| **optipng** | PNG optimization niche | `ImageMagick` |
| **svgo** | SVG optimization niche | Raw SVG |
| **sox** | Audio processing niche | ffmpeg |
| **asciidoctor** | AsciiDoc niche | `pandoc`, `mkdocs` |

---

*This curated inventory serves as the reference for building a lean, effective agent toolkit. Tools were removed based on criteria: (1) agents cannot operate them (TUI), (2) better alternatives exist, (3) complexity exceeds value, (4) account/server required, (5) niche platform, (6) declining/deprecated, (7) language-specific outside Python/JS, (8) ORM-specific, (9) redundant with better alternative, (10) human-focused UX features.*
