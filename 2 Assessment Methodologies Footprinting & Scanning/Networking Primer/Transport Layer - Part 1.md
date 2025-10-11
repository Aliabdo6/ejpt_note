# Transport Layer Fundamentals for Penetration Testing

## Overview

The transport layer (Layer 4 of the OSI model) is arguably the most important layer for penetration testers to understand, especially when working with tools like Nmap. This layer facilitates direct communication between devices and handles critical functions like error detection, flow control, and data segmentation.

## Core Transport Layer Functions

### Primary Responsibilities

- **End-to-End Communication**: Direct communication between two specific devices
- **Reliable Data Transfer**: Ensures accurate and ordered delivery
- **Error Detection**: Identifies and handles transmission errors
- **Flow Control**: Manages data transmission rates
- **Segmentation**: Breaks data into smaller manageable units

## Key Transport Protocols

### TCP (Transmission Control Protocol)

**Connection-Oriented Protocol**

- Establishes formal connection before data exchange
- Provides reliable, ordered data delivery
- Includes error checking and retransmission
- Slower but more reliable than UDP

**Key Characteristics:**

- **Connection-Oriented**: Requires handshake before data transfer
- **Reliable Delivery**: Guarantees data arrives intact and in order
- **Ordered Transfer**: Reassembles data in correct sequence
- **Flow Control**: Prevents overwhelming receiving device

### UDP (User Datagram Protocol)

**Connectionless Protocol**

- No pre-establishment connection required
- Faster but no delivery guarantees
- No error checking or retransmission
- Ideal for real-time applications

## TCP Three-Way Handshake

### Connection Establishment Process

The TCP three-way handshake creates a reliable connection before any data transmission occurs:

```
Client                  Server
  |                      |
  |--- SYN ------------->|
  |                      |
  |<-- SYN-ACK ----------|
  |                      |
  |--- ACK ------------->|
  |                      |
```

**Step-by-Step Breakdown:**

1. **SYN (Synchronize)**

   - Client sends TCP segment with SYN flag set
   - Indicates intention to establish connection
   - Includes initial sequence number (ISN)

2. **SYN-ACK (Synchronize-Acknowledge)**

   - Server responds with SYN and ACK flags set
   - Acknowledges client's SYN
   - Provides server's own ISN

3. **ACK (Acknowledge)**
   - Client sends ACK to confirm connection
   - Both devices can now exchange data

### Practical Significance

- **Port Scanning**: Nmap uses SYN packets to detect open ports
- **Service Identification**: Handshake success indicates service availability
- **Network Analysis**: Handshake failures reveal firewall rules or filtering

## TCP Header Structure

### Key Header Fields

```
┌─────────────────────────────────────────────────────────┐
│                     TCP Header (20+ bytes)               │
├───────────────────┬───────────────────┬─────────────────┤
│ Source Port       │ Destination Port  │                 │
│ (16 bits)         │ (16 bits)         │                 │
├───────────────────┴───────────────────┴─────────────────┤
│ Sequence Number (32 bits)                               │
├─────────────────────────────────────────────────────────┤
│ Acknowledgment Number (32 bits)                         │
├─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────────┤
│ Data│     │ U R G│ A C K│ P S H│ R S T│ S Y N│ F I N│ │
│Offset│Resvd│     │     │     │     │     │     │       │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────────┘
```

### Critical Control Flags

- **URG**: Urgent pointer field significant
- **ACK**: Acknowledgment field significant
- **PSH**: Push function
- **RST**: Reset connection
- **SYN**: Synchronize sequence numbers
- **FIN**: No more data from sender

## TCP Port Ranges and Classification

### Port Number Ranges

- **Total Range**: 1 - 65,535 (16-bit unsigned integers)

### Standard Port Categories

**Well-Known Ports (0-1023)**

- Reserved for fundamental services
- Standardized by IANA (Internet Assigned Numbers Authority)
- Examples:
  - `22/tcp` - SSH
  - `80/tcp` - HTTP
  - `443/tcp` - HTTPS
  - `21/tcp` - FTP
  - `25/tcp` - SMTP

**Registered Ports (1024-49151)**

- Assigned to specific applications
- Not strictly standardized but commonly recognized
- Examples:
  - `1433/tcp` - Microsoft SQL Server
  - `3306/tcp` - MySQL
  - `3389/tcp` - RDP (Remote Desktop)
  - `5432/tcp` - PostgreSQL

**Dynamic/Private Ports (49152-65535)**

- Used for temporary or private connections
- Typically assigned dynamically to client applications

## Practical Analysis with Wireshark

### TCP Session Analysis

```bash
# Start Wireshark and filter for TCP traffic
tcp

# Look for three-way handshake patterns:
# 1. SYN → SYN-ACK → ACK (connection establishment)
# 2. Data exchange with ACK flags
# 3. FIN packets (connection termination)
```

### Key Analysis Points

- **Sequence Numbers**: Track data ordering
- **ACK Numbers**: Confirm received data
- **Window Size**: Flow control mechanism
- **Flags**: Connection state indicators

## Penetration Testing Applications

### Port Scanning Fundamentals

```bash
# TCP SYN Scan (most common)
nmap -sS target_ip

# TCP Connect Scan (completes handshake)
nmap -sT target_ip

# Service version detection
nmap -sV target_ip
```

### Service Identification

- **Well-known ports** provide immediate service context
- **Unexpected services** on standard ports indicate potential misconfigurations
- **Multiple services** on single host reveal attack surface

### Connection Analysis

- **Successful handshakes** indicate accessible services
- **RST responses** suggest filtered or closed ports
- **No response** may indicate firewall blocking

## Practical Exercises

### Exercise 1: TCP Handshake Analysis

```bash
# Capture TCP traffic during web browsing
wireshark &
curl http://www.example.com

# Analyze captured packets for:
# 1. Three-way handshake to example.com:80
# 2. HTTP request/response sequence
# 3. Connection termination
```

### Exercise 2: Port Behavior Testing

```bash
# Test different port states
nmap -p 22,80,443,9999 target_ip

# Observe responses:
# - Open: SYN-ACK received
# - Closed: RST received
# - Filtered: No response
```

## Key Takeaways for Penetration Testers

### Critical Understanding

- TCP provides reliable, ordered communication essential for most services
- UDP offers speed at the cost of reliability for time-sensitive applications
- Port numbers map to specific services and applications
- Three-way handshake reveals service availability and network filtering

### Practical Applications

- **Service Enumeration**: Identify running services by port
- **Network Mapping**: Understand host communication patterns
- **Firewall Analysis**: Detect filtering rules through connection responses
- **Vulnerability Assessment**: Identify misconfigured services

---

_This transport layer foundation enables effective port scanning, service enumeration, and network analysis - core skills for any penetration tester working with network assessment tools._
