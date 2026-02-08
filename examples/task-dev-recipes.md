---
description: "Development tools recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-dev with language and context included"
when-to-use: "Receives prepared arguments from task-dev with language and context included"
context: fork
model: inherit
allowed-tools: Read
---

# Development Tools Recipes

**Tools:** make, cmake, ninja, just, gcc, g++, clang, clang++, ldd, shellcheck, shfmt, eslint, prettier, ruff, black, isort, pyright, mypy, bandit, python, pip, uv, poetry, pdm, pyenv, node, npm, pnpm, yarn, bun, deno, volta, nvm, tsc, esbuild, cargo, go, hadolint

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Build with make | `make -j$(nproc)` |
| Build with cmake | `cmake -B build && cmake --build build` |
| Compile C file | `gcc -Wall -O2 -o prog main.c` |
| Lint Python | `ruff check .` |
| Format Python | `ruff format .` |
| Type check Python | `pyright` |
| Lint shell | `shellcheck script.sh` |
| Format shell | `shfmt -w script.sh` |
| Lint JS/TS | `eslint .` |
| Format JS/TS | `prettier --write .` |
| Install Python deps | `pip install -r requirements.txt` |
| Install Node deps | `npm ci` |
| Go test | `go test ./...` |
| Rust build | `cargo build --release` |
| Watch and re-run | `ls *.py \| entr pytest` |
| Pre-commit hooks | `pre-commit run --all-files` |
| Audit Python deps | `pip-audit` |
| Audit npm deps | `npm audit` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| Build automation | make | Universal, pre-installed |
| Modern build runner | just | Simpler syntax than make |
| Shell script linting | shellcheck | Finds common bash pitfalls |
| Shell formatting | shfmt | Consistent style |
| Python linting + formatting | ruff | Fastest, replaces flake8+black |
| Python type checking | pyright / mypy | Static type analysis |
| JS/TS formatting | prettier | Opinionated, consistent |
| JS/TS linting | eslint | Configurable rule engine |
| Python package install | uv / pip | uv is faster, pip universal |
| Node package install | pnpm / npm | pnpm saves disk, npm standard |
| Dockerfile linting | hadolint | Best practices for Dockerfiles |
| GitHub Actions linting | actionlint | Validates workflow YAML |
| Security scanning | semgrep / bandit | Code pattern vulnerability detection |

---

## Make / CMake / Just / Ninja

### Make

```bash
# Basic targets
make                                  # Run default target
make build                            # Specific target
make clean                            # Clean target
make install                          # Install target

# Parallel builds
make -j$(nproc)                       # Use all CPU cores
make -j8                              # Explicit 8 jobs
make -j$(nproc) -l4.0                 # Limit load average to 4.0

# Dry run and debugging
make -n                               # Dry run: print commands without executing
make -n install                       # Preview install commands
make -B                               # Force rebuild all targets (unconditional)
make -d                               # Debug: print detailed decision info
make --debug=basic                    # Why each target was rebuilt
make -p                               # Print database of rules and variables
make --trace                          # Print each target as it executes

# Directory and file options
make -C subdir/                       # Change to directory before running
make -f Makefile.custom               # Use alternate makefile
make -I /path/to/includes             # Search path for included makefiles

# Variables
make CC=clang                         # Override compiler
make CFLAGS="-O3 -march=native"      # Override flags
make PREFIX=/usr/local                # Override install prefix
make V=1                              # Verbose mode (common convention)
make DEBUG=1                          # Debug mode (common convention)

# Multiple targets
make clean all                        # Run clean then all
make -j$(nproc) test install          # Parallel build, then test, then install
```

#### Makefile Syntax Reference

```makefile
# Variables
CC      := gcc
CFLAGS  := -Wall -Wextra -O2
LDFLAGS := -lm -lpthread
SRC     := $(wildcard src/*.c)
OBJ     := $(SRC:.c=.o)

# Automatic variables
# $@  = target name
# $<  = first prerequisite
# $^  = all prerequisites
# $*  = stem of pattern match
# $(@D) = directory part of target
# $(@F) = file part of target

# Pattern rules
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# Explicit rule
prog: $(OBJ)
	$(CC) $(LDFLAGS) $^ -o $@

# Phony targets
.PHONY: clean install test all

clean:
	rm -f $(OBJ) prog

install: prog
	install -m 755 prog /usr/local/bin/

# Conditional
ifeq ($(DEBUG),1)
  CFLAGS += -g -O0 -DDEBUG
else
  CFLAGS += -O2 -DNDEBUG
endif

ifdef VERBOSE
  Q :=
else
  Q := @
endif

# Functions
SOURCES := $(wildcard *.c)
OBJECTS := $(patsubst %.c,%.o,$(SOURCES))
DIRS    := $(shell find . -type d)
RESULT  := $(strip $(VAR))

# Include other makefiles
-include deps/*.d
include config.mk
```

### CMake

```bash
# Configure
cmake -B build                        # Configure into build/ directory
cmake -B build -DCMAKE_BUILD_TYPE=Release    # Release build (optimized)
cmake -B build -DCMAKE_BUILD_TYPE=Debug      # Debug build (symbols, no opt)
cmake -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo  # Release + debug info
cmake -B build -DCMAKE_INSTALL_PREFIX=/usr/local  # Install prefix
cmake -B build -DCMAKE_C_COMPILER=clang      # Use clang
cmake -B build -DCMAKE_CXX_STANDARD=20      # C++20 standard
cmake -B build -DBUILD_SHARED_LIBS=ON        # Build shared libraries
cmake -B build -DBUILD_TESTING=OFF           # Disable tests

# Generators
cmake -B build -G "Ninja"            # Use Ninja generator (faster)
cmake -B build -G "Unix Makefiles"   # Use Make generator (default on Linux)

# Build
cmake --build build                   # Build using detected generator
cmake --build build -j$(nproc)        # Parallel build
cmake --build build --target mytarget # Build specific target
cmake --build build --config Release  # Multi-config generator build type
cmake --build build --verbose         # Show compile commands
cmake --build build --clean-first     # Clean before building

# Install
cmake --install build                 # Install to configured prefix
cmake --install build --prefix /opt   # Override install prefix
cmake --install build --component dev # Install specific component

# Inspect cache
cmake -B build -L                     # List non-advanced cache variables
cmake -B build -LA                    # List all cache variables
cmake -B build -LAH                   # List all with help strings

# Presets (CMakePresets.json)
cmake --list-presets                  # List available presets
cmake --preset release                # Configure using preset
cmake --build --preset release        # Build using preset

# CTest (testing)
ctest --test-dir build                # Run tests
ctest --test-dir build -j$(nproc)     # Parallel tests
ctest --test-dir build -V             # Verbose test output
ctest --test-dir build -R "test_name" # Run matching tests
ctest --test-dir build --rerun-failed # Re-run only failed tests
```

