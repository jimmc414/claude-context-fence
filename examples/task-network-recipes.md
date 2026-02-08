---
description: "Network diagnostics recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-network with target, symptom, and context included"
when-to-use: "Receives prepared arguments from task-network with target, symptom, and context included"
context: fork
model: inherit
allowed-tools: Read
---

# Network Diagnostics Recipes

**Tools:** ping, traceroute, tracepath, mtr, dig, nslookup, host, whois, ss, ip, nmap, nc, ncat, tcpdump, tshark, iptables, nft, ufw, ssh, autossh, sshuttle

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Test connectivity | `ping -c 4 host` |
| Trace route with AS | `mtr -z host` |
| DNS lookup | `dig +short host` |
| All DNS records | `dig host ANY +noall +answer` |
| Reverse DNS | `dig -x IP` |
| Listening ports | `ss -tuln` |
| Connections with process | `ss -tunp` |
| Quick port check | `nc -zv host 80` |
| Scan common ports | `nmap host` |
| Capture packets | `tcpdump -i eth0 -w capture.pcap` |
| Local forward | `ssh -L 8080:remote:80 user@jump` |
| Show routes | `ip route` |
| Show interfaces | `ip addr` |
| Firewall rules | `sudo iptables -L -n -v` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| Basic connectivity test | ping | Pre-installed, simple |
| Network path diagnosis | mtr | Combined ping + traceroute, live |
| Full route trace | traceroute | Shows each hop |
| DNS lookup | dig | Most detailed, scriptable |
| Quick DNS check | host / nslookup | Simpler output |
| List listening ports | ss -tuln | Modern replacement for netstat |
| List connections with PIDs | ss -tulnp | Needs root for PID info |
| Port scan (remote) | nmap | Feature-rich network scanner |
| Quick port test | nc -zv | Single port, no install needed |
| Packet capture | tcpdump | Lightweight, scriptable |
| Deep packet inspection | tshark | Wireshark CLI, protocol decode |
| Persistent SSH tunnel | autossh | Auto-reconnecting SSH |
| Low-latency remote shell | mosh | Handles roaming/intermittent |

---

## Connectivity Testing

### ping

```bash
# Basic connectivity
ping -c 4 host                        # 4 packets
ping -c 10 -i 0.2 host               # 10 packets, 200ms interval
ping -W 2 -c 3 host                  # 2-second timeout per packet
ping -q -c 100 host                  # Quiet mode, summary only
ping -D -c 10 host                   # Print timestamps before each line

# Packet size and fragmentation
ping -s 1472 host                    # Large packets (1472 + 28 header = 1500 MTU)
ping -s 1472 -M do host             # Don't fragment -- test MTU
ping -s 8192 host                    # Jumbo frame test

# Interface and TTL control
ping -I eth0 host                    # Use specific interface
ping -I 192.168.1.100 host          # Use specific source IP
ping -t 5 host                      # Set TTL to 5 (limits hops)

# Flood and adaptive
sudo ping -f host                    # Flood ping (requires root, one dot per lost packet)
sudo ping -f -c 1000 host           # Flood 1000 packets
ping -A host                        # Adaptive: adjust interval to RTT

# Audible and pattern
ping -a host                        # Beep on each reply
ping -p ff host                     # Fill payload with pattern (hex)
```

### traceroute

```bash
# Basic route tracing
traceroute host                      # UDP probes (default)
traceroute -I host                   # ICMP echo probes
traceroute -T host                   # TCP SYN probes (good through firewalls)
traceroute -T -p 443 host           # TCP SYN to port 443

# Options
traceroute -n host                   # No DNS resolution (faster)
traceroute -m 30 host               # Max 30 hops (default)
traceroute -w 3 host                # 3-second wait per probe
traceroute -q 1 host                # 1 probe per hop (default 3, faster)
traceroute -s 192.168.1.100 host    # Set source address
traceroute -f 5 host                # Start from hop 5

# Port-specific (bypass firewalls)
traceroute -T -p 80 host            # TCP to port 80
traceroute -p 33434 host            # Start from UDP port 33434
```

### tracepath

```bash
# MTU discovery (no root required)
tracepath host                       # Discover path MTU
tracepath -n host                    # No DNS resolution
tracepath -b host                    # Show both hostname and IP
tracepath -l 1500 host              # Set initial packet length
```

### mtr (My Traceroute)

```bash
# Interactive mode (default)
mtr host                             # Live updating display

# Report mode (non-interactive)
mtr -r host                         # Report mode, 10 cycles
mtr -r -c 100 host                  # Report mode, 100 cycles
mtr -r -c 50 -n host               # 50 cycles, no DNS
mtr -r -w host                     # Wide report (full hostnames)

# Protocol selection
mtr --tcp host                      # TCP SYN probes
mtr --tcp -P 443 host              # TCP to port 443
mtr --udp host                     # UDP probes
mtr --udp -P 53 host              # UDP to port 53

# Display options
mtr -z host                        # Show AS numbers
mtr -z -b host                    # AS numbers + both hostname and IP
mtr -n host                        # Numeric only (no DNS)

# Output formats
mtr -r --json host                 # JSON output
mtr -r --xml host                  # XML output
mtr -r --csv host                  # CSV output

# Packet options
mtr -s 1472 host                   # Set packet size
mtr -i 0.5 host                   # 0.5 second interval between probes
mtr -m 25 host                    # Max 25 hops
```

---

## DNS

### dig

#### Basic Queries

```bash
# Standard lookups
dig host                             # A record (default)
dig +short host                     # Just the IP address
dig +noall +answer host             # Clean answer section only
dig +noall +answer +ttlid host     # Answer with TTL values

# Specific record types
dig host A                          # IPv4 address
dig host AAAA                       # IPv6 address
dig host MX                         # Mail exchange
dig host NS                         # Name servers
dig host TXT                        # TXT records (SPF, DKIM, etc.)
dig host SOA                        # Start of authority
dig host SOA +multiline            # SOA in readable multi-line format
dig host SRV                        # Service records
dig host CAA                        # Certificate authority authorization
dig host NAPTR                      # Name authority pointer
dig host PTR                        # Pointer record
dig host CNAME                      # Canonical name
dig host ANY +noall +answer        # All record types (may be limited by server)
```

