---
description: "Log analysis - errors in logs but can't find root cause, application crashed and need to investigate, what went wrong, logs filling up disk, viewing system logs, filtering by severity, and real-time monitoring (journalctl, lnav, goaccess)"
when_to_use: "Use when: seeing errors in logs but can't find root cause, application crashed and need to investigate what happened, need to understand what went wrong from logs, logs are filling up disk and need rotation, searching log files for patterns, filtering by severity or time range, monitoring logs in real-time, parsing access logs. Triggers: errors in logs, application crashed, what went wrong, logs filling up disk, find root cause, why did it crash, error messages, journalctl, syslog, log file, tail log, log search, error log, access log, logrotate, structured log, lnav, log analysis, log monitor, docker logs, container log, multiline, stack trace, compressed log, zgrep, audit log, error rate, log rotation, log level, log grep, parse log, log statistics."
when-to-use: "Use when: seeing errors in logs but can't find root cause, application crashed and need to investigate what happened, need to understand what went wrong from logs, logs are filling up disk and need rotation, searching log files for patterns, filtering by severity or time range, monitoring logs in real-time, parsing access logs. Triggers: errors in logs, application crashed, what went wrong, logs filling up disk, find root cause, why did it crash, error messages, journalctl, syslog, log file, tail log, log search, error log, access log, logrotate, structured log, lnav, log analysis, log monitor, docker logs, container log, multiline, stack trace, compressed log, zgrep, audit log, error rate, log rotation, log level, log grep, parse log, log statistics."
argument-hint: "[describe what log analysis you need]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Log Analysis Task Router

You have access to the **full conversation context**. Your job is to build a well-formed argument for the log analysis recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Analyze the conversation to identify:

### 1. Goal
What log operation is needed?
- View logs, search patterns, filter by severity/time, parse structured logs
- Monitor real-time output, analyze access logs, correlate events
- Configure rotation, aggregate statistics, extract fields

### 2. Log Source
Where are the logs?
- systemd journal (journalctl)
- Log files (/var/log/*, application logs)
- Structured logs (JSON, key=value)
- Access logs (Apache, Nginx)
- Application-specific paths

### 3. Filter Criteria
What to look for?
- **Time range**: since, until, today, last hour
- **Severity**: emerg, alert, crit, err, warning, notice, info, debug
- **Service/unit**: specific systemd unit
- **Pattern**: text search, regex
- **Field**: specific structured log fields

### 4. Output Requirements
What format?
- Raw log lines
- JSON output
- Counts/statistics
- Specific fields extracted
- Aggregated by time window

### 5. Context
Additional details?
- Real-time monitoring vs historical search
- Single service vs system-wide
- Log rotation configuration needs
- Correlation across multiple sources

---

## Goal-to-Tool Mapping

| Goal | Primary Tool | Notes |
|------|-------------|-------|
| System journal logs | journalctl | Systemd journal queries |
| Interactive log viewer | lnav | TUI with filtering, SQL queries |
| Web access log analysis | goaccess | Real-time HTTP log analyzer |
| Log rotation config | logrotate | Manage log file lifecycle |
| Colorize log output | ccze | Highlight log severity |
| Follow log file | tail -f | Real-time log monitoring |
| Search compressed logs | zgrep | grep inside .gz files |
| Custom log aggregation | awk | Parse and aggregate fields |
| Structured log queries | jq | JSON log filtering/extraction |

---

## Argument Format

Construct a structured argument string:

```
<goal> from <source> - filter: <criteria>, time: <range>, output: <format>
```

**Include only relevant components.**

### Examples

**View recent errors:**
```
show errors from journalctl - service: nginx, time: last 1 hour
```

**Search log file:**
```
search for "connection refused" in /var/log/app/error.log - time: today, context: 3 lines
```

**Parse access log:**
```
analyze /var/log/nginx/access.log - top 10 IPs by request count
```

**Monitor real-time:**
```
follow logs from journalctl - service: myapp, severity: err and above
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-logs-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Log source is unclear
- Time range is ambiguous
- Multiple log sources could apply

**Use Read tool first if:**
- Need to verify log file exists
- Need to check log format before parsing
