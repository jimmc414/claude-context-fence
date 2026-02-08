---
description: "Debug and profiling recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-debug with target, symptom, and context included"
when-to-use: "Receives prepared arguments from task-debug with target, symptom, and context included"
context: fork
model: inherit
allowed-tools: Read
---

# Debug & Profiling Recipes

**Tools:** strace, ltrace, perf, bpftrace, gdb, lldb, valgrind, massif, heaptrack,
py-spy, scalene, memray, austin, hyperfine, time, flamegraph, clinic, 0x

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Trace syscalls | `strace -f -tt -T -yy -s 512 -o trace.log ./command` |
| Attach to running | `strace -p PID` |
| Syscall stats | `strace -c ./command` |
| Failed calls only | `strace -Z ./command` |
| Inject fault | `strace -e inject=write:error=ENOSPC ./command` |
| Profile CPU | `perf record -g ./command && perf report` |
| Memory leaks | `valgrind --leak-check=full ./command` |
| Benchmark | `hyperfine 'command1' 'command2'` |
| Python profiler | `py-spy record -o flame.svg -- python app.py` |
| Debug binary | `gdb ./program` |
| Trace library calls | `ltrace ./command` |
| Dynamic tracing | `bpftrace -e 'tracepoint:syscalls:sys_enter_open { printf("%s\n", str(args->filename)); }'` |
| List core dumps | `coredumpctl list` |
| Debug core dump | `coredumpctl debug PID` |
| Node.js profiling | `npx clinic doctor -- node app.js` |

---

## When to Use What

| Tool | Best For | Limitation |
|------|----------|------------|
| strace | Syscall tracing, file/network debugging | High overhead, Linux only |
| ltrace | Library/function call tracing | Less maintained, can miss inlined calls |
| bpftrace | Dynamic kernel/userspace tracing, low overhead | Requires root, newer kernel (4.9+) |
| perf | CPU profiling, PMU counters, flamegraphs | Linux only, needs debug symbols for best results |
| gdb | Interactive debugging, breakpoints, core dumps | Steep learning curve |
| lldb | Same as gdb but for LLVM/macOS | Less Linux ecosystem support |
| valgrind | Memory leak detection, cache profiling | 10-50x slowdown |
| heaptrack | Heap allocation profiling | Linux only, needs debug symbols |
| py-spy | Python CPU profiling (no code changes) | Sampling only, no memory |
| clinic | Node.js performance diagnosis | Node.js only |
| hyperfine | Command benchmarking with statistics | Measures whole-command, not functions |

---

## System Call Tracing and Process Debugging (strace)

### Quick Reference

| Task | Command |
|------|---------|
| Trace a command | `strace ./command` |
| Attach to running process | `strace -p PID` |
| Follow child processes | `strace -f ./command` |
| Separate file per PID | `strace -ff -o trace ./command` |
| File operations only | `strace -e trace=%file ./command` |
| Network operations only | `strace -e trace=%network ./command` |
| Syscall statistics | `strace -c ./command` |
| Show timing per syscall | `strace -T ./command` |
| Full-detail golden standard | `strace -f -tt -T -yy -s 512 -o trace.log ./command` |
| Decode file descriptors | `strace -yy ./command` |
| Failed syscalls only | `strace -Z ./command` |
| Inject fault (disk full) | `strace -e inject=write:error=ENOSPC ./command` |

---

### Basic Usage

#### Trace a Command

```bash
strace ./myapp                        # Trace from start to exit
strace -o trace.log ./myapp           # Output to file (stderr stays clean)
strace -A -o trace.log ./myapp        # Append to existing trace file
```

#### Attach to a Running Process

```bash
strace -p 1234                        # Attach to PID 1234
strace -p 1234 -p 5678               # Attach to multiple PIDs
strace -p $(pgrep -f myapp)          # Attach by name
```

#### Follow Forks and Children

```bash
strace -f ./myapp                     # Follow all child processes
strace -ff -o trace ./myapp           # Separate file per PID (trace.PID)
strace --output-separately -o trace ./myapp  # Same as -ff -o
```

#### Output Control

```bash
strace -s 512 ./myapp                 # Show 512 bytes of strings (default: 32)
strace -s 0 ./myapp                   # Suppress string output entirely
strace -x ./myapp                     # Hex output for non-printable chars
strace -xx ./myapp                    # All string output in hex
strace -X verbose ./myapp             # Named constants (default)
strace -X raw ./myapp                 # Raw numeric values for all arguments
```

#### Verbose and Decode Modes

```bash
strace -v ./myapp                     # Verbose: don't abbreviate structs
strace -y ./myapp                     # Decode FD numbers to paths
strace -yy ./myapp                    # Full FD decode: socket addresses, device info
strace --trace-fds=3,4,5 ./myapp      # Trace only FDs 3, 4, 5 (v5.5+)
strace --trace-fds=!0,1,2 ./myapp    # Trace all FDs except stdin/stdout/stderr
strace -n ./myapp                     # Print syscall numbers
strace -N ./myapp                     # Argument names in output (v6.16+, --arg-names)
```

#### Daemonize strace

```bash
strace -D ./myapp                     # Daemonize: strace runs as detached grandchild
strace -DD ./myapp                    # Suppress strace exit status
strace -DDD ./myapp                   # Suppress strace and tracee exit status
```

Daemonize is useful for tracing services where the parent process must exit cleanly.

---

### Filtering Syscalls

#### By Name

```bash
strace -e trace=open,read,write ./myapp         # Specific syscalls
strace -e trace=openat,read,write,close ./myapp  # Modern Linux uses openat
strace -e trace=!mmap,brk,mprotect ./myapp       # Exclude noisy syscalls
```

#### By Category

| Category | Alias | What It Captures |
|----------|-------|-----------------|
| `%file` | `file` | open, stat, chmod, chown, link, unlink, rename, etc. |
| `%network` | `network` | socket, bind, connect, send, recv, etc. |
| `%process` | `process` | fork, clone, exec, wait, exit, etc. |
| `%signal` | `signal` | signal, sigaction, kill, sigprocmask, etc. |
| `%ipc` | `ipc` | shmget, semop, msgget, etc. |
| `%desc` | `desc` | read, write, close, dup, poll, select, etc. |
| `%memory` | `memory` | mmap, mprotect, brk, madvise, etc. |
| `%stat` | `stat` | stat, fstat, lstat, statx, etc. |
| `%pure` | `pure` | getpid, getuid, etc. (no side effects) |
| `%creds` | `creds` | capget, capset, setuid, setgid, etc. |
| `%clock` | `clock` | clock_gettime, gettimeofday, etc. |

```bash
strace -e trace=%file ./myapp                    # All file operations
strace -e trace=%network ./myapp                 # All network operations
strace -e trace=%file,%network ./myapp           # Combine categories
strace -e trace=!%memory,!%signal ./myapp        # Exclude categories
```

#### By Path

```bash
strace -P /etc/config.yaml ./myapp               # Only syscalls touching this path
strace -P /etc -P /var/log ./myapp               # Multiple path filters
```

#### By Return Status

```bash
strace -z ./myapp                     # Successful syscalls only
strace -Z ./myapp                     # Failed syscalls only
strace -e status=failed ./myapp       # Same as -Z
strace -e status=successful ./myapp   # Same as -z
strace -e status=unfinished ./myapp   # Interrupted/unfinished calls
strace -e status=unavailable ./myapp  # Calls with no return (exec, exit)
strace -e status=detached ./myapp     # Calls interrupted by detach
```

