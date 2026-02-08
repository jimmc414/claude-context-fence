# Remaining Skills - Draft for Review

These are the 14 remaining task skills, ready to be split into individual files in `.claude/commands/` after review.

---

# task-media.md

```markdown
---
description: "Media processing - video/audio conversion, image manipulation, metadata extraction, and document conversion"
when_to_use: "Use when: converting video formats, extracting audio from video, resizing images, reading EXIF metadata, creating thumbnails, converting documents to PDF, generating diagrams. Triggers: ffmpeg, video convert, audio extract, image resize, thumbnail, exif, pdf convert, diagram."
when-to-use: "Use when: converting video formats, extracting audio from video, resizing images, reading EXIF metadata, creating thumbnails, converting documents to PDF, generating diagrams. Triggers: ffmpeg, video convert, audio extract, image resize, thumbnail, exif, pdf convert, diagram."
argument-hint: "[describe what you want to do with media]"
context: fork
model: inherit
allowed-tools: Read
---

# Media Processing Tasks

Primary tools: **ffmpeg**, **imagemagick**, **sox**, **pandoc**

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Tool | Command |
|------|------|---------|
| Convert video format | ffmpeg | `ffmpeg -i input.mp4 output.webm` |
| Extract audio | ffmpeg | `ffmpeg -i video.mp4 -vn audio.mp3` |
| Resize video | ffmpeg | `ffmpeg -i input.mp4 -vf scale=1280:720 output.mp4` |
| Trim video | ffmpeg | `ffmpeg -i input.mp4 -ss 00:01:00 -t 00:00:30 output.mp4` |
| Resize image | convert | `convert input.jpg -resize 800x600 output.jpg` |
| Convert image format | convert | `convert input.png output.jpg` |
| Create thumbnail | convert | `convert input.jpg -thumbnail 150x150^ -gravity center -extent 150x150 thumb.jpg` |
| Get media info | ffprobe | `ffprobe -v quiet -print_format json -show_format input.mp4` |
| Read EXIF | exiftool | `exiftool image.jpg` |
| Strip EXIF | exiftool | `exiftool -all= image.jpg` |
| Convert to PDF | pandoc | `pandoc input.md -o output.pdf` |

---

## Video Operations (ffmpeg)

### Format Conversion
```bash
# Basic conversion (auto-detect codecs)
ffmpeg -i input.mp4 output.webm
ffmpeg -i input.mov output.mp4

# Specify codec
ffmpeg -i input.mp4 -c:v libx264 -c:a aac output.mp4

# Copy streams (fast, no re-encoding)
ffmpeg -i input.mp4 -c copy output.mkv
```

### Extract Audio
```bash
# Extract as MP3
ffmpeg -i video.mp4 -vn -acodec mp3 audio.mp3

# Extract as original format
ffmpeg -i video.mp4 -vn -acodec copy audio.aac

# Extract as WAV
ffmpeg -i video.mp4 -vn audio.wav
```

### Trim/Cut
```bash
# Cut from timestamp for duration
ffmpeg -i input.mp4 -ss 00:01:00 -t 00:00:30 -c copy output.mp4

# Cut from start to end timestamp
ffmpeg -i input.mp4 -ss 00:01:00 -to 00:01:30 -c copy output.mp4
```

### Resize/Scale
```bash
# Specific dimensions
ffmpeg -i input.mp4 -vf scale=1280:720 output.mp4

# Scale by percentage
ffmpeg -i input.mp4 -vf scale=iw/2:ih/2 output.mp4

# Scale to width, keep aspect ratio
ffmpeg -i input.mp4 -vf scale=640:-1 output.mp4
```

### Extract Frames
```bash
# Single frame at timestamp
ffmpeg -i input.mp4 -ss 00:00:05 -frames:v 1 frame.jpg

# One frame per second
ffmpeg -i input.mp4 -vf fps=1 frames/frame_%04d.jpg

# Extract all frames
ffmpeg -i input.mp4 frames/frame_%04d.jpg
```

### Create GIF
```bash
# Basic GIF
ffmpeg -i input.mp4 -vf "fps=10,scale=320:-1" output.gif

# High quality GIF (two-pass)
ffmpeg -i input.mp4 -vf "fps=10,scale=320:-1:flags=lanczos,palettegen" palette.png
ffmpeg -i input.mp4 -i palette.png -filter_complex "fps=10,scale=320:-1:flags=lanczos[x];[x][1:v]paletteuse" output.gif
```

---

## Image Operations (ImageMagick)

### Resize
```bash
# Resize to dimensions
convert input.jpg -resize 800x600 output.jpg

# Resize by percentage
convert input.jpg -resize 50% output.jpg

# Resize to fit within bounds (maintain aspect)
convert input.jpg -resize 800x600\> output.jpg
```

### Format Conversion
```bash
convert input.png output.jpg
convert input.jpg output.webp
convert input.svg output.png
```

### Thumbnails
```bash
# Simple thumbnail
convert input.jpg -thumbnail 150x150 thumb.jpg

# Square crop thumbnail
convert input.jpg -thumbnail 150x150^ -gravity center -extent 150x150 thumb.jpg
```

### Batch Operations
```bash
# Convert all PNGs to JPG
mogrify -format jpg *.png

# Resize all images in place
mogrify -resize 800x600 *.jpg
```

### Get Image Info
```bash
identify image.jpg
identify -verbose image.jpg
```

---

## Metadata (exiftool)

```bash
# Read all metadata
exiftool image.jpg

# Read specific fields
exiftool -DateTimeOriginal -GPSLatitude -GPSLongitude image.jpg

# Strip all metadata
exiftool -all= image.jpg

# Copy metadata between files
exiftool -TagsFromFile source.jpg target.jpg
```

---

## Document Conversion (pandoc)

```bash
# Markdown to PDF
pandoc input.md -o output.pdf

# Markdown to HTML
pandoc input.md -o output.html

# Markdown to DOCX
pandoc input.md -o output.docx

# With template
pandoc input.md --template=template.html -o output.html
```

---

## Audio Operations (sox)

```bash
# Convert format
sox input.wav output.mp3

# Trim audio
sox input.wav output.wav trim 0 30  # first 30 seconds

# Adjust volume
sox input.wav output.wav vol 0.5    # reduce by half

# Get audio info
soxi audio.wav
```

---

## Installation

```bash
# Core tools
sudo apt install ffmpeg imagemagick exiftool pandoc sox

# For PDF output with pandoc
sudo apt install texlive-xetex
```
```

---

# task-archives.md

```markdown
---
description: "Archive and compression - creating/extracting archives, compressing files, and handling various archive formats"
when_to_use: "Use when: creating tar archives, extracting zip files, compressing with gzip/zstd, handling 7z or rar files, batch compression, extracting specific files from archives. Triggers: tar, zip, unzip, gzip, compress, extract, archive, 7z, zstd, decompress."
when-to-use: "Use when: creating tar archives, extracting zip files, compressing with gzip/zstd, handling 7z or rar files, batch compression, extracting specific files from archives. Triggers: tar, zip, unzip, gzip, compress, extract, archive, 7z, zstd, decompress."
argument-hint: "[describe what you want to archive or extract]"
context: fork
model: inherit
allowed-tools: Read
---

# Archive and Compression Tasks

Primary tools: **tar**, **gzip**, **zip**, **7z**, **zstd**

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Create tar.gz | `tar -czvf archive.tar.gz directory/` |
| Extract tar.gz | `tar -xzvf archive.tar.gz` |
| Create tar.zst | `tar -I zstd -cvf archive.tar.zst directory/` |
| Extract tar.zst | `tar -I zstd -xvf archive.tar.zst` |
| Create zip | `zip -r archive.zip directory/` |
| Extract zip | `unzip archive.zip` |
| Create 7z | `7z a archive.7z directory/` |
| Extract 7z | `7z x archive.7z` |
| Compress file | `gzip file` |
| Decompress file | `gunzip file.gz` |
| List archive contents | `tar -tvf archive.tar.gz` |

---

## Tar Archives

### Create Archives
```bash
# tar.gz (gzip compression)
tar -czvf archive.tar.gz directory/
tar -czvf archive.tar.gz file1 file2 file3

# tar.bz2 (bzip2 - smaller but slower)
tar -cjvf archive.tar.bz2 directory/

# tar.xz (best compression, slowest)
tar -cJvf archive.tar.xz directory/

# tar.zst (zstd - fast and good compression)
tar -I zstd -cvf archive.tar.zst directory/
tar --zstd -cvf archive.tar.zst directory/

# Uncompressed tar
tar -cvf archive.tar directory/
```

### Extract Archives
```bash
# Auto-detect compression
tar -xvf archive.tar.gz
tar -xvf archive.tar.bz2
tar -xvf archive.tar.xz

# Extract to specific directory
tar -xvf archive.tar.gz -C /target/directory/

# Extract specific files
tar -xvf archive.tar.gz path/to/file.txt

# Extract zstd
tar -I zstd -xvf archive.tar.zst
```

### List Contents
```bash
tar -tvf archive.tar.gz
tar -tvf archive.tar.gz | grep pattern
```

### Tar Flags Explained
```
-c  create archive
-x  extract archive
-t  list contents
-v  verbose
-f  filename (must be last flag before filename)
-z  gzip compression
-j  bzip2 compression
-J  xz compression
-C  change to directory
```

---

## Zip Archives

```bash
# Create zip
zip archive.zip file1 file2
zip -r archive.zip directory/

# With compression level (0-9)
zip -9 -r archive.zip directory/

# Exclude patterns
zip -r archive.zip directory/ -x "*.log" -x "*.tmp"

# Extract zip
unzip archive.zip
unzip archive.zip -d /target/directory/

# List contents
unzip -l archive.zip

# Extract specific file
unzip archive.zip path/to/file.txt
```

---

## 7z Archives

```bash
# Create 7z (best compression)
7z a archive.7z directory/

# Create with password
7z a -p archive.7z directory/

# Extract
7z x archive.7z
7z x archive.7z -o/target/directory/

# List contents
7z l archive.7z

