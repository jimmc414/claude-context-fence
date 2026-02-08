# Agent Toolkit: Complete Tool Inventory

**Status:** Consolidated from research  
**Date:** 2026-02-03  
**Source:** Yegge analysis + extended research + community contributions

---

## Overview

Total tools cataloged: **147**

| Category | Count |
|----------|-------|
| Text Search & Processing | 14 |
| JSON/Structured Data | 10 |
| Database & Data | 14 |
| Version Control | 4 |
| Schema & Validation | 9 |
| Task Runners & Build | 10 |
| HTTP & Networking | 14 |
| Containers & Environments | 12 |
| Static Analysis & Linting | 12 |
| Workflow & Orchestration | 8 |
| Media Processing | 8 |
| Document Generation | 7 |
| Security & Secrets | 10 |
| Testing & Mocking | 9 |
| Unix Primitives | 22 |
| Debugging & Observability | 12 |
| Math & Calculation | 6 |

---

## Text Search & Processing

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| ripgrep (rg) | Fast text search | Excellent | Tier 1 | Respects .gitignore, sane defaults |
| grep | Universal text search | Good | Tier 1 | Everywhere, slower |
| ast-grep (sg) | Structural code search | Moderate | Tier 2 | Syntax-tree aware, underexplored |
| ag (silver searcher) | Fast text search | Good | Tier 3 | Declining mindshare |
| ugrep | grep-compatible search | Good | Tier 3 | Very fast |
| awk | Text processing language | Good | Tier 2 | Substrate efficiency champion |
| sed | Stream editor | Moderate | Tier 2 | Agents hallucinate syntax |
| tr | Character translation | Excellent | Tier 2 | Simple, reliable |
| cut | Column extraction | Excellent | Tier 2 | Simpler than awk for fields |
| paste | Merge lines/files | Good | Tier 3 | Combining parallel outputs |
| sort | Sort lines | Excellent | Tier 1 | Fundamental primitive |
| uniq | Deduplicate lines | Excellent | Tier 1 | Requires sorted input |
| wc | Count lines/words/chars | Excellent | Tier 1 | Agents should never count manually |
| head/tail | First/last lines | Excellent | Tier 1 | Basic but essential |

---

## JSON/Structured Data

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| jq | JSON query/transform | Moderate | Tier 1 | Powerful but hallucination-prone |
| jc | CLI output → JSON | Excellent | Tier 1 | **Critical discovery**: modernizes 100+ Unix tools |
| jo | Create JSON from args | Good | Tier 2 | Safer than string formatting |
| gron | Flatten JSON for grep | Good | Tier 2 | Makes JSON greppable |
| dasel | Multi-format query | Good | Tier 2 | JSON/YAML/TOML/XML |
| miller (mlr) | Tabular data processing | Good | Tier 2 | Record-oriented, multi-format |
| gojq | jq + YAML support | Moderate | Tier 3 | Drop-in jq replacement |
| jaq | Fast jq clone | Moderate | Tier 3 | Rust, faster |
| yq | YAML processor | Moderate | Tier 2 | jq for YAML |
| xsv | CSV toolkit | Good | Tier 2 | Fast CSV operations |

---

## Database & Data

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| sqlite3 | Embedded SQL database | Excellent | Tier 1 | Universal, agents know it cold |
| DuckDB | Analytical queries | Excellent | Tier 1 | Query CSV/Parquet directly |
| PostgreSQL | Relational database | Excellent | Tier 2 | Full-featured, massive ecosystem |
| sqlite-utils | JSON/CSV → SQLite CLI | Excellent | Tier 1 | Turns file processing into SQL |
| datasette | SQLite → JSON API | Good | Tier 2 | Instant read-only API |
| usql | Universal SQL client | Moderate | Tier 3 | One CLI for all databases |
| Redis | In-memory data store | Good | Tier 2 | Caching, queues, pub/sub |
| Dragonfly | Redis alternative | Good | Tier 3 | Faster drop-in |
| Valkey | Redis fork | Good | Tier 3 | Community fork |
| libsql | SQLite fork | Good | Tier 3 | Extended features |
| Dolt | Git for databases | Good | Tier 2 | Version-controlled data |
| Neon | Serverless Postgres | Good | Tier 3 | Branching, autoscaling |
| Meilisearch | Full-text search | Excellent | Tier 2 | Simple, fast |
| Typesense | Full-text search | Excellent | Tier 2 | Similar to Meilisearch |

---

## Version Control

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| git | Version control | Excellent | Tier 1 | Universal, deeply embedded |
| jj (jujutsu) | Git alternative | Moderate | Tier 3 | Better model, low awareness |
| sapling | Git alternative | Moderate | Tier 3 | Meta's tool |
| git-delta | Better git diffs | Good | Tier 2 | Syntax highlighting |