#### By Architecture (Multi-ABI)

```bash
strace -e trace=/open@64 ./myapp      # 64-bit open calls only
strace -e trace=/open@32 ./myapp      # 32-bit compat open calls only
```

#### Noise Reduction Pipeline

Common strace noise -- calls that almost every program makes but rarely matter:

```bash
# Filter common noise from trace output
strace -o /dev/stdout ./myapp 2>&1 | \
  grep -vE '(mmap|mprotect|munmap|brk|clock_gettime|gettimeofday|getpid|getuid|arch_prctl|set_tid_address|set_robust_list|prlimit64|rseq|getrandom)'
```

#### Performance-Optimized Filtering

```bash
strace --seccomp-bpf -e trace=%file ./myapp      # seccomp-BPF filter (v5.3+)
# ~2x faster than ptrace-only filtering -- kernel skips non-matching syscalls
# without stopping the process

strace --syscall-limit=1000 ./myapp               # Stop after 1000 syscalls (v5.8+)
strace --syscall-limit=100 -e trace=%network ./myapp  # Cap network syscall capture
```

`--seccomp-bpf` works by installing a BPF filter in the kernel that only triggers ptrace stops for matching syscalls. Unmatched syscalls execute at native speed.

---

### Timestamps & Timing

#### Timestamp Modes

```bash
strace -t ./myapp                     # HH:MM:SS
strace -tt ./myapp                    # HH:MM:SS.usec (microsecond precision)
strace -ttt ./myapp                   # Epoch seconds.usec (for programmatic parsing)
strace -r ./myapp                     # Relative: time since previous syscall
```

#### Fine-Grained Precision (v5.11+)

```bash
strace --absolute-timestamps=time:ns ./myapp      # Nanosecond timestamps
strace --absolute-timestamps=format:unix,time:us ./myapp  # Unix epoch with microseconds
strace --syscall-times=ns ./myapp                  # Nanosecond per-call durations
```

#### Per-Syscall Duration

```bash
strace -T ./myapp                     # Duration in angle brackets: <0.000123>
strace -T -e trace=read ./myapp       # Duration of read() calls only
```

#### Wall Clock Time (v5.0+)

```bash
strace -w -c ./myapp                  # Summary includes wall-clock time (not just CPU)
# Critical for I/O-bound processes where time is spent waiting, not computing
```

#### Statistics

```bash
strace -c ./myapp                     # Summary table only (no trace output)
strace -C ./myapp                     # Both trace output AND summary table
strace -c -S time ./myapp             # Sort summary by cumulative time
strace -c -S calls ./myapp            # Sort by call count
strace -c -S errors ./myapp           # Sort by error count
strace -c -S name ./myapp             # Sort alphabetically
strace -c -U name,calls,errors ./myapp  # Custom columns in summary (v5.7+)
strace -w -c -S time ./myapp          # Sort by wall-clock time (best for I/O analysis)
```

#### Find Slow Syscalls from Trace Output

```bash
# Extract syscalls taking more than 1 second
awk -F'[<>]' 'NF>1 && $(NF-1)+0 > 1.0' trace.log

# Top 20 slowest syscalls
awk -F'[<>]' 'NF>1 {print $(NF-1), $0}' trace.log | sort -rn | head -20

# Average duration per syscall type
awk -F'[<>]' 'NF>1 {match($1, /^[0-9: .]* ([a-z_]+)\(/, a); if(a[1]) {s[a[1]]+=$(NF-1); c[a[1]]++}} END {for(k in s) printf "%s: avg=%.6fs count=%d total=%.6fs\n", k, s[k]/c[k], c[k], s[k]}' trace.log | sort -t= -k4 -rn
```

---

### Fault Injection

Fault injection (v4.16+) allows you to simulate errors, delays, and signals without modifying application code. This is strace's most underused power feature.

#### Full Syntax

```
-e inject=SET[:error=ERRNO][:retval=VAL][:signal=SIG][:delay_enter=US][:delay_exit=US][:when=EXPR][:poke_enter=@ARG=DATAV][:poke_exit=@ARG=DATAV]
```

Where SET is a syscall set (names, categories, or `all`).

#### Error Injection

```bash
# Disk full
strace -e inject=write:error=ENOSPC ./myapp

# I/O error on reads
strace -e inject=read:error=EIO ./myapp

# Permission denied on file opens
strace -e inject=openat:error=EACCES ./myapp

# File not found
strace -e inject=openat:error=ENOENT ./myapp

# Connection refused
strace -e inject=connect:error=ECONNREFUSED ./myapp

# Network unreachable
strace -e inject=connect:error=ENETUNREACH ./myapp

# Connection reset
strace -e inject=read:error=ECONNRESET ./myapp

# Out of memory
strace -e inject=mmap:error=ENOMEM ./myapp
strace -e inject=brk:error=ENOMEM ./myapp
```

#### Return Value Injection

```bash
# Force write() to return 0 bytes written (short write)
strace -e inject=write:retval=0 ./myapp

# Force read() to return 0 (simulate EOF)
strace -e inject=read:retval=0 ./myapp
```

#### Signal Injection

```bash
# Deliver SIGKILL on write
strace -e inject=write:signal=SIGKILL ./myapp

# Deliver SIGSEGV on mmap
strace -e inject=mmap:signal=SIGSEGV ./myapp
```

#### Delay Injection (Latency Simulation)

```bash
# 100ms delay before each read enters kernel
strace -e inject=read:delay_enter=100000 ./myapp

# 50ms delay after each write completes
strace -e inject=write:delay_exit=50000 ./myapp

# Simulate slow network (500ms on connect, 100ms on send/recv)
strace -e inject=connect:delay_enter=500000 \
       -e inject=sendto,recvfrom:delay_enter=100000 ./myapp

# Simulate slow disk (200ms on every file operation)
strace -e inject=%file:delay_enter=200000 ./myapp
```

#### Conditional Injection (when=)

```bash
# Fail only the 5th write call
strace -e inject=write:error=ENOSPC:when=5 ./myapp

# Fail from the 3rd write onward
strace -e inject=write:error=ENOSPC:when=3+ ./myapp

# Fail writes 2 through 10
strace -e inject=write:error=ENOSPC:when=2..10 ./myapp

# Fail every 3rd write starting from the 1st
strace -e inject=write:error=ENOSPC:when=1+3 ./myapp

# Fail every 2nd read starting from the 5th
strace -e inject=read:error=EIO:when=5+2 ./myapp
```

#### Chaos Engineering Recipes

```bash
# Simulate intermittent disk full (every 10th write fails)
strace -e inject=write:error=ENOSPC:when=1+10 ./myapp

# Simulate flaky network (every 3rd recv fails with reset)
strace -e inject=recvfrom:error=ECONNRESET:when=1+3 ./myapp

# Simulate slow + failing disk (200ms delay, then EIO after 50 reads)
strace -e inject=read:delay_enter=200000:error=EIO:when=50+ ./myapp

# Test OOM handling (fail the 1st mmap, let subsequent ones succeed)
strace -e inject=mmap:error=ENOMEM:when=1..1 ./myapp

# Combined: slow network connect + intermittent send failure
strace -e inject=connect:delay_enter=2000000 \
       -e inject=sendto:error=EPIPE:when=5+ ./myapp
```