# Extract specific file
7z x archive.7z path/to/file.txt
```

---

## Individual File Compression

### gzip
```bash
gzip file              # Creates file.gz, removes original
gzip -k file           # Keep original
gzip -9 file           # Best compression
gunzip file.gz         # Decompress
zcat file.gz           # View without extracting
```

### bzip2
```bash
bzip2 file             # Creates file.bz2
bzip2 -k file          # Keep original
bunzip2 file.bz2       # Decompress
bzcat file.bz2         # View without extracting
```

### xz
```bash
xz file                # Creates file.xz
xz -k file             # Keep original
unxz file.xz           # Decompress
xzcat file.xz          # View without extracting
```

### zstd (recommended for speed + compression)
```bash
zstd file              # Creates file.zst
zstd -k file           # Keep original
zstd -19 file          # Max compression
unzstd file.zst        # Decompress
zstdcat file.zst       # View without extracting
```

---

## Compression Comparison

| Format | Speed | Compression | Best For |
|--------|-------|-------------|----------|
| gzip | Fast | Good | General use |
| bzip2 | Slow | Better | Archival |
| xz | Slowest | Best | Distribution |
| zstd | Fastest | Very Good | Modern default |

---

## Common Patterns

### Backup with date
```bash
tar -czvf "backup_$(date +%Y%m%d).tar.gz" directory/
```

### Parallel compression (pigz)
```bash
tar -cvf - directory/ | pigz > archive.tar.gz
pigz -dc archive.tar.gz | tar -xvf -
```

### Extract and strip leading directory
```bash
tar -xvf archive.tar.gz --strip-components=1
```

---

## Installation

```bash
# Most tools pre-installed

# Additional tools
sudo apt install p7zip-full    # 7z support
sudo apt install zstd          # zstd compression
sudo apt install pigz          # parallel gzip
```
```

---

# task-process.md

```markdown
---
description: "Process management - listing processes, job control, killing processes, resource limits, and parallel execution"
when_to_use: "Use when: listing running processes, killing processes by name or PID, running commands in background, setting process priority, limiting CPU/memory, running commands in parallel, batch processing with xargs. Triggers: ps, kill, process list, background job, parallel, xargs, nice, priority, timeout, lsof."
when-to-use: "Use when: listing running processes, killing processes by name or PID, running commands in background, setting process priority, limiting CPU/memory, running commands in parallel, batch processing with xargs. Triggers: ps, kill, process list, background job, parallel, xargs, nice, priority, timeout, lsof."
argument-hint: "[describe what you want to do with processes]"
context: fork
model: inherit
allowed-tools: Read
---

# Process Management Tasks

Primary tools: **ps**, **kill**, **xargs**, **parallel**, **lsof**

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| List all processes | `ps aux` |
| Find process by name | `pgrep -a nginx` |
| Kill by PID | `kill 1234` |
| Kill by name | `pkill nginx` |
| Force kill | `kill -9 1234` |
| Run in background | `command &` |
| Run immune to hangup | `nohup command &` |
| Limit execution time | `timeout 30s command` |
| Run with low priority | `nice -n 19 command` |
| Find what's using port | `lsof -i :8080` |
| Find what's using file | `lsof /path/to/file` |
| Parallel execution | `parallel command ::: arg1 arg2 arg3` |

---

## Listing Processes

### ps (process status)
```bash
# All processes, full format
ps aux

# All processes, tree view
ps auxf

# Filter by name
ps aux | grep nginx

# Custom columns
ps -eo pid,ppid,user,%cpu,%mem,cmd

# Sort by CPU usage
ps aux --sort=-%cpu | head

# Sort by memory usage
ps aux --sort=-%mem | head
```

### pgrep (find by name)
```bash
# Find PID by name
pgrep nginx

# Show full command
pgrep -a nginx

# Show with details
pgrep -l nginx

# Match exact name
pgrep -x nginx
```

### Process tree
```bash
pstree
pstree -p              # Show PIDs
pstree -a              # Show command arguments
pstree -u user         # Specific user
```

---

## Killing Processes

### kill (by PID)
```bash
# Graceful termination (SIGTERM)
kill 1234

# Force kill (SIGKILL)
kill -9 1234
kill -KILL 1234

# Other signals
kill -HUP 1234         # Reload config
kill -STOP 1234        # Pause process
kill -CONT 1234        # Resume process
```

### pkill / killall (by name)
```bash
# Kill by name
pkill nginx
killall nginx

# Force kill by name
pkill -9 nginx

# Kill by user
pkill -u username

# Kill processes older than 1 hour
pkill -o 3600 processname
```

---

## Job Control

### Background jobs
```bash
# Run in background
command &

# List background jobs
jobs

# Bring to foreground
fg %1

# Send to background
bg %1

# Disown (detach from shell)
disown %1
```

### nohup (survive logout)
```bash
# Run immune to hangups
nohup command &

# With output redirect
nohup command > output.log 2>&1 &
```

### timeout (limit execution time)
```bash
# Kill after 30 seconds
timeout 30s command

# Kill after 5 minutes
timeout 5m command

# Send different signal
timeout -s KILL 30s command

# With warning before kill
timeout --foreground 30s command
```

---

## Resource Limits

### nice / renice (CPU priority)
```bash
# Run with low priority (nice = 19, lowest)
nice -n 19 command

# Run with high priority (requires root)
sudo nice -n -20 command

# Change priority of running process
renice 10 -p 1234
```

### ionice (I/O priority)
```bash
# Run with low I/O priority
ionice -c 3 command

# Idle I/O (only when nothing else needs I/O)
ionice -c 3 -p 1234
```

### ulimit (resource limits)
```bash
# Show all limits
ulimit -a

# Set max memory
ulimit -v 1000000      # 1GB virtual memory

# Set max file descriptors
ulimit -n 10000
```

---

## Finding Resource Usage

### lsof (list open files)
```bash
# What's using a port
lsof -i :8080

# What's using a file
lsof /path/to/file

# All network connections
lsof -i

# Files opened by process
lsof -p 1234

# Files opened by user
lsof -u username
```

### fuser (file/port users)
```bash
# What's using a file
fuser /path/to/file

# What's using a port
fuser 8080/tcp

# Kill processes using file
fuser -k /path/to/file
```

---

## Parallel Execution

### xargs
```bash
# Basic parallel
echo "a b c" | xargs -n1 -P4 command

# From file, 4 parallel
cat files.txt | xargs -P4 -I{} process {}

# Find and process in parallel
find . -name "*.log" | xargs -P4 gzip
```

### GNU parallel
```bash
# Basic parallel
parallel command ::: arg1 arg2 arg3

# From file
parallel command :::: args.txt

# With placeholders
parallel convert {} {.}.png ::: *.jpg

# Limit jobs
parallel -j4 command ::: args
```

### Batch processing patterns
```bash
# Process files in parallel
find . -name "*.txt" | parallel gzip

# Run command for each line
cat urls.txt | parallel -j10 curl -O {}
```

---

## Monitoring

### watch (repeat command)
```bash
# Update every 2 seconds
watch 'ps aux | grep nginx'

# Update every 0.5 seconds
watch -n 0.5 'command'

# Highlight differences
watch -d 'command'
```

### top / htop
```bash
# Interactive process viewer
top
htop

# Batch mode (for scripts)
top -b -n1
```

---

## Installation

```bash
# Most tools pre-installed (procps-ng)

# GNU parallel
sudo apt install parallel

# htop
sudo apt install htop
```
```

---

# task-system.md

```markdown
---
description: "System information and administration - hardware info, services, logs, scheduling, and date/time operations"
when_to_use: "Use when: checking system specs, viewing memory/CPU usage, reading system logs, managing systemd services, scheduling cron jobs, working with dates and timestamps, checking hardware sensors. Triggers: system info, memory usage, cpu info, journalctl, systemctl, cron, logs, date, uptime, hostname."
when-to-use: "Use when: checking system specs, viewing memory/CPU usage, reading system logs, managing systemd services, scheduling cron jobs, working with dates and timestamps, checking hardware sensors. Triggers: system info, memory usage, cpu info, journalctl, systemctl, cron, logs, date, uptime, hostname."
argument-hint: "[describe what system information you need]"
context: fork
model: inherit
allowed-tools: Read
---

# System Information and Administration Tasks

Primary tools: **systemctl**, **journalctl**, **date**, **lscpu**, **free**

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| System info | `uname -a` |
| CPU info | `lscpu` |
| Memory usage | `free -h` |
| Disk usage | `df -h` |
| Uptime | `uptime` |
| Current user | `whoami` |
| Hostname | `hostname` |
| View service status | `systemctl status nginx` |
| View logs | `journalctl -u nginx` |
| Current date | `date` |
| Edit crontab | `crontab -e` |

---

## System Information

### Basic Info
```bash
# Kernel and OS
uname -a
uname -r               # Kernel version only

# Hostname
hostname
hostname -I            # IP addresses

# Uptime and load
uptime

# Current user
whoami
id                     # User and group IDs
```

### Hardware Info
```bash
# CPU
lscpu
cat /proc/cpuinfo

# Memory
free -h
cat /proc/meminfo

# Block devices
lsblk
lsblk -f               # With filesystem info

# PCI devices
lspci

# USB devices
lsusb

# All hardware (requires lshw)
sudo lshw -short
```

### Resource Usage
```bash
# Memory
free -h

# CPU and memory summary
vmstat 1 5             # Every 1 sec, 5 times

# I/O statistics
iostat -x 1 5

# Per-process resource usage
top -b -n1 | head -20
```

---

## Systemd Services

### Status and Control
```bash
# Check status
systemctl status nginx

# Start/stop/restart
systemctl start nginx
systemctl stop nginx
systemctl restart nginx

# Reload config (no restart)
systemctl reload nginx

# Enable/disable at boot
systemctl enable nginx
systemctl disable nginx

# List all services
systemctl list-units --type=service

# List failed services
systemctl --failed
```

### Service Information
```bash
# Show service file
systemctl cat nginx

# Show dependencies
systemctl list-dependencies nginx

# Show properties
systemctl show nginx
```

---

## Logs (journalctl)

### Basic Queries
```bash
# All logs
journalctl

# Follow (like tail -f)
journalctl -f

# Specific service
journalctl -u nginx

# Specific service, follow
journalctl -u nginx -f

# Since boot
journalctl -b

# Previous boot
journalctl -b -1
```

### Filtering
```bash
# By time
journalctl --since "1 hour ago"
journalctl --since "2024-01-15" --until "2024-01-16"
journalctl --since today

# By priority
journalctl -p err      # Errors and above
journalctl -p warning

# Kernel messages
journalctl -k
```

### Output Formats
```bash
# JSON output
journalctl -o json

# Short with timestamps
journalctl -o short-iso

# Reverse (newest first)
journalctl -r

# Last N lines
journalctl -n 50
```

---

## Date and Time

### Current Date/Time
```bash
# Current date/time
date

