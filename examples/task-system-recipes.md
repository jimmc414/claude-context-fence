---
description: "System administration recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-system with target and context included"
when-to-use: "Receives prepared arguments from task-system with target and context included"
context: fork
model: inherit
allowed-tools: Read
---

# System Administration Recipes

**Tools:** uname, hostname, hostnamectl, uptime, free, vmstat, iostat, mpstat, sar, lscpu, lsmem, lshw, lspci, lsusb, dmidecode, sensors, nvidia-smi, smartctl, hdparm, dmesg, journalctl, systemctl, systemd-analyze, sysctl, crontab, at, batch, date, timedatectl, hwclock, dateadd, datediff, dateround, dateseq, lsmod, modprobe, modinfo

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| System info | `uname -a` |
| CPU info | `lscpu` |
| Memory | `free -h` |
| Disk usage | `df -h` |
| Load average | `uptime` |
| Service status | `systemctl status nginx` |
| Recent logs | `journalctl -u nginx --since '1h ago'` |
| Follow logs | `journalctl -f -u nginx` |
| Edit crontab | `crontab -e` |
| Current date | `date '+%Y-%m-%d %H:%M:%S %Z'` |
| Epoch to date | `date -d @1705276800` |
| Sensors | `sensors` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| Service management | systemctl | Standard systemd interface |
| System journal | journalctl | Structured log queries |
| Kernel messages | dmesg | Boot and hardware messages |
| Memory usage | free -h | Quick overview |
| Detailed memory/IO | vmstat / iostat | Time-series system stats |
| CPU info | lscpu | Architecture and core details |
| Hardware overview | lshw | Comprehensive hardware tree |
| Date/time config | timedatectl | Timezone, NTP management |
| Date math | dateutils | Add, diff, round dates |
| Job scheduling | crontab | Periodic task execution |
| One-time scheduling | at | Single future execution |

---

## System Information

### uname

```bash
uname -a                              # All info: kernel, hostname, arch, OS
uname -r                              # Kernel release: 5.15.0-91-generic
uname -m                              # Machine architecture: x86_64
uname -n                              # Hostname
uname -o                              # Operating system: GNU/Linux
uname -s                              # Kernel name: Linux
uname -v                              # Kernel version string (build date)
uname -p                              # Processor type
```

### hostnamectl

```bash
hostnamectl                           # Full hostname info (static, pretty, transient, chassis, OS)
hostnamectl status                    # Same as above
hostnamectl set-hostname myserver     # Set static hostname
hostnamectl set-hostname "My Server" --pretty  # Set pretty hostname
hostnamectl set-hostname "" --pretty  # Clear pretty hostname
hostnamectl set-chassis server        # Set chassis type (server, desktop, laptop, vm, container)
hostnamectl set-deployment production # Set deployment environment
```

### uptime

```bash
uptime                                # Uptime, users, load average (1/5/15 min)
uptime -p                             # Pretty format: "up 5 days, 3 hours, 22 minutes"
uptime -s                             # System up since: "2024-01-10 08:15:03"
# Load average interpretation:
#   Values relative to CPU count: load 4.0 on 4-core = 100% utilized
#   1-min: current pressure, 5-min: recent trend, 15-min: longer trend
#   > CPU count = overloaded, processes queuing
```

### lscpu

```bash
lscpu                                 # Full CPU info: architecture, model, cores, threads, caches
lscpu -e                              # Extended: per-CPU details (online, socket, core, L1/L2/L3)
lscpu -e=CPU,SOCKET,CORE,ONLINE      # Select specific columns
lscpu -p                              # Parseable output for scripts
lscpu --json                          # JSON output
# Key fields:
#   Architecture, CPU(s), Thread(s) per core, Core(s) per socket
#   Socket(s), Model name, CPU MHz (current/min/max)
#   L1d/L1i/L2/L3 cache sizes
#   Virtualization type (VT-x, AMD-V), Hypervisor vendor
#   Vulnerability flags: Spectre, Meltdown, etc.
```

### lsmem

```bash
lsmem                                 # Memory ranges and total
lsmem -b                              # Sizes in bytes
lsmem --summary                       # Summary: online, offline, total
lsmem -o RANGE,SIZE,STATE,REMOVABLE,BLOCK  # Specific columns
```

### lshw

```bash
sudo lshw -short                      # Compact hardware summary (class, description)
sudo lshw -class memory               # Memory details (DIMMs, speed, type)
sudo lshw -class disk                 # Disk details (model, size, serial)
sudo lshw -class network              # Network interfaces (driver, speed, MAC)
sudo lshw -class display              # GPU details (driver, memory)
sudo lshw -class processor            # CPU details
sudo lshw -class storage              # Storage controllers
sudo lshw -html > hardware.html       # HTML report
sudo lshw -json                       # JSON output
sudo lshw -sanitize                   # Remove sensitive info (serial numbers)
```

### dmidecode

```bash
sudo dmidecode -t bios                # BIOS info (vendor, version, date, features)
sudo dmidecode -t system              # System info (manufacturer, product, serial, UUID)
sudo dmidecode -t memory              # Memory: DIMM slots, speed, type, size per slot
sudo dmidecode -t processor           # Processor socket info
sudo dmidecode -t chassis             # Chassis info (type, serial, asset tag)
sudo dmidecode -t baseboard           # Motherboard info
sudo dmidecode -t cache               # CPU cache details
sudo dmidecode -s system-serial-number  # Single string value
sudo dmidecode -s bios-version        # BIOS version string
sudo dmidecode -s system-product-name # Product name string
```

### /proc and /etc info files

```bash
cat /proc/cpuinfo                     # Per-core CPU info (model, MHz, cache, flags)
cat /proc/meminfo                     # Detailed memory: MemTotal, MemFree, MemAvailable, Buffers, Cached, SwapTotal, SwapFree
cat /proc/version                     # Kernel version and compiler info
cat /proc/loadavg                     # Load average: 1min 5min 15min running/total lastpid
cat /proc/uptime                      # Uptime in seconds, idle time in seconds
cat /etc/os-release                   # OS name, version, ID, pretty name
lsb_release -a                        # Distributor ID, description, release, codename
cat /etc/hostname                     # Hostname
cat /etc/machine-id                   # Unique machine identifier
```

