# Network Mapping & Host Discovery Fundamentals

## Introduction to Network Mapping

Network mapping is the crucial first step in active information gathering during penetration testing. After passive reconnaissance, this process helps us understand the target network's structure, identify active systems, and plan subsequent attack strategies.

## The Network Mapping Process

### Core Objectives

- **Host Discovery**: Identify active devices on the network
- **Port Scanning**: Discover open ports and services
- **Service Enumeration**: Determine running service versions
- **OS Detection**: Fingerprint operating systems
- **Topology Mapping**: Understand network architecture
- **Security Measure Identification**: Detect firewalls and IDS/IPS

### Real-World Scenario

**Example Scope:**

```
Target Network: 200.200.0.0/16
IP Range: 200.200.0.0 - 200.200.255.255
Potential Hosts: 65,536 addresses
```

**Challenge:** Determine which of the 65,536 possible IP addresses are actually assigned to active hosts.

## Network Mapping Workflow

### Step 1: Host Discovery

- Identify live hosts within the target range
- Map IP addresses to active systems
- Build initial inventory of targets

### Step 2: Service Discovery

- Scan active hosts for open ports
- Identify running services
- Document service configurations

### Step 3: Network Topology

- Create network diagrams
- Identify routers, switches, firewalls
- Understand network segmentation

### Step 4: Security Assessment

- Detect security controls
- Identify filtering mechanisms
- Plan evasion strategies

## Nmap: The Network Mapper

### Core Capabilities

**Host Discovery**

- ICMP echo requests
- ARP requests (local networks)
- TCP/UDP probes
- Multiple protocol support

**Port Scanning**

- Comprehensive port range coverage
- Multiple scan techniques (SYN, Connect, UDP, etc.)
- Operating system agnostic

**Service Detection**

- Version detection across services
- Application fingerprinting
- Vulnerability correlation

**OS Fingerprinting**

- TCP/IP stack analysis
- Behavior-based detection
- Version-specific identification

## Practical Network Mapping Approach

### Initial Assessment

```bash
# Example: Starting with a /16 network (65,536 hosts)
# Objective: Identify which hosts are actually active
nmap -sn 200.200.0.0/16
```

### Progressive Discovery

1. **Broad Discovery**: Identify all active hosts
2. **Target Selection**: Focus on interesting systems
3. **Deep Enumeration**: Comprehensive service scanning
4. **Vulnerability Mapping**: Identify potential attack vectors

## Strategic Importance

### For Penetration Testers

- **Black Box Testing**: Start with zero knowledge of network structure
- **Attack Surface Identification**: Map all potential entry points
- **Resource Allocation**: Focus efforts on high-value targets
- **Compliance Reporting**: Document network scope and boundaries

### Business Context

- **Scope Validation**: Ensure testing stays within authorized boundaries
- **Risk Assessment**: Understand overall network exposure
- **Remediation Planning**: Provide actionable security improvements

## Key Terminology

**CIDR Notation**

- `/16` = 65,536 hosts (200.200.0.0 - 200.200.255.255)
- `/24` = 256 hosts (e.g., 192.168.1.0 - 192.168.1.255)
- Used to specify network size and scope

**Dynamic vs Static IPs**

- Dynamic: Automatically assigned (DHCP)
- Static: Manually configured
- Both require discovery in penetration testing

## Next Steps

In the following sections, we'll dive into practical host discovery techniques using Nmap, exploring various methods to identify active systems across different network environments and scenarios.

---

_This foundation in network mapping principles sets the stage for effective host discovery and comprehensive network assessment during penetration testing engagements._