### Just

```bash
# Running recipes
just                                  # Run default recipe (first in file)
just build                            # Run specific recipe
just build release                    # Recipe with arguments
just --list                           # List available recipes with descriptions
just --summary                        # List recipe names only

# Dry run and info
just -n build                         # Dry run: show commands without executing
just --evaluate                       # Show variable values
just --show build                     # Show recipe source code
just --dump                           # Print entire justfile

# Options
just --justfile path/justfile         # Use specific justfile
just --working-directory /path        # Set working directory
just --set var value                  # Override variable
just --dotenv-path .env.local         # Use specific env file
```

#### Justfile Syntax Reference

```just
# Variables
version := "1.0.0"
arch := `uname -m`

# Default recipe
default: build test

# Recipe with dependencies
build: clean
    gcc -O2 -o prog main.c

# Recipe with arguments
deploy target="staging":
    rsync -avz build/ {{target}}:/app/

# Shebang recipe (use any language)
analyze:
    #!/usr/bin/env python3
    import json
    with open("data.json") as f:
        data = json.load(f)
    print(f"Records: {len(data)}")

# Platform-specific
test:
    #!/usr/bin/env bash
    {{ if os() == "linux" { "pytest" } else { "python -m pytest" } }}

# Load .env file
set dotenv-load

# Aliases
alias b := build
alias t := test
```

### Ninja

```bash
ninja                                 # Build default targets
ninja -j$(nproc)                      # Parallel build
ninja -v                              # Verbose: show full compile commands
ninja -n                              # Dry run
ninja -t targets                      # List all targets
ninja -t deps target                  # Show deps of target
ninja -t clean                        # Clean built files
ninja -C build/                       # Change to directory
```

---

## C/C++ Compilation

### GCC / G++ / Clang

```bash
# Basic compilation
gcc -o prog main.c                    # Compile and link C
g++ -o prog main.cpp                  # Compile and link C++
clang -o prog main.c                  # Clang C
clang++ -o prog main.cpp             # Clang C++

# Compile only (no link)
gcc -c main.c                         # Produces main.o
gcc -c -o output.o main.c            # Specify output object file

# Preprocess only
gcc -E main.c > main.i               # Show preprocessor output

# Generate assembly
gcc -S main.c                         # Produces main.s
gcc -S -masm=intel main.c            # Intel syntax assembly
```

### Optimization Levels

```bash
gcc -O0 -o prog main.c               # No optimization (default, fast compile)
gcc -O1 -o prog main.c               # Basic optimization
gcc -O2 -o prog main.c               # Standard optimization (recommended)
gcc -O3 -o prog main.c               # Aggressive optimization
gcc -Os -o prog main.c               # Optimize for size
gcc -Og -o prog main.c               # Optimize for debugging (minimal, debuggable)
gcc -Ofast -o prog main.c            # O3 + fast-math (may break IEEE compliance)
gcc -march=native -O2 -o prog main.c # Optimize for current CPU architecture
```

### Warning Flags

```bash
gcc -Wall -o prog main.c                     # Common warnings
gcc -Wall -Wextra -o prog main.c             # More warnings
gcc -Wall -Wextra -Werror -o prog main.c     # Warnings as errors
gcc -Wall -Wextra -pedantic -o prog main.c   # Strict ISO compliance
gcc -Wshadow -o prog main.c                  # Variable shadowing
gcc -Wconversion -o prog main.c              # Implicit conversion warnings
gcc -Wformat=2 -o prog main.c               # Printf format string checks
gcc -Wno-unused-parameter -o prog main.c     # Suppress specific warning
gcc -Wdouble-promotion -o prog main.c        # Float to double promotion

# Recommended strict set
gcc -Wall -Wextra -Werror -pedantic -Wshadow -Wconversion -Wformat=2 -o prog main.c
```

### Debug Symbols and Sanitizers

```bash
# Debug symbols
gcc -g -o prog main.c                # Basic debug info
gcc -g3 -o prog main.c               # Maximum debug info (includes macros)
gcc -ggdb -o prog main.c             # GDB-specific debug info

# Address Sanitizer (buffer overflow, use-after-free)
gcc -fsanitize=address -g -o prog main.c
# Memory Sanitizer (uninitialized reads, clang only)
clang -fsanitize=memory -g -o prog main.c
# Thread Sanitizer (data races)
gcc -fsanitize=thread -g -o prog main.c
# Undefined Behavior Sanitizer
gcc -fsanitize=undefined -g -o prog main.c
# Combined sanitizers
gcc -fsanitize=address,undefined -g -o prog main.c
# Leak Sanitizer (standalone, lighter than ASan)
gcc -fsanitize=leak -g -o prog main.c

# Profiling (gprof)
gcc -pg -o prog main.c
./prog
gprof prog gmon.out > profile.txt
```

### Language Standards

```bash
gcc -std=c11 -o prog main.c          # C11 standard
gcc -std=c17 -o prog main.c          # C17 standard (latest stable C)
gcc -std=gnu17 -o prog main.c        # C17 + GNU extensions
g++ -std=c++17 -o prog main.cpp      # C++17
g++ -std=c++20 -o prog main.cpp      # C++20
g++ -std=c++23 -o prog main.cpp      # C++23 (partial support)
```

### Linking