---

## Memory & CPU Monitoring

### free

```bash
free -h                               # Human readable: total, used, free, shared, buff/cache, available
free -h -s 2                          # Repeat every 2 seconds
free -h -c 5                          # Repeat 5 times
free -h -s 2 -c 5                     # Every 2 seconds, 5 times
free -t                               # Include total row (RAM + swap)
free -b                               # Bytes (also -k KB, -m MB, -g GB)
free -w                               # Wide mode: separate buffers and cache columns
# Key understanding:
#   "available" = estimated memory available for new apps (without swapping)
#   "available" = free + reclaimable buffer/cache
#   "used" includes buffer/cache that can be reclaimed
#   Low "available" (< 10% of total) = memory pressure
#   Any swap used (> 0) = system has been under memory pressure
```

### vmstat

```bash
vmstat                                # Single snapshot
vmstat 2                              # Every 2 seconds, continuous
vmstat 2 5                            # Every 2 seconds, 5 times
vmstat -w                             # Wide output (doesn't truncate columns)
vmstat -a                             # Active/inactive memory instead of buffer/cache
vmstat -s                             # Event counters and memory stats
vmstat -d                             # Disk statistics
vmstat -D                             # Disk summary
vmstat -t                             # Add timestamp column
vmstat -S M                           # Display in megabytes
# Key fields:
#   r: processes running/waiting for CPU (> CPU count = saturated)
#   b: processes blocked on I/O (> 0 = I/O bottleneck)
#   swpd: swap used (> 0 = memory pressure has occurred)
#   si/so: swap in/out per second (> 0 = active swapping, bad for performance)
#   bi/bo: blocks in/out per second (disk I/O)
#   us: user CPU%, sy: system CPU%, wa: I/O wait% (> 20% = I/O bottleneck)
#   id: idle CPU%, st: steal% (VM host taking CPU)
```

### mpstat

```bash
mpstat                                # All CPUs aggregate
mpstat -P ALL                         # Per-CPU breakdown
mpstat -P ALL 1 5                     # Per-CPU, every 1 second, 5 times
mpstat -P 0,1,2,3                     # Specific CPUs only
mpstat -I ALL                         # Interrupt statistics
mpstat -o JSON 1 3                    # JSON output
# Key fields:
#   %usr: user-level, %sys: kernel-level, %iowait: waiting for I/O
#   %irq: hardware interrupts, %soft: software interrupts
#   %steal: hypervisor stealing time, %idle: idle
#   Unbalanced per-CPU load = affinity issue or single-threaded bottleneck
```

### iostat

```bash
iostat                                # CPU and disk summary
iostat -x                             # Extended disk statistics
iostat -xz                            # Extended, skip zero-activity devices
iostat -xz 1 5                        # Every 1 second, 5 times
iostat -xz -p sda                     # Specific device including partitions
iostat -h                             # Human readable sizes
iostat -t                             # Add timestamps
iostat -o JSON 1 3                    # JSON output
# Key fields:
#   r/s, w/s: reads/writes per second
#   rMB/s, wMB/s: read/write throughput
#   rrqm/s, wrqm/s: merged reads/writes (consecutive I/Os combined)
#   await: average wait time in ms (includes queue + service)
#   r_await, w_await: read/write await separately
#   svctm: average service time (deprecated, use await)
#   %util: device utilization (100% = saturated, but SSDs handle >100% OK due to parallelism)
```

### sar (System Activity Reporter)

```bash
# Real-time monitoring
sar -u 1 5                            # CPU usage every 1 second, 5 times
sar -r 1 5                            # Memory usage
sar -d 1 5                            # Disk I/O
sar -n DEV 1 5                        # Network interface stats
sar -n SOCK 1 5                       # Socket usage (total, TCP, UDP, RAW)
sar -q 1 5                            # Load average and run queue
sar -b 1 5                            # I/O transfer rate
sar -w 1 5                            # Process creation and context switches
sar -v 1 5                            # Kernel tables (dentries, inodes, file handles)
sar -B 1 5                            # Paging statistics

# Historical data (from /var/log/sysstat/sa##)
sar -u -f /var/log/sysstat/sa15       # CPU from day 15 of current month
sar -r -f /var/log/sysstat/sa15       # Memory from day 15
sar -u -s 09:00:00 -e 17:00:00       # Today's CPU between 9am-5pm
sar -u -s 09:00:00 -e 17:00:00 -f /var/log/sysstat/sa15  # Specific day and time range

# All-in-one
sar -A                                # All statistics at once
```

### /proc/loadavg

```bash
cat /proc/loadavg                     # "1.25 0.89 0.75 2/350 12345"
# Fields: 1min 5min 15min running_processes/total_processes last_pid
# Load > nproc (CPU count) = CPU-bound processes are queuing
# Rising 1min > 5min > 15min = load increasing
# Falling 1min < 5min < 15min = load decreasing
```

---

## Hardware & Sensors

### lspci

```bash
lspci                                 # List all PCI devices
lspci -v                              # Verbose: driver, IRQ, memory regions
lspci -vv                             # More verbose: capabilities, power management
lspci -k                              # Show kernel drivers in use
lspci -t                              # Tree view of PCI bus hierarchy
lspci -nn                             # Show vendor:device IDs in numeric format
lspci -s 01:00.0                      # Specific device by bus:slot.func
lspci -s 01:00.0 -v                   # Verbose info for specific device
lspci -d 8086:*                       # Filter by vendor ID (8086 = Intel)
lspci -d :1234                        # Filter by device ID
```

### lsusb

```bash
lsusb                                 # List all USB devices
lsusb -t                              # Tree view with driver info
lsusb -v                              # Verbose: all descriptors
lsusb -s 001:002                      # Specific device by bus:device
lsusb -d 0x1234:0x5678               # Filter by vendor:product ID
```

