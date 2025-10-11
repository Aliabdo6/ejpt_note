# Nmap Port Scanning: SYN Scan vs TCP Connect Scan

## Overview

In this study, we'll explore Nmap's primary port scanning techniques, focusing on the SYN scan (stealth scan) and TCP Connect scan. Understanding these methods is crucial for effective network reconnaissance and security assessment.

## Core Scanning Methods

### SYN Scan (Stealth Scan)

The SYN scan is Nmap's default scanning method when run with privileged (root) access. It's often called a "half-open" scan because it doesn't complete the TCP three-way handshake.

**How it works:**

- Nmap sends a SYN packet to the target port
- If the port is open, the target responds with SYN-ACK
- Nmap responds with RST (reset) instead of ACK, terminating the connection
- If the port is closed, the target responds with RST
- No response typically indicates filtering by a firewall

**Key advantages:**

- Stealthier - avoids creating connection logs on target systems
- Faster than TCP Connect scan
- Less network overhead

### TCP Connect Scan

This is Nmap's default when running without privileged access. It completes the full TCP three-way handshake.

**How it works:**

- Nmap sends SYN packet to target port
- If port is open, target responds with SYN-ACK
- Nmap completes handshake with ACK
- Nmap then sends RST to terminate the connection
- Creates actual TCP connections that are logged

## Practical Usage

### Basic SYN Scan

```bash
# Scan specific ports
nmap -sS -p 80,445,3389 <target_ip>

# Scan port range
nmap -sS -p 1-1000 <target_ip>

# Use default top ports (1000 most common)
nmap -sS --top-ports 1000 <target_ip>
```

**Example Output:**

```
Starting Nmap 7.92 ( https://nmap.org )
Nmap scan report for <target_ip>
Host is up (0.0012s latency).

PORT     STATE    SERVICE
80/tcp   open     http
445/tcp  open     microsoft-ds
3389/tcp open     ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds
```

### TCP Connect Scan

```bash
# Basic TCP Connect scan
nmap -sT -p 80,443,22 <target_ip>

# Scan with timing template
nmap -sT -T4 -p- <target_ip>
```

### UDP Scanning

```bash
# Scan common UDP ports
nmap -sU -p 53,137,138,139 <target_ip>

# Fast UDP scan
nmap -sU --top-ports 50 <target_ip>
```

## Advanced Examples

### Comprehensive Network Assessment

```bash
# Multi-technique scan with service detection
nmap -sS -sU -p- -T4 -A -v <target_ip>

# Script scanning with vulnerability assessment
nmap -sS -p 80,443,22 --script vuln <target_ip>
```

### Firewall Evasion Techniques

```bash
# Fragmented packets
nmap -sS -f <target_ip>

# Timing manipulation
nmap -sS -T2 <target_ip>

# Decoy scanning
nmap -sS -D RND:10 <target_ip>
```

## Real-World Scenarios

### Practical Example: Web Server Assessment

```bash
# Focused web server scan
nmap -sS -p 80,443,8080,8443 --script http-enum,http-security-headers <target_ip>
```

**Expected Output:**

```
PORT    STATE SERVICE
80/tcp  open  http
| http-enum:
|   /admin/: Admin login
|   /backup/: Backup directory
|_  /test/: Test page
443/tcp open  https
```

### Advanced Example: Comprehensive Service Mapping

```bash
# Full service discovery with OS detection
nmap -sS -O -sV --version-intensity 5 -p- <target_ip>
```

## Traffic Analysis

When analyzing scans with tools like Wireshark:

**SYN Scan Traffic Pattern:**

- SYN → [target]
- SYN-ACK ← [target] (if open)
- RST → [target] (from Nmap)

**TCP Connect Scan Traffic Pattern:**

- SYN → [target]
- SYN-ACK ← [target] (if open)
- ACK → [target] (completes handshake)
- RST → [target] (terminates connection)

## Best Practices

1. **Always use SYN scan when you have privileges** - it's faster and stealthier
2. **Be mindful of network policies** - only scan systems you're authorized to test
3. **Use appropriate timing** - adjust scan speed based on network conditions
4. **Combine techniques** - use multiple scan types for comprehensive results
5. **Document findings** - keep detailed records of scan parameters and results

## Common Port States

- **Open**: Service is actively accepting connections
- **Closed**: Port is accessible but no service listening
- **Filtered**: Firewall is likely blocking access
- **Unfiltered**: Port is accessible but state cannot be determined

## Next Steps

After port scanning, the logical progression is to:

1. Perform service version detection (`-sV`)
2. Conduct OS fingerprinting (`-O`)
3. Run NSE scripts for vulnerability assessment
4. Map out the complete attack surface

This foundation in port scanning techniques provides the essential first step in any comprehensive security assessment or penetration test.