```bash
# Libraries
gcc -o prog main.c -lm               # Link math library
gcc -o prog main.c -lpthread          # Link pthread
gcc -o prog main.c -lssl -lcrypto    # Link OpenSSL
gcc -o prog main.c -L/opt/lib -lfoo  # Add library search path
gcc -o prog main.c -Wl,-rpath,/opt/lib  # Runtime library path

# Include paths
gcc -I/usr/local/include -o prog main.c
gcc -Iinclude/ -Isrc/ -o prog main.c

# Defines
gcc -DNDEBUG -o prog main.c          # Define NDEBUG
gcc -DVERSION=\"1.0\" -o prog main.c # Define with value
gcc -UDEBUG -o prog main.c           # Undefine

# Static and shared
gcc -static -o prog main.c           # Fully static binary
gcc -shared -fPIC -o libfoo.so foo.c # Create shared library
gcc -fPIC -c foo.c                   # Position-independent code (for .so)

# Link-Time Optimization
gcc -flto -O2 -o prog main.c         # LTO (whole program optimization)

# Code coverage
gcc -fprofile-arcs -ftest-coverage -o prog main.c
./prog
gcov main.c                           # Generate coverage report

# Faster builds
gcc -pipe -o prog main.c             # Use pipes between stages (less I/O)
```

### pkg-config

```bash
# Query package flags
pkg-config --cflags openssl           # Compiler flags (-I...)
pkg-config --libs openssl             # Linker flags (-L... -l...)
pkg-config --modversion openssl       # Installed version
pkg-config --list-all                 # List all known packages

# Use in compilation
gcc $(pkg-config --cflags --libs openssl) -o prog main.c
gcc $(pkg-config --cflags --libs gtk+-3.0) -o gui main.c
```

### Static Libraries and Binary Tools

```bash
# Create static library
ar rcs libfoo.a foo.o bar.o           # Create archive from objects
ar t libfoo.a                         # List archive contents

# Check library dependencies
ldd ./prog                            # List shared library deps
ldd -v ./prog                         # Verbose with version info

# Strip symbols (reduce binary size)
strip ./prog                          # Remove all symbols
strip --strip-debug ./prog            # Remove debug symbols only
strip --strip-unneeded ./prog         # Remove unneeded symbols

# Inspect binary
nm ./prog                             # List symbols
nm -D ./prog                          # Dynamic symbols only
objdump -d ./prog                     # Disassemble
readelf -h ./prog                     # ELF header info
readelf -d ./prog                     # Dynamic section (shared libs)
size ./prog                           # Section sizes (text/data/bss)
```

### Cross-Compilation

```bash
# ARM 64-bit
aarch64-linux-gnu-gcc -o prog main.c
# ARM 32-bit
arm-linux-gnueabihf-gcc -o prog main.c
# Clang cross-compile
clang --target=aarch64-linux-gnu -o prog main.c
# Windows cross-compile (MinGW)
x86_64-w64-mingw32-gcc -o prog.exe main.c
```

---

## Python Development (venv, pip, ruff, mypy, pytest)

### pip

```bash
# Install packages
pip install package                   # Install latest
pip install package==2.1.0            # Install specific version
pip install 'package>=2.0,<3.0'      # Version range
pip install 'package[extra]'         # Install with extras
pip install -e .                      # Editable install (development)
pip install -e '.[dev,test]'         # Editable with extras

# From files
pip install -r requirements.txt       # Install from requirements
pip install -c constraints.txt -r requirements.txt  # With constraints
pip install --no-cache-dir -r requirements.txt      # Skip cache

# Upgrade
pip install --upgrade package         # Upgrade to latest
pip install --upgrade pip setuptools wheel  # Upgrade pip itself

# Inspect
pip show package                      # Package details and location
pip list                              # All installed packages
pip list --outdated                   # Packages with newer versions
pip list --format=json | jq '.'      # JSON output
pip freeze > requirements.txt         # Export installed versions
pip index versions package            # Available versions on PyPI

# Uninstall
pip uninstall package                 # Remove package
pip uninstall -y -r requirements.txt  # Remove all from file
```

### uv (Fast Python Package Manager)

```bash
# Virtual environment
uv venv                               # Create .venv in current directory
uv venv --python 3.12                 # Specify Python version
uv venv myenv                         # Custom name

# Install packages (pip-compatible, 10-100x faster)
uv pip install package                # Install package
uv pip install -r requirements.txt    # Install from requirements
uv pip install -e '.[dev]'           # Editable install

# Lock and sync (reproducible environments)
uv pip compile requirements.in -o requirements.txt  # Resolve and lock
uv pip sync requirements.txt          # Install exact versions (remove extras)

# Run scripts
uv run script.py                      # Run with auto-resolved deps
uv run --with requests script.py      # Run with additional dependency

# Tool management
uv tool install ruff                  # Install CLI tool globally
uv tool install --from 'ruff==0.4.0' ruff  # Specific version
uv tool list                          # List installed tools
uv tool run ruff check .              # Run tool without installing
```

### poetry

```bash
# Project management
poetry init                           # Initialize new project interactively
poetry new myproject                  # Create new project with structure

# Dependencies
poetry add package                    # Add production dependency
poetry add --group dev pytest         # Add dev dependency
poetry add 'package>=2.0,<3.0'      # Version constraint
poetry remove package                 # Remove dependency
poetry update                         # Update all dependencies
poetry update package                 # Update specific package

# Install and sync
poetry install                        # Install all dependencies
poetry install --no-root              # Install deps without project itself
poetry install --only dev             # Install only dev group
poetry install --sync                 # Sync: remove packages not in lock

# Lock file
poetry lock                           # Regenerate lock file
poetry lock --no-update               # Lock without updating versions

# Run and environment
poetry run pytest                     # Run command in venv
poetry run python script.py           # Run script in venv
poetry shell                          # Activate venv shell
poetry env info                       # Show environment details
poetry env list                       # List all envs for project

# Export
poetry export -f requirements.txt --output requirements.txt          # Export
poetry export -f requirements.txt --without-hashes --with dev        # With dev deps

# Build and publish
poetry build                          # Build sdist + wheel
poetry publish                        # Publish to PyPI
poetry publish --build                # Build and publish
```

### pdm

