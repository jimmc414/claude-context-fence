# Definitive strace Reference

**Tools**: strace, ltrace, perf trace, bpftrace

---

## Quick Reference

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

## Basic Usage

### Trace a Command

```bash
strace ./myapp                        # Trace from start to exit
strace -o trace.log ./myapp           # Output to file (stderr stays clean)
strace -A -o trace.log ./myapp        # Append to existing trace file
```

### Attach to a Running Process

```bash
strace -p 1234                        # Attach to PID 1234
strace -p 1234 -p 5678               # Attach to multiple PIDs
strace -p $(pgrep -f myapp)          # Attach by name
```

### Follow Forks and Children

```bash
strace -f ./myapp                     # Follow all child processes
strace -ff -o trace ./myapp           # Separate file per PID (trace.PID)
strace --output-separately -o trace ./myapp  # Same as -ff -o
```

### Output Control

```bash
strace -s 512 ./myapp                 # Show 512 bytes of strings (default: 32)
strace -s 0 ./myapp                   # Suppress string output entirely
strace -x ./myapp                     # Hex output for non-printable chars
strace -xx ./myapp                    # All string output in hex
strace -X verbose ./myapp             # Named constants (default)
strace -X raw ./myapp                 # Raw numeric values for all arguments
```

### Verbose and Decode Modes

```bash
strace -v ./myapp                     # Verbose: don't abbreviate structs
strace -y ./myapp                     # Decode FD numbers to paths
strace -yy ./myapp                    # Full FD decode: socket addresses, device info
strace --trace-fds=3,4,5 ./myapp      # Trace only FDs 3, 4, 5 (v5.5+)
strace --trace-fds=!0,1,2 ./myapp    # Trace all FDs except stdin/stdout/stderr
strace -n ./myapp                     # Print syscall numbers
strace -N ./myapp                     # Argument names in output (v6.16+, --arg-names)
```

### Daemonize strace

```bash
strace -D ./myapp                     # Daemonize: strace runs as detached grandchild
strace -DD ./myapp                    # Suppress strace exit status
strace -DDD ./myapp                   # Suppress strace and tracee exit status
```

Daemonize is useful for tracing services where the parent process must exit cleanly.

---

## Filtering Syscalls

### By Name

```bash
strace -e trace=open,read,write ./myapp         # Specific syscalls
strace -e trace=openat,read,write,close ./myapp  # Modern Linux uses openat
strace -e trace=!mmap,brk,mprotect ./myapp       # Exclude noisy syscalls
```

### By Category

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

### By Path

```bash
strace -P /etc/config.yaml ./myapp               # Only syscalls touching this path
strace -P /etc -P /var/log ./myapp               # Multiple path filters
```

### By Return Status

```bash
strace -z ./myapp                     # Successful syscalls only
strace -Z ./myapp                     # Failed syscalls only
strace -e status=failed ./myapp       # Same as -Z
strace -e status=successful ./myapp   # Same as -z
strace -e status=unfinished ./myapp   # Interrupted/unfinished calls
strace -e status=unavailable ./myapp  # Calls with no return (exec, exit)
strace -e status=detached ./myapp     # Calls interrupted by detach
```

### By Architecture (Multi-ABI)

```bash
strace -e trace=/open@64 ./myapp      # 64-bit open calls only
strace -e trace=/open@32 ./myapp      # 32-bit compat open calls only
```

### Noise Reduction Pipeline

Common strace noise — calls that almost every program makes but rarely matter:

```bash
# Filter common noise from trace output
strace -o /dev/stdout ./myapp 2>&1 | \
  grep -vE '(mmap|mprotect|munmap|brk|clock_gettime|gettimeofday|getpid|getuid|arch_prctl|set_tid_address|set_robust_list|prlimit64|rseq|getrandom)'
```

### Performance-Optimized Filtering