# Specific format
date +%Y-%m-%d
date +%H:%M:%S
date "+%Y-%m-%d %H:%M:%S"
date +%s               # Unix timestamp

# UTC
date -u
```

### Date Arithmetic (with dateutils)
```bash
# Add duration
dateadd now +1week
dateadd 2024-01-15 +30days

# Difference between dates
datediff 2024-01-01 2024-12-31

# Generate date sequence
dateseq 2024-01-01 2024-01-07
```

### Date Conversion
```bash
# Unix timestamp to date
date -d @1705276800

# String to timestamp
date -d "2024-01-15 10:00:00" +%s

# Relative dates
date -d "yesterday"
date -d "next monday"
date -d "+1 week"
```

### Timezone
```bash
# Current timezone
timedatectl

# List timezones
timedatectl list-timezones

# Set timezone
sudo timedatectl set-timezone America/New_York
```

---

## Scheduling (cron)

### Crontab
```bash
# Edit crontab
crontab -e

# List crontab
crontab -l

# Remove crontab
crontab -r
```

### Cron Syntax
```
# ┌───────────── minute (0-59)
# │ ┌───────────── hour (0-23)
# │ │ ┌───────────── day of month (1-31)
# │ │ │ ┌───────────── month (1-12)
# │ │ │ │ ┌───────────── day of week (0-6, Sunday=0)
# │ │ │ │ │
# * * * * * command

# Examples:
0 * * * *    # Every hour
0 0 * * *    # Daily at midnight
0 0 * * 0    # Weekly on Sunday
0 0 1 * *    # Monthly on 1st
*/5 * * * *  # Every 5 minutes
0 9-17 * * 1-5  # Hourly 9am-5pm weekdays
```

### One-time Jobs (at)
```bash
# Run at specific time
echo "command" | at 10:00

# Run in 1 hour
echo "command" | at now + 1 hour

# List pending jobs
atq

# Remove job
atrm 1
```

---

## Sensors and Monitoring

```bash
# Temperature sensors
sensors

# GPU (NVIDIA)
nvidia-smi

# Continuous monitoring
watch sensors
watch -n1 nvidia-smi
```

---

## Installation

```bash
# Most tools pre-installed

# dateutils
sudo apt install dateutils

# sensors
sudo apt install lm-sensors
sudo sensors-detect

# lshw
sudo apt install lshw
```
```

---

# task-network.md

```markdown
---
description: "Network diagnostics - connectivity testing, DNS queries, port scanning, traffic analysis, and firewall rules"
when_to_use: "Use when: testing network connectivity, resolving DNS names, checking open ports, analyzing network traffic, configuring IP addresses, debugging connection issues, setting firewall rules. Triggers: ping, traceroute, dns, dig, nmap, ports, netstat, ss, ip address, tcpdump, firewall, ssh."
when-to-use: "Use when: testing network connectivity, resolving DNS names, checking open ports, analyzing network traffic, configuring IP addresses, debugging connection issues, setting firewall rules. Triggers: ping, traceroute, dns, dig, nmap, ports, netstat, ss, ip address, tcpdump, firewall, ssh."
argument-hint: "[describe the network task]"
context: fork
model: inherit
allowed-tools: Read
---

# Network Diagnostics Tasks

Primary tools: **ping**, **dig**, **ss**, **ip**, **nmap**, **curl**

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Test connectivity | `ping -c 4 google.com` |
| Trace route | `traceroute google.com` |
| DNS lookup | `dig google.com` |
| Reverse DNS | `dig -x 8.8.8.8` |
| Show IP addresses | `ip addr` |
| Show routes | `ip route` |
| List open ports | `ss -tuln` |
| List connections | `ss -tun` |
| Scan ports | `nmap -p 1-1000 host` |
| Test port open | `nc -zv host 80` |

---

## Connectivity Testing

### ping
```bash
# Basic ping
ping google.com

# Limit count
ping -c 4 google.com

# Set interval
ping -i 0.5 google.com

# Quiet (just stats)
ping -c 4 -q google.com
```

### traceroute
```bash
# Trace route
traceroute google.com

# Use ICMP (may need sudo)
sudo traceroute -I google.com

# Faster (no DNS)
traceroute -n google.com
```

### mtr (combined ping + traceroute)
```bash
# Interactive
mtr google.com

# Report mode
mtr -r -c 10 google.com
```

---

## DNS

### dig
```bash
# Basic lookup
dig google.com

# Specific record type
dig google.com A
dig google.com MX
dig google.com TXT
dig google.com NS

# Short answer only
dig +short google.com

# Reverse lookup
dig -x 8.8.8.8

# Use specific DNS server
dig @8.8.8.8 google.com

# Trace delegation
dig +trace google.com
```

### nslookup / host
```bash
# Simple lookup
nslookup google.com
host google.com

# Reverse lookup
host 8.8.8.8
```

### whois
```bash
# Domain info
whois google.com

# IP info
whois 8.8.8.8
```

---

## IP Configuration

### ip (modern)
```bash
# Show addresses
ip addr
ip a                   # Short form

# Show specific interface
ip addr show eth0

# Show routes
ip route
ip r                   # Short form

# Show neighbors (ARP)
ip neigh

# Add/remove IP (temporary)
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr del 192.168.1.100/24 dev eth0
```

### ifconfig (legacy)
```bash
# Show all interfaces
ifconfig

# Show specific interface
ifconfig eth0
```

---

## Ports and Connections

### ss (socket statistics - modern)
```bash
# Listening ports
ss -tuln

# All connections
ss -tun

# With process info
ss -tulnp

# Filter by port
ss -tuln | grep :80

# TCP connections
ss -t

# UDP connections
ss -u
```

### netstat (legacy)
```bash
# Listening ports
netstat -tuln

# All connections with process
netstat -tunp
```

### lsof (network)
```bash
# What's using port 80
lsof -i :80

# All network connections
lsof -i

# Specific protocol
lsof -i tcp
```

---

## Port Scanning and Testing

### nmap
```bash
# Scan common ports
nmap host

# Scan specific ports
nmap -p 22,80,443 host

# Scan port range
nmap -p 1-1000 host

# Scan all ports
nmap -p- host

# Service detection
nmap -sV host

# OS detection
sudo nmap -O host

# Quick scan
nmap -F host
```

### nc (netcat)
```bash
# Test if port is open
nc -zv host 80

# Scan port range
nc -zv host 1-100

# Listen on port
nc -l 8080

# Send data
echo "test" | nc host 80
```

---

## Traffic Analysis

### tcpdump
```bash
# Capture all traffic
sudo tcpdump

# Specific interface
sudo tcpdump -i eth0

# Specific port
sudo tcpdump port 80

# Specific host
sudo tcpdump host 192.168.1.1

# Save to file
sudo tcpdump -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Show packet contents
sudo tcpdump -A port 80
```

### tshark (Wireshark CLI)
```bash
# Capture traffic
tshark -i eth0

# Filter
tshark -i eth0 -f "port 80"

# Read pcap
tshark -r capture.pcap
```

---

## SSH

### Basic Usage
```bash
# Connect
ssh user@host

# Specific port
ssh -p 2222 user@host

# With key
ssh -i ~/.ssh/key.pem user@host

# Run command
ssh user@host 'command'
```

### SSH Tunnels
```bash
# Local port forward (access remote:3306 via localhost:3306)
ssh -L 3306:localhost:3306 user@host

# Remote port forward (expose local:8080 on remote:8080)
ssh -R 8080:localhost:8080 user@host

# SOCKS proxy
ssh -D 1080 user@host

# Jump host
ssh -J jumphost user@finalhost
```

### SSH Config (~/.ssh/config)
```
Host myserver
    HostName 192.168.1.100
    User admin
    Port 22
    IdentityFile ~/.ssh/mykey
```

---

## Firewall (iptables/nft)

### iptables
```bash
# List rules
sudo iptables -L -n

# Allow port
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Block IP
sudo iptables -A INPUT -s 10.0.0.1 -j DROP

# Save rules
sudo iptables-save > /etc/iptables.rules
```

### ufw (simpler)
```bash
# Enable
sudo ufw enable

# Allow port
sudo ufw allow 80/tcp

# Allow SSH
sudo ufw allow ssh

# Status
sudo ufw status
```

---

## Installation

```bash
# Most tools pre-installed

# Additional tools
sudo apt install nmap
sudo apt install mtr
sudo apt install whois
sudo apt install tcpdump
```
```

---

# task-permissions.md

```markdown
---
description: "Users and permissions - file permissions, ownership, user management, and access control"
when_to_use: "Use when: changing file permissions, setting ownership, creating users, managing groups, configuring sudo access, setting up ACLs, checking user identity. Triggers: chmod, chown, permissions, user, group, sudo, acl, ownership, 755, rwx, access denied."
when-to-use: "Use when: changing file permissions, setting ownership, creating users, managing groups, configuring sudo access, setting up ACLs, checking user identity. Triggers: chmod, chown, permissions, user, group, sudo, acl, ownership, 755, rwx, access denied."
argument-hint: "[describe the permission or user task]"
context: fork
model: inherit
allowed-tools: Read
---

# Users and Permissions Tasks

Primary tools: **chmod**, **chown**, **useradd**, **sudo**

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Make executable | `chmod +x script.sh` |
| Set permissions | `chmod 755 file` |
| Change owner | `chown user:group file` |
| Current user | `whoami` |
| User info | `id` |
| Add user | `sudo useradd -m username` |
| Set password | `sudo passwd username` |
| Add to group | `sudo usermod -aG groupname username` |

---

## File Permissions

### chmod (change mode)

#### Symbolic Mode
```bash
# Add execute
chmod +x script.sh

# Remove write for others
chmod o-w file

# Set read for all
chmod a+r file

# User=rwx, group=rx, others=rx
chmod u=rwx,g=rx,o=rx file
```

#### Numeric Mode
```
4 = read (r)
2 = write (w)
1 = execute (x)

Common patterns:
755 = rwxr-xr-x (executable)
644 = rw-r--r-- (regular file)
600 = rw------- (private)
700 = rwx------ (private dir)
777 = rwxrwxrwx (dangerous!)
```

```bash
chmod 755 script.sh
chmod 644 file.txt
chmod 600 secrets.txt
```

#### Recursive
```bash
chmod -R 755 directory/
chmod -R u+rwX,go+rX,go-w directory/  # Safe recursive
```

### chown (change owner)
```bash
# Change owner
chown user file