---

## Schema & Validation

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| Pydantic | Python validation | Excellent | Tier 1 | Standard for Python |
| Zod | TypeScript validation | Excellent | Tier 1 | Standard for TS |
| JSON Schema | Language-agnostic schemas | Good | Tier 2 | Universal but verbose |
| ajv | JSON Schema validator (JS) | Good | Tier 2 | Fastest validator |
| typebox | JSON Schema + TS types | Good | Tier 3 | Generates both |
| cerberus | Python validation | Moderate | Tier 3 | Simpler than Pydantic |
| marshmallow | Python serialization | Good | Tier 3 | Older, still solid |
| shellcheck | Shell script linter | Excellent | Tier 1 | **Critical**: agents write bad bash |
| file | Identify file types | Good | Tier 2 | Magic bytes, not extensions |

---

## Task Runners & Build

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| just | Simple task runner | Excellent | Tier 1 | No tab issues, simple syntax |
| make | Build automation | Moderate | Tier 2 | Universal but arcane |
| task (go-task) | YAML task runner | Good | Tier 2 | Cross-platform |
| invoke | Python task runner | Good | Tier 3 | Pythonic |
| nx | Monorepo builds | Moderate | Tier 3 | Complex setup |
| turborepo | JS/TS monorepo | Moderate | Tier 3 | Vercel ecosystem |
| bazel | Hermetic builds | Poor | Avoid | Steep learning curve |
| cargo | Rust package manager | Excellent | Tier 1 | Model for others |
| poetry | Python packaging | Good | Tier 2 | Deterministic deps |
| pnpm | Node package manager | Good | Tier 2 | Faster, stricter |

---

## HTTP & Networking

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| curl | HTTP client | Good | Tier 1 | Universal, arcane flags |
| httpie | Human-readable HTTP | Moderate | Tier 2 | Cleaner syntax |
| hurl | HTTP request sequences | Good | Tier 2 | File-based, testable |
| xh | Fast httpie-like | Moderate | Tier 3 | Rust rewrite |
| wget | File downloads | Moderate | Tier 2 | Better for recursive |
| nc (netcat) | Raw TCP/UDP | Good | Tier 2 | Connectivity debugging |
| socat | Advanced netcat | Moderate | Tier 3 | Protocol conversion |
| nmap | Port scanning | Moderate | Tier 3 | Service discovery |
| dig | DNS queries | Good | Tier 2 | DNS debugging |
| mtr | Network path analysis | Moderate | Tier 3 | Routing issues |
| ngrok | Public URLs for local | Good | Tier 2 | Webhook testing |
| mitmproxy | HTTP interception | Good | Tier 3 | Debug actual traffic |
| curlie | curl + httpie output | Good | Tier 3 | Best of both |
| rsync | Efficient file sync | Good | Tier 2 | Only transfer changes |

---

## Containers & Environments

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| docker | Containerization | Good | Tier 1 | Standard interface |
| podman | Rootless containers | Good | Tier 2 | Docker-compatible |
| colima | Docker runtime | Good | Tier 2 | macOS/Linux |
| nerdctl | containerd CLI | Moderate | Tier 3 | containerd-native |
| direnv | Per-directory env vars | Good | Tier 2 | Automatic switching |
| asdf | Multi-version tools | Good | Tier 2 | Node/Python/Ruby versions |
| mise | asdf alternative | Good | Tier 2 | Faster, Rust-based |
| dotenv | Load .env files | Good | Tier 2 | Standard pattern |
| env/printenv | Show environment | Excellent | Tier 1 | Verify assumptions |
| which/type | Find commands | Excellent | Tier 1 | Verify tool locations |
| ldd | Library dependencies | Moderate | Tier 3 | Debug link errors |
| virtualenv/venv | Python isolation | Good | Tier 2 | Project isolation |

---

## Static Analysis & Linting

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| ruff | Python linting | Excellent | Tier 1 | Fast, comprehensive |
| eslint | JS/TS linting | Good | Tier 1 | Standard, configurable |
| biome | JS/TS linting | Good | Tier 2 | Rust-based, faster |
| oxlint | Fast JS linting | Good | Tier 3 | Fastest, fewer rules |
| mypy | Python type checking | Good | Tier 1 | Strict, catches bugs |
| pyright | Python type checking | Good | Tier 2 | Faster than mypy |
| clippy | Rust linting | Excellent | Tier 1 | Idiomatic suggestions |
| golangci-lint | Go linting | Good | Tier 2 | Aggregates linters |
| semgrep | Security scanning | Good | Tier 2 | Code patterns |
| bandit | Python security | Good | Tier 2 | Python-specific |
| trivy | Vulnerability scanning | Good | Tier 2 | Containers, repos |
| gitleaks | Secret detection | Good | Tier 2 | Leaked credentials |