```bash
strace --seccomp-bpf -e trace=%file ./myapp      # seccomp-BPF filter (v5.3+)
# ~2x faster than ptrace-only filtering — kernel skips non-matching syscalls
# without stopping the process

strace --syscall-limit=1000 ./myapp               # Stop after 1000 syscalls (v5.8+)
strace --syscall-limit=100 -e trace=%network ./myapp  # Cap network syscall capture
```

`--seccomp-bpf` works by installing a BPF filter in the kernel that only triggers ptrace stops for matching syscalls. Unmatched syscalls execute at native speed.

---

## Timestamps & Timing

### Timestamp Modes

```bash
strace -t ./myapp                     # HH:MM:SS
strace -tt ./myapp                    # HH:MM:SS.usec (microsecond precision)
strace -ttt ./myapp                   # Epoch seconds.usec (for programmatic parsing)
strace -r ./myapp                     # Relative: time since previous syscall
```

### Fine-Grained Precision (v5.11+)

```bash
strace --absolute-timestamps=time:ns ./myapp      # Nanosecond timestamps
strace --absolute-timestamps=format:unix,time:us ./myapp  # Unix epoch with microseconds
strace --syscall-times=ns ./myapp                  # Nanosecond per-call durations
```

### Per-Syscall Duration

```bash
strace -T ./myapp                     # Duration in angle brackets: <0.000123>
strace -T -e trace=read ./myapp       # Duration of read() calls only
```

### Wall Clock Time (v5.0+)

```bash
strace -w -c ./myapp                  # Summary includes wall-clock time (not just CPU)
# Critical for I/O-bound processes where time is spent waiting, not computing
```

### Statistics

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

### Find Slow Syscalls from Trace Output

```bash
# Extract syscalls taking more than 1 second
awk -F'[<>]' 'NF>1 && $(NF-1)+0 > 1.0' trace.log

# Top 20 slowest syscalls
awk -F'[<>]' 'NF>1 {print $(NF-1), $0}' trace.log | sort -rn | head -20

# Average duration per syscall type
awk -F'[<>]' 'NF>1 {match($1, /^[0-9: .]* ([a-z_]+)\(/, a); if(a[1]) {s[a[1]]+=$(NF-1); c[a[1]]++}} END {for(k in s) printf "%s: avg=%.6fs count=%d total=%.6fs\n", k, s[k]/c[k], c[k], s[k]}' trace.log | sort -t= -k4 -rn
```

---

## Fault Injection

Fault injection (v4.16+) allows you to simulate errors, delays, and signals without modifying application code. This is strace's most underused power feature.

### Full Syntax

```
-e inject=SET[:error=ERRNO][:retval=VAL][:signal=SIG][:delay_enter=US][:delay_exit=US][:when=EXPR][:poke_enter=@ARG=DATAV][:poke_exit=@ARG=DATAV]
```

Where SET is a syscall set (names, categories, or `all`).

### Error Injection

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

### Return Value Injection

```bash
# Force write() to return 0 bytes written (short write)
strace -e inject=write:retval=0 ./myapp

# Force read() to return 0 (simulate EOF)
strace -e inject=read:retval=0 ./myapp
```

### Signal Injection

```bash
# Deliver SIGKILL on write
strace -e inject=write:signal=SIGKILL ./myapp

# Deliver SIGSEGV on mmap
strace -e inject=mmap:signal=SIGSEGV ./myapp
```

### Delay Injection (Latency Simulation)

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

### Conditional Injection (when=)

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

### Chaos Engineering Recipes

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

## Multi-Process Tracing

### Follow Forks

```bash
strace -f ./myapp                     # Follow all fork/clone/vfork children
# Output interleaves PIDs: [pid 1234] read(3, ...) = 512
```

### Separate Output Files

```bash
strace -ff -o trace ./myapp           # Creates trace.PID for each process
strace --output-separately -o trace ./myapp  # Equivalent long form

# Merge sorted by timestamp afterward
strace-log-merge trace > merged.log   # Merge all trace.PID files
strace-log-merge trace.123 trace.456  # Merge specific files
```

### Detach on Exec

