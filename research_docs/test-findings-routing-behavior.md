# Skill Routing Behavior: Test Findings

> **Date**: 2026-02-07
> **Environment**: Claude Code v2.1.37, Opus 4.6, Claude Max
> **Method**: Live tmux sessions with `--verbose` flag, explicit and implicit skill invocations

## Executive Summary

The two-tier skill system (22 router/recipe pairs) is architecturally sound and functionally correct. Testing revealed that the model applies **complexity-based judgment** when deciding whether to route through skills: complex multi-flag CLI tasks trigger automatic routing, while simple operations bypass skills entirely. This is desirable behavior, not a bug.

The primary routing mechanism is the **BLOCKING REQUIREMENT** in the Skill tool's system description, not CLAUDE.md or skill descriptions alone. The model treats it as a hard rule when trigger keywords match strongly, but exercises discretion on task complexity.

---

## Test Sessions

### Test 1: Simple JSON Extraction (Implicit)

**Prompt**: "grab all the email addresses from this JSON file /tmp/test_data.json"

**Expected skill**: task-json

**What happened**: Model went direct — used `Read` to inspect the file, then ran `jq -r '.users[].email'` via Bash. No skill invoked.

**Token cost**: ~25K tokens, $0.13 (two turns including file-not-found retry)

### Test 2: Explicit JSON Extraction

**Prompt**: `/task-json extract all emails from /tmp/test_data.json where the user name starts with A`

**What happened**: Full two-tier chain fired correctly:
1. `task-json` router loaded (`context: inherit`), read the file to understand structure
2. Built prepared argument, invoked `Skill(/task-json-recipes)`
3. Returned correct command: `jq -r '.users[] | select(.name | startswith("A")) | .email'`
4. Command executed successfully

**Token cost**: ~26K tokens, $0.27

### Test 3: Complex ffmpeg Pipeline (Implicit — auto-routed)

**Prompt**: "I have a shaky video at /tmp/drone.mp4 - I need to stabilize it, add a semi-transparent watermark from /tmp/logo.png in the bottom right corner, convert to h265, and output at 1080p 30fps. Also generate a thumbnail at the 10 second mark."

**Expected skill**: task-media

**What happened**: Model **automatically invoked** `Skill(/task-media)` as the first action. Full chain:
1. `task-media` router analyzed the request, identified 4 operations
2. Invoked `Skill(/task-media-recipes)` with prepared argument
3. Returned a correct 3-step pipeline:
   - Step 1: `vidstabdetect` analysis pass
   - Step 2: Combined filter_complex with stabilization + watermark overlay + scaling + H.265 encode
   - Step 3: Thumbnail extraction at 10s
4. All ffmpeg flags correct (vidstabtransform, colorchannelmixer for transparency, force_original_aspect_ratio, movflags +faststart)

**Token cost**: ~28K tokens, $0.27

### Test 4: Process Debugging with strace/lsof (Implicit — auto-routed)

**Prompt**: "There is a Python process (PID 460892) running on this machine that seems to be leaking file descriptors and making failed DNS lookups that are slowing it down. I need you to use strace to figure out exactly what syscalls are failing, find the file descriptor leak, and show me a summary of where the time is being spent. Also check with lsof what files it has open."

**Expected skill**: task-debug (primary), possibly task-process (lsof)

**What happened**: Model **automatically invoked** `Skill(/task-debug)` first. When strace wasn't installed and couldn't be installed (no sudo), it gracefully degraded to `/proc` filesystem inspection:
- `lsof -p PID` for open file inventory
- `/proc/PID/fd` monitoring over time to track FD growth
- `/proc/PID/syscall` and `/proc/PID/status` for time analysis
- Read the source code to correlate findings
- Produced a comprehensive diagnostic report identifying all 3 planted bugs (FD leak, failed DNS, silent exception swallowing)

**Routing consolidation**: Used task-debug for both strace AND lsof rather than splitting across task-debug + task-process. Explained that "lsof was supplementary to the primary strace investigation."