```bash
pdm init                              # Initialize project
pdm add package                       # Add dependency
pdm add -dG test pytest               # Add to test group
pdm install                           # Install all dependencies
pdm run script                        # Run script from pyproject.toml
pdm update                            # Update dependencies
pdm list                              # List installed packages
pdm export -f requirements -o requirements.txt  # Export
```

### pyenv

```bash
pyenv install --list                  # List available versions
pyenv install 3.12.4                  # Install specific version
pyenv versions                        # List installed versions
pyenv global 3.12.4                   # Set global default
pyenv local 3.12.4                    # Set for current directory (.python-version)
pyenv shell 3.12.4                    # Set for current shell session
pyenv which python                    # Show path to active python
pyenv uninstall 3.11.0                # Remove version
```

### virtualenv / venv

```bash
# Create virtual environment
python -m venv .venv                  # Create in .venv directory
python -m venv --clear .venv          # Recreate from scratch
python -m venv --system-site-packages .venv  # Include system packages

# Activate/deactivate
source .venv/bin/activate             # Activate (bash/zsh)
deactivate                            # Deactivate

# Check active environment
which python                          # Should point to .venv/bin/python
python -c "import sys; print(sys.prefix)"  # Verify venv active
```

### ruff (Linter + Formatter)

```bash
# Linting
ruff check .                          # Lint all files
ruff check src/                       # Lint specific directory
ruff check --fix .                    # Auto-fix safe issues
ruff check --fix --unsafe-fixes .     # Auto-fix including unsafe
ruff check --select E,W,F .          # Only specific rule categories
ruff check --ignore E501 .           # Ignore specific rules
ruff check --diff .                   # Show what would change
ruff check --output-format=json .    # JSON output
ruff check --statistics .            # Show error counts by rule
ruff check --watch .                 # Watch mode (re-lint on change)

# Formatting
ruff format .                         # Format all files
ruff format src/                      # Format specific directory
ruff format --check .                 # Dry run: check if formatted
ruff format --diff .                  # Show what would change
ruff format --line-length 120 .      # Override line length

# Rule info
ruff rule E501                        # Explain specific rule
ruff linter                           # List available linters
```

#### pyproject.toml config for ruff:

```toml
[tool.ruff]
line-length = 88
target-version = "py312"
src = ["src"]

[tool.ruff.lint]
select = ["E", "F", "W", "I", "N", "UP", "B", "A", "C4", "SIM"]
ignore = ["E501"]
fixable = ["ALL"]

[tool.ruff.lint.per-file-ignores]
"tests/*" = ["S101"]  # Allow assert in tests

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

### black (Formatter)

```bash
black .                               # Format all files
black --check .                       # Dry run: check if formatted
black --diff .                        # Show what would change
black --line-length 120 .            # Custom line length
black --target-version py312 .       # Target Python version
black --include '\.pyi?$' .          # Include pattern
black --extend-exclude '/migrations/' .  # Exclude pattern
```

### isort (Import Sorting)

```bash
isort .                               # Sort imports in all files
isort --profile black .              # Use black-compatible profile
isort --check-only .                 # Dry run: check if sorted
isort --diff .                       # Show what would change
isort --atomic .                     # Only write if no syntax errors
```

### mypy (Type Checker)

```bash
mypy .                                # Type check current directory
mypy src/                             # Type check specific path
mypy --strict .                       # Strict mode (all optional checks)
mypy --ignore-missing-imports .      # Ignore missing stubs
mypy --show-error-codes .            # Show error codes (e.g. [attr-defined])
mypy --no-implicit-reexport .        # Strict re-export checking
mypy --disallow-untyped-defs .       # Require type annotations
mypy --check-untyped-defs .          # Check bodies of untyped functions
mypy --html-report htmlcov .         # HTML coverage report

# Install stubs for third-party libraries
pip install types-requests types-pyyaml types-setuptools
mypy --install-types .               # Auto-install missing stubs
```

#### mypy.ini / pyproject.toml config:

```toml
[tool.mypy]
python_version = "3.12"
strict = true
warn_return_any = true
warn_unused_configs = true

[[tool.mypy.overrides]]
module = "third_party.*"
ignore_missing_imports = true
```

### pyright (Type Checker)

```bash
pyright                               # Type check project
pyright src/                          # Type check specific path
pyright --level basic                 # Basic checking
pyright --level standard              # Standard checking (default)
pyright --level strict                # Strict checking
pyright --outputjson                  # JSON output
pyright --verifytypes package         # Verify type completeness of a package
pyright --watch                       # Watch mode
```

#### pyrightconfig.json:

```json
{
  "include": ["src"],
  "exclude": ["**/node_modules", "**/__pycache__"],
  "venvPath": ".",
  "venv": ".venv",
  "reportMissingImports": true,
  "reportMissingTypeStubs": false,
  "pythonVersion": "3.12",
  "typeCheckingMode": "standard"
}
```

### bandit (Security Linter)

```bash
bandit -r src/                        # Scan recursively
bandit -r src/ -ll                    # Only medium+ severity
bandit -r src/ -s B101               # Skip assert warning
bandit -r src/ --format json         # JSON output
bandit -r src/ -c pyproject.toml     # Use config file
bandit -r src/ --severity-level high # Only high severity
```

### pytest

```bash
# Running tests
pytest                                # Run all tests
pytest tests/                         # Run tests in directory
pytest tests/test_foo.py              # Run specific file
pytest tests/test_foo.py::test_bar   # Run specific test
pytest -k "test_name"                # Filter by name pattern
pytest -k "test_foo and not slow"    # Complex filter

# Output and verbosity
pytest -v                             # Verbose: show each test name
pytest -vv                            # More verbose: show diffs
pytest -s                             # Show print/stdout output
pytest --tb=short                     # Short tracebacks
pytest --tb=long                      # Full tracebacks
pytest --tb=no                        # No tracebacks
pytest -q                             # Quiet: minimal output

# Failure handling
pytest -x                             # Stop on first failure
pytest --maxfail=3                   # Stop after 3 failures
pytest --lf                           # Re-run last failed tests only
pytest --ff                           # Run failed first, then rest
pytest --sw                           # Stepwise: stop on fail, resume next run