---

### Multi-Process Tracing

#### Follow Forks

```bash
strace -f ./myapp                     # Follow all fork/clone/vfork children
# Output interleaves PIDs: [pid 1234] read(3, ...) = 512
```

#### Separate Output Files

```bash
strace -ff -o trace ./myapp           # Creates trace.PID for each process
strace --output-separately -o trace ./myapp  # Equivalent long form

# Merge sorted by timestamp afterward
strace-log-merge trace > merged.log   # Merge all trace.PID files
strace-log-merge trace.123 trace.456  # Merge specific files
```

#### Detach on Exec

```bash
strace -b execve ./wrapper.sh         # Detach when child calls exec
# Useful for tracing only the launcher, not the launched program
```

#### Interrupt Handling

```bash
strace -I1 ./myapp    # Default: no signals blocked
strace -I2 ./myapp    # SIGINT/SIGQUIT/SIGTERM ignored while tracee runs
strace -I3 ./myapp    # SIGINT/SIGQUIT/SIGTERM cause strace to detach (tracee continues)
strace -I4 ./myapp    # Like -I3 but also for SIGTERM
```

#### Practical Multi-Process Patterns

```bash
# Trace all Apache child workers
strace -f -p $(pgrep -o httpd) -e trace=%network -o apache-net.log

# Trace shell pipeline (each pipe segment is a child)
strace -f -ff -o pipeline bash -c 'cat /var/log/syslog | grep error | wc -l'

# Trace only the child (not the fork parent)
strace -f -b execve -e trace=%file ./launcher.sh
```

---

### Output Analysis

#### Output Format

Standard strace output line:

```
[PID] [timestamp] syscall(arg1, arg2, ...) = return_value [errno (description)] <duration>
```

Example lines:

```
openat(AT_FDCWD, "/etc/passwd", O_RDONLY) = 3
read(3, "root:x:0:0:root:/root:/bin/bash\n"..., 4096) = 1523
openat(AT_FDCWD, "/no/such/file", O_RDONLY) = -1 ENOENT (No such file or directory)
```

#### Unfinished and Resumed Calls

When a syscall is interrupted (common with `-f`):

```
[pid 1234] read(3,  <unfinished ...>
[pid 1235] write(4, "data", 4)  = 4
[pid 1234] <... read resumed>"result", 4096) = 6
```

#### Hex Dump I/O

```bash
strace -e read=3 ./myapp              # Hex dump all reads from FD 3
strace -e write=1,2 ./myapp           # Hex dump writes to stdout/stderr
strace -e read=all ./myapp            # Hex dump all read data
strace -e write=all ./myapp           # Hex dump all written data
```

#### Stack Traces (v4.9+)

```bash
strace -k ./myapp                     # Print userspace stack trace per syscall
strace -k -e trace=openat ./myapp     # Stack traces only for openat
# Requires libunwind; install: sudo apt install libunwind-dev
# Then rebuild strace from source with --with-libunwind
```

#### Grep Patterns for Trace Files

```bash
# All errors
grep ' = -1 ' trace.log

# Specific errno
grep 'ENOENT' trace.log               # File not found
grep 'EACCES' trace.log               # Permission denied
grep 'ECONNREFUSED' trace.log         # Connection refused
grep 'ETIMEDOUT' trace.log            # Connection timeout

# All file opens (with paths)
grep 'openat(' trace.log

# All file opens that failed
grep 'openat(.*= -1' trace.log

# Network connections (destination addresses)
grep 'connect(' trace.log

# Signals received
grep '---' trace.log                  # Signal delivery lines: --- SIGCHLD {si_signo=...} ---

# Specific FD activity
grep -E '(read|write|close|ioctl)\(3,' trace.log   # All activity on FD 3
```

#### awk Extraction Patterns

```bash
# Extract all file paths opened
awk -F'"' '/openat\(/ {print $2}' trace.log | sort -u

# Count syscalls by type
awk '{match($0, /^[0-9: .]* ([a-z_]+)\(/, a); if(a[1]) c[a[1]]++} END {for(k in c) print c[k], k}' trace.log | sort -rn

# Extract durations and find outliers (>100ms)
awk -F'[<>]' 'NF>1 && $(NF-1)+0 > 0.1 {print $(NF-1), $0}' trace.log | sort -rn

# Summarize errors by type
awk '/= -1/ {match($0, /E[A-Z]+/, e); if(e[0]) c[e[0]]++} END {for(k in c) print c[k], k}' trace.log | sort -rn

# Timeline: syscalls per second
awk -F'[:. ]' '/^[0-9]/ {sec=$1":"$2":"$3; c[sec]++} END {for(k in c) print k, c[k]}' trace.log | sort
```

#### Noise Reduction Recipes

```bash
# Exclude startup noise (loader, memory mapping)
strace -e trace=!mmap,mprotect,munmap,brk,arch_prctl,set_tid_address,set_robust_list,rseq,prlimit64,getrandom ./myapp

# Trace only after startup (skip first N calls)
strace -e trace=%file ./myapp 2>&1 | tail -n +50

# Exclude clock queries (frequent in event loops)
strace -e trace=!clock_gettime,gettimeofday,clock_nanosleep ./myapp
```

---

### Cross-Tool Integration

#### strace + /proc

```bash
# Map FD numbers to actual files/sockets
ls -la /proc/PID/fd/
# Example output:
# 3 -> /etc/passwd
# 4 -> socket:[12345]
# 5 -> pipe:[67890]

# Read kernel stack of a stuck process
cat /proc/PID/stack
# Shows where in the kernel the process is blocked

# Check process memory map (correlate with mmap calls)
cat /proc/PID/maps

# See process limits (correlate with EMFILE/ENOMEM)
cat /proc/PID/limits

# File descriptor count
ls /proc/PID/fd/ | wc -l

# Current syscall being executed
cat /proc/PID/syscall
```

#### strace + lsof

```bash
# List all open files for a process
lsof -p PID

# Correlate strace FD numbers with lsof output
strace -yy -p PID &                   # Trace with FD decode in background
lsof -p PID                           # Snapshot of current FDs

# Find which process has a file open
lsof /path/to/file                    # Then strace -p that PID

# Monitor network connections alongside strace
lsof -i -p PID                        # TCP/UDP connections
```

#### strace + ss/netstat

```bash
# Socket state alongside strace network tracing
ss -tnp | grep PID                    # TCP socket states
ss -unp | grep PID                    # UDP socket states

# Full socket detail for correlation
ss -tnpe | grep PID                   # Includes timer, uid, inode

# Workflow: identify stuck connections
strace -e trace=%network -p PID &     # Watch syscalls
ss -tn state time-wait dst ADDR       # Check socket states
```

#### strace + tcpdump

Two complementary perspectives -- application layer (strace) vs wire (tcpdump):

```bash
# Terminal 1: application-level view
strace -tt -e trace=%network -p PID -o app.log

# Terminal 2: network-level view
tcpdump -i any -tt -w wire.pcap port PORT

# Correlate by timestamp: -tt gives microsecond precision in both tools
# strace shows: what the app asked for (connect, send, recv args)
# tcpdump shows: what actually went on the wire (retransmits, resets, latency)
```

#### strace + gdb