---

## Workflow & Orchestration

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| Temporal | Durable execution | Moderate | Tier 2 | Most powerful |
| Inngest | Event-driven workflows | Good | Tier 2 | Simpler, TypeScript-first |
| Hatchet | Distributed tasks | Good | Tier 3 | Simpler Temporal alt |
| Trigger.dev | Background jobs | Good | Tier 3 | Open source |
| Windmill | Scripts as workflows | Good | Tier 3 | UI-driven, polyglot |
| Prefect | Data pipelines | Moderate | Tier 3 | Python-focused |
| Dagster | Data orchestration | Moderate | Tier 3 | Asset-focused |
| Airflow | Legacy pipelines | Poor | Avoid | Terrible agent UX |

---

## Media Processing

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| ffmpeg | Video/audio processing | Poor | Tier 2 | Powerful, arcane syntax |
| yt-dlp | Video downloads | Good | Tier 2 | Media fetching |
| ImageMagick | Image processing | Moderate | Tier 2 | Powerful, confusing |
| libvips | Fast image processing | Good | Tier 3 | Lower memory |
| sharp | Node.js images | Good | Tier 2 | libvips bindings |
| Pillow | Python images | Excellent | Tier 1 | Agents know it well |
| GraphicsMagick | ImageMagick fork | Moderate | Tier 3 | More stable |
| SoX | Audio processing | Moderate | Tier 3 | Swiss army knife |

---

## Document Generation

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| pandoc | Universal converter | Good | Tier 1 | Markdown → anything |
| mermaid-cli | Diagram rendering | Good | Tier 2 | Mermaid → PNG/SVG |
| graphviz (dot) | Graph rendering | Good | Tier 2 | Classic directed graphs |
| plantuml | UML diagrams | Moderate | Tier 3 | Sequence, class diagrams |
| d2 | Modern diagrams | Good | Tier 2 | Better syntax |
| weasyprint | HTML → PDF | Good | Tier 3 | CSS-based PDF |
| asciidoctor | AsciiDoc processing | Moderate | Tier 3 | Technical docs |

---

## Security & Secrets

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| sops | Encrypted secrets | Good | Tier 2 | Secrets in version control |
| age | File encryption | Good | Tier 2 | Modern, minimal |
| pass | Password store | Moderate | Tier 3 | GPG-based |
| vault | Secrets management | Moderate | Tier 3 | Enterprise |
| mkcert | Local TLS certs | Good | Tier 2 | Trusted local certs |
| certbot | Let's Encrypt | Good | Tier 3 | Production TLS |
| step | Smallstep CLI | Moderate | Tier 3 | Advanced certs |
| openssl | TLS debugging | Moderate | Tier 2 | Swiss army knife |
| ssh-keygen | SSH keys | Good | Tier 2 | Key generation |
| gpg | Encryption/signing | Moderate | Tier 3 | Complex but standard |

---

## Testing & Mocking

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| mockserver | HTTP mock server | Good | Tier 2 | Test API integrations |
| wiremock | HTTP stubbing | Good | Tier 3 | Record and replay |
| json-server | REST API from JSON | Good | Tier 2 | Instant mock backend |
| prism | OpenAPI mock server | Good | Tier 3 | Generate from spec |
| diff | File comparison | Excellent | Tier 1 | Canonical |
| difftastic (difft) | Structural diff | Good | Tier 2 | Syntax-aware |
| jd | JSON diff | Good | Tier 2 | Semantic comparison |
| dyff | YAML diff | Good | Tier 3 | Semantic YAML |
| hypothesis | Property testing (Python) | Good | Tier 2 | Auto-generate edge cases |

---