# Coverage
pytest --cov=src                      # Coverage for src/
pytest --cov=src --cov-report=html   # HTML report in htmlcov/
pytest --cov=src --cov-report=term-missing  # Show missing lines
pytest --cov=src --cov-fail-under=80 # Fail if coverage < 80%

# Parallel execution (requires pytest-xdist)
pytest -n auto                        # Use all CPU cores
pytest -n 4                           # Use 4 workers
pytest -n auto --dist loadscope      # Group by module

# Markers
pytest -m slow                        # Run only @pytest.mark.slow tests
pytest -m "not slow"                 # Skip slow tests
pytest --markers                      # List available markers

# Fixtures and debugging
pytest --fixtures                     # List available fixtures
pytest --setup-show                  # Show fixture setup/teardown
pytest --pdb                          # Drop to debugger on failure
pytest --pdb-trace                   # Drop to debugger at start of each test
```

---

## Node.js Development (npm, eslint, tsc, jest)

### npm

```bash
# Install dependencies
npm install                           # Install from package.json
npm ci                                # Clean install (exact versions from lock)
npm install package                   # Add production dependency
npm install --save-dev package        # Add dev dependency
npm install --save-exact package      # Pin exact version
npm install -g package                # Install globally

# Scripts
npm run build                         # Run build script
npm run test                          # Run test script
npm start                             # Run start script (shortcut)
npm test                              # Run test script (shortcut)

# Package info
npm ls --depth 0                      # List top-level dependencies
npm ls --all                          # List all dependencies
npm outdated                          # Show outdated packages
npm view package versions            # Available versions
npm explain package                  # Why is package installed?

# Security
npm audit                             # Check for vulnerabilities
npm audit fix                         # Auto-fix vulnerabilities
npm audit fix --force                 # Force fix (may have breaking changes)

# Maintenance
npm update                            # Update to latest allowed by semver
npm prune                             # Remove extraneous packages
npm cache clean --force              # Clear cache
npm dedupe                            # Reduce duplicate packages

# Publishing
npm version major|minor|patch        # Bump version
npm pack                              # Create tarball
npm publish                           # Publish to registry
npm link                              # Link local package for development

# npx (run without installing)
npx create-react-app myapp           # Run package binary
npx -y package                       # Auto-confirm install
npx package@version                  # Specific version

# Workspaces
npm install -w packages/shared       # Install in workspace
npm run build -w packages/app        # Run script in workspace
npm run build --workspaces           # Run in all workspaces
```

### pnpm

```bash
# Install
pnpm install                          # Install all deps
pnpm add package                      # Add production dep
pnpm add -D package                   # Add dev dep
pnpm add -g package                   # Global install

# Run
pnpm run build                        # Run script
pnpm exec tsc                         # Run binary
pnpm dlx create-vite myapp           # One-off execution (like npx)

# Workspace
pnpm --filter package-name build     # Run in specific package
pnpm -r build                         # Run in all workspace packages
pnpm --filter "package-name..." build  # Package and its dependencies

# Maintenance
pnpm store prune                      # Clean unused packages from store
pnpm import                           # Import from npm/yarn lockfile
pnpm update                           # Update dependencies
pnpm outdated                         # Show outdated packages
```

### yarn (Classic & Berry)

```bash
# Classic (v1)
yarn install                          # Install all deps
yarn add package                      # Add production dep
yarn add -D package                   # Add dev dep
yarn run build                        # Run script
yarn upgrade package                  # Update package

# Berry (v2+)
yarn set version stable               # Upgrade to latest stable
yarn install                          # Install (uses Plug'n'Play by default)
yarn add package                      # Add dep
yarn dlx create-vite myapp           # One-off execution
yarn workspaces foreach run build    # Run in all workspaces
```

### bun

```bash
# Install
bun install                           # Install all deps (faster than npm)
bun add package                       # Add production dep
bun add -d package                    # Add dev dep

# Run
bun run build                         # Run script
bun run index.ts                      # Run TypeScript directly
bun test                              # Run tests (built-in test runner)

# Build
bun build ./src/index.ts --outdir ./dist  # Bundle
bun build --compile ./src/index.ts   # Compile to standalone binary

# Create
bun create vite myapp                 # Create from template
```

### deno

```bash
# Run with permissions
deno run script.ts                    # Run (no permissions)
deno run --allow-net script.ts       # Allow network access
deno run --allow-read script.ts      # Allow file reads
deno run --allow-all script.ts       # Allow everything
deno run -A script.ts                # Shorthand for --allow-all

# Tools
deno test                             # Run tests
deno lint                             # Lint code
deno fmt                              # Format code
deno bench                            # Run benchmarks
deno compile script.ts               # Compile to binary
deno task build                       # Run task from deno.json

# Dependencies
deno cache deps.ts                    # Cache dependencies
deno info script.ts                   # Show dependency tree
```

### volta / nvm

```bash
# volta (fast, manages node/npm/yarn versions per project)
volta install node@20                 # Install Node 20
volta install npm@10                  # Install npm 10
volta pin node@20                     # Pin version in package.json
volta pin npm@10                      # Pin npm version
volta list                            # List installed tools
volta which node                      # Show active node path

# nvm
nvm install 20                        # Install Node 20
nvm install --lts                     # Install latest LTS
nvm use 20                            # Switch to Node 20
nvm use --lts                         # Use latest LTS
nvm alias default 20                  # Set default version
nvm ls                                # List installed versions
nvm ls-remote --lts                  # List available LTS versions
nvm current                           # Show current version
# .nvmrc file: just put version number (e.g. "20") in project root
```

### eslint

```bash
# Lint
eslint .                              # Lint all files
eslint src/                           # Lint specific directory
eslint --fix .                        # Auto-fix issues
eslint --fix-dry-run .               # Preview fixes
eslint --ext .js,.ts .               # Specify extensions

# Rules and config
eslint --rule '{"semi": "error"}' .  # Override rule
eslint --ignore-pattern "dist/" .    # Ignore pattern
eslint --max-warnings 0 .           # Fail on any warning
eslint --format stylish .            # Output format
eslint --print-config file.js        # Show effective config for file
```

#### eslint.config.js (flat config, ESLint 9+):

```javascript
import js from "@eslint/js";
import tseslint from "typescript-eslint";

