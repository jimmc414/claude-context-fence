# Agent Toolkit: Comprehensive Tool Exploration

**Status:** Exploration Complete
**Date:** 2026-02-03
**Purpose:** Master inventory of all CLI tools with potential agent use cases (no eliminations at this stage)

---

## Overview

This document consolidates research from 7 parallel agents exploring different tool categories. Total tools cataloged: **400+**

**Research Sources:**
1. Unix tool suites (moreutils, util-linux, coreutils, procps, etc.)
2. Modern CLI rewrites (Rust/Go tools)
3. Data processing and transformation tools
4. Developer productivity tools
5. System introspection and debugging tools
6. File/text manipulation tools
7. Cloud, infrastructure, and miscellaneous tools

---

## Category 1: Unix Tool Suites

### moreutils (Critical for Agents)

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **sponge** | Soak stdin, write to file safely | In-place file modification without race conditions |
| **chronic** | Run command, show output only on failure | Prevent context flooding with success logs |
| **ts** | Timestamp lines | Add timestamps to any output |
| **vidir** | Edit directory in text editor | Bulk rename files |
| **vipe** | Insert text editor into pipeline | Manual intervention in pipelines |
| **ifne** | Run command only if stdin non-empty | Conditional execution |
| **pee** | Tee stdin to multiple commands | Split output to parallel processors |
| **combine** | Combine lines from files using set operations | Set operations on file contents |
| **isutf8** | Check if input is valid UTF-8 | Encoding validation |
| **lckdo** | Run command with a lock | Prevent concurrent execution |
| **mispipe** | Return exit code of first command in pipe | Better error handling in pipes |
| **parallel** | Run commands in parallel | Batch processing |
| **zrun** | Run command with decompressed input | Process compressed files transparently |

### util-linux (System Utilities)

| Tool | Description | Agent Use Case | JSON Support |
|------|-------------|----------------|--------------|
| **lsblk** | List block devices | Disk detection | `-J` |
| **lscpu** | CPU information | Capability detection | `-J` |
| **lsmem** | Memory info | Resource awareness | `-J` |
| **findmnt** | Find mounted filesystems | Mount point discovery | `-J` |
| **flock** | File locking | Prevent race conditions | - |
| **uuidgen** | Generate UUIDs | Create unique identifiers | - |
| **column** | Columnate output | Table formatting | `-J` |
| **getopt** | Parse command options | Argument parsing | - |
| **hexdump** | Hex dump | Binary analysis | - |
| **rename** | Batch rename files | File renaming | - |
| **rev** | Reverse lines | Text manipulation | - |
| **script** | Record terminal session | Session capture | - |
| **setsid** | Run in new session | Process isolation | - |
| **taskset** | CPU affinity | Performance tuning | - |
| **timeout** | Limit execution time | Prevent infinite hangs | - |
| **cal** | Calendar display | Date reference | - |
| **chrt** | Set scheduling policy | Process priority | - |
| **dmesg** | Kernel messages | Hardware debugging | `--json` |
| **eject** | Eject media | Device control | - |
| **fallocate** | Preallocate space | Disk management | - |
| **ionice** | I/O scheduling | Resource management | - |
| **logger** | Write to syslog | Logging | - |
| **look** | Binary search in sorted file | Fast lookup | - |
| **mcookie** | Generate random cookie | Token generation | - |
| **namei** | Follow pathname | Path resolution | - |
| **nsenter** | Enter namespaces | Container debugging | - |
| **partx** | Partition management | Disk operations | - |
| **prlimit** | Process resource limits | Resource control | - |
| **renice** | Alter priority | Process management | - |
| **setterm** | Terminal settings | Display control | - |
| **swapon/swapoff** | Swap management | Memory management | - |
| **unshare** | Create new namespace | Isolation | - |
| **wdctl** | Watchdog control | System monitoring | - |
| **wipefs** | Wipe filesystem signatures | Disk cleanup | - |

### GNU coreutils