### sensors (lm-sensors)

```bash
# Setup (first time)
sudo sensors-detect                   # Detect sensor chips (follow prompts, say yes)
sudo modprobe coretemp                # Load CPU temp module if needed

# Usage
sensors                               # All sensor readings (temps, fans, voltages)
sensors -f                            # Fahrenheit instead of Celsius
sensors -u                            # Raw output (for scripting)
sensors -j                            # JSON output
sensors coretemp-isa-0000             # Specific chip only
watch -n 1 sensors                    # Live monitoring every 1 second
# Output includes:
#   CPU core temperatures (Core 0, Core 1, ...)
#   Fan speeds (RPM)
#   Voltages (in0, 3.3V, 5V, 12V)
#   High/critical thresholds
```

### nvidia-smi

```bash
nvidia-smi                            # GPU summary: name, temp, utilization, memory, processes
nvidia-smi -l 1                       # Repeat every 1 second
nvidia-smi -q                         # Full query (all details)
nvidia-smi -q -d TEMPERATURE          # Temperature details only
nvidia-smi -q -d MEMORY               # Memory details only
nvidia-smi -q -d UTILIZATION          # Utilization details only
nvidia-smi -q -d CLOCK                # Clock speeds
nvidia-smi -q -d POWER                # Power consumption

# CSV monitoring for logging/graphing
nvidia-smi --query-gpu=timestamp,name,gpu_util,memory.used,memory.total,temperature.gpu,power.draw --format=csv -l 1
nvidia-smi --query-gpu=index,gpu_util,memory.used --format=csv,noheader,nounits -l 1

# Process monitoring
nvidia-smi pmon -i 0                  # Per-process monitoring on GPU 0
nvidia-smi pmon -s um -i 0            # Process utilization and memory

# Device monitoring
nvidia-smi dmon -i 0                  # Device monitoring (power, temp, util, memory)
nvidia-smi dmon -i 0 -d 1            # Every 1 second
nvidia-smi dmon -s pucet             # Select fields: power, utilization, clock, ecc, temperature

watch -n 1 nvidia-smi                 # Live display every 1 second
```

### smartctl (Disk Health)

```bash
sudo smartctl -a /dev/sda             # All SMART info for disk
sudo smartctl -H /dev/sda             # Health status: PASSED or FAILED
sudo smartctl -i /dev/sda             # Device info (model, serial, firmware, capacity)
sudo smartctl -A /dev/sda             # SMART attributes table
sudo smartctl -l error /dev/sda       # Error log
sudo smartctl -l selftest /dev/sda    # Self-test log

# Run tests
sudo smartctl -t short /dev/sda       # Short self-test (~2 minutes)
sudo smartctl -t long /dev/sda        # Long self-test (~hours, thorough)
sudo smartctl -t conveyance /dev/sda  # Conveyance test (after shipping)

# NVMe drives
sudo smartctl -a /dev/nvme0           # NVMe SMART info
sudo smartctl -H /dev/nvme0           # NVMe health
# Key attributes to watch:
#   Reallocated_Sector_Ct: > 0 = bad sectors replaced, disk degrading
#   Current_Pending_Sector: > 0 = sectors waiting to be remapped
#   UDMA_CRC_Error_Count: > 0 = cable/connection issues
#   Wear_Leveling_Count (SSD): lower = more wear
#   Temperature_Celsius: > 50C sustained = concerning
```

### hdparm

```bash
sudo hdparm -tT /dev/sda              # Benchmark: buffered and cached read speed
sudo hdparm -I /dev/sda               # Detailed drive info (model, features, modes)
sudo hdparm -i /dev/sda               # Brief drive info from kernel
sudo hdparm -g /dev/sda               # Drive geometry
sudo hdparm --security-help           # Security feature help
# Benchmark interpretation:
#   Timing cached reads = RAM speed (should be several GB/s)
#   Timing buffered reads = actual disk throughput
#   HDD: ~100-200 MB/s, SATA SSD: ~500 MB/s, NVMe: ~2000-7000 MB/s
```

---

## systemd Services

### systemctl

#### Service Management

```bash
systemctl status nginx                # Status: loaded, active/inactive, recent logs
systemctl start nginx                 # Start service
systemctl stop nginx                  # Stop service
systemctl restart nginx               # Stop then start
systemctl reload nginx                # Reload config without stopping (if supported)
systemctl reload-or-restart nginx     # Reload if possible, otherwise restart
systemctl enable nginx                # Enable at boot (creates symlink)
systemctl disable nginx               # Disable at boot (removes symlink)
systemctl enable --now nginx          # Enable and start immediately
systemctl disable --now nginx         # Disable and stop immediately
systemctl mask nginx                  # Prevent starting (even manually)
systemctl unmask nginx                # Remove mask
```

#### Service Inspection

```bash
systemctl is-active nginx             # Returns "active" or "inactive"
systemctl is-enabled nginx            # Returns "enabled", "disabled", "masked"
systemctl is-failed nginx             # Returns "failed" or "active"
systemctl show nginx                  # All properties
systemctl show -p MainPID nginx       # Specific property: main process ID
systemctl show -p MemoryCurrent nginx # Current memory usage
systemctl show -p CPUUsageNSec nginx  # CPU time consumed
systemctl show -p ActiveState nginx   # Active state
systemctl show -p SubState nginx      # Sub-state (running, dead, failed)
systemctl cat nginx                   # Show unit file contents
systemctl edit nginx                  # Create override (drop-in) file
systemctl edit --full nginx           # Edit full unit file
systemctl edit --force myapp.service  # Create new unit file
```

#### Listing Services

```bash
systemctl list-units --type=service               # All loaded service units
systemctl list-units --type=service --state=running  # Running services only
systemctl list-units --type=service --state=failed   # Failed services only
systemctl list-units --type=service --all            # All including inactive
systemctl list-unit-files --type=service             # All installed service files (enabled/disabled)
systemctl list-unit-files --type=service --state=enabled  # Enabled services only
systemctl list-dependencies nginx                    # Dependency tree
systemctl list-dependencies nginx --reverse          # What depends on nginx
```

