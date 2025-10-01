# Port Scanning with Nmap

## Understanding Port Scanning

Port scanning is the process of identifying open ports and services on target systems. This crucial phase follows host discovery and provides the foundation for vulnerability assessment and exploitation.

## Nmap Scanning Fundamentals

### Basic Scan Types

```bash
# Default Nmap scan (TCP SYN on top 1000 ports)
nmap target_ip

# Skip host discovery (-Pn)
nmap -Pn target_ip

# Fast scan (top 100 ports)
nmap -F target_ip

# Specific port scan
nmap -p 80,443,22 target_ip

# Port range scan
nmap -p 1-1000 target_ip

# All TCP ports (not recommended for large scans)
nmap -p- target_ip
```

## Comprehensive Scanning Techniques

### Service and Version Detection

```bash
# Service version detection
nmap -sV target_ip

# Operating system detection
nmap -O target_ip

# Aggressive scan (OS, version, script scanning)
nmap -A target_ip

# Default script scan
nmap -sC target_ip
```

### UDP Port Scanning

```bash
# UDP port scan (slower but essential)
nmap -sU target_ip

# UDP scan on specific ports
nmap -sU -p 53,161,162 target_ip
```

## Advanced Scanning Strategies

### Timing and Performance

```bash
# Timing templates (T0-T5)
nmap -T0 target_ip    # Paranoid (slowest)
nmap -T1 target_ip    # Sneaky
nmap -T2 target_ip    # Polite
nmap -T3 target_ip    # Normal (default)
nmap -T4 target_ip    # Aggressive
nmap -T5 target_ip    # Insane (fastest)

# Custom timing control
nmap --max-rtt-timeout 100ms target_ip
nmap --min-rate 100 target_ip
nmap --max-rate 1000 target_ip
```

### Output Formats

```bash
# Normal text output
nmap -oN scan_results.txt target_ip

# XML output (for Metasploit integration)
nmap -oX scan_results.xml target_ip

# All formats
nmap -oA scan_name target_ip

# Grepable format
nmap -oG scan_results.gnmap target_ip
```

## Practical Implementation Scripts

### Comprehensive Port Scanner

```bash
#!/bin/bash

TARGET=${1}
OUTPUT_DIR="nmap_scans_$(date +%Y%m%d_%H%M%S)"
TIMING=${2:-T4}

if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target_ip> [timing_template]"
    echo "Example: $0 192.168.1.100 T4"
    exit 1
fi

mkdir -p $OUTPUT_DIR
echo "[*] Starting comprehensive Nmap scan for: $TARGET"
echo "[*] Output directory: $OUTPUT_DIR"
echo "[*] Timing template: $TIMING"

# Initial TCP SYN scan
echo "[*] Performing TCP SYN scan..."
nmap -sS -Pn -$TIMING -oA $OUTPUT_DIR/tcp_syn_scan $TARGET

# Service version detection
echo "[*] Performing service version detection..."
nmap -sV -Pn -$TIMING -oA $OUTPUT_DIR/service_versions $TARGET

# OS detection
echo "[*] Performing OS detection..."
nmap -O -Pn -$TIMING -oA $OUTPUT_DIR/os_detection $TARGET

# Default script scan
echo "[*] Performing default script scan..."
nmap -sC -Pn -$TIMING -oA $OUTPUT_DIR/script_scan $TARGET

# UDP top ports scan
echo "[*] Performing UDP scan (top 100 ports)..."
nmap -sU -Pn --top-ports 100 -$TIMING -oA $OUTPUT_DIR/udp_scan $TARGET

echo "[*] Scan complete. Results saved to: $OUTPUT_DIR"
```

**Usage:**

```bash
./comprehensive_scan.sh 192.168.1.100 T4
```

### Targeted Service Scanner