#### Reverse Lookups

```bash
dig -x 8.8.8.8                     # Reverse lookup: IP to hostname
dig -x 2001:4860:4860::8888       # IPv6 reverse lookup
dig +short -x 10.0.1.50           # Quick reverse lookup
```

#### Server Selection

```bash
dig @8.8.8.8 host                  # Query Google DNS
dig @1.1.1.1 host                  # Query Cloudflare DNS
dig @8.8.4.4 host                  # Query Google secondary
dig @ns1.example.com host         # Query specific authoritative server
dig @127.0.0.1 host               # Query local resolver
dig @::1 host                     # Query local resolver via IPv6
```

#### Advanced Options

```bash
# Tracing
dig +trace host                    # Full delegation trace from root
dig +trace +nodnssec host         # Trace without DNSSEC records

# DNSSEC
dig +dnssec host                   # Request DNSSEC records
dig +dnssec +cd host              # DNSSEC with checking disabled
dig host DNSKEY +dnssec           # Get DNSSEC keys
dig host DS +dnssec               # Get delegation signer record

# Protocol options
dig +tcp host                      # Force TCP (instead of UDP)
dig +notcp host                   # Force UDP
dig +recurse host                 # Enable recursion (default)
dig +norecurse host               # Disable recursion (query authoritative only)
dig +bufsize=4096 host            # Set EDNS buffer size

# Output control
dig +stats host                    # Include query statistics
dig +nostats host                  # Suppress statistics
dig +all host                     # All sections (question, answer, authority, additional)
dig +qr host                      # Show outgoing query

# Batch mode
dig -f domains.txt                 # Query all domains from file
dig -f domains.txt +short         # Batch with short output
```

#### Zone Transfer

```bash
# AXFR (full zone transfer -- usually restricted)
dig @ns1.example.com example.com AXFR
dig @ns1.example.com example.com AXFR +noall +answer

# IXFR (incremental zone transfer)
dig @ns1.example.com example.com IXFR=2024010100
```

#### DNS Debugging Workflows

```bash
# Trace full resolution path
dig +trace +nodnssec example.com

# Check all authoritative servers for consistency
for ns in $(dig +short example.com NS); do
  echo "=== $ns ===" && dig @$ns example.com +short
done

# Compare public resolvers
for dns in 8.8.8.8 1.1.1.1 9.9.9.9 208.67.222.222; do
  echo "=== $dns ===" && dig @$dns example.com +short
done

# Check propagation after DNS change
for ns in $(dig +short example.com NS); do
  echo "=== $ns ===" && dig @$ns example.com +noall +answer
done

# Check TTL countdown (run multiple times)
dig +noall +answer +ttlid example.com

# Verify CNAME chain
dig +trace +nodnssec www.example.com CNAME
```

### nslookup

```bash
nslookup host                      # Basic lookup
nslookup host 8.8.8.8            # Use specific DNS server
nslookup -type=mx host           # MX records
nslookup -type=ns host           # NS records
nslookup -type=txt host          # TXT records
nslookup -type=soa host          # SOA record
nslookup -type=any host          # All records
```

### host

```bash
host hostname                      # Basic lookup
host -t mx hostname               # MX records
host -t ns hostname               # NS records
host -t txt hostname              # TXT records
host -t soa hostname              # SOA record
host -a hostname                  # All record types (verbose)
host -l domain ns1.domain        # Zone transfer attempt
host 8.8.8.8                     # Reverse lookup
host -v hostname                  # Verbose output
```

### whois

```bash
whois example.com                  # Domain registration info
whois 8.8.8.8                    # IP allocation info
whois -h whois.arin.net 8.8.8.8 # Query specific WHOIS server
whois AS15169                     # AS number lookup
whois --verbose example.com       # Verbose output
```

---

## Socket & Connection Inspection

### ss (Socket Statistics)

#### Basic Usage

```bash
# Listening sockets
ss -tuln                           # TCP/UDP listening, numeric
ss -tulnp                         # + process info (requires root for other users)
ss -tln                           # TCP listening only
ss -uln                           # UDP listening only

# All connections
ss -tun                           # TCP/UDP established, numeric
ss -tunp                          # + process info
ss -tan                           # All TCP states, numeric
ss -s                             # Socket summary statistics
```

#### State Filters

```bash
ss state established              # All established connections
ss state listening                # All listening sockets
ss state time-wait                # TIME_WAIT connections
ss state close-wait               # CLOSE_WAIT connections (often a leak)
ss state syn-sent                 # Outgoing connections in progress
ss state syn-recv                 # Incoming connections in progress
ss state fin-wait-1               # Closing connections
ss state fin-wait-2               # Half-closed connections
ss state last-ack                 # Final close acknowledgment pending
ss state closing                  # Both sides closing simultaneously

# Exclude states
ss state established exclude close-wait
```

#### Socket Expressions (Filters)

```bash
# Filter by port
ss -tn 'sport = :80'             # Source port 80
ss -tn 'dport = :443'            # Destination port 443
ss -tn 'sport = :80 or sport = :443'  # Port 80 or 443
ss -tn '( sport = :80 or sport = :443 )'

# Filter by address
ss -tn 'dst 10.0.0.0/8'         # Destination in 10.x.x.x range
ss -tn 'src 192.168.1.0/24'     # Source in 192.168.1.x range
ss -tn 'dst 10.0.1.50'          # Specific destination

# Combined filters
ss -tn 'dst 10.0.1.50 and dport = :443'
ss -tn state established 'dport = :80'
ss -tn 'sport >= :1024 and sport <= :2048'
```

#### Extended Info

```bash
ss -tunei                         # Extended info + internal TCP info
ss -tunp -o                       # + timer information
ss -tunp -m                       # + memory usage per socket
ss -tunp -i                       # + internal TCP info (RTT, cwnd, retransmits)
ss -tn -o state time-wait         # TIME_WAIT with timer info
```

#### Counting and Summarizing

```bash
# Count connections by state
ss -tan state established | wc -l
ss -tan state time-wait | wc -l
ss -tan state close-wait | wc -l

# Connections per remote IP
ss -tn state established | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -20

# Connections per local port
ss -tln | awk 'NR>1 {print $4}' | rev | cut -d: -f1 | rev | sort | uniq -c | sort -rn

# All states summary
ss -s
```