#### Reload After Changes

```bash
systemctl daemon-reload               # Reload unit files after editing (required!)
systemctl daemon-reexec               # Re-execute systemd manager (rare, after systemd update)
```

### Writing Unit Files

Unit files go in `/etc/systemd/system/` (admin overrides) or `/usr/lib/systemd/system/` (package defaults).

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application Server
Documentation=https://example.com/docs
After=network.target postgresql.service
Requires=postgresql.service
Wants=redis.service
ConditionPathExists=/opt/myapp/bin/server

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
Environment=NODE_ENV=production
Environment=PORT=3000
EnvironmentFile=/opt/myapp/.env
ExecStartPre=/opt/myapp/bin/check-config
ExecStart=/opt/myapp/bin/server --port 3000
ExecStartPost=/usr/bin/curl -s http://localhost:3000/health
ExecStop=/bin/kill -SIGTERM $MAINPID
ExecReload=/bin/kill -SIGHUP $MAINPID
Restart=on-failure
RestartSec=5
WatchdogSec=30
TimeoutStartSec=30
TimeoutStopSec=30
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

# Security hardening
ProtectSystem=strict
ProtectHome=yes
NoNewPrivileges=yes
PrivateTmp=yes
PrivateDevices=yes
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes
ReadWritePaths=/opt/myapp/data /var/log/myapp
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX
RestrictNamespaces=yes
RestrictRealtime=yes
RestrictSUIDSGID=yes
LockPersonality=yes
SystemCallArchitectures=native

# Resource limits
MemoryMax=512M
MemoryHigh=400M
CPUQuota=200%
TasksMax=100
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

#### Service Types

```bash
# Type=simple       Default. Process started by ExecStart is the main process.
# Type=forking      ExecStart forks, parent exits. Set PIDFile= for tracking.
# Type=oneshot      Process exits after completing. Use with RemainAfterExit=yes for status tracking.
# Type=notify       Like simple, but service sends sd_notify("READY=1") when ready.
# Type=dbus         Ready when specified BusName appears on D-Bus.
# Type=idle         Like simple, but delayed until all jobs finish.
```

### systemd-analyze

```bash
systemd-analyze                       # Total boot time (kernel + userspace)
systemd-analyze blame                 # Services sorted by startup time (slowest first)
systemd-analyze blame | head -20      # Top 20 slowest services
systemd-analyze critical-chain        # Critical path: chain of blocking dependencies
systemd-analyze critical-chain nginx  # Critical chain for specific unit
systemd-analyze plot > boot.svg       # SVG timeline of entire boot process
systemd-analyze dot | dot -Tsvg > deps.svg  # Dependency graph
systemd-analyze security nginx        # Security audit of unit (scored 0-10, lower = more secure)
systemd-analyze verify myapp.service  # Validate unit file syntax
systemd-analyze calendar "Mon..Fri *-*-* 09:00"  # Parse calendar expression
systemd-analyze timespan "2h 30min"   # Parse time span
```

---

## systemd Timers

### Timer Unit File

```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Run backup daily at 2am

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true
RandomizedDelaySec=30min
AccuracySec=1s

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/backup.service (paired with timer)
[Unit]
Description=Backup job

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
StandardOutput=journal
```

### Timer Triggers

```ini
# Time-based (calendar)
OnCalendar=*-*-* 02:00:00            # Daily at 2am
OnCalendar=Mon..Fri *-*-* 09:00:00   # Weekdays at 9am
OnCalendar=*-*-01 00:00:00           # First of every month at midnight
OnCalendar=*-01,04,07,10-01 00:00:00 # Quarterly
OnCalendar=hourly                     # Every hour (:00)
OnCalendar=daily                      # Daily (00:00:00)
OnCalendar=weekly                     # Monday 00:00:00
OnCalendar=monthly                    # 1st of month 00:00:00
OnCalendar=*:0/15                     # Every 15 minutes
OnCalendar=*-*-* *:00,30:00          # Every 30 minutes

# Boot/activation-based
OnBootSec=5min                        # 5 minutes after boot
OnUnitActiveSec=1h                    # 1 hour after last activation (recurring)
OnUnitInactiveSec=30min               # 30 minutes after unit becomes inactive
OnStartupSec=10min                    # 10 minutes after systemd start

# Options
Persistent=true                       # Run missed executions after downtime
RandomizedDelaySec=30min              # Add random delay (0 to 30min) to prevent thundering herd
AccuracySec=1s                        # Timer accuracy (default 1min, set lower if exact timing needed)
```

### Timer Management

```bash
systemctl list-timers                 # List all active timers with next/last trigger
systemctl list-timers --all           # Include inactive timers
systemctl start backup.timer          # Activate timer
systemctl enable backup.timer         # Enable timer at boot
systemctl stop backup.timer           # Deactivate timer
systemctl status backup.timer         # Timer status and next trigger time
journalctl -u backup.service          # Logs from timer-triggered runs

# Quick transient timer (no files needed)
systemd-run --on-active=30s /usr/local/bin/task.sh          # Run in 30 seconds
systemd-run --on-calendar="*:0/5" /usr/local/bin/task.sh    # Every 5 minutes
systemd-run --on-boot=5min --timer-property=Persistent=true /usr/local/bin/task.sh
```

---

## Logging (journalctl)

### Basic Usage

```bash
journalctl                            # All logs (paged)
journalctl -f                         # Follow new entries (like tail -f)
journalctl -n 50                      # Last 50 lines
journalctl -n 100 --no-pager          # Last 100 lines without paging
journalctl -r                         # Reverse order (newest first)
journalctl -x                         # Include explanatory catalog text
journalctl --no-hostname              # Omit hostname field (cleaner output)
```

### Filtering by Unit

```bash
journalctl -u nginx                   # Logs for nginx service
journalctl -u nginx -u php-fpm        # Multiple units
journalctl -u nginx --since '1 hour ago'  # Last hour
journalctl -u nginx -f                # Follow nginx logs
journalctl -u nginx -n 200           # Last 200 lines for nginx
```

