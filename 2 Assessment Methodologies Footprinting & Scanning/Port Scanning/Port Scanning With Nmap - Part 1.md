# Port Scanning with Nmap: Comprehensive Guide

## Introduction to Port Scanning

Port scanning is the logical next step after host discovery, allowing us to identify open ports and services on target systems. This phase builds the foundation for service enumeration and vulnerability assessment.

## Default Nmap Scan Behavior

### Basic Default Scan

```bash
# Default scan (includes host discovery)
nmap target_ip

# Example output analysis:
# - Performs host discovery first
# - If host is down, skips port scanning
# - Scans 1000 most common TCP ports
# - Uses SYN scan if privileged, TCP connect if not
```

### Skipping Host Discovery

```bash
# Skip host discovery with -Pn
nmap -Pn target_ip

# Useful for:
# - Windows systems blocking ICMP
# - Known active hosts
# - Avoiding false negatives
```

## Port Specification Methods

### Single Port Scanning

```bash
# Scan specific port
nmap -p 80 target_ip

# With host discovery skipped
nmap -Pn -p 80 target_ip
```

### Multiple Port Scanning

```bash
# Multiple specific ports
nmap -p 80,443,3389 target_ip

# Common service ports
nmap -p 21,22,23,25,53,80,443,445,3389 target_ip
```

### Port Ranges

```bash
# Port range
nmap -p 1-100 target_ip

# Multiple ranges
nmap -p 1-100,1000-2000 target_ip

# All TCP ports (0-65535)
nmap -p- target_ip
```

## Scan Profiles and Performance

### Fast Scan Profile

```bash
# Scan top 100 most common ports
nmap -F target_ip

# With host discovery skipped
nmap -Pn -F target_ip
```

### Timing Templates

```bash
# Polite (T1) - Slowest, stealthiest
nmap -T1 target_ip

# Normal (T3) - Balanced
nmap -T3 target_ip

# Aggressive (T4) - Faster, noisier
nmap -T4 target_ip

# Insane (T5) - Fastest, very noisy
nmap -T5 target_ip
```

## Port States and Interpretation

### Common Port States

- **Open**: Service is listening and accessible
- **Closed**: Port is accessible but no service listening
- **Filtered**: Firewall is blocking access (uncertain state)
- **Unfiltered**: Accessible but state undetermined

### Windows Firewall Detection

```bash
# Filtered ports often indicate Windows Firewall
nmap -p 80,443,3389,8080 target_ip

# Expected results on Windows:
# - Open ports: Services running
# - Filtered ports: Firewall blocking
# - Closed ports: No firewall interference
```

## Comprehensive Scanning Strategies

### Initial Reconnaissance

```bash
# Quick assessment
nmap -Pn -F -T4 target_ip

# Expected: Scan top 100 ports quickly
# Use case: Initial target assessment
```

### Standard Service Discovery

```bash
# Common services scan
nmap -Pn -p 21,22,23,25,53,80,110,135,139,143,443,445,993,995,1433,3389,5432,5900,8080 target_ip

# Covers most Windows/Linux services
```

### Comprehensive TCP Scan

```bash
# All TCP ports (comprehensive but slow)
nmap -Pn -p- -T4 target_ip

# Use case: Important targets requiring full coverage
# Time: Several minutes to hours depending on network
```

## Scan Techniques Overview

### SYN Scan (Default for Root)

**Operation:**

1. Send SYN packet
2. Receive SYN-ACK = Port open
3. Receive RST = Port closed
4. No response = Filtered

**Advantages:**

- Stealthy (half-open connections)
- Fast and efficient
- Less likely to be logged

### TCP Connect Scan (Non-root)

**Operation:**

1. Complete TCP three-way handshake
2. Established connection = Port open
3. Connection refused = Port closed

**Advantages:**

- No special privileges required
- Reliable results

## Practical Scanning Methodology

### Step 1: Quick Assessment

```bash
# Fast scan for initial overview
nmap -Pn -F -T4 target_ip
```

### Step 2: Common Services

```bash
# Focus on common business services
nmap -Pn -p 21,22,23,25,53,80,443,445,3389,5432,5900,8080,8443 target_ip
```

### Step 3: Comprehensive Scan

```bash
# Full TCP port range for important targets
nmap -Pn -p- -T4 target_ip
```

## Performance Optimization

### Large Network Scanning

```bash
# For multiple targets, use fast scans initially
nmap -Pn -F -T4 192.168.1.0/24

# Then focus comprehensive scans on active hosts
```

### Verbose Output

```bash
# See detailed scan progress
nmap -Pn -v target_ip

# Benefits:
# - Real-time progress updates
# - Detailed reasoning for results
# - Troubleshooting information
```

## Real-World Examples

### Internal Network Assessment

```bash
# Quick internal scan
nmap -Pn -F -T4 192.168.1.100

# Comprehensive internal scan
nmap -Pn -p- -T4 192.168.1.100
```

### External Assessment

```bash
# External target common services
nmap -Pn -p 80,443,8080,8443,22 target_ip
```

### Windows Environment Focus

```bash
# Windows-specific ports
nmap -Pn -p 135,139,445,3389,5985,5986 target_ip
```

## Key Takeaways

### Best Practices

1. **Always use -Pn** for known active hosts to avoid missed scans
2. **Start with fast scans** (-F) for initial assessment
3. **Use comprehensive scans** (-p-) for important targets
4. **Consider timing** (-T4) for efficiency vs. stealth balance
5. **Interpret port states** correctly for firewall detection

### Common Pitfalls

- Relying solely on default scans (may miss uncommon ports)
- Not scanning all TCP ports for critical targets
- Misinterpreting filtered vs. closed states
- Overlooking performance impact on large networks

### Efficiency Tips

```bash
# For large networks: quick scan first
nmap -Pn -F -T4 target_list

# Then comprehensive scan on interesting hosts
nmap -Pn -p- -T4 important_hosts
```

## Next Steps

This port scanning foundation enables service version detection and operating system fingerprinting, which we'll explore in subsequent sections to build a complete picture of target systems.

---

_Mastering Nmap port scanning provides the essential visibility into target services and configurations, forming the basis for effective vulnerability assessment and exploitation planning in penetration testing engagements._
