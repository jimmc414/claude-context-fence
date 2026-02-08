---
description: "Process management recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-process with target process and context included"
when-to-use: "Receives prepared arguments from task-process with target process and context included"
context: fork
model: inherit
allowed-tools: Read
---

# Process Management Recipes

**Tools:** ps, top, htop, btop, kill, pkill, killall, pgrep, pidof, pstree, jobs, bg, fg, nohup, disown, timeout, xargs, parallel, nice, renice, ionice, chrt, taskset, prlimit, ulimit, lsof, fuser, pmap

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| All processes | `ps aux` |
| Find by name | `pgrep -a nginx` |
| Find by port | `lsof -i :8080` |
| Kill by PID | `kill 1234` |
| Kill by name | `pkill nginx` |
| Force kill | `kill -9 1234` |
| Background immune | `nohup cmd > out.log 2>&1 &` |
| Limit time | `timeout 30s command` |
| Low priority | `nice -n 19 command` |
| Parallel xargs | `find . -name '*.gz' \| xargs -P8 -I{} gunzip {}` |
| Parallel GNU | `parallel gzip ::: *.log` |
| Process tree | `pstree -p PID` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| List processes | ps aux | Snapshot, scriptable |
| Interactive monitoring | htop / btop | Live process viewer |
| Find process by name | pgrep -a | Returns PID + command |
| Kill by name | pkill | Pattern-based signal delivery |
| Graceful stop | kill (SIGTERM) | Allows cleanup |
| Force stop | kill -9 (SIGKILL) | Last resort, no cleanup |
| Batch processing | xargs | Pipe-friendly, simple |
| Parallel execution | GNU parallel | Job control, progress, retry |
| Survive logout | nohup / disown | Process outlives terminal |
| Limit execution time | timeout | Auto-kill after duration |
| Low CPU priority | nice -n 19 | Won't starve other processes |
| What's using a port | lsof -i :PORT | Maps port to PID |
| What's using a file | fuser / lsof | File-level process lookup |

---

## Listing Processes

### All Processes
```bash
ps aux                                        # All users, full format
ps -ef                                        # Full format, POSIX style
ps -eo pid,ppid,user,%cpu,%mem,vsz,rss,stat,start,time,cmd  # Custom columns
ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu    # Sort by CPU descending
ps -eo pid,user,%cpu,%mem,cmd --sort=-%mem    # Sort by memory descending
ps -eo pid,user,vsz --sort=-vsz | head -20   # Top 20 by virtual memory
ps aux --sort=-%cpu | head -11                # Top 10 CPU hogs + header
ps aux --sort=-%mem | head -11                # Top 10 memory hogs + header
```

### Find by Name/User
```bash
ps -C nginx                                   # Select by command name
ps -u www-data                                # Select by user
ps -u root -o pid,cmd --sort=-%cpu | head -20 # Root processes by CPU
ps aux | grep '[n]ginx'                       # Grep-safe (no self-match)
ps aux | grep '[j]ava.*-Xmx'                 # Grep-safe with pattern
```

### Process Tree
```bash
ps auxf                                       # Tree with --forest
ps -eo pid,ppid,cmd --forest                  # Minimal tree
pstree                                        # ASCII tree
pstree -p                                     # Show PIDs
pstree -a                                     # Show command arguments
pstree -p 1234                                # Tree from PID 1234
pstree -h                                     # Highlight current process
pstree -H 1234                                # Highlight specific PID
pstree -c                                     # Don't compact identical subtrees
pstree -T                                     # Hide threads
pstree -u                                     # Show uid transitions
```

### Thread View
```bash
ps -eLf                                       # All threads
ps -eo pid,nlwp,cmd | sort -k2 -rn | head    # Most threads per process
ps -T -p 1234                                 # Threads of specific PID
```

### pgrep
```bash
pgrep nginx                                   # PIDs by name
pgrep -a nginx                                # PID + full command
pgrep -l nginx                                # PID + name
pgrep -u www-data                             # By user
pgrep -x nginx                                # Exact name match
pgrep -f "python.*server"                     # Match full command line
pgrep -c nginx                                # Count matches
pgrep -n nginx                                # Newest matching PID
pgrep -o nginx                                # Oldest matching PID
pgrep -P 1234                                 # Children of PID 1234
pgrep -t pts/0                                # By terminal
```