# Note: netstat is deprecated; use ss instead (shown above). See task-process for comprehensive lsof usage.

### /proc/net Inspection

```bash
# Raw TCP connections (hex encoded)
cat /proc/net/tcp                  # IPv4 TCP
cat /proc/net/tcp6                 # IPv6 TCP
cat /proc/net/udp                  # IPv4 UDP
cat /proc/net/udp6                 # IPv6 UDP

# Decode connection states (field 4 in /proc/net/tcp)
# 01=ESTABLISHED 02=SYN_SENT 03=SYN_RECV 04=FIN_WAIT1
# 05=FIN_WAIT2 06=TIME_WAIT 07=CLOSE 08=CLOSE_WAIT
# 09=LAST_ACK 0A=LISTEN 0B=CLOSING

# Network statistics
cat /proc/net/dev                  # Interface statistics
cat /proc/net/snmp                 # Protocol statistics (IP, ICMP, TCP, UDP)
cat /proc/net/sockstat             # Socket usage summary
```

### conntrack (NAT Connection Tracking)

```bash
# List tracked connections
sudo conntrack -L                  # All tracked connections
sudo conntrack -L -p tcp          # TCP connections only
sudo conntrack -L -p udp          # UDP connections only
sudo conntrack -L --dst-nat       # DNAT'd connections
sudo conntrack -L -s 10.0.1.50   # Connections from specific source

# Statistics
sudo conntrack -S                  # Connection tracking stats
sudo conntrack -C                  # Count of tracked connections

# Monitor events
sudo conntrack -E                  # Watch new/destroyed connections in real-time
sudo conntrack -E -p tcp          # Watch TCP events only

# Flush
sudo conntrack -F                  # Flush all tracked connections
sudo conntrack -D -s 10.0.1.50   # Delete entries from specific source
```

---

## Port Scanning

### nmap

#### Scan Types

```bash
# TCP scans
nmap host                          # Default: SYN scan (root) or connect scan
nmap -sT host                     # TCP connect scan (no root needed)
nmap -sS host                     # TCP SYN stealth scan (requires root)
nmap -sA host                     # TCP ACK scan (detect firewall rules)
nmap -sW host                     # TCP Window scan (detect open behind firewall)
nmap -sN host                     # TCP NULL scan (no flags set)
nmap -sF host                     # TCP FIN scan
nmap -sX host                     # TCP Xmas scan (FIN+PSH+URG)

# UDP scan
nmap -sU host                     # UDP scan (slow, requires root)
nmap -sU -p 53,123,161 host      # UDP scan specific ports
nmap -sU --top-ports 20 host     # Top 20 UDP ports

# Combined TCP+UDP
nmap -sS -sU host                # SYN + UDP scan
```

#### Port Specification

```bash
nmap -p 22 host                   # Single port
nmap -p 22,80,443 host           # Multiple ports
nmap -p 1-1024 host              # Port range
nmap -p- host                    # All 65535 ports
nmap -p 1-65535 host             # Same as -p-
nmap --top-ports 100 host        # Top 100 common ports
nmap --top-ports 1000 host       # Top 1000 common ports
nmap -p T:80,443,U:53,161 host  # Specify TCP and UDP ports separately
```

#### Service and OS Detection

```bash
nmap -sV host                     # Service version detection
nmap -sV --version-intensity 5 host  # More aggressive version probing
nmap -O host                      # OS detection (requires root)
nmap -A host                      # Aggressive: OS + version + scripts + traceroute
```

#### Timing and Performance

```bash
nmap -T0 host                     # Paranoid (IDS evasion, very slow)
nmap -T1 host                     # Sneaky
nmap -T2 host                     # Polite (reduced bandwidth)
nmap -T3 host                     # Normal (default)
nmap -T4 host                     # Aggressive (faster, reliable networks)
nmap -T5 host                     # Insane (fastest, may miss ports)
nmap --min-rate 1000 host        # Minimum 1000 packets/sec
nmap --max-retries 2 host        # Limit retries per probe
```

#### Target Specification

```bash
nmap 10.0.1.0/24                  # CIDR notation
nmap 10.0.1.1-50                  # IP range
nmap 10.0.1.1,2,3                # Multiple IPs
nmap -iL targets.txt             # Read targets from file
nmap --exclude 10.0.1.1 10.0.1.0/24  # Exclude specific hosts
nmap -sn 10.0.1.0/24             # Ping sweep (host discovery only)
nmap -Pn host                    # Skip ping, scan directly (assume host is up)
```

#### NSE Scripts

```bash
nmap --script=default host        # Default scripts
nmap --script=vuln host           # Vulnerability detection scripts
nmap --script=http-enum host      # HTTP directory enumeration
nmap --script=ssl-enum-ciphers -p 443 host  # SSL/TLS cipher enumeration
nmap --script=http-headers -p 80 host  # HTTP headers
nmap --script=dns-brute host      # DNS subdomain brute force
nmap --script=smb-os-discovery -p 445 host  # SMB OS detection
nmap --script=banner host         # Grab service banners
```

#### Output Formats

```bash
nmap -oN scan.txt host            # Normal output to file
nmap -oX scan.xml host            # XML output
nmap -oG scan.gnmap host          # Grepable output
nmap -oA scan host                # All formats (scan.nmap, scan.xml, scan.gnmap)
```

#### Common Scan Recipes

```bash
# Quick scan (top 100 ports, fast)
nmap -T4 --top-ports 100 host

# Full port scan with versions
nmap -sS -sV -p- -T4 host

# Service enumeration
nmap -sV -sC -p 22,80,443,3306,5432,6379,8080 host

# Vulnerability scan
nmap -sV --script=vuln host

# Stealth scan
nmap -sS -T2 -f -D RND:5 host

# Network discovery
nmap -sn -PE -PA21,23,80,3389 10.0.1.0/24
```

### nc / ncat (Netcat)