| Category | Tools | Notes |
|----------|-------|-------|
| **File Operations** | cat, cp, mv, rm, ln, mkdir, rmdir, touch, chmod, chown, chgrp | Fundamentals |
| **Text Processing** | head, tail, cut, paste, sort, uniq, wc, tr, fold, fmt, pr | Text manipulation |
| **File Info** | ls, stat, file, du, df | File metadata |
| **Output** | echo, printf, yes, seq | Generators |
| **Hashing** | md5sum, sha1sum, sha256sum, sha512sum, b2sum | Integrity checks |
| **Encoding** | base64, base32, basenc | Data encoding |
| **Comparison** | comm, diff, cmp | File comparison |
| **Path** | basename, dirname, realpath, readlink, pwd | Path manipulation |
| **System** | date, sleep, timeout, env, printenv, uname, hostname | System info |
| **Users** | id, whoami, groups, users, who, w | User info |
| **Misc** | tee, xargs, nohup, nice, stdbuf, numfmt, factor, shuf | Utilities |

### procps (Process Tools)

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **ps** | Process status | Process enumeration |
| **top** | Process viewer | Resource monitoring (batch mode: `-b`) |
| **free** | Memory usage | Resource awareness |
| **vmstat** | Virtual memory stats | System health |
| **uptime** | System uptime | Status check |
| **w** | Who is logged in | User activity |
| **pgrep** | Find processes by name | Process discovery |
| **pkill** | Kill by name | Process control |
| **pidof** | Find PID | Process lookup |
| **pmap** | Process memory map | Memory analysis |
| **pwdx** | Process working directory | Context discovery |
| **sysctl** | Kernel parameters | System tuning |
| **watch** | Repeat command | Polling |

### iproute2 (Network Tools)

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **ip** | Network configuration | Interface/route management |
| **ss** | Socket statistics | Connection monitoring |
| **tc** | Traffic control | QoS/shaping |
| **bridge** | Bridge management | Network configuration |
| **nstat** | Network statistics | Traffic analysis |
| **devlink** | Device link control | NIC management |
| **rdma** | RDMA configuration | High-performance networking |

### net-tools (Legacy Network)

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **ifconfig** | Interface config (legacy) | Basic network info |
| **netstat** | Network statistics (legacy) | Connection info |
| **route** | Routing table (legacy) | Route viewing |
| **arp** | ARP cache | Network debugging |
| **hostname** | Hostname operations | Identity |

### dateutils

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **dateadd** | Add duration to date | Date arithmetic |
| **datediff** | Compute date difference | Duration calculation |
| **dateseq** | Generate date sequence | Date iteration |
| **dateround** | Round date to unit | Date normalization |
| **datesort** | Sort by date | Chronological ordering |
| **datetest** | Compare dates | Date conditionals |
| **datezone** | Timezone conversion | TZ handling |
| **strptime** | Parse date strings | Date parsing |

### num-utils

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **numaverage** | Calculate average | Statistics |
| **numbound** | Find min/max | Range detection |
| **numgrep** | Filter by numeric range | Numeric filtering |
| **numinterval** | Show intervals | Difference analysis |
| **numnormalize** | Normalize to 0-1 | Data normalization |
| **numprocess** | Math expressions | Numeric transformation |
| **numrandom** | Random numbers | Test data |
| **numrange** | Generate sequences | Iteration |
| **numround** | Round numbers | Precision control |
| **numsum** | Sum numbers | Aggregation |

---

## Category 2: Modern CLI Replacements

### File Listing

| Tool | Replaces | Language | Key Feature |
|------|----------|----------|-------------|
| **eza** | ls | Rust | Git integration, icons, JSON output |
| **lsd** | ls | Rust | Icons, tree view, colors |
| **nat** | ls | Rust | Minimal, colorful |
| **broot** | tree/find | Rust | Interactive navigation, fuzzy search |

### File Viewing

| Tool | Replaces | Language | Key Feature |
|------|----------|----------|-------------|
| **bat** | cat | Rust | Syntax highlighting, git integration |
| **glow** | - | Go | Markdown rendering |
| **most** | less | C | Multi-window, horizontal scroll |

### File Finding

| Tool | Replaces | Language | Key Feature |
|------|----------|----------|-------------|
| **fd** | find | Rust | 10-20x faster, sane defaults |
| **bfs** | find | C | Breadth-first search |

### Search

| Tool | Replaces | Language | Key Feature |
|------|----------|----------|-------------|
| **ripgrep (rg)** | grep | Rust | Fastest, respects gitignore |
| **ag** | grep | C | Silver Searcher, mature |
| **ugrep** | grep | C++ | Interactive TUI, boolean queries |
| **ast-grep** | grep | Rust | AST-aware structural search |

### Disk Usage

| Tool | Replaces | Language | Key Feature |
|------|----------|----------|-------------|
| **dust** | du | Rust | Visual bars, largest files |
| **duf** | df | Go | Beautiful output, JSON |
| **ncdu** | du | C | Interactive TUI |
| **gdu** | du | Go | Fastest on SSDs |

