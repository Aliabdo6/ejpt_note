# Nmap Firewall Detection & IDS Evasion

## Overview

This guide covers advanced Nmap techniques for detecting firewalls and evading Intrusion Detection Systems (IDS). These methods provide granular control over packet transmission to mask scanning activities and identify filtering mechanisms.

## Firewall Detection

### ACK Scan Technique

The ACK scan (`-sA`) is specifically designed to detect firewall configurations by sending packets with ACK flags set.

```bash
# Basic ACK scan for firewall detection
nmap -sA <target_ip>

# Target specific ports
nmap -sA -p 80,445,3389 <target_ip>
```

**Interpretation:**

- **Unfiltered**: No firewall interference detected
- **Filtered**: Stateful firewall likely present
- **Closed**: Port accessible but no service listening

### Windows Firewall Detection

```bash
# Standard SYN scan on Windows target
nmap -sS --top-ports 100 <target_ip>
```

**Key Indicators:**

- **Filtered ports** indicate Windows Firewall is active
- **Closed ports** suggest no firewall filtering
- Common filtered ports: 135, 139, 445

## IDS Evasion Techniques

### Packet Fragmentation

Break packets into smaller fragments to avoid pattern detection:

```bash
# Basic fragmentation
nmap -sS -f <target_ip>

# Custom MTU size (8-byte fragments)
nmap -sS --mtu 8 <target_ip>

# Combined with service detection
nmap -sS -sV -f -p 80,445,3389 <target_ip>
```

**Wireshark Analysis:**

- Fragmented IP packets with offset values
- Multiple packets reassembled at destination
- Harder for IDS to reconstruct scanning patterns

### Data Length Manipulation

Append random data to sent packets:

```bash
# Append 200 bytes of random data
nmap -sS --data-length 200 <target_ip>

# Combine with fragmentation
nmap -sS -f --data-length 200 <target_ip>
```

### Source Port Spoofing

Masquerade as common services:

```bash
# Spoof DNS source port
nmap -sS -g 53 <target_ip>

# Spoof HTTP source port
nmap -sS -g 80 <target_ip>

# Multiple evasion techniques combined
nmap -sS -f -g 53 --data-length 100 <target_ip>
```

### Decoy Scanning

Spoof source IP addresses to mask true origin:

```bash
# Single decoy IP
nmap -sS -D 192.168.1.1 <target_ip>

# Multiple decoy IPs
nmap -sS -D 192.168.1.1,192.168.1.2,192.168.1.3 <target_ip>

# Include real IP randomly (ME placeholder)
nmap -sS -D 192.168.1.1,ME,192.168.1.2 <target_ip>

# Comprehensive evasion scan
nmap -sS -sV -f -D 192.168.1.1,192.168.1.2 -g 53 --data-length 200 <target_ip>
```

**Network Analysis:**

- Packets appear to originate from decoy IPs
- Responses still come to real source
- Creates confusion for network monitoring

## Advanced Evasion Options

### TTL Manipulation

```bash
# Set custom Time-To-Live
nmap -sS --ttl 128 <target_ip>
```

### Scan Delay

```bash
# Add delay between packets (milliseconds)
nmap -sS --scan-delay 1000ms <target_ip>

# Randomize delay
nmap -sS --max-scan-delay 5000ms <target_ip>
```

### IP Options

```bash
# Send packets with IP options
nmap -sS --ip-options "S 192.168.1.1" <target_ip>
```

## Practical Evasion Scenarios

### Stealth Network Assessment

```bash
# Comprehensive evasion scan
nmap -sS -sV -f -D <gateway_ip>,<other_hosts> -g 53 --data-length 128 --ttl 64 --scan-delay 500ms -p- <target_ip>
```

**Example:**

```bash
nmap -sS -sV -f -D 10.10.23.1,10.10.23.2 -g 53 --data-length 128 -p 80,443,445,3389 10.10.23.45
```

### Quick Firewall Assessment

```bash
# Rapid firewall detection
nmap -sA -T4 --top-ports 50 <target_ip>

# Service detection through firewall
nmap -sS -sV -f -g 80 --data-length 64 <target_ip>
```

## Traffic Analysis Patterns

### Normal SYN Scan Traffic

- Complete SYN packets
- Predictable source ports
- Consistent timing
- Easy to fingerprint as Nmap

### Evaded SYN Scan Traffic

- Fragmented packets
- Spoofed source IPs
- Common service source ports
- Random data padding
- Variable timing

## Best Practices

### Evasion Strategy

1. **Start conservative**: Use minimal evasion initially
2. **Gradual escalation**: Add techniques if detected
3. **Network awareness**: Understand normal traffic patterns
4. **Legal compliance**: Only use on authorized networks

### Performance Considerations

- Fragmentation increases packet count
- Decoy scanning multiplies network traffic
- Delays significantly increase scan time
- Balance stealth with practicality

### Detection Avoidance

```bash
# Minimal footprint scan
nmap -sS -T2 -f -g 53 --data-length 32 --max-scan-delay 2s <target_ip>

# Aggressive but evasive
nmap -sS -T4 -f -D <decoy_ips> --data-length 64 --ttl 128 <target_ip>
```

## Defensive Perspective

### What Security Teams See

- **Fragmented scans**: Multiple small packets
- **Decoy attacks**: Apparent scans from multiple sources
- **Spoofed services**: Scans appearing as legitimate traffic
- **Padded packets**: Unusually large packet sizes

### Detection Challenges

- Reconstructing fragmented traffic
- Identifying true source among decoys
- Distinguishing from legitimate network issues
- Correlation across multiple evasion techniques

## Key Takeaways

- **ACK scans** effectively detect stateful firewalls
- **Fragmentation** breaks scanning patterns
- **Decoy IPs** create attribution confusion
- **Source port spoofing** mimics legitimate traffic
- **Data padding** alters packet signatures
- **Combined techniques** provide layered evasion

These advanced Nmap capabilities demonstrate the importance of understanding both offensive scanning techniques and defensive detection mechanisms for comprehensive network security assessment.

## Next Steps

After mastering evasion techniques, focus on:

- Timing template optimization (`-T` options)
- Scan performance tuning
- Network protocol deep diving
- Defensive countermeasure development

Remember: These techniques should only be used on networks you own or have explicit permission to test.