## Unix Primitives & Safety

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| timeout | Limit execution time | Excellent | Tier 1 | **Critical**: prevents infinite hangs |
| yes | Auto-confirm prompts | Moderate | Tier 3 | **Dangerous**: use tool flags instead |
| sponge | Safe in-place edit | Excellent | Tier 1 | Prevents read/write race |
| watch | Repeated execution | Good | Tier 2 | Monitor changing state |
| chronic | Output only on failure | Excellent | Tier 1 | Prevents context flooding |
| expect | Script interactive tools | Good | Tier 2 | Solves prompt-blockers |
| xargs | Build commands from stdin | Good | Tier 2 | Parallelize operations |
| parallel | GNU Parallel | Good | Tier 2 | More powerful than xargs |
| tee | Split output streams | Good | Tier 2 | Save while passing on |
| pv | Pipe viewer | Moderate | Tier 3 | Progress monitoring |
| time | Measure execution | Good | Tier 2 | Performance awareness |
| ts | Add timestamps | Good | Tier 3 | Timestamp any output |
| ifne | Conditional execution | Moderate | Tier 3 | Run only if stdin non-empty |
| tree | Directory visualization | Good | Tier 2 | Cheaper than recursive ls |
| fd | Fast file finding | Good | Tier 2 | Better UX than find |
| dust/du | Disk usage | Good | Tier 2 | Space awareness |
| fdupes | Duplicate detection | Moderate | Tier 3 | Find duplicate files |
| base64 | Encoding | Excellent | Tier 1 | Binary-to-text |
| xxd | Hex dumps | Moderate | Tier 3 | Debug binary |
| tar | Archiving | Good | Tier 2 | Standard archiving |
| zstd | Compression | Good | Tier 2 | Faster than gzip |
| rclone | Cloud sync | Good | Tier 2 | rsync for cloud storage |

---

## Debugging & Observability

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| strace | Syscall tracing | Good | Tier 2 | "The Why Tool" for silent failures |
| lsof | Open files/ports | Good | Tier 2 | Debug "in use" errors |
| hyperfine | Benchmarking | Moderate | Tier 3 | A/B test performance |
| htop/top | Process monitoring | Poor | Avoid | TUI, not agent-friendly |
| ps | Process listing | Good | Tier 2 | Use with jc for JSON |
| free | Memory info | Good | Tier 2 | Use with jc for JSON |
| df | Disk space | Good | Tier 2 | Use with jc for JSON |
| ss | Socket statistics | Good | Tier 2 | Modern netstat |
| dmesg | Kernel messages | Moderate | Tier 3 | Hardware/driver issues |
| journalctl | Systemd logs | Moderate | Tier 2 | Log querying |
| lnav | Log navigator | Moderate | Tier 3 | Has CLI mode |
| angle-grinder | Log aggregation | Good | Tier 3 | SQL-like log queries |

---

## Math & Calculation

| Tool | Purpose | Agent UX | Priority | Notes |
|------|---------|----------|----------|-------|
| bc | Arbitrary precision math | Excellent | Tier 1 | Cheaper than LLM math |
| qalc | Unit-aware calculator | Good | Tier 2 | Handles conversions |
| units | Unit conversion | Good | Tier 2 | Comprehensive database |
| dateutils | Date arithmetic | Good | Tier 2 | Time zones, leap years |
| shuf | Random numbers | Good | Tier 2 | True entropy from OS |
| uuidgen | UUID generation | Good | Tier 2 | Cryptographically valid |

---

## Summary Statistics

### By Priority Tier

| Tier | Description | Count |
|------|-------------|-------|
| Tier 1 | Core agent toolkit | 32 |
| Tier 2 | Common workflow tools | 67 |
| Tier 3 | Domain-specific/niche | 42 |
| Avoid | Anti-patterns | 6 |

### By Survival Lever (Primary)

| Lever | Count | Examples |
|-------|-------|----------|
| Substrate Efficiency (2) | 38 | bc, ripgrep, DuckDB |
| Friction Reduction (5) | 52 | jc, sponge, timeout |
| Insight Compression (1) | 24 | git, Temporal, pandoc |
| Broad Utility (3) | 18 | sqlite, jq, curl |
| Awareness (4) | 8 | (most tools need this) |
| Human Coefficient (6) | 7 | (specialized domain) |

### Critical Discoveries

Tools that emerged as more important than initially recognized:

| Tool | Why Critical |
|------|--------------|
| **jc** | Modernizes entire Unix toolkit to JSON |
| **timeout** | Prevents infinite hangs (agents lack time sense) |
| **sponge** | Prevents read/write race condition |
| **chronic** | Prevents context flooding |
| **shellcheck** | Agents write bad bash constantly |
| **sqlite-utils** | Turns file processing into SQL |
| **expect** | Solves interactive tool anti-pattern |

### Minimum Viable Toolkit

If context-constrained, these 10 tools provide maximum coverage:

```
ripgrep, jq, jc, sqlite3, git, curl, timeout, sponge, shellcheck, just
```

---

## Appendix: Tools Explicitly Marked "Avoid"

| Tool | Reason |
|------|--------|
| Bazel | Steep learning curve, agents struggle |
| Airflow | Terrible agent UX (DAG files) |
| htop/top | TUI, agents can't interact |
| GUI-only tools | No programmatic access |
| Tools with mandatory prompts | Use `--yes` flags or `expect` |
| "Smart" auto-fixing tools | Obscures feedback agents need |

---

*This inventory will be used to prioritize skill development and toolkit configuration.*