### pidof
```bash
pidof nginx                                   # All PIDs of nginx
pidof -s nginx                                # Single (first) PID
pidof -x script.sh                            # Include scripts
```

### top / htop / btop
```bash
top -bn1                                      # Batch mode, one snapshot
top -bn1 -o %CPU | head -20                   # Sort by CPU, top entries
top -bn1 -o %MEM | head -20                   # Sort by MEM
top -u www-data -bn1                          # Filter by user
top -p 1234,5678 -bn1                         # Monitor specific PIDs
htop -t                                       # Tree view
htop -u www-data                              # Filter by user
htop -p 1234                                  # Specific PID
```

---

## Signals

### Signal Reference

| # | Signal | Default | Common Use |
|---|--------|---------|------------|
| 1 | SIGHUP | Terminate | Reload config (daemons) |
| 2 | SIGINT | Terminate | Ctrl-C, interrupt |
| 3 | SIGQUIT | Core dump | Ctrl-\\, quit with dump |
| 9 | SIGKILL | Terminate | Force kill (uncatchable) |
| 10 | SIGUSR1 | Terminate | App-defined (log rotate) |
| 12 | SIGUSR2 | Terminate | App-defined |
| 13 | SIGPIPE | Terminate | Broken pipe |
| 14 | SIGALRM | Terminate | Timer expired |
| 15 | SIGTERM | Terminate | Graceful shutdown (default) |
| 17 | SIGCHLD | Ignore | Child process exited |
| 18 | SIGCONT | Continue | Resume stopped process |
| 19 | SIGSTOP | Stop | Pause (uncatchable) |
| 20 | SIGTSTP | Stop | Ctrl-Z, terminal stop |
| 28 | SIGWINCH | Ignore | Terminal resize |

### kill
```bash
kill 1234                                     # SIGTERM (graceful, default)
kill -15 1234                                 # SIGTERM explicit
kill -9 1234                                  # SIGKILL (force, last resort)
kill -1 1234                                  # SIGHUP (reload config)
kill -STOP 1234                               # Pause process
kill -CONT 1234                               # Resume paused process
kill -0 1234                                  # Check if process exists (no signal)
kill %1                                       # Kill job number 1
kill -l                                       # List all signals
kill -l 9                                     # Name for signal 9
```

### pkill
```bash
pkill nginx                                   # SIGTERM all nginx
pkill -9 nginx                                # SIGKILL all nginx
pkill -HUP nginx                              # Reload config
pkill -STOP nginx                             # Pause all nginx
pkill -u www-data                             # Kill all user's processes
pkill -t pts/2                                # Kill all on terminal
pkill -x nginx                                # Exact name match only
pkill -f "python manage.py runserver"         # Match full command
pkill -g 1234                                 # Kill process group
pkill -P 1234                                 # Kill children of PID
pkill --signal TERM -u baduser                # Explicit signal + user
```

### killall
```bash
killall nginx                                 # Kill all by exact name
killall -s KILL nginx                         # SIGKILL by name
killall -u www-data                           # All processes of user
killall -w nginx                              # Wait for processes to die
killall -I nginx                              # Case insensitive
killall -r "node.*"                           # Regex match
killall -o 1h nginx                           # Only older than 1 hour
killall -y 10m nginx                          # Only younger than 10 minutes
```

### Process Group Kill
```bash
kill -- -1234                                 # Kill entire process group 1234
ps -o pgid= -p 5678                          # Find process group of PID
kill -- -$(ps -o pgid= -p 5678 | tr -d ' ')  # Kill PID's entire group
```

### Signal Trapping in Scripts
```bash
trap 'echo "Caught SIGINT"; cleanup; exit 1' INT
trap 'rm -f /tmp/lockfile; exit' EXIT INT TERM
trap '' HUP                                   # Ignore SIGHUP
trap - INT                                    # Reset to default handler
```

---

## Job Control

### Background & Foreground
```bash
command &                                     # Start in background
Ctrl-Z                                        # Suspend foreground job
bg                                            # Resume last suspended in background
bg %2                                         # Resume job 2 in background
fg                                            # Bring last job to foreground
fg %1                                         # Bring job 1 to foreground
jobs                                          # List all jobs
jobs -l                                       # List with PIDs
jobs -r                                       # Running only
jobs -s                                       # Stopped only
```

