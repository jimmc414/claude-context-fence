# Agent Toolkit: Objective

## What This Project Is

A skill library for Claude Code that gives the agent fast, reliable access to ~275 CLI tools without paying the token cost of knowing them all upfront.

## The Problem

AI agents interact with the world through tools. On Linux, that means CLI utilities -- hundreds of them, each with dozens of flags. An agent that "knows" all of them carries enormous context overhead. An agent that doesn't know them reaches for slow, wrong, or overcomplicated alternatives.

Three specific failure modes:

1. **Hallucination.** The agent invents plausible but wrong flags. `sed -E 's/old/new/g'` becomes `sed --replace old new` because the agent is pattern-matching syntax from other tools. The command fails, the agent retries with another wrong guess, burning tokens and patience.

2. **Reinvention.** The agent writes a 40-line Python script to extract a field from JSON because it doesn't know `jq -r '.users[].email' file.json` exists. The script works, but it took 10x the tokens and introduced 10x the surface area for bugs.

3. **Context bloat.** Stuffing complete tool documentation into system prompts works for one tool. It doesn't work for 275. At ~250 tokens per tool reference, a full inventory would consume 70K tokens of context before the conversation even starts.

## The Solution

A two-tier skill architecture that separates **knowing when to use a tool** from **knowing how to use it**.

### Tier 1: Router Skills (~50 lines each)

Lightweight skills that live in the main conversation context. They have `context: inherit`, meaning they can see the full conversation history. Their job is narrow: figure out what the user actually needs, extract relevant details (file paths, constraints, output format) from the conversation, and pass a well-formed request to the recipe skill.

A router never contains tool documentation. It contains instructions for reading the room.

### Tier 2: Recipe Skills (~200-600 lines each)

Dense reference documents packed with commands, flags, patterns, and edge cases. They have `context: fork`, meaning they load in isolation -- their content doesn't bloat the main conversation. They receive a prepared argument from the router and return an executable command.

A recipe skill is a cookbook. It doesn't need to know what you had for dinner last night; it just needs to know what dish you're making.

### Why Two Tiers Instead of One

The fundamental tension: **context awareness** requires `context: inherit` (expensive, sees conversation), but **dense references** require `context: fork` (cheap, isolated). One skill can't be both without compromise. The two-tier split lets each tier do what it's good at.

Token math: a recipe skill with 300 lines of jq commands costs ~2500 tokens if loaded into the main context. With fork isolation, the main context grows by ~100 tokens (the router's handoff and the returned command). That's an 80%+ reduction per invocation, multiplied across every skill in the library.

## What "Working" Looks Like

A user says: *"grab the emails from that API response we were looking at"*

Without skills: The agent either hallucinates a jq command, writes a Python script, or asks the user to be more specific.

With the two-tier system:

1. The router skill (`task-json`) fires because the user said "API response" -- a trigger word in its description.
2. The router sees the full conversation. It finds the file path `/tmp/api-response.json` from 5 messages ago. It notices the user was looking at `.data.users[]` earlier. It builds: `"extract emails from /tmp/api-response.json - structure: .data.users[].contact.email, output: raw lines"`.
3. The recipe skill (`task-json-recipes`) receives this prepared argument in an isolated fork. It matches the request to the right jq pattern and returns: `jq -r '.data.users[].contact.email' /tmp/api-response.json`.
4. The command returns to the main context. The recipe content is discarded. The user gets a correct answer without the agent guessing or the context growing.

## Current State

### Deployed (19 two-tier pairs)

| Domain | Router | Recipes |
|--------|--------|---------|
| JSON | task-json | task-json-recipes |
| Tabular | task-tabular | task-tabular-recipes |
| Text Search | task-text-search | task-text-search-recipes |
| Files | task-files | task-files-recipes |
| Reverse Engineering | task-reverse-engineer | task-reverse-engineer-recipes |
| Media | task-media | task-media-recipes |
| Git | task-git | task-git-recipes |
| Debug | task-debug | task-debug-recipes |
| Network | task-network | task-network-recipes |
| Containers | task-containers | task-containers-recipes |
| HTTP | task-http | task-http-recipes |
| Database | task-database | task-database-recipes |
| Process | task-process | task-process-recipes |
| System | task-system | task-system-recipes |
| Dev | task-dev | task-dev-recipes |
| API Test | task-api-test | task-api-test-recipes |
| Crypto | task-crypto | task-crypto-recipes |
| Logs | task-logs | task-logs-recipes |
| Backup | task-backup | task-backup-recipes |

### Deployed (22 two-tier pairs total)

All 22 skills now use the two-tier architecture with `context: inherit` routers and `context: fork` recipe files.

### Supporting Documents

| Document | Purpose |
|----------|---------|
| `agent-toolkit-research.md` | Theoretical framework (Yegge's survival ratio, agent UX principles, tool taxonomy) |
| `agent-toolkit-comprehensive-inventory.md` | Catalog of ~275 tools with agent UX ratings and priority levels |
| `skill-pattern-two-tier.md` | Architecture spec for building new skill pairs |
| `complete-skills-reference.md` | Deployed skill documentation |
| `CLAUDE.md` | Routing guidance that steers the model to routers over recipes |

## Design Principles

**1. Recipes are expendable, context is not.** Fork isolation means we can make recipes as long as needed without worrying about context cost. The constraint is on routers -- they must stay small.

**2. Win the routing competition.** The router's `description` field must always be richer than the recipe's. The recipe description says "(internal)". This asymmetry ensures the model picks the router for user-facing queries and only hits recipes when explicitly chained.

**3. Graceful degradation.** If someone invokes `task-json-recipes` directly with explicit arguments, it works. The router is an optimization, not a gate. If the router can't determine the file path, it asks the user rather than guessing.

**4. Organize by task, not by tool.** Users don't say "I need awk." They say "sum the third column." Skills are named for what the user wants to do (task-tabular), not what tool they'll end up using (task-awk). The recipe skill decides which tool fits.

**5. Boring tools win.** The inventory favors established tools (jq, grep, ffmpeg) over trendy alternatives. Stable interfaces mean less hallucination. Extensive training data means better command generation. Single responsibility means composable pipelines. This aligns with the research finding that the most agent-friendly tools are often the oldest.

## What This Is Not

- **Not a tool installer.** Skills assume tools are already on the system. Installation hints are included in recipes as a convenience, not as a provisioning system.

- **Not a learning resource.** Skills are reference material optimized for an agent to produce correct commands, not tutorials for humans learning CLI tools.

- **Not a replacement for knowing the tools.** The agent still generates the final command. Skills reduce hallucination and improve coverage; they don't eliminate the need for the model to understand what it's doing.

- **Not a static system.** Skills can be added, split, or merged as usage patterns emerge. The two-tier architecture is designed to scale to arbitrary domain count without proportional context growth.