### Process Viewing

| Tool | Replaces | Language | Key Feature |
|------|----------|----------|-------------|
| **procs** | ps | Rust | Colorized, docker-aware |
| **bottom (btm)** | top | Rust | Graphs, vim keys |
| **btop** | top | C++ | Feature-rich, mouse |
| **zenith** | top | Rust | GPU monitoring |

### System Info

| Tool | Replaces | Language | Key Feature |
|------|----------|----------|-------------|
| **fastfetch** | neofetch | C | 10x faster, accurate |
| **macchina** | neofetch | Rust | Git info, theming |

### Navigation

| Tool | Replaces | Language | Key Feature |
|------|----------|----------|-------------|
| **zoxide** | cd | Rust | Frecency, fzf integration |
| **autojump** | cd | Python | Learns directories |

### Diff Tools

| Tool | Replaces | Language | Key Feature |
|------|----------|----------|-------------|
| **delta** | diff | Rust | Syntax highlighting, git integration |
| **difftastic** | diff | Rust | AST-aware structural diff |

### Text Processing

| Tool | Replaces | Language | Key Feature |
|------|----------|----------|-------------|
| **sd** | sed | Rust | Simple syntax, no escaping |
| **choose** | cut/awk | Rust | Python-like slicing |
| **hck** | cut | Rust | Header-based selection |

### HTTP Clients

| Tool | Replaces | Language | Key Feature |
|------|----------|----------|-------------|
| **xh** | curl/httpie | Rust | Fast HTTPie clone |
| **curlie** | curl | Go | HTTPie output, curl power |
| **hurl** | curl | Rust | Declarative API testing |

### Benchmarking

| Tool | Replaces | Language | Key Feature |
|------|----------|----------|-------------|
| **hyperfine** | time | Rust | Statistical analysis, JSON export |

### Other Modern Tools

| Tool | Replaces | Language | Key Feature |
|------|----------|----------|-------------|
| **tokei** | cloc | Rust | Fast code counting |
| **bandwhich** | nethogs | Rust | Bandwidth by process |
| **dog** | dig | Rust | DNS with JSON output |
| **grex** | - | Rust | Generate regex from examples |
| **hexyl** | xxd | Rust | Colorized hex viewer |
| **tealdeer (tldr)** | man | Rust | Practical examples |

---

## Category 3: Data Processing Tools

### CSV/TSV Tools

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **csvkit** | CSV utilities suite | Full CSV processing |
| **xsv** | Fast CSV toolkit | Large file handling |
| **miller (mlr)** | Multi-format processor | CSV/JSON/tabular |
| **q** | SQL on CSV | Query CSV with SQL |
| **textql** | SQL on text | Similar to q |
| **csvq** | CSV query tool | SQL syntax |
| **qsv** | Ultra-fast CSV | Performance-critical |

### JSON Tools

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **jq** | JSON query/transform | Universal JSON processing |
| **gojq** | jq in Go | YAML support |
| **jaq** | jq in Rust | Faster |
| **gron** | Flatten JSON for grep | Searchable JSON |
| **jo** | Create JSON from args | Safe JSON construction |
| **jc** | CLI output to JSON | Modernize Unix tools |
| **fx** | Interactive JSON | JavaScript queries |
| **jless** | JSON viewer | Interactive exploration |
| **jd** | JSON diff | Semantic comparison |

### YAML Tools

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **yq** | YAML processor | jq for YAML |
| **dyff** | YAML diff | Semantic comparison |
| **yamllint** | YAML linter | Validation |

### XML/HTML Tools

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **xmlstarlet** | XML toolkit | Query/transform XML |
| **xidel** | XML/HTML query | XPath/CSS selectors |
| **pup** | HTML processor | jq for HTML |
| **htmlq** | HTML query | CSS selectors |
| **hxselect** | HTML/XML select | Element extraction |

### Log Processing

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **lnav** | Log navigator | Log analysis |
| **angle-grinder** | Log aggregation | SQL-like queries |
| **goaccess** | Web log analyzer | Traffic analysis |

### Format Converters

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **pandoc** | Universal converter | Document conversion |
| **unoconv** | Office formats | Convert docs |
| **pdftotext** | PDF to text | Text extraction |
| **wkhtmltopdf** | HTML to PDF | Report generation |

