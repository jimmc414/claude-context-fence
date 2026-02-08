# Two-Tier Skill Routing

<!-- TEMPLATE: Copy this file into your project root as CLAUDE.md.
     Fill in the routing table and disambiguation table with your
     actual skill pairs, then delete these HTML comments. -->

## Skill Routing Rules

This project uses two-tier skills. Each skill domain has a **router**
and a **recipes** file in `.claude/commands/`. Always prefer the router.

1. **Always invoke the router skill**, never the recipes skill directly.
   The router has `context: inherit` and sees the full conversation.
2. **Skills marked "(internal)" are recipes stores.** The router calls
   them automatically. Do not invoke them for user requests.
3. **The router extracts context** (goals, file paths, constraints,
   prior errors) from the conversation and builds a structured argument
   before calling recipes. Skipping the router loses this context.

## Skills Reference

<!-- Replace these rows with your actual skill pairs.
     One row per domain. Router = user-facing, Recipes = internal. -->

| Router (invoke this) | Recipes (auto-called) |
|----------------------|-----------------------|
| `domain-a`           | `domain-a-recipes`    |
| `domain-b`           | `domain-b-recipes`    |
| `domain-c`           | `domain-c-recipes`    |

## Skill Disambiguation

<!-- Add rows here when two skills could plausibly match the same
     user request. This gives the model an explicit tiebreaker.
     Delete this section if you have fewer than 3 skills. -->

When a request could match multiple skills, use this table:

| User Request Pattern              | Use This   | Not This   |
|-----------------------------------|------------|------------|
| "request that sounds like A"      | domain-a   | domain-b   |
| "request that sounds like B"      | domain-b   | domain-a   |

<!-- Disambiguation is needed when domains share a tool or concept.
     Example: "strace" could match both a debugging skill and a
     reverse-engineering skill. The user's intent is the tiebreaker. -->

## Adding a New Skill Pair

### Router (`.claude/commands/my-domain.md`)

```yaml
---
description: "Domain name - user symptom 1, symptom 2, what goes wrong, what they want to do (tool1, tool2, tool3)"
when_to_use: "Use when: scenario1, scenario2. Triggers: keyword1, keyword2."
context: inherit
allowed-tools: Skill, Read
---
```

- Lead the description with user problems, not tool names.
- Body: context extraction sections, argument format template, invocation of recipes skill.
- Target: 50-160 lines.

### Recipes (`.claude/commands/my-domain-recipes.md`)

```yaml
---
description: "Domain recipes reference (internal)"
context: fork
allowed-tools: Read
---
```

- Description must be short and marked "(internal)" so it loses the routing competition.
- Body: quick-reference table, categorized recipes, tool selection guidance.
- Size is unbounded. The `context: fork` boundary means recipes content never enters the main context window.

### Then update this file

Add one row to the Skills Reference table. If the new domain overlaps with
existing domains, add disambiguation rows.
