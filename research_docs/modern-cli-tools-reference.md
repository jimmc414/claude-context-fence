# Modern CLI Tool Replacements Reference Guide

A comprehensive guide to modern command-line tools written in Rust, Go, and other modern languages that improve upon classic Unix utilities. This document covers key improvements, agent-relevant features, and potential drawbacks for each tool.

---

## Table of Contents

1. [File Listing](#1-file-listing)
2. [File Viewing](#2-file-viewing)
3. [File Finding](#3-file-finding)
4. [Search Tools](#4-search-tools)
5. [Disk Usage](#5-disk-usage)
6. [Process Viewing](#6-process-viewing)
7. [System Information](#7-system-information)
8. [Directory Navigation](#8-directory-navigation)
9. [Diff Tools](#9-diff-tools)
10. [HTTP Clients](#10-http-clients)
11. [JSON Tools](#11-json-tools)
12. [Git Tools](#12-git-tools)
13. [Text Processing](#13-text-processing)
14. [Benchmarking](#14-benchmarking)
15. [Other Modern Rewrites](#15-other-modern-rewrites)

---

## 1. File Listing

### eza (formerly exa)

| Attribute | Details |
|-----------|---------|
| **Replaces** | `ls` |
| **Language** | Rust |
| **Repository** | [github.com/eza-community/eza](https://github.com/eza-community/eza) |
| **Status** | Actively maintained (fork of discontinued exa) |

**Key Improvements:**
- Colorized output by file type, permissions, and size by default
- Git status integration showing modified/staged files
- Tree view built-in (`--tree` or `-T`)
- Extended attributes and metadata display
- Human-readable file sizes
- Icons support (with Nerd Fonts)
- Hyperlink support for terminal emulators
- `theme.yml` configuration file for customization

**Agent-Relevant Features:**
- JSON output mode for structured data (`--json`)
- Git-aware (respects `.gitignore`)
- Consistent, parseable output format
- Icons provide visual hierarchy for faster scanning

**Drawbacks:**
- Requires Nerd Fonts for icon display
- Additional dependency vs standard `ls`
- Configuration can be complex for full customization

**Ubuntu/Debian Note:** Available via apt or cargo install.

---

### lsd (LSDeluxe)

| Attribute | Details |
|-----------|---------|
| **Replaces** | `ls` |
| **Language** | Rust |
| **Repository** | [github.com/lsd-rs/lsd](https://github.com/lsd-rs/lsd) |
| **Status** | Actively maintained |

**Key Improvements:**
- Rich colors and Nerd Font icons
- Tree view support
- Git status indicators
- Customizable via config file
- Cross-platform (Windows, macOS, Linux)
- Sorting by various attributes

**Agent-Relevant Features:**
- Clean, consistent formatting
- Visual hierarchy through colors/icons
- Configuration file for reproducible setups

**Drawbacks:**
- Icon display requires Nerd Fonts
- Slightly slower than native `ls` for simple operations
- No JSON output mode

---

### nat

| Attribute | Details |
|-----------|---------|
| **Replaces** | `ls` |
| **Language** | Rust |
| **Repository** | [github.com/willdoescode/nat](https://github.com/willdoescode/nat) |
| **Status** | Less actively maintained than eza/lsd |

**Key Improvements:**
- Colorful output with useful file information
- Simple, minimal design philosophy
- Fast performance

**Agent-Relevant Features:**
- Lightweight alternative
- Quick directory overview

**Drawbacks:**
- Fewer features than eza or lsd
- Less active development
- Smaller community

---

### broot

| Attribute | Details |
|-----------|---------|
| **Replaces** | `tree`, `ls` (interactive) |
| **Language** | Rust |
| **Repository** | [github.com/Canop/broot](https://github.com/Canop/broot) |
| **Website** | [dystroy.org/broot](https://dystroy.org/broot/) |

**Key Improvements:**
- Interactive, fuzzy-searchable tree view
- Navigate by typing partial directory names
- Split panel support for comparisons
- Preview files without leaving the tree
- Git integration (status, branch info)
- "Whale spotting" mode for disk usage analysis
- Image preview support (Kitty/WezTerm)
- Built-in file operations (move, copy, rm, mkdir)

**Agent-Relevant Features:**
- Fast navigation through large directory structures
- Fuzzy matching reduces keystrokes
- `.gitignore` aware
- Can `cd` into directories from within broot

**Drawbacks:**
- Learning curve for key bindings
- Not a drop-in replacement (interactive TUI)
- Requires terminal that supports advanced features for best experience

---

## 2. File Viewing

### bat

| Attribute | Details |
|-----------|---------|
| **Replaces** | `cat`, `less` |
| **Language** | Rust |
| **Repository** | [github.com/sharkdp/bat](https://github.com/sharkdp/bat) |
| **Ubuntu Command** | `batcat` (package name conflict) |

**Key Improvements:**
- Syntax highlighting for 170+ languages
- Git integration showing modifications in margin
- Automatic paging for long files
- Line numbers by default
- Non-printable character visualization
- Multiple themes available
- Automatic language detection via shebang

**Agent-Relevant Features:**
- `--plain` mode for undecorated output
- Pipe-friendly (plain text when non-interactive)
- `--list-languages` for capability inspection
- Can be used as `MANPAGER`

**Drawbacks:**
- Package named `batcat` on Debian/Ubuntu
- Slower than `cat` for simple operations
- Theme configuration required for some terminals

**Recommended Alias:**
```bash
alias cat='batcat --paging=never'
```

---

### most

| Attribute | Details |
|-----------|---------|
| **Replaces** | `less`, `more` |
| **Language** | C |
| **Status** | Mature, stable |

**Key Improvements:**
- Multiple window support (split views)
- Left-right scrolling for wide content
- Better organized interface
- Can read gzip-compressed files
- More informative status line

**Agent-Relevant Features:**
- Multi-file viewing in split windows
- Truncates long lines instead of wrapping (optional)

**Drawbacks:**
- Not as feature-rich as `less` overall
- Less ubiquitous than `less`

---

### glow

| Attribute | Details |
|-----------|---------|
| **Replaces** | Manual markdown viewing |
| **Language** | Go |
| **Repository** | [github.com/charmbracelet/glow](https://github.com/charmbracelet/glow) |
| **Developer** | Charm |

**Key Improvements:**
- Renders Markdown beautifully in terminal
- Syntax highlighting for code blocks
- TUI mode for browsing multiple files
- Stash/bookmark functionality
- Fetches READMEs from GitHub/GitLab URLs
- Multiple style themes
- Cloud sync with encryption (Charm Cloud)

**Agent-Relevant Features:**
- Read markdown from stdin, file, or URL
- `-w` flag for width control
- Pager integration (`-p`)
- Search within documents (`/`)
- Auto-detects light/dark terminal background

**Drawbacks:**
- Specialized for Markdown only
- Charm Cloud features may not be needed
- Requires terminal with good Unicode support

---

## 3. File Finding

### fd

| Attribute | Details |
|-----------|---------|
| **Replaces** | `find` |
| **Language** | Rust |
| **Repository** | [github.com/sharkdp/fd](https://github.com/sharkdp/fd) |
| **Ubuntu Command** | `fdfind` (package name conflict) |

**Key Improvements:**
- 10-20x faster than `find` due to parallel traversal
- Intuitive syntax: `fd PATTERN` vs `find -iname '*PATTERN*'`
- Smart case sensitivity (case-insensitive by default, switches if uppercase present)
- Respects `.gitignore` by default
- Colorized output by file type
- Regular expressions by default

**Agent-Relevant Features:**
- `-x/--exec` runs commands in parallel
- `-X/--exec-batch` for batch processing
- `--full-path` for path matching
- Global ignore file support (`~/.config/fd/ignore`)
- `-0` for null-separated output (safe for scripts)

**Drawbacks:**
- Named `fdfind` on Debian/Ubuntu
- Different syntax than `find` (not drop-in)
- Some advanced `find` features not replicated

**Example:**
```bash
# Find all Python test files
fdfind "test_.*\.py$"

# Find and delete all .tmp files
fdfind -e tmp -x rm {}
```

---

### bfs

| Attribute | Details |
|-----------|---------|
| **Replaces** | `find` |
| **Language** | C |
| **Repository** | [github.com/tavianator/bfs](https://github.com/tavianator/bfs) |

**Key Improvements:**
- Breadth-first search order (finds shallow results first)
- Compatible with `find` syntax (mostly drop-in)
- `-exclude` operator for skipping subtrees
- Multiple search strategies available
- Often finds target files faster due to BFS

**Agent-Relevant Features:**
- Drop-in `find` replacement
- Results ordered by depth (shallow first)
- Iterative deepening mode for memory efficiency
- Less server timeouts on network filesystems

**Drawbacks:**
- Less well-known than `fd`
- Not as fast as `fd` for regex-based searches
- Fewer modern conveniences (no smart-case, etc.)

---

## 4. Search Tools

### ripgrep (rg)

| Attribute | Details |
|-----------|---------|
| **Replaces** | `grep`, `ag`, `ack` |
| **Language** | Rust |
| **Repository** | [github.com/BurntSushi/ripgrep](https://github.com/BurntSushi/ripgrep) |

**Key Improvements:**
- Order of magnitude faster than GNU grep
- Multithreaded by default
- Respects `.gitignore` automatically
- Full Unicode support without performance penalty
- PCRE2 regex support (optional)
- Compressed file searching
- File type filtering (`-t py`, `-T js`)

**Agent-Relevant Features:**
- `--json` output mode for structured results
- `--vimgrep` format for editor integration
- `--files` to list files that would be searched
- `--stats` for search statistics
- First-class Windows/macOS/Linux support

**Drawbacks:**
- Different default behavior than grep (directory recursive by default)
- `.gitignore` respect can be surprising
- PCRE2 requires feature flag at compile time

**Example:**
```bash
# Search Python files for "def main"
rg "def main" --type py

# JSON output for programmatic use
rg "TODO" --json
```

---

### ag (The Silver Searcher)

| Attribute | Details |
|-----------|---------|
| **Replaces** | `grep`, `ack` |
| **Language** | C |
| **Repository** | [github.com/ggreer/the_silver_searcher](https://github.com/ggreer/the_silver_searcher) |
| **Status** | Maintenance mode (no new features) |

**Key Improvements:**
- 3-5x faster than ack
- Respects `.gitignore`
- Multiline searching by default
- Searches inside binary files (optional)

**Agent-Relevant Features:**
- Mature, widely available
- Similar interface to ack

**Drawbacks:**
- Slower than ripgrep
- Development has stalled
- Some `.gitignore` edge case bugs

---

### ugrep

| Attribute | Details |
|-----------|---------|
| **Replaces** | `grep` |
| **Language** | C++ |
| **Repository** | [github.com/Genivia/ugrep](https://github.com/Genivia/ugrep) |

**Key Improvements:**
- Interactive TUI mode
- Boolean query support (AND, OR, NOT)
- Fuzzy matching
- Archive/compressed file searching
- Output formats: JSON, XML, CSV

**Agent-Relevant Features:**
- `--json` structured output
- Boolean queries for complex searches
- Fuzzy search for typo tolerance

**Drawbacks:**
- Less well-known than ripgrep
- More complex feature set

---

### ast-grep (sg)

| Attribute | Details |
|-----------|---------|
| **Replaces** | Text-based grep for code |
| **Language** | Rust |
| **Repository** | [github.com/ast-grep/ast-grep](https://github.com/ast-grep/ast-grep) |
| **Website** | [ast-grep.github.io](https://ast-grep.github.io/) |

**Key Improvements:**
- AST-based structural code search
- Pattern looks like actual code (isomorphic)
- Language-aware matching
- Built-in code rewriting/refactoring
- YAML-based rule configuration for linting
- Supports many languages via tree-sitter

**Agent-Relevant Features:**
- Meta-variables for flexible matching (`$VAR`, `$$$ARGS`)
- Multiple strictness levels (cst, smart, ast, relaxed, signature)
- jQuery-like API for traversal
- Programmatic use via npm/pip/cargo

**Drawbacks:**
- Learning curve for pattern syntax
- Requires understanding of AST concepts
- Overkill for simple text searches

**Example:**
```bash
# Find all console.log calls with any arguments
sg -p 'console.log($$$ARGS)'

# Find function definitions with specific pattern
sg -p 'function $NAME($$$PARAMS) { $$$BODY }'
```

---

## 5. Disk Usage

### dust

| Attribute | Details |
|-----------|---------|
| **Replaces** | `du` |
| **Language** | Rust |
| **Repository** | [github.com/bootandy/dust](https://github.com/bootandy/dust) |

**Key Improvements:**
- Visual bar chart of disk usage
- Shows largest files within directories
- Sensible defaults (no options needed)
- Color-coded output
- Tree hierarchy visualization

**Agent-Relevant Features:**
- Immediate visual understanding of disk usage
- Shows both directories and large files
- No need to pipe through `sort` or `head`

**Drawbacks:**
- Output optimized for humans, not scripts
- Different interface than `du`

---

### duf

| Attribute | Details |
|-----------|---------|
| **Replaces** | `df` |
| **Language** | Go |
| **Repository** | [github.com/muesli/duf](https://github.com/muesli/duf) |

**Key Improvements:**
- Beautiful, colorized table output
- Shows all mount points clearly
- Groups by device type
- JSON output support
- Progress bars for usage visualization

**Agent-Relevant Features:**
- `--json` for structured output
- Clear visual indicators
- Cross-platform

**Drawbacks:**
- Additional dependency
- May show too much information for simple needs

---

### ncdu

| Attribute | Details |
|-----------|---------|
| **Replaces** | `du` |
| **Language** | C |
| **Repository** | [dev.yorhel.nl/ncdu](https://dev.yorhel.nl/ncdu) |

**Key Improvements:**
- Interactive ncurses interface
- Navigate and delete files interactively
- Works well over SSH on remote servers
- Minimal dependencies

**Agent-Relevant Features:**
- Disk usage analysis on headless servers
- Export/import scan results
- Lightweight

**Drawbacks:**
- Interactive only (no scripting mode)
- Slower than gdu on SSDs

---

### gdu

| Attribute | Details |
|-----------|---------|
| **Replaces** | `du`, `ncdu` |
| **Language** | Go |
| **Repository** | [github.com/dundee/gdu](https://github.com/dundee/gdu) |

**Key Improvements:**
- Much faster than ncdu (especially on SSDs)
- Parallel processing
- Interactive interface like ncdu
- Can analyze apparent vs disk size

**Agent-Relevant Features:**
- Fast scanning (8x faster than ncdu in some benchmarks)
- Non-interactive mode available
- Export results

**Drawbacks:**
- Larger binary than ncdu
- Newer, less battle-tested

---

## 6. Process Viewing

### procs

| Attribute | Details |
|-----------|---------|
| **Replaces** | `ps` |
| **Language** | Rust |
| **Repository** | [github.com/dalance/procs](https://github.com/dalance/procs) |

**Key Improvements:**
- Human-readable, colorized output by default
- Searchable columns
- Tree view of process hierarchy
- Paging support
- Watch mode
- Multi-column sorting

**Agent-Relevant Features:**
- Intuitive column selection
- Built-in search/filter
- Configuration file support

**Drawbacks:**
- Different output format than `ps`
- Learning new column names

---

### bottom (btm)

| Attribute | Details |
|-----------|---------|
| **Replaces** | `top`, `htop` |
| **Language** | Rust |
| **Repository** | [github.com/ClementTsang/bottom](https://github.com/ClementTsang/bottom) |

**Key Improvements:**
- Modern, graphical TUI
- CPU, memory, network, disk graphs
- Process management (kill, sort)
- Vi keybindings
- Battery monitoring
- Cross-platform (Windows, macOS, Linux)

**Agent-Relevant Features:**
- Customizable layouts via config file
- Vim-style navigation
- Freeze display to explore (`f`)
- Multiple color schemes

**Drawbacks:**
- Display not stable across updates (requires freeze)
- Heavier than htop
- Learning curve for keybindings

---

### btop / btop++

| Attribute | Details |
|-----------|---------|
| **Replaces** | `top`, `htop` |
| **Language** | C++ |
| **Repository** | [github.com/aristocratos/btop](https://github.com/aristocratos/btop) |

**Key Improvements:**
- Highly visual, BBS-inspired interface
- Full mouse support
- Process tree view
- Filtering and searching
- Game-inspired menu system
- GPU monitoring support

**Agent-Relevant Features:**
- Most feature-rich htop alternative
- Highly customizable appearance
- Process filtering

**Drawbacks:**
- Higher resource usage
- Visual complexity may be distracting

---

### zenith

| Attribute | Details |
|-----------|---------|
| **Replaces** | `top`, `htop` |
| **Language** | Rust |
| **Repository** | [github.com/bvaisvil/zenith](https://github.com/bvaisvil/zenith) |

**Key Improvements:**
- Zoomable charts (CPU, disk, network)
- GPU monitoring support
- Information-dense layout
- Expandable/collapsible sections

**Agent-Relevant Features:**
- Zoom in/out on charts (`+`/`-`)
- More traditional `top`-like layout
- GPU support for NVIDIA

**Drawbacks:**
- Less actively maintained than alternatives
- Fewer features than btop

---

## 7. System Information

### fastfetch

| Attribute | Details |
|-----------|---------|
| **Replaces** | `neofetch` (discontinued) |
| **Language** | C |
| **Repository** | [github.com/fastfetch-cli/fastfetch](https://github.com/fastfetch-cli/fastfetch) |

**Key Improvements:**
- 10x faster than neofetch
- More accurate information
- Better Wayland support
- JSONC configuration
- Cross-platform (Linux, macOS, Windows, Android, FreeBSD)
- More consistent number formatting

**Agent-Relevant Features:**
- Configurable modules (choose what to display)
- JSON output available
- Custom logos/images
- Active development

**Drawbacks:**
- Different configuration format than neofetch
- Larger feature set may be overkill

---

### macchina

| Attribute | Details |
|-----------|---------|
| **Replaces** | `neofetch` |
| **Language** | Rust |
| **Repository** | [github.com/Macchina-CLI/macchina](https://github.com/Macchina-CLI/macchina) |

**Key Improvements:**
- Fast, minimal Rust implementation
- TOML configuration
- Theming system
- Git repository information
- Memory safe

**Agent-Relevant Features:**
- Lightweight
- Custom themes separate from config
- Shows Git info in directories

**Drawbacks:**
- Smaller community than fastfetch
- Fewer built-in modules

---

### neofetch

| Attribute | Details |
|-----------|---------|
| **Replaces** | `screenfetch` |
| **Language** | Bash |
| **Status** | **Discontinued** (archived April 2024) |

**Note:** While neofetch is no longer maintained, it remains widely installed. Consider migrating to fastfetch or macchina for active development and bug fixes.

---

## 8. Directory Navigation

### zoxide

| Attribute | Details |
|-----------|---------|
| **Replaces** | `cd`, `autojump`, `z` |
| **Language** | Rust |
| **Repository** | [github.com/ajeetdsouza/zoxide](https://github.com/ajeetdsouza/zoxide) |

**Key Improvements:**
- Learns frequently used directories
- "Frecency" algorithm (frequency + recency)
- Fuzzy matching
- Works with all major shells
- Optional fzf integration for interactive selection

**Agent-Relevant Features:**
- Minimal keystrokes to navigate
- `zi` for interactive fzf-powered selection
- Cross-shell compatibility
- Imports from autojump/z

**Drawbacks:**
- Requires shell initialization
- Database builds over time (cold start)
- May jump to unexpected directories initially

**Setup:**
```bash
# Add to .bashrc/.zshrc
eval "$(zoxide init bash)"  # or zsh, fish, etc.

# Usage
z project   # Jump to most frecent "project" directory
zi          # Interactive selection with fzf
```

---

### autojump

| Attribute | Details |
|-----------|---------|
| **Replaces** | `cd` |
| **Language** | Python |
| **Repository** | [github.com/wting/autojump](https://github.com/wting/autojump) |

**Key Improvements:**
- Learns from directory visits
- Simple `j` command alias
- Mature, well-tested

**Agent-Relevant Features:**
- Wide shell support
- Predictable behavior

**Drawbacks:**
- Slower than zoxide (Python vs Rust)
- Less active development
- No fzf integration built-in

---

### z (original)

| Attribute | Details |
|-----------|---------|
| **Replaces** | `cd` |
| **Language** | Shell |
| **Status** | Mature, minimal maintenance |

**Note:** zoxide is the recommended modern replacement, offering the same functionality with better performance and more features.

---

## 9. Diff Tools

### delta

| Attribute | Details |
|-----------|---------|
| **Replaces** | `diff`, `git diff` |
| **Language** | Rust |
| **Repository** | [github.com/dandavison/delta](https://github.com/dandavison/delta) |

**Key Improvements:**
- Syntax highlighting (same themes as bat)
- Word-level diff highlighting
- Line numbers
- Side-by-side view
- `n`/`N` navigation between files
- Git blame improvements
- Hyperlinks
- Supports `--color-moved`

**Agent-Relevant Features:**
- Works with git diff, grep output, blame
- Highly configurable via gitconfig
- diff-so-fancy compatibility mode
- Merge conflict display

**Drawbacks:**
- Requires configuration in gitconfig
- Output optimized for humans
- Can be slow on very large diffs

**Git Configuration:**
```gitconfig
[core]
    pager = delta

[interactive]
    diffFilter = delta --color-only
```

---

### difftastic

| Attribute | Details |
|-----------|---------|
| **Replaces** | `diff` |
| **Language** | Rust |
| **Repository** | [github.com/Wilfred/difftastic](https://github.com/Wilfred/difftastic) |

**Key Improvements:**
- **Structural/AST-aware diffing**
- Understands syntax, not just lines
- Language-aware highlighting
- Shows meaningful changes only
- Supports 50+ languages

**Agent-Relevant Features:**
- Reduces cognitive overhead (no false positives from formatting)
- Better for code review
- Can be used with Git

**Drawbacks:**
- Slower than line-based diffs
- May miss some text-only changes
- Less mature than delta

---

### diff-so-fancy

| Attribute | Details |
|-----------|---------|
| **Replaces** | `diff` |
| **Language** | Perl |
| **Repository** | [github.com/so-fancy/diff-so-fancy](https://github.com/so-fancy/diff-so-fancy) |

**Key Improvements:**
- Cleaner diff output
- Word-level highlighting
- Better defaults than raw diff

**Agent-Relevant Features:**
- Drop-in git pager
- Simpler than delta

**Drawbacks:**
- Less feature-rich than delta
- No syntax highlighting
- Perl dependency

---

## 10. HTTP Clients

### HTTPie

| Attribute | Details |
|-----------|---------|
| **Replaces** | `curl`, `wget` |
| **Language** | Python |
| **Repository** | [github.com/httpie/cli](https://github.com/httpie/cli) |
| **Website** | [httpie.io](https://httpie.io/) |

**Key Improvements:**
- Human-friendly syntax
- Colorized, formatted output
- JSON support built-in
- Sessions for authentication persistence
- Form and file uploads simplified
- HTTPS by default

**Agent-Relevant Features:**
- Intuitive request building
- JSON input/output handling
- Plugin ecosystem
- Desktop app available

**Drawbacks:**
- Python dependency (slower startup)
- Different syntax than curl (scripts may need rewriting)
- Some advanced curl features missing

**Example:**
```bash
# GET request
http httpbin.org/get

# POST with JSON
http POST httpbin.org/post name=John email=john@example.com

# Authentication
http -a user:pass httpbin.org/basic-auth/user/pass
```

---

### xh

| Attribute | Details |
|-----------|---------|
| **Replaces** | `curl`, `httpie` |
| **Language** | Rust |
| **Repository** | [github.com/ducaale/xh](https://github.com/ducaale/xh) |

**Key Improvements:**
- HTTPie-compatible syntax
- Much faster than HTTPie (Rust vs Python)
- Single binary, no dependencies
- Smaller memory footprint

**Agent-Relevant Features:**
- Drop-in HTTPie replacement
- Fast startup time
- Portable binary

**Drawbacks:**
- Fewer features than HTTPie
- Smaller community
- Some HTTPie plugins incompatible

---

### curlie

| Attribute | Details |
|-----------|---------|
| **Replaces** | `curl` |
| **Language** | Go |
| **Repository** | [github.com/rs/curlie](https://github.com/rs/curlie) |

**Key Improvements:**
- HTTPie-style output with curl's power
- All curl options available
- Formatted JSON responses
- Headers shown by default

**Agent-Relevant Features:**
- Familiar curl syntax
- Better output formatting
- Uses curl under the hood (full compatibility)

**Drawbacks:**
- Less intuitive than HTTPie
- Requires curl installed
- Go binary size

---

### hurl

| Attribute | Details |
|-----------|---------|
| **Replaces** | `curl` (for testing), Postman |
| **Language** | Rust |
| **Repository** | [github.com/Orange-OpenSource/hurl](https://github.com/Orange-OpenSource/hurl) |
| **Website** | [hurl.dev](https://hurl.dev/) |

**Key Improvements:**
- Plain text file format for HTTP requests
- Chain multiple requests
- Built-in assertions (status, headers, body)
- XPath/JSONPath queries
- Performance testing
- CI/CD friendly (JUnit, TAP, HTML reports)
- Parallel execution

**Agent-Relevant Features:**
- Declarative test files
- Export to curl commands
- JSON output
- Cookie management
- Ideal for API testing pipelines

**Drawbacks:**
- Learning curve for file format
- More suited for testing than ad-hoc requests
- Less interactive than httpie

**Example Hurl file:**
```hurl
GET https://api.example.com/users
HTTP 200
[Asserts]
jsonpath "$.users" count > 0

POST https://api.example.com/users
{
  "name": "John"
}
HTTP 201
```

---

## 11. JSON Tools

### jq

| Attribute | Details |
|-----------|---------|
| **Replaces** | Manual JSON parsing |
| **Language** | C |
| **Repository** | [github.com/jqlang/jq](https://github.com/jqlang/jq) |

**Key Improvements:**
- Powerful JSON querying and transformation
- Streaming support for large files
- Compact one-liner syntax
- Widely available and documented

**Agent-Relevant Features:**
- Ubiquitous (available everywhere)
- Compact, shareable snippets
- Streaming for memory efficiency
- Raw output mode (`-r`)

**Drawbacks:**
- Steep learning curve
- Cryptic syntax for complex operations
- Single-threaded

**Example:**
```bash
# Extract all names from array
cat data.json | jq '.users[].name'

# Filter and transform
cat data.json | jq '.items | map(select(.active)) | length'
```

---

### gojq

| Attribute | Details |
|-----------|---------|
| **Replaces** | `jq` |
| **Language** | Go |
| **Repository** | [github.com/itchyny/gojq](https://github.com/itchyny/gojq) |

**Key Improvements:**
- Pure Go implementation
- YAML input/output support
- Can be used as Go library
- Generally compatible with jq

**Agent-Relevant Features:**
- Embeddable in Go applications
- YAML support built-in
- Consistent cross-platform behavior

**Drawbacks:**
- Some edge-case differences from jq
- Slightly different error messages

---

### jaq

| Attribute | Details |
|-----------|---------|
| **Replaces** | `jq` |
| **Language** | Rust |
| **Repository** | [github.com/01mf02/jaq](https://github.com/01mf02/jaq) |

**Key Improvements:**
- Faster than jq in many cases
- More predictable language semantics
- Better error messages
- Rust memory safety

**Agent-Relevant Features:**
- Performance boost for large files
- More correct behavior in edge cases
- Compatible with most jq filters

**Drawbacks:**
- Missing some jq features
- Smaller community
- Less documentation

---

### fx

| Attribute | Details |
|-----------|---------|
| **Replaces** | `jq` (interactive) |
| **Language** | Go |
| **Repository** | [github.com/antonmedv/fx](https://github.com/antonmedv/fx) |

**Key Improvements:**
- Interactive JSON exploration
- Uses JavaScript for queries
- Both TUI and CLI modes
- Streaming support

**Agent-Relevant Features:**
- JavaScript syntax (familiar to many)
- Interactive exploration before scripting
- Can chain operations like jq

**Drawbacks:**
- JavaScript, not jq syntax
- Interactive mode less scriptable
- Requires learning different syntax

---

### jless

| Attribute | Details |
|-----------|---------|
| **Replaces** | `jq` (viewing), `less` (for JSON) |
| **Language** | Rust |
| **Repository** | [github.com/PaulJuliusMartinez/jless](https://github.com/PaulJuliusMartinez/jless) |

**Key Improvements:**
- Interactive JSON viewer
- Vim keybindings
- Collapsible tree view
- Search functionality
- Outputs jq selectors for paths

**Agent-Relevant Features:**
- Explore JSON before writing jq queries
- Copy jq path for selected node
- Fast for large files

**Drawbacks:**
- Read-only (no transformation)
- Vim keybindings may not suit everyone
- Interactive only

---

## 12. Git Tools

### lazygit

| Attribute | Details |
|-----------|---------|
| **Replaces** | `git` (for common operations) |
| **Language** | Go |
| **Repository** | [github.com/jesseduffield/lazygit](https://github.com/jesseduffield/lazygit) |

**Key Improvements:**
- Simple terminal UI
- Visual staging of changes
- Interactive rebasing
- Conflict resolution helper
- Commit graph visualization
- Undo operations

**Agent-Relevant Features:**
- Beginner-friendly interface
- Discoverability (menus show commands)
- Streamlined common workflows
- Cross-platform

**Drawbacks:**
- Limited customization
- Less integration with external tools
- Memory overhead for large repos

---

### gitui

| Attribute | Details |
|-----------|---------|
| **Replaces** | `git` (for common operations) |
| **Language** | Rust |
| **Repository** | [github.com/gitui-org/gitui](https://github.com/gitui-org/gitui) |

**Key Improvements:**
- Blazing fast (handles Linux kernel repo)
- Keyboard-driven interface
- External tool integration
- Extensive customization
- Low memory usage

**Agent-Relevant Features:**
- Performs well on large repositories
- Customizable keybindings
- External difftool/editor integration
- Async operations

**Drawbacks:**
- Steeper learning curve
- Less visually polished than lazygit
- More complex interface

---

### tig

| Attribute | Details |
|-----------|---------|
| **Replaces** | `git log`, `git blame` |
| **Language** | C |
| **Repository** | [github.com/jonas/tig](https://github.com/jonas/tig) |

**Key Improvements:**
- Text-mode interface for repository browsing
- Multiple views (log, diff, blame, tree)
- Stage hunks interactively
- Can act as pager for git commands

**Agent-Relevant Features:**
- Lightweight
- Good for browsing history
- Works well over SSH

**Drawbacks:**
- Development slowed
- Less feature-rich than lazygit/gitui
- Text-mode only (no colors in basic mode)

---

## 13. Text Processing

### sd

| Attribute | Details |
|-----------|---------|
| **Replaces** | `sed` |
| **Language** | Rust |
| **Repository** | [github.com/chmln/sd](https://github.com/chmln/sd) |

**Key Improvements:**
- Intuitive syntax (no escaping nightmares)
- Uses modern regex (Rust regex crate)
- 2-11x faster than sed
- Easy read/write

**Agent-Relevant Features:**
- Simple find/replace: `sd 'from' 'to' file`
- No need to escape slashes
- Regex by default, `-F` for fixed strings

**Drawbacks:**
- Not a full sed replacement
- Single purpose (find/replace only)
- Different regex flavor than sed

**Example:**
```bash
# Replace in file
sd 'hello' 'world' file.txt

# Pipe usage
echo "hello world" | sd 'world' 'universe'
```

---

### sad

| Attribute | Details |
|-----------|---------|
| **Replaces** | `sed` (with preview) |
| **Language** | Rust |
| **Repository** | [github.com/ms-jpq/sad](https://github.com/ms-jpq/sad) |

**Key Improvements:**
- Interactive preview before replacement
- Uses fzf for selection
- Syntax highlighting via delta
- Selective replacement

**Agent-Relevant Features:**
- Preview changes before applying
- Select which replacements to apply
- Undo-friendly workflow

**Drawbacks:**
- Requires fzf
- Interactive (not purely scriptable)

---

### choose

| Attribute | Details |
|-----------|---------|
| **Replaces** | `cut`, `awk` (field selection) |
| **Language** | Rust |
| **Repository** | [github.com/theryangeary/choose](https://github.com/theryangeary/choose) |

**Key Improvements:**
- Python-like slicing syntax
- Negative indexing from end
- Regex delimiters
- Zero-indexed

**Agent-Relevant Features:**
- Intuitive field selection
- Reverse ranges supported
- Faster than awk

**Drawbacks:**
- Different indexing than cut (0-based)
- Not a full awk replacement
- Learning new syntax

**Example:**
```bash
# Select fields 0, 2, and 4
echo "a b c d e" | choose 0 2 4

# Select from end
echo "a b c d e" | choose -1   # "e"

# Range
echo "a b c d e" | choose 1:3  # "b c"
```

---

### hck

| Attribute | Details |
|-----------|---------|
| **Replaces** | `cut`, `awk` (field selection) |
| **Language** | Rust |
| **Repository** | [github.com/sstadick/hck](https://github.com/sstadick/hck) |

**Key Improvements:**
- Near drop-in cut replacement
- Regex delimiters
- Column reordering
- Select by header name (`-F`)

**Agent-Relevant Features:**
- Familiar cut syntax
- Header-based selection
- Output column reordering
- Fast for large files

**Drawbacks:**
- Some cut edge cases differ
- Learning header selection syntax

---

## 14. Benchmarking

### hyperfine

| Attribute | Details |
|-----------|---------|
| **Replaces** | `time`, custom benchmarking |
| **Language** | Rust |
| **Repository** | [github.com/sharkdp/hyperfine](https://github.com/sharkdp/hyperfine) |

**Key Improvements:**
- Statistical analysis (mean, median, stddev)
- Warmup runs
- Cache clearing between runs
- Outlier detection
- Multiple command comparison
- Parameter scanning

**Agent-Relevant Features:**
- Export to JSON, CSV, Markdown
- Parameterized benchmarks
- Shell-independent mode (`-N`)
- Progress feedback

**Drawbacks:**
- Overkill for simple timing
- May require many runs for accuracy
- Learning parameter syntax

**Example:**
```bash
# Compare two commands
hyperfine 'fd "\.py$"' 'find . -name "*.py"'

# With warmup
hyperfine --warmup 3 'cargo build'

# Parameter scan
hyperfine -P threads 1 8 'my_program --threads {threads}'

# Export results
hyperfine --export-json results.json 'sleep 0.5'
```

---

## 15. Other Modern Rewrites

### tokei

| Attribute | Details |
|-----------|---------|
| **Replaces** | `cloc`, `wc -l` |
| **Language** | Rust |
| **Repository** | [github.com/XAMPPRocky/tokei](https://github.com/XAMPPRocky/tokei) |

**Key Improvements:**
- Very fast (parallel processing)
- 150+ languages supported
- Accurate counting (ignores comments/blanks properly)
- Multiple output formats

**Agent-Relevant Features:**
- JSON, YAML, CBOR output
- Sort by various metrics
- Exclude patterns
- Compact or detailed views

**Drawbacks:**
- Some language detection edge cases
- Different counts than cloc occasionally

---

### bandwhich

| Attribute | Details |
|-----------|---------|
| **Replaces** | `nethogs`, `iftop` |
| **Language** | Rust |
| **Repository** | [github.com/imsnif/bandwhich](https://github.com/imsnif/bandwhich) |

**Key Improvements:**
- Shows bandwidth by process, connection, and remote host
- Real-time updates
- Identifies which processes use network
- Cross-platform

**Agent-Relevant Features:**
- Diagnose network-heavy processes
- Identify unexpected connections
- Clean interface

**Drawbacks:**
- Requires root/admin privileges
- May not capture all traffic types

---

### dog

| Attribute | Details |
|-----------|---------|
| **Replaces** | `dig`, `nslookup` |
| **Language** | Rust |
| **Repository** | [github.com/ogham/dog](https://github.com/ogham/dog) |

**Key Improvements:**
- Colorized output
- Simpler syntax
- JSON output
- DNS-over-TLS and DNS-over-HTTPS support

**Agent-Relevant Features:**
- `--json` for structured output
- Modern DNS protocols
- Human-readable defaults

**Drawbacks:**
- Less feature-complete than dig
- Smaller community
- Development slowed

---

### grex

| Attribute | Details |
|-----------|---------|
| **Replaces** | Manual regex writing |
| **Language** | Rust |
| **Repository** | [github.com/pemistahl/grex](https://github.com/pemistahl/grex) |

**Key Improvements:**
- Generates regex from examples
- Guaranteed to match provided test cases
- Simplification options
- Library and CLI

**Agent-Relevant Features:**
- Rapid regex prototyping
- Avoids regex syntax errors
- Can be used as Rust library

**Drawbacks:**
- Generated regex may be overly specific
- May need manual refinement
- Not for complex patterns

**Example:**
```bash
# Generate regex from examples
grex 'hello' 'hallo' 'hullo'
# Output: ^h[aeu]llo$

# More permissive
grex -d 'hello' 'hallo' 'hullo'
# Output: ^h.llo$
```

---

### hexyl

| Attribute | Details |
|-----------|---------|
| **Replaces** | `xxd`, `hexdump` |
| **Language** | Rust |
| **Repository** | [github.com/sharkdp/hexyl](https://github.com/sharkdp/hexyl) |

**Key Improvements:**
- Colorized hex output
- Distinguishes ASCII, non-ASCII, null, whitespace
- Clean, readable format
- Byte grouping options

**Agent-Relevant Features:**
- Quick binary file inspection
- Visual byte categorization
- Range selection (`--skip`, `--length`)

**Drawbacks:**
- No editing capability
- Output optimized for viewing, not scripting

---

## Summary: Quick Reference

### Installation Priority (Most Impactful First)

1. **Essential (immediate productivity boost):**
   - `ripgrep` (rg) - searching
   - `fd` - file finding
   - `bat` - file viewing
   - `eza` - file listing
   - `delta` - git diffs
   - `zoxide` - directory navigation

2. **Highly Recommended:**
   - `dust` - disk usage
   - `bottom` or `btop` - process viewing
   - `sd` - find/replace
   - `hyperfine` - benchmarking
   - `jq` - JSON processing (usually pre-installed)

3. **Situational (use case dependent):**
   - `lazygit`/`gitui` - git TUI
   - `ast-grep` - structural code search
   - `hurl` - API testing
   - `glow` - markdown viewing
   - `broot` - interactive navigation

### Ubuntu/Debian Package Names

| Tool | Package/Command |
|------|-----------------|
| bat | `batcat` |
| fd | `fdfind` |
| ripgrep | `rg` |

### Alias Recommendations

```bash
# ~/.bashrc or ~/.zshrc

# File operations
alias cat='batcat --paging=never'
alias ls='eza --icons'
alias ll='eza -la --icons --git'
alias tree='eza --tree --icons'

# Finding
alias find='fdfind'

# Navigation
eval "$(zoxide init bash)"  # or zsh

# Git
[[ -f ~/.gitconfig ]] && git config --global core.pager delta
```

---

## Sources

- [It's FOSS - Rust Alternative CLI Tools](https://itsfoss.com/rust-alternative-cli-tools/)
- [Zaiste - Shell Commands Rewritten in Rust](https://zaiste.net/posts/shell-commands-rust/)
- [eza Documentation](https://eza.rocks/)
- [sharkdp/bat GitHub](https://github.com/sharkdp/bat)
- [sharkdp/fd GitHub](https://github.com/sharkdp/fd)
- [BurntSushi/ripgrep GitHub](https://github.com/BurntSushi/ripgrep)
- [bootandy/dust GitHub](https://github.com/bootandy/dust)
- [ajeetdsouza/zoxide GitHub](https://github.com/ajeetdsouza/zoxide)
- [dandavison/delta GitHub](https://github.com/dandavison/delta)
- [sharkdp/hyperfine GitHub](https://github.com/sharkdp/hyperfine)
- [ast-grep Documentation](https://ast-grep.github.io/)
- [Hurl Documentation](https://hurl.dev/)
- [charmbracelet/glow GitHub](https://github.com/charmbracelet/glow)
- [fastfetch-cli/fastfetch GitHub](https://github.com/fastfetch-cli/fastfetch)
- [jesseduffield/lazygit GitHub](https://github.com/jesseduffield/lazygit)
- [gitui-org/gitui GitHub](https://github.com/gitui-org/gitui)
- [beyondgrep.com Feature Comparison](https://beyondgrep.com/feature-comparison/)
- [Wesley Moore - Rust Top Alternatives](https://www.wezm.net/v2/posts/2020/rust-top-alternatives/)
- [LinuxLinks - jq Alternatives](https://www.linuxlinks.com/alternatives-popular-cli-tools-jq/)
- [httpie Documentation](https://httpie.io/docs/cli)