### Filtering by Time

```bash
journalctl --since "2024-01-15 08:00:00"            # Since specific time
journalctl --since "2024-01-15" --until "2024-01-16" # Date range
journalctl --since "1 hour ago"                      # Relative time
journalctl --since "30 min ago"                      # Last 30 minutes
journalctl --since "yesterday"                       # Since yesterday
journalctl --since "today"                           # Since midnight today
journalctl --since "2024-01-15" --until "1 hour ago" # Combined absolute and relative
```

### Filtering by Priority

```bash
journalctl -p emerg                   # Emergency: system is unusable
journalctl -p alert                   # Alert: action must be taken immediately
journalctl -p crit                    # Critical: critical conditions
journalctl -p err                     # Error: error conditions
journalctl -p warning                 # Warning: warning conditions
journalctl -p notice                  # Notice: normal but significant
journalctl -p info                    # Info: informational
journalctl -p debug                   # Debug: debug-level messages
journalctl -p err..crit               # Range: err through crit
journalctl -u nginx -p err            # Errors for specific unit
```

### Filtering by Boot

```bash
journalctl -b                         # Current boot only
journalctl -b -1                      # Previous boot
journalctl -b -2                      # Two boots ago
journalctl --list-boots               # List all recorded boots
journalctl -b 0 -p err               # Errors from current boot
```

### Field Matching

```bash
journalctl _PID=1234                  # By process ID
journalctl _UID=1000                  # By user ID
journalctl _GID=1000                  # By group ID
journalctl _SYSTEMD_UNIT=nginx.service  # By systemd unit (same as -u)
journalctl _TRANSPORT=kernel          # Kernel messages only
journalctl _TRANSPORT=syslog          # Syslog messages only
journalctl _COMM=sshd                 # By command name
journalctl _EXE=/usr/sbin/sshd       # By executable path
journalctl _HOSTNAME=myserver         # By hostname
journalctl CONTAINER_NAME=mycontainer # Container logs (if using journald driver)
```

### Pattern Matching

```bash
journalctl -g "error|fail|timeout"    # Grep for pattern (regex)
journalctl -u nginx -g "502\|503"     # Grep nginx for 502 or 503
journalctl -g "authentication failure" # Auth failures
journalctl -g "Out of memory"         # OOM events
```

### Output Formats

```bash
journalctl -o short                   # Default: timestamp hostname process[pid]: message
journalctl -o short-precise           # Microsecond timestamps
journalctl -o short-iso               # ISO 8601 timestamps
journalctl -o short-full              # Full date and time
journalctl -o json                    # One JSON object per line
journalctl -o json-pretty             # Pretty-printed JSON
journalctl -o cat                     # Message only (no metadata)
journalctl -o verbose                 # All fields for each entry
journalctl -o export                  # Binary export format (for import)
```

### Cursor-based Streaming

```bash
journalctl --cursor="s=abc123..."     # Start from specific cursor
journalctl --after-cursor="s=abc123..." # Start after specific cursor
journalctl -f --after-cursor="s=abc123..." # Follow from cursor
# Useful for log streaming: save cursor, resume later
journalctl -o json -n 1 | jq -r '.__CURSOR'  # Get current cursor
```

### Log Management

```bash
journalctl --disk-usage               # Show journal disk usage
journalctl --vacuum-size=500M         # Reduce to 500MB
journalctl --vacuum-time=30d          # Remove entries older than 30 days
journalctl --vacuum-files=5           # Keep only 5 journal files
journalctl --rotate                   # Force log rotation

# Persistent config: /etc/systemd/journald.conf
# SystemMaxUse=500M                   # Max disk usage
# SystemMaxFileSize=50M               # Max single file size
# MaxRetentionSec=30day               # Max retention time
# RateLimitIntervalSec=30s            # Rate limit window
# RateLimitBurst=10000                # Max messages per window
# Compress=yes                        # Compress stored logs
# Storage=persistent                  # Store in /var/log/journal (survives reboot)
```

### dmesg (Kernel Ring Buffer)

```bash
dmesg                                 # All kernel messages
dmesg -T                              # Human-readable timestamps
dmesg -H                              # Human-readable with pager and color
dmesg --color=always                  # Force color output
dmesg -l err,warn                     # Filter by level: errors and warnings
dmesg -l err                          # Errors only
dmesg -f kern                         # Facility: kernel messages
dmesg -f kern,daemon                  # Multiple facilities
dmesg --follow                        # Follow new kernel messages (like journalctl -k -f)
dmesg -c                              # Display and clear ring buffer (requires root)
dmesg | tail -50                      # Last 50 kernel messages
dmesg -T | grep -i "error\|fail\|oom\|kill"  # Search for problems
dmesg -T | grep -i "usb"             # USB device events
dmesg -T | grep -i "link"            # Network link state changes
```

---

## Scheduling (cron)

### Cron Syntax

```
# ┌───────────── minute (0-59)
# │ ┌───────────── hour (0-23)
# │ │ ┌───────────── day of month (1-31)
# │ │ │ ┌───────────── month (1-12 or jan-dec)
# │ │ │ │ ┌───────────── day of week (0-7, 0=Sunday, 7=Sunday, or mon-sun)
# │ │ │ │ │
# * * * * * command
```

### Common Schedules

```bash
* * * * *         # Every minute
*/5 * * * *       # Every 5 minutes
*/15 * * * *      # Every 15 minutes
0 * * * *         # Every hour at :00
0 */2 * * *       # Every 2 hours at :00
30 * * * *        # Every hour at :30
0 0 * * *         # Daily at midnight
0 6 * * *         # Daily at 6am
0 2 * * *         # Daily at 2am
0 0 * * 0         # Weekly on Sunday at midnight
0 0 * * 1         # Weekly on Monday at midnight
0 0 1 * *         # Monthly on the 1st at midnight
0 0 1 1 *         # Yearly on January 1st at midnight
0 9 * * 1-5       # Weekdays at 9am
0 0 * * 6,0       # Weekends at midnight
0 9,17 * * *      # 9am and 5pm daily
0 0 1,15 * *      # 1st and 15th of each month at midnight
```