### Survive Terminal Close
```bash
nohup command > out.log 2>&1 &                # Classic: immune to hangup
nohup command > /dev/null 2>&1 &              # Discard output
command & disown                              # Start + detach from shell
command & disown %1                           # Detach specific job
disown -h %1                                  # Keep running on shell exit
disown -a                                     # Disown all jobs
setsid command                                # New session leader (fully detached)
```

### Wait for Background Jobs
```bash
command1 & PID1=$!                            # Capture background PID
command2 & PID2=$!
wait $PID1                                    # Wait for specific PID
echo "PID1 exit code: $?"
wait $PID2
wait                                          # Wait for ALL background jobs
```

### Comparison: Background Methods

| Method | Survives Logout | New Session | Logs |
|--------|----------------|-------------|------|
| `cmd &` | No | No | Terminal |
| `nohup cmd &` | Yes | No | nohup.out |
| `cmd & disown` | Yes | No | Lost |
| `setsid cmd` | Yes | Yes | Lost |
| `systemd service` | Yes | Yes | journalctl |

### Daemonize Pattern in Scripts
```bash
# Self-daemonize
if [ "$1" != "--daemon" ]; then
    nohup "$0" --daemon > /var/log/myapp.log 2>&1 &
    echo "Started as PID $!"
    exit 0
fi
# Daemon code continues here
```

---

## Process Priority

### nice (Launch with Priority)
```bash
nice command                                  # Default niceness +10
nice -n 19 command                            # Lowest priority (background)
nice -n -20 command                           # Highest priority (root only)
nice -n 10 make -j8                           # Build with lower priority
nice -n 15 tar czf backup.tar.gz /data        # Low priority backup
```

### renice (Change Running Priority)
```bash
renice -n 19 -p 1234                          # Set PID to low priority
renice -n -5 -p 1234                          # Higher priority (root only)
renice -n 19 -u www-data                      # All processes of user
renice -n 10 -g 1234                          # Entire process group
```

### ionice (I/O Priority)
```bash
ionice -c 3 command                           # Idle: only when no other I/O
ionice -c 2 -n 7 command                      # Best-effort, lowest within class
ionice -c 2 -n 0 command                      # Best-effort, highest within class
ionice -c 1 -n 0 command                      # Realtime highest (root only)
ionice -p 1234                                # Query PID's I/O class
ionice -c 3 -p 1234                           # Set running PID to idle
ionice -c 3 nice -n 19 rsync -a /src /dst     # Low CPU + low I/O
```

### chrt (Realtime Scheduling)
```bash
chrt -f 50 command                            # FIFO policy, priority 50
chrt -r 25 command                            # Round-robin, priority 25
chrt -o 0 command                             # Other (normal), priority 0
chrt -b 0 command                             # Batch (non-interactive)
chrt -i 0 command                             # Idle scheduling
chrt -p 1234                                  # Query PID's policy
chrt -f -p 50 1234                            # Set running PID to FIFO 50
chrt --max                                    # Show min/max for each policy
```

### taskset (CPU Affinity)
```bash
taskset -c 0 command                          # Pin to CPU 0
taskset -c 0-3 command                        # Pin to CPUs 0-3
taskset -c 0,2,4 command                      # Pin to specific CPUs
taskset -p 1234                               # Query PID's affinity mask
taskset -cp 0-3 1234                          # Set running PID to CPUs 0-3
taskset 0x3 command                           # Hex mask: CPUs 0,1
taskset -c 0 nice -n 19 command               # Pin + low priority combo
nproc                                         # How many CPUs available
```

---

## Resource Limits

### ulimit (Per-Shell Limits)
```bash
ulimit -a                                     # Show all current limits
ulimit -n                                     # Max open files (soft)
ulimit -n 65536                               # Set max open files
ulimit -Hn                                    # Hard limit for open files
ulimit -u                                     # Max user processes
ulimit -u 4096                                # Set max processes
ulimit -v 4194304                             # Max virtual memory (KB) = 4GB
ulimit -t 300                                 # Max CPU time (seconds)
ulimit -c unlimited                           # Enable core dumps
ulimit -c 0                                   # Disable core dumps
ulimit -f 1048576                             # Max file size (512B blocks)
ulimit -s 32768                               # Stack size (KB)
```

