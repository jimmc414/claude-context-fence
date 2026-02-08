# Claude Code Skills Reference

**Version**: 2.6
**Claude Code Version**: 2.1.11 (Phase 8 tests)
**Date**: 2026-01-17
**Methodology**: Systematic testing of 175+ skill configurations + diagnostic introspection + binary analysis

---

## Executive Summary

Skills frontmatter is **metadata for the system**, NOT content for Claude. The model only receives:
1. The skill body content
2. Substituted `$ARGUMENTS`
3. Base directory path

All frontmatter fields are processed at the system level and never shown to Claude directly.

---

## What Claude Receives When a Skill Runs

### Passed to the Model

| Item | Example | Notes |
|------|---------|-------|
| **Skill body** | `# My Skill\nDo this task...` | Everything after the `---` closing frontmatter |
| **$ARGUMENTS** | `user-input-here` | Substituted where `$ARGUMENTS` appears, or appended to end |
| **Base directory** | `Base directory for this skill: /path/to/skill` | Prepended to skill body |

### NOT Passed to the Model (System-Only)

| Field | What It Does | Why Not Passed |
|-------|--------------|----------------|
| `name` | Display name in /skills | Routing metadata |
| `description` | Primary routing signal | LLM routing decision input |
| `when_to_use` | Additional routing hints | LLM routing decision input |
| `context` | Controls execution isolation | System execution config |
| `model` | Model selection | System execution config |
| `agent` | Agent type selection | System execution config |
| `allowed-tools` | (Doesn't work) | Parsed but not enforced |
| `mode` | (No effect) | Unknown purpose |
| `hooks` | (Don't fire) | Parsed but not executed |
| All others | Various | Metadata only |

---

## How `context: fork` Works

### Fork Execution Model

```
┌─────────────────────────────────────────────────────────────┐
│ Parent Session (your main Claude conversation)              │
│                                                             │
│  User: /my-skill                                            │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Forked Execution (separate context)                   │  │
│  │                                                       │  │
│  │ • CAN see full conversation history before fork       │  │
│  │ • Gets agent's system prompt & tools                  │  │
│  │ • Runs independently with own token count             │  │
│  │ • Returns full textual output to parent (Phase 6)     │  │
│  │                                                       │  │
│  │ Return: 'Skill "X" completed (forked execution).'     │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  Claude: The skill completed. Here's what it found...       │
└─────────────────────────────────────────────────────────────┘
```

### Fork Requirements

```yaml
# REQUIRED combination
context: fork
model: inherit  # Optional - defaults to inherit if omitted (Feedback Test verified)

# Optional but powerful
agent: Explore  # Or Plan - applies agent restrictions
```

**Feedback Test Finding**: Omitting `model:` entirely with `context: fork` works correctly - defaults to inherit behavior.

### Fork Behavior

| Aspect | Behavior |
|--------|----------|
| **Conversation history** | Forked agent CAN see all prior messages |
| **Token counting** | Separate token budget |
| **Return to parent** | Full textual output (Phase 6 verified) |
| **Tool access** | Controlled by agent type |
| **Model used** | Must be `inherit` (session model) |

---

## Built-In Agent Types

### All Agent Types (9 identified through testing)

```
agentType:""              # Empty - falls back to general-purpose
agentType:"Bash"          # Command execution specialist
agentType:"Explore"       # Read-only codebase exploration
agentType:"Plan"          # Architecture planning
agentType:"general-purpose"    # Full capabilities, default
agentType:"magic-docs"         # Internal only - NOT user-callable
agentType:"main-session"       # Parent session (works in skills!)
agentType:"statusline-setup"   # Terminal status line config
agentType:"subagent"           # Generic type (works in skills!)
```

### Agent Comparison Table

| Agent | Model | Tool Restrictions | Use Case |
|-------|-------|-------------------|----------|
| `Explore` | haiku | Write, Edit, Task, NotebookEdit, ExitPlanMode BLOCKED; **Skill ALLOWED** | Read-only codebase exploration |
| `Plan` | inherit | Same as Explore | Architecture planning |
| `Bash` | inherit | Bash only | Command execution |
| `general-purpose` | inherit | None (all 17 tools) | Default behavior |
| `statusline-setup` | sonnet | Read, Edit only | Status line configuration |
| `claude-code-guide` | haiku | Glob, Grep, Read, WebFetch, WebSearch | Documentation help |
| `magic-docs` | sonnet | Edit only | **Internal only - NOT user-callable** |
| `main-session` | inherit | None (all tools) | **Full access (Feedback Test verified)** |
| `subagent` | inherit | None (all tools) | **Full access (Feedback Test verified)** |
| `""` (empty) | inherit | None (all tools) | Falls back to general-purpose |

### Agent Details

#### Explore Agent
```yaml
agent: Explore
# Best for: Codebase search, file finding, code understanding
# Tools: Glob, Grep, Read, WebFetch, WebSearch, TodoWrite, Skill (Phase 6 verified)
# Blocked: Write, Edit, Task, NotebookEdit, ExitPlanMode
# Model: haiku (fast, cheap)
# System prompt includes: "CRITICAL: This is a READ-ONLY task"
# Note: Skill tool IS available (surprising - enables read-only delegation)
```

#### Plan Agent
```yaml
agent: Plan
# Best for: Architecture design, implementation planning
# Tools: Same as Explore
# Blocked: Same as Explore (read-only)
# Model: inherit (uses session model)
# System prompt includes: "You are a software architect"
```

#### Bash Agent
```yaml
agent: Bash
# Best for: Running commands, git operations
# Tools: ONLY Bash
# Model: inherit
# Note: More restrictive than expected - only Bash tool
```

#### general-purpose Agent
```yaml
agent: general-purpose
# Default agent when none specified
# Tools: All tools available
# Model: inherit
# No special restrictions
```

### Using Agents in Skills

```yaml
---
# Create a read-only exploration skill
agent: Explore
context: fork
model: inherit
---
Search the codebase for authentication implementations.
```

**Important**: `agent` only takes effect WITH `context: fork`. Without fork, the agent field has no effect.

**Model Override Behavior**: When using an agent with fork, the **agent's internal model setting overrides** your `model: inherit`:

| Agent | Your Setting | Actual Model Used |
|-------|--------------|-------------------|
| `Explore` | `model: inherit` | **Haiku** (agent's hardcoded model) |
| `Plan` | `model: inherit` | Session model (agent uses inherit) |
| `Bash` | `model: inherit` | Session model (agent uses inherit) |
| None | `model: inherit` | Session model |

This means `agent: Explore` skills always run on Haiku regardless of your session model (Opus, Sonnet, etc.). Use `agent: Plan` if you need your session model with read-only restrictions.

---

## Field Reference

### Working Fields

| Field | Syntax | Effect |
|-------|--------|--------|
| `name` | `name: My Skill` | Display name in /skills list |
| `description` | `description: Does X` | Primary routing signal for auto-selection |
| `when_to_use` | `when_to_use: Use when...` | **Underscore required** - appended to description |
| `context` | `context: fork` | Enables forked execution |
| `model` | `model: inherit` | **Only `inherit` works** |
| `agent` | `agent: Explore` | Applies agent restrictions (with fork) |
| `disable-model-invocation` | `disable-model-invocation: true` | **Blocks auto-routing AND Skill tool** (only manual `/skill` works) |

### Non-Working Fields (v2.1.5) → Fixed in v2.1.7

| Field | v2.1.5 | v2.1.7 |
|-------|--------|--------|
| `model: haiku` | 404 error | ✅ **NOW WORKS** |
| `model: sonnet` | 404 error | ✅ **NOW WORKS** |
| `model: opus` | 404 error | ✅ **NOW WORKS** (v2.5 verified) |
| `model: claude-*` (full ID) | 404 error | Untested (short names work) |
| `allowed-tools` | Restrict tools | No effect (cannot override agent) |
| `mode` | Unknown | No effect |
| `user-invocable: false` | Hide from /skills | Still visible |
| `hooks` | Fire on tool use | Parsed, never fire |
| `when-to-use` (hyphen) | Same as underscore | Silently ignored |
| `permissionMode` | Override permissions | No effect |

### Silently Ignored Fields (Phase 7 Verified)

These fields are accepted by the YAML parser but have **no effect**:

| Category | Fields | Notes |
|----------|--------|-------|
| **Routing** | `priority`, `weight`, `routing_priority` | No routing priority system |
| **Categorization** | `category`, `tags`, `group` | No filtering/grouping |
| **Context** | `context_match`, `file_types`, `requires`, `triggers` | No context-aware routing |
| **Execution** | `timeout`, `timeout_ms`, `max_duration`, `max_turns` | No execution limits |
| **Experimental** | `experimental`, `beta`, `internal` | No feature flags |
| **Debug** | `debug`, `verbose`, `trace`, `log_level` | No debug output |
| **Integration** | `mcp_servers`, `mcp`, `requires_mcp` | MCP config in settings.json only |
| **Settings** | `settings: {}`, `override_settings` | Cannot override settings |
| **Environment** | `env: {}`, `environment: {}` | Cannot inject env vars |

### Body-Only Substitution

**`$ARGUMENTS` only works in the body**, not frontmatter:

```yaml
---
name: Skill $ARGUMENTS     # ❌ Literal text, NOT substituted
description: For $ARGUMENTS # ❌ Literal text, NOT substituted
---
Process this: $ARGUMENTS    # ✅ SUBSTITUTED with user input
```

### Metadata-Only Fields

| Field | Notes |
|-------|-------|
| `argument-hint` | Parsed but no visible effect |
| `version` | Parsed but no visible effect |
| Custom fields | Silently ignored |

---

## Syntax Rules

### YAML Frontmatter Requirements

```yaml
---
# Frontmatter must start with exactly three dashes
field: value
another_field: another value
# And end with exactly three dashes
---
```

### Field Name Rules

| Rule | Example | Notes |
|------|---------|-------|
| Underscore vs hyphen | `when_to_use` ✅ `when-to-use` ❌ | Routing field requires underscore |
| Case sensitivity | `name` ✅ `Name` ❌ | Lowercase required |
| String values | `name: My Skill` | Quotes optional unless special chars |
| Boolean values | `disable-model-invocation: true` | Lowercase `true`/`false` |

### Array Syntax (Both Parsed, Neither Restricts)

```yaml
# String syntax
allowed-tools: Read,Write,Grep

# Array syntax
allowed-tools:
  - Read
  - Write
  - Grep

# Both parse correctly but neither restricts tools
```

### Argument Substitution

```yaml
---
name: My Skill
---
# Skill body

Do something with: $ARGUMENTS

# If $ARGUMENTS not present, arguments are appended to end
```

**Phase 5 Findings on $ARGUMENTS:**

| Syntax | Works? | Notes |
|--------|--------|-------|
| `$ARGUMENTS` | ✅ YES | Standard syntax |
| Multiple `$ARGUMENTS` | ✅ YES | All occurrences substituted |
| `${ARGUMENTS}` | ❌ NO | Literal text, not substituted |
| `$1`, `$2` positional | ❌ NO | Not supported |
| `$SKILL_NAME`, `$BASE_DIR` | ❌ NO | Not variables (base dir is prepended line) |

---

## Routing: How Claude Selects Skills

### Routing Input

Claude sees skills as options with:
```
- skill-name: description - when_to_use (Tools: X, Y, Z)
```

The `description` and `when_to_use` fields are concatenated with ` - ` separator.

### Routing Decision

Claude uses pure LLM reasoning to match user intent against skill descriptions. No embeddings or semantic search - just text matching via the model.

### Controlling Routing

```yaml
# Make skill highly visible for routing
description: Process CSV files and generate reports
when_to_use: |
  Use this skill when:
  - User mentions CSV, spreadsheet, or tabular data
  - User wants data transformation or analysis
  NOT for: JSON files (use json-processor instead)

# Hide from auto-routing but keep manual invocation
disable-model-invocation: true
```

---

## Practical Patterns

### Read-Only Exploration Skill

```yaml
---
name: Code Explorer
description: Safely explore codebase without modifications
when_to_use: Use for searching, reading, and understanding code
agent: Explore
context: fork
model: inherit
---
You are a read-only code explorer. Search and analyze the codebase.

$ARGUMENTS
```

### Hidden Utility Skill

```yaml
---
name: Internal Utility
description: Internal tool - not for general use
disable-model-invocation: true
---
# This skill won't be auto-selected but can be invoked with /internal-utility
```

### Routing-Optimized Skill

```yaml
---
name: Database Query Helper
description: Write and optimize SQL queries for PostgreSQL
when_to_use: |
  Use when user asks about:
  - SQL queries, database operations
  - PostgreSQL specific features
  - Query optimization, indexes
  NOT for: NoSQL databases, MongoDB, Redis
---
Help the user with PostgreSQL queries.

User request: $ARGUMENTS
```

---

## What Gets Parsed But Doesn't Work

These fields are recognized by the parser but have no functional effect:

| Field | Status | Evidence |
|-------|--------|----------|
| `allowed-tools` | Parsed, not enforced | All tools remain accessible |
| `hooks` | Parsed, not executed | Log files never created |
| `mode` | Parsed, unknown purpose | No observable effect |
| `user-invocable: false` | Parsed, doesn't hide | Still in /skills list |
| `tools` (array) | Plugin agent field | No effect on skills |
| `skills` (array) | Plugin agent field | No effect on skills |
| `color` | Plugin agent field | No visible effect |
| `forkContext` (boolean) | Plugin agent field | Use `context: fork` instead |

**Note on hooks:** The `PermissionRequest` hook event exists but does not fire from skill frontmatter. All hook configurations in skills are parsed but never executed.

---

## Common Mistakes

### ❌ Wrong: Hyphen in when_to_use
```yaml
when-to-use: This is ignored!
```

### ✅ Correct: Underscore
```yaml
when_to_use: This works!
```

### Model Selection (v2.1.7 Update)
```yaml
# v2.1.5: Only inherit worked
model: inherit  # Uses session model

# v2.1.7: Short names NOW WORK!
model: haiku   # ✅ Works in v2.1.7
model: sonnet  # ✅ Works in v2.1.7
model: opus    # ✅ Works in v2.1.7 (v2.5 verified)

# Omitting defaults to inherit
```

### ❌ Wrong: Agent without fork
```yaml
agent: Explore  # No effect without fork
```

### ✅ Correct: Agent with fork
```yaml
agent: Explore
context: fork
model: inherit
```

### ❌ Wrong: Expecting allowed-tools to restrict
```yaml
allowed-tools: Read  # All tools still accessible
```

### ✅ Correct: Use agent for restrictions
```yaml
agent: Explore  # Actually restricts to read-only
context: fork
model: inherit
```

### ❌ Wrong: Assuming empty $ARGUMENTS is empty string
```markdown
$ARGUMENTS

If empty, use defaults...  # WRONG - won't detect "empty"!
```
When invoked with NO arguments (`/skill-name`), `$ARGUMENTS` stays as **literal text** `"$ARGUMENTS"` - NOT empty string!

### ✅ Correct: Check for literal "$ARGUMENTS"
```markdown
$ARGUMENTS

If the text above is literally "$ARGUMENTS" (no substitution occurred),
then no arguments were provided. Use defaults: x=1, y=2
```

### ❌ Wrong: Trying to bypass agent with frontmatter
```yaml
agent: Explore
allowed-tools: Write,Edit,Bash  # IGNORED
permissionMode: allow           # IGNORED
```
Agent restrictions CANNOT be bypassed. Explore stays read-only.

---

## Minimal Valid Skill

```markdown
---
---
Do this task.
```

Yes, empty frontmatter works. Uses directory name for invocation, body for description.

---

## Recommended Template

```yaml
---
name: Human-Readable Name
description: One-line description for routing
when_to_use: |
  Detailed routing hints:
  - When to use this skill
  - What triggers it
  NOT for: What to avoid
---
# Skill Title

Detailed instructions for Claude.

User input: $ARGUMENTS
```

---

## Skill Chaining (Phase 5 & 6 Discoveries)

Skills can invoke other skills via the Skill tool. This enables powerful composition patterns.

### Chaining Behavior (Empirically Verified)

| Capability | Status | Notes |
|------------|--------|-------|
| Skill A invokes Skill B | ✅ WORKS | Via Skill tool |
| Arguments pass through | ✅ WORKS | Use `args` parameter |
| **Arguments can be modified** | ✅ WORKS | (Phase 6) Skill can transform args before passing |
| Forked skill → non-forked | ✅ WORKS | Cross-context chaining works |
| **Non-forked → forked** | ✅ WORKS | (Phase 6) Both directions work |
| Context persistence | ✅ WORKS | In non-forked context, memory persists |
| **Self-recursion** | ✅ WORKS | (Phase 6) Skills can invoke themselves |
| Recursion limits | **None built-in** | Must implement manual depth tracking |

### Chaining Syntax (Phase 6 Finding)

**Both explicit and natural language work:**

| Syntax Style | Example | Result |
|--------------|---------|--------|
| Explicit | "Use the Skill tool to invoke 'skill-name'" | ✅ Works |
| Natural | "Ask the skill-name skill to process this" | ✅ Also works |

Claude interprets both as Skill tool invocations. No special syntax required - just mention the skill by name with clear intent.

### Argument Modification in Chains (Phase 6 Finding)

Skills CAN modify arguments before passing to chained skills:

```yaml
---
name: Transform and Pass
---
# Received: $ARGUMENTS

# Modify before passing
Take the input and:
1. Convert to uppercase
2. Append " [PROCESSED]"

# Pass modified version to next skill
Invoke skill "processor" with the transformed text (not the original).
```

**Key Finding**: The chained skill receives whatever arguments you pass in the Skill tool call - not locked to original input.

### Self-Recursion (Phase 6 Finding)

Skills CAN invoke themselves. **Must implement manual depth limiting!**

```yaml
---
name: Recursive Worker
---
# Parse depth from: $ARGUMENTS

If depth >= MAX_DEPTH:
  - STOP recursion
  - Create final report
Else:
  - Do work
  - Invoke skill "recursive-worker" with args "{depth + 1}"
```

**Warning**: No built-in recursion limit exists. Without depth tracking, infinite loops are possible.

### Forked Chain Returns (Phase 6 Finding)

When a non-forked skill invokes a forked skill:
- The forked skill executes with its agent restrictions
- Returns **full textual output** (not just success/fail)
- Parent receives detailed report of forked execution

### Chain Example

```yaml
---
name: Chain Starter
description: Invokes another skill
---
# Step 1: Do something

# Step 2: Invoke another skill
Use the Skill tool to invoke "other-skill" with args "data to pass"

# Step 3: Continue after chain returns
```

### Chaining Restrictions

**What's blocked:**
- Skills with `disable-model-invocation: true` **CANNOT be invoked via Skill tool** (Phase 7 verified)
- Only manual `/skill-name` command works for DMI-protected skills
- Task tool is blocked for Explore and Plan agents

**What's allowed (surprising):**
- Skill tool IS available in forked Explore agent (Phase 6 verified)
- This enables "read-only skill that delegates to other skills" pattern

---

## Magic Docs System (Phase 5 Discovery)

> **⚠️ NOT FUNCTIONAL / NOT USER-CALLABLE**: The `magic-docs` agent exists in the codebase but:
> - Cannot be invoked via Task tool (not in available agents list)
> - Auto-update behavior never observed in any test
> - Files with `# MAGIC DOC:` header remain unchanged
>
> Documented here for completeness only.

### What Exists (Internal Only)

| Aspect | Details |
|--------|---------|
| **Agent name** | `magic-docs` |
| **Agent model** | sonnet |
| **Agent tools** | Edit only |
| **Header regex** | `/^#\s*MAGIC\s+DOC:\s*(.+)$/im` |
| **User-callable** | **NO** - Not in Task tool's available agents |
| **Auto-trigger** | **NOT WORKING** - Never observed |

### Header Pattern (For Reference)

```markdown
# MAGIC DOC: Document Title

*Optional update instructions in italics*

Document content here...
```

### Test Results

| Test | Result |
|------|--------|
| Task tool invocation | ERROR: Agent type 'magic-docs' not found |
| Auto-update after discussion | File unchanged |
| Different file locations | No difference |
| With/without skills | No difference |

**Conclusion**: Feature exists internally but is not accessible or functional for users.

---

## All Built-In Tools (17)

| Tool | Purpose | Blocked By |
|------|---------|------------|
| Read | Read files | - |
| Write | Create files | Explore, Plan |
| Edit | Modify files | Explore, Plan |
| Bash | Execute commands | - |
| Glob | Pattern matching | - |
| Grep | Content search | - |
| Task | Launch subagents | Explore, Plan |
| Skill | Invoke skills | - |
| WebFetch | Fetch URLs | - |
| WebSearch | Web search | - |
| TodoWrite | Task management | - |
| NotebookEdit | Jupyter editing | Explore, Plan |
| EnterPlanMode | Start planning | - |
| ExitPlanMode | Exit planning | Explore, Plan |
| AskUserQuestion | Ask user | - |
| KillShell | Kill background shell | - |
| TaskOutput | Get task output | - |

---

## Multiple Arguments Workaround (Phase 6 Verified)

**Only `$ARGUMENTS` is supported** - no `$name`, `$email`, `$subject`, etc.

### All 4 Patterns Empirically Verified (Phase 6)

| Pattern | Example Input | Status |
|---------|---------------|--------|
| Structured | `name="John" email="j@e.com"` | ✅ WORKS |
| Delimiter | `Title | Author | Content` | ✅ WORKS |
| JSON | `{"env": "prod", "port": 3000}` | ✅ WORKS |
| Positional | `feat: Add feature` | ✅ WORKS |

### Pattern 1: Structured Parsing (Recommended)
```yaml
---
name: Email Composer
description: Compose an email with structured input
---
Parse the following input to extract: name, email, subject, and body.

Input format: name="..." email="..." subject="..." body="..."

User input: $ARGUMENTS

Extract each field and compose the email.
```

**Usage:** `/email-composer name="John" email="john@example.com" subject="Hello" body="Meeting tomorrow"`

**Phase 6 Test Result:** Successfully extracted all fields from `name="John" email="john@example.com" subject="Test Message"`

### Pattern 2: Delimiter-Based
```yaml
---
name: Quick Note
description: Create a note with title and content
---
The input uses | as delimiter: TITLE | CONTENT

Input: $ARGUMENTS

Split on | and use:
- First part as title
- Second part as content
```

**Usage:** `/quick-note Project Update | We completed the migration successfully`

**Phase 6 Test Result:** Successfully split `My Article Title | Jane Smith | This is the content` into 3 fields.

### Pattern 3: JSON Input
```yaml
---
name: Config Generator
description: Generate config from JSON parameters
---
Parse the JSON input and generate configuration.

JSON Input: $ARGUMENTS

Expected fields: { "env": "...", "port": "...", "debug": "..." }
```

**Usage:** `/config-generator {"env": "production", "port": 3000, "debug": false}`

**Phase 6 Test Result:** Successfully parsed `{"env": "production", "port": 3000, "debug": false, "features": ["auth", "logging"]}` including nested arrays.

### Pattern 4: Positional with Labels
```yaml
---
name: Git Commit Helper
description: Create commit with type and message
---
Input format: TYPE: MESSAGE

Where TYPE is one of: feat, fix, docs, refactor, test

Input: $ARGUMENTS

Parse the type and message, then create the commit.
```

**Usage:** `/git-commit-helper feat: Add user authentication`

**Phase 6 Test Result:** Successfully extracted TYPE=`feat` and MESSAGE=`Add user authentication with OAuth2 support` from `feat: Add user authentication with OAuth2 support`

### Choosing a Pattern

| Pattern | Best For | Pros | Cons |
|---------|----------|------|------|
| Structured | Named fields | Clear, self-documenting | Verbose syntax |
| Delimiter | 2-3 fields | Concise | Field order matters |
| JSON | Complex/nested data | Flexible, supports arrays | Requires valid JSON |
| Positional | Type: Message pairs | Familiar (git commits) | Limited to 2 fields |

---

## Skill Orchestration Patterns

> **Key Insight**: Skills are not just "prompt templates with routing." They are **composable state transformers with delegation**. The argument is the state. Each invocation is a state transition. Delegation enables controlled capability escalation.

### Argument-as-State-Machine

Encode workflow state directly in the argument string. Each skill invocation transforms the argument before passing to the next skill (or self). There's no separate state storage—the argument IS the state.

**State Encoding Strategies:**

| Strategy | Format | Example | Best For |
|----------|--------|---------|----------|
| Positional | `value1 value2` | `"3 10"` (current=3, max=10) | Simple counters |
| Path+Context | `path depth max` | `"./src 2 5"` | Tree traversal |
| JSON | `{...}` | `{"iteration": 2, "history": [...]}` | Complex state with history |

**Enables**: Bounded recursion, convergence loops, pagination, undo/rollback (via history).

**Examples**: #9 Recursive Counter, #10 Deep Directory Analyzer, #13 Iterative Refinement

---

### Validate-Then-Execute

Fork a read-only skill to assess risk before the main skill proceeds with modifications. **Fork isolation guarantees validation integrity**—the validator cannot modify anything even if compromised.

```
Main Skill (full access)
    │
    ├──► Fork to Validator (Explore agent - read only)
    │         • CAN read, search, analyze
    │         • CANNOT write, edit, delete
    │         • CANNOT lie (output visible to parent)
    │
    ◄──── Returns risk assessment
    │
    └──► Proceed or abort based on assessment
```

**Security Property**: Defense-in-depth. Even adversarial validator prompts can't cause writes.

**Enables**: Safe refactoring, auditable changes, pre-flight checks.

**Examples**: #12 Validate-Then-Execute

---

### Fork + Delegate (Security Airlock)

Exploit the undocumented fact that **Explore agent has Skill tool access**. This creates a security airlock: the entry point can't write directly but can request writes through delegation.

```
Untrusted Request
    │
    ▼
Forked Explore Skill (can't write)
    │
    ├── Analyzes request
    ├── Decides what's needed
    │
    ▼
Invokes specific skill with specific args
    │
    ▼
Target skill runs in ITS context (may have write access)
```

**Security Property**: Principle of least privilege with controlled escalation. The Explore skill triages; specialists execute.

**Enables**: Safe entry points, audit trails, capability-based delegation.

**Examples**: #16 Fork-Protected Skill Chain

---

### Conditional Delegation (Plugin Architecture)

Check for specialist skills before falling back to direct handling. Skills become optional dependencies—the orchestrator degrades gracefully if specialists are missing.

```
detect_technology(request)
    │
    ├── Python detected → if python-specialist exists → delegate
    ├── JavaScript detected → if js-specialist exists → delegate
    ├── SQL detected → if sql-specialist exists → delegate
    │
    └── No match or no specialist → handle directly (fallback)
```

**Extensibility**: Add new capabilities by dropping in new skill files. No changes to orchestrator.

**Enables**: Plugin architectures, graceful degradation, modular skill ecosystems.

**Examples**: #7 Conditional Fallback, #15 Conversational Skill Router

---

### Pipeline with Argument Modification

Sequential stages where each stage transforms the data before passing to the next. Unlike simple chaining, the argument content changes at each stage.

```
Raw Input → Extract → Transform → Load → Final Output
              │          │         │
              └──────────┴─────────┘
              Argument modified at each stage
```

**Enables**: ETL workflows, multi-stage transformations, format conversions.

**Examples**: #2 Multi-Stage Pipeline, #11 Data Transformation Pipeline

---

### Pattern Summary

| Pattern | Core Mechanism | Security Property | Use Case |
|---------|----------------|-------------------|----------|
| Argument-as-State | Args encode state | State is visible and auditable (no hidden mutations) | Bounded loops |
| Validate-Then-Execute | Fork isolation | Validator is physically incapable of writes | Safe refactoring |
| Fork + Delegate | Explore + Skill tool | Entry point has no direct write path | Security airlocks |
| Conditional Delegation | Skill existence check | Graceful degradation (no hard failures) | Plugin systems |
| Pipeline + Arg Mod | Transform at each stage | Data lineage visible at each stage | ETL workflows |

> **Note**: The first three patterns provide **capability-based security** ("can't do bad things"), not policy-based security ("don't do bad things"). The constraints are architectural, not behavioral.

---

## Use Case Examples

**See: [claude-code-skills-examples.md](claude-code-skills-examples.md)**

16 complete, deployable skill examples demonstrating the patterns above:

### Design Patterns

| Pattern | Description | Key Feature |
|---------|-------------|-------------|
| **Orchestrator** | Route requests to specialist skills | Analyze input, invoke appropriate skill |
| **Pipeline** | Sequential multi-stage processing | Chain stages, pass data between |
| **Building Block** | Utility skill designed for chaining | Structured output for callers |
| **Protected** | Dangerous ops requiring explicit intent | `disable-model-invocation: true` |
| **Stateful** | Context persists across phases | Non-forked execution |
| **Multi-Format** | Generate multiple output formats | Same input, different outputs |
| **Fallback** | Try specialist, fall back to default | Conditional skill invocation |
| **Self-Recursion** | Skill invokes itself with modified args | Depth tracking, termination condition |
| **Validate-Execute** | Read-only check before changes | Forked Explore for validation |
| **Multi-Arg** | Parse complex structured input | Structured/JSON/delimiter parsing |
| **Fork + Delegate** | Read-only skill that invokes others | `agent: Explore` + Skill tool |

---

## Known Unknowns (Untested Hypotheses)

The following SOTA hypotheses remain **unverified** through empirical testing:

### YAML Syntax (Categories S1-S5) ✅ ALL TESTED
- ~~S1: YAML anchors and aliases~~ → **Phase 7: ✅ SUPPORTED**
- ~~S2: Multi-document YAML~~ → **Phase 7: First doc only (standard behavior)**
- ~~S3: Nested structures (deep objects)~~ → **Phase 7: WORKS (4+ levels)**
- ~~S4: Complex string escaping~~ → **Phase 7: ✅ ALL PRESERVED (Unicode, quotes, etc.)**
- ~~S5: Case sensitivity variations~~ → **Phase 7: CASE SENSITIVE (lowercase required)**

### Variable/Environment Injection (Categories T1-T4)
- ~~T1: Shell variable expansion in frontmatter (`$HOME`, `${VAR}`)~~ → **Phase 7: NOT WORKING**
- ~~T2: Environment variable references~~ → **Phase 7: Accessible via Bash tool only**
- ~~T3: Template syntax (Jinja, Handlebars)~~ → **Phase 7: NOT WORKING (all literal)**
- T4: Expression evaluation

### Routing Manipulation (Categories V1-V4) ✅ ALL TESTED
- ~~V1: Priority/weight fields~~ → **Phase 7: SILENTLY IGNORED**
- ~~V2: Category/tag-based routing~~ → **Phase 7: SILENTLY IGNORED**
- ~~V3: Context-aware routing~~ → **Phase 7: SILENTLY IGNORED (context_match, file_types, triggers)**
- V4: Dynamic description modification (untested - likely ignored)

### Execution Control (Categories W3-W4)
- ~~W3: Timeout configuration fields~~ → **Phase 7: SILENTLY IGNORED**
- ~~W4: max_turns/exit fields~~ → **Phase 7: SILENTLY IGNORED**

### Security Boundaries (Categories X1-X4) ✅ ALL TESTED
- ~~X1: Path traversal in skill references~~ → **Phase 7: BLOCKED (secure)**
- ~~X2: Privilege escalation via field combinations~~ → **Phase 7: BLOCKED (secure)**
- ~~X3: Sandbox escape attempts~~ → **Phase 7: Read-only enforced (write blocked)**
- ~~X4: Cross-skill data exfiltration~~ → **Phase 7: No isolation (by design - shared FS)**

### Experimental Features (Categories Z1-Z4)
- ~~Z1-Z4: experimental/beta/internal/debug/verbose/trace~~ → **Phase 7: ALL SILENTLY IGNORED**

### Integration Fields (Categories AA1-AA3) ✅ ALL TESTED
- ~~AA1: MCP server references~~ → **Phase 7: SILENTLY IGNORED (MCP config in settings.json only)**
- ~~AA2: Settings override fields~~ → **Phase 7: SILENTLY IGNORED**
- ~~AA3: Environment injection via frontmatter~~ → **Phase 7: SILENTLY IGNORED**

### Edge Cases (Category E)
- ~~E1: Empty arguments~~ → **Phase 7: $ARGUMENTS stays as literal text**
- E2-E4: Special chars, very long args, conflicting frontmatter
- ~~E5: `disable-model-invocation` Skill tool blocking~~ → **Phase 7: BLOCKS Skill tool**
- ~~Positional $1 $2 $3~~ → **Phase 7: NOT WORKING (literal text)**

### Phase 7 Test Results (v1.8-2.0)
| Test | Result | Finding |
|------|--------|---------|
| X1: Path traversal | ✅ BLOCKED | Skills looked up by name registry, not file path |
| X2: Privilege escalation | ✅ BLOCKED | `allowed-tools`/`permissionMode` cannot override agent restrictions |
| X3: Sandbox escape | ✅ SECURE | Explore agent read-only enforced; file reading allowed (by design) |
| E5: DMI Skill tool | ✅ VERIFIED | `disable-model-invocation` blocks Skill tool invocations |
| T1: Shell variables | ❌ NOT WORKING | `$HOME`, `$USER` appear as literal text in body |
| S3: Nested YAML | ✅ WORKS | 4-level deep nested structures parse correctly |
| S5: Case sensitivity | ✅ DOCUMENTED | Field names MUST be lowercase; uppercase silently ignored |
| T2: Environment vars | ✅ WORKS (Bash) | Accessible via Bash tool, not in skill body text |
| T3: Template syntax | ❌ NOT WORKING | Jinja/Handlebars/ERB/Mustache all literal text |
| E1: Empty arguments | ✅ DOCUMENTED | `$ARGUMENTS` stays as literal `"$ARGUMENTS"` when no args |
| Positional $1 $2 $3 | ❌ NOT WORKING | Appear as literal text; only `$ARGUMENTS` substitutes |
| V1: Priority/weight | ❌ IGNORED | Fields accepted but have no effect on routing |
| V2: Category/tags | ❌ IGNORED | Fields accepted but not used for filtering/display |
| Z1-Z4: Experimental | ❌ ALL IGNORED | experimental/beta/debug/verbose/trace have no effect |
| W3-W4: Timeout/turns | ❌ IGNORED | timeout/max_turns/max_duration not enforced in skills |
| AA2-AA3: Settings/env | ❌ IGNORED | settings:/env: blocks in frontmatter have no effect |
| S1: YAML anchors | ✅ WORKS | &anchor/*alias/<<: merge all supported |
| S2: Multi-document | ✅ DOCUMENTED | First document only; rest becomes body |
| S4: String escaping | ✅ WORKS | Unicode, quotes, special chars all preserved |
| V3: Context routing | ❌ IGNORED | context_match/file_types/triggers ignored |
| AA1: MCP references | ❌ IGNORED | mcp_servers/requires_mcp not processed |
| X4: Data exfiltration | ✅ BY DESIGN | No FS isolation - skills share file access |

### Phase 6 Newly Tested (v1.7)
| Test | Result | Finding |
|------|--------|---------|
| B2: `$1`, `$2`, `$3` positional syntax | ❌ NOT WORKING | Only `$ARGUMENTS` substitutes; positional vars appear as literal text |
| B2: `argNames` frontmatter field | ❌ NOT WORKING | Named args (`$first`, `$second`) not substituted |
| D: Magic Docs auto-update | ❌ NOT WORKING | File unchanged after discussion; no observed auto-update behavior |

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.6 | 2026-01-17 | **Phase 8: Arguments Deep Dive**: Binary analysis + empirical testing on v2.1.11. Confirmed: @ file reference NOT expanded (literal), $ARGUMENTS case-sensitive (only uppercase), argument length limit ≥20,000 chars, special chars preserved verbatim. Total tests: 175+. |
| 2.5 | 2026-01-15 | **v2.1.7 Full Re-test**: Complete re-verification on v2.1.7 with 26 new tests. All model short names work (haiku/sonnet/opus). Confirmed: hooks don't fire, allowed-tools ignored, agent requires fork, DMI blocks Skill tool, Explore read-only, Magic Docs non-functional. Total tests: 167. |
| 2.4 | 2026-01-15 | **v2.1.7 Spot Check**: `model: haiku/sonnet` NOW WORKS (was 404 in v2.1.5). Other findings unchanged: allowed-tools still ignored, DMI still blocks Skill tool, Explore still read-only. Total tests: 141. |
| 2.3 | 2026-01-15 | **Feedback Response**: Fixed fork return values (full output, not summary). Added Magic Docs "NOT FUNCTIONAL" warning. Clarified Skill tool available in Explore. Verified main-session/subagent have full access. Confirmed model: inherit is optional (defaults to inherit). Total tests: 137. |
| 2.3 | 2026-01-15 | Feedback fixes: fork returns full output (not summary), Magic Docs warning, Skill in Explore, main-session/subagent verified, model optional. |
| 2.2 | 2026-01-15 | Added comprehensive "Silently Ignored Fields" table. Clarified $ARGUMENTS only substitutes in body, not frontmatter. |
| 2.1 | 2026-01-15 | Phase 7 FINAL: All remaining tests complete. S1/S4 YAML features WORK. V3/AA1 IGNORED. X4 no FS isolation (by design). Total tests: 133. ALL HYPOTHESES TESTED. |
| 2.0 | 2026-01-15 | Phase 7 COMPLETE: All 31 hypotheses tested. V1-V2/Z1-Z4/W3-W4/AA2-AA3 all IGNORED. Security X1-X3 PASS. Only ~5 items remain untested. Total tests: 127. |
| 1.9 | 2026-01-15 | Phase 7 continued: Case sensitivity (S5), env vars via Bash only (T2), template syntax not working (T3), empty args behavior (E1), positional $1/$2/$3 not working. Total tests now 122. |
| 1.8 | 2026-01-15 | Phase 7: Security tests (X1-X3 PASS), DMI blocks Skill tool (E5 VERIFIED), shell vars not expanded (T1), nested YAML works (S3). Total tests now 117. |
| 1.7 | 2026-01-15 | Tested B2 & D hypotheses: `$1`/`$2`/`$3` positional syntax NOT WORKING, `argNames` NOT WORKING, Magic Docs auto-update NOT WORKING. Total tests now 110. |
| 1.6 | 2026-01-15 | Added PermissionRequest hook finding. Added "Known Unknowns" section. Note: At this point, DMI Skill tool blocking was uncertain; later confirmed by Phase 7 test E5. |
| 1.5 | 2026-01-15 | All 17 use case examples now complete deployable skills with full working syntax. Fixed use case numbering (was duplicate 11). Recursive counter skill verified working and included. |
| 1.4 | 2026-01-15 | Added 8 Phase 6 enabled use cases (examples 10-17): recursive counter, deep directory analyzer, ETL pipelines, validate-then-execute, iterative refinement, multi-format forms, natural language routing, fork-protected chains |
| 1.3 | 2026-01-15 | Phase 6: Verified chaining syntax (natural/explicit both work), argument modification in chains, self-recursion, all 4 multiarg patterns verified |
| 1.2 | 2026-01-15 | Added advanced use cases, multiple arguments workarounds |
| 1.1 | 2026-01-15 | Phase 5: Skill chaining, magic docs, 9 agent types, all tools |
| 1.0 | 2026-01-14 | Initial comprehensive reference |

---

## Phase 6 Test Summary

### Category A: Chaining Syntax (7 tests)
| Test | Result | Key Finding |
|------|--------|-------------|
| A1-1: Explicit Skill tool | ✅ WORKS | "Use the Skill tool to invoke..." works |
| A1-2: Natural language | ✅ WORKS | "Ask the skill to..." also works |
| A2-1: Argument modification | ✅ WORKS | Can transform args before passing |
| A3-1: Non-forked → forked | ✅ WORKS | Both directions work |
| A3-2: Self-recursion | ✅ WORKS | No built-in limit - need manual depth tracking |
| A3-3: Forked returns | ✅ Full output | Not just success/fail - detailed text |
| A3-4: Skill tool in forked Explore | ✅ AVAILABLE | Surprising - Skill tool works in forked Explore |

### Category B: Multiple Arguments (4 tests)
| Test | Result | Example |
|------|--------|---------|
| B1-1: Structured | ✅ WORKS | `name="John" email="j@e.com"` |
| B1-2: Delimiter | ✅ WORKS | `Title | Author | Content` |
| B1-3: JSON | ✅ WORKS | `{"key": "value", "nested": [...]}` |
| B1-4: Positional | ✅ WORKS | `feat: Add feature` |

---

*Based on systematic testing of 167 skill configurations on Claude Code v2.1.7*

**Test breakdown:**
- Phase 1-3A: 82 tests (58 original + 7 agent + 1 mode + 1 c7 + 5 Phase 2 + 10 Phase 3A)
- Phase 5: 14 tests (skill chaining, magic docs)
- Phase 6: 14 tests (11 original + 3 new: positional syntax, argNames, Magic Docs auto-update)
- Phase 7: 23 tests (X1-X4, E1/E5, T1-T3, S1-S5, V1-V3, Z1-Z4, W3-W4, AA1-AA3)
- Feedback Tests: 4 tests (fork-nomodel, main-session, subagent, routing A/B)
- v2.1.7 Spot Check: 4 tests (model:haiku, model:sonnet, allowed-tools, DMI blocking)
- **v2.1.7 Full Re-test: 26 tests** (A3-A12, B1-B5, C1-C5, D1, E1-E5, F1)

**ALL KNOWN UNKNOWNS NOW TESTED** ✅

---

## v2.1.7 Re-Test Results Summary

| Test ID | Field/Feature | Result | Notes |
|---------|---------------|--------|-------|
| A3 | `user-invocable: false` | ❌ Still visible | Does not hide from /skills |
| A4 | `hooks` in frontmatter | ❌ Don't fire | Parsed but not executed |
| A5 | `mode: true` | ❌ No effect | Unknown purpose |
| A6 | `allowed-tools` | ❌ Ignored | Does not restrict tools |
| A7 | `permissionMode` | ❌ Ignored | Cannot override permissions |
| A9 | `agent` without fork | ❌ No effect | Agent requires `context: fork` |
| A12 | `model: opus` | ✅ **WORKS** | Short model names fixed in v2.1.7 |
| B1 | `$ARGUMENTS` in body | ✅ Works | Substituted correctly |
| B2 | `$ARGUMENTS` in frontmatter | ❌ Literal | NOT substituted |
| B3 | `${ARGUMENTS}` syntax | ❌ Literal | Only `$ARGUMENTS` works |
| B4 | `$1 $2 $3` positional | ❌ Literal | Not supported |
| B5 | Empty args | ❌ Literal | Stays as "$ARGUMENTS" |
| C1 | Skill chaining | ✅ Works | Skills can invoke other skills |
| C2 | Argument passing | ✅ Works | Args passed through chain |
| C3 | Self-recursion | ✅ Works | No built-in limit |
| C5 | Forked returns | ✅ Full output | Complete text returned |
| D1 | Ignored fields | ✅ Confirmed | priority/weight/tags/etc ignored |
| E1 | Explore + Skill tool | ✅ Available | Skill tool works in Explore |
| E2 | Explore blocks Write | ✅ Blocked | "No such tool available" |
| E3 | Explore blocks Task | ✅ Blocked | Cannot spawn subagents |
| E4 | Bash agent | ✅ Confirmed | Only Bash tool available |
| E5 | Plan agent | ✅ Confirmed | Same as Explore (read-only) |
| F1 | Magic Docs | ❌ Not functional | No auto-update observed |

---

## Phase 8 Test Summary: Arguments Deep Dive (v2.1.11)

### Methodology

1. **Binary Analysis**: String extraction from `/home/jim/.local/share/claude/versions/2.1.11` (779K+ lines)
2. **Empirical Testing**: 8 new test skills with systematic hypothesis verification
3. **Scope**: @ file references, $ARGUMENTS behavior, length limits, special characters

### Binary Analysis Results

| Search Pattern | Result | Conclusion |
|----------------|--------|------------|
| `$ARGUMENTS` handling | Noise from minified JS | Inconclusive |
| `maxArgLength` / `inputMaxLength` | Not found | No hardcoded limit detected |
| `@file` / `expandFile` patterns | Not found | No @ expansion logic |
| `replaceAll` in skill context | Unclear due to minification | Simple substitution likely |

### Empirical Test Results

#### Category A: @ File Reference (Hypothesis B1-B3)

| Test | Input | Result | Finding |
|------|-------|--------|---------|
| A1 | `@/mnt/c/python/temp1/test-file.txt` | Literal `@/mnt/c/python/temp1/test-file.txt` | **NO expansion** |

**Conclusion**: ❌ **@ prefix has NO special meaning** - passed as literal text. Unlike `curl @file` or `gh --body @file`, Claude Code does not expand @ file references in skill arguments.

#### Category B: $ARGUMENTS Substitution (Hypothesis C1-C4)

| Test | Pattern | Result | Finding |
|------|---------|--------|---------|
| B1 | `$ARGUMENTS` (uppercase) | ✅ Substituted | Works - all 9 occurrences replaced |
| B2 | `$arguments` (lowercase) | ❌ Literal | NOT substituted |
| B3 | `$Arguments` (mixed) | ❌ Literal | NOT substituted |
| B4 | `${ARGUMENTS}` (braces) | ❌ Literal | NOT substituted |

**Conclusion**:
- ✅ **Case-sensitive**: ONLY uppercase `$ARGUMENTS` works
- ✅ **Multiple occurrences**: All instances substituted
- ❌ `$arguments`, `$Arguments`, `${ARGUMENTS}` remain literal

#### Category C: Argument Length Limits (Hypothesis A1-A3)

| Test | Length | Result | Notes |
|------|--------|--------|-------|
| C1 | 100 chars | ✅ Complete | All received |
| C2 | 1,000 chars | ✅ Complete | All received |
| C3 | 10,000 chars | ✅ Complete | All received |
| C4 | 20,000 chars | ✅ Complete | All received |

**Conclusion**: ✅ **No practical limit found up to 20,000 characters**. Actual limits likely determined by:
- Terminal input buffer (~4096 chars for interactive)
- API message size limits
- Model context window (practical, not enforced)

#### Category D: Special Character Handling (Hypothesis D1-D4)

| Character Type | Input | Output | Result |
|----------------|-------|--------|--------|
| Double quotes | `"quoted"` | `"quoted"` | ✅ Preserved |
| Single quotes | `'single'` | `'single'` | ✅ Preserved |
| Backticks | `` `backtick` `` | `` `backtick` `` | ✅ Preserved |
| Backslashes | `\n\t` | `\n\t` | ✅ Literal (not escaped) |
| Dollar sign | `$HOME` | `$HOME` | ✅ Literal (not shell-expanded) |
| Spaces | `a  b` | `a  b` | ✅ Preserved |

**Conclusion**: ✅ **Arguments passed VERBATIM** - no shell expansion, no escape processing, no transformations.

### Test Skills Created

| Skill | Purpose | Location |
|-------|---------|----------|
| `phase8-length-test` | Argument length limits | `~/.claude/skills/phase8-length-test/` |
| `phase8-at-file-test` | @ file reference handling | `~/.claude/skills/phase8-at-file-test/` |
| `phase8-substitution-test` | $ARGUMENTS substitution behavior | `~/.claude/skills/phase8-substitution-test/` |
| `phase8-special-chars-test` | Special character handling | `~/.claude/skills/phase8-special-chars-test/` |

### Key Findings Summary

| Finding | Verified | Implication |
|---------|----------|-------------|
| @ is literal | ✅ | Cannot use `@file.txt` to inject file contents |
| $ARGUMENTS case-sensitive | ✅ | Must use exact uppercase `$ARGUMENTS` |
| No length limit ≤20K | ✅ | Large arguments supported |
| Verbatim passing | ✅ | Shell metacharacters safe, quotes preserved |

### Implications for Skill Authors

1. **File Content Injection**: Cannot use `@filename` - must use alternative patterns:
   ```markdown
   # Read file via Bash
   Read the file at: $ARGUMENTS
   Use Bash with `cat $ARGUMENTS` to get contents
   ```

2. **Case Matters**: Only `$ARGUMENTS` works, not `$arguments` or `${ARGUMENTS}`

3. **Large Arguments OK**: Feel free to pass long JSON, code snippets, or multi-line text

4. **No Shell Escaping Needed**: `$HOME`, quotes, backticks all pass through literally
