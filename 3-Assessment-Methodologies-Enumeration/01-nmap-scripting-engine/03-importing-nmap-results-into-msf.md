# Importing Nmap Scan Results into Metasploit Framework

## Overview

This guide covers the process of importing Nmap scan results into Metasploit Framework's database and demonstrates how to perform Nmap scans directly from within Metasploit. This integration enables seamless transition from reconnaissance to vulnerability analysis and exploitation.

## Prerequisites

- Nmap scan results in XML format (`-oX` flag)
- PostgreSQL database service running
- Metasploit Framework installed
- Proper workspace configuration

## Database Setup and Verification

### Starting PostgreSQL Service

```bash
# Start PostgreSQL service
sudo service postgresql start

# Alternatively on modern systems
sudo systemctl start postgresql

# Verify service status
sudo systemctl status postgresql
```

### Launching Metasploit Framework

```bash
# Start Metasploit console
msfconsole

# Verify database connection
msf6 > db_status
[*] Connected to msf. Connection type: postgresql.
```

## Workspace Management

### Creating and Managing Workspaces

```bash
# List all workspaces
msf6 > workspace

# Create new workspace
msf6 > workspace -a windows_scan

# Switch to specific workspace
msf6 > workspace windows_scan

# Delete workspace
msf6 > workspace -d old_workspace

# Rename workspace
msf6 > workspace -r old_name new_name
```

### Dynamic Workspace Creation Script

```bash
#!/bin/bash

# Create timestamped workspace
WORKSPACE_NAME="scan_$(date +%Y%m%d_%H%M%S)"
TARGET=$1

echo "[*] Creating workspace: $WORKSPACE_NAME"
echo "[*] Target: $TARGET"

msfconsole -q -x "
workspace -a $WORKSPACE_NAME;
db_nmap -Pn -sV -O $TARGET;
hosts;
services;
exit"
```

**Usage**:

```bash
chmod +x create_workspace.sh
./create_workspace.sh 192.168.1.100
```

## Importing Nmap XML Results

### Basic Import Process

```bash
# Import Nmap XML file into current workspace
msf6 > db_import /path/to/nmap_scan.xml

# Example with actual file
msf6 > db_import /root/scan_results.xml
```

### Verifying Imported Data

```bash
# View imported hosts
msf6 > hosts

# View discovered services
msf6 > services

# Detailed service information
msf6 > services -u

# View by specific port
msf6 > services -p 80,443
```

### Sample Import Workflow

```bash
# Generate Nmap scan with XML output
nmap -Pn -sV -O -oX comprehensive_scan.xml 192.168.1.0/24

# Import into Metasploit
msf6 > workspace -a corporate_network
msf6 > db_import /path/to/comprehensive_scan.xml

# Verify import
msf6 > hosts
msf6 > services
```

## Direct Nmap Scanning from Metasploit

### Using db_nmap Command

```bash
# Basic scan from within Metasploit
msf6 > db_nmap -Pn 192.168.1.100

# Comprehensive scan with service detection
msf6 > db_nmap -Pn -sV -O 192.168.1.100

# Network range scan
msf6 > db_nmap -Pn -sS 192.168.1.0/24

# Specific port range
msf6 > db_nmap -Pn -p 1-1000 192.168.1.100
```

### Advanced db_nmap Examples

```bash
# Stealth SYN scan with timing
msf6 > db_nmap -sS -T4 --top-ports 1000 192.168.1.100

# UDP scan for common services
msf6 > db_nmap -sU -p 53,67,68,69,123,161 192.168.1.100

# Comprehensive network assessment
msf6 > db_nmap -sS -sU -sV -O -p- 192.168.1.100
```

## Practical Lab Setup

For hands-on practice, use the **VulnHub Kioptrix Level 1** machine:

**Lab URL**: [Kioptrix Level 1 on VulnHub](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/)

**Setup Instructions**:

1. Download the Kioptrix Level 1 VM
2. Import into VirtualBox/VMware
3. Set network adapter to NAT or Bridged
4. Start the machine and identify its IP address

**Default Credentials**: Not required for scanning phase

## Complete Workflow Example

### Scenario: Corporate Network Assessment

