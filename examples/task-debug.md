---
description: "Debugging and profiling - program crashes randomly, CPU at 100%, takes forever to run, eating all my RAM, why is it slow, memory leak, segfault, system call tracing, performance profiling, and benchmarking (strace, perf, valgrind, gdb, hyperfine, py-spy)"
when_to_use: "Use when: program crashes randomly or intermittently, CPU usage at 100%, program takes forever to run, application eating all RAM, wondering why something is slow, suspecting a memory leak, getting segfaults or core dumps, need to trace system calls, profile CPU usage, benchmark command performance, debug process hangs. Triggers: program crashes, CPU at 100%, takes forever, eating all RAM, why is it slow, memory leak, segfault, core dump, process hangs, getting slower over time, file descriptor leak, strace, trace, profile, perf, valgrind, benchmark, gdb, debug, flamegraph, performance, slow process, hang, fault injection, syscall, high cpu, out of memory, trace calls, latency, bottleneck, flame graph, cache miss, io wait, node profiling, python profiling, clinic, 0x, py-spy, scalene, hyperfine."
when-to-use: "Use when: program crashes randomly or intermittently, CPU usage at 100%, program takes forever to run, application eating all RAM, wondering why something is slow, suspecting a memory leak, getting segfaults or core dumps, need to trace system calls, profile CPU usage, benchmark command performance, debug process hangs. Triggers: program crashes, CPU at 100%, takes forever, eating all RAM, why is it slow, memory leak, segfault, core dump, process hangs, getting slower over time, file descriptor leak, strace, trace, profile, perf, valgrind, benchmark, gdb, debug, flamegraph, performance, slow process, hang, fault injection, syscall, high cpu, out of memory, trace calls, latency, bottleneck, flame graph, cache miss, io wait, node profiling, python profiling, clinic, 0x, py-spy, scalene, hyperfine."
argument-hint: "[describe what you want to debug or profile]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Debug & Profiling Task Router

You have access to the **full conversation context**. Your job is to analyze the user's debug/profiling need and build a well-formed argument for the recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Debug requests are symptom-driven -- users describe what's wrong, not which tool to use. Analyze the conversation to identify:

### 1. Symptom / Goal
What's the problem or what does the user want to understand?
- Process is slow --> strace -c, then perf
- Process hangs --> strace -p PID
- Segfault/crash --> gdb, valgrind
- Memory leak --> valgrind --leak-check, heaptrack
- Need syscall detail --> strace
- Need fault testing --> strace -e inject=
- Need benchmarking --> hyperfine
- Need CPU profiling --> perf record, py-spy, scalene
- Need flamegraph --> perf + flamegraph.pl, py-spy
- General "what does this do" --> strace -f -tt -T -yy -s 512

### 2. Target
What are we debugging?
- Command to run (./myapp, python script.py)
- Running process (PID)
- Binary path
- Container process

### 3. Environment Context
- Running in Docker/K8s? (needs SYS_PTRACE)
- What OS/distro? (tool availability)
- Root access available?
- Language runtime? (Java, Python, Node patterns differ)

### 4. Prior Findings
- Error messages from earlier in conversation
- Previous strace/perf output discussed
- What's already been tried

### 5. Scope
- Specific syscall categories? (file, network, memory)
- Duration/count limits?
- Output format needs?

---

## Symptom --> Tool Guide

| Symptom | Primary Tool | Argument Prefix |
|---------|-------------|----------------|
| Slow process | strace -c, then perf | "diagnose slow process" |
| Process hangs | strace -p | "diagnose hanging process" |
| Segfault/crash | gdb / valgrind | "debug crash in" |
| Memory leak | valgrind / heaptrack | "find memory leak in" |
| Syscall tracing | strace | "trace syscalls of" |
| Fault injection | strace -e inject= | "inject fault into" |
| CPU profiling | perf / py-spy / scalene | "profile CPU usage of" |
| Benchmarking | hyperfine | "benchmark" |
| Permission errors | strace -e trace=%file -Z | "diagnose permission errors in" |
| Network issues | strace -e trace=%network | "trace network activity of" |

---

## Argument Format

Construct a structured argument string:

```
<symptom/goal> on <target> - tool: <if known>, focus: <category>, context: <prior findings>
```

**Include only relevant components.**

### Examples

**Slow process:**
```
diagnose slow process on ./myapp - no prior findings
```

**Hanging process:**
```
diagnose hanging process on PID 1234 - stuck for 5 minutes, serves HTTP
```

**Syscall trace with focus:**
```
trace syscalls of /usr/bin/myapp - focus: file and network operations, running in Docker
```

**Fault injection:**
```
inject fault into ./myapp - simulate disk full on write operations
```

**Memory leak:**
```
find memory leak in ./server - grows 50MB/hour, C++ binary
```

**Benchmark:**
```
benchmark 'sort large_file.txt' vs 'sort --parallel=4 large_file.txt'
```

**Python profiling:**
```
profile CPU usage of app.py - focus: hot functions, generate flamegraph
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-debug-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Symptom is vague ("it's broken") -- need more specifics
- Multiple unrelated symptoms -- prioritize which to investigate first
- Production system -- confirm acceptable overhead before tracing

**Use Read tool first if:**
- Need to verify PID is running
- Need to check if target binary exists
- Need to examine prior strace/perf output files mentioned in conversation