### Database Tools

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **sqlite3** | SQLite CLI | Embedded database |
| **sqlite-utils** | JSON/CSV to SQLite | File to SQL |
| **usql** | Universal SQL client | Multi-database |
| **pgcli** | PostgreSQL CLI | Enhanced psql |
| **mycli** | MySQL CLI | Enhanced mysql |
| **litecli** | SQLite CLI | Enhanced sqlite |

---

## Category 4: Developer Productivity Tools

### Code Scaffolding

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **cookiecutter** | Project templates | Project generation |
| **yeoman** | Scaffolding | Generator ecosystem |
| **hygen** | Code generator | Component scaffolding |
| **plop** | Micro-generator | Consistent patterns |

### Code Transformation

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **comby** | Structural search/replace | Large-scale refactoring |
| **codemod** | Facebook codemod | API migration |
| **jscodeshift** | JS/TS codemod | AST transforms |
| **libcst** | Python CST | Python refactoring |
| **semgrep** | Pattern-based lint/fix | Security fixes |

### Code Generation

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **quicktype** | Types from JSON/Schema | API client generation |
| **openapi-generator** | OpenAPI code gen | SDK creation |
| **protoc** | Protocol Buffers | gRPC services |
| **sqlc** | SQL to Go | Type-safe database |

### Documentation

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **sphinx** | Python docs | API documentation |
| **mkdocs** | Markdown site | Project docs |
| **doxygen** | Multi-language docs | C/C++ docs |
| **typedoc** | TypeScript docs | TS API docs |
| **pdoc** | Python docs | Quick docs |

### API Tools

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **spectral** | API linting | Spec validation |
| **prism** | OpenAPI mock | API testing |
| **swagger-cli** | Swagger tools | Spec validation |

### Dependency Analysis

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **pipdeptree** | Python deps | Dependency tree |
| **depcheck** | JS deps | Unused deps |
| **cargo-tree** | Rust deps | Dependency analysis |
| **pip-tools** | Python pinning | Reproducible builds |

### Code Metrics

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **cloc** | Line counting | Codebase analysis |
| **tokei** | Fast counting | Project statistics |
| **scc** | Complexity | COCOMO estimates |

### Snippet Managers

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **tldr** | Practical examples | Command reference |
| **navi** | Interactive cheatsheets | Command lookup |
| **pet** | Snippet manager | Command storage |
| **cheat** | Cheatsheets | Custom examples |

### Build Tools

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **just** | Task runner | Simple automation |
| **make** | Build system | Universal |
| **task** | YAML tasks | Modern make |
| **invoke** | Python tasks | Pythonic |

### Formatting

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **prettier** | JS/CSS/HTML | Web formatting |
| **black** | Python | Python formatting |
| **gofmt** | Go | Go formatting |
| **rustfmt** | Rust | Rust formatting |
| **shfmt** | Shell | Shell formatting |

### Git Helpers

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **gh** | GitHub CLI | GitHub automation |
| **glab** | GitLab CLI | GitLab automation |
| **pre-commit** | Git hooks | Quality gates |
| **git-extras** | Git utilities | Extended commands |
| **lazygit** | Git TUI | Visual git |
| **gitui** | Git TUI (Rust) | Fast git UI |

---

## Category 5: System Debugging Tools

### Process Inspection

| Tool | Description | Agent Use Case | JSON |
|------|-------------|----------------|------|
| **strace** | Syscall tracing | Debug failures | - |
| **ltrace** | Library calls | API debugging | - |
| **lsof** | Open files | Resource debugging | - |
| **fuser** | File users | Find processes using files | - |
| **pstree** | Process tree | Hierarchy view | - |

### Performance

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **perf** | Linux profiler | CPU profiling |
| **bpftrace** | eBPF tracing | Advanced debugging |
| **py-spy** | Python profiler | Python performance |
| **valgrind** | Memory checker | Memory debugging |

### File Monitoring

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **inotifywait** | File events | Watch for changes |
| **fswatch** | Cross-platform watch | File monitoring |
| **watchman** | Facebook watcher | Large codebase |
| **entr** | Run on change | Auto-rebuild |

### Network Debugging

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **tcpdump** | Packet capture | Network debugging |
| **tshark** | Wireshark CLI | Protocol analysis |
| **ngrep** | Network grep | Pattern in traffic |
| **mitmproxy** | HTTPS proxy | API debugging |
| **nc (netcat)** | TCP/UDP tool | Connectivity testing |
| **curl** | HTTP client | API testing |
| **dig** | DNS lookup | DNS debugging |
| **nmap** | Port scanning | Service discovery |