```bash
#!/bin/bash

NETWORK_RANGE=$1
WORKSPACE_NAME="corp_assessment_$(date +%Y%m%d)"

echo "[*] Starting corporate network assessment"
echo "[*] Target: $NETWORK_RANGE"
echo "[*] Workspace: $WORKSPACE_NAME"

# Perform comprehensive Nmap scan
nmap -sS -sV -O -p- --open -oX full_scan.xml $NETWORK_RANGE

# Import into Metasploit
msfconsole -q -x "
workspace -a $WORKSPACE_NAME;
db_import /root/full_scan.xml;
echo '[+] Hosts discovered:';
hosts;
echo '[+] Services found:';
services;
echo '[+] Assessment complete';
exit"

echo "[*] Results stored in workspace: $WORKSPACE_NAME"
```

**Usage**:

```bash
chmod +x network_assessment.sh
./network_assessment.sh 192.168.1.0/24
```

## Data Management and Analysis

### Querying Imported Data

```bash
# Find specific services
msf6 > services -s http -c port,info

# Search for Windows hosts
msf6 > hosts -c address,os_name -S Windows

# List hosts with specific ports open
msf6 > services -p 22 -c host,port,info

# Export results
msf6 > hosts -o /tmp/hosts.csv
msf6 > services -o /tmp/services.csv
```

### Advanced Data Filtering

```bash
# Hosts with web services
msf6 > services -s http,https --rhosts

# Hosts running specific software
msf6 > services -S "Apache" -c host,port,info

# Hosts with unknown services
msf6 > services -u -c host,port,name
```

## Integration with Auxiliary Modules

### Using Imported Data with Scanners

```bash
# Automatically target all imported web servers
msf6 > services -s http --rhosts -o /tmp/web_hosts.txt
msf6 > use auxiliary/scanner/http/http_version
msf6 > set RHOSTS file:/tmp/web_hosts.txt
msf6 > run

# Target all SSH servers
msf6 > services -s ssh --rhosts -o /tmp/ssh_hosts.txt
msf6 > use auxiliary/scanner/ssh/ssh_version
msf6 > set RHOSTS file:/tmp/ssh_hosts.txt
msf6 > run
```

## Best Practices

### Workspace Organization

1. **Project-based Workspaces**: Create separate workspaces for different engagements
2. **Timestamp Naming**: Include dates in workspace names for tracking
3. **Regular Backups**: Export important data from workspaces
4. **Cleanup**: Remove obsolete workspaces to maintain performance

### Scan Optimization

```bash
# Balanced approach for speed and stealth
msf6 > db_nmap -sS -T4 --max-rtt-timeout 500ms --host-timeout 15m 192.168.1.0/24

# Comprehensive but slower scan
msf6 > db_nmap -sS -sV -O -p- --script default 192.168.1.100
```

### Error Handling and Validation

```bash
# Verify XML file before import
msf6 > db_import /path/to/scan.xml
[*] Importing 'Nmap XML' data
[*] Import: Parsing with 'Nokogiri v1.10.8'
[*] Successfully imported /path/to/scan.xml

# Check for import errors
msf6 > hosts
# If no hosts appear, check XML format and file permissions
```

## Troubleshooting Common Issues

### Database Connection Problems

```bash
# Restart database service
sudo service postgresql restart

# Reinitialize Metasploit database
msfdb reinit

# Check database status
msf6 > db_status
```

### Import Failures

- Ensure Nmap output is in XML format (`-oX`)
- Verify file permissions and path
- Check XML file structure for corruption
- Validate Nmap version compatibility

### Workspace Issues

```bash
# List all workspaces to verify current
msf6 > workspace

# Reset to default workspace if needed
msf6 > workspace default

# Check workspace-specific data
msf6 > workspace windows_scan
msf6 > hosts
```

## Next Steps

After successful import:

1. Use `vulns` command to check for detected vulnerabilities
2. Launch service-specific auxiliary modules for deeper enumeration
3. Generate reports using `db_export`
4. Proceed to vulnerability analysis and exploitation phases

This integration creates a powerful workflow where reconnaissance data seamlessly transitions into the exploitation phase, maintaining comprehensive tracking and documentation throughout the penetration testing process.