### Special Strings

```bash
@yearly           # 0 0 1 1 *   (January 1st at midnight)
@annually         # Same as @yearly
@monthly          # 0 0 1 * *   (1st of month at midnight)
@weekly           # 0 0 * * 0   (Sunday at midnight)
@daily            # 0 0 * * *   (midnight)
@midnight         # Same as @daily
@hourly           # 0 * * * *   (every hour at :00)
@reboot           # Run once at system startup
```

### crontab Management

```bash
crontab -e                            # Edit current user's crontab
crontab -l                            # List current user's crontab
crontab -r                            # Remove current user's crontab (careful!)
crontab -u www-data -e                # Edit another user's crontab (requires root)
crontab -u www-data -l                # List another user's crontab
crontab /path/to/crontab-file         # Install crontab from file (replaces existing!)
```

### Cron Environment

```bash
# Set environment variables at top of crontab
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=admin@example.com
LANG=en_US.UTF-8
HOME=/home/myuser

# Example complete crontab
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin
MAILTO=admin@example.com

# Backup database daily at 2am, log output
0 2 * * * /usr/local/bin/backup-db.sh >> /var/log/backup.log 2>&1

# Clean temp files every hour
0 * * * * find /tmp -type f -mtime +7 -delete

# Health check every 5 minutes, email on failure
*/5 * * * * /usr/local/bin/healthcheck.sh || echo "Health check failed" | mail -s "Alert" admin@example.com
```

### Cron Debugging

```bash
# Check if cron is running
systemctl status cron
systemctl status crond                # On RHEL/CentOS

# View cron logs
grep CRON /var/log/syslog             # Debian/Ubuntu
grep CRON /var/log/cron               # RHEL/CentOS
journalctl -u cron                    # systemd

# Common gotchas:
# 1. PATH is minimal in cron (use full paths: /usr/bin/python3 not python3)
# 2. No interactive environment (no aliases, no .bashrc)
# 3. % is special in crontab (means newline), escape with \%
# 4. Relative paths resolve from HOME, not script location
# 5. No DISPLAY variable (GUI apps won't work without setting it)
# 6. Check file permissions: script must be executable
# 7. Redirect output or MAILTO will generate mail for any output
```

### at (One-Time Scheduling)

```bash
# Schedule one-time jobs
echo '/usr/local/bin/task.sh' | at 2:00 AM              # At 2am today/tomorrow
echo '/usr/local/bin/task.sh' | at 2:00 AM tomorrow     # At 2am tomorrow
echo '/usr/local/bin/task.sh' | at now + 1 hour          # In 1 hour
echo '/usr/local/bin/task.sh' | at now + 30 minutes      # In 30 minutes
echo '/usr/local/bin/task.sh' | at now + 2 days          # In 2 days
echo '/usr/local/bin/task.sh' | at 4pm + 3 days          # 4pm, 3 days from now
echo '/usr/local/bin/task.sh' | at 10:00 AM Jul 31       # Specific date

# Management
atq                                   # List pending jobs (job ID, time, queue, user)
at -c 5                               # Show job 5 contents
atrm 5                                # Remove job 5

# batch: runs when system load drops below 1.5
echo '/usr/local/bin/heavy-task.sh' | batch
```

---

## Date & Time

### date Format Strings

```bash
date '+%Y-%m-%d'                      # 2024-01-15
date '+%Y-%m-%d %H:%M:%S'            # 2024-01-15 14:30:00
date '+%Y-%m-%d %H:%M:%S %Z'         # 2024-01-15 14:30:00 EST
date '+%Y%m%d_%H%M%S'                # 20240115_143000 (filename-safe)
date '+%F'                            # 2024-01-15 (same as %Y-%m-%d)
date '+%T'                            # 14:30:00 (same as %H:%M:%S)
date '+%F %T %Z'                      # 2024-01-15 14:30:00 EST
date '+%s'                            # 1705344600 (Unix epoch seconds)
date '+%s%N'                          # Epoch with nanoseconds
date '+%A, %B %d, %Y'                # Monday, January 15, 2024
date '+%a %b %d %Y'                  # Mon Jan 15 2024
date '+%j'                            # Day of year (001-366)
date '+%u'                            # Day of week (1=Monday, 7=Sunday)
date '+%V'                            # ISO week number (01-53)
date '+%z'                            # Timezone offset: -0500
date '+%:z'                           # Timezone offset: -05:00
date '+%Z'                            # Timezone abbreviation: EST
date '+%c'                            # Locale's date and time
```

### date -d (Date Parsing and Arithmetic)

```bash
# Relative dates
date -d "now"                         # Current time (same as date)
date -d "today"                       # Today at 00:00:00
date -d "yesterday"                   # Yesterday at 00:00:00
date -d "tomorrow"                    # Tomorrow at 00:00:00
date -d "next friday"                 # Next Friday at 00:00:00
date -d "last monday"                 # Previous Monday at 00:00:00
date -d "next week"                   # 7 days from now
date -d "next month"                  # Same day next month
date -d "next year"                   # Same day next year

# Arithmetic
date -d "+3 days"                     # 3 days from now
date -d "-3 days"                     # 3 days ago
date -d "+2 hours"                    # 2 hours from now
date -d "-30 minutes"                 # 30 minutes ago
date -d "+1 month"                    # 1 month from now
date -d "-1 year"                     # 1 year ago
date -d "+1 day +3 hours"            # 1 day and 3 hours from now
date -d "2024-01-15 +30 days"        # 30 days after specific date
date -d "2024-01-15 -1 month"        # 1 month before specific date

# Parse specific dates
date -d "2024-06-15"                  # Parse ISO date
date -d "2024-06-15 14:30:00"        # Parse datetime
date -d "2024-06-15 14:30:00 UTC"    # Parse with timezone
date -d "Jun 15, 2024"               # Parse natural date
date -d "15 Jun 2024"                # Another natural format

# Epoch conversion
date -d @1705276800                   # Epoch to human readable
date -d @1705276800 '+%Y-%m-%d %H:%M:%S %Z'  # Epoch to formatted
date -d @0                            # Epoch 0 = 1970-01-01 00:00:00 UTC
date +%s                              # Current time to epoch
date -d "2024-06-15 14:30:00 UTC" +%s  # Specific time to epoch
```