export default [
  js.configs.recommended,
  ...tseslint.configs.recommended,
  {
    rules: {
      "no-unused-vars": "warn",
      semi: "error",
    },
  },
  {
    files: ["tests/**"],
    rules: {
      "no-unused-expressions": "off",
    },
  },
];
```

### prettier

```bash
# Format
prettier --write .                    # Format all files in place
prettier --write "src/**/*.{ts,tsx}" # Format specific pattern
prettier --check .                    # Check if formatted (CI)
prettier --list-different .          # List unformatted files

# Options
prettier --single-quote --write .    # Single quotes
prettier --trailing-comma all --write .  # Trailing commas
prettier --tab-width 4 --write .     # 4-space indent
prettier --print-width 100 --write . # Line width
prettier --no-semi --write .         # No semicolons

# Plugins
prettier --plugin prettier-plugin-tailwindcss --write .
prettier --plugin prettier-plugin-organize-imports --write .
```

#### .prettierrc:

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2
}
```

### tsc (TypeScript Compiler)

```bash
# Type checking (no output)
tsc --noEmit                          # Type check only
tsc --noEmit --watch                 # Watch mode

# Compilation
tsc                                   # Compile using tsconfig.json
tsc --outDir dist                    # Output to directory
tsc --target es2022 --module esnext  # Specify target and module
tsc --declaration                    # Generate .d.ts files
tsc --declarationMap                 # Generate .d.ts.map files
tsc --sourceMap                      # Generate .js.map files

# Project info
tsc --showConfig                      # Show effective tsconfig
tsc --listFiles                       # List files that would be compiled
tsc --init                            # Create tsconfig.json

# Build mode (project references)
tsc --build                           # Build project and dependencies
tsc --build --clean                  # Clean build outputs
tsc --build --force                  # Force rebuild
tsc --build --watch                  # Watch mode with build
```

#### Key tsconfig.json options:

```json
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "outDir": "dist",
    "declaration": true,
    "sourceMap": true,
    "paths": { "@/*": ["./src/*"] },
    "baseUrl": "."
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

### esbuild

```bash
# Bundle
esbuild src/index.ts --bundle --outfile=dist/bundle.js
esbuild src/index.ts --bundle --outdir=dist

# Platform and format
esbuild src/index.ts --bundle --platform=node --format=cjs --outfile=dist/index.cjs
esbuild src/index.ts --bundle --platform=browser --format=esm --outfile=dist/index.mjs

# Options
esbuild src/index.ts --bundle --minify --sourcemap --outfile=dist/bundle.js
esbuild src/index.ts --bundle --target=es2020 --outfile=dist/bundle.js
esbuild src/index.ts --bundle --define:process.env.NODE_ENV=\"production\" --outfile=dist/bundle.js
esbuild src/index.ts --bundle --loader:.svg=file --outdir=dist
esbuild src/index.ts --bundle --external:react --external:react-dom --outfile=dist/bundle.js

# Serve (dev server)
esbuild src/index.ts --bundle --serve=8000 --outdir=dist
esbuild src/index.ts --bundle --serve --servedir=public --outdir=public/dist

# Watch
esbuild src/index.ts --bundle --watch --outdir=dist
```

### vite

```bash
vite                                  # Start dev server
vite build                            # Production build
vite preview                          # Preview production build
vite --port 3000                     # Custom port
vite --host                          # Expose to network
vite build --mode staging            # Build with custom mode
```

---

## Shell Script Development (shellcheck, shfmt)

### shellcheck

```bash
# Basic linting
shellcheck script.sh                  # Lint single file
shellcheck *.sh                       # Lint all shell files
shellcheck -x script.sh              # Follow source'd files

# Shell dialect
shellcheck -s bash script.sh         # Bash dialect
shellcheck -s sh script.sh           # POSIX sh dialect
shellcheck -s dash script.sh         # Dash dialect
shellcheck -s ksh script.sh          # Ksh dialect

# Severity filtering
shellcheck -S error script.sh        # Only errors
shellcheck -S warning script.sh      # Warnings and above
shellcheck -S info script.sh         # Info and above
shellcheck -S style script.sh        # All (default)

# Exclude/include rules
shellcheck -e SC2086 script.sh       # Exclude specific code
shellcheck -e SC2086,SC2046 script.sh  # Exclude multiple
shellcheck -i SC2086 script.sh       # Include only specific code

# Output format
shellcheck -f gcc script.sh          # GCC-compatible (for editors)
shellcheck -f json script.sh         # JSON output
shellcheck -f diff script.sh         # Diff output (for patching)
shellcheck -f checkstyle script.sh   # Checkstyle XML

# In-script directives
# shellcheck disable=SC2086           # Disable for next line
# shellcheck disable=SC2086,SC2046    # Disable multiple
# shellcheck source=./lib.sh          # Tell SC where source files are
```

#### Common shellcheck codes:

| Code | Issue | Fix |
|------|-------|-----|
| SC2086 | Unquoted variable | `"$var"` |
| SC2046 | Unquoted command substitution | `"$(cmd)"` |
| SC2164 | `cd` without `\|\| exit` | `cd dir \|\| exit 1` |
| SC2006 | Backticks instead of `$()` | `$(command)` |
| SC2034 | Unused variable | Remove or export |
| SC1090 | Can't follow source | Add `# shellcheck source=file` |
| SC2155 | Declare and assign separately | `local var; var=$(cmd)` |
| SC2162 | `read` without `-r` | `read -r var` |
| SC2068 | Unquoted `$@` | `"$@"` |
| SC2115 | `rm -rf "$var/"` danger | Verify var is set first |

### shfmt