### /etc/security/limits.conf (Persistent)
```
# /etc/security/limits.conf
www-data  soft  nofile  65536
www-data  hard  nofile  65536
www-data  soft  nproc   4096
www-data  hard  nproc   4096
*         soft  core    0
@devteam  hard  as      8388608
```

### prlimit (Per-Process Limits)
```bash
prlimit --pid 1234                            # Show all limits for PID
prlimit --pid 1234 --nofile                   # Show open files limit
prlimit --pid 1234 --nofile=8192:16384        # Set soft:hard open files
prlimit --pid 1234 --as=4294967296:4294967296 # 4GB virtual memory
prlimit --pid 1234 --cpu=300:300              # 300s CPU time
prlimit --nofile=65536 command                # Launch with limit
prlimit --nproc=100 --as=2147483648 command   # Multiple limits at launch
```

### timeout (Time Limit)
```bash
timeout 30s command                           # Kill after 30 seconds
timeout 5m command                            # Kill after 5 minutes
timeout 2h command                            # Kill after 2 hours
timeout 1d command                            # Kill after 1 day
timeout --signal=KILL 30s command             # SIGKILL instead of SIGTERM
timeout -k 10s 60s command                    # SIGTERM at 60s, SIGKILL at 70s
timeout --preserve-status 30s command         # Exit with command's status
timeout --foreground 30s command              # Allow terminal input
timeout 30s bash -c 'while true; do work; done'  # Timeout a loop
```

### cgroups v2 (Advanced Resource Control)
```bash
# Create cgroup
sudo mkdir /sys/fs/cgroup/myapp

# Limit memory to 1GB
echo 1073741824 | sudo tee /sys/fs/cgroup/myapp/memory.max

# Limit CPU to 50% of one core
echo "50000 100000" | sudo tee /sys/fs/cgroup/myapp/cpu.max

# Limit to 200% CPU (2 cores max)
echo "200000 100000" | sudo tee /sys/fs/cgroup/myapp/cpu.max

# Add process to cgroup
echo 1234 | sudo tee /sys/fs/cgroup/myapp/cgroup.procs

# Launch in cgroup via systemd-run
systemd-run --scope -p MemoryMax=1G command
systemd-run --scope -p MemoryMax=1G -p CPUQuota=200% command
systemd-run --scope -p MemoryMax=512M -p MemorySwapMax=0 command

# Set on existing slice
sudo systemctl set-property myapp.slice MemoryMax=2G

# Check current usage
cat /sys/fs/cgroup/myapp/memory.current
cat /sys/fs/cgroup/myapp/cpu.stat

# Cleanup
sudo rmdir /sys/fs/cgroup/myapp
```

---

## File & Port Lookup

### lsof (List Open Files)
```bash
lsof -i :8080                                # What's using port 8080
lsof -i TCP:8080                              # TCP only on port 8080
lsof -i TCP:8080 -sTCP:LISTEN                # Only listeners on 8080
lsof -i UDP:53                                # UDP port 53
lsof -i @192.168.1.100                        # Connections to host
lsof -i TCP:1-1024                            # All privileged ports
lsof /var/log/syslog                          # What's using this file
lsof +D /var/log/                             # All open files in dir (recursive)
lsof -u www-data                              # All files opened by user
lsof -u ^root                                 # All files NOT opened by root
lsof -c nginx                                 # All files opened by nginx
lsof -c nginx -c apache2                      # By nginx OR apache2
lsof -p 1234                                  # All files opened by PID
lsof -p 1234 -a -i                            # Network files of PID (-a = AND)
lsof +L1                                      # Deleted but still open files
lsof -t -i :8080                              # PIDs only (for scripting)
kill $(lsof -t -i :8080)                      # Kill whatever's on port 8080
lsof -i -P -n                                 # All network, numeric ports/hosts
lsof -nP -iTCP -sTCP:LISTEN                   # All listening TCP ports
```

### fuser
```bash
fuser 8080/tcp                                # PIDs using TCP port 8080
fuser -v 8080/tcp                             # Verbose: PID, user, access, command
fuser -k 8080/tcp                             # Kill process on TCP 8080
fuser -k -KILL 8080/tcp                       # SIGKILL process on port
fuser -m /mnt/usb                             # PIDs using mount point
fuser -v /var/log/syslog                      # Who has file open
fuser -s 8080/tcp; echo $?                    # Silent check (0=in use, 1=free)
fuser -n udp 53                               # UDP port 53
```