```bash
# In gdb: catch specific syscalls
(gdb) catch syscall write
(gdb) catch syscall openat
(gdb) run
# Stops at each matching syscall with full backtrace available

# gdb syscall with condition
(gdb) catch syscall write
(gdb) condition 1 $rdi == 2           # Only when writing to stderr (FD 2)
```

**Limitation**: strace and gdb both use ptrace -- you cannot attach both to the same process simultaneously. Use one or the other, or use gdb's `catch syscall` for combined debugging.

#### strace + perf

```bash
# Low-overhead syscall counting (alternative to strace -c)
perf stat -e 'syscalls:sys_enter_*' ./myapp

# perf trace: eBPF-based tracing with lower overhead
perf trace ./myapp                     # Similar output to strace
perf trace -s ./myapp                  # Summary mode (like strace -c)
perf trace -e openat ./myapp           # Filter by syscall

# Combined workflow: strace for detail, perf for overhead measurement
perf stat strace -c ./myapp            # Measure strace's own overhead
```

#### strace + bpftrace

| Aspect | strace | bpftrace |
|--------|--------|----------|
| **Mechanism** | ptrace (2 context switches/syscall) | eBPF (in-kernel, zero copies) |
| **Overhead** | 2-100x slowdown | <5% typically |
| **Detail** | Full argument decode, return values | Customizable probes |
| **Production** | Not recommended | Designed for production |
| **Ease of use** | One command, zero setup | Requires script writing |
| **Fault injection** | Built-in (`-e inject=`) | Not built-in |
| **FD decode** | Automatic (`-yy`) | Manual via /proc |

```bash
# bpftrace equivalent of strace -e trace=openat -p PID
bpftrace -e 'tracepoint:syscalls:sys_enter_openat /pid == PID/ { printf("%s %s\n", comm, str(args.filename)); }'

# bpftrace for production syscall latency histogram
bpftrace -e 'tracepoint:syscalls:sys_enter_read /pid == PID/ { @start[tid] = nsecs; }
tracepoint:syscalls:sys_exit_read /pid == PID/ { @usecs = hist((nsecs - @start[tid]) / 1000); delete(@start[tid]); }'
```

#### When to Use Each Tool

| Scenario | Best Tool | Why |
|----------|-----------|-----|
| Debug failing process | **strace** | Full args, return values, error decode |
| Find what files a process opens | **strace -e trace=%file** | Path decode built-in |
| Production latency investigation | **perf trace** or **bpftrace** | Low overhead |
| Syscall latency histograms | **bpftrace** | In-kernel aggregation |
| Library call tracing | **ltrace** | Traces shared library calls, not syscalls |
| Dynamic linker debugging | **LD_DEBUG=libs** | Zero overhead for loader issues |
| Chaos testing / fault injection | **strace -e inject=** | Only tool with built-in injection |
| Multi-threaded deep debugging | **gdb catch syscall** | Full backtrace + variable inspection |

---

### Container & Permission Debugging

#### ptrace_scope (Yama LSM)

```bash
cat /proc/sys/kernel/yama/ptrace_scope
```

| Value | Meaning | strace Impact |
|-------|---------|---------------|
| 0 | Classic: any process can trace any other owned by same user | Normal usage works |
| 1 | Restricted: only direct parent can trace children | `strace -p PID` fails for non-children |
| 2 | Admin-only: only root/CAP_SYS_PTRACE can trace | Requires sudo |
| 3 | Disabled: no process may ptrace any other | strace completely broken |

```bash
# Temporarily allow (requires root)
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope

# Permanent (survives reboot)
echo "kernel.yama.ptrace_scope = 0" | sudo tee /etc/sysctl.d/10-ptrace.conf
sudo sysctl -p /etc/sysctl.d/10-ptrace.conf
```

#### Capabilities

```bash
# Grant ptrace capability to a specific binary
sudo setcap cap_sys_ptrace+ep /usr/bin/strace

# Check capabilities
getcap /usr/bin/strace

# Run with ptrace capability (without full root)
sudo capsh --caps="cap_sys_ptrace+eip" -- -c "strace -p PID"
```

#### Docker

```bash
# Run container with strace capability
docker run --cap-add=SYS_PTRACE myimage

# strace inside container (if strace is installed in image)
docker exec -it container strace -p 1

# strace from host into container process
PID=$(docker top mycontainer -o pid | tail -1)
strace -p $PID

# docker-compose: add capability
# services:
#   myapp:
#     cap_add:
#       - SYS_PTRACE

# Run with seccomp disabled (allows all syscalls including ptrace)
docker run --security-opt seccomp=unconfined myimage
```

#### nsenter (Trace from Host Namespace)

```bash
# Enter container's PID and network namespace, then trace PID 1
nsenter -t $(docker inspect -f '{{.State.Pid}}' container) -p -n strace -p 1

# Trace a specific process inside the container from host
CONTAINER_PID=$(docker inspect -f '{{.State.Pid}}' mycontainer)
nsenter -t $CONTAINER_PID -p strace -f -p 1
```

#### Kubernetes

```yaml
# Pod spec with SYS_PTRACE capability
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: myapp
    securityContext:
      capabilities:
        add: ["SYS_PTRACE"]
```

```bash
# Ephemeral debug container (K8s 1.23+)
kubectl debug -it pod/myapp --image=alpine --target=myapp -- sh
# Inside: apk add strace && strace -p 1

# kubectl exec with strace (if installed in image)
kubectl exec -it pod/myapp -- strace -p 1

# Copy strace binary into running container
kubectl cp /usr/bin/strace pod/myapp:/tmp/strace
kubectl exec -it pod/myapp -- /tmp/strace -p 1
```

#### "Operation not permitted" Troubleshooting

```bash
# Diagnosis checklist:
# 1. Check ptrace_scope
cat /proc/sys/kernel/yama/ptrace_scope

# 2. Check if process is owned by you
ps -o pid,uid,user -p PID

# 3. Check if running in container without SYS_PTRACE
grep -i cap /proc/1/status

# 4. Check if seccomp blocks ptrace
grep Seccomp /proc/self/status

# 5. Check SELinux/AppArmor
getenforce 2>/dev/null                # SELinux
aa-status 2>/dev/null                 # AppArmor

# Solutions (in order of preference):
sudo strace -p PID                    # Use root
# Or: adjust ptrace_scope
# Or: add SYS_PTRACE capability
# Or: disable seccomp profile
# Or: adjust SELinux/AppArmor policy
```

---

### Debugging Workflows

#### Process is Slow

```bash
# Step 1: Identify which syscall category dominates
strace -c ./myapp
# Look at: % time column -- is it file? network? memory? futex (locks)?

# Step 2: Drill into the dominant category
strace -e trace=%file -T ./myapp -o slow.log     # If file-heavy
strace -e trace=%network -T ./myapp -o slow.log   # If network-heavy
strace -e trace=futex -T ./myapp -o slow.log      # If lock-heavy

# Step 3: Find the slowest individual calls
awk -F'[<>]' 'NF>1 {print $(NF-1), $0}' slow.log | sort -rn | head -20

# Step 4: Understand what's slow
# - read() slow? -> disk I/O or remote filesystem
# - connect() slow? -> DNS or routing
# - futex() slow? -> contention, wrong thread pool size
# - poll()/epoll_wait() slow? -> waiting for events (may be normal)
```

#### Process Hangs

