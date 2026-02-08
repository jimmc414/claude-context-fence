---
description: "Process management - process won't die, port already in use, runaway process consuming resources, zombie processes, listing processes, signals, job control, parallel execution, and batch processing (ps, kill, lsof, xargs, parallel)"
when_to_use: "Use when: process won't die or isn't responding to kill, port already in use by unknown process, runaway process consuming CPU or memory, zombie processes accumulating, need to find what's using a port or file, running commands in parallel, batch processing files, setting process priority or resource limits. Triggers: process won't die, port already in use, runaway process, zombie process, what is using port, can't kill process, process stuck, ps, kill, process list, background job, parallel, xargs, nice, priority, timeout, lsof, fuser, pgrep, pkill, jobs, nohup, ulimit, cgroup, batch processing, kill process, parallel execution, memory hog, cpu hog, orphan process, resource limit, process tree, signal, daemon, process monitor."
when-to-use: "Use when: process won't die or isn't responding to kill, port already in use by unknown process, runaway process consuming CPU or memory, zombie processes accumulating, need to find what's using a port or file, running commands in parallel, batch processing files, setting process priority or resource limits. Triggers: process won't die, port already in use, runaway process, zombie process, what is using port, can't kill process, process stuck, ps, kill, process list, background job, parallel, xargs, nice, priority, timeout, lsof, fuser, pgrep, pkill, jobs, nohup, ulimit, cgroup, batch processing, kill process, parallel execution, memory hog, cpu hog, orphan process, resource limit, process tree, signal, daemon, process monitor."
argument-hint: "[describe what you want to do with processes]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Process Management Task Router

You have access to the **full conversation context**. Your job is to analyze the user's process management need and build a well-formed argument for the recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Process tasks are often about diagnosing, controlling, or parallelizing. Analyze the conversation to identify:

### 1. Goal
What process operation is needed?
- Find a process (by name, PID, resource usage)
- Kill/signal a process
- Run in background (survive logout)
- Parallelize a command
- Limit resources (CPU, memory, time)
- Find what's using a port/file
- Manage job control (fg, bg, jobs)
- Set priority/scheduling

### 2. Target
What process or command?
- Process name/PID from conversation
- Command to run in parallel
- File/port to check
- List of items to process in batch

### 3. Context
- Why kill it? (hung, high CPU, freeing port)
- Why parallel? (many files, batch processing)
- What resource is limited? (CPU, memory, file descriptors)
- Running as root?

### 4. Prior Findings
- ps/top output discussed
- Error messages (port already in use, permission denied)
- Previous kill attempts

---

## Goal --> Tool Guide

| Goal | Primary Tool | Argument Prefix |
|------|-------------|----------------|
| Find process | ps/pgrep | "find process" |
| Kill process | kill/pkill | "kill" |
| Background job | nohup/disown | "run in background" |
| Parallel execution | xargs -P/parallel | "run in parallel" |
| Limit time | timeout | "limit time" |
| Limit resources | ulimit/prlimit/cgroups | "limit resources for" |
| Set priority | nice/renice/ionice | "set priority for" |
| Port/file usage | lsof/fuser | "find what uses" |
| Process tree | pstree | "show process tree" |
| CPU affinity | taskset | "pin to CPU" |

---

## Argument Format

```
<goal> <target> - context: <relevant details>
```

### Examples

**Find high CPU process:**
```
find process using most CPU - need top 5 with full command lines
```

**Kill by name:**
```
kill all nginx workers - graceful first, force if needed
```

**Parallel file processing:**
```
run in parallel gzip on 500 .log files in /var/log/archive/ - use 8 cores
```

**Background with persistence:**
```
run in background ./server.py - survive terminal close, log to /tmp/server.log
```

**Resource limit:**
```
limit resources for ./build.sh - max 4GB memory, max 300s CPU time
```

**Port investigation:**
```
find what uses port 3000 - need PID and command
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-process-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Process name is ambiguous (multiple matches)
- Kill scope unclear (one instance vs all?)
- Parallel job could be destructive (confirm batch size)

**Use Read tool first if:**
- Need to verify PID is still running
- Need to check /proc/PID/status