```bash
# Port checking
nc -zv host 80                    # Test single port
nc -zv host 20-100                # Test port range
nc -zv -w 3 host 443             # 3-second timeout
nc -zvu host 53                   # Test UDP port

# Banner grabbing
nc -v host 80                     # Connect and type HTTP request
echo "HEAD / HTTP/1.0\r\n\r\n" | nc host 80  # Grab HTTP banner
nc -v host 22                     # Grab SSH banner
nc -v host 25                     # Grab SMTP banner

# Listen mode
nc -l -p 8080                     # Listen on port 8080
nc -l -p 8080 -k                 # Keep listening after disconnect
ncat -l -p 8080 --keep-open      # ncat persistent listener

# File transfer
# Receiver:
nc -l -p 9999 > received_file
# Sender:
nc host 9999 < file_to_send

# Simple chat
# Host A:
nc -l -p 12345
# Host B:
nc hostA 12345

# Port forwarding (ncat)
ncat -l -p 8080 --sh-exec "ncat remote_host 80"

# Proxy
ncat -l -p 8080 --proxy-type http

# UDP
nc -u host 53                     # UDP connection
nc -lu -p 5000                    # UDP listener
echo "test" | nc -u -w 1 host 514  # Send UDP syslog message
```

---

## Traffic Capture

### tcpdump

#### Basic Capture

```bash
# Capture on interface
sudo tcpdump -i eth0              # Capture on eth0
sudo tcpdump -i any               # Capture on all interfaces
sudo tcpdump -i lo                # Capture loopback traffic

# Output control
sudo tcpdump -n -i eth0          # No DNS resolution (faster)
sudo tcpdump -nn -i eth0         # No DNS or port name resolution
sudo tcpdump -v -i eth0          # Verbose
sudo tcpdump -vv -i eth0         # More verbose
sudo tcpdump -vvv -i eth0        # Maximum verbosity
sudo tcpdump -c 100 -i eth0     # Capture 100 packets then stop
sudo tcpdump -q -i eth0          # Quiet (less protocol info)
```

#### File Operations

```bash
# Write to file
sudo tcpdump -i eth0 -w capture.pcap       # Save to pcap file
sudo tcpdump -i eth0 -w capture.pcap -c 10000  # Save 10000 packets

# Read from file
tcpdump -r capture.pcap                     # Read pcap file
tcpdump -r capture.pcap -n                  # Read without DNS resolution
tcpdump -r capture.pcap -c 50              # Read first 50 packets
tcpdump -r capture.pcap 'port 80'          # Filter while reading

# Ring buffer (rotate files)
sudo tcpdump -i eth0 -w capture.pcap -C 100 -W 10  # 10 files of 100MB each
sudo tcpdump -i eth0 -w capture.pcap -G 3600 -W 24  # Rotate hourly, keep 24 files
```

#### Snap Length

```bash
sudo tcpdump -i eth0 -s 0                  # Capture full packets (default on modern)
sudo tcpdump -i eth0 -s 96                 # Only first 96 bytes (headers only)
sudo tcpdump -i eth0 -s 1500              # Full Ethernet frame
```

#### Timestamp Formats

```bash
sudo tcpdump -tttt -i eth0                 # Full date and time
sudo tcpdump -ttt -i eth0                  # Delta between packets
sudo tcpdump -tt -i eth0                   # Unix timestamp
sudo tcpdump -t -i eth0                    # No timestamp
```

#### Packet Content Display

```bash
sudo tcpdump -X -i eth0                    # Hex + ASCII
sudo tcpdump -XX -i eth0                   # Hex + ASCII including link-level header
sudo tcpdump -A -i eth0                    # ASCII only (good for HTTP)
sudo tcpdump -x -i eth0                    # Hex only
```

#### BPF Filters

```bash
# Host filters
sudo tcpdump -i eth0 host 10.0.1.50       # Traffic to/from host
sudo tcpdump -i eth0 src host 10.0.1.50   # Traffic from host
sudo tcpdump -i eth0 dst host 10.0.1.50   # Traffic to host
sudo tcpdump -i eth0 net 10.0.1.0/24      # Traffic to/from subnet

# Port filters
sudo tcpdump -i eth0 port 80              # Port 80 (src or dst)
sudo tcpdump -i eth0 src port 80          # Source port 80
sudo tcpdump -i eth0 dst port 443         # Destination port 443
sudo tcpdump -i eth0 portrange 8000-9000  # Port range

# Protocol filters
sudo tcpdump -i eth0 tcp                   # TCP only
sudo tcpdump -i eth0 udp                   # UDP only
sudo tcpdump -i eth0 icmp                  # ICMP only
sudo tcpdump -i eth0 arp                   # ARP only

# Combined filters
sudo tcpdump -i eth0 'host 10.0.1.50 and port 80'
sudo tcpdump -i eth0 'src 10.0.1.50 and dst port 443'
sudo tcpdump -i eth0 'port 80 or port 443'
sudo tcpdump -i eth0 'not port 22'        # Exclude SSH
sudo tcpdump -i eth0 'not (port 22 or port 53)'  # Exclude SSH and DNS

# TCP flag filters
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'     # SYN packets
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-rst != 0'     # RST packets
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-fin != 0'     # FIN packets
sudo tcpdump -i eth0 'tcp[tcpflags] = tcp-syn'          # SYN only (no ACK)
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn|tcp-ack) = tcp-syn'  # SYN without ACK
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0'       # SYN or FIN
```

#### Common Capture Recipes

```bash
# HTTP traffic (request/response content)
sudo tcpdump -i eth0 -A -s 0 'port 80'

# DNS queries and responses
sudo tcpdump -i eth0 -nn 'port 53'

# TLS/SSL handshake
sudo tcpdump -i eth0 -nn 'tcp port 443 and (tcp[((tcp[12:1] & 0xf0) >> 2):1] = 0x16)'

# SYN packets only (new connections)
sudo tcpdump -i eth0 -nn 'tcp[tcpflags] = tcp-syn'

# RST packets (connection resets)
sudo tcpdump -i eth0 -nn 'tcp[tcpflags] & tcp-rst != 0'

# ICMP (ping, unreachable, etc.)
sudo tcpdump -i eth0 -nn icmp

# ARP traffic
sudo tcpdump -i eth0 -nn arp

# Traffic between two hosts
sudo tcpdump -i eth0 'host 10.0.1.50 and host 10.0.1.100'

# Large packets (possible fragmentation issues)
sudo tcpdump -i eth0 'greater 1400'

# Capture only headers (save space)
sudo tcpdump -i eth0 -s 68 -w headers.pcap
```