```bash
strace -b execve ./wrapper.sh         # Detach when child calls exec
# Useful for tracing only the launcher, not the launched program
```

### Interrupt Handling

```bash
strace -I1 ./myapp    # Default: no signals blocked
strace -I2 ./myapp    # SIGINT/SIGQUIT/SIGTERM ignored while tracee runs
strace -I3 ./myapp    # SIGINT/SIGQUIT/SIGTERM cause strace to detach (tracee continues)
strace -I4 ./myapp    # Like -I3 but also for SIGTERM
```

### Practical Multi-Process Patterns

```bash
# Trace all Apache child workers
strace -f -p $(pgrep -o httpd) -e trace=%network -o apache-net.log

# Trace shell pipeline (each pipe segment is a child)
strace -f -ff -o pipeline bash -c 'cat /var/log/syslog | grep error | wc -l'

# Trace only the child (not the fork parent)
strace -f -b execve -e trace=%file ./launcher.sh
```

---

## Output Analysis

### Output Format

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

### Unfinished and Resumed Calls

When a syscall is interrupted (common with `-f`):

```
[pid 1234] read(3,  <unfinished ...>
[pid 1235] write(4, "data", 4)  = 4
[pid 1234] <... read resumed>"result", 4096) = 6
```

### Hex Dump I/O

```bash
strace -e read=3 ./myapp              # Hex dump all reads from FD 3
strace -e write=1,2 ./myapp           # Hex dump writes to stdout/stderr
strace -e read=all ./myapp            # Hex dump all read data
strace -e write=all ./myapp           # Hex dump all written data
```

### Stack Traces (v4.9+)

```bash
strace -k ./myapp                     # Print userspace stack trace per syscall
strace -k -e trace=openat ./myapp     # Stack traces only for openat
# Requires libunwind; install: sudo apt install libunwind-dev
# Then rebuild strace from source with --with-libunwind
```

### Grep Patterns for Trace Files

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

### awk Extraction Patterns

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

### Noise Reduction Recipes

```bash
# Exclude startup noise (loader, memory mapping)
strace -e trace=!mmap,mprotect,munmap,brk,arch_prctl,set_tid_address,set_robust_list,rseq,prlimit64,getrandom ./myapp

# Trace only after startup (skip first N calls)
strace -e trace=%file ./myapp 2>&1 | tail -n +50

# Exclude clock queries (frequent in event loops)
strace -e trace=!clock_gettime,gettimeofday,clock_nanosleep ./myapp
```

---

## Cross-Tool Integration

### strace + /proc

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

### strace + lsof

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

### strace + ss/netstat

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

### strace + tcpdump

Two complementary perspectives — application layer (strace) vs wire (tcpdump):

```bash
# Terminal 1: application-level view
strace -tt -e trace=%network -p PID -o app.log

# Terminal 2: network-level view
tcpdump -i any -tt -w wire.pcap port PORT

# Correlate by timestamp: -tt gives microsecond precision in both tools
# strace shows: what the app asked for (connect, send, recv args)
# tcpdump shows: what actually went on the wire (retransmits, resets, latency)
```

### strace + gdb

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

**Limitation**: strace and gdb both use ptrace — you cannot attach both to the same process simultaneously. Use one or the other, or use gdb's `catch syscall` for combined debugging.

### strace + perf

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

### strace + bpftrace

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

### When to Use Each Tool

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

## Container & Permission Debugging

### ptrace_scope (Yama LSM)

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

### Capabilities

```bash
# Grant ptrace capability to a specific binary
sudo setcap cap_sys_ptrace+ep /usr/bin/strace

# Check capabilities
getcap /usr/bin/strace

# Run with ptrace capability (without full root)
sudo capsh --caps="cap_sys_ptrace+eip" -- -c "strace -p PID"
```

### Docker

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

### nsenter (Trace from Host Namespace)

```bash
# Enter container's PID and network namespace, then trace PID 1
nsenter -t $(docker inspect -f '{{.State.Pid}}' container) -p -n strace -p 1

# Trace a specific process inside the container from host
CONTAINER_PID=$(docker inspect -f '{{.State.Pid}}' mycontainer)
nsenter -t $CONTAINER_PID -p strace -f -p 1
```