### Timezone Operations

```bash
# Show current timezone
date '+%Z %z'                         # Abbreviation and offset
timedatectl                           # Full timezone info

# Display time in other timezone
TZ=UTC date                           # UTC
TZ=America/New_York date              # US Eastern
TZ=America/Los_Angeles date           # US Pacific
TZ=Europe/London date                 # UK
TZ=Asia/Tokyo date                    # Japan
TZ=Australia/Sydney date              # Australia Eastern

# Convert between timezones
TZ=America/New_York date -d "TZ=\"UTC\" 2024-01-15 14:30:00"  # UTC to Eastern
TZ=UTC date -d "TZ=\"America/New_York\" 2024-01-15 09:30:00"  # Eastern to UTC

# Compare times across zones
for tz in UTC America/New_York America/Los_Angeles Europe/London Asia/Tokyo; do
  echo "$tz: $(TZ=$tz date '+%Y-%m-%d %H:%M %Z')"
done
```

### timedatectl

```bash
timedatectl                           # Show time, timezone, NTP status
timedatectl status                    # Same as above
timedatectl list-timezones            # List all available timezones
timedatectl list-timezones | grep America  # Filter timezones
timedatectl set-timezone America/New_York  # Set timezone
timedatectl set-time "2024-01-15 14:30:00" # Set time manually (disables NTP)
timedatectl set-ntp true              # Enable NTP synchronization
timedatectl set-ntp false             # Disable NTP
timedatectl timesync-status           # NTP sync status (server, poll interval, offset)
```

### hwclock

```bash
hwclock --show                        # Show hardware clock time
hwclock --systohc                     # Set hardware clock from system time
hwclock --hctosys                     # Set system time from hardware clock
hwclock --set --date "2024-01-15 14:30:00"  # Set hardware clock manually
hwclock --utc                         # Show/set assuming UTC hardware clock
hwclock --localtime                   # Show/set assuming local time hardware clock
```

### dateutils (Extended Date Operations)

```bash
# dateadd: add duration to date
dateadd 2024-01-15 +30d              # Add 30 days: 2024-02-14
dateadd 2024-01-15 +1m               # Add 1 month: 2024-02-15
dateadd 2024-01-15 +1y               # Add 1 year: 2025-01-15
dateadd 2024-01-15T14:30:00 +2h      # Add 2 hours
dateadd 2024-01-15 -10d              # Subtract 10 days
dateadd today +90d                    # 90 days from today

# datediff: difference between dates
datediff 2024-01-01 2024-06-15       # Days between: 166
datediff 2024-01-01 2024-06-15 -f '%m months %d days'  # Formatted difference
datediff 2024-01-01 2024-12-31 -f '%w weeks'           # Weeks between
datediff 2024-01-01T09:00 2024-01-01T17:30 -f '%Hh%Mm' # Hours and minutes

# dateround: round dates
dateround 2024-03-15 -m              # Round to month: 2024-03-01
dateround 2024-03-15 +m              # Round up to month: 2024-04-01
dateround 2024-03-15 -y              # Round to year: 2024-01-01
dateround now -d                      # Round to start of today

# dateseq: generate date sequences
dateseq 2024-01-01 2024-01-31        # Every day in January
dateseq 2024-01-01 2024-12-31 +1m    # First of every month in 2024
dateseq 2024-01-01 +7d 2024-03-31    # Every 7 days for Q1
dateseq --format '%Y-%m-%d %A' 2024-01-01 2024-01-07  # With weekday names

# datesort: sort dates from stdin
echo -e "2024-03-15\n2024-01-10\n2024-06-20" | datesort
echo -e "2024-03-15\n2024-01-10\n2024-06-20" | datesort -r  # Reverse

# datezone: timezone conversion
datezone --from-zone=UTC --zone=America/New_York 2024-01-15T14:30:00
datezone --from-zone=America/New_York --zone=UTC 2024-01-15T09:30:00
datezone --zone=Asia/Tokyo 2024-01-15T14:30:00  # Local to Tokyo
```

---

## Kernel Modules

```bash
# List loaded modules
lsmod                                 # All loaded kernel modules
lsmod | grep nvidia                   # Filter for specific module
lsmod | sort -k 2 -n -r | head       # Sort by size (largest first)

# Module information
modinfo ext4                          # Details: author, description, params, depends
modinfo -p ext4                       # Parameters only
modinfo nvidia                        # GPU driver info

# Load/unload modules
sudo modprobe vfio-pci                # Load module (and dependencies)
sudo modprobe -r vfio-pci             # Unload module (and unused dependencies)
sudo modprobe bonding mode=1          # Load with parameters
sudo insmod /path/to/module.ko        # Load by path (no dependency resolution)
sudo rmmod modulename                 # Unload by name (no dependency handling)

# Persistent configuration (/etc/modprobe.d/)
# /etc/modprobe.d/blacklist-nouveau.conf:
#   blacklist nouveau
#   options nouveau modeset=0
# /etc/modprobe.d/bonding.conf:
#   options bonding mode=1 miimon=100

# DKMS (Dynamic Kernel Module Support)
dkms status                           # List DKMS-managed modules
sudo dkms install module/version      # Install for current kernel
sudo dkms remove module/version --all # Remove from all kernels
```

---

## sysctl (Kernel Parameters)

