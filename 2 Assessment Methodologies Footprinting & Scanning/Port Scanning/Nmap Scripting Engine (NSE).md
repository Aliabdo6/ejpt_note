# Nmap Scripting Engine (NSE) Guide

## Overview

The Nmap Scripting Engine (NSE) is one of Nmap's most powerful features, allowing automated information gathering, vulnerability detection, and advanced enumeration. This guide covers practical NSE usage for enhancing service detection and OS fingerprinting.

## NSE Fundamentals

### What is NSE?

- **Automation framework** for network discovery and security assessment
- **Lua-based scripting** with `.nse` file extension
- **Pre-packaged scripts** located in `/usr/share/nmap/scripts/`
- **Categorized functionality** for different assessment phases

### Script Categories

- **default**: Safe scripts that provide useful information
- **discovery**: Network and service discovery
- **auth**: Authentication-related scripts
- **brute**: Password brute-forcing
- **safe**: Non-intrusive scripts only
- **exploit**: Vulnerability exploitation (use with caution)

## Practical NSE Usage

### Discovering Available Scripts

```bash
# List all NSE scripts
ls /usr/share/nmap/scripts/

# Search for specific service scripts
ls /usr/share/nmap/scripts/ | grep http
ls /usr/share/nmap/scripts/ | grep ftp
ls /usr/share/nmap/scripts/ | grep mongodb
```

### Default Script Scan

```bash
# Run default safe scripts on target
nmap -sS -sC 192.168.1.43

# Combine with service detection
nmap -sS -sV -sC 192.168.1.43

# Target specific ports with scripts
nmap -sS -sV -sC -p 6421,4128,55413 192.168.1.43
```

**Example Output:**

```
PORT      STATE SERVICE VERSION
6421/tcp  open  mongodb MongoDB 2.6.10
| mongodb-databases:
|   OK
|   admin (empty)
|   local
|     version 2.6.10
|     sysinfo: Linux victim1 3.13.0-24-generic #46-Ubuntu SMP x86_64
|     distro: Ubuntu 14.04
|_    uptime 2585
```

### Individual Script Execution

```bash
# Run specific script
nmap -sS --script=mongodb-info 192.168.1.43

# Run multiple specific scripts
nmap -sS --script=mongodb-info,memcached-info 192.168.1.43

# Target specific port with script
nmap -sS --script=mongodb-info -p 6421 192.168.1.43
```

### Script Category Execution

```bash
# Run all scripts from a category
nmap -sS --script=ftp* 192.168.1.43

# Run safe discovery scripts
nmap -sS --script="safe and discovery" 192.168.1.43
```

## Advanced NSE Techniques

### Getting Script Information

```bash
# View script documentation
nmap --script-help=mongodb-databases
nmap --script-help=ftp-anon
```

**Example Output:**

```
mongodb-databases
Categories: default discovery safe
Gets list of MongoDB databases and system information
```

### Comprehensive Assessment Scan

```bash
# All-in-one aggressive scan
nmap -sS -A 192.168.1.43

# Equivalent to: -sV -sC -O --traceroute
nmap -sS -sV -sC -O --traceroute 192.168.1.43
```

## Real-World Scenarios

### Database Server Enumeration

```bash
# MongoDB specific enumeration
nmap -sS -sV --script=mongodb-databases,mongodb-info -p 6421 192.168.1.43

# Memcached information gathering
nmap -sS --script=memcached-info -p 4128 192.168.1.43
```

**Memcached Output:**

```
4128/tcp open  memcache
| memcached-info:
|   Process ID: 853
|   Uptime: 2585 seconds
|   Architecture: x86_64
|   Current connections: 1
|   Total connections: 5
|   Authentication: not required
```

### FTP Service Assessment

```bash
# FTP anonymous login check
nmap -sS --script=ftp-anon -p 55413 192.168.1.43

# Comprehensive FTP enumeration
nmap -sS --script=ftp* -p 55413 192.168.1.43
```

## Information Extraction Examples

### MongoDB Intelligence Gathering

From our lab example, NSE revealed:

- **OS Distribution**: Ubuntu 14.04
- **Kernel Version**: 3.13.0-24-generic
- **Hostname**: victim1
- **Database Info**: Local and admin databases
- **Service Uptime**: 2585 seconds
- **Architecture**: x86_64

### Security Implications

The extracted information enables:

1. **Vulnerability Research**: Target specific OS and service versions
2. **Attack Planning**: Identify misconfigurations (e.g., no authentication)
3. **Privilege Escalation**: Kernel version-specific exploits
4. **Lateral Movement**: Database content analysis

## Best Practices

### Script Selection

```bash
# Use safe categories for initial assessment
nmap -sS --script="default or safe" <target>

# Avoid intrusive scripts during reconnaissance
nmap -sS --script="not intrusive" <target>
```

### Performance Optimization

```bash
# Limit to relevant ports
nmap -sS --script=mongodb* -p 27017,28017 <target>

# Use timing templates
nmap -sS -sC -T4 <target>
```

### Output Management

```bash
# Save detailed results
nmap -sS -sC -oA full_scan <target>

# Verbose output for debugging
nmap -sS -sC -v <target>
```

## Common Use Cases

### Web Service Assessment

```bash
nmap -sS -sV --script=http-enum,http-title -p 80,443,8080 <target>
```

### Database Service Mapping

```bash
nmap -sS -sV --script=mysql-info,mongodb-info -p 3306,27017 <target>
```

### Network Service Discovery

```bash
nmap -sS --script=broadcast,smb-os-discovery <target>
```

## Troubleshooting

### Script Execution Issues

- **Check script categories**: Ensure scripts are in default/safe categories
- **Verify service detection**: Scripts run based on detected services
- **Port specification**: Some scripts require specific ports
- **Privilege requirements**: Root access needed for certain scripts

### Common Patterns

```bash
# If script doesn't run automatically
nmap -sS -sV --script=<script-name> -p <specific-port> <target>

# Debug script execution
nmap -sS --script=<script-name> --script-trace <target>
```

## Aggressive Scanning

### The -A Option

```bash
# Comprehensive assessment
nmap -sS -A 192.168.1.43

# What -A includes:
# -sV: Service version detection
# -sC: Default script scan
# -O: OS detection
# --traceroute: Network path tracing
```

## Next Steps

After NSE enumeration:

1. **Vulnerability Analysis**: Research specific service versions
2. **Exploitation Planning**: Identify potential attack vectors
3. **Credential Testing**: Use auth and brute scripts (when authorized)
4. **Service-specific Enumeration**: Deeper investigation of discovered services

## Key Takeaways

- **NSE extends Nmap** beyond basic port scanning
- **Default scripts** provide safe, valuable information
- **Service misconfigurations** often reveal system details
- **Script categories** help manage assessment intensity
- **Combined scanning** (`-A`) efficient for comprehensive assessment

The Nmap Scripting Engine transforms basic network reconnaissance into detailed intelligence gathering, providing critical information for security assessments and penetration testing engagements.