### /proc/PID Inspection
```bash
cat /proc/1234/status                         # General info (Name, State, Pid, PPid, Threads, VmRSS)
cat /proc/1234/cmdline | tr '\0' ' '          # Full command line
cat /proc/1234/environ | tr '\0' '\n'         # Environment variables
ls -la /proc/1234/fd/                         # Open file descriptors (symlinks)
ls -la /proc/1234/fd/ | wc -l                 # Count open FDs
cat /proc/1234/maps | head -20                # Memory mappings
cat /proc/1234/limits                         # Resource limits (soft/hard)
cat /proc/1234/io                             # I/O counters (read/write bytes)
cat /proc/1234/stat | awk '{print $14,$15}'   # utime, stime (CPU jiffies)
cat /proc/1234/oom_score                      # OOM killer score (higher = more likely)
cat /proc/1234/oom_score_adj                  # OOM adjustment (-1000 to 1000)
echo -500 > /proc/1234/oom_score_adj          # Protect from OOM killer (root)
cat /proc/1234/stack                          # Kernel call stack (root)
readlink /proc/1234/cwd                       # Working directory
readlink /proc/1234/exe                       # Executable path
```

### pmap (Memory Map)
```bash
pmap 1234                                     # Memory map summary
pmap -x 1234                                  # Extended: RSS, dirty pages
pmap -X 1234                                  # Even more detail
pmap -p 1234                                  # Show full paths
```

---

## Parallel Execution

### xargs Basics
```bash
echo "a b c" | xargs command                  # command a b c
echo "a b c" | xargs -n1 command              # command a; command b; command c
cat list.txt | xargs command                  # Args from file
cat list.txt | xargs -I{} command {}          # Replacement string
echo "" | xargs -r command                    # -r: don't run if empty
```

### xargs Parallel
```bash
find . -name '*.log' | xargs -P8 gzip        # 8 parallel gzip jobs
find . -name '*.log' | xargs -P8 -I{} gzip "{}"  # Safe with spaces
find . -name '*.png' -print0 | xargs -0 -P4 -I{} convert {} -resize 50% {}  # Null-safe
cat urls.txt | xargs -P10 -I{} curl -sO {}   # Download 10 at a time
seq 100 | xargs -P4 -I{} bash -c 'process_item {}'  # 4 parallel workers
ls *.csv | xargs -P$(nproc) -I{} python process.py {}  # Use all cores
```

### xargs Batch Mode
```bash
cat list.txt | xargs -n 50 command            # 50 args per invocation
find . -name '*.txt' | xargs -n 100 -P4 gzip # 100 files per gzip, 4 parallel
cat data.txt | xargs -L 2 command             # 2 lines per invocation
```

### xargs Advanced Patterns
```bash
# Multiple arguments per call
echo "src1 dst1 src2 dst2" | xargs -n2 cp    # cp src1 dst1; cp src2 dst2

# Dry run
find . -name '*.bak' | xargs -I{} echo rm {} # Preview what would run

# With shell features
find . -name '*.log' | xargs -I{} sh -c 'gzip "{}" && echo "Done: {}"'

# Handle failures gracefully
cat urls.txt | xargs -P8 -I{} sh -c 'curl -sf "{}" -o /tmp/$(basename "{}") || echo "FAILED: {}"'

# Xargs from two columns
paste src.txt dst.txt | xargs -n2 -P4 cp     # Parallel copy pairs

# Prompt before each
find . -name '*.tmp' | xargs -p rm            # -p: confirm each
```

### GNU parallel Basics
```bash
parallel echo ::: A B C                       # echo A; echo B; echo C
parallel gzip ::: *.log                       # Parallel gzip all logs
parallel -j8 gzip ::: *.log                   # 8 jobs max
parallel -j+0 gzip ::: *.log                  # One job per CPU core
parallel -j200% gzip ::: *.log                # 2x CPU cores
```

### GNU parallel Input Sources
```bash
parallel command ::: arg1 arg2 arg3           # Inline args
parallel command :::: file.txt                # Args from file
cat file.txt | parallel command               # Args from stdin
parallel command ::: A B ::: 1 2              # Combinations: A1 A2 B1 B2
parallel command :::+ A B :::+ 1 2            # Linked pairs: A1 B2
seq 100 | parallel -j8 command                # Numbered sequence
```