### tshark (Wireshark CLI)

```bash
# Basic capture
sudo tshark -i eth0                        # Capture on interface
sudo tshark -i eth0 -c 100               # Capture 100 packets
sudo tshark -i eth0 -a duration:30       # Capture for 30 seconds

# Display filters (Wireshark syntax)
tshark -r capture.pcap -Y 'http.request'              # HTTP requests
tshark -r capture.pcap -Y 'dns.flags.response == 1'   # DNS responses
tshark -r capture.pcap -Y 'tcp.flags.syn == 1 and tcp.flags.ack == 0'  # SYN only
tshark -r capture.pcap -Y 'ip.addr == 10.0.1.50'     # Specific host
tshark -r capture.pcap -Y 'tcp.port == 443'           # HTTPS traffic
tshark -r capture.pcap -Y 'http.response.code >= 400' # HTTP errors

# Field extraction
tshark -r capture.pcap -T fields -e ip.src -e ip.dst -e tcp.dstport  # Specific fields
tshark -r capture.pcap -T fields -e http.host -e http.request.uri -Y 'http.request'  # HTTP URLs
tshark -r capture.pcap -T fields -e dns.qry.name -Y 'dns.flags.response == 0'  # DNS queries
tshark -r capture.pcap -T fields -e ip.src -e ip.dst -E separator=, -E quote=d  # CSV format

# JSON output
tshark -r capture.pcap -T json -Y 'http.request' | jq '.'

# Statistics
tshark -r capture.pcap -z conv,tcp                    # TCP conversations
tshark -r capture.pcap -z conv,ip                     # IP conversations
tshark -r capture.pcap -z io,stat,1                   # Packets per second
tshark -r capture.pcap -z io,stat,1,"COUNT(tcp.analysis.retransmission)"  # Retransmits/sec
tshark -r capture.pcap -z endpoints,ip               # IP endpoints
tshark -r capture.pcap -z http,tree                   # HTTP request tree
tshark -r capture.pcap -z dns,tree                    # DNS query tree

# Follow stream
tshark -r capture.pcap -z follow,tcp,ascii,0          # Follow first TCP stream
tshark -r capture.pcap -z follow,http,ascii,0         # Follow first HTTP stream

# Live capture with display filter
sudo tshark -i eth0 -Y 'http.request.method == "POST"'
sudo tshark -i eth0 -Y 'dns' -T fields -e dns.qry.name
```

---

## Firewall

### iptables

#### Listing Rules

```bash
sudo iptables -L -n -v                    # List all filter rules (verbose, numeric)
sudo iptables -L -n -v --line-numbers     # With line numbers
sudo iptables -t nat -L -n -v             # List NAT rules
sudo iptables -t mangle -L -n -v          # List mangle rules
sudo iptables -t raw -L -n -v             # List raw rules
sudo iptables -S                           # List rules as iptables commands
sudo iptables -S INPUT                     # List INPUT chain commands only
```

#### Basic Rules

```bash
# Allow incoming SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow incoming HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Drop all other incoming
sudo iptables -A INPUT -j DROP

# Allow outgoing
sudo iptables -A OUTPUT -j ACCEPT

# Deny specific IP
sudo iptables -A INPUT -s 10.0.1.100 -j DROP

# Deny specific IP to specific port
sudo iptables -A INPUT -s 10.0.1.100 -p tcp --dport 80 -j REJECT
```

#### Advanced Match Extensions

```bash
# Multiple ports
sudo iptables -A INPUT -p tcp -m multiport --dports 80,443,8080 -j ACCEPT

# Rate limiting (prevent brute force)
sudo iptables -A INPUT -p tcp --dport 22 -m limit --limit 3/min --limit-burst 5 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j DROP

# Recent module (block repeated failed connections)
sudo iptables -A INPUT -p tcp --dport 22 -m recent --set --name SSH
sudo iptables -A INPUT -p tcp --dport 22 -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP

# Connection state
sudo iptables -A INPUT -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# Comment
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT -m comment --comment "Allow HTTP"

# IP range
sudo iptables -A INPUT -m iprange --src-range 10.0.1.100-10.0.1.200 -j ACCEPT

# Time-based
sudo iptables -A INPUT -p tcp --dport 80 -m time --timestart 08:00 --timestop 18:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
```

#### NAT and Port Forwarding

```bash
# Enable IP forwarding
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# SNAT (masquerade outgoing traffic)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -s 10.0.1.0/24 -o eth0 -j MASQUERADE

# DNAT (port forwarding)
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 10.0.1.50:80
sudo iptables -A FORWARD -p tcp -d 10.0.1.50 --dport 80 -j ACCEPT

# Redirect (local port redirect)
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

#### Logging

```bash
# Log dropped packets
sudo iptables -A INPUT -j LOG --log-prefix "IPT-DROP: " --log-level 4
sudo iptables -A INPUT -j DROP

# Log specific traffic
sudo iptables -A INPUT -p tcp --dport 22 -j LOG --log-prefix "SSH-ATTEMPT: "

# Rate-limited logging (prevent log flooding)
sudo iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "IPT-INPUT: "
```

#### Save and Restore

```bash
# Save current rules
sudo iptables-save > /etc/iptables/rules.v4
sudo ip6tables-save > /etc/iptables/rules.v6

# Restore rules
sudo iptables-restore < /etc/iptables/rules.v4
sudo ip6tables-restore < /etc/iptables/rules.v6

# Flush all rules (careful!)
sudo iptables -F                           # Flush all chains
sudo iptables -t nat -F                    # Flush NAT rules
sudo iptables -X                           # Delete user-defined chains
sudo iptables -P INPUT ACCEPT             # Reset default policies
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
```

### nftables (iptables successor)

```bash
# List all rules
sudo nft list ruleset
sudo nft list tables
sudo nft list table inet filter

# Create table and chain
sudo nft add table inet filter
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'
sudo nft add chain inet filter forward '{ type filter hook forward priority 0; policy drop; }'
sudo nft add chain inet filter output '{ type filter hook output priority 0; policy accept; }'