# Change owner and group
chown user:group file

# Recursive
chown -R user:group directory/

# Only change group
chgrp group file
```

---

## User Information

```bash
# Current user
whoami

# User ID and groups
id
id username

# Groups for user
groups
groups username

# Who is logged in
who
w

# User details
finger username  # If installed
getent passwd username
```

---

## User Management

### Create User
```bash
# Create with home directory
sudo useradd -m username

# Create with shell
sudo useradd -m -s /bin/bash username

# Create with groups
sudo useradd -m -G sudo,docker username

# Set password
sudo passwd username
```

### Modify User
```bash
# Add to group
sudo usermod -aG groupname username

# Change shell
sudo usermod -s /bin/zsh username

# Change home directory
sudo usermod -d /new/home username

# Lock account
sudo usermod -L username

# Unlock account
sudo usermod -U username
```

### Delete User
```bash
# Remove user
sudo userdel username

# Remove with home directory
sudo userdel -r username
```

---

## Group Management

```bash
# Create group
sudo groupadd groupname

# Delete group
sudo groupdel groupname

# Add user to group
sudo usermod -aG groupname username

# Remove user from group
sudo gpasswd -d username groupname

# List all groups
cat /etc/group
getent group
```

---

## Sudo

### Basic Usage
```bash
# Run as root
sudo command

# Run as different user
sudo -u username command

# Edit with sudo
sudo -e /etc/file  # Uses $EDITOR

# Root shell
sudo -i
sudo su -
```

### Sudoers (/etc/sudoers)
```bash
# Edit safely
sudo visudo

# Allow user full sudo
username ALL=(ALL:ALL) ALL

# Allow without password
username ALL=(ALL) NOPASSWD: ALL

# Allow specific command
username ALL=(ALL) /usr/bin/systemctl restart nginx
```

---

## Access Control Lists (ACL)

```bash
# View ACL
getfacl file

# Set ACL
setfacl -m u:username:rwx file

# Set for group
setfacl -m g:groupname:rx file

# Default ACL (for new files in dir)
setfacl -d -m u:username:rwx directory/

# Remove ACL
setfacl -x u:username file

# Remove all ACL
setfacl -b file
```

---

## Special Permissions

### Setuid/Setgid/Sticky
```bash
# Setuid (run as owner)
chmod u+s file
chmod 4755 file

# Setgid (run as group / inherit group in dir)
chmod g+s directory/
chmod 2755 directory/

# Sticky bit (only owner can delete in dir)
chmod +t directory/
chmod 1777 directory/
```

### umask (default permissions)
```bash
# View current umask
umask

# Set umask (subtract from 777/666)
umask 022  # Files: 644, Dirs: 755
umask 077  # Files: 600, Dirs: 700
```

---

## Common Patterns

### Fix permission issues
```bash
# Web server files
sudo chown -R www-data:www-data /var/www/
sudo chmod -R 755 /var/www/
sudo chmod -R 644 /var/www/**/*.html

# SSH keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 644 ~/.ssh/authorized_keys
```

### Find permission problems
```bash
# Find world-writable files
find / -perm -002 -type f 2>/dev/null

# Find setuid files
find / -perm -4000 -type f 2>/dev/null

# Find files owned by user
find / -user username 2>/dev/null
```
```

---

# task-git.md

```markdown
---
description: "Version control - Git operations, GitHub/GitLab CLI, diffs, history, and repository management"
when_to_use: "Use when: committing changes, branching, merging, viewing git history, creating pull requests, managing GitHub issues, comparing commits, cleaning up branches, resolving conflicts. Triggers: git, commit, branch, merge, pull request, gh, github, gitlab, diff, history, rebase, stash."
when-to-use: "Use when: committing changes, branching, merging, viewing git history, creating pull requests, managing GitHub issues, comparing commits, cleaning up branches, resolving conflicts. Triggers: git, commit, branch, merge, pull request, gh, github, gitlab, diff, history, rebase, stash."
argument-hint: "[describe the git operation]"
context: fork
model: inherit
allowed-tools: Read
---

# Version Control Tasks

Primary tools: **git**, **gh** (GitHub CLI)

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Clone repo | `git clone url` |
| Status | `git status` |
| Add files | `git add file` or `git add .` |
| Commit | `git commit -m "message"` |
| Push | `git push` |
| Pull | `git pull` |
| Create branch | `git checkout -b branch-name` |
| Switch branch | `git checkout branch-name` |
| Merge branch | `git merge branch-name` |
| View log | `git log --oneline` |
| Create PR | `gh pr create` |

---

## Basic Operations

### Setup
```bash
# Configure user
git config --global user.name "Name"
git config --global user.email "email@example.com"

# Init new repo
git init

# Clone existing
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git
```

### Daily Workflow
```bash
# Check status
git status

# Stage changes
git add file.txt
git add .                    # All changes
git add -p                   # Interactive staging

# Commit
git commit -m "message"
git commit -am "message"     # Add + commit tracked files

# Push
git push
git push -u origin branch    # First push of branch

# Pull
git pull
git pull --rebase            # Rebase instead of merge
```

---

## Branches

### Create and Switch
```bash
# Create and switch
git checkout -b feature-branch

# Just create
git branch feature-branch

# Switch
git checkout main
git switch main              # Modern alternative

# List branches
git branch                   # Local
git branch -r                # Remote
git branch -a                # All
```

### Merge and Delete
```bash
# Merge branch into current
git merge feature-branch

# Delete local branch
git branch -d feature-branch
git branch -D feature-branch  # Force delete

# Delete remote branch
git push origin --delete feature-branch
```

---

## History and Diffs

### View Log
```bash
# Basic log
git log

# One line per commit
git log --oneline

# With graph
git log --oneline --graph --all

# Last N commits
git log -5

# By author
git log --author="name"

# By date
git log --since="2024-01-01" --until="2024-01-31"

# Show changes
git log -p
```

### View Diffs
```bash
# Working dir vs staged
git diff

# Staged vs last commit
git diff --staged

# Between commits
git diff abc123 def456

# Between branches
git diff main..feature-branch

# Specific file
git diff file.txt
```

### Blame
```bash
# Who changed each line
git blame file.txt

# Specific lines
git blame -L 10,20 file.txt
```

---

## Undoing Changes

### Unstage
```bash
# Unstage file
git reset HEAD file.txt
git restore --staged file.txt  # Modern
```

### Discard Changes
```bash
# Discard working dir changes
git checkout -- file.txt
git restore file.txt           # Modern

# Discard all changes (dangerous!)
git checkout -- .
git restore .
```

### Amend Last Commit
```bash
# Change message
git commit --amend -m "new message"

# Add files to last commit
git add forgotten-file.txt
git commit --amend --no-edit
```

### Revert Commit
```bash
# Create new commit that undoes changes
git revert abc123

# Revert without committing
git revert --no-commit abc123
```

### Reset (dangerous)
```bash
# Soft reset (keep changes staged)
git reset --soft HEAD~1

# Mixed reset (keep changes unstaged)
git reset HEAD~1

# Hard reset (discard changes)
git reset --hard HEAD~1
```

---

## Stash

```bash
# Stash changes
git stash

# Stash with message
git stash push -m "description"

# List stashes
git stash list

# Apply latest stash
git stash pop

# Apply specific stash
git stash apply stash@{0}

# Drop stash
git stash drop stash@{0}

# Clear all stashes
git stash clear
```

---

## Remote Operations

```bash
# List remotes
git remote -v

# Add remote
git remote add origin url

# Change remote URL
git remote set-url origin new-url

# Fetch without merge
git fetch

# Fetch and prune deleted branches
git fetch --prune
```

---

## GitHub CLI (gh)

### Pull Requests
```bash
# Create PR
gh pr create
gh pr create --title "Title" --body "Description"
gh pr create --draft

# List PRs
gh pr list

# View PR
gh pr view 123

# Checkout PR locally
gh pr checkout 123

# Merge PR
gh pr merge 123
```

### Issues
```bash
# Create issue
gh issue create

# List issues
gh issue list

# View issue
gh issue view 123

# Close issue
gh issue close 123
```

### Repo Operations
```bash
# Clone
gh repo clone owner/repo

# Create repo
gh repo create repo-name --public
gh repo create repo-name --private

# Fork
gh repo fork owner/repo
```

---

## Common Patterns

### Feature Branch Workflow
```bash
git checkout main
git pull
git checkout -b feature-xyz
# ... make changes ...
git add .
git commit -m "Add feature XYZ"
git push -u origin feature-xyz
gh pr create
```

### Sync Fork with Upstream
```bash
git remote add upstream https://github.com/original/repo.git
git fetch upstream
git checkout main
git merge upstream/main
git push
```

### Interactive Rebase
```bash
# Rebase last 3 commits
git rebase -i HEAD~3

# In editor, change 'pick' to:
# - squash (s): combine with previous
# - reword (r): change message
# - drop (d): remove commit
```

---

## Installation

```bash
# Git is usually pre-installed

# GitHub CLI
sudo apt install gh
gh auth login
```
```

---

# task-dev.md

```markdown
---
description: "Development tools - build systems, compilers, linters, formatters, and language toolchains"
when_to_use: "Use when: building projects with make/cmake, compiling code, linting shell scripts, formatting code, managing Python/Node environments, running linters, checking types. Triggers: make, build, compile, lint, format, shellcheck, eslint, ruff, prettier, gcc, npm, pip, python, node."
when-to-use: "Use when: building projects with make/cmake, compiling code, linting shell scripts, formatting code, managing Python/Node environments, running linters, checking types. Triggers: make, build, compile, lint, format, shellcheck, eslint, ruff, prettier, gcc, npm, pip, python, node."
argument-hint: "[describe the development task]"
context: fork
model: inherit
allowed-tools: Read
---

# Development Tools Tasks

Primary tools: **make**, **shellcheck**, **ruff**, **prettier**

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Build with make | `make` or `make target` |
| Lint shell script | `shellcheck script.sh` |
| Lint Python | `ruff check .` |
| Format Python | `ruff format .` or `black .` |
| Format JS/TS | `prettier --write .` |
| Type check Python | `pyright` or `mypy .` |
| Lint JS/TS | `eslint .` |
| Install Python deps | `pip install -r requirements.txt` |
| Install Node deps | `npm install` |

---

## Build Systems

### make
```bash
# Run default target
make

# Run specific target
make build
make test
make clean

# Parallel build
make -j4