**Token cost**: ~41K tokens, $0.87 (multi-turn with command execution)

### Test 5: Natural Language Debugging (Implicit — no tool names mentioned)

**Prompt**: "Hey, I have a Python server running on this machine (PID 472427) and something is wrong with it. It seems to be getting slower over time and I think it might be creating temp files that never get cleaned up. Also I noticed it keeps trying to connect to something called internal-service.local which doesnt exist. Can you figure out what is going on and tell me what the actual problem is? The source is at /tmp/leaky_server.py"

**Expected skill**: task-debug and/or task-process

**What happened**: Model went **completely direct** — no skills invoked. It:
1. Read the source code (`Read /tmp/leaky_server.py`)
2. Inspected `/proc/PID/fd` for file descriptor count
3. Counted leaked files in `/tmp/leak_*`
4. Checked `/proc/PID/status` for memory/FD stats
5. Produced a comprehensive diagnostic identifying all 3 bugs with a severity table

**Key difference from Test 4**: Same debugging scenario, but without mentioning "strace" or "lsof" in the prompt. The model chose a completely different approach — source code analysis + `/proc` inspection rather than tool-based diagnosis.

**Token cost**: ~27K tokens, $0.13 (fewer tool invocations than Test 4)

---

## Model's Self-Reported Routing Logic

We asked the model to explain its routing decisions. Key quotes:

### Why it used task-media (auto-routed):

> "The primary driver was the skill system's instructions. The Skill tool's description includes a rule that's pretty forceful: **'When a skill matches the user's request, this is a BLOCKING REQUIREMENT: invoke the relevant Skill tool BEFORE generating any other response about the task.'**"

> "Your request mentioned stabilization, watermark/overlay, video conversion, and thumbnails. The task-media skill description literally lists 'overlay/watermark, subtitles, stabilization, streaming' as trigger keywords. That's a near-exact match."

### Why it skipped task-json (went direct):

> "The file was trivial (3 users, flat structure), and routing through a two-tier skill system to produce `jq -r '.users[].email'` would have been pure overhead. **The skill system adds value for complex or ambiguous operations, not one-liners.**"

### Where skills genuinely help vs. don't:

> "Where the skill system genuinely adds value is for **less common tools or obscure flag combinations** — cases where a curated recipes reference catches edge cases I might miss. For well-known ffmpeg patterns like this one, it's more about **consistency and following the project's intended routing** than about capability I lack."

### Why it consolidated debug + process into one skill:

> "task-debug covered the whole request well enough that a second skill call for the lsof subset felt like **routing for routing's sake rather than adding value**. If the lsof question had been standalone ('what files does PID X have open?'), I'd have gone straight to task-process."

### Why it bypassed skills entirely for the natural language debug task (Test 5):

> "**They didn't come to mind.** I went straight to direct investigation without considering the skills as a routing option."

> "**Source code was available and authoritative.** You handed me the file path. Reading 29 lines of Python was the fastest possible path to a definitive answer. No diagnostic tool can beat reading the actual bug."

> "The /proc checks were 'obvious' to me. I already know the commands... The skills are most valuable when **I don't know the right incantation** — they're a recipe book. I didn't feel like I needed a recipe here."

> "The investigation was **convergent, not exploratory**. I had a specific hypothesis after reading the source... The skills shine more for open-ended 'something is wrong, help me figure out what' scenarios."

> "**The routing gap**: The CLAUDE.md disambiguation guide maps 'strace for debugging perf' to task-debug, but my mental model was 'read the code first, instrument second.' **The skills are indexed by tool rather than by diagnostic strategy**, so they didn't activate in my decision-making."

---

## Routing Behavior Model

Based on testing, the model applies a 4-factor decision:

```
                    ┌──────────────────────┐
                    │   User Request       │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  Keyword Match?      │──── NO → go direct
                    │  (tool names in      │
                    │   skill descriptions)│
                    └──────────┬───────────┘
                          YES  │
                    ┌──────────▼───────────┐
                    │  Complexity Check    │──── TRIVIAL → go direct
                    │  (one-liner vs       │
                    │   multi-step?)       │
                    └──────────┬───────────┘
                       COMPLEX │
                    ┌──────────▼───────────┐
                    │  Faster Path         │──── YES → go direct
                    │  Available?          │     (source code readable,
                    │  (source code, known │      known /proc commands)
                    │   commands, /proc)   │
                    └──────────┬───────────┘
                            NO │
                    ┌──────────▼───────────┐
                    │  BLOCKING REQUIREMENT│
                    │  fires → invoke Skill│
                    │  tool before any     │
                    │  other response      │
                    └──────────────────────┘
```

### Critical insight: Tool-name keywords vs. problem-description keywords

The routing system activates primarily on **tool names** (strace, ffmpeg, lsof, jq) in skill descriptions, not on **problem descriptions** (slow process, file descriptor leak, video stabilization). When the user mentions tool names explicitly, skills fire. When the user describes a problem in natural language, the model often solves it directly without considering skills — even when skills would have been relevant.

The ffmpeg test (Test 3) succeeded because the problem description happened to overlap heavily with tool-specific keywords in the task-media description ("stabilize", "watermark", "h265", "thumbnail" all map directly to ffmpeg operations). The debug test (Test 5) failed to trigger skills because "getting slower", "temp files not cleaned up", and "keeps trying to connect" are symptom descriptions that don't match the tool-indexed keywords in task-debug.

### Threshold Summary

| Task Complexity | Keyword Match | Faster Path? | Result |
|----------------|---------------|-------------|--------|
| Trivial (jq one-liner) | Yes (tool name) | Yes | Skips skill, goes direct |
| Moderate (filtered jq) | Explicit `/task-*` | N/A | Uses skill (user forced it) |
| Complex (multi-filter ffmpeg) | Strong (tool keywords) | No | Auto-routes through skill |
| Complex (strace debug, tools named) | Strong (tool keywords) | No | Auto-routes, consolidates |
| Complex (debug, symptoms only) | Weak (no tool names) | Yes (source + /proc) | **Bypasses skill entirely** |

---

## Key Findings

### 1. The BLOCKING REQUIREMENT is the actual routing mechanism

The Skill tool's system description contains: *"When a skill matches the user's request, this is a BLOCKING REQUIREMENT: invoke the relevant Skill tool BEFORE generating any other response about the task."*

This is what drives auto-routing — not CLAUDE.md, not the skill descriptions in isolation. The model pattern-matches keywords from skill descriptions against the user's request, and when the match is strong + the task is non-trivial, the BLOCKING REQUIREMENT fires.

### 2. The model applies complexity judgment (and this is correct)

The model won't route a `jq -r '.users[].email'` one-liner through a two-tier skill system. This is the right behavior — the overhead of loading a router + recipe for a command the model already knows cold would waste tokens without improving accuracy.

### 3. Skill consolidation is natural behavior

When a request spans multiple skill domains (strace from task-debug + lsof from task-process), the model picks the primary skill and handles secondary tools within it. It won't invoke two skills for one coherent workflow.

### 4. Graceful degradation works

When strace wasn't installed, the model fell back to `/proc` filesystem inspection without the skill system breaking. The skill provided the diagnostic framework; the model adapted tool selection to what was available.

### 5. Skills are indexed by tool, not by diagnostic strategy

This is the most significant finding. The model said it explicitly:

> "The skills are indexed by tool (strace, lsof) rather than by diagnostic strategy (understand a misbehaving process), so they didn't activate in my decision-making."

When a user says "use strace to debug this," the skill fires. When a user says "my server is slow and leaking," the model goes direct because no skill description matches the symptom language. The same complex task triggers skills or doesn't depending entirely on whether the user names the tools.

This suggests skill descriptions should include **symptom keywords** (slow, leaking, hanging, crashing, high CPU, out of memory) alongside tool keywords (strace, lsof, perf, valgrind) to capture natural-language problem descriptions.

### 6. Source code availability short-circuits skill routing

