---
description: "Log analysis recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-logs with log source and context included"
when-to-use: "Receives prepared arguments from task-logs with log source and context included"
context: fork
model: inherit
allowed-tools: Read
---

# Log Analysis Recipes

**Tools:** journalctl, logger, lnav, ccze, multitail, logrotate, goaccess, tail, awk, sed, jq

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| View all logs | `journalctl` |
| Follow logs | `journalctl -f` |
| Service logs | `journalctl -u nginx` |
| Errors only | `journalctl -p err` |
| Since time | `journalctl --since "1 hour ago"` |
| Tail log file | `tail -f /var/log/syslog` |
| Search log | `grep -i "error" /var/log/app.log` |
| Colorize logs | `journalctl -u nginx | ccze -A` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| Systemd logs | journalctl | Native journal queries |
| Interactive log browsing | lnav | TUI with SQL, filtering |
| Web access log analysis | goaccess | Real-time dashboard |
| Log colorization | ccze | Highlight severity levels |
| Log rotation | logrotate | Manage file lifecycle |
| Compressed log search | zgrep / zless | Search .gz without decompressing |
| JSON log filtering | jq | Structured log extraction |
| Custom log parsing | awk | Field-based text processing |

---

## journalctl (systemd Journal)

### Basic Viewing
```bash
# All logs
journalctl

# Follow new entries (like tail -f)
journalctl -f

# Last 100 lines
journalctl -n 100

# No pager (direct output)
journalctl --no-pager

# Reverse order (newest first)
journalctl -r
```

### Filter by Service/Unit
```bash
# Specific service
journalctl -u nginx
journalctl -u ssh

# Multiple services
journalctl -u nginx -u php-fpm

# User services
journalctl --user -u myapp

# Kernel messages only
journalctl -k
```

### Filter by Time
```bash
# Since specific time
journalctl --since "2024-01-15 10:00:00"
journalctl --since "1 hour ago"
journalctl --since "today"
journalctl --since "yesterday"

# Time range
journalctl --since "2024-01-15" --until "2024-01-16"
journalctl --since "09:00" --until "10:00"

# Current boot only
journalctl -b
journalctl -b -1      # Previous boot
journalctl --list-boots  # List all boots
```

### Filter by Priority
```bash
# Priority levels: emerg(0), alert(1), crit(2), err(3), warning(4), notice(5), info(6), debug(7)
journalctl -p err                  # Errors and above
journalctl -p warning              # Warnings and above
journalctl -p err..crit            # Range: errors to critical

# Combined filters
journalctl -u nginx -p err --since "1 hour ago"
```

### Output Formats
```bash
# JSON output (one object per line)
journalctl -u nginx -o json

# Pretty JSON
journalctl -u nginx -o json-pretty -n 5

# Short with timestamp
journalctl -o short-precise

# Verbose (all fields)
journalctl -o verbose -n 1

# Export format (for processing)
journalctl -o export -n 100
```

### Search and Grep
```bash
# Grep within journal
journalctl -u nginx --grep "error"
journalctl -u nginx --grep "50[0-9]" --since "today"

# Case insensitive grep
journalctl --grep "error" --case-sensitive=no

# Combine with external grep for complex patterns
journalctl -u myapp --no-pager | grep -E "ERROR|FATAL"
```

### Disk Usage
```bash
# Journal disk usage
journalctl --disk-usage

# Vacuum by size
sudo journalctl --vacuum-size=500M

# Vacuum by time
sudo journalctl --vacuum-time=30d

# Rotate journals
sudo journalctl --rotate
```

### Process and PID Filtering
```bash
# By PID
journalctl _PID=1234

# By executable path
journalctl _EXE=/usr/sbin/nginx

# By user
journalctl _UID=1000
```

---

## Log File Analysis

### Tail and Follow
```bash
# Follow a log file
tail -f /var/log/syslog

# Follow multiple files
tail -f /var/log/syslog /var/log/auth.log

# Follow with grep
tail -f /var/log/app.log | grep --line-buffered "ERROR"

# Last N lines then follow
tail -n 100 -f /var/log/app.log

# multitail (multiple files in split view)
multitail /var/log/syslog /var/log/auth.log
multitail -l "journalctl -f -u nginx" -l "journalctl -f -u php-fpm"
```

### Searching and Filtering
```bash
# Search with context
grep -C 3 "ERROR" /var/log/app.log

# Search with timestamps
grep "2024-01-15.*ERROR" /var/log/app.log

# Exclude patterns
grep -v "DEBUG" /var/log/app.log | grep -v "INFO"

# Count errors by type
grep -oP 'ERROR: \K[^:]+' /var/log/app.log | sort | uniq -c | sort -rn

# Extract timestamps of errors
grep "ERROR" /var/log/app.log | awk '{print $1, $2}' | sort | uniq -c
```