### GNU parallel Progress & Logging
```bash
parallel --eta gzip ::: *.log                 # Show ETA
parallel --bar gzip ::: *.log                 # Progress bar
parallel --joblog job.log gzip ::: *.log      # Log results to file
parallel --resume --joblog job.log gzip ::: *.log  # Resume interrupted
parallel --results output_dir/ command ::: args    # Save stdout/stderr per job
parallel --halt soon,fail=1 command ::: args  # Stop on first failure
parallel --halt now,fail=20% command ::: args # Stop if 20% fail
parallel --timeout 60 command ::: args        # Per-job timeout
parallel --retries 3 command ::: args         # Retry failures
```

### GNU parallel Advanced
```bash
# Remote execution
parallel --sshlogin user@server1,user@server2 command ::: args
parallel -S server1,server2 --transferfile {} command ::: files

# Pipe mode (chunk stdin)
cat huge.csv | parallel --pipe -j4 wc -l      # Count lines in chunks
cat huge.csv | parallel --pipe --block 10M -j4 sort | sort -m  # Sort chunks then merge
cat huge.log | parallel --pipe grep ERROR      # Parallel grep

# Record grouping
cat data.csv | parallel --pipe -N 1000 command # 1000 records per chunk
cat data.csv | parallel --header : --pipe command  # Preserve CSV header

# Multiple arguments
parallel command {1} {2} ::: A B ::: 1 2      # {1} {2} positional
parallel mv {} {.}.bak ::: *.txt              # {.} removes extension
parallel gzip {} ::: *.log && parallel rm {.}.log ::: *.log.gz  # Chain

# Replacement strings
parallel echo {} ::: A.txt B.txt              # Full: A.txt
parallel echo {.} ::: A.txt B.txt             # No extension: A
parallel echo {/} ::: /path/A.txt             # Basename: A.txt
parallel echo {//} ::: /path/A.txt            # Dirname: /path
parallel echo {/.} ::: /path/A.txt            # Basename no ext: A
parallel echo {#} ::: A B C                   # Job number: 1, 2, 3
```

### Background Job Parallelism (Pure Bash)
```bash
# Simple parallel loop
for f in *.log; do
    gzip "$f" &
done
wait
echo "All done"

# With max parallelism
MAX=4
for f in *.log; do
    gzip "$f" &
    [ $(jobs -r | wc -l) -ge $MAX ] && wait -n  # Wait for any one to finish
done
wait

# Collect exit codes
pids=()
for f in *.log; do
    gzip "$f" &
    pids+=($!)
done
for pid in "${pids[@]}"; do
    wait $pid || echo "PID $pid failed"
done

# Process substitution for parallel diff
diff <(sort file1.txt) <(sort file2.txt)
diff <(ssh server1 cat /etc/config) <(ssh server2 cat /etc/config)
```

---

## Zombie & Orphan Processes

### Detect Zombies
```bash
ps aux | awk '$8 ~ /^Z/'                     # All zombie processes
ps -eo pid,ppid,stat,cmd | grep -w Z         # Zombies with parent PID
ps -eo pid,ppid,stat,cmd | awk '$3 ~ /^Z/ {print "Zombie PID:",$1,"Parent:",$2}'
cat /proc/1234/status | grep State            # Check: State: Z (zombie)
```

### Reap Zombies
```bash
# Kill the parent to reap zombie (init/PID 1 adopts and cleans up)
kill $(ps -o ppid= -p ZOMBIE_PID)            # Kill zombie's parent
kill -SIGCHLD $(ps -o ppid= -p ZOMBIE_PID)   # Ask parent to reap

# If parent won't reap, kill parent forcefully
kill -9 $(ps -o ppid= -p ZOMBIE_PID)         # Force kill parent
```

### Prevent Zombies in Scripts
```bash
# Always wait for children
command1 &
command2 &
wait                                          # Reap all children

# Trap SIGCHLD
trap 'wait' CHLD                              # Auto-reap children

# Ignore SIGCHLD (children auto-reaped)
trap '' CHLD
```

### Orphan Processes
```bash
# Orphans are adopted by PID 1 (init/systemd)
# Check if process is orphaned (PPID == 1)
ps -eo pid,ppid,cmd | awk '$2 == 1'          # All orphans (adopted by init)
```

### btop (Modern Resource Monitor)

