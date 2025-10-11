# Nmap Scan Optimization: Timing & Performance

## Overview

Optimizing Nmap scans involves balancing speed, stealth, and accuracy. This guide covers timing templates and performance options to control scan duration and network footprint.

## Timing Templates

Nmap provides six built-in timing templates that control scan aggressiveness:

### Template Overview

| Template | Name       | Description                          | Use Case                       |
| -------- | ---------- | ------------------------------------ | ------------------------------ |
| `-T0`    | Paranoid   | Slowest, 5min between packets        | IDS evasion, extreme stealth   |
| `-T1`    | Sneaky     | 15s between packets                  | IDS evasion, high stealth      |
| `-T2`    | Polite     | 0.4s between packets                 | Reduced network impact         |
| `-T3`    | Normal     | **Default** balanced timing          | General scanning               |
| `-T4`    | Aggressive | Faster parallel scanning             | Reliable networks              |
| `-T5`    | Insane     | Maximum speed, potential packet loss | Fast networks, acceptable loss |

## Practical Usage

### Stealth Scanning

```bash
# Paranoid - maximum stealth
nmap -T0 -sS <target_ip>

# Sneaky - high stealth with reasonable timing
nmap -T1 -sS <target_ip>

# Polite - reduced network impact
nmap -T2 -sS <target_ip>
```

### Performance Scanning

```bash
# Aggressive - faster scanning
nmap -T4 -sS <target_ip>

# Insane - maximum speed
nmap -T5 -sS <target_ip>

# Combined with service detection
nmap -T4 -sS -sV <target_ip>
```

## Advanced Timing Controls

### Host Timeout

Set maximum time per host before moving on:

```bash
# Timeout after 30 seconds per host
nmap --host-timeout 30s <target_range>

# Timeout after 5 minutes
nmap --host-timeout 5m <target_range>

# Practical example with subnet scanning
nmap -sS --host-timeout 30s 192.168.1.0/24
```

**Best Practice:** Use 30-second timeout for large networks to skip unresponsive hosts.

### Scan Delay

Control time between packet probes:

```bash
# 5-second delay between probes
nmap --scan-delay 5s <target_ip>

# 15-second delay for high stealth
nmap --scan-delay 15s <target_ip>

# Millisecond precision
nmap --scan-delay 500ms <target_ip>
```

### Max Retries

```bash
# Reduce retry attempts for speed
nmap --max-retries 1 <target_ip>

# Increase for unreliable networks
nmap --max-retries 5 <target_ip>
```

## Real-World Scenarios

### Large Network Assessment

```bash
# Balance speed and completeness
nmap -T4 --host-timeout 30s --max-retries 2 -sS -p- 192.168.1.0/24
```

### Stealth Penetration Testing

```bash
# Maximum stealth approach
nmap -T1 --scan-delay 15s -sS -f -D <decoy_ips> <target_ip>
```

### Quick Service Discovery

```bash
# Fast, comprehensive scan
nmap -T5 -sS -sV --top-ports 1000 <target_ip>
```

## Performance Analysis

### Network Traffic Patterns

**Normal Timing (`-T3`):**

- Consistent packet flow
- Moderate network impact
- Balanced performance

**Aggressive Timing (`-T4`):**

- High packet frequency
- Potential network noise
- Faster completion

**Stealth Timing (`-T1`):**

- Sparse packet distribution
- Minimal network footprint
- Extended scan duration

### Scan Duration Comparison

Example scan times for 1000 ports:

| Template | Estimated Time | Network Impact |
| -------- | -------------- | -------------- |
| `-T0`    | ~83 hours      | Minimal        |
| `-T1`    | ~4 hours       | Very Low       |
| `-T2`    | ~7 minutes     | Low            |
| `-T3`    | ~1 minute      | Moderate       |
| `-T4`    | ~30 seconds    | High           |
| `-T5`    | ~15 seconds    | Very High      |

## Best Practices

### Template Selection Guidelines

```bash
# Internal networks - speed prioritized
nmap -T4 -sS -sV <internal_target>

# External assessments - stealth prioritized
nmap -T2 -sS --scan-delay 2s <external_target>

# Unknown environments - start conservative
nmap -T3 -sS --max-retries 3 <target>
```

### Performance Optimization

```bash
# Combine timing with other optimizations
nmap -T4 --min-rate 100 --max-rate 500 -sS <target_ip>

# Limit ports for speed
nmap -T4 --top-ports 1000 <target_ip>

# Parallel host scanning
nmap -T4 --min-hostgroup 64 --min-parallelism 10 <target_range>
```

### Stealth Optimization

```bash
# Comprehensive stealth approach
nmap -T1 --scan-delay 10s --max-scan-delay 20s -sS -f <target_ip>

# Randomize timing for evasion
nmap -T2 --randomize-hosts --scan-delay 500ms,2s <target_range>
```

## Traffic Analysis

### Wireshark Observations

**Fast Scans (`-T4/-T5`):**

- Packets sent in rapid succession
- High packet density in short timeframes
- Easily detectable as scanning activity

**Slow Scans (`-T0/-T1`):**

- Sparse packet distribution
- Mimics legitimate network traffic
- Difficult to distinguish from background noise

### Defensive Perspective

Security teams monitor for:

- High-frequency sequential port access
- Consistent timing patterns
- Multiple connection attempts in short periods

## Troubleshooting

### Common Issues

**Slow Scans:**

- Check network connectivity
- Verify target responsiveness
- Adjust timing templates upward

**Incomplete Results:**

- Increase timeout values
- Reduce parallel scanning
- Check for rate limiting

**Packet Loss:**

- Decrease scan aggressiveness
- Increase retry attempts
- Verify network stability

## Key Takeaways

1. **Timing templates** provide quick performance presets
2. **Host timeout** prevents hanging on unresponsive targets
3. **Scan delay** controls stealth through packet spacing
4. **Template selection** depends on network environment and goals
5. **Combined approaches** balance speed and detection risk

## Recommended Defaults

### Internal Network Scanning

```bash
nmap -T4 -sS -sV --host-timeout 30s <target>
```

### External Penetration Testing

```bash
nmap -T2 -sS --scan-delay 2s --max-retries 2 <target>
```

### Comprehensive Assessment

```bash
nmap -T3 -A --host-timeout 60s <target>
```

These optimization techniques enable efficient network assessment while maintaining appropriate stealth levels for different engagement types.
