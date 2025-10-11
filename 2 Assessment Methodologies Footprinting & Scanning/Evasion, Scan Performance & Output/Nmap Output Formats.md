# Nmap Output Formats Guide

## Overview

Properly saving and managing Nmap scan results is crucial for penetration testing documentation, analysis, and tool integration. This guide covers Nmap's output formats and their practical applications.

## Output Format Options

### Normal Format (`-oN`)

Saves results in human-readable format identical to terminal output.

```bash
# Save normal format to text file
nmap -sS -T4 --top-ports 100 <target_ip> -oN scan_results.txt

# Practical example
nmap -sS -T4 192.168.1.43 -oN nmap_normal.txt
```

**Use Case:** Penetration testing reports, human analysis, documentation

### XML Format (`-oX`)

Structured format for tool integration and data processing.

```bash
# Save XML format
nmap -sS -T4 <target_ip> -oX nmap_scan.xml

# With service detection
nmap -sS -sV -T4 <target_ip> -oX detailed_scan.xml
```

**Use Case:** Metasploit integration, automated processing, data parsing

### Grepable Format (`-oG`)

Machine-readable format for scripting and text processing.

```bash
# Save grepable format
nmap -sS -T4 <target_ip> -oG nmap_grep.txt

# Combined scanning
nmap -sS -sV -O -T4 <target_ip> -oG comprehensive_scan.grep
```

**Use Case:** Scripting, automation, custom parsing with grep/awk

### All Formats (`-oA`)

Generates all three major formats simultaneously.

```bash
# Generate all output formats
nmap -sS -sV -T4 <target_ip> -oA complete_scan

# Creates:
# complete_scan.nmap (normal)
# complete_scan.xml (XML)
# complete_scan.gnmap (grepable)
```

**Use Case:** Comprehensive scanning with multiple format requirements

## Practical Implementation

### Basic Scan with Output

```bash
# Comprehensive scan with normal output
nmap -sS -sV -O -T4 <target_ip> -oN full_scan.txt

# Service detection with XML output
nmap -sS -sV --top-ports 1000 <target_ip> -oX services.xml

# Quick network scan with grepable output
nmap -sn 192.168.1.0/24 -oG hosts_up.grep
```

### Metasploit Integration

```bash
# Start PostgreSQL database
service postgresql start

# Launch Metasploit framework
msfconsole

# Create workspace
workspace -a pentest_engagement

# Import Nmap XML results
db_import /path/to/nmap_scan.xml

# Verify imported hosts
hosts
services
```

### Direct Metasploit Scanning

```bash
# Scan from within Metasploit
db_nmap -sS -sV -O <target_ip>

# Results automatically saved to database
hosts
services
```

## Advanced Usage

### Verbose Output

```bash
# Increased verbosity for debugging
nmap -sS -sV -v -T4 <target_ip> -oN verbose_scan.txt

# Maximum verbosity
nmap -sS -sV -vv -T4 <target_ip> -oN detailed_scan.txt
```

### Reason Output

```bash
# Show reason for port states
nmap -sS --reason -T4 <target_ip> -oN reasoned_scan.txt
```

## File Management

### Organized Output Structure

```bash
# Create dated scan directory
mkdir scans_$(date +%Y%m%d)
cd scans_$(date +%Y%m%d)

# Perform scans with organized naming
nmap -sS -sV -T4 192.168.1.0/24 -oA network_scan
nmap -sS -p- 192.168.1.43 -oA full_port_scan
```

### Batch Processing

```bash
# Scan multiple targets with output
for ip in 192.168.1.{1..50}; do
    nmap -sS -T4 $ip -oN scan_$ip.txt
done
```

## Real-World Scenarios

### Penetration Testing Workflow

```bash
# Initial discovery
nmap -sn 192.168.1.0/24 -oG hosts_discovered.grep

# Port scanning discovered hosts
nmap -sS -T4 -iL active_hosts.txt -oA port_scan

# Service enumeration
nmap -sS -sV -O -T4 -iL open_ports_hosts.txt -oA service_scan

# Import to Metasploit
db_import service_scan.xml
```

### Continuous Assessment

```bash
# Weekly network scans with timestamp
nmap -sS -T4 192.168.1.0/24 -oA weekly_scan_$(date +%Y%m%d)
```

## Output Analysis

### Normal Format Analysis

- Human-readable summary
- Perfect for reports and documentation
- Easy to review and share

### XML Format Benefits

- Structured data parsing
- Metasploit integration
- Automated tool processing
- Data validation capabilities

### Grepable Format Usage

```bash
# Extract open ports
grep "open" nmap_scan.gnmap

# Find specific services
grep "http" nmap_scan.gnmap

# Custom parsing with awk
awk '/open/ {print $2}' nmap_scan.gnmap
```

## Best Practices

### Documentation Standards

1. **Always save scan results** - Never rely solely on terminal output
2. **Use descriptive filenames** - Include date, scope, and scan type
3. **Maintain organization** - Create directory structures for engagements
4. **Version control** - Track changes in significant rescans

### Metasploit Integration

```bash
# Workspace management
workspace -a client_engagement
workspace client_engagement

# Regular database backups
db_export -f xml client_backup.xml

# Query imported data
hosts -c address,os_name
services -c port,proto,name,info
```

### Reporting Preparation

```bash
# Generate clean reports
nmap -sS -sV --reason -T4 <target> -oN final_report.txt

# Include in documentation
cat final_report.txt >> penetration_test_report.md
```

## Troubleshooting

### Common Issues

- **File permissions** - Ensure write access to output directories
- **Disk space** - Large scans can generate significant output
- **Format compatibility** - Verify XML validity for tool import
- **Character encoding** - Check for special characters in output

### Verification Commands

```bash
# Check file creation
ls -la scan_output.*

# Validate XML format
xmllint scan_output.xml

# Test grepable format parsing
grep "Status: Up" scan_output.gnmap
```

## Key Takeaways

1. **Normal format** for human consumption and reporting
2. **XML format** for tool integration and automation
3. **Grepable format** for scripting and text processing
4. **All formats** for comprehensive documentation
5. **Metasploit integration** streamlines workflow
6. **Proper organization** ensures data accessibility

## Final Recommendations

### Essential Workflow

```bash
# Complete assessment sequence
nmap -sn <target_range> -oG hosts.grep
nmap -sS -sV -O -T4 -iL targets.txt -oA comprehensive_scan
db_import comprehensive_scan.xml
```

### Maintenance Commands

```bash
# Clean up old scans
find ./scans -name "*.xml" -mtime +30 -delete

# Archive important results
tar -czf scan_archive_$(date +%Y%m).tar.gz scans/
```

This output management approach ensures professional documentation, tool interoperability, and efficient workflow in penetration testing engagements.
