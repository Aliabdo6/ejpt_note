# Nmap Scanning and Metasploit Integration Guide

## Overview

This guide covers practical Nmap scanning techniques and demonstrates how to import scan results into Metasploit Framework for vulnerability analysis and exploitation. The workflow focuses on dynamic target scanning with proper output formatting for seamless integration with Metasploit.

## Nmap Fundamentals

Nmap (Network Mapper) is a free, open-source network discovery and security auditing tool. Key capabilities include:

- Host discovery on networks
- Port scanning and service detection
- Version detection and OS fingerprinting
- Output generation for tool integration

## Basic Scanning Techniques

### Simple Port Scan

```bash
nmap <target_ip>
```

### No-Ping Scan (Bypasses ICMP Blocks)

```bash
nmap -Pn <target_ip>
```

## Advanced Scanning with Service Enumeration

### Comprehensive Scan with Version Detection

```bash
nmap -Pn -sV <target_ip>
```

### Full Enumeration Scan (OS + Services)

```bash
nmap -Pn -sV -O <target_ip>
```

### XML Output for Metasploit Integration

```bash
nmap -Pn -sV -O -oX scan_results.xml <target_ip>
```

## Practical Lab Environment

For hands-on practice, use the free **Metasploitable 2** lab from TryHackMe:

**Lab URL**: [Metasploitable 2 on TryHackMe](https://tryhackme.com/room/metasploitable2)

**Setup Instructions**:

1. Create free account on TryHackMe
2. Navigate to the Metasploitable 2 room
3. Start the machine and note the IP address
4. Use the provided IP as your target

**Default Credentials**: Not required for scanning phase

## Dynamic Scanning Examples

### Basic Usage with Variable Target

```bash
# Set target IP as variable
TARGET="10.10.10.10"
nmap -Pn $TARGET
```

### Advanced Scan with Custom Output

```bash
#!/bin/bash
TARGET=$1
OUTPUT_FILE="scan_$(date +%Y%m%d_%H%M%S).xml"

echo "[*] Scanning target: $TARGET"
echo "[*] Output file: $OUTPUT_FILE"

nmap -Pn -sV -O -oX $OUTPUT_FILE $TARGET

echo "[+] Scan completed. Results saved to $OUTPUT_FILE"
```

**Usage**:

```bash
chmod +x nmap_scan.sh
./nmap_scan.sh 10.10.10.10
```

**Sample Output**:

```
[*] Scanning target: 10.10.10.10
[*] Output file: scan_20231201_143022.xml
Starting Nmap 7.80 ( https://nmap.org ) at 2023-12-01 14:30 UTC
Nmap scan report for 10.10.10.10
Host is up (0.15s latency).
Not shown: 977 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
443/tcp  open  ssl/https   Apache httpd 2.2.8 ((Ubuntu) DAV/2)
MAC Address: 00:0C:29:XX:XX:XX (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

[+] Scan completed. Results saved to scan_20231201_143022.xml
```

## Real-World Advanced Example

### Comprehensive Network Assessment Script

```bash
#!/bin/bash

# Configuration
TARGET_NETWORK=$1
SCAN_NAME=$2
OUTPUT_DIR="./scan_results"

# Create output directory
mkdir -p $OUTPUT_DIR

# Generate timestamp
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

echo "[*] Starting comprehensive network assessment"
echo "[*] Target: $TARGET_NETWORK"
echo "[*] Scan name: $SCAN_NAME"

# Host discovery
echo "[+] Performing host discovery..."
nmap -sn -oX $OUTPUT_DIR/host_discovery_$TIMESTAMP.xml $TARGET_NETWORK

# Comprehensive port scan on discovered hosts
echo "[+] Performing comprehensive port scan..."
nmap -Pn -sS -sV -O --script vuln -p- -oX $OUTPUT_DIR/full_scan_$TIMESTAMP.xml $TARGET_NETWORK

# UDP top ports scan
echo "[+] Performing UDP scan..."
nmap -Pn -sU --top-ports 100 -oX $OUTPUT_DIR/udp_scan_$TIMESTAMP.xml $TARGET_NETWORK

echo "[+] Assessment completed. Files saved in $OUTPUT_DIR:"
ls -la $OUTPUT_DIR/*$TIMESTAMP.xml
```

**Usage**:

```bash
./network_assessment.sh 192.168.1.0/24 "corporate_network"
```

## Metasploit Integration Preparation

### XML Output Format

The `-oX` flag generates XML output that Metasploit can parse directly:

```bash
# Standard import format
nmap -Pn -sV -O -oX metasploit_import.xml <target_ip>
```

### Validating XML Output

```bash
# Check XML file structure
head -n 20 scan_results.xml

# Verify file was created
ls -la *.xml
```

## Best Practices

1. **Always use `-Pn`** for modern networks where ICMP may be blocked
2. **Combine `-sV` and `-O`** for comprehensive enumeration
3. **Use XML output (`-oX`)** when planning Metasploit integration
4. **Timestamp output files** for organization and tracking
5. **Validate scan results** before Metasploit import

## Next Steps

After generating Nmap scan results:

1. Import XML files into Metasploit using `db_import`
2. Analyze services and vulnerabilities within Metasploit
3. Use gathered information to select appropriate exploit modules
4. Leverage Metasploit's database for tracking and reporting

This workflow enables efficient transition from reconnaissance to exploitation while maintaining organized results for professional penetration testing engagements.