```bash
# Format
shfmt -w script.sh                    # Format in place
shfmt -w *.sh                         # Format all shell files
shfmt -w .                            # Format all shell files recursively

# Indent style
shfmt -i 2 -w script.sh              # 2-space indent
shfmt -i 4 -w script.sh              # 4-space indent
shfmt -i 0 -w script.sh              # Tab indent (default)

# Style options
shfmt -ci -w script.sh               # Indent case labels
shfmt -bn -w script.sh               # Binary ops at start of line
shfmt -s -w script.sh                # Simplify code
shfmt -mn -w script.sh               # Minify

# Check / diff
shfmt -d script.sh                    # Show diff
shfmt -l .                            # List files that differ
shfmt -d .                            # Diff all files

# Language
shfmt -ln bash script.sh             # Bash
shfmt -ln posix script.sh            # POSIX sh
shfmt -ln mksh script.sh             # MirBSD Korn Shell
```

---

## Go

### go build / test / run

```bash
# Build
go build ./...                        # Build all packages
go build -o myapp ./cmd/app           # Build with output name
go build -v ./...                     # Verbose (show packages)
go build -race ./...                  # Enable race detector
go build -ldflags '-s -w' ./...      # Strip debug info (smaller binary)
go build -tags integration ./...     # Build with tags

# Cross-compile
GOOS=linux GOARCH=amd64 go build -o myapp-linux ./cmd/app
GOOS=darwin GOARCH=arm64 go build -o myapp-mac ./cmd/app
GOOS=windows GOARCH=amd64 go build -o myapp.exe ./cmd/app

# Test
go test ./...                         # Test all packages
go test -v ./...                      # Verbose
go test -run TestName ./...          # Run matching tests
go test -run TestName/SubTest ./...  # Run subtest
go test -race ./...                   # Race detector
go test -count=1 ./...               # No caching
go test -timeout 30s ./...           # Set timeout
go test -shuffle=on ./...            # Randomize order
go test -short ./...                 # Skip long tests

# Benchmarks
go test -bench=. ./...               # Run all benchmarks
go test -bench=BenchmarkName ./...   # Specific benchmark
go test -bench=. -benchmem ./...     # Include memory stats
go test -bench=. -count=5 ./...      # Multiple runs

# Coverage
go test -cover ./...                  # Show coverage percentages
go test -coverprofile=coverage.out ./...   # Generate coverage profile
go tool cover -html=coverage.out     # Open HTML coverage report
go tool cover -func=coverage.out     # Show per-function coverage

# Vet (static analysis)
go vet ./...                          # Run all vet checks
```

### go mod (Module Management)

```bash
go mod init module-name               # Initialize module
go mod tidy                           # Add missing, remove unused deps
go mod download                       # Download dependencies
go mod vendor                         # Vendor dependencies
go mod graph                          # Print dependency graph
go mod why package                    # Explain why package is needed
go mod edit -replace=old=new@v1.0    # Replace module
go mod edit -dropreplace=old         # Remove replacement
go get package@latest                 # Add/update dependency
go get package@v1.2.3                 # Specific version
go get -u ./...                       # Update all dependencies
```

### golangci-lint

```bash
golangci-lint run                     # Run all enabled linters
golangci-lint run ./...               # Run on all packages
golangci-lint run --fix              # Auto-fix issues
golangci-lint run --enable-all       # Enable all linters
golangci-lint run --disable errcheck # Disable specific linter
golangci-lint run --new-from-rev HEAD~1  # Only new issues since commit
golangci-lint linters                 # List all available linters

# Common linters: errcheck, gosimple, govet, ineffassign, staticcheck,
# unused, bodyclose, gocritic, gosec, prealloc, revive
```

### go fmt / goimports

```bash
go fmt ./...                          # Format all Go files
goimports -w .                        # Format + manage imports
goimports -l .                        # List files needing formatting
```

---

## Rust

### cargo

```bash
# Build
cargo build                           # Debug build
cargo build --release                 # Release build (optimized)
cargo build --target x86_64-unknown-linux-musl  # Cross-compile

# Run
cargo run                             # Build and run
cargo run --release                   # Release mode run
cargo run -- arg1 arg2               # Pass arguments

# Test
cargo test                            # Run all tests
cargo test test_name                  # Run matching tests
cargo test -- --nocapture            # Show println! output
cargo test -- --test-threads=1       # Single threaded
cargo test --doc                      # Run doc tests only

# Check and lint
cargo check                           # Fast syntax/type check (no codegen)
cargo clippy                          # Lint with clippy
cargo clippy --fix                    # Auto-fix clippy suggestions
cargo clippy -- -W clippy::all       # Warn on all clippy lints
cargo clippy -- -D clippy::unwrap_used  # Deny unwrap usage

# Format
cargo fmt                             # Format all code
cargo fmt --check                     # Check if formatted

# Docs
cargo doc                             # Build documentation
cargo doc --open                      # Build and open in browser
cargo doc --no-deps                  # Skip dependency docs

# Dependencies
cargo add package                     # Add dependency
cargo add package --dev               # Add dev dependency
cargo add package --features feat1    # Add with features
cargo remove package                  # Remove dependency
cargo update                          # Update dependencies
cargo tree                            # Show dependency tree
cargo tree -d                         # Show duplicate dependencies
cargo audit                           # Check for security advisories

# Workspace
cargo build --workspace              # Build all workspace members
cargo test --workspace               # Test all workspace members
cargo build -p package-name          # Build specific package

# Publishing
cargo publish                         # Publish to crates.io
cargo publish --dry-run              # Verify without publishing
cargo package                         # Create .crate file

# Benchmarks
cargo bench                           # Run benchmarks
cargo bench bench_name               # Run matching benchmarks

# Features
cargo build --features feat1,feat2   # Enable features
cargo build --no-default-features    # Disable default features
cargo build --all-features           # Enable all features
```

### rustup

```bash
rustup default stable                 # Use stable toolchain
rustup default nightly               # Use nightly toolchain
rustup update                         # Update all toolchains
rustup component add rustfmt         # Add rustfmt
rustup component add clippy          # Add clippy
rustup component add rust-analyzer   # Add LSP server
rustup target add wasm32-unknown-unknown  # Add WASM target
rustup target add x86_64-unknown-linux-musl  # Add musl target
rustup show                           # Show installed toolchains
```

---

## Linter Comparison Table

