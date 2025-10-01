# Host Discovery with Nmap

## Understanding Network Discovery

When performing penetration testing on internal networks, your first objective is to identify all active devices on the target network. This host discovery phase is crucial for mapping the attack surface before proceeding to port scanning and vulnerability assessment.

## Nmap Fundamentals

Nmap (Network Mapper) is the industry standard for network exploration and security auditing. It's designed to rapidly scan large networks and provides comprehensive information about discovered hosts.

### Key Nmap Host Discovery Options

```bash
# Basic ping sweep (host discovery only)
sudo nmap -sn 192.168.2.0/24

# Host discovery with OS detection
sudo nmap -sn -O 192.168.2.0/24

# Host discovery with verbose output
sudo nmap -sn -v 192.168.2.0/24
```

## Practical Implementation

### Determining Your Network Range

First, identify your current network configuration:

```bash
# Check network interfaces and IP addresses
ip a
# or
ip addr show

# Alternative methods
ifconfig
hostname -I
```

**Example Output:**

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    inet 192.168.2.190/24 brd 192.168.2.255 scope global eth0
```

### Performing Ping Sweeps

```bash
# Basic subnet scan
sudo nmap -sn 192.168.2.0/24

# Expected output
Starting Nmap 7.91 ( https://nmap.org ) at 2023-01-01 10:00 UTC
Nmap scan report for router.local (192.168.2.1)
Host is up (0.0010s latency).
MAC Address: AA:BB:CC:DD:EE:FF (Xiaomi Electronics)
Nmap scan report for workstation.local (192.168.2.2)
Host is up (0.0020s latency).
MAC Address: 11:22:33:44:55:66 (ASRock)
Nmap scan report for kali.local (192.168.2.190)
Host is up.
Nmap done: 256 IP addresses (3 hosts up) scanned in 2.50 seconds
```

## Advanced Host Discovery Techniques

### Using netdiscover for ARP-based Discovery

```bash
# Install netdiscover if not available
sudo apt update && sudo apt install netdiscover

# ARP-based network discovery
sudo netdiscover -i eth0 -r 192.168.2.0/24

# Example output
Currently scanning: 192.168.2.0/24   |   Screen View: Unique Hosts
3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180
IP            At MAC Address     Count     Len  MAC Vendor / Hostname
192.168.2.1   aa:bb:cc:dd:ee:ff      1      60  Xiaomi Electronics
192.168.2.2   11:22:33:44:55:66      1      60  ASRock
192.168.2.190 66:55:44:33:22:11      1      60  PCS Systemtechnik GmbH
```

### Custom Host Discovery Script

```bash
#!/bin/bash

# Network discovery automation script
NETWORK=${1:-192.168.2.0/24}
OUTPUT_FILE="host_discovery_$(date +%Y%m%d_%H%M%S).txt"

echo "[*] Starting comprehensive host discovery for: $NETWORK"
echo "[*] Output file: $OUTPUT_FILE"

# Method 1: Nmap ping sweep
echo "[*] Performing Nmap ping sweep..."
sudo nmap -sn $NETWORK | tee -a $OUTPUT_FILE

echo "----------------------------------------" >> $OUTPUT_FILE

# Method 2: ARP discovery (if on local network)
echo "[*] Performing ARP discovery..."
sudo netdiscover -r $NETWORK -P >> $OUTPUT_FILE 2>/dev/null

# Method 3: Extract just IP addresses for further processing
echo "[*] Extracting active IP addresses..."
grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' $OUTPUT_FILE | sort -u > "active_hosts_$(date +%Y%m%d_%H%M%S).txt"

echo "[*] Host discovery complete"
echo "[*] Active hosts saved to: active_hosts_$(date +%Y%m%d_%H%M%S).txt"
```

**Usage:**

```bash
./host_discovery.sh 192.168.1.0/24
./host_discovery.sh 10.0.0.0/16
```

## Dynamic Network Scanning

### Multi-network Scanner

```bash
#!/bin/bash

# Scan multiple common private network ranges
NETWORKS=(
    "192.168.0.0/24"
    "192.168.1.0/24"
    "192.168.2.0/24"
    "10.0.0.0/24"
    "172.16.0.0/24"
)

echo "[*] Scanning multiple network ranges..."

for network in "${NETWORKS[@]}"; do
    echo "[*] Scanning network: $network"
    sudo nmap -sn $network | grep -E "(Nmap scan report|MAC Address)"
    echo "---"
done
```

### Targeted Host Discovery with Custom Ranges

```bash
#!/bin/bash

TARGET_FILE=${1:-targets.txt}
TIMEOUT=${2:-2}

if [[ ! -f "$TARGET_FILE" ]]; then
    echo "Usage: $0 <targets_file> [timeout]"
    echo "Targets file should contain one IP or range per line"
    exit 1
fi

echo "[*] Starting targeted host discovery..."
echo "[*] Timeout: ${TIMEOUT}s per host"

while IFS= read -r target; do
    [[ -z "$target" ]] && continue
    [[ "$target" =~ ^# ]] && continue

    echo "[*] Scanning: $target"
    sudo nmap -sn --host-timeout ${TIMEOUT}s $target | \
        grep -E "(Nmap scan report|Host is up)"
done < "$TARGET_FILE"
```

**Example targets.txt:**

```
192.168.2.0/24
10.0.0.1-50
172.16.1.100-200
# This is a comment
```

## Understanding the Results

### Interpreting Nmap Output

- **Host is up**: Device responded to discovery probes
- **MAC Address**: Physical address and manufacturer information
- **Scan statistics**: Total hosts scanned vs. hosts found
- **Response times**: Latency information for each host

### Common Manufacturer MAC Prefixes

- Xiaomi: Various prefixes for networking equipment
- ASRock: Motherboard and computer hardware
- Apple, Dell, HP: Common computer manufacturers
- Cisco, Netgear: Networking equipment

## Best Practices for Host Discovery

### Stealth Considerations

```bash
# Slower scan to avoid detection
sudo nmap -sn --max-rate 10 192.168.2.0/24

# Randomize host order
sudo nmap -sn --randomize-hosts 192.168.2.0/24
```

### Comprehensive Discovery

```bash
# Combine multiple discovery methods
sudo nmap -sn -PE -PS21,22,23,25,80,443,3389 192.168.2.0/24
```

### Output Management

```bash
# Save results in multiple formats
sudo nmap -sn -oA host_discovery 192.168.2.0/24

# Generate target list for next phase
sudo nmap -sn 192.168.2.0/24 | grep -Eo '([0-9]{1,3}\.){3}[0-9]{1,3}' > targets.txt
```

## Next Steps After Host Discovery

Once you've identified active hosts, proceed to:

1. **Port Scanning**: `nmap -sS -sV -O target_ip`
2. **Service Enumeration**: Identify running services and versions
3. **Vulnerability Assessment**: Scan for known vulnerabilities
4. **Exploitation**: Attempt to compromise identified services

---

_This documentation reflects my practical approach to network host discovery. Always ensure you have proper authorization before scanning any network. The techniques shown here are for educational purposes and legitimate security testing._
