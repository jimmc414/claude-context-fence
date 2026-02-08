# Complete CLI Toolkit Skills Reference

> **This file is deprecated.** It was a snapshot of skill contents that became stale as skills evolved.

## Canonical References

| What You Need | Where To Find It |
|---------------|-----------------|
| Browse all skills | `.claude/commands/toolkit-master.md` or use `/toolkit-master` |
| Routing guidance | `CLAUDE.md` (Two-Tier Skills Reference + Disambiguation Guide) |
| Actual deployed skills | `.claude/commands/task-*.md` (source of truth) |
| Architecture spec | `skill-pattern-two-tier.md` |
| Tool inventory | `agent-toolkit-comprehensive-inventory.md` |

## Current Deployment

- **22 task skills** (19 two-tier pairs + 3 standalone) + 1 master index = 23 total skill files
- All deployed to `.claude/commands/`
- Skills are lazy-loaded via `context: fork` (recipes) or `context: inherit` (routers)