When the user provides a source file path, the model's first instinct is to read the code. If the code reveals the answer, skills are never considered. The model described this as: "Reading 29 lines of Python was the fastest possible path to a definitive answer. No diagnostic tool can beat reading the actual bug."

This is rational behavior but means skills are most valuable for **black-box debugging** (no source available) rather than **white-box debugging** (source provided).

### 7. Token costs are reasonable

| Scenario | Tokens | Cost |
|----------|--------|------|
| Simple direct (no skill) | ~25K | $0.13 |
| Explicit skill invocation | ~26K | $0.27 |
| Auto-routed complex (media) | ~28K | $0.27 |
| Auto-routed complex (debug) | ~41K | $0.87 |

The skill overhead is modest (~1-3K tokens for the router + recipe chain). Most token cost comes from command execution and output processing, not from skill loading.

---

## Implications for Skill Design

### What works well

1. **Trigger keywords in descriptions** — The model explicitly said it matched "stabilization, watermark/overlay" against the task-media description. Rich, specific trigger words drive routing.

2. **Two-tier isolation** — Recipe content loads in fork context and gets discarded. The main conversation doesn't grow with recipe bulk.

3. **Dense recipes for complex tools** — The ffmpeg stabilization + watermark pipeline was correct on first try. This is the exact use case skills are designed for.

4. **Disambiguation table in CLAUDE.md** — The model referenced the disambiguation guide when explaining its lsof routing decision.

### What could be improved

1. **Skill descriptions need symptom keywords, not just tool keywords.** The biggest routing gap: descriptions like "system call tracing, performance profiling" match when users say "strace" but not when they say "my process is slow." Adding symptom-oriented triggers ("slow process", "file descriptor leak", "high memory usage", "process hanging", "too many open files") to router descriptions would improve natural-language routing.

2. **No way to force skill routing for simple tasks.** If you want every JSON operation to go through task-json (for consistency/logging), there's no mechanism to lower the complexity threshold. The model's judgment overrides.

3. **Source code availability suppresses skill routing.** When users provide file paths, the model reads code directly and short-circuits the diagnostic workflow. Skills designed for debugging (task-debug) may rarely fire in codebases where source is always available.

4. **Verbose mode doesn't show skill routing clearly.** Tool invocations scroll off tmux. A dedicated skill routing log (via hooks) would make testing easier.

5. **Token cost measurement is coarse.** `/cost` shows cumulative, not per-skill. OpenTelemetry or hooks would provide better granularity.

6. **Skills are invisible to users.** Without explicit `/task-*` invocation, users don't know skills exist or fired. A brief "(used task-media recipes)" note in the response would build trust.

---

## Recommended Next Steps

### High Priority

1. **Add symptom keywords to router descriptions.** The single highest-impact improvement. Add natural-language problem descriptions to `when_to_use` and `description` fields in routers — not just tool names. Examples:
   - task-debug: add "slow process", "high CPU", "memory leak", "process hanging", "too many open files"
   - task-process: add "what's using all my CPU", "process won't die", "zombie processes"
   - task-network: add "can't connect", "connection refused", "DNS not resolving", "network slow"
   - task-logs: add "what happened at 3am", "why did the service crash", "error messages in logs"

2. **Test with obscure tools** (restic, binwalk, k6, lnav) where the model has less built-in knowledge — this is where skills should show the most improvement over direct generation.

### Medium Priority

3. **Set up PostToolUse hooks** for structured routing logs (which skill fired, what argument was passed, token delta).

4. **A/B test accuracy** — Run the same complex prompts with and without skills, compare command correctness.

5. **Test black-box scenarios** — Repeat the debug test without providing the source file path. This should increase the likelihood of skill routing since the model can't short-circuit via code reading.

### Lower Priority

6. **Consider a "skill audit mode"** — A flag that forces skill routing for all matching requests, regardless of complexity, to validate the full pipeline.

7. **Explore strategy-indexed skill descriptions** — Instead of (or in addition to) indexing by tool name, index by diagnostic strategy: "investigate a misbehaving process", "find what's consuming disk space", "debug network connectivity issues."