```bash
# Step 1: Attach to hanging process
strace -p PID
# The FIRST line of output is the syscall it's stuck in

# Step 2: If stuck in futex() -- lock contention or deadlock
cat /proc/PID/stack                    # Kernel stack trace
cat /proc/PID/wchan                    # Wait channel name

# Step 3: If stuck in read()/recvfrom() -- waiting for data
lsof -p PID                            # What FDs are open?
ss -tnp | grep PID                     # What's the socket state?

# Step 4: If stuck in wait4()/waitpid() -- waiting for child
ps --ppid PID                          # What children exist?
strace -p CHILD_PID                    # What's the child doing?

# Step 5: If stuck in poll()/select()/epoll_wait() -- event loop waiting
# This may be NORMAL (idle event loop) -- check if it resumes on input
```

#### Permission Denied Errors

```bash
# Step 1: Find what file is failing
strace -e trace=%file -Z ./myapp 2>&1 | grep EACCES

# Step 2: Check the file
stat /path/to/file                     # Owner, group, permissions
getfacl /path/to/file                  # ACLs if applicable

# Step 3: Check the process identity
strace -e trace=getuid,geteuid,getgid,getegid ./myapp

# Step 4: Check capabilities
grep Cap /proc/PID/status | while read line; do
  echo "$line -> $(capsh --decode=$(echo $line | awk '{print $2}'))"
done
```

#### DNS Resolution Failures

```bash
# Step 1: Watch DNS-related activity
strace -f -e trace=%network,%file -yy ./myapp 2>&1 | grep -E '(resolv|connect|53\b)'

# Step 2: Check resolv.conf reads
strace -e trace=openat,read -P /etc/resolv.conf -P /etc/nsswitch.conf -P /etc/hosts ./myapp

# Step 3: Watch for DNS connect attempts
strace -f -e trace=connect -yy ./myapp 2>&1 | grep ':53'
# If connect() to DNS server times out, the issue is network-level

# Step 4: Timeout diagnosis
strace -f -T -e trace=connect,poll,recvfrom -yy ./myapp 2>&1 | grep -A1 ':53'
# poll() timeout after connect() to :53 = DNS server not responding
```

#### Config File Search (What Files Does a Process Read?)

```bash
# Trace all file open attempts
strace -e trace=openat -y ./myapp 2>&1 | grep -v '= -1'

# Include failed opens (config search paths)
strace -e trace=openat ./myapp 2>&1 | grep -E '\.(conf|cfg|ini|yaml|yml|json|toml|xml|env)'

# Full config search: opens + stats (checking existence)
strace -e trace=openat,stat,statx,access -y ./myapp 2>&1 | grep -E '(etc|conf|config|\.d/)'
```

#### Library Loading Issues

```bash
# Best approach: LD_DEBUG (lower overhead, more detail)
LD_DEBUG=libs ./myapp 2>&1 | head -50

# With strace: find .so loads
strace -e trace=openat ./myapp 2>&1 | grep '\.so'

# Find dlopen calls (runtime loading)
strace -e trace=openat ./myapp 2>&1 | grep -E '\.so(\.[0-9]+)*"'

# Library search path exploration
strace -e trace=openat -Z ./myapp 2>&1 | grep '\.so'
# Failed opens (-Z) show every path the loader tries before finding the lib
```

#### Language-Specific Patterns

**Java:**
```bash
# Thread activity (Java uses futex heavily for synchronization)
strace -f -e trace=futex -c java -jar app.jar
# High futex count = lock contention; investigate thread pool sizing

# GC activity (mmap/munmap for heap management)
strace -f -e trace=mmap,munmap,madvise -c java -jar app.jar

# Class loading (openat for .class and .jar files)
strace -f -e trace=openat java -jar app.jar 2>&1 | grep -E '\.(jar|class)'
```

**Python:**
```bash
# Module imports (Python opens .py files during import)
strace -e trace=openat python3 app.py 2>&1 | grep '\.py'

# GIL contention (futex = Global Interpreter Lock)
strace -f -e trace=futex -c python3 app.py
# Very high futex count in multi-threaded Python = GIL bottleneck

# C extension loading
strace -e trace=openat python3 app.py 2>&1 | grep '\.so'
```

**Node.js:**
```bash
# Event loop (libuv uses epoll on Linux)
strace -f -e trace=epoll_wait,epoll_ctl -T node app.js
# Long epoll_wait durations = idle event loop (normal)
# Short, frequent epoll_wait = high event throughput

# Module loading
strace -e trace=openat node app.js 2>&1 | grep -E '\.(js|json|node)'

# DNS resolution (Node uses getaddrinfo in thread pool)
strace -f -e trace=%network -yy node app.js 2>&1 | grep ':53'
```

---

### Version Matrix

| Feature | Min Version | Flag/Option | Notes |
|---------|-------------|-------------|-------|
| FD path decode | 4.7 | `-y` | Resolves FD numbers to paths |
| Stack traces | 4.9 | `-k` | Requires libunwind |
| Socket address decode | 4.15 | `-yy` | IP:port for sockets, device info |
| Fault injection | 4.16 | `-e inject=` | Error, signal, delay injection |
| Wall clock summary | 5.0 | `-w` | With `-c`, includes wait time |
| Status filtering | 5.2 | `-e status=`, `-z`, `-Z` | Filter by success/failure |
| seccomp-BPF acceleration | 5.3 | `--seccomp-bpf` | ~2x faster filtered traces |
| FD set tracing | 5.5 | `--trace-fds=SET` | Trace only specific FDs |
| Custom summary columns | 5.7 | `-U` | Choose columns for `-c` output |
| Syscall limit | 5.8 | `--syscall-limit=N` | Stop after N syscalls |
| Timestamp precision | 5.11 | `--absolute-timestamps=time:ns` | Nanosecond timestamps |
| Argument names | 6.16 | `-N` / `--arg-names` | Named arguments in output |

Check your version:

```bash
strace --version
# strace -- version 6.x
```

Feature detection:

```bash
# Check if fault injection is supported
strace -e inject=exit:when=0 /bin/true 2>&1 && echo "inject supported"

# Check if seccomp-bpf is supported
strace --seccomp-bpf -e trace=exit_group /bin/true 2>&1 && echo "seccomp-bpf supported"
```

---

### strace Installation & Performance Notes

#### Installation

```bash
# Debian/Ubuntu
sudo apt install strace

# RHEL/CentOS/Fedora
sudo dnf install strace

# Alpine
apk add strace

# From source (for latest features)
git clone https://github.com/strace/strace.git
cd strace && ./bootstrap && ./configure --with-libunwind && make && sudo make install
```

#### Performance Impact

| Scenario | Overhead | Explanation |
|----------|----------|-------------|
| CPU-bound (few syscalls) | ~2-5% | ptrace cost is negligible |
| Mixed workload | ~2x slowdown | Typical for most processes |
| I/O-heavy (many small reads) | 10-100x | Every read/write = 2 extra context switches |
| With `--seccomp-bpf` + filter | ~1.5x | Unmatched syscalls skip ptrace |
| With `-c` only (no output) | ~1.5x | No formatting/output overhead |
| With `-k` (stack traces) | ~3-5x | libunwind unwinding per syscall |

**Why ptrace is expensive:** Every traced syscall causes two context switches to the tracer (one on entry, one on exit). For a process making millions of small syscalls, this doubles (or worse) the total context switch count.

#### Production Guidelines