```bash
# Read parameters
sysctl -a                             # List all parameters (hundreds of entries)
sysctl -a 2>/dev/null | wc -l        # Count all parameters
sysctl vm.swappiness                  # Read specific parameter
sysctl net.ipv4.ip_forward            # Check IP forwarding
sysctl kernel.hostname                # Current hostname

# Write parameters (temporary, lost on reboot)
sudo sysctl -w vm.swappiness=10                        # Reduce swap aggressiveness
sudo sysctl -w net.core.somaxconn=65535                # Increase listen backlog
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535      # Increase SYN backlog
sudo sysctl -w net.ipv4.ip_forward=1                   # Enable IP forwarding
sudo sysctl -w fs.file-max=2097152                     # Increase max open files
sudo sysctl -w fs.inotify.max_user_watches=524288      # Increase inotify watches (for IDEs, file watchers)
sudo sysctl -w kernel.pid_max=4194304                  # Increase max PID
sudo sysctl -w vm.overcommit_memory=1                  # Allow overcommit (for Redis)
sudo sysctl -w net.ipv4.tcp_tw_reuse=1                 # Reuse TIME_WAIT sockets
sudo sysctl -w net.ipv4.tcp_fin_timeout=30             # Reduce FIN timeout
sudo sysctl -w net.core.rmem_max=16777216              # Increase receive buffer max
sudo sysctl -w net.core.wmem_max=16777216              # Increase send buffer max

# Persistent configuration (/etc/sysctl.d/)
# /etc/sysctl.d/99-custom.conf:
#   vm.swappiness = 10
#   net.core.somaxconn = 65535
#   net.ipv4.ip_forward = 1
#   fs.inotify.max_user_watches = 524288

# Apply persistent config
sudo sysctl -p                        # Reload /etc/sysctl.conf
sudo sysctl -p /etc/sysctl.d/99-custom.conf  # Reload specific file
sudo sysctl --system                  # Reload all sysctl config files
```

### Disk Investigation

```bash
# Disk usage analysis
du -ah /var | sort -rh | head -20              # Largest files/dirs in /var
du -sh /var/*                                  # Summary per subdirectory
du -sh --apparent-size /path                   # Apparent size (not disk blocks)
du -sh --exclude='*.log' /var                  # Exclude patterns
ncdu /var                                      # Interactive disk usage explorer

# Filesystem info
df -hT                                         # All filesystems with types
df -i                                          # Inode usage (can run out of inodes!)
df -h | awk '$5+0 > 80{print "WARNING:", $6, "is", $5}'  # Alert on >80% usage

# Find large files
find /var -type f -size +100M -exec ls -lh {} + 2>/dev/null | sort -k5 -rh
```

### User and Group Management

```bash
# Create users
useradd -m -s /bin/bash username               # Create with home dir and shell
useradd -m -s /bin/bash -G sudo,docker user    # Create with groups

# Modify users
usermod -aG docker username                    # Add to group (append)
usermod -s /bin/zsh username                   # Change shell
passwd username                                # Set password

# Delete users
userdel -r username                            # Remove user and home directory

# Group management
groupadd developers                            # Create group
gpasswd -a user developers                     # Add user to group
gpasswd -d user developers                     # Remove user from group

# User info
id username                                    # Show UID, GID, groups
who                                            # Who is logged in
w                                              # Who is logged in + what they're doing
last -n 20                                     # Recent login history
chage -l username                              # Password aging info
```

### Permissions

```bash
# chmod — change file mode
chmod 755 dir/                                 # rwxr-xr-x
chmod 644 file.txt                             # rw-r--r--
chmod u+x script.sh                            # Add execute for owner
chmod -R g+w shared/                           # Recursive group write

# chown — change ownership
chown user:group file                          # Change owner and group
chown -R user:group dir/                       # Recursive
chown --reference=ref_file target_file         # Match another file's ownership

# umask — default permission mask
umask 022                                      # New files: 644, dirs: 755
umask 077                                      # New files: 600, dirs: 700

# ACL (fine-grained permissions)
setfacl -m u:username:rwx file                 # Grant user specific permissions
setfacl -m g:developers:rx dir/                # Grant group permissions
getfacl file                                   # Show ACL
setfacl -R -m u:username:rx dir/               # Recursive ACL
setfacl -b file                                # Remove all ACL entries
```

### Package Management (Debian/Ubuntu)

```bash
# Update and upgrade
sudo apt update && sudo apt upgrade -y         # Update package lists and upgrade
sudo apt full-upgrade                          # Upgrade with dependency changes

# Install and remove
sudo apt install package                       # Install package
sudo apt remove package                        # Remove (keep config)
sudo apt purge package                         # Remove including config
sudo apt autoremove                            # Remove unused dependencies

# Search and info
apt search keyword                             # Search package names/descriptions
apt show package                               # Package details
dpkg -l | grep package                         # Check if installed
dpkg -L package                                # List files in installed package
apt-file search filename                       # Find which package provides a file

# Cleanup
sudo apt clean                                 # Clear package cache
sudo apt autoclean                             # Clear old package versions
```

### Combined Workflows

```bash
# Disk space triage
du -ah /var | sort -rh | head -20 && echo "---" && df -h | awk '$5+0>80'

# Service failure investigation
systemctl --failed | awk 'NR>1&&NF>1{print $2}' | xargs -I{} sh -c 'echo "=== {} ===" && journalctl -u {} -n 5 --no-pager'

# System health summary
echo "=== Load ===" && uptime && echo "=== Memory ===" && free -h && echo "=== Disk ===" && df -h | awk '$5+0>50' && echo "=== Failed ===" && systemctl --failed --no-legend
```

---

## Installation

```bash
# Core tools (most pre-installed on modern Linux)
sudo apt install procps sysstat util-linux          # vmstat, iostat, mpstat, sar, lscpu, lsmem

# Hardware inspection
sudo apt install lshw pciutils usbutils dmidecode   # lshw, lspci, lsusb, dmidecode
sudo apt install smartmontools hdparm               # smartctl, hdparm

# Sensors
sudo apt install lm-sensors                         # sensors, sensors-detect

# Date utilities
sudo apt install dateutils                          # dateadd, datediff, dateround, dateseq

# All at once
sudo apt install procps sysstat util-linux lshw pciutils usbutils \
  dmidecode smartmontools hdparm lm-sensors dateutils
```