# Add rules
sudo nft add rule inet filter input ct state established,related accept
sudo nft add rule inet filter input iifname lo accept
sudo nft add rule inet filter input tcp dport 22 accept
sudo nft add rule inet filter input tcp dport { 80, 443 } accept
sudo nft add rule inet filter input ip saddr 10.0.1.0/24 accept
sudo nft add rule inet filter input counter drop

# Delete rule (by handle)
sudo nft -a list chain inet filter input   # Show handles
sudo nft delete rule inet filter input handle 5

# Save and load
sudo nft list ruleset > /etc/nftables.conf
sudo nft -f /etc/nftables.conf

# Flush
sudo nft flush ruleset
```

### ufw (Uncomplicated Firewall)

```bash
# Enable/disable
sudo ufw enable
sudo ufw disable
sudo ufw reset                             # Reset to defaults

# Status
sudo ufw status                            # Brief status
sudo ufw status verbose                    # Verbose status
sudo ufw status numbered                   # Numbered rules

# Allow/deny
sudo ufw allow 22                          # Allow SSH
sudo ufw allow 80/tcp                      # Allow HTTP TCP
sudo ufw allow 443                         # Allow HTTPS
sudo ufw allow from 10.0.1.0/24           # Allow subnet
sudo ufw allow from 10.0.1.50 to any port 22  # Allow IP to SSH
sudo ufw allow proto tcp from any to any port 80,443  # Allow HTTP+HTTPS

sudo ufw deny 23                           # Deny telnet
sudo ufw deny from 10.0.1.100             # Deny specific IP
sudo ufw reject 23                         # Reject (sends ICMP unreachable)

# Delete rules
sudo ufw delete allow 80                   # Delete by rule
sudo ufw delete 3                          # Delete by number

# Application profiles
sudo ufw app list                          # List known apps
sudo ufw allow 'OpenSSH'                   # Allow by app name
sudo ufw allow 'Nginx Full'               # Allow Nginx HTTP+HTTPS

# Logging
sudo ufw logging on                        # Enable logging
sudo ufw logging medium                    # Set log level (low/medium/high/full)
```

---

## SSH

### Basic Connections and Tunneling

```bash
# Basic connection
ssh user@host                              # Connect
ssh -p 2222 user@host                     # Custom port
ssh -v user@host                          # Verbose (debug connection)
ssh -vv user@host                         # More verbose
ssh -vvv user@host                        # Maximum verbosity
```

### Local Port Forwarding (-L)

```bash
# Forward local port to remote service
ssh -L 8080:localhost:80 user@host        # Access remote :80 via local :8080
ssh -L 3306:db.internal:3306 user@jump   # Access internal DB via jump host
ssh -L 5432:localhost:5432 user@host     # Forward PostgreSQL
ssh -L 6379:redis.internal:6379 user@jump  # Forward Redis via jump

# Multiple forwards
ssh -L 3306:db:3306 -L 6379:redis:6379 user@jump

# Bind to all interfaces (not just localhost)
ssh -L 0.0.0.0:8080:remote:80 user@host

# Background tunnel (no shell)
ssh -f -N -L 8080:localhost:80 user@host
```

### Remote Port Forwarding (-R)

```bash
# Expose local service to remote
ssh -R 8080:localhost:80 user@host        # Remote :8080 -> local :80
ssh -R 9090:localhost:3000 user@host     # Expose local dev server

# Background tunnel
ssh -f -N -R 8080:localhost:80 user@host

# Bind to all interfaces on remote (requires GatewayPorts yes in sshd)
ssh -R 0.0.0.0:8080:localhost:80 user@host
```

### Dynamic Port Forwarding (SOCKS Proxy)

```bash
# SOCKS proxy
ssh -D 1080 user@host                    # SOCKS5 proxy on local :1080
ssh -D 0.0.0.0:1080 user@host           # SOCKS5 on all interfaces
ssh -f -N -D 1080 user@host             # Background SOCKS proxy

# Use with curl
curl --socks5-hostname localhost:1080 http://internal.example.com

# Use with browser: set SOCKS5 proxy to localhost:1080
```

### ProxyJump (Bastion/Jump Host)

```bash
# Jump through bastion
ssh -J bastion user@internal             # Single jump
ssh -J bastion1,bastion2 user@target    # Multiple jumps

# Old-style ProxyCommand
ssh -o ProxyCommand="ssh -W %h:%p bastion" user@internal
```

### Connection Multiplexing

```bash
# ~/.ssh/config multiplexing
# Host *
#     ControlMaster auto
#     ControlPath ~/.ssh/sockets/%r@%h-%p
#     ControlPersist 600

# Manual control
ssh -M -S /tmp/ssh-mux user@host        # Create master connection
ssh -S /tmp/ssh-mux user@host           # Reuse existing master
ssh -S /tmp/ssh-mux -O check user@host  # Check master status
ssh -S /tmp/ssh-mux -O exit user@host   # Close master connection
ssh -S /tmp/ssh-mux -O forward -L 8080:localhost:80 user@host  # Add forward to existing
```

### Escape Sequences

```bash
# While in SSH session, press Enter then:
# ~.    Disconnect (kill hung session)
# ~C    Open command line (add port forwards on the fly)
# ~?    List available escapes
# ~#    List forwarded connections
# ~&    Background SSH when waiting for connections to close
# ~~    Send literal ~
```

### SSH Config File (~/.ssh/config)

```bash
# Example config entries
# Host dev
#     HostName dev.example.com
#     User deploy
#     Port 2222
#     IdentityFile ~/.ssh/dev_key
#     ForwardAgent yes
#
# Host internal-*
#     ProxyJump bastion
#     User admin
#
# Host bastion
#     HostName bastion.example.com
#     User jump
#     IdentityFile ~/.ssh/bastion_key
#     LocalForward 3306 db.internal:3306
#
# Host *
#     ServerAliveInterval 60
#     ServerAliveCountMax 3
#     TCPKeepAlive yes
```

### autossh (Persistent Tunnels)

```bash
# Persistent local forward (reconnects on failure)
autossh -M 20000 -f -N -L 8080:localhost:80 user@host

# Persistent SOCKS proxy
autossh -M 20000 -f -N -D 1080 user@host

# Persistent reverse tunnel
autossh -M 20000 -f -N -R 9090:localhost:3000 user@host