### Resource Monitoring

| Tool | Description | Agent Use Case | JSON |
|------|-------------|----------------|------|
| **vmstat** | Memory stats | System health | - |
| **iostat** | I/O stats | Disk performance | `-o JSON` |
| **mpstat** | CPU stats | CPU monitoring | `-o JSON` |
| **sar** | Historical stats | Performance history | `-j JSON` |
| **glances** | All-in-one | Comprehensive | `--export json` |

### Container Debugging

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **dive** | Image explorer | Optimize images |
| **ctop** | Container top | Resource usage |
| **docker stats** | Container stats | Monitoring |
| **crictl** | CRI CLI | Runtime debugging |
| **nsenter** | Namespace entry | Container access |

### Log Analysis

| Tool | Description | Agent Use Case | JSON |
|------|-------------|----------------|------|
| **journalctl** | Systemd journal | Service logs | `-o json` |
| **dmesg** | Kernel messages | Hardware issues | `--json` |
| **ausearch** | Audit logs | Security | `--format json` |

### Hardware Info

| Tool | Description | Agent Use Case | JSON |
|------|-------------|----------------|------|
| **lshw** | Hardware lister | Inventory | `-json` |
| **lspci** | PCI devices | GPU/NIC detection | - |
| **lsusb** | USB devices | Device detection | - |
| **sensors** | Hardware sensors | Temperature | `-j` |
| **nvidia-smi** | GPU info | GPU monitoring | - |

---

## Category 6: File & Text Manipulation

### In-Place Editing

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **sponge** | Safe in-place | Prevent race conditions |
| **ed** | Line editor | Scriptable editing |
| **patch** | Apply diffs | Code patching |

### Encoding/Decoding

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **base64** | Base64 | Binary-to-text |
| **xxd** | Hex dump | Binary inspection |
| **iconv** | Charset conversion | Fix encoding |
| **recode** | Charset with transforms | HTML entities |
| **uuencode** | Legacy encoding | Compatibility |

### Hashing

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **md5sum** | MD5 hash | Quick checksums |
| **sha256sum** | SHA-256 | Secure hashes |
| **b2sum** | BLAKE2 | Fast secure hash |
| **rhash** | Multi-hash | Multiple algorithms |

### Compression

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **gzip** | Standard | Universal |
| **zstd** | Modern | Best speed/ratio |
| **lz4** | Fastest | Real-time |
| **xz** | Best ratio | Archival |
| **pigz** | Parallel gzip | Multi-core |

### Archiving

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **tar** | Archive | Standard |
| **zip/unzip** | ZIP | Windows compat |
| **7z** | Multi-format | Best compression |
| **cpio** | Copy archive | RPM, initramfs |

### Text Tools

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **tr** | Character translate | Quick transforms |
| **cut** | Field extraction | Simple columns |
| **paste** | Merge files | Column combining |
| **join** | SQL-like join | Data merging |
| **comm** | Compare files | Set operations |
| **sort** | Sort lines | Ordering |
| **uniq** | Deduplicate | Remove duplicates |
| **shuf** | Randomize | Shuffle lines |
| **nl** | Number lines | Line numbering |
| **rev** | Reverse chars | Text manipulation |
| **tac** | Reverse lines | Bottom-to-top |

### Splitting

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **split** | Split by size/lines | Chunk files |
| **csplit** | Split by pattern | Context splitting |

### Templating

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **envsubst** | Env var substitution | Config templates |
| **j2cli** | Jinja2 CLI | Complex templates |
| **mo** | Bash Mustache | Simple templates |

### Diffing

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **diff** | Line comparison | File comparison |
| **diff3** | Three-way | Merge conflicts |
| **sdiff** | Side-by-side | Visual diff |
| **wiggle** | Apply rejected patches | Salvage patches |

### Statistics

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **datamash** | Tabular stats | Quick statistics |
| **tsort** | Topological sort | Dependency ordering |

---

## Category 7: Cloud & Infrastructure Tools

### Cloud CLIs

| Tool | Cloud | Agent Use Case | JSON |
|------|-------|----------------|------|
| **aws** | AWS | Full AWS access | `--output json` |
| **gcloud** | GCP | Full GCP access | `--format json` |
| **az** | Azure | Full Azure access | `--output json` |
| **doctl** | DigitalOcean | DO management | `--output json` |
| **flyctl** | Fly.io | Fly deployment | `--json` |