```bash
btop                                           # Launch interactive monitor
# Inside btop:
# 1-4 — switch between CPU/MEM/NET/DISK views
# f — filter processes
# t — tree view toggle
# s — sort menu
# / — search processes
# k — kill selected process (sends SIGTERM)
# Esc — back/exit menus

# Configuration: ~/.config/btop/btop.conf
```

### Cross-References

```bash
# For strace/ltrace/perf: see task-debug skill (strace, ltrace, bpftrace, perf)
# For tmux/screen sessions: see task-containers skill (tmux, screen, zellij)
# For system services: see task-system skill (systemctl, journalctl)
```

### Advanced Process Analysis

```bash
# Top memory consumers
ps aux --sort=-%mem | head -20                 # Sort by memory usage (descending)

# Watch process list in real-time
watch -n1 'ps -eo pid,%cpu,%mem,cmd --sort=-%cpu | head -15'

# Process resource limits
prlimit --pid $PID                             # Show all limits for PID
prlimit --pid $PID --nofile=65536:65536        # Set open file limit

# Find by pattern and check limits
pgrep -a python | xargs -I{} sh -c 'echo "=== PID {} ===" && prlimit --pid {}'

# Process tree with resource usage
pstree -p -a $PID                              # Show process tree with args
ps -eo pid,ppid,%cpu,%mem,cmd --forest | head -30  # Forest view with resources
```

### Combined Workflows

```bash
# Process triage — FDs, stack, memory for a PID
PID=1234; echo "=== FDs: $(ls /proc/$PID/fd 2>/dev/null | wc -l) ===" && \
  grep -E "VmRSS|VmSize" /proc/$PID/status && \
  cat /proc/$PID/cmdline | tr '\0' ' ' && echo

# Find processes with most open files
for pid in $(ls /proc | grep '^[0-9]'); do
  count=$(ls /proc/$pid/fd 2>/dev/null | wc -l)
  [ "$count" -gt 100 ] && echo "$count $(cat /proc/$pid/cmdline 2>/dev/null | tr '\0' ' ')"
done | sort -rn | head -10

# Kill all zombies' parents
ps -eo ppid,stat | awk '$2~/^Z/{print $1}' | sort -u | xargs -I{} kill {}
```

---

## Moreutils (Agent-Safe Pipe Tools)

### sponge (safe in-place edit)
```bash
sort file.txt | sponge file.txt             # Safe in-place sort (reads all before writing)
grep -v "^#" config | sponge config          # Remove comments in-place
# Without sponge: sort file.txt > file.txt   # DANGER: truncates file!
```

### chronic (quiet on success)
```bash
chronic backup_script.sh                     # Silent if exit 0, shows output on failure
# Perfect for cron: only get email on errors
* * * * * chronic /path/to/script.sh
```

### ts (add timestamps)
```bash
long_command | ts '[%Y-%m-%d %H:%M:%S]'      # Timestamp each output line
tail -f log.txt | ts                          # Timestamp log stream
```

### ifne (run only if input)
```bash
find . -name "*.tmp" | ifne xargs rm         # Only delete if files found
grep ERROR log.txt | ifne mail -s "Errors" admin@co
```

### pv (progress monitor)
```bash
pv large_file.gz | gunzip > output           # Progress bar for pipe
dd if=/dev/sda | pv | gzip > backup.gz       # Monitor dd progress
cat big.sql | pv | psql mydb                 # SQL import progress
```

### flock (file locking)
```bash
flock /tmp/script.lock ./script.sh           # Only one instance at a time
flock -w 5 /tmp/lock command                 # Wait 5 seconds for lock
flock -n /tmp/lock command || echo "locked"  # Non-blocking, fail if locked
```

### expect (script interactive programs)
```bash
# Auto-answer prompts
expect -c '
  spawn ssh user@host
  expect "password:"
  send "mypassword\r"
  interact
'
```

---

## Installation

```bash
# Core tools (usually pre-installed via procps)
sudo apt install procps                       # ps, top, pgrep, pkill, pstree, etc.

# Additional tools
sudo apt install htop                         # Interactive process viewer
sudo apt install btop                         # Modern resource monitor
sudo apt install parallel                     # GNU parallel
sudo apt install lsof                         # List open files
sudo apt install psmisc                       # fuser, killall, pstree

# Verify installation
parallel --version
htop --version
lsof -v
```
