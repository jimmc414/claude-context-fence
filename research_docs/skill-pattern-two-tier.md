# Two-Tier Skill Architecture Specification

A comprehensive specification for building context-aware, token-efficient skills using the router + recipes pattern.

---

## Executive Summary

This pattern solves the fundamental tension between **context awareness** (needing conversation history) and **token efficiency** (not bloating context with dense references).

**The Problem:**
- Skills with `context: fork` are token-efficient but lose conversation context
- Vague requests like "fix that json thing" fail because the skill doesn't know what file was discussed
- Dense recipe skills (500+ lines) would bloat main context if not forked

**The Solution:**
A two-tier architecture where:
1. **Router Skill** - Lightweight, sees full context, builds rich arguments
2. **Recipe Skill** - Dense references, isolated by fork, receives prepared arguments

**The Result:** Unlimited recipe density with full context awareness.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           CLAUDE.md                                      │
│  ─────────────────────────────────────────────────────────────────────  │
│  "When invoking task-* skills, prefer the more descriptive skill."      │
│                                                                          │
│  This single line guides ALL routing decisions for ALL skill pairs.     │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        ROUTING DECISION                                  │
│  ─────────────────────────────────────────────────────────────────────  │
│  Model sees both skills in routing list:                                │
│                                                                          │
│  • task-json: "JSON processing - extracting, filtering, transforming    │
│    JSON data" + triggers: json, jq, extract field, filter array...      │
│                                                                          │
│  • task-json-recipes: "JSON recipes reference (internal)"               │
│                                                                          │
│  CLAUDE.md guidance + stronger description → routes to task-json        │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     ROUTER SKILL (task-json.md)                          │
│  ─────────────────────────────────────────────────────────────────────  │
│  context: inherit     │ Sees FULL conversation history                  │
│  allowed-tools: Skill │ Can invoke other skills                         │
│                                                                          │
│  EXTRACTS FROM CONVERSATION:                                            │
│  • Goal: "extract emails"                                               │
│  • File: /mnt/c/project/data.json (mentioned 5 messages ago)           │
│  • Structure: .data.users[].contact.email (discussed in error msg)     │
│  • Filter: .status == "active" (user's constraint)                     │
│                                                                          │
│  BUILDS ARGUMENT:                                                       │
│  "extract emails from /mnt/c/project/data.json                         │
│   - structure: .data.users[].contact.email                             │
│   - filter: .status == 'active'                                        │
│   - output: raw lines"                                                  │
│                                                                          │
│  SIZE: ~50 lines, ~600 tokens                                           │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   RECIPE SKILL (task-json-recipes.md)                    │
│  ─────────────────────────────────────────────────────────────────────  │
│  context: fork        │ Isolated - doesn't pollute main context         │
│  allowed-tools: Read  │ Can read files to inspect structure             │
│                                                                          │
│  RECEIVES: Well-formed argument with all context                        │
│  MATCHES: Task to appropriate jq recipe                                 │
│  RETURNS: Executable command                                            │
│                                                                          │
│  jq -r '.data.users[] | select(.status == "active")                    │
│       | .contact.email' /mnt/c/project/data.json                       │
│                                                                          │
│  SIZE: ~250 lines, ~2500 tokens (protected by fork)                    │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  RESULT: Command returns to main context                                │
│  CLEANUP: Recipe skill content discarded (fork isolation)               │
│  EFFICIENCY: Main context grew by ~100 tokens, not 2500                │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Component 1: CLAUDE.md Guidance

**Purpose:** Provide routing preference that scales to unlimited skill pairs.

**Location:** Project root `CLAUDE.md`

**Content:**

```markdown
## Skill Routing

When invoking `task-*` skills:

1. **Prefer the more descriptive skill** over internal/recipes variants
2. Skills with "(internal)" in description are helpers - let parent invoke them
3. The descriptive skill has conversation context to build proper arguments

The pattern:
- ✅ `task-json` → descriptive, has context, builds arguments
- ❌ `task-json-recipes` → internal, receives prepared arguments
```

**Why This Works:**
- Single rule covers infinite skill pairs
- "More descriptive" is unambiguous - router always wins
- Explains WHY so model understands the reasoning

---

## Component 2: Router Skill

**Filename Pattern:** `task-{domain}.md`

### Frontmatter (Controls Routing)

```yaml
---
description: "{Domain} processing - {verb1}, {verb2}, {verb3}, and {verb4}"
when_to_use: "Use when: {scenario1}, {scenario2}, {scenario3}. Triggers: {word1}, {word2}, {word3}, {word4}, {word5}, {word6}."
when-to-use: "Use when: {scenario1}, {scenario2}, {scenario3}. Triggers: {word1}, {word2}, {word3}, {word4}, {word5}, {word6}."
argument-hint: "[describe what you want to do with {domain}]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---
```

### Frontmatter Field Optimization

| Field | Goal | Strategy |
|-------|------|----------|
| `description` | **WIN** routing competition | Rich, specific, action verbs, 10-15 words |
| `when_to_use` | Match user queries | 6-10 trigger words, common phrasings |
| `context` | See conversation | **MUST be `inherit`** |
| `allowed-tools` | Chain to recipes | **MUST include `Skill`** |

### Body Template

```markdown
# {Domain} Task Router

You have access to the **full conversation context**. Your job is to build a well-formed argument for the recipes skill.

**User's request**: $ARGUMENTS

---

## Context Extraction

Analyze the conversation to identify:

### 1. Goal
What operation is needed?
- Extract, filter, transform, aggregate, convert, merge, validate, etc.

### 2. File Path(s)
Where is the data?
- Explicit paths mentioned by user
- Files created/modified earlier in conversation
- Files referenced in error messages
- Current working directory context

### 3. Structure
How is the data organized?
- **JSON**: Nested paths like `.data.users[].email`
- **Tabular**: Column numbers, delimiters, header presence
- **Text**: Line patterns, field positions
- **Binary**: Section names, offsets, patterns

### 4. Filter Conditions
What constraints apply?
- Value comparisons: `.price > 100`
- String matching: `contains("error")`
- Null handling: `!= null`
- Boolean logic: `and`, `or`, `not`

### 5. Output Format
What should the result look like?
- Raw values (one per line)
- Formatted (CSV, TSV, JSON)
- With context (line numbers, filenames)
- Aggregated (count, sum, average)

### 6. Scope Limits
Any boundaries?
- First/last N items
- Date ranges
- Size limits
- Depth limits (recursive operations)

### 7. Error Context (if debugging)
What went wrong?
- Error messages from previous attempts
- Unexpected output received
- What was tried and failed

---

## Argument Format

Construct a structured argument string:

<goal> from <file_path> - structure: <path>, filter: <condition>, output: <format>, scope: <limit>

**Include only relevant components.** Not every task needs all fields.

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

Skill: task-{domain}-recipes
Args: <your constructed argument>

### When to Ask Instead

**Ask the user if:**
- File path cannot be determined from context
- Multiple files could apply and it's ambiguous
- The goal itself is unclear

**Use Read tool first if:**
- Structure is unknown and file path is known
- Need to verify file exists
- Need to understand data format before building argument
```

### Size Target

- **Lines:** 50-80
- **Tokens:** 600-1000
- **Principle:** Instructions, not content

---

## Component 3: Recipe Skill

**Filename Pattern:** `task-{domain}-recipes.md`

### Frontmatter (Discourages Direct Routing)

```yaml
---
description: "{Domain} recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-{domain} with file paths and context included"
when-to-use: "Receives prepared arguments from task-{domain} with file paths and context included"
context: fork
model: inherit
allowed-tools: Read
---
```

### Frontmatter Field Optimization

| Field | Goal | Strategy |
|-------|------|----------|
| `description` | **LOSE** routing competition | Minimal, includes "(internal)" |
| `when_to_use` | Not match user queries | Describes input, not triggers |
| `context` | Token isolation | **MUST be `fork`** |
| `allowed-tools` | File inspection only | `Read` only (leaf node, no chaining) |

### Body Template

```markdown
# {Domain} Recipes

**Tools:** {tool1}, {tool2}, {tool3}, {tool4}

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| {common operation 1} | `{command}` |
| {common operation 2} | `{command}` |

---

## {Category 1}: {Name}

### {Subcategory A}
{commands}

---

## Common Patterns

### {Workflow 1}
{multi-step workflow}

---

## Edge Cases

### Handling {edge case 1}
{command with special handling}

---

## Installation

# Debian/Ubuntu
sudo apt install {packages}
```

### Size Target

- **Lines:** 200-600+
- **Tokens:** 2000-6000+
- **Principle:** Comprehensive coverage (protected by fork)

---

## Argument Structure Standards

### File-Centric Pattern

For skills operating on files (JSON, tabular, text, archives, media, binary):

```
<goal> from <file_path> - structure: <hints>, filter: <conditions>, output: <format>, scope: <limits>
```

| Component | Required | Description |
|-----------|----------|-------------|
| `goal` | ✅ | Action verb + target: "extract emails", "sum prices" |
| `file_path` | ✅ | Absolute or relative path |
| `structure` | Situational | Data organization hints |
| `filter` | Situational | Conditions to apply |
| `output` | Situational | Desired format |
| `scope` | Situational | Limits/boundaries |

### Target-Centric Pattern

For skills operating on targets (network, HTTP, API):

```
<goal> on <target> - options: <flags>, output: <format>
```

### Action-Centric Pattern

For skills performing system actions (process, permissions):

```
<action> [target] - scope: <what>, options: <how>
```

---

## Optimization Strategies

### 1. Win the Routing Competition

**Router description must be richer than recipes:**

```yaml
# ✅ Router - WINS
description: "JSON processing - extracting fields, filtering arrays, transforming structure, converting formats"

# ✅ Recipes - LOSES
description: "JSON recipes reference (internal)"
```

**Router triggers must cover user vocabulary:**

```yaml
# ❌ Sparse - misses queries
when_to_use: "Use for JSON"

# ✅ Dense - catches variations
when_to_use: "Use when: extracting fields from JSON, filtering JSON arrays, reshaping JSON, converting JSON to CSV. Triggers: json, jq, parse json, json field, json array, json filter, nested json, api response."
```

### 2. Extract Context Reliably

**Be explicit about what to look for:**
- List specific context types (goal, file, structure, filter, output, scope, errors)
- Give examples of each
- Define fallback behavior when context is missing

### 3. Maximize Recipe Density

**Quick reference table:** Enables fast pattern matching
**Categorized sections:** Organized by task type, not tool flag
**Copy-paste ready:** Commands work with placeholder substitution
**Edge cases:** Cover the 20% that causes 80% of problems
**Error patterns:** Show common failures and fixes

### 4. Graceful Degradation

The system should work even when "misused":

| Scenario | Behavior |
|----------|----------|
| Direct recipe invocation | Works with explicit args |
| Vague router invocation | Router asks for clarification |
| Missing file path | Router prompts user |
| Unknown structure | Router uses Read tool to inspect |

---

## Anti-Patterns to Avoid

### ❌ Routing Logic in Recipes

```yaml
# BAD - confuses skill when correctly invoked
when_to_use: "STOP - use task-json instead"
```

### ❌ Dense Recipes in Router

```yaml
# BAD - defeats token efficiency
context: inherit
---
[500 lines of jq commands]
```

### ❌ Weak Router Description

```yaml
# BAD - loses routing competition
description: "JSON task router"
```

### ❌ Strong Recipe Triggers

```yaml
# BAD - recipes compete with router
when_to_use: "Use when: extracting JSON, filtering arrays..."
```

### ❌ Router Without Skill Tool

```yaml
# BAD - can't chain to recipes
allowed-tools: Read
```

---

## Implementation Checklist

For each skill pair:

### CLAUDE.md
- [ ] Contains "prefer more descriptive skill" guidance
- [ ] Explains the pattern with examples

### Router Skill
- [ ] Filename: `task-{domain}.md`
- [ ] `description`: Rich, action-oriented (10-15 words)
- [ ] `when_to_use`: 6-10 trigger words
- [ ] `context: inherit`
- [ ] `allowed-tools`: includes `Skill`
- [ ] Body: Context extraction instructions
- [ ] Body: Argument format with examples
- [ ] Body: Invocation instruction
- [ ] Size: 50-80 lines

### Recipe Skill
- [ ] Filename: `task-{domain}-recipes.md`
- [ ] `description`: Minimal with "(internal)"
- [ ] `when_to_use`: Describes input, not triggers
- [ ] `context: fork`
- [ ] `allowed-tools: Read`
- [ ] Body: Quick reference table
- [ ] Body: Categorized recipes
- [ ] Body: Common patterns
- [ ] Body: Edge cases
- [ ] Body: Installation
- [ ] Size: 200-600+ lines

### Testing
- [ ] Vague query routes to router
- [ ] Router extracts context correctly
- [ ] Router chains to recipes
- [ ] Recipes return useful commands
- [ ] Direct recipe invocation works with explicit args

---

## Implemented Skills

| Skill | Router | Recipes | Status |
|-------|--------|---------|--------|
| task-json | ✅ | ✅ | Complete |
| task-tabular | ✅ | ✅ | Complete |
| task-text-search | ✅ | ✅ | Complete |
| task-files | ✅ | ✅ | Complete |
| task-reverse-engineer | ✅ | ✅ | Complete |
| task-media | ✅ | ✅ | Complete |
| task-git | ✅ | ✅ | Complete |
| task-debug | ✅ | ✅ | Complete |
| task-network | ✅ | ✅ | Complete |
| task-containers | ✅ | ✅ | Complete |
| task-http | ✅ | ✅ | Complete |
| task-database | ✅ | ✅ | Complete |
| task-process | ✅ | ✅ | Complete |
| task-system | ✅ | ✅ | Complete |
| task-dev | ✅ | ✅ | Complete |
| task-api-test | ✅ | ✅ | Complete |
| task-crypto | ✅ | ✅ | Complete |
| task-logs | ✅ | ✅ | Complete |
| task-backup | ✅ | ✅ | Complete |

**Not recommended for two-tier** (explicit enough without router):
- task-archives, task-permissions, task-watch

---

## Success Metrics

| Metric | Target | How to Measure |
|--------|--------|----------------|
| Routing accuracy | >95% | Queries route to router, not recipes |
| Context extraction | >90% | Router finds file paths from conversation |
| Argument quality | >85% | Recipes receive actionable arguments |
| Token efficiency | >80% savings | Compare context growth with/without fork |
| Fallback success | 100% | Direct recipe invocation works |

---

## Reference Implementations

See the working implementations in `.claude/commands/`:

**Routers:**
- `task-json.md`
- `task-tabular.md`
- `task-text-search.md`
- `task-files.md`
- `task-reverse-engineer.md`
- `task-media.md`
- `task-git.md`
- `task-debug.md`
- `task-network.md`
- `task-containers.md`
- `task-http.md`
- `task-database.md`
- `task-process.md`
- `task-system.md`
- `task-dev.md`
- `task-api-test.md`
- `task-crypto.md`
- `task-logs.md`
- `task-backup.md`

**Recipes:**
- `task-json-recipes.md`
- `task-tabular-recipes.md`
- `task-text-search-recipes.md`
- `task-files-recipes.md`
- `task-reverse-engineer-recipes.md`
- `task-media-recipes.md`
- `task-git-recipes.md`
- `task-debug-recipes.md`
- `task-network-recipes.md`
- `task-containers-recipes.md`
- `task-http-recipes.md`
- `task-database-recipes.md`
- `task-process-recipes.md`
- `task-system-recipes.md`
- `task-dev-recipes.md`
- `task-api-test-recipes.md`
- `task-crypto-recipes.md`
- `task-logs-recipes.md`
- `task-backup-recipes.md`
