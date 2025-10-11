# Nmap Service Version & OS Detection

## Overview

Building on our port scanning knowledge, we now dive into service version detection and operating system fingerprinting. These techniques transform basic port information into actionable intelligence for security assessments and penetration testing.

## The Assessment Workflow

1. **Host Discovery** → **Port Scanning** → **Service Detection** → **OS Detection**

This natural progression helps build a comprehensive understanding of target systems and their potential vulnerabilities.

## Practical Lab Environment

### Network Discovery

First, we identify our target network range:

```bash
# Check network configuration
ifconfig eth1
ip addr show eth1

# Perform host discovery
nmap -sn 192.168.1.0/24
```

**Example Output:**

```
Starting Nmap 7.92 ( https://nmap.org )
Nmap scan report for router (192.168.1.1)
Host is up (0.001s latency).
Nmap scan report for target1 (192.168.1.43)
Host is up (0.002s latency).
Nmap scan report for kali (192.168.1.44)
Host is up (0.000s latency).
Nmap done: 256 IP addresses (3 hosts up) scanned in 2.45 seconds
```

### Comprehensive Port Scanning

```bash
# Full TCP port scan with timing optimization
nmap -sS -T4 -p- 192.168.1.43
```

**Example Output:**

```
Starting Nmap 7.92 ( https://nmap.org )
Nmap scan report for target1 (192.168.1.43)
Host is up (0.001s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE
6421/tcp  open  unknown
4128/tcp  open  unknown
55413/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 18.23 seconds
```

## Service Version Detection

### Basic Service Detection

```bash
# Service version detection on discovered ports
nmap -sS -sV 192.168.1.43
```

**Example Output:**

```
PORT      STATE SERVICE VERSION
6421/tcp  open  mongodb MongoDB 2.6.10
4128/tcp  open  memcache?
55413/tcp open  ftp     vsftpd 3.0.3
Service Info: OS: Unix
```

### Advanced Service Detection

```bash
# Aggressive service detection with intensity control
nmap -sS -sV --version-intensity 8 192.168.1.43

# Maximum intensity detection
nmap -sS -sV --version-intensity 9 192.168.1.43
```

**Key Findings:**

- **MongoDB 2.6.10** on port 6421 (NoSQL database)
- **Memcache** service on port 4128 (caching system)
- **VSFTPD 3.0.3** on port 55413 (FTP server)
- Likely **Unix-based** operating system

## Operating System Detection

### Basic OS Fingerprinting

```bash
# Combined service and OS detection
nmap -sS -sV -O 192.168.1.43
```

### Aggressive OS Detection

```bash
# Enhanced OS detection with guessing
nmap -sS -sV -O --osscan-guess 192.168.1.43
```

**Example Output:**

```
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (92%)
No exact OS matches for host.
```

## Real-World Scenarios

### Comprehensive Target Assessment

```bash
# Complete reconnaissance scan
nmap -sS -sV -O -T4 -p- 192.168.1.43
```

### Focused Service Enumeration

```bash
# Target specific services with detailed version info
nmap -sS -sV --version-intensity 9 -p 6421,4128,55413 192.168.1.43
```

## Advanced Techniques

### Service Detection Intensity Levels

The version intensity scale (0-9) controls how aggressively Nmap probes services:

```bash
# Lightweight detection (faster)
nmap -sV --version-intensity 3 <target>

# Balanced detection (default)
nmap -sV --version-intensity 5 <target>

# Comprehensive detection (slower)
nmap -sV --version-intensity 9 <target>
```

### OS Detection Options

```bash
# Maximum OS detection effort
nmap -O --max-os-tries 1 <target>  # Faster
nmap -O --max-os-tries 5 <target>  # More thorough
```

## Practical Usage Examples

### Web Server Assessment

```bash
# Focused web service detection
nmap -sS -sV -p 80,443,8080,8443 <target_ip>
```

### Database Server Profiling

```bash
# Database service enumeration
nmap -sS -sV -p 1433,1521,27017,3306,5432 <target_ip>
```

### Complete Network Service Mapping

```bash
# All-in-one reconnaissance
nmap -sS -sV -O -T4 --script default <target_ip>
```

## Analysis and Next Steps

### Information Utilization

The gathered intelligence enables:

1. **Vulnerability Research**: Specific service versions allow targeted vulnerability assessment
2. **Exploit Planning**: Known versions help identify potential exploits
3. **Attack Surface Mapping**: Understanding what services are exposed
4. **Threat Modeling**: Assessing potential attack vectors

### Common Service Patterns

- **Non-standard ports**: Services running on unusual ports (like MongoDB on 6421)
- **Version-specific vulnerabilities**: Older versions often have known security issues
- **Service combinations**: Multiple services can reveal system purpose

## Best Practices

1. **Always scan all ports** - services often run on non-standard ports
2. **Use appropriate timing** - balance between speed and accuracy
3. **Combine detection methods** - use both service and OS detection together
4. **Validate findings** - cross-reference results with other tools
5. **Document thoroughly** - keep detailed records for analysis and reporting

## Limitations and Considerations

- **Firewall interference** may affect detection accuracy
- **Service masking** techniques can obscure true versions
- **Network conditions** impact scan reliability
- **Some systems** resist precise OS fingerprinting

## Next Steps

After service and OS detection, the logical progression is:

1. **Nmap Scripting Engine (NSE)** for advanced enumeration
2. **Vulnerability scanning** with specialized tools
3. **Manual service enumeration** for deeper intelligence
4. **Exploitation planning** based on gathered information

This foundation in service and OS detection provides critical intelligence for any security assessment, enabling targeted vulnerability analysis and informed decision-making in penetration testing engagements.