```bash
#!/bin/bash

TARGET=${1}
PORTS=${2:-"21,22,23,25,53,80,110,135,139,143,443,445,993,995,1433,3306,3389,5432,5900,8080"}

if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target_ip> [ports]"
    echo "Default ports: $PORTS"
    exit 1
fi

echo "[*] Starting targeted service scan..."
echo "[*] Target: $TARGET"
echo "[*] Ports: $PORTS"

nmap -sS -sV -sC -O -p $PORTS -Pn -T4 -oA targeted_scan_$(date +%Y%m%d_%H%M%S) $TARGET
```

**Usage:**

```bash
./targeted_scan.sh 192.168.1.100
./targeted_scan.sh 192.168.1.100 "80,443,22,21,25"
```

## Real-World Scan Examples

### Windows Target Scan

```bash
# Windows systems often block ICMP, use -Pn
nmap -Pn -sS -sV -sC -O 192.168.1.100

# Common Windows services
nmap -Pn -p 135,139,445,3389,5985,5986 -sV 192.168.1.100
```

### Web Server Scan

```bash
# Comprehensive web server scan
nmap -Pn -p 80,443,8000,8080,8443 -sV -sC --script http* target_ip

# SSL/TLS specific checks
nmap -Pn -p 443 --script ssl* target_ip
```

## Interpreting Scan Results

### Common Port States

- **Open**: Service is accepting connections
- **Closed**: Port is accessible but no service listening
- **Filtered**: Firewall is blocking probes (may be open)
- **Unfiltered**: Accessible but state cannot be determined

### Service Identification

```bash
# Example output analysis
PORT     STATE SERVICE      VERSION
80/tcp   open  http         Apache httpd 2.4.41
443/tcp  open  ssl/http     Apache httpd 2.4.41
22/tcp   open  ssh          OpenSSH 8.2p1
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds
3389/tcp open  ms-wbt-server Microsoft Terminal Services
```

## Advanced Script Usage

### Nmap Scripting Engine (NSE)

```bash
# Run specific script categories
nmap --script safe target_ip
nmap --script vuln target_ip
nmap --script exploit target_ip

# Individual scripts
nmap --script http-enum target_ip
nmap --script smb-security-mode target_ip
nmap --script ssh-auth-methods target_ip

# Multiple script categories
nmap --script "safe and discovery" target_ip
```

## Best Practices

### Stealth Considerations

```bash
# Slower, stealthier scans
nmap -T2 --max-parallelism 1 --scan-delay 5s target_ip

# Fragment packets
nmap -f target_ip

# Use decoy IPs
nmap -D RND:10 target_ip
```

### Network Considerations

```bash
# Adjust for network conditions
nmap --min-rate 50 --max-rate 100 target_ip    # Slow network
nmap --min-rate 500 --max-rate 1000 target_ip  # Fast network

# Handle firewalls
nmap -sS -f --data-length 16 target_ip
```

## Output Analysis and Documentation

### Generating Reports

```bash
# Comprehensive scan with all outputs
nmap -A -oA full_scan target_ip

# Convert XML to HTML
xsltproc scan_results.xml -o scan_results.html

# Extract open ports
grep "open" scan_results.nmap | cut -d'/' -f1
```

### Continuous Monitoring

```bash
#!/bin/bash
# Diff scanner for detecting changes
BASELINE="baseline_scan.xml"
CURRENT_SCAN="current_scan.xml"

nmap -oX $CURRENT_SCAN target_ip
ndiff $BASELINE $CURRENT_SCAN
```

## Common Service Ports Reference

### Essential Ports to Scan

- **21**: FTP
- **22**: SSH
- **23**: Telnet
- **25**: SMTP
- **53**: DNS
- **80**: HTTP
- **110**: POP3
- **135**: MSRPC
- **139/445**: SMB
- **143**: IMAP
- **443**: HTTPS
- **993**: IMAPS
- **995**: POP3S
- **1433**: MSSQL
- **3306**: MySQL
- **3389**: RDP
- **5432**: PostgreSQL
- **5900**: VNC

---

_This documentation reflects my practical approach to network port scanning. Always ensure proper authorization before scanning any system. These techniques are for legitimate security testing and educational purposes._