# Dry run (show commands)
make -n

# Debug (show why targets rebuilt)
make -d
```

### cmake
```bash
# Configure
mkdir build && cd build
cmake ..

# Configure with options
cmake -DCMAKE_BUILD_TYPE=Release ..

# Build
cmake --build .

# Build with parallelism
cmake --build . -j4
```

### just (simpler alternative)
```bash
# List recipes
just --list

# Run recipe
just build
just test
```

---

## Shell Script Linting

### shellcheck
```bash
# Check script
shellcheck script.sh

# Check all scripts
shellcheck *.sh

# Specific shell dialect
shellcheck -s bash script.sh

# Exclude rules
shellcheck -e SC2034 script.sh

# Output format
shellcheck -f json script.sh
```

### shfmt (formatting)
```bash
# Format script
shfmt -w script.sh

# Check without modifying
shfmt -d script.sh

# Indent with 4 spaces
shfmt -i 4 -w script.sh
```

---

## Python Tools

### Linting
```bash
# ruff (fast, recommended)
ruff check .
ruff check --fix .        # Auto-fix

# pylint
pylint module.py

# flake8
flake8 .
```

### Formatting
```bash
# ruff format (recommended)
ruff format .

# black
black .
black --check .           # Check only

# isort (imports)
isort .
```

### Type Checking
```bash
# pyright (fast)
pyright

# mypy
mypy .
mypy --strict .
```

### Package Management
```bash
# pip
pip install package
pip install -r requirements.txt
pip freeze > requirements.txt

# uv (fast alternative)
uv pip install package
uv pip install -r requirements.txt

# poetry
poetry install
poetry add package
poetry run python script.py

# Virtual environments
python -m venv .venv
source .venv/bin/activate
```

---

## JavaScript/TypeScript Tools

### Linting
```bash
# eslint
eslint .
eslint --fix .

# Check config
eslint --print-config file.js
```

### Formatting
```bash
# prettier
prettier --write .
prettier --check .

# Specific files
prettier --write "src/**/*.{js,ts,jsx,tsx}"
```

### Type Checking
```bash
# TypeScript compiler
tsc --noEmit              # Check only
tsc                       # Compile

# With project
tsc -p tsconfig.json
```

### Package Management
```bash
# npm
npm install
npm install package
npm install -D package    # Dev dependency
npm run script

# pnpm (faster)
pnpm install
pnpm add package

# yarn
yarn install
yarn add package
```

### Running
```bash
# Node
node script.js

# TypeScript (direct)
npx tsx script.ts
npx ts-node script.ts

# Bun
bun script.ts
```

---

## Compilers

### GCC/G++
```bash
# Compile C
gcc -o output source.c
gcc -Wall -O2 -o output source.c

# Compile C++
g++ -o output source.cpp
g++ -std=c++17 -Wall -O2 -o output source.cpp

# With libraries
gcc -o output source.c -lm -lpthread
```

### Clang
```bash
clang -o output source.c
clang++ -o output source.cpp
```

---

## Security Scanning

```bash
# Python security
bandit -r .
safety check

# Secrets detection
gitleaks detect

# Dockerfile linting
hadolint Dockerfile

# GitHub Actions
actionlint
```

---

## Docker

```bash
# Build image
docker build -t image-name .

# Run container
docker run -it image-name

# With volume
docker run -v $(pwd):/app image-name

# Compose
docker-compose up
docker-compose up -d      # Detached
docker-compose down
```

---

## Installation

```bash
# Shell tools
sudo apt install shellcheck shfmt

# Python tools
pip install ruff black mypy pyright bandit

# Node tools
npm install -g eslint prettier typescript

# Build tools
sudo apt install build-essential cmake
```
```

---

# task-crypto.md

```markdown
---
description: "Cryptography and encoding - hashing, checksums, encryption, key management, and base64 encoding"
when_to_use: "Use when: computing file checksums, encoding/decoding base64, encrypting files, generating SSH keys, managing GPG keys, creating certificates, hashing passwords, verifying file integrity. Triggers: hash, md5, sha256, checksum, base64, encrypt, gpg, ssh key, certificate, openssl."
when-to-use: "Use when: computing file checksums, encoding/decoding base64, encrypting files, generating SSH keys, managing GPG keys, creating certificates, hashing passwords, verifying file integrity. Triggers: hash, md5, sha256, checksum, base64, encrypt, gpg, ssh key, certificate, openssl."
argument-hint: "[describe the cryptography task]"
context: fork
model: inherit
allowed-tools: Read
---

# Cryptography and Encoding Tasks

Primary tools: **sha256sum**, **base64**, **openssl**, **gpg**, **ssh-keygen**

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| SHA256 hash | `sha256sum file` |
| MD5 hash | `md5sum file` |
| Verify checksum | `sha256sum -c checksums.txt` |
| Base64 encode | `base64 file` |
| Base64 decode | `base64 -d file` |
| Generate SSH key | `ssh-keygen -t ed25519` |
| Encrypt with GPG | `gpg -c file` |
| Decrypt with GPG | `gpg -d file.gpg` |

---

## Checksums and Hashing

### SHA256 (recommended)
```bash
# Hash file
sha256sum file.txt

# Hash multiple files
sha256sum *.txt

# Create checksum file
sha256sum file.txt > checksums.txt

# Verify checksums
sha256sum -c checksums.txt

# Hash string
echo -n "text" | sha256sum
```

### Other Hash Algorithms
```bash
# SHA512 (more secure)
sha512sum file.txt

# SHA1 (legacy, avoid for security)
sha1sum file.txt

# MD5 (legacy, avoid for security)
md5sum file.txt

# BLAKE2 (fast)
b2sum file.txt
```

### Hash String
```bash
# Note: -n prevents trailing newline
echo -n "password" | sha256sum
echo -n "password" | md5sum
```

---

## Base64 Encoding

### Encode
```bash
# Encode file
base64 file.txt
base64 file.txt > encoded.txt

# Encode string
echo -n "text" | base64

# Encode without line wrapping
base64 -w 0 file.txt
```

### Decode
```bash
# Decode file
base64 -d encoded.txt

# Decode string
echo "dGV4dA==" | base64 -d
```

---

## Hex Encoding

### xxd
```bash
# File to hex
xxd file.txt

# Hex to binary
xxd -r hexdump.txt > file.bin

# Plain hex (no formatting)
xxd -p file.txt

# Reverse plain hex
xxd -r -p hex.txt > file.bin
```

### hexdump
```bash
hexdump -C file.txt
```

---

## SSH Keys

### Generate Keys
```bash
# Ed25519 (recommended)
ssh-keygen -t ed25519 -C "email@example.com"

# RSA (wider compatibility)
ssh-keygen -t rsa -b 4096 -C "email@example.com"

# With custom filename
ssh-keygen -t ed25519 -f ~/.ssh/custom_key
```

### Manage Keys
```bash
# List keys in agent
ssh-add -l

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Copy public key to server
ssh-copy-id user@host

# View public key
cat ~/.ssh/id_ed25519.pub
```

### Key Fingerprint
```bash
# Show fingerprint
ssh-keygen -lf ~/.ssh/id_ed25519.pub

# Show fingerprint of remote host
ssh-keyscan github.com | ssh-keygen -lf -
```

---

## GPG Encryption

### Symmetric (password-based)
```bash
# Encrypt
gpg -c file.txt                # Creates file.txt.gpg

# Decrypt
gpg -d file.txt.gpg > file.txt
gpg file.txt.gpg               # Interactive
```

### Asymmetric (key-based)
```bash
# Generate key pair
gpg --full-generate-key

# List keys
gpg --list-keys
gpg --list-secret-keys

# Export public key
gpg --export -a "Name" > public.key

# Import public key
gpg --import public.key

# Encrypt for recipient
gpg -e -r "recipient@email.com" file.txt

# Decrypt
gpg -d file.txt.gpg
```

### Signing
```bash
# Sign file
gpg --sign file.txt

# Verify signature
gpg --verify file.txt.gpg

# Clearsign (readable)
gpg --clearsign file.txt
```

---

## OpenSSL

### Random Data
```bash
# Random bytes (hex)
openssl rand -hex 32

# Random bytes (base64)
openssl rand -base64 32

# Random bytes (binary)
openssl rand -out random.bin 32
```

### Symmetric Encryption
```bash
# Encrypt
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc

# Decrypt
openssl enc -d -aes-256-cbc -in file.enc -out file.txt

# With explicit password
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc -pass pass:mypassword
```

### Certificates
```bash
# Generate self-signed cert
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

# View certificate
openssl x509 -in cert.pem -text -noout

# Check certificate expiry
openssl x509 -in cert.pem -noout -dates

# Generate CSR
openssl req -new -newkey rsa:4096 -nodes -keyout key.pem -out request.csr
```

### Hash with OpenSSL
```bash
openssl dgst -sha256 file.txt
openssl dgst -md5 file.txt
```

---

## Age (Modern Encryption)

```bash
# Generate key
age-keygen -o key.txt

# Encrypt with public key
age -r age1... -o file.enc file.txt

# Encrypt with passphrase
age -p -o file.enc file.txt

# Decrypt
age -d -i key.txt -o file.txt file.enc
```

---

## Password Hashing

```bash
# Generate bcrypt hash (requires mkpasswd)
mkpasswd -m bcrypt

# Generate SHA512 hash
mkpasswd -m sha-512

# Using openssl
openssl passwd -6  # SHA512
openssl passwd -5  # SHA256
```

---

## Installation

```bash
# Most tools pre-installed

# Age (modern encryption)
sudo apt install age

# Additional utilities
sudo apt install gnupg2
```
```

---

# task-database.md

```markdown
---
description: "Database operations - SQL queries, SQLite management, Redis commands, and data import/export"
when_to_use: "Use when: running SQL queries, managing SQLite databases, importing CSV to database, querying with DuckDB, using Redis for caching, exporting data to files. Triggers: sqlite, sql query, database, psql, postgres, mysql, redis, duckdb, import csv, export data."
when-to-use: "Use when: running SQL queries, managing SQLite databases, importing CSV to database, querying with DuckDB, using Redis for caching, exporting data to files. Triggers: sqlite, sql query, database, psql, postgres, mysql, redis, duckdb, import csv, export data."
argument-hint: "[describe the database task]"
context: fork
model: inherit
allowed-tools: Read
---

# Database Operations Tasks

Primary tools: **sqlite3**, **psql**, **duckdb**, **redis-cli**

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Open SQLite DB | `sqlite3 database.db` |
| Run SQL query | `sqlite3 db.db "SELECT * FROM table"` |
| Import CSV to SQLite | `sqlite3 db.db ".import file.csv table"` |
| Export to CSV | `sqlite3 -csv db.db "SELECT * FROM t" > out.csv` |
| Query CSV with DuckDB | `duckdb -c "SELECT * FROM 'file.csv'"` |
| Connect to Postgres | `psql -h host -U user -d database` |
| Redis get/set | `redis-cli GET key` / `redis-cli SET key value` |

---

## SQLite

### Basic Usage
```bash
# Open database (creates if not exists)
sqlite3 database.db