### Access Log Analysis (Apache/Nginx)
```bash
# Top 10 IPs
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Top requested URLs
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Response code distribution
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# 4xx/5xx errors
awk '$9 >= 400' /var/log/nginx/access.log

# Requests per hour
awk '{print substr($4,2,14)}' /var/log/nginx/access.log | sort | uniq -c

# Requests from specific IP
grep "192.168.1.100" /var/log/nginx/access.log

# Slow requests (by response time, if logged)
awk -F'"' '{print $NF}' /var/log/nginx/access.log | sort -rn | head -20

# Bandwidth by URL
awk '{urls[$7]+=$10} END {for (url in urls) print urls[url], url}' /var/log/nginx/access.log | sort -rn | head -20
```

---

## Structured Logs (JSON)

```bash
# Parse JSON logs with jq
cat app.log | jq -r 'select(.level == "error") | "\(.timestamp) \(.message)"'

# Count by level
cat app.log | jq -r '.level' | sort | uniq -c | sort -rn

# Filter by time range
cat app.log | jq -r 'select(.timestamp >= "2024-01-15T10:00:00")'

# Extract specific fields
cat app.log | jq -r '[.timestamp, .level, .message] | @tsv'

# Correlate by request ID
cat app.log | jq -r 'select(.request_id == "abc-123")'

# Group errors by module
cat app.log | jq -r 'select(.level == "error") | .module' | sort | uniq -c | sort -rn

# Key=value structured logs (parse with awk)
grep 'level=error' app.log | sed 's/.*msg="\([^"]*\)".*/\1/' | sort | uniq -c | sort -rn
```

---

## Docker and Container Logs

### Docker Logs
```bash
# Recent container logs
docker logs --tail 100 mycontainer

# Follow logs with timestamps
docker logs -f --timestamps mycontainer

# Logs since time
docker logs --since 1h mycontainer
docker logs --since "2024-01-15T10:00:00" mycontainer

# Errors only from container
docker logs mycontainer 2>&1 | grep -i error

# Error count per container
for c in $(docker ps --format '{{.Names}}'); do
  echo "$c: $(docker logs "$c" 2>&1 | grep -ci error)"
done

# Categorize errors from container
docker logs mycontainer 2>&1 | awk '/ERROR/{match($0,/ERROR: ([^:]+)/,a); count[a[1]]++} END{for(k in count) print count[k], k}' | sort -rn
```

### Docker Compose Logs
```bash
# Follow all services
docker compose logs -f

# Specific service
docker compose logs -f --tail 50 web

# Since time
docker compose logs --since 1h

# Errors across all services
docker compose logs --no-color 2>&1 | grep -i error
```

---

## Multiline Log Parsing

### Java Stack Traces
```bash
# Group multiline Java exceptions (new entry starts with timestamp)
awk '/^[0-9]{4}-[0-9]{2}-[0-9]{2}/{if(buf && buf ~ /Exception/) print buf; buf=$0; next} {buf=buf ORS $0} END{if(buf ~ /Exception/) print buf}' app.log

# Extract stack traces with context
grep -A 30 'Exception' app.log | head -100

# Count exception types
grep -oP '^\S+Exception' app.log | sort | uniq -c | sort -rn
```

### Python Tracebacks
```bash
# Group Python tracebacks
awk '/^Traceback/{p=1} p{print} /^[^ ]/ && p && !/^Traceback/{p=0; print "---"}' app.log

# Count Python error types
grep -oP '^\w+Error' app.log | sort | uniq -c | sort -rn
```

### Generic Multiline Grouping
```bash
# Group entries starting with timestamp (handles multi-line messages)
awk '/^[0-9]{4}-/{if(buf)print buf; buf=$0; next}{buf=buf ORS $0} END{if(buf)print buf}' app.log

# Extract multiline entries matching pattern
awk '/^[0-9]{4}-/{if(buf && match(buf,/CRITICAL/))print buf; buf=$0; next}{buf=buf ORS $0} END{if(buf && match(buf,/CRITICAL/))print buf}' app.log
```

---

## Compressed Log Search

```bash
# Search compressed rotated logs
zgrep "ERROR" /var/log/syslog.*.gz
zgrep -c "ERROR" /var/log/syslog.*.gz          # Count per file

# Search across both compressed and uncompressed
zgrep "OutOfMemory" /var/log/app.log*

# Decompress and process
zcat /var/log/app.log.1.gz | awk '/ERROR/{print $1, $2, $5}' | sort | uniq -c | sort -rn

# Search bzip2 compressed logs
bzgrep "FATAL" /var/log/archived.log.bz2

# Search xz compressed logs
xzgrep "panic" /var/log/kern.log.*.xz

# Search all rotated logs sorted by time
zgrep -h "ERROR" /var/log/app.log* | sort | head -50
```