- **Never** run unfiltered strace on production processes without understanding the overhead
- **Always** filter (`-e trace=`) and use `--seccomp-bpf` when available
- **Prefer** bpftrace or perf trace for production monitoring
- **Use** strace for debugging specific issues on staging/development
- **Consider** `strace -c` (summary only) for quick production triage -- lower overhead than full trace
- **Set** `--syscall-limit` to cap trace duration for safety
- **Use** `-o` to write to a file, not to the terminal -- terminal output can be a bottleneck itself

---

## Performance Profiling (perf)

### Recording & Reporting

```bash
# Record performance profile
perf record ./command                  # Basic recording
perf record -g ./command               # With call graph (stack traces)
perf record -g --call-graph dwarf ./command  # DWARF unwinding (more accurate)
perf record -g -p PID                  # Attach to running process
perf record -g -a sleep 10            # System-wide for 10 seconds

# Analyze results
perf report                            # Interactive TUI browser
perf report --stdio                    # Text output
perf report --sort=dso,symbol          # Sort by library and function
perf report -n --stdio | head -40      # Top functions with sample counts
```

### Quick Statistics

```bash
# Hardware counter statistics
perf stat ./command                    # CPU cycles, instructions, cache misses
perf stat -d ./command                 # Detailed: L1, LLC, TLB stats
perf stat -dd ./command                # Very detailed: all available counters
perf stat -r 5 ./command               # Average over 5 runs

# Specific events
perf stat -e cycles,instructions,cache-misses ./command
perf stat -e 'syscalls:sys_enter_*' ./command  # Syscall counts
```

### Live Monitoring

```bash
# Real-time function profiling
perf top                               # System-wide top functions
perf top -p PID                        # Specific process
perf top -g                            # With call stacks
```

### perf trace (Low-Overhead strace Alternative)

```bash
perf trace ./command                   # Similar to strace output
perf trace -s ./command                # Summary mode (like strace -c)
perf trace -e openat,read,write ./command  # Filter by syscall
perf trace -p PID                      # Attach to running process
perf trace --duration 100 ./command    # Only syscalls >100ms
```

### Flamegraph Generation

```bash
# Record with call graph
perf record -g --call-graph dwarf ./command

# Generate flamegraph (requires flamegraph tools)
perf script | stackcollapse-perf.pl | flamegraph.pl > flame.svg

# Alternative: folded output for other tools
perf script | stackcollapse-perf.pl > out.folded

# Differential flamegraph (before vs after)
perf record -g -o before.data ./command_v1
perf record -g -o after.data ./command_v2
perf script -i before.data | stackcollapse-perf.pl > before.folded
perf script -i after.data | stackcollapse-perf.pl > after.folded
difffolded.pl before.folded after.folded | flamegraph.pl > diff.svg
```

### Common Profiling Workflows

```bash
# CPU hotspots
perf record -g ./command && perf report

# Cache misses (memory access patterns)
perf stat -e cache-references,cache-misses,L1-dcache-load-misses ./command

# Context switches (scheduler overhead)
perf stat -e context-switches,cpu-migrations ./command
perf record -e context-switches -g ./command

# Page faults (memory allocation)
perf stat -e page-faults,minor-faults,major-faults ./command

# Branch prediction misses
perf stat -e branch-misses,branch-instructions ./command

# Lock contention
perf lock record ./command && perf lock report

# Scheduler latency
perf sched record ./command && perf sched latency
```

---

## Memory Leak Detection and Analysis

### Valgrind Memcheck

```bash
# Basic leak detection
valgrind ./program                     # Default memcheck tool
valgrind --leak-check=full ./program   # Detailed leak report
valgrind --leak-check=full --show-leak-kinds=all ./program  # All leak types
valgrind --leak-check=full --track-origins=yes ./program     # Track uninitialized values
valgrind --leak-check=full --show-reachable=yes ./program    # Show still-reachable blocks

# Output control
valgrind --log-file=valgrind.log ./program     # Log to file
valgrind --xml=yes --xml-file=val.xml ./program  # XML output for tools

# Suppressions (ignore known false positives)
valgrind --gen-suppressions=all ./program       # Generate suppression entries
valgrind --suppressions=my.supp ./program       # Use suppression file
```

### Valgrind Cachegrind (Cache Profiling)

```bash
# Profile cache usage
valgrind --tool=cachegrind ./program
cg_annotate cachegrind.out.PID                  # Annotate source with cache stats
cg_annotate --auto=yes cachegrind.out.PID       # Auto-annotate all source files

# Compare two runs
cg_diff cachegrind.out.1 cachegrind.out.2
cg_annotate cachegrind.out.diff
```

### Valgrind Callgrind (Call Graph Profiling)

```bash
# Record call graph with costs
valgrind --tool=callgrind ./program
callgrind_annotate callgrind.out.PID            # Text annotation

# Interactive visualization
kcachegrind callgrind.out.PID                   # GUI (install: sudo apt install kcachegrind)

# Control during execution
callgrind_control -i on                         # Enable instrumentation
callgrind_control -i off                        # Disable instrumentation
callgrind_control -d                            # Force dump of current data
```

### Valgrind Massif (Heap Profiler)

```bash
# Profile heap usage over time
valgrind --tool=massif ./program
ms_print massif.out.PID                         # Text visualization of heap timeline

# Include stack memory
valgrind --tool=massif --stacks=yes ./program

# Custom snapshot threshold
valgrind --tool=massif --threshold=0.5 ./program  # Report allocations >0.5%
```

### Heaptrack (Modern Heap Profiler)

```bash
# Record allocations
heaptrack ./program                    # Trace from start
heaptrack -p PID                       # Attach to running process

# Analyze results
heaptrack_print heaptrack.*.zst        # Text summary
heaptrack_gui heaptrack.*.zst          # GUI visualization (recommended)

# Key metrics reported:
# - Peak heap usage
# - Total allocations
# - Allocation hotspots (by call stack)
# - Memory leaks (allocated but never freed)
# - Temporary allocations (allocated then immediately freed)
```

### AddressSanitizer (ASan) -- Compile-Time Alternative

```bash
# Compile with ASan (much faster than Valgrind, ~2x overhead vs ~20x)
gcc -fsanitize=address -g -o program program.c
clang -fsanitize=address -g -o program program.c

# Run normally -- ASan reports errors to stderr
./program

# Additional sanitizers
gcc -fsanitize=address,undefined -g -o program program.c   # + undefined behavior
gcc -fsanitize=leak -g -o program program.c                # Leak detection only (lighter)
gcc -fsanitize=memory -g -o program program.c              # Uninitialized reads (clang only)
gcc -fsanitize=thread -g -o program program.c              # Data race detection
```

---

## GDB Debugging

### Starting and Running

```bash
# Start debugging
gdb ./program                          # Load program
gdb -q ./program                       # Quiet mode (no banner)
gdb --args ./program arg1 arg2         # Pass arguments
gdb -p PID                             # Attach to running process

# Inside gdb
(gdb) run                              # Start execution
(gdb) run arg1 arg2                    # Start with arguments
(gdb) start                            # Run to main()
(gdb) continue                         # Resume after breakpoint
(gdb) kill                             # Stop execution
```

### Breakpoints