### Kubernetes

| Tool | Description | Agent Use Case | JSON |
|------|-------------|----------------|------|
| **kubectl** | K8s CLI | Cluster management | `-o json` |
| **helm** | Package manager | Chart deployment | `-o json` |
| **kustomize** | Config customization | Overlays | - |
| **stern** | Multi-pod logs | Log aggregation | `--output json` |
| **kubectx/kubens** | Context switching | Cluster navigation | - |

### Infrastructure as Code

| Tool | Description | Agent Use Case | JSON |
|------|-------------|----------------|------|
| **terraform** | IaC | Infrastructure | `-json` |
| **pulumi** | IaC (code-first) | Infrastructure | `--json` |
| **ansible** | Config management | Automation | callback plugins |
| **cdktf** | Terraform CDK | Code-based IaC | - |

### Secrets Management

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **sops** | Encrypted secrets | Version-controlled secrets |
| **age** | File encryption | Simple encryption |
| **vault** | HashiCorp Vault | Enterprise secrets |
| **pass** | Password store | GPG-based secrets |
| **gopass** | Team passwords | Shared secrets |
| **op** | 1Password CLI | 1Password automation |

### Certificates

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **mkcert** | Local TLS | Dev certificates |
| **certbot** | Let's Encrypt | Production TLS |
| **step** | PKI toolkit | Certificate management |
| **cfssl** | CloudFlare PKI | CA operations |
| **openssl** | Crypto toolkit | General crypto |

### SSH Tools

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **ssh** | Secure shell | Remote commands |
| **scp** | Secure copy | File transfer |
| **rsync** | Incremental sync | Efficient sync |
| **sshfs** | SSH filesystem | Remote mount |
| **mosh** | Mobile shell | Unreliable connections |
| **sshuttle** | VPN over SSH | Network access |
| **autossh** | Persistent tunnels | Reliable tunnels |

### Tunneling

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **ngrok** | Public URLs | Webhook testing |
| **localtunnel** | Simple tunnels | Quick sharing |
| **bore** | Minimal tunnels | Lightweight |
| **cloudflared** | Cloudflare tunnels | Zero-trust access |

### Time/Date

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **date** | Date/time | Timestamp generation |
| **dateutils** | Date arithmetic | Date calculations |

### Math

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **bc** | Calculator | Arbitrary precision |
| **dc** | RPN calculator | Stack-based math |
| **qalc** | Unit-aware calc | Conversions |
| **units** | Unit conversion | Dimensional analysis |
| **numfmt** | Human numbers | SI/IEC formatting |

### Random/UUID

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **uuidgen** | Generate UUIDs | Unique identifiers |
| **openssl rand** | Random data | Tokens, keys |

### Notifications

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **notify-send** | Linux notifications | Desktop alerts |
| **terminal-notifier** | macOS notifications | Desktop alerts |
| **ntfy** | Push notifications | Cross-device |

### Clipboard

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **xclip** | X11 clipboard | Copy/paste |
| **xsel** | X11 clipboard | Alternative |
| **pbcopy/pbpaste** | macOS clipboard | Copy/paste |
| **wl-copy/wl-paste** | Wayland clipboard | Modern Linux |

### Terminal Multiplexing

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **tmux** | Multiplexer | Session management |
| **screen** | Multiplexer | Legacy sessions |
| **zellij** | Modern mux | Plugin system |

### Interactive Utilities

| Tool | Description | Agent Use Case |
|------|-------------|----------------|
| **fzf** | Fuzzy finder | Interactive selection |
| **gum** | Shell glamour | Script UX |

---

## Summary Statistics

| Category | Tool Count |
|----------|------------|
| Unix Tool Suites | ~150 |
| Modern CLI Rewrites | ~50 |
| Data Processing | ~40 |
| Developer Productivity | ~60 |
| System Debugging | ~50 |
| File/Text Manipulation | ~45 |
| Cloud/Infrastructure | ~50 |
| **Total** | **~445** |

---

## Next Steps

1. **Deduplication** - Identify overlapping tools and pick winners
2. **Prioritization** - Rank by agent value vs complexity
3. **Recipe Development** - Create embedded recipes for complex tools
4. **Skill Structure** - Design hierarchical lazy-loading skill system
5. **Testing** - Validate tool recommendations empirically

---

*This exploration document serves as the foundation for building a lean, effective agent toolkit.*
