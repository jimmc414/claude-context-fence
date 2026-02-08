---
description: "System administration - system running slow, out of memory, service won't start, high load average, disk full, hardware specs, memory/CPU monitoring, systemd services, and cron scheduling (systemctl, journalctl, free, iostat)"
when_to_use: "Use when: system is running slow or unresponsive, out of memory or high memory usage, a service won't start or keeps failing, high load average, disk is full, need to check system specs, monitoring CPU or memory usage, managing systemd services, scheduling cron jobs, checking hardware sensors. Triggers: system slow, out of memory, service won't start, high load average, disk full, system unresponsive, high cpu, OOM killer, system info, memory usage, cpu info, journalctl, systemctl, cron, logs, date, uptime, hostname, service, timer, dmesg, sysctl, sensors, hardware, package install, cron job, systemd service, kernel, swap, boot time, timezone, locale, apt, dnf."
when-to-use: "Use when: system is running slow or unresponsive, out of memory or high memory usage, a service won't start or keeps failing, high load average, disk is full, need to check system specs, monitoring CPU or memory usage, managing systemd services, scheduling cron jobs, checking hardware sensors. Triggers: system slow, out of memory, service won't start, high load average, disk full, system unresponsive, high cpu, OOM killer, system info, memory usage, cpu info, journalctl, systemctl, cron, logs, date, uptime, hostname, service, timer, dmesg, sysctl, sensors, hardware, package install, cron job, systemd service, kernel, swap, boot time, timezone, locale, apt, dnf."
argument-hint: "[describe what system information you need]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# System Administration Task Router

You have access to the **full conversation context**. Your job is to analyze the user's system info/admin need and build a well-formed argument for the recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

System tasks range from quick info lookups to service management. Analyze conversation to identify:

### 1. Goal
What system operation is needed?
- Get system/hardware info
- Monitor resources (CPU, memory, disk, IO)
- Manage a service (start, stop, enable, status)
- Read/search logs
- Schedule a task (cron or timer)
- Date/time calculation or conversion
- Kernel parameter tuning

### 2. Specifics
- Which service name?
- Which log unit or time range?
- What date format or timezone?
- What cron schedule?
- Which hardware component?

### 3. Environment Context
- systemd or init.d?
- WSL2? (limited systemd)
- Server or desktop?
- Root access available?

### 4. Prior Findings
- Error from service status
- Log entries discussed
- System specs mentioned earlier

---

## Goal --> Tool Guide

| Goal | Primary Tool | Argument Prefix |
|------|-------------|----------------|
| System specs | uname/lscpu/lsmem | "show system info" |
| Memory/CPU stats | free/vmstat/mpstat | "show resource usage" |
| Disk I/O | iostat | "show disk stats" |
| Service management | systemctl | "manage service" |
| Read logs | journalctl | "show logs for" |
| Schedule task | crontab/systemd timer | "schedule" |
| Date calculation | date/dateutils | "calculate date" |
| Hardware sensors | sensors/nvidia-smi | "show sensors" |
| Kernel tuning | sysctl | "tune kernel" |
| Boot analysis | systemd-analyze | "analyze boot" |

---

## Argument Format

```
<goal> - target: <specific item>, context: <relevant details>
```

### Examples

**Service management:**
```
manage service nginx - check status and recent errors, restart if needed
```

**Log search:**
```
show logs for sshd - last 2 hours, errors only, show source IPs
```

**Cron scheduling:**
```
schedule backup.sh to run daily at 2am - log output to /var/log/backup.log
```

**Date conversion:**
```
calculate date - convert epoch 1705276800 to human readable in US Eastern
```

**System overview:**
```
show system info - CPU, RAM, disk, kernel version, load average
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-system-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Service name is ambiguous
- Cron schedule requirements unclear
- Date format not specified

**Use Read tool first if:**
- Need to check existing crontab
- Need to verify unit file contents
- Need to inspect log output