```bash
(gdb) break main                       # Break at function
(gdb) break file.c:42                  # Break at file:line
(gdb) break *0x400520                  # Break at address
(gdb) tbreak main                      # Temporary breakpoint (hit once)

# Conditional breakpoints
(gdb) break file.c:42 if x > 10       # Break when condition is true
(gdb) condition 1 x > 10              # Add condition to breakpoint #1

# Manage breakpoints
(gdb) info breakpoints                 # List all breakpoints
(gdb) delete 1                         # Delete breakpoint #1
(gdb) disable 1                        # Temporarily disable
(gdb) enable 1                         # Re-enable
```

### Stepping and Navigation

```bash
(gdb) next                             # Step over (execute line, skip function calls)
(gdb) step                             # Step into (enter function calls)
(gdb) finish                           # Run until current function returns
(gdb) until 50                         # Run until line 50
(gdb) advance file.c:50               # Run to specific location
```

### Inspecting State

```bash
(gdb) print variable                   # Print variable value
(gdb) print *ptr                       # Dereference pointer
(gdb) print array[0]@10               # Print 10 elements of array
(gdb) print/x variable                 # Print in hex
(gdb) print/t variable                 # Print in binary

(gdb) info locals                      # All local variables
(gdb) info args                        # Function arguments
(gdb) info registers                   # CPU registers

(gdb) backtrace                        # Full call stack
(gdb) backtrace full                   # Call stack with local variables
(gdb) frame 3                          # Switch to stack frame #3
(gdb) up                               # Move up one frame
(gdb) down                             # Move down one frame
```

### Watchpoints

```bash
(gdb) watch variable                   # Break when variable changes (write)
(gdb) rwatch variable                  # Break when variable is read
(gdb) awatch variable                  # Break on read or write
(gdb) watch *(int*)0x600a00           # Watch memory address
```

### Catch Syscalls

```bash
(gdb) catch syscall                    # Break on any syscall
(gdb) catch syscall write              # Break on write()
(gdb) catch syscall openat read write  # Break on multiple syscalls
```

### Post-Mortem Debugging (Core Dumps)

```bash
# Enable core dumps
ulimit -c unlimited                    # Allow core files
echo "/tmp/core.%e.%p" | sudo tee /proc/sys/kernel/core_pattern  # Set path

# After crash, analyze core
gdb ./program /tmp/core.program.1234
(gdb) backtrace                        # See where it crashed
(gdb) info threads                     # All threads at crash time
(gdb) thread 2                         # Switch to thread 2
(gdb) backtrace                        # That thread's stack
```

### Reverse Debugging

```bash
# Record execution for replay
(gdb) target record-full              # Start recording
(gdb) run
# ... hit a bug ...
(gdb) reverse-next                     # Step backwards
(gdb) reverse-step                     # Step back into function
(gdb) reverse-continue                 # Continue backwards to previous breakpoint
(gdb) set exec-direction reverse       # All commands run backwards
```

### GDB with Multi-Threaded Programs

```bash
(gdb) info threads                     # List all threads
(gdb) thread 3                         # Switch to thread 3
(gdb) thread apply all backtrace       # Backtrace all threads
(gdb) set scheduler-locking on         # Only step current thread
(gdb) set scheduler-locking off        # Normal: all threads run
```

---

## Python Profiling

### py-spy (Sampling Profiler, No Code Changes)

```bash
# Generate flamegraph SVG
py-spy record -o flame.svg -- python app.py
py-spy record -o flame.svg --pid PID   # Attach to running process

# Live top-like view
py-spy top -- python app.py
py-spy top --pid PID                   # Attach to running process

# Dump current stack traces
py-spy dump --pid PID                  # One-shot stack dump

# Options
py-spy record -o flame.svg --rate 100 -- python app.py  # 100 samples/sec
py-spy record -o flame.svg --subprocesses -- python app.py  # Include subprocesses
py-spy record -o profile.speedscope --format speedscope -- python app.py  # Speedscope format
py-spy record --native -- python app.py  # Include C extension frames
```

### Scalene (CPU + Memory + GPU)

```bash
# Profile with line-level attribution
scalene script.py                      # Basic profiling
scalene --html --outfile report.html script.py  # HTML report
scalene --cpu --memory script.py       # CPU and memory
scalene --reduced-profile script.py    # Only show hot lines
scalene --profile-all script.py        # Include library code

# Profile specific function
scalene --- -m pytest tests/           # Profile test suite
```

### Memray (Memory Profiler)

```bash
# Record memory allocations
memray run script.py                   # Record to output file
memray run -o mem.bin script.py        # Specify output file
memray run --native script.py          # Include C/C++ allocations

# Analyze results
memray flamegraph mem.bin              # Memory flamegraph
memray table mem.bin                   # Tabular view
memray stats mem.bin                   # Summary statistics
memray tree mem.bin                    # Tree view of allocations

# Live monitoring
memray run --live script.py            # Real-time TUI
memray run --live-remote script.py     # Remote monitoring
```

### cProfile / profile (Built-in)

```bash
# Command-line profiling
python -m cProfile script.py           # Basic profile
python -m cProfile -o profile.stats script.py  # Save for analysis
python -m cProfile -s cumulative script.py     # Sort by cumulative time
python -m cProfile -s tottime script.py        # Sort by total time

# Analyze saved profile
python -c "import pstats; p = pstats.Stats('profile.stats'); p.sort_stats('cumulative'); p.print_stats(20)"

# Visualize with snakeviz
pip install snakeviz
snakeviz profile.stats                 # Opens browser visualization
```

### line_profiler (Line-by-Line)

```bash
# Decorate functions with @profile, then:
kernprof -l -v script.py               # Line-by-line timing

# Or programmatic usage:
pip install line_profiler
# Add @profile decorator to functions, run with kernprof
```

---

## Node.js Profiling

### Built-in Profiler

```bash
# V8 profiler
node --prof app.js                     # Generate isolate-*.log
node --prof-process isolate-*.log > processed.txt  # Human-readable

# CPU profile (Chrome DevTools format)
node --cpu-prof app.js                 # Generate *.cpuprofile
# Open in Chrome DevTools > Performance tab
```

### clinic (Diagnostics Suite)

```bash
# Auto-detect performance issues
npx clinic doctor -- node app.js       # General diagnosis
npx clinic flame -- node app.js        # CPU flamegraph
npx clinic bubbleprof -- node app.js   # Async operation analysis
npx clinic heapprofiler -- node app.js # Memory profiling

# Each generates an HTML report that opens in browser
```

### 0x (Flamegraph Generator)

```bash
# Generate flamegraph
npx 0x app.js                         # Profile and generate SVG
npx 0x -o flamegraph.html app.js      # Specify output
npx 0x -p PID                         # Attach to running process
```

### Node.js Inspector

```bash
# Start with inspector
node --inspect app.js                  # Inspector on port 9229
node --inspect-brk app.js             # Break on first line

# Connect with Chrome DevTools: chrome://inspect
# Use Performance tab for CPU profiling
# Use Memory tab for heap snapshots
```

---

## Benchmarking

### hyperfine (Statistical Benchmarking)

