---
description: "Development tools - build failed, compilation error, dependency conflict, module not found, tests failing, linting errors, Make/CMake build systems, and package management (gcc, npm, pip, ruff, eslint, cargo)"
when_to_use: "Use when: build failed or won't compile, compilation error, dependency conflict or version mismatch, module not found or import error, tests failing, linting errors or warnings, need to set up project environment, formatting code, type checking, managing packages. Triggers: build failed, compilation error, dependency conflict, module not found, tests failing, import error, version mismatch, can't install package, build broken, make, build, compile, lint, format, shellcheck, eslint, ruff, prettier, gcc, npm, pip, python, node, cargo, go, cmake, just, poetry, uv, mypy, pyright, tsc, type check, pre-commit, virtual environment, outdated packages, vulnerability scan, pip-audit, bandit, hadolint, actionlint, code coverage."
when-to-use: "Use when: build failed or won't compile, compilation error, dependency conflict or version mismatch, module not found or import error, tests failing, linting errors or warnings, need to set up project environment, formatting code, type checking, managing packages. Triggers: build failed, compilation error, dependency conflict, module not found, tests failing, import error, version mismatch, can't install package, build broken, make, build, compile, lint, format, shellcheck, eslint, ruff, prettier, gcc, npm, pip, python, node, cargo, go, cmake, just, poetry, uv, mypy, pyright, tsc, type check, pre-commit, virtual environment, outdated packages, vulnerability scan, pip-audit, bandit, hadolint, actionlint, code coverage."
argument-hint: "[describe the development task]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Development Tools Task Router

You have access to the **full conversation context**. Your job is to analyze the user's dev tool need and build a well-formed argument for the recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Dev tool requests depend on language, project setup, and toolchain. Analyze conversation to identify:

### 1. Goal
What development operation is needed?
- Build project (which build system?)
- Compile code (language, flags, optimization?)
- Lint code (which linter, fix or check?)
- Format code (which formatter?)
- Type check (mypy, pyright, tsc?)
- Manage packages/dependencies
- Set up environment (venv, node version)

### 2. Language / Project
What language ecosystem?
- Python (pip, uv, poetry, pdm, ruff, mypy)
- Node.js (npm, pnpm, yarn, bun, eslint, prettier, tsc)
- C/C++ (gcc, clang, cmake, make)
- Go (go build, test, vet)
- Rust (cargo)
- Shell (shellcheck, shfmt)
- Multi-language project?

### 3. Build Context
- Build system in use? (Makefile, CMakeLists.txt, justfile, package.json scripts)
- Target platform?
- Debug vs release?
- Specific error from build?

### 4. Prior Findings
- Build errors from conversation
- Lint warnings discussed
- Dependency conflicts

---

## Goal --> Tool Guide

| Goal | Primary Tool | Argument Prefix |
|------|-------------|----------------|
| Build project | make/cmake/just | "build with" |
| Compile C/C++ | gcc/clang | "compile" |
| Lint Python | ruff | "lint python" |
| Format Python | ruff format/black | "format python" |
| Type check Python | pyright/mypy | "type check python" |
| Python packages | pip/uv/poetry | "manage python packages" |
| Python env | venv/pyenv | "setup python env" |
| Lint JS/TS | eslint | "lint javascript" |
| Format JS/TS | prettier | "format javascript" |
| Type check TS | tsc | "type check typescript" |
| Node packages | npm/pnpm/bun | "manage node packages" |
| Lint shell | shellcheck | "lint shell" |
| Format shell | shfmt | "format shell" |
| Go build/test | go | "go" |
| Rust build/test | cargo | "cargo" |
| Lint Dockerfile | hadolint | "lint dockerfile" |

---

## Argument Format

```
<goal> <target> - language: <lang>, tool: <if specific>, context: <details>
```

### Examples

**Build with make:**
```
build with make - parallel 8 jobs, target: install, debug mode
```

**Python linting:**
```
lint python src/ - fix auto-fixable issues, ignore E501 line length
```

**Compile C++ with sanitizers:**
```
compile main.cpp - C++17, address sanitizer, debug symbols, warnings as errors
```

**Node setup:**
```
manage node packages - install from package.json, using pnpm, frozen lockfile
```

**Shell linting:**
```
lint shell scripts in ./bin/ - show error codes, suggest fixes
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-dev-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Language/toolchain not evident from context
- Multiple build systems present (which one?)
- Lint rules unclear (strict or relaxed?)

**Use Read tool first if:**
- Need to check Makefile/CMakeLists.txt/package.json
- Need to inspect pyproject.toml for tool config
- Need to see existing lint config