# Run query directly
sqlite3 database.db "SELECT * FROM users"

# Run SQL file
sqlite3 database.db < script.sql

# Interactive mode
sqlite3 database.db
sqlite> .tables
sqlite> .schema tablename
sqlite> SELECT * FROM users;
sqlite> .quit
```

### Import/Export
```bash
# Import CSV
sqlite3 database.db
sqlite> .mode csv
sqlite> .import data.csv tablename

# One-liner import
sqlite3 database.db ".mode csv" ".import data.csv tablename"

# Export to CSV
sqlite3 -csv database.db "SELECT * FROM users" > users.csv

# Export with headers
sqlite3 -header -csv database.db "SELECT * FROM users" > users.csv

# Dump entire database
sqlite3 database.db .dump > backup.sql

# Restore from dump
sqlite3 newdb.db < backup.sql
```

### Useful Commands
```bash
# In SQLite shell:
.tables              # List tables
.schema tablename    # Show table schema
.mode column         # Column output
.headers on          # Show headers
.output file.txt     # Output to file
.read script.sql     # Run SQL file
```

---

## sqlite-utils

```bash
# Insert JSON
sqlite-utils insert database.db tablename data.json

# Insert CSV
sqlite-utils insert database.db tablename data.csv --csv

# Query with SQL
sqlite-utils database.db "SELECT * FROM tablename" --csv

# Create table from CSV
sqlite-utils insert database.db newtable data.csv --csv --detect-types

# Export table
sqlite-utils rows database.db tablename --csv > output.csv
```

---

## DuckDB

### Query Files Directly
```bash
# Query CSV
duckdb -c "SELECT * FROM 'data.csv'"

# Query Parquet
duckdb -c "SELECT * FROM 'data.parquet'"

# Query JSON
duckdb -c "SELECT * FROM read_json_auto('data.json')"

# Aggregate
duckdb -c "SELECT category, COUNT(*) FROM 'sales.csv' GROUP BY category"
```

### Save Results
```bash
# To CSV
duckdb -c "COPY (SELECT * FROM 'data.csv') TO 'output.csv'"

# To Parquet
duckdb -c "COPY (SELECT * FROM 'data.csv') TO 'output.parquet'"
```

### Interactive
```bash
duckdb mydb.duckdb
D SELECT * FROM 'data.csv' LIMIT 10;
D .quit
```

---

## PostgreSQL (psql)

### Connect
```bash
# Basic connection
psql -h localhost -U username -d database

# With password prompt
psql -h localhost -U username -d database -W

# Connection string
psql "postgresql://user:pass@host:5432/database"
```

### Run Queries
```bash
# Run query
psql -h host -U user -d db -c "SELECT * FROM users"

# Run SQL file
psql -h host -U user -d db -f script.sql

# Output to CSV
psql -h host -U user -d db -c "COPY (SELECT * FROM users) TO STDOUT CSV HEADER" > users.csv
```

### Interactive Commands
```bash
# In psql shell:
\l              # List databases
\dt             # List tables
\d tablename    # Describe table
\du             # List users
\q              # Quit
\i script.sql   # Run SQL file
\copy table TO 'file.csv' CSV HEADER  # Export
```

---

## MySQL

### Connect
```bash
mysql -h localhost -u username -p database
```

### Run Queries
```bash
# Run query
mysql -h host -u user -p -e "SELECT * FROM users" database

# Run SQL file
mysql -h host -u user -p database < script.sql

# Export to CSV
mysql -h host -u user -p -e "SELECT * FROM users" database | tr '\t' ',' > users.csv
```

---

## Redis

### Basic Operations
```bash
# Connect
redis-cli
redis-cli -h host -p 6379

# Get/Set
redis-cli SET key "value"
redis-cli GET key

# Delete
redis-cli DEL key

# List keys
redis-cli KEYS "*"
redis-cli KEYS "user:*"

# Check existence
redis-cli EXISTS key

# Set with expiry
redis-cli SETEX key 3600 "value"  # 1 hour
```

### Data Types
```bash
# Hash
redis-cli HSET user:1 name "John" email "john@example.com"
redis-cli HGET user:1 name
redis-cli HGETALL user:1

# List
redis-cli LPUSH queue "item1"
redis-cli RPOP queue

# Set
redis-cli SADD tags "tag1" "tag2"
redis-cli SMEMBERS tags
```

### Monitoring
```bash
# Server info
redis-cli INFO

# Monitor commands in real-time
redis-cli MONITOR

# Memory usage
redis-cli MEMORY USAGE key
```

---

## Installation

```bash
# SQLite (usually pre-installed)
sudo apt install sqlite3

# sqlite-utils
pip install sqlite-utils

# DuckDB
pip install duckdb

# PostgreSQL client
sudo apt install postgresql-client

# Redis client
sudo apt install redis-tools
```
```

---

# task-containers.md

```markdown
---
description: "Containers and environments - Docker operations, terminal multiplexing, namespace isolation, and environment management"
when_to_use: "Use when: running Docker containers, using docker-compose, managing tmux sessions, isolating processes, setting up development environments, inspecting container images. Triggers: docker, container, docker-compose, tmux, screen, namespace, devcontainer, image, volume."
when-to-use: "Use when: running Docker containers, using docker-compose, managing tmux sessions, isolating processes, setting up development environments, inspecting container images. Triggers: docker, container, docker-compose, tmux, screen, namespace, devcontainer, image, volume."
argument-hint: "[describe the container or environment task]"
context: fork
model: inherit
allowed-tools: Read
---

# Containers and Environments Tasks

Primary tools: **docker**, **docker-compose**, **tmux**, **screen**

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Run container | `docker run -it ubuntu bash` |
| List containers | `docker ps` |
| Stop container | `docker stop container_id` |
| Build image | `docker build -t name .` |
| Compose up | `docker-compose up -d` |
| Compose down | `docker-compose down` |
| New tmux session | `tmux new -s name` |
| Attach tmux | `tmux attach -t name` |
| Detach tmux | `Ctrl-b d` |

---

## Docker

### Running Containers
```bash
# Run interactive
docker run -it ubuntu bash

# Run detached
docker run -d nginx

# Run with name
docker run -d --name myapp nginx

# Run with port mapping
docker run -d -p 8080:80 nginx

# Run with volume
docker run -d -v $(pwd):/app nginx
docker run -d -v myvolume:/data nginx

# Run with environment variables
docker run -d -e MY_VAR=value nginx

# Run and remove when done
docker run --rm -it ubuntu bash
```

### Managing Containers
```bash
# List running containers
docker ps

# List all containers
docker ps -a

# Stop container
docker stop container_id

# Start container
docker start container_id

# Restart container
docker restart container_id

# Remove container
docker rm container_id

# Remove all stopped containers
docker container prune

# View logs
docker logs container_id
docker logs -f container_id    # Follow

# Execute command in running container
docker exec -it container_id bash
docker exec container_id ls /app
```

### Images
```bash
# List images
docker images

# Pull image
docker pull nginx:latest

# Build image
docker build -t myapp .
docker build -t myapp:v1.0 .

# Build with different Dockerfile
docker build -f Dockerfile.prod -t myapp .

# Remove image
docker rmi image_id

# Remove unused images
docker image prune
```

### Volumes
```bash
# Create volume
docker volume create myvolume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect myvolume

# Remove volume
docker volume rm myvolume

# Remove unused volumes
docker volume prune
```

### Networks
```bash
# List networks
docker network ls

# Create network
docker network create mynetwork

# Run container in network
docker run -d --network mynetwork nginx

# Connect container to network
docker network connect mynetwork container_id
```

---

## Docker Compose

### Basic Commands
```bash
# Start services
docker-compose up
docker-compose up -d              # Detached

# Stop services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# Rebuild images
docker-compose up --build

# View logs
docker-compose logs
docker-compose logs -f service_name

# List services
docker-compose ps

# Execute command in service
docker-compose exec service_name bash

# Scale service
docker-compose up -d --scale web=3
```

### Example docker-compose.yml
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8080:80"
    volumes:
      - .:/app
    environment:
      - DEBUG=true
    depends_on:
      - db

  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=secret

volumes:
  postgres_data:
```

---

## tmux

### Sessions
```bash
# New session
tmux
tmux new -s session_name

# List sessions
tmux ls

# Attach to session
tmux attach -t session_name
tmux a -t session_name

# Detach from session
# Press: Ctrl-b d

# Kill session
tmux kill-session -t session_name
```

### Key Bindings (Ctrl-b prefix)
```
Ctrl-b d     Detach
Ctrl-b c     New window
Ctrl-b n     Next window
Ctrl-b p     Previous window
Ctrl-b 0-9   Go to window N
Ctrl-b %     Split vertically
Ctrl-b "     Split horizontally
Ctrl-b arrow Move between panes
Ctrl-b x     Kill pane
Ctrl-b [     Scroll mode (q to exit)
```

### Windows and Panes
```bash
# From command line
tmux new-window -t session_name
tmux split-window -h    # Horizontal split
tmux split-window -v    # Vertical split

# Rename window
# Press: Ctrl-b ,
```

---

## screen

### Basic Usage
```bash
# New session
screen
screen -S session_name

# List sessions
screen -ls

# Attach to session
screen -r session_name

# Detach
# Press: Ctrl-a d

# Kill session
screen -X -S session_name quit
```

### Key Bindings (Ctrl-a prefix)
```
Ctrl-a d     Detach
Ctrl-a c     New window
Ctrl-a n     Next window
Ctrl-a p     Previous window
Ctrl-a "     List windows
Ctrl-a k     Kill window
Ctrl-a [     Scroll mode
```

---

## Environment Management

### direnv
```bash
# Create .envrc in project
echo 'export MY_VAR=value' > .envrc

# Allow direnv
direnv allow

# Variables automatically loaded when entering directory
```