### Kubernetes

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

### "Operation not permitted" Troubleshooting

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

## Debugging Workflows

### Process is Slow

```bash
# Step 1: Identify which syscall category dominates
strace -c ./myapp
# Look at: % time column — is it file? network? memory? futex (locks)?

# Step 2: Drill into the dominant category
strace -e trace=%file -T ./myapp -o slow.log     # If file-heavy
strace -e trace=%network -T ./myapp -o slow.log   # If network-heavy
strace -e trace=futex -T ./myapp -o slow.log      # If lock-heavy

# Step 3: Find the slowest individual calls
awk -F'[<>]' 'NF>1 {print $(NF-1), $0}' slow.log | sort -rn | head -20

# Step 4: Understand what's slow
# - read() slow? → disk I/O or remote filesystem
# - connect() slow? → DNS or routing
# - futex() slow? → contention, wrong thread pool size
# - poll()/epoll_wait() slow? → waiting for events (may be normal)
```

### Process Hangs

```bash
# Step 1: Attach to hanging process
strace -p PID
# The FIRST line of output is the syscall it's stuck in

# Step 2: If stuck in futex() — lock contention or deadlock
cat /proc/PID/stack                    # Kernel stack trace
cat /proc/PID/wchan                    # Wait channel name

# Step 3: If stuck in read()/recvfrom() — waiting for data
lsof -p PID                            # What FDs are open?
ss -tnp | grep PID                     # What's the socket state?

# Step 4: If stuck in wait4()/waitpid() — waiting for child
ps --ppid PID                          # What children exist?
strace -p CHILD_PID                    # What's the child doing?

# Step 5: If stuck in poll()/select()/epoll_wait() — event loop waiting
# This may be NORMAL (idle event loop) — check if it resumes on input
```

### Permission Denied Errors

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

### DNS Resolution Failures

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

### Config File Search (What Files Does a Process Read?)

```bash
# Trace all file open attempts
strace -e trace=openat -y ./myapp 2>&1 | grep -v '= -1'

# Include failed opens (config search paths)
strace -e trace=openat ./myapp 2>&1 | grep -E '\.(conf|cfg|ini|yaml|yml|json|toml|xml|env)'

# Full config search: opens + stats (checking existence)
strace -e trace=openat,stat,statx,access -y ./myapp 2>&1 | grep -E '(etc|conf|config|\.d/)'
```

### Library Loading Issues

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

### Language-Specific Patterns

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

## Version Matrix

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

## Installation & Performance Notes

### Installation

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

### Performance Impact

| Scenario | Overhead | Explanation |
|----------|----------|-------------|
| CPU-bound (few syscalls) | ~2-5% | ptrace cost is negligible |
| Mixed workload | ~2x slowdown | Typical for most processes |
| I/O-heavy (many small reads) | 10-100x | Every read/write = 2 extra context switches |
| With `--seccomp-bpf` + filter | ~1.5x | Unmatched syscalls skip ptrace |
| With `-c` only (no output) | ~1.5x | No formatting/output overhead |
| With `-k` (stack traces) | ~3-5x | libunwind unwinding per syscall |

**Why ptrace is expensive:** Every traced syscall causes two context switches to the tracer (one on entry, one on exit). For a process making millions of small syscalls, this doubles (or worse) the total context switch count.

### Production Guidelines

- **Never** run unfiltered strace on production processes without understanding the overhead
- **Always** filter (`-e trace=`) and use `--seccomp-bpf` when available
- **Prefer** bpftrace or perf trace for production monitoring
- **Use** strace for debugging specific issues on staging/development
- **Consider** `strace -c` (summary only) for quick production triage — lower overhead than full trace
- **Set** `--syscall-limit` to cap trace duration for safety
- **Use** `-o` to write to a file, not to the terminal — terminal output can be a bottleneck itself
