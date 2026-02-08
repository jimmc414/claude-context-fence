---
description: "File watching recipes reference (internal) - Receives prepared arguments from task-watch with watch target and context included"
argument-hint: "[prepared argument from task-watch router]"
context: fork
model: inherit
allowed-tools: Read
---

# File Watching Tasks

**Tools:** entr, watchexec, inotifywait, fswatch, watchman, nodemon, cargo-watch, chokidar-cli, watch

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Run on file change | `ls file.py \| entr python file.py` |
| Watch and rebuild | `find . -name "*.go" \| entr go build` |
| Watch directory | `inotifywait -m -r directory/` |
| Watch by extension | `watchexec -e py,js -- pytest` |
| Node.js dev server | `nodemon app.js` |
| Rust watch + test | `cargo watch -x test` |
| Stream changes | `fswatch -r src/` |
| JS glob watcher | `chokidar '**/*.js' -c 'npm test'` |

---

## When to Use What

| Tool | Best For | Limitation |
|------|----------|------------|
| entr | Simple "re-run on change", piped file lists | Needs file list piped in, no glob built-in |
| watchexec | Extension/glob filtering, cross-platform | Requires install (cargo/binary) |
| inotifywait | Low-level Linux filesystem events | Linux only, verbose scripting |
| fswatch | Cross-platform event streaming | macOS-focused, less common on Linux |
| nodemon | Node.js app restart on change | Node.js focused |
| cargo watch | Rust development workflow | Rust/cargo only |
| chokidar-cli | JS glob patterns, npm ecosystem | Requires Node.js |

---

## entr (recommended)

```bash
ls file.py | entr python file.py                 # Run on change
find . -name "*.py" | entr pytest                # Watch .py files
ls *.py | entr -c python main.py                 # Clear screen
ls *.py | entr -r python server.py               # Restart server
```

---

## inotifywait

```bash
inotifywait -m -e modify file.txt                # Watch single file
inotifywait -m -r -e modify,create,delete dir/   # Watch directory

# Run command on change
while inotifywait -e modify file.py; do python file.py; done
```

---

## watchexec

```bash
watchexec -e py,js -- pytest                     # Watch by extension
watchexec -w src/ -- make build                  # Watch specific directory
watchexec -e rs -- cargo test                    # Rust: re-test on change
watchexec -r -e py -- python server.py           # Restart on change
watchexec --ignore '*.log' -- npm test           # Ignore patterns
watchexec -e py,js,ts --debounce 500 -- make     # Debounce rapid changes
```

---

## nodemon

```bash
nodemon app.js                                   # Watch and restart Node.js
nodemon --exec python script.py                  # Use with any command
nodemon --watch src --ext js,ts app.js           # Specific dirs and extensions
nodemon --delay 2 app.js                         # Delay restart by 2 seconds
```

---

## cargo watch (Rust)

```bash
cargo watch -x test                              # Re-run tests on change
cargo watch -x 'run -- --port 8080'              # Re-run with args
cargo watch -x check -x test -x run              # Chain commands
```

---

## fswatch

```bash
fswatch -r src/ | while read f; do echo "Changed: $f"; done
fswatch -r --event Updated src/ | xargs -I{} echo "Modified: {}"
fswatch -r -e '.*' -i '\\.py$' src/              # Only Python files
```

---

## chokidar-cli

```bash
chokidar '**/*.js' -c 'npm test'                 # Glob pattern + command
chokidar 'src/**/*.ts' -c 'npm run build'        # Watch TypeScript
chokidar '**/*.css' --initial -c 'postcss build'  # Run initially too
```

---

## Common Patterns

```bash
# Auto-rebuild
find . -name "*.go" | entr -c go build

# Auto-test
find . -name "*.py" | entr -c pytest

# Restart server
ls *.py | entr -r python server.py

# Watch and run make
watchexec -e c,h -- make

# Node.js dev with non-Node command
nodemon --exec 'python manage.py runserver' --ext py
```

---

## Installation

```bash
# Linux (apt)
sudo apt install entr inotify-tools fswatch

# watchexec
cargo install watchexec-cli    # or download binary from GitHub

# nodemon / chokidar-cli
npm install -g nodemon chokidar-cli

# cargo watch (Rust)
cargo install cargo-watch
```
