---
description: "Network diagnostics - can't connect to server, network is slow, SSH not working, connection refused, DNS not resolving, ping, traceroute, port scanning, and traffic capture (nmap, dig, tcpdump, mtr, ssh)"
when_to_use: "Use when: can't connect to a server or remote host, network is slow or unreliable, SSH connection not working or timing out, getting connection refused errors, DNS not resolving hostnames, need to check if a port is open on a remote host, debugging network path or routing issues, setting up SSH tunnels. Triggers: can't connect, network slow, SSH not working, connection refused, DNS not resolving, host unreachable, connection timeout, network down, ping, traceroute, dns, dig, nmap, ports, netstat, ss, ip address, tcpdump, firewall, ssh, tunnel, mtr, whois, socket, port scan, iptables, ssh tunnel, packet capture, bandwidth test, network interface, latency test, route, arp, open ports."
when-to-use: "Use when: can't connect to a server or remote host, network is slow or unreliable, SSH connection not working or timing out, getting connection refused errors, DNS not resolving hostnames, need to check if a port is open on a remote host, debugging network path or routing issues, setting up SSH tunnels. Triggers: can't connect, network slow, SSH not working, connection refused, DNS not resolving, host unreachable, connection timeout, network down, ping, traceroute, dns, dig, nmap, ports, netstat, ss, ip address, tcpdump, firewall, ssh, tunnel, mtr, whois, socket, port scan, iptables, ssh tunnel, packet capture, bandwidth test, network interface, latency test, route, arp, open ports."
argument-hint: "[describe the network task]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Network Diagnostics Task Router

You have access to the **full conversation context**. Your job is to analyze the user's network diagnostic need and build a well-formed argument for the recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Network requests are symptom-driven -- users describe what's wrong, not which tool to use. Analyze the conversation to identify:

### 1. Symptom / Goal
What's the problem or what does the user want to know?
- Can't reach a host --> ping, traceroute, mtr
- DNS not resolving --> dig, nslookup, host
- Port closed/filtered --> nmap, nc, ss
- Need to see traffic --> tcpdump, tshark
- Connection refused vs timeout --> ss, iptables check
- Slow transfers --> mtr, tcpdump window analysis
- Need SSH tunnel --> ssh -L/-R/-D
- Firewall issue --> iptables, nft, ufw

### 2. Target
What are we investigating?
- Hostname/IP from conversation
- Port number(s)
- Network interface
- Container/service name

### 3. Environment Context
- Cloud provider? (AWS SG, GCP firewall rules)
- VPN active?
- Container network? (Docker bridge, K8s overlay)
- WSL? (different network stack)
- Root access available?

### 4. Prior Findings
- Error messages from earlier in conversation
- Previous dig/ping output discussed
- What's already been tried

### 5. Scope
- Specific protocol? (TCP, UDP, ICMP)
- Duration/count limits?
- Output format needs?

---

## Symptom --> Tool Guide

| Symptom | Primary Tool | Argument Prefix |
|---------|-------------|----------------|
| Can't reach host | ping/traceroute/mtr | "test connectivity to" |
| DNS not resolving | dig/nslookup | "resolve DNS for" |
| Port not responding | nmap/nc/ss | "check port on" |
| Need traffic capture | tcpdump/tshark | "capture traffic on" |
| Connection refused | ss/iptables | "diagnose connection refused to" |
| Slow network | mtr/tcpdump | "diagnose slow connection to" |
| Need tunnel/forward | ssh | "create tunnel to" |
| Firewall config | iptables/nft/ufw | "configure firewall for" |
| What's listening | ss/lsof | "show listening ports" |
| IP configuration | ip addr/route | "show network config" |

---

## Argument Format

Construct a structured argument string:

```
<symptom/goal> on <target> - protocol: <if relevant>, interface: <if relevant>, context: <prior findings>
```

**Include only relevant components.**

### Examples

**Can't reach host:**
```
test connectivity to 10.0.1.50 - timeout after 3 hops, VPN active
```

**DNS debugging:**
```
resolve DNS for api.example.com - returns NXDOMAIN, worked yesterday
```

**Port scan:**
```
check ports 80,443,8080 on staging.example.com - need service versions
```

**Traffic capture:**
```
capture HTTP traffic on eth0 - filter for host 10.0.1.50 port 8080
```

**SSH tunnel:**
```
create local tunnel to access remote postgres - local:5432 to db.internal:5432 via jump.example.com
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-network-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Target host/IP is unclear from context
- Multiple network issues described -- prioritize which to investigate
- Production traffic capture -- confirm acceptable before sniffing

**Use Read tool first if:**
- Need to check /etc/resolv.conf or /etc/hosts
- Need to verify network config files mentioned in conversation