### mise (version manager)
```bash
# Install tool version
mise install python@3.11
mise install node@20

# Use in project
mise use python@3.11
mise use node@20

# List installed
mise list
```

---

## Installation

```bash
# Docker
sudo apt install docker.io docker-compose

# tmux
sudo apt install tmux

# screen
sudo apt install screen

# direnv
sudo apt install direnv

# mise
curl https://mise.run | sh
```
```

---

# task-debug.md

```markdown
---
description: "Debugging and profiling - system call tracing, performance profiling, memory analysis, and benchmarking"
when_to_use: "Use when: tracing system calls, profiling CPU usage, finding memory leaks, benchmarking commands, debugging process hangs, analyzing performance bottlenecks, understanding program behavior. Triggers: strace, trace, profile, perf, valgrind, memory leak, benchmark, gdb, debug, flamegraph, performance."
when-to-use: "Use when: tracing system calls, profiling CPU usage, finding memory leaks, benchmarking commands, debugging process hangs, analyzing performance bottlenecks, understanding program behavior. Triggers: strace, trace, profile, perf, valgrind, memory leak, benchmark, gdb, debug, flamegraph, performance."
argument-hint: "[describe what you want to debug or profile]"
context: fork
model: inherit
allowed-tools: Read
---

# Debugging and Profiling Tasks

Primary tools: **strace**, **perf**, **valgrind**, **hyperfine**, **gdb**

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Trace system calls | `strace command` |
| Trace running process | `strace -p PID` |
| Profile CPU | `perf record command && perf report` |
| Check memory leaks | `valgrind --leak-check=yes command` |
| Benchmark command | `hyperfine 'command'` |
| Time command | `time command` |
| Debug with gdb | `gdb ./program` |

---

## System Call Tracing

### strace
```bash
# Trace command
strace command

# Trace specific syscalls
strace -e open,read,write command

# Trace file operations
strace -e trace=file command

# Trace network operations
strace -e trace=network command

# Trace running process
strace -p 1234

# With timestamps
strace -t command
strace -tt command         # Microseconds

# Count syscalls
strace -c command

# Output to file
strace -o trace.log command

# Follow child processes
strace -f command
```

### ltrace (library calls)
```bash
# Trace library calls
ltrace command

# Specific functions
ltrace -e malloc+free command
```

---

## Performance Profiling

### perf
```bash
# Record performance data
perf record command
perf record -g command     # With call graph

# Analyze recording
perf report

# Real-time CPU usage by function
perf top

# System-wide profiling
sudo perf record -a -g sleep 10
sudo perf report

# Count specific events
perf stat command
perf stat -e cache-misses,cache-references command
```

### Flame Graphs
```bash
# Record with perf
perf record -F 99 -g command

# Generate flame graph
perf script | stackcollapse-perf.pl | flamegraph.pl > flame.svg
```

### time
```bash
# Basic timing
time command

# More detail
/usr/bin/time -v command
```

### hyperfine (statistical benchmarking)
```bash
# Benchmark single command
hyperfine 'command'

# Compare commands
hyperfine 'command1' 'command2'

# With warmup
hyperfine --warmup 3 'command'

# Minimum runs
hyperfine --min-runs 20 'command'

# Export results
hyperfine --export-json results.json 'command'
hyperfine --export-markdown results.md 'command'
```

---

## Memory Analysis

### valgrind
```bash
# Memory leak check
valgrind --leak-check=yes ./program

# Full leak check
valgrind --leak-check=full --show-leak-kinds=all ./program

# Track origins of uninitialized values
valgrind --track-origins=yes ./program
```

### massif (heap profiler)
```bash
# Profile heap usage
valgrind --tool=massif ./program

# Visualize results
ms_print massif.out.*
```

### heaptrack
```bash
# Profile allocations
heaptrack ./program

# Analyze
heaptrack --analyze heaptrack.*.gz
```

---

## Python Profiling

### py-spy
```bash
# Profile running process
py-spy record -o profile.svg --pid 1234

# Profile script
py-spy record -o profile.svg -- python script.py

# Top-like view
py-spy top --pid 1234
```

### scalene
```bash
# Profile with CPU, memory, and GPU
scalene script.py

# Output to file
scalene --outfile profile.html script.py
```

### memray
```bash
# Profile memory
memray run script.py

# Generate flame graph
memray flamegraph memray-*.bin
```

---

## GDB Debugging

### Basic Usage
```bash
# Start debugging
gdb ./program

# Run program
(gdb) run
(gdb) run arg1 arg2

# Set breakpoint
(gdb) break main
(gdb) break file.c:42
(gdb) break function_name

# Step through code
(gdb) next          # Next line (step over)
(gdb) step          # Step into
(gdb) continue      # Continue to next breakpoint

# Print variables
(gdb) print variable
(gdb) print *pointer

# Backtrace
(gdb) backtrace
(gdb) bt

# Quit
(gdb) quit
```

### Attach to Process
```bash
gdb -p 1234
gdb ./program 1234
```

### Core Dumps
```bash
# Enable core dumps
ulimit -c unlimited

# Debug core dump
gdb ./program core
```

---

## Node.js Profiling

### clinic
```bash
# Doctor (overall health)
clinic doctor -- node app.js

# Flame (CPU)
clinic flame -- node app.js

# Bubbleprof (async)
clinic bubbleprof -- node app.js
```

### 0x (flame graphs)
```bash
0x app.js
# Opens flame graph in browser
```

---

## eBPF Tracing

### bpftrace
```bash
# Trace open syscalls
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_open { printf("%s\n", str(args->filename)); }'

# Count syscalls by process
sudo bpftrace -e 'tracepoint:raw_syscalls:sys_enter { @[comm] = count(); }'

# Trace disk I/O
sudo bpftrace -e 'tracepoint:block:block_rq_issue { printf("%d %s\n", args->bytes, args->comm); }'
```

---

## Installation

```bash
# Core tools
sudo apt install strace ltrace gdb valgrind

# Performance tools
sudo apt install linux-tools-generic  # perf

# Benchmarking
cargo install hyperfine
# or
sudo apt install hyperfine

# Python profilers
pip install py-spy scalene memray

# Flame graphs
git clone https://github.com/brendangregg/FlameGraph
```
```

---

# task-watch.md

```markdown
---
description: "File watching and triggers - monitoring file changes and running commands on change"
when_to_use: "Use when: watching files for changes, auto-rebuilding on save, triggering commands when files change, monitoring directories for new files. Triggers: watch files, on change, auto rebuild, inotify, file monitor, entr, watch directory."
when-to-use: "Use when: watching files for changes, auto-rebuilding on save, triggering commands when files change, monitoring directories for new files. Triggers: watch files, on change, auto rebuild, inotify, file monitor, entr, watch directory."
argument-hint: "[describe what files to watch and what to run]"
context: fork
model: inherit
allowed-tools: Read
---

# File Watching Tasks

Primary tools: **entr**, **inotifywait**, **fswatch**, **watchman**

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Run on file change | `ls file.py \| entr python file.py` |
| Watch and rebuild | `find . -name "*.go" \| entr go build` |
| Watch directory | `inotifywait -m -r directory/` |
| Run command on any change | `fswatch -o . \| xargs -n1 make` |

---

## entr (recommended)

### Basic Usage
```bash
# Run command when file changes
ls file.py | entr python file.py

# Run command when any .py file changes
find . -name "*.py" | entr python main.py

# Clear screen and run
ls file.py | entr -c python file.py

# Restart server on change
ls *.py | entr -r python server.py

# Run shell command
ls *.md | entr sh -c 'pandoc doc.md -o doc.pdf'
```

### Watch Patterns
```bash
# Watch specific files
echo file1.txt file2.txt | tr ' ' '\n' | entr command

# Watch directory tree
find . -name "*.js" -o -name "*.ts" | entr npm run build

# Watch with exclusions
find . -name "*.py" ! -path "./.venv/*" | entr pytest

# Keep watching even if files are added
while true; do
  find . -name "*.go" | entr -d go build
done
```

### Options
```
-c  Clear screen before running
-r  Restart command (for servers)
-s  Use shell to run command
-d  Exit when new file is added to directory
```

---

## inotifywait (Linux)

### Basic Usage
```bash
# Watch single file
inotifywait -m file.txt

# Watch directory
inotifywait -m directory/

# Watch recursively
inotifywait -m -r directory/

# Specific events
inotifywait -m -e modify,create,delete directory/
```

### Run Command on Change
```bash
# Simple loop
while inotifywait -e modify file.py; do
  python file.py
done

# With event info
inotifywait -m -r -e modify,create,delete --format '%w%f %e' . | while read file event; do
  echo "File: $file Event: $event"
  make build
done
```

### Events
```
access      File was read
modify      File was modified
attrib      Metadata changed
close_write Closed after writing
close       File closed
open        File opened
moved_to    File moved into directory
moved_from  File moved out of directory
create      File created
delete      File deleted
```

---

## fswatch (cross-platform)

### Basic Usage
```bash
# Watch directory
fswatch directory/

# Watch with command
fswatch -o directory/ | xargs -n1 make build

# Recursive (default)
fswatch -r directory/

# Specific extensions
fswatch -e ".*" -i "\\.py$" directory/
```

### Options
```bash
# One event per line
fswatch -o directory/

# Include/exclude patterns
fswatch -e ".*" -i "\\.go$" .

# Latency (batch events)
fswatch -l 2 directory/    # 2 second delay
```

---

## watchman (Facebook)

### Setup
```bash
# Initialize in project
watchman watch .

# Check what's being watched
watchman watch-list
```

### Triggers
```bash
# Create trigger
watchman -- trigger . buildme '*.c' -- make build

# List triggers
watchman trigger-list .

# Delete trigger
watchman trigger-del . buildme
```

### Command Line Queries
```bash
# Find changed files since timestamp
watchman since . '{"since": "c:1234567890:0"}'

# Find all files matching pattern
watchman query . '{"expression": ["suffix", "py"]}'
```

---

## Common Patterns

### Auto-rebuild on Save
```bash
# Go
find . -name "*.go" | entr -c go build

# Python tests
find . -name "*.py" | entr -c pytest

# Node.js
find . -name "*.js" -o -name "*.ts" | entr -c npm run build

# LaTeX
ls *.tex | entr pdflatex document.tex

# Markdown to HTML
ls *.md | entr pandoc doc.md -o doc.html
```