```bash
# Basic benchmark
hyperfine 'command'                    # Run with automatic iterations

# Compare commands
hyperfine 'command1' 'command2'        # Side-by-side comparison
hyperfine 'sort file.txt' 'sort --parallel=4 file.txt'

# Options
hyperfine --warmup 3 'command'         # Warmup runs before measurement
hyperfine --min-runs 20 'command'      # Minimum number of runs
hyperfine --max-runs 100 'command'     # Cap iterations
hyperfine --prepare 'sync; echo 3 | sudo tee /proc/sys/vm/drop_caches' 'command'  # Prepare between runs

# Parameter scans
hyperfine --parameter-scan threads 1 8 'sort --parallel={threads} file.txt'
hyperfine --parameter-list size 100,1000,10000 'generate_data {size} | process'

# Export results
hyperfine --export-markdown bench.md 'command1' 'command2'
hyperfine --export-json bench.json 'command1' 'command2'
hyperfine --export-csv bench.csv 'command1' 'command2'

# Shell selection
hyperfine --shell=none 'command'       # No shell wrapping (for precise measurement)
hyperfine --shell=bash 'command'       # Explicit shell
```

### time (Basic Timing)

```bash
# Shell builtin (basic)
time command                           # real/user/sys times

# GNU time (detailed)
/usr/bin/time -v command               # Verbose: memory, context switches, page faults
/usr/bin/time -f "%e seconds, %M KB max RSS" command  # Custom format
/usr/bin/time -o timing.txt -v command # Output to file

# Key metrics from /usr/bin/time -v:
# - Elapsed (wall clock) time
# - Maximum resident set size (peak memory)
# - Voluntary/involuntary context switches
# - File system inputs/outputs
# - Page faults (major = disk, minor = in-memory)
```

### Comparing Performance

```bash
# Before/after comparison pattern
hyperfine --export-json before.json 'old_command'
# ... make changes ...
hyperfine --export-json after.json 'new_command'

# Benchmark with setup/cleanup
hyperfine \
  --prepare 'cp original.txt test.txt' \
  --cleanup 'rm test.txt' \
  'process test.txt'

# Statistical significance
hyperfine --min-runs 30 'command1' 'command2'  # More runs = tighter confidence intervals
```

---

## Library Call Tracing and Dynamic Analysis (ltrace)

```bash
# Basic library call trace
ltrace ./program

# Trace specific functions
ltrace -e malloc+free ./program

# Trace all functions
ltrace -e '*' ./program

# Call statistics summary
ltrace -c ./program

# Attach to running process
ltrace -p PID

# Output to file
ltrace -o trace.out ./program

# Filter by library
ltrace -l libcurl.so ./program

# Longer string output
ltrace -s 200 ./program

# Indent nested calls
ltrace -n 2 ./program
```

---

## Dynamic Kernel and Userspace Tracing (bpftrace)

```bash
# List available probes
bpftrace -l 'tracepoint:*' | head -50
bpftrace -l 'tracepoint:syscalls:*'

# Trace file opens by process
bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args.filename)); }'

# TCP connect tracing
bpftrace -e 'kprobe:tcp_connect { printf("%s -> %s\n", comm, ntop(((struct sock*)arg0)->__sk_common.skc_daddr)); }'

# Read/write size histogram
bpftrace -e 'tracepoint:syscalls:sys_exit_read /args.ret > 0/ { @bytes = hist(args.ret); }'

# Syscall latency by process
bpftrace -e 'tracepoint:raw_syscalls:sys_enter { @start[tid] = nsecs; }
tracepoint:raw_syscalls:sys_exit /@start[tid]/ { @ns[comm] = hist(nsecs - @start[tid]); delete(@start[tid]); }'

# Count syscalls by process
bpftrace -e 'tracepoint:raw_syscalls:sys_enter { @[comm] = count(); }'

# File I/O latency histogram
bpftrace -e 'kprobe:vfs_read { @start[tid] = nsecs; }
kretprobe:vfs_read /@start[tid]/ { @us = hist((nsecs - @start[tid]) / 1000); delete(@start[tid]); }'

# CPU scheduler latency
bpftrace -e 'tracepoint:sched:sched_wakeup { @qtime[args.pid] = nsecs; }
tracepoint:sched:sched_switch /@qtime[args.next_pid]/ {
  @usecs = hist((nsecs - @qtime[args.next_pid]) / 1000);
  delete(@qtime[args.next_pid]);
}'
```

---

## LLDB Debugging (macOS/LLVM Alternative to GDB)

```bash
# Start debugging
lldb program
lldb -- program arg1 arg2

# Set breakpoint and run
(lldb) b main
(lldb) r

# Backtrace
(lldb) bt
(lldb) bt all                             # All threads

# Inspect variables
(lldb) frame variable
(lldb) p variable_name
(lldb) p *pointer

# Stepping
(lldb) n                                  # Step over
(lldb) s                                  # Step into
(lldb) c                                  # Continue

# Watchpoints
(lldb) watchpoint set variable x

# Load core dump
(lldb) target create --core corefile program

# Thread management
(lldb) thread list
(lldb) thread select 2
```

---

## Core Dump Management and Post-Mortem Analysis (coredumpctl)

```bash
# List recent core dumps
coredumpctl list

# Show info about last crash
coredumpctl info

# Show info about specific PID
coredumpctl info PID

# Debug last crash with GDB
coredumpctl gdb

# Debug specific crash
coredumpctl gdb PID

# Export core dump file
coredumpctl dump PID -o /tmp/core.dump

# Configure core pattern
cat /proc/sys/kernel/core_pattern
# Default on systemd: |/lib/systemd/systemd-coredump ...
```

---

## BCC Tools (eBPF)

```bash
# Trace new process execution
execsnoop

# Trace file opens
opensnoop
opensnoop -p PID                         # Specific process

# Disk I/O latency histogram
biolatency

# TCP connection lifecycle
tcplife

# Cache hit/miss ratio
cachestat 1                              # 1-second interval

# Count function calls
funccount 'vfs_*'                        # VFS function calls
funccount -p PID 'malloc'               # malloc calls for process

# Trace DNS lookups
gethostlatency

# Trace TCP retransmits
tcpretrans
```

---

## Combined Workflows

```bash
# Process triage pipeline
PID=1234
echo "=== FDs ===" && ls /proc/$PID/fd | wc -l && \
echo "=== Stack ===" && cat /proc/$PID/stack && \
echo "=== Memory ===" && grep -E "VmRSS|VmSize" /proc/$PID/status

# strace syscall frequency analysis
strace -c -f ./app 2>&1 | tail -n +3 | head -n -2 | sort -k4 -rn

# GDB batch core analysis
gdb -batch -ex "thread apply all bt" -ex "info sharedlibrary" ./program core.dump > crash-report.txt 2>&1

# valgrind leak summary extraction
valgrind --leak-check=full ./program 2>&1 | awk '/definitely lost|possibly lost|still reachable/ {print}'

# hyperfine results to JSON analysis
hyperfine --export-json bench.json 'cmd1' 'cmd2'
jq '.results[] | {command, mean, stddev}' bench.json

# Combined profiling: time + strace summary
/usr/bin/time -v strace -c ./program 2>&1 | tail -30
```

---

## Installation

```bash
# Core debugging tools
sudo apt install strace gdb valgrind linux-tools-generic

# Enhanced tools
sudo apt install heaptrack ltrace

# Benchmarking
cargo install hyperfine

# Python profilers
pip install py-spy scalene memray line_profiler snakeviz

# Node.js profilers
npm install -g clinic 0x

# Flamegraph tools
git clone https://github.com/brendangregg/FlameGraph.git
# Add to PATH: export PATH="$PATH:/path/to/FlameGraph"

# bpftrace (requires kernel 4.9+)
sudo apt install bpftrace
```