---

## Error Rate Calculation

```bash
# Error count per time window (per hour)
awk '/ERROR/{split($1,d,"-"); split($2,t,":"); print d[1]"-"d[2]"-"d[3]" "t[1]":00"}' app.log | sort | uniq -c

# Error percentage
awk '{total++} /ERROR/{err++} END{printf "Errors: %d/%d (%.2f%%)\n", err, total, err*100/total}' app.log

# Requests per second from access log
awk '{print $4}' access.log | cut -c2-21 | sort | uniq -c | sort -rn | head -20

# Response code distribution with percentages
awk '{s[$9]++; t++} END{for(k in s) printf "%s: %d (%.1f%%)\n", k, s[k], s[k]*100/t}' access.log | sort -t: -k2 -rn

# Error rate per minute (sliding window)
awk '/ERROR/{split($2,t,":"); key=t[1]":"t[2]; c[key]++} END{for(k in c) print k, c[k]}' app.log | sort
```

---

## Cross-Service Correlation

```bash
# Grep by request ID across multiple log files
grep -rh "req-abc123" /var/log/app*.log | sort -t' ' -k1,2

# Merged timeline from multiple services
grep -h "req-abc123" /var/log/api.log /var/log/worker.log /var/log/db.log | sort

# Find all request IDs with errors, then trace them
grep -oP 'req-[a-f0-9-]+' /var/log/app.log | sort -u | while read id; do
  if grep -q "ERROR.*$id" /var/log/app.log; then
    echo "=== $id ==="
    grep "$id" /var/log/app.log
  fi
done

# Cross-correlate by timestamp window (±5 seconds)
grep "ERROR" /var/log/api.log | awk '{print $1" "$2}' | while read ts; do
  grep "$ts" /var/log/worker.log
done
```

---

## Application Log Formats

### Django / Flask
```bash
# Django log format (default: [timestamp] level module: message)
grep "ERROR" /var/log/django/app.log | awk -F': ' '{print $2}' | sort | uniq -c | sort -rn

# Flask request logs
grep "POST\|PUT\|DELETE" /var/log/flask/access.log | awk '{print $6, $7, $9}' | sort | uniq -c | sort -rn
```

### Node.js / Express
```bash
# Morgan combined format (Express)
awk '$9 >= 500' /var/log/node/access.log | head -20

# PM2 structured logs
cat /var/log/pm2/app-*.log | grep -i error | jq -r '.message' 2>/dev/null || grep -i error
```

### Spring Boot
```bash
# Spring Boot default log format
grep "ERROR" /var/log/spring/app.log | awk -F'---' '{print $2}' | sed 's/^ *//' | sort | uniq -c | sort -rn

# Extract slow queries from Spring Data logs
grep -oP 'executed in \K[0-9]+ms' /var/log/spring/app.log | sort -n | tail -20
```

---

## Log Anonymization

```bash
# Redact IP addresses
sed -E 's/[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}/XXX.XXX.XXX.XXX/g' access.log

# Mask email addresses
sed -E 's/[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/***@***.****/g' app.log

# Replace sensitive query parameters
sed -E 's/(password|token|secret|key)=[^& ]*/\1=REDACTED/gi' app.log

# Scrub credit card numbers (basic pattern)
sed -E 's/[0-9]{4}[- ]?[0-9]{4}[- ]?[0-9]{4}[- ]?[0-9]{4}/XXXX-XXXX-XXXX-XXXX/g' app.log
```

---

## Real-time Alerting Patterns

```bash
# Alert on CRITICAL in real-time
tail -f /var/log/app.log | grep --line-buffered "CRITICAL" | while read line; do
  logger -p user.crit "ALERT: $line"
done

# Count errors per minute, alert if threshold exceeded
tail -f /var/log/app.log | awk '/ERROR/{c++; if(c%10==0) system("logger -p user.warning \"10 more errors detected\"")}'

# Monitor for specific pattern with timestamp
tail -f /var/log/app.log | grep --line-buffered "OutOfMemory" | ts '[%Y-%m-%d %H:%M:%S]'
```

---

## Audit Logs

```bash
# Search audit logs (auditd)
ausearch -m LOGIN --start today
ausearch -m USER_AUTH --success no --start today
ausearch -ua 1000 --start recent

# Audit reports
aureport --auth                           # Authentication report
aureport --failed                         # Failed events
aureport --login --summary               # Login summary

# Failed login attempts
lastb | head -20                          # Last failed logins
faillog -a                                # All user failure counts

# Auth log analysis (non-auditd systems)
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn | head -10
```

---

## lnav (Log File Navigator)