### Restart Server on Change
```bash
# Python
ls *.py | entr -r python server.py

# Node
find . -name "*.js" | entr -r node server.js

# Go
find . -name "*.go" | entr -r go run .
```

### Watch and Sync
```bash
# Sync on change
fswatch -o src/ | xargs -n1 -I{} rsync -av src/ remote:dst/
```

---

## Installation

```bash
# entr (recommended)
sudo apt install entr

# inotify-tools
sudo apt install inotify-tools

# fswatch
sudo apt install fswatch

# watchman
# See: https://facebook.github.io/watchman/docs/install
```
```

---

# task-api-test.md

```markdown
---
description: "API and load testing - HTTP benchmarking, load testing, API validation, and performance testing"
when_to_use: "Use when: load testing APIs, benchmarking HTTP endpoints, stress testing servers, validating API responses, testing OpenAPI specs, measuring request latency. Triggers: load test, benchmark, stress test, api test, k6, wrk, vegeta, requests per second, latency, performance test."
when-to-use: "Use when: load testing APIs, benchmarking HTTP endpoints, stress testing servers, validating API responses, testing OpenAPI specs, measuring request latency. Triggers: load test, benchmark, stress test, api test, k6, wrk, vegeta, requests per second, latency, performance test."
argument-hint: "[describe the API or load testing task]"
context: fork
model: inherit
allowed-tools: Read
---

# API and Load Testing Tasks

Primary tools: **k6**, **wrk**, **vegeta**, **hurl**, **ab**

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Simple benchmark | `ab -n 1000 -c 10 http://localhost:8080/` |
| Load test with wrk | `wrk -t4 -c100 -d30s http://localhost:8080/` |
| Scripted load test | `k6 run script.js` |
| Rate-limited attack | `echo "GET http://localhost:8080" \| vegeta attack -rate=100 -duration=30s \| vegeta report` |
| Test API sequence | `hurl test.hurl` |

---

## Apache Bench (ab)

### Basic Usage
```bash
# 1000 requests, 10 concurrent
ab -n 1000 -c 10 http://localhost:8080/

# With keep-alive
ab -n 1000 -c 10 -k http://localhost:8080/

# POST request
ab -n 1000 -c 10 -p data.json -T application/json http://localhost:8080/api

# With headers
ab -n 1000 -c 10 -H "Authorization: Bearer token" http://localhost:8080/
```

### Output Explained
```
Requests per second:    1234.56 [#/sec] (mean)
Time per request:       8.1 [ms] (mean)
Time per request:       0.81 [ms] (mean, across all concurrent requests)
```

---

## wrk

### Basic Usage
```bash
# 4 threads, 100 connections, 30 seconds
wrk -t4 -c100 -d30s http://localhost:8080/

# With timeout
wrk -t4 -c100 -d30s --timeout 10s http://localhost:8080/

# With latency stats
wrk -t4 -c100 -d30s --latency http://localhost:8080/
```

### Lua Scripts
```lua
-- post.lua
wrk.method = "POST"
wrk.body   = '{"key": "value"}'
wrk.headers["Content-Type"] = "application/json"
```

```bash
wrk -t4 -c100 -d30s -s post.lua http://localhost:8080/api
```

---

## vegeta

### Basic Attack
```bash
# Simple GET attack
echo "GET http://localhost:8080/" | vegeta attack -duration=30s | vegeta report

# Rate-limited
echo "GET http://localhost:8080/" | vegeta attack -rate=100 -duration=30s | vegeta report

# Multiple endpoints
cat targets.txt | vegeta attack -duration=30s | vegeta report
```

### Target File Format
```
GET http://localhost:8080/api/users
GET http://localhost:8080/api/posts

POST http://localhost:8080/api/users
Content-Type: application/json
{"name": "test"}
```

### Reports and Plots
```bash
# JSON report
vegeta attack ... | vegeta report -type=json

# Histogram
vegeta attack ... | vegeta report -type=hist[0,10ms,50ms,100ms,500ms,1s]

# Plot (requires gnuplot)
vegeta attack ... | vegeta plot > plot.html
```

---

## k6

### Basic Script
```javascript
// script.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 10,           // virtual users
  duration: '30s',
};

export default function () {
  const res = http.get('http://localhost:8080/');
  check(res, {
    'status is 200': (r) => r.status === 200,
  });
  sleep(1);
}
```

```bash
k6 run script.js
```

### Stages (Ramp Up/Down)
```javascript
export const options = {
  stages: [
    { duration: '30s', target: 20 },   // Ramp up to 20 VUs
    { duration: '1m', target: 20 },    // Stay at 20 VUs
    { duration: '10s', target: 0 },    // Ramp down
  ],
};
```

### POST Requests
```javascript
import http from 'k6/http';

export default function () {
  const payload = JSON.stringify({ key: 'value' });
  const params = {
    headers: {
      'Content-Type': 'application/json',
    },
  };
  http.post('http://localhost:8080/api', payload, params);
}
```

### Output
```bash
# JSON output
k6 run --out json=results.json script.js

# InfluxDB
k6 run --out influxdb=http://localhost:8086/k6 script.js
```

---

## hey

### Basic Usage
```bash
# 1000 requests, 100 concurrent
hey -n 1000 -c 100 http://localhost:8080/

# Duration-based
hey -z 30s -c 100 http://localhost:8080/

# With rate limit
hey -q 100 -z 30s http://localhost:8080/
```

### POST Request
```bash
hey -n 1000 -c 100 -m POST -d '{"key":"value"}' \
    -H "Content-Type: application/json" \
    http://localhost:8080/api
```

---

## hurl (API Testing)

### Basic Test File
```hurl
# test.hurl

# GET request
GET http://localhost:8080/api/users

HTTP 200
[Asserts]
jsonpath "$.users" count == 10


# POST request
POST http://localhost:8080/api/users
Content-Type: application/json
{
  "name": "Test User",
  "email": "test@example.com"
}

HTTP 201
[Asserts]
jsonpath "$.id" exists


# With variable capture
GET http://localhost:8080/api/users/1
HTTP 200
[Captures]
user_name: jsonpath "$.name"

GET http://localhost:8080/api/search?q={{user_name}}
HTTP 200
```

### Running Tests
```bash
# Run test file
hurl test.hurl

# With verbose output
hurl --verbose test.hurl

# Multiple files
hurl tests/*.hurl

# JSON report
hurl --report-json report.json test.hurl
```

---

## schemathesis (OpenAPI Fuzzing)

```bash
# Test all endpoints
schemathesis run http://localhost:8080/openapi.json

# Specific endpoint
schemathesis run --endpoint /api/users http://localhost:8080/openapi.json

# With authentication
schemathesis run -H "Authorization: Bearer token" http://localhost:8080/openapi.json
```

---

## Installation

```bash
# Apache Bench
sudo apt install apache2-utils

# wrk
sudo apt install wrk

# hey
go install github.com/rakyll/hey@latest

# vegeta
go install github.com/tsenart/vegeta@latest

# k6
sudo apt install k6
# or
snap install k6

# hurl
cargo install hurl

# schemathesis
pip install schemathesis
```
```

---

# Update toolkit-master.md

After review, replace the current toolkit-master.md with this updated version that references all 19 categories:

```markdown
---
description: "CLI Toolkit Master Index - browse all available task-based command-line skills (human use)"
disable-model-invocation: true
argument-hint: "[optional: specific task or tool query]"
context: fork
model: inherit
allowed-tools: Read, Skill
---

# CLI Toolkit Master Index

A task-oriented skill system for command-line operations. Skills are organized by **what you want to accomplish**, not by individual tools.

Use `/toolkit-master` to browse available skills. Agents auto-route to appropriate skills based on what they're trying to do.

---

## Available Task Skills

### Data Processing
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-json` | Parse, query, transform JSON/YAML/XML | jq, yq, xq |
| `/task-tabular` | CSV/TSV processing, column operations, aggregation | awk, cut, sort, miller |
| `/task-text` | Search patterns, filter lines, find/replace | grep, rg, sed |

### File Operations
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-files` | Find files, compare, sync, disk usage | find, fd, diff, rsync, du |
| `/task-archives` | Create/extract archives, compress files | tar, gzip, zip, 7z, zstd |
| `/task-watch` | Watch files, auto-run on change | entr, inotifywait, fswatch |

### Networking
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-http` | API calls, downloads, HTTP debugging | curl, wget, httpie |
| `/task-network` | Connectivity, DNS, ports, traffic | ping, dig, nmap, ss, tcpdump |
| `/task-api-test` | Load testing, benchmarking, API validation | k6, wrk, vegeta, hurl |

### System Administration
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-process` | Manage processes, parallel execution | ps, kill, xargs, parallel |
| `/task-system` | System info, services, logs, scheduling | systemctl, journalctl, cron |
| `/task-permissions` | Users, groups, file permissions | chmod, chown, useradd |
| `/task-containers` | Docker, tmux, environments | docker, docker-compose, tmux |

### Development
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-git` | Version control, GitHub, diffs | git, gh, delta |
| `/task-dev` | Build, lint, format, language tools | make, shellcheck, ruff, prettier |
| `/task-debug` | Tracing, profiling, benchmarking | strace, perf, valgrind, hyperfine |
| `/task-database` | SQL queries, SQLite, Redis | sqlite3, psql, duckdb, redis-cli |

### Media & Crypto
| Skill | What It Helps With | Key Tools |
|-------|-------------------|-----------|
| `/task-media` | Video, audio, images, documents | ffmpeg, imagemagick, pandoc |
| `/task-crypto` | Hashing, encoding, encryption | sha256sum, base64, gpg, openssl |

---

## Quick Decision Guide

| I want to... | Use |
|--------------|-----|
| Parse JSON / extract fields | `/task-json` |
| Process CSV / sum columns | `/task-tabular` |
| Search text / filter lines | `/task-text` |
| Find files / compare dirs | `/task-files` |
| Make API requests | `/task-http` |
| Convert video / resize image | `/task-media` |
| Compress / extract archives | `/task-archives` |
| Debug network issues | `/task-network` |
| Manage processes | `/task-process` |
| View system logs | `/task-system` |
| Run Docker containers | `/task-containers` |
| Git operations / PRs | `/task-git` |
| Lint / format code | `/task-dev` |
| Profile performance | `/task-debug` |
| Run SQL queries | `/task-database` |
| Load test an API | `/task-api-test` |
| Watch files for changes | `/task-watch` |
| Hash files / encrypt | `/task-crypto` |
| Set file permissions | `/task-permissions` |
```