# Without monitoring port (use ServerAlive instead)
autossh -M 0 -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3" -f -N -L 8080:localhost:80 user@host
```

### sshuttle (VPN over SSH)

```bash
# Route subnet through SSH
sshuttle -r user@host 10.0.0.0/8         # Route 10.x via SSH
sshuttle -r user@host 0/0                # Route ALL traffic via SSH
sshuttle -r user@host 10.0.0.0/8 172.16.0.0/12  # Multiple subnets

# DNS forwarding
sshuttle --dns -r user@host 0/0          # Route DNS through tunnel too

# Exclude subnets
sshuttle -r user@host 10.0.0.0/8 -x 10.0.1.0/24  # Exclude specific subnet

# Daemon mode
sshuttle -D -r user@host 10.0.0.0/8     # Run as daemon
```

### SSH Key Management

```bash
# Generate keys
ssh-keygen -t ed25519 -C "user@host"     # Ed25519 (recommended)
ssh-keygen -t rsa -b 4096 -C "user@host" # RSA 4096-bit
ssh-keygen -t ed25519 -f ~/.ssh/mykey    # Specify output file
ssh-keygen -p -f ~/.ssh/mykey            # Change passphrase

# Copy public key to server
ssh-copy-id user@host                     # Copy default key
ssh-copy-id -i ~/.ssh/mykey.pub user@host  # Copy specific key

# Agent management
eval $(ssh-agent)                         # Start agent
ssh-add                                   # Add default key
ssh-add ~/.ssh/mykey                     # Add specific key
ssh-add -l                               # List loaded keys
ssh-add -D                               # Remove all keys
ssh-add -t 3600 ~/.ssh/mykey            # Add key with 1-hour timeout
```

---

## IP Configuration

### ip addr

```bash
ip addr                                    # Show all interfaces
ip addr show eth0                          # Show specific interface
ip -4 addr                                 # IPv4 only
ip -6 addr                                 # IPv6 only
ip addr show up                            # Only interfaces that are up
ip -br addr                                # Brief format (compact)

# Add/remove addresses
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr add 192.168.1.101/24 dev eth0 label eth0:1  # Alias
sudo ip addr del 192.168.1.100/24 dev eth0
sudo ip addr flush dev eth0               # Remove all addresses
```

### ip link

```bash
ip link                                    # Show all interfaces
ip link show eth0                          # Show specific interface
ip -br link                                # Brief format
ip -s link                                 # Statistics (TX/RX packets, errors)
ip -s -s link show eth0                   # Detailed statistics

# Interface control
sudo ip link set eth0 up                  # Bring interface up
sudo ip link set eth0 down                # Bring interface down
sudo ip link set eth0 mtu 9000           # Set MTU (jumbo frames)
sudo ip link set eth0 promisc on         # Enable promiscuous mode
sudo ip link set eth0 promisc off        # Disable promiscuous mode
sudo ip link set eth0 name newname       # Rename interface
```

### ip route

```bash
ip route                                   # Show routing table
ip route show table all                   # All routing tables
ip -6 route                               # IPv6 routes
ip route get 8.8.8.8                     # Show route to specific destination

# Add/remove routes
sudo ip route add 10.0.2.0/24 via 10.0.1.1          # Add route via gateway
sudo ip route add 10.0.2.0/24 dev eth0               # Add route via interface
sudo ip route add default via 10.0.1.1               # Add default route
sudo ip route add 10.0.2.0/24 via 10.0.1.1 metric 100  # Route with metric
sudo ip route add 10.0.2.0/24 via 10.0.1.1 src 10.0.1.50  # Source address hint
sudo ip route del 10.0.2.0/24                        # Delete route
sudo ip route replace default via 10.0.1.1           # Replace existing route

# Policy routing
sudo ip rule add from 10.0.1.0/24 table 100
sudo ip route add default via 10.0.2.1 table 100
ip rule show                                          # Show policy rules
```

### ip neigh (ARP/NDP)

```bash
ip neigh                                   # Show ARP table
ip neigh show dev eth0                    # ARP for specific interface
ip -s neigh                               # With statistics

# Manage entries
sudo ip neigh add 10.0.1.50 lladdr aa:bb:cc:dd:ee:ff dev eth0  # Static ARP
sudo ip neigh del 10.0.1.50 dev eth0     # Delete entry
sudo ip neigh flush dev eth0             # Flush ARP cache
```

### VLANs

```bash
# Create VLAN interface
sudo ip link add link eth0 name eth0.100 type vlan id 100
sudo ip addr add 10.100.0.1/24 dev eth0.100
sudo ip link set eth0.100 up

# Delete VLAN
sudo ip link del eth0.100
```

### Network Namespaces

```bash
# Manage namespaces
sudo ip netns add ns1                     # Create namespace
sudo ip netns list                        # List namespaces
sudo ip netns exec ns1 ip addr           # Run command in namespace
sudo ip netns del ns1                     # Delete namespace

# Connect namespaces with veth pair
sudo ip link add veth0 type veth peer name veth1
sudo ip link set veth1 netns ns1
sudo ip addr add 10.0.0.1/24 dev veth0
sudo ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth1
sudo ip link set veth0 up
sudo ip netns exec ns1 ip link set veth1 up
sudo ip netns exec ns1 ip link set lo up

# Test connectivity
sudo ip netns exec ns1 ping 10.0.0.1
```

### Bridge

```bash
# Create bridge
sudo ip link add br0 type bridge
sudo ip link set br0 up
sudo ip link set eth0 master br0         # Add interface to bridge
sudo ip link set eth1 master br0

# Show bridge info
bridge link                               # Show bridge ports
bridge fdb show                           # Forwarding database
bridge vlan show                          # VLAN configuration
```

---

## Debugging Workflows

### "Can't Reach Host" Checklist

```bash
# 1. Check local interface is up
ip -br link
ip -br addr

# 2. Ping gateway
ip route | grep default                   # Find gateway
ping -c 2 $(ip route | awk '/default/ {print $3}')

# 3. Check routing
ip route get TARGET_IP                    # How would we reach target?

# 4. DNS resolution (if using hostname)
dig +short hostname                       # Does DNS work?
dig @8.8.8.8 +short hostname            # Try external DNS
cat /etc/resolv.conf                     # Check resolver config