| Language | Linter | Formatter | Type Checker | Security |
|----------|--------|-----------|-------------|----------|
| Python | ruff | ruff format | pyright / mypy | bandit |
| JavaScript | eslint | prettier | - | npm audit |
| TypeScript | eslint + typescript-eslint | prettier | tsc | npm audit |
| Shell | shellcheck | shfmt | - | - |
| Go | golangci-lint | gofmt / goimports | go vet | gosec |
| Rust | clippy | rustfmt | cargo check | cargo audit |
| Dockerfile | hadolint | - | - | - |
| YAML | yamllint | prettier | - | - |
| GitHub Actions | actionlint | - | - | - |
| CSS | stylelint | prettier | - | - |
| SQL | sqlfluff | sqlfluff fix | - | - |
| Markdown | markdownlint | prettier | - | - |

### hadolint (Dockerfile Linter)

```bash
hadolint Dockerfile                   # Lint Dockerfile
hadolint --ignore DL3008 Dockerfile  # Ignore specific rule
hadolint --format json Dockerfile    # JSON output
hadolint -t error Dockerfile         # Only errors
cat Dockerfile | hadolint -           # Lint from stdin
```

### yamllint

```bash
yamllint .                            # Lint all YAML files
yamllint -d relaxed file.yaml        # Use relaxed config
yamllint -d "{extends: default, rules: {line-length: {max: 120}}}" .
yamllint -f parsable file.yaml       # Machine-readable output
```

### actionlint

```bash
actionlint                            # Lint all GitHub Actions workflows
actionlint .github/workflows/ci.yml  # Lint specific workflow
actionlint -format '{{json .}}'     # JSON output
```

---

### Watch / Live Reload

```bash
# entr: rerun command when files change
ls *.py | entr pytest                          # Rerun tests on Python file change
ls *.py | entr -r python app.py                # Restart app on change (-r kills previous)
fdfind -e py | entr -r python app.py           # fd + entr combo
fdfind -e py | entr -c pytest                  # Clear screen before each run (-c)

# watchexec: modern file watcher
watchexec -e py -- pytest                      # Watch .py files, run pytest
watchexec -e py,html -- python manage.py runserver  # Watch multiple extensions
watchexec -r -e rs -- cargo run                # Restart on Rust file change

# nodemon (Node.js)
nodemon app.js                                 # Auto-restart on change
nodemon --ext py --exec "python app.py"        # Use with non-Node files

# cargo watch (Rust)
cargo watch -x test                            # Run tests on change
cargo watch -x 'run -- --port 8080'            # Run with args on change
```

### Pre-commit Framework

```bash
# Install and setup
pip install pre-commit
pre-commit install                             # Install hooks in current repo
pre-commit install --hook-type commit-msg      # Also validate commit messages

# Run hooks
pre-commit run --all-files                     # Run all hooks on all files
pre-commit run ruff --all-files                # Run specific hook
pre-commit autoupdate                          # Update hook versions

# Sample .pre-commit-config.yaml
# repos:
#   - repo: https://github.com/astral-sh/ruff-pre-commit
#     hooks: [{id: ruff, args: [--fix]}, {id: ruff-format}]
#   - repo: https://github.com/pre-commit/mirrors-mypy
#     hooks: [{id: mypy}]
#   - repo: https://github.com/pre-commit/pre-commit-hooks
#     hooks: [{id: trailing-whitespace}, {id: end-of-file-fixer}]
```

### Inline Debugging

```bash
# Python (pdb)
python3 -m pdb script.py                       # Start in debugger
# In code: breakpoint()                        # Drop into pdb at this line
# pdb commands: n (next), s (step), c (continue), p expr, bt (backtrace), l (list)

# Node.js
node --inspect app.js                          # Start with debugger (Chrome DevTools)
node --inspect-brk app.js                      # Break at first line
node inspect app.js                            # Built-in CLI debugger
```

### Dependency Vulnerability Scanning

```bash
# Python
pip-audit                                      # Scan installed packages
pip-audit -r requirements.txt                  # Scan requirements file
safety check                                   # Alternative scanner

# Node.js
npm audit                                      # Scan for vulnerabilities
npm audit --json | jq '.vulnerabilities | to_entries[] | {name: .key, severity: .value.severity}'
npm audit fix                                  # Auto-fix where possible

# Rust
cargo audit                                    # Scan Cargo.lock

# Multi-language
trivy fs .                                     # Scan filesystem for vuln deps
```

### Combined Workflows

```bash
# Lint + format pipeline
ruff check --fix . && ruff format . && mypy .

# Find tests affected by recent changes
git diff --name-only HEAD~1 | grep '.py$' | xargs pytest --co -q

# Bulk lint shell scripts
fdfind -e sh | xargs shellcheck

# Outdated deps report
pip list --outdated --format=json | jq -r '.[] | "\(.name): \(.version) → \(.latest_version)"'

# Watch + lint combo
fdfind -e py | entr -c sh -c 'ruff check . && mypy .'
```

---

## Installation

```bash
# Build systems
sudo apt install make cmake ninja-build
cargo install just                     # Or: sudo apt install just

# C/C++ toolchain
sudo apt install gcc g++ clang lld pkg-config
sudo apt install gcc-aarch64-linux-gnu  # ARM cross-compiler

# Python tools
pip install ruff black isort mypy pyright bandit pytest pytest-cov pytest-xdist
pip install uv                         # Or: curl -LsSf https://astral.sh/uv/install.sh | sh
pip install poetry                     # Or: curl -sSL https://install.python-poetry.org | python3 -

# Node.js tools
npm install -g eslint prettier typescript
npm install -g pnpm                   # Or: corepack enable pnpm
curl -fsSL https://bun.sh/install | bash  # Bun

# Shell tools
sudo apt install shellcheck
go install mvdan.cc/sh/v3/cmd/shfmt@latest  # Or: sudo snap install shfmt

# Go tools
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
go install golang.org/x/tools/cmd/goimports@latest

# Rust tools (included via rustup)
rustup component add clippy rustfmt rust-analyzer

# Container/config linters
sudo apt install hadolint || docker run --rm -i hadolint/hadolint < Dockerfile
pip install yamllint
go install github.com/rhysd/actionlint/cmd/actionlint@latest
```