```bash
# Open log files (auto-detects format)
lnav /var/log/syslog

# Open multiple files
lnav /var/log/syslog /var/log/auth.log

# Follow journalctl output
journalctl -f | lnav

# SQL queries on logs (within lnav)
# Press : then type SQL:
# ;SELECT count(*) FROM syslog_log WHERE log_level = 'error'
# ;SELECT log_hostname, count(*) FROM syslog_log GROUP BY log_hostname
# ;SELECT * FROM syslog_log WHERE log_body LIKE '%failed%' ORDER BY log_time DESC LIMIT 20

# Filter by log level (within lnav)
# Press : then type:  :filter-in error

# Headless mode (export)
lnav -n -c ";SELECT * FROM syslog_log WHERE log_level = 'error'" /var/log/syslog
```

---

## logrotate

```bash
# View current configuration
cat /etc/logrotate.conf
ls /etc/logrotate.d/

# Test configuration (dry run)
sudo logrotate -d /etc/logrotate.conf

# Force rotation
sudo logrotate -f /etc/logrotate.conf

# Force rotate specific config
sudo logrotate -f /etc/logrotate.d/nginx

# Example custom config (/etc/logrotate.d/myapp):
# /var/log/myapp/*.log {
#     daily
#     rotate 14
#     compress
#     delaycompress
#     missingok
#     notifempty
#     create 0640 www-data www-data
#     sharedscripts
#     postrotate
#         systemctl reload myapp > /dev/null 2>&1 || true
#     endscript
# }
```

---

## goaccess (Web Log Analyzer)

```bash
# Interactive terminal dashboard
goaccess /var/log/nginx/access.log --log-format=COMBINED

# Generate HTML report
goaccess /var/log/nginx/access.log --log-format=COMBINED -o report.html

# Real-time HTML report
goaccess /var/log/nginx/access.log --log-format=COMBINED -o report.html --real-time-html

# Process multiple log files
goaccess /var/log/nginx/access.log /var/log/nginx/access.log.1 --log-format=COMBINED

# Apache log format
goaccess /var/log/apache2/access.log --log-format=COMBINED
```

---

## Logging (logger)

```bash
# Send message to syslog
logger "Application started successfully"

# With priority
logger -p local0.error "Database connection failed"

# With tag
logger -t myapp "Backup completed"

# Log from script
logger -t backup-script "Backup of /data completed, size: $(du -sh /data | cut -f1)"
```

---

## Common Workflows

### Incident Investigation
```bash
# 1. Check for errors around incident time
journalctl --since "2024-01-15 14:00" --until "2024-01-15 14:30" -p err

# 2. Check specific service
journalctl -u myapp --since "2024-01-15 14:00" --no-pager

# 3. Check for OOM kills
journalctl -k --grep "Out of memory" --since "today"

# 4. Check disk-related issues
journalctl --grep "No space left" --since "today"

# 5. Check auth failures
journalctl -u ssh --grep "Failed" --since "today"
```

### Log Aggregation
```bash
# Aggregate errors per hour
journalctl -u myapp -p err --since "today" -o json | \
  jq -r '.["__REALTIME_TIMESTAMP"]' | \
  awk '{print strftime("%Y-%m-%d %H:00", $1/1000000)}' | sort | uniq -c

# Error rate over time (from log file)
awk '/ERROR/ {print substr($1,1,13)}' app.log | uniq -c
```

---

## Combined Workflows

### Incident Investigation Pipeline
```bash
# Journal errors -> JSON -> extract -> aggregate
journalctl -u app --since "1h ago" -o json | \
  jq -r 'select(.PRIORITY <= "3") | .MESSAGE' | \
  sort | uniq -c | sort -rn

# Docker error triage across all containers
for c in $(docker ps -q); do
  echo "$(docker inspect --format '{{.Name}}' $c): $(docker logs "$c" 2>&1 | grep -c ERROR)"
done

# Access log response code breakdown with percentages
awk '{s[$9]++} END{for(k in s) printf "%s: %d (%.1f%%)\n",k,s[k],s[k]*100/NR}' access.log | sort -t: -k2 -rn

# Compressed log search across rotated files
zgrep -h "OutOfMemory" /var/log/app.log* | sort | head -20

# Multi-service error timeline
grep -h "ERROR" /var/log/{api,worker,scheduler}.log | sort | tail -50

# Log -> anonymize -> aggregate
sed -E 's/[0-9]{1,3}(\.[0-9]{1,3}){3}/X.X.X.X/g' access.log | awk '{print $7, $9}' | sort | uniq -c | sort -rn | head -20
```

---

## Installation

```bash
# journalctl is part of systemd (pre-installed)

# Log navigator
sudo apt install lnav

# Colorize logs
sudo apt install ccze

# Multiple log tailing
sudo apt install multitail

# Web log analyzer
sudo apt install goaccess
```