# 5. Ping target
ping -c 4 TARGET_IP                      # Reachable?

# 6. Trace the path
mtr -r -c 10 TARGET_IP                  # Where does it fail?

# 7. Check firewall
sudo iptables -L -n | grep -i drop      # Local drops?
nc -zv -w 3 TARGET_IP PORT              # Port reachable?

# 8. Check for packet loss
mtr -r -c 100 TARGET_IP                 # Extended test for intermittent loss
```

### "DNS Not Resolving" Checklist

```bash
# 1. Check resolver config
cat /etc/resolv.conf
cat /etc/nsswitch.conf

# 2. Test with system resolver
dig hostname
host hostname

# 3. Test with external resolver
dig @8.8.8.8 hostname
dig @1.1.1.1 hostname

# 4. Trace delegation
dig +trace hostname

# 5. Check authoritative servers
dig +short hostname NS
for ns in $(dig +short hostname NS); do
  echo "=== $ns ===" && dig @$ns hostname +short
done

# 6. Check DNSSEC
dig +dnssec hostname
dig hostname DS

# 7. Flush local caches
sudo systemd-resolve --flush-caches 2>/dev/null
sudo resolvectl flush-caches 2>/dev/null
```

### "Connection Refused vs Timeout" Diagnosis

```bash
# Connection REFUSED = host reached, port closed or service not running
# - TCP RST received immediately
# - Check: is the service running? Is it bound to the right interface?
ss -tlnp | grep PORT                     # Is anything listening?
systemctl status SERVICE                  # Is the service running?

# Connection TIMEOUT = packets dropped (firewall/routing) or host down
# - No response at all
# - Check: firewall rules, routing, host status
sudo iptables -L -n -v | grep PORT      # Firewall blocking?
traceroute -T -p PORT host              # Where does it stop?
nmap -Pn -p PORT host                   # What does nmap say?

# Quick diagnosis
nc -zv -w 3 host PORT
# "Connection refused" = port closed, host reachable
# "Connection timed out" = firewall/routing/host issue
# "Name or service not known" = DNS failure
```

### "Slow Network" Diagnosis

```bash
# 1. Test latency and packet loss
mtr -r -c 50 host                       # Look for high loss% or latency spikes

# 2. Check MTU path
ping -c 4 -M do -s 1472 host           # MTU issues? (ICMP too big = MTU problem)
tracepath host                           # Discover path MTU

# 3. Bandwidth test
# Install iperf3 on both ends
iperf3 -s                                # Server side
iperf3 -c host                          # Client side
iperf3 -c host -R                       # Reverse direction
iperf3 -c host -u -b 100M              # UDP 100Mbit test

# 4. Check for retransmits
ss -ti dst host                          # Look at retrans counter
sudo tcpdump -i eth0 -nn 'tcp[tcpflags] & tcp-syn != 0 and host HOST' -c 20  # SYN retransmits

# 5. Check TCP window
sudo tcpdump -i eth0 -nn -v 'host HOST and port PORT' -c 20  # Watch window size

# 6. Check interface errors
ip -s link show eth0                     # Look for errors, dropped, overruns
```

### "Port Already In Use"

```bash
# Find what's using the port
ss -tlnp | grep :PORT                    # TCP listener on port
ss -ulnp | grep :PORT                    # UDP listener on port
lsof -i :PORT                            # Process using port
fuser PORT/tcp                           # PID using TCP port
fuser -k PORT/tcp                        # Kill process using port (careful!)

# Alternative: find and handle
PID=$(ss -tlnp | grep :PORT | awk '{print $NF}' | grep -oP 'pid=\K[0-9]+')
ps -fp $PID                             # Identify the process
kill $PID                               # Graceful stop
kill -9 $PID                            # Force kill (last resort)
```

### iperf3 (Network Performance)

```bash
# Server mode
iperf3 -s                                      # Start server (default port 5201)
iperf3 -s -p 9999                              # Custom port

# Client — TCP throughput
iperf3 -c server-ip                            # Basic TCP test
iperf3 -c server-ip -t 30                      # 30-second test
iperf3 -c server-ip -P 4                       # 4 parallel streams
iperf3 -c server-ip -R                         # Reverse (server→client)

# Client — UDP
iperf3 -c server-ip -u -b 100M                 # UDP at 100Mbps target
iperf3 -c server-ip -u -b 1G -l 9000          # UDP with jumbo frames

# JSON output
iperf3 -c server-ip -J | jq '.end.sum_sent.bits_per_second / 1e6'  # Mbps
```

### socat (Multipurpose Relay)

```bash
# Port forwarding
socat TCP-LISTEN:8080,fork TCP:remote-host:80  # Forward local 8080 → remote 80

# Simple TCP server/client
socat - TCP-LISTEN:1234                        # Listen and echo
socat - TCP:localhost:1234                     # Connect and send

# TLS test connection
socat - OPENSSL:host:443,verify=0              # Connect with TLS (skip verify)

# File transfer
socat TCP-LISTEN:9999 FILE:received.txt,create # Receiver
socat TCP:host:9999 FILE:send.txt              # Sender
```

### Combined Workflows

```bash
# DNS → whois pipeline
dig +short example.com | xargs -I{} whois {}

# Connection monitoring — track established connections by remote IP
ss -tn state established | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -10

# Port scan → service detection pipeline
nmap -sT --top-ports 20 target-host -oG - | grep 'open' | awk '{print $2, $3}'
```

---

## Installation

```bash
# Core networking tools (most pre-installed)
sudo apt install iputils-ping traceroute iproute2 dnsutils netcat-openbsd

# Enhanced diagnostics
sudo apt install mtr-tiny nmap tcpdump whois

# Wireshark CLI
sudo apt install tshark

# Persistent tunnels
sudo apt install autossh

# VPN over SSH
sudo apt install sshuttle

# Connection tracking
sudo apt install conntrack

# nftables
sudo apt install nftables

# Performance testing
sudo apt install iperf3

# All at once
sudo apt install iputils-ping traceroute iproute2 dnsutils netcat-openbsd \
  mtr-tiny nmap tcpdump tshark whois autossh sshuttle conntrack nftables iperf3
```
