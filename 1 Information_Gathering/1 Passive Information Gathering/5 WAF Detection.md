# WAF Detection with WAFW00F Guide

## Overview

Web Application Firewall (WAF) detection is a crucial reconnaissance step that identifies security controls protecting target web applications. WAFW00F is the industry-standard tool for fingerprinting WAF solutions through passive analysis of HTTP responses.

## Understanding Web Application Firewalls

### What is a WAF?

A Web Application Firewall is a security solution that monitors, filters, and blocks HTTP traffic to and from web applications. WAFs protect against attacks like SQL injection, XSS, and other application-layer threats.

### Common WAF Providers

- **Cloudflare**: Enterprise CDN and security
- **Sucuri**: Website security platform
- **Imperva**: Application delivery and security
- **AWS WAF**: Amazon's web application firewall
- **ModSecurity**: Open-source WAF engine
- **Akamai**: Cloud security solutions

## WAFW00F Installation and Setup

### Kali Linux (Pre-installed)

```bash
# Verify installation
which wafw00f

# Check version
wafw00f --version
```

### Manual Installation

```bash
# Clone repository
git clone https://github.com/EnableSecurity/wafw00f.git
cd wafw00f

# Install dependencies
pip install -r requirements.txt

# Install package
python setup.py install
```

## Basic Usage

### Listing Detectable WAFs

```bash
wafw00f -l
```

**Example Output:**

```
[+] Can test for these WAFs:

Cloudflare
Sucuri
Incapsula
ModSecurity
AWS WAF
Akamai
Barracuda
FortiWeb
[...]
```

### Basic WAF Detection

```bash
# Single domain detection
wafw00f target-domain.com

# With full URL specification
wafw00f https://target-domain.com
```

## Advanced Usage

### Comprehensive Detection

```bash
# Test for all possible WAF instances
wafw00f -a target-domain.com

# Verbose output with detailed analysis
wafw00f -v target-domain.com

# Save results to file
wafw00f target-domain.com -o waf_results.txt
```

### Batch Domain Analysis

```bash
# Multiple domains from file
wafw00f -i domains.txt

# Create domains file
echo "domain1.com" > domains.txt
echo "domain2.com" >> domains.txt
echo "domain3.com" >> domains.txt

# Process multiple domains
wafw00f -i domains.txt -o batch_results.txt
```

## Practical Examples

### Basic Detection Script

```bash
#!/bin/bash

# WAF Detection Script
# Usage: ./waf_detect.sh -d domain.com -o output_file

while getopts "d:o:" opt; do
  case $opt in
    d) DOMAIN="$OPTARG" ;;
    o) OUTPUT="$OPTARG" ;;
    *) echo "Usage: $0 -d domain -o output_file" >&2
       exit 1 ;;
  esac
done

if [[ -z "$DOMAIN" ]]; then
    echo "Domain required"
    exit 1
fi

echo "[*] Starting WAF detection for: $DOMAIN"
echo "[*] Timestamp: $(date)"

# Perform WAF detection
wafw00f -v "$DOMAIN" ${OUTPUT:+-o $OUTPUT}

echo "[+] WAF detection complete"
```

### Multi-Domain Campaign Analysis

```bash
#!/bin/bash

# Analyze multiple related domains
DOMAINS=("primary.com" "api.primary.com" "admin.primary.com" "staging.primary.com")

for domain in "${DOMAINS[@]}"; do
    echo "Analyzing: $domain"
    wafw00f "$domain" | grep -E "(behind|not behind)"
    echo "---"
done
```

## Output Interpretation

### Positive Detection

```
[~] The site https://target-domain.com is behind Cloudflare WAF.
[~] Number of requests: 5
```

### Negative Detection

```
[~] The site https://target-domain.com does not seem to be behind a WAF.
[~] Number of requests: 3
```

### Inconclusive Results

```
[~] The site https://target-domain.com seems to be behind a WAF.
[~] Could not identify the WAF solution.
```

## Integration with Reconnaissance Workflow

### Comprehensive Security Assessment

```bash
#!/bin/bash

# Integrated reconnaissance with WAF detection
DOMAIN="$1"
OUTPUT_DIR="recon_$DOMAIN"

mkdir -p "$OUTPUT_DIR"

echo "[*] Starting comprehensive reconnaissance for: $DOMAIN"

# WAF Detection
echo "[+] Detecting WAF..."
wafw00f "$DOMAIN" > "$OUTPUT_DIR/waf_detection.txt"

# DNS Enumeration
echo "[+] Performing DNS reconnaissance..."
dnsrecon -d "$DOMAIN" > "$OUTPUT_DIR/dns_records.txt"

# WHOIS Lookup
echo "[+] Gathering WHOIS information..."
whois "$DOMAIN" > "$OUTPUT_DIR/whois.txt"

echo "[+] Reconnaissance complete. Results in: $OUTPUT_DIR"
```

## Advanced Analysis Techniques

### WAF Bypass Research

```bash
# Identify WAF for security testing preparation
wafw00f target.com > waf_info.txt

# Research identified WAF for bypass techniques
echo "[*] WAF Identified: $(grep "behind" waf_info.txt)"
echo "[*] Research bypass techniques for specific WAF"
```

### Historical WAF Detection

```bash
# Track WAF changes over time
#!/bin/bash
DOMAIN="$1"
DATE=$(date +%Y%m%d)

wafw00f "$DOMAIN" > "waf_${DOMAIN}_${DATE}.txt"

# Compare with previous results
if [[ -f "waf_${DOMAIN}_previous.txt" ]]; then
    diff "waf_${DOMAIN}_previous.txt" "waf_${DOMAIN}_${DATE}.txt"
fi
```

## Operational Security Considerations

### Stealth Operations

- **Passive Detection**: WAFW00F uses normal HTTP requests
- **Low Footprint**: Minimal interaction with target
- **Legal Compliance**: Publicly accessible information

### Rate Limiting

```bash
# Add delays between multiple scans
for domain in $(cat domains.txt); do
    wafw00f "$domain"
    sleep 5  # 5-second delay between scans
done
```

## Real-World Scenarios

### Penetration Testing

```bash
# Pre-engagement WAF assessment
./waf_detect.sh -d client-domain.com -o pre_engagement_waf.txt

# Document WAF presence for testing scope
echo "WAF Status: $(grep "behind" pre_engagement_waf.txt)" >> scope_documentation.txt
```

### Bug Bounty Hunting

```bash
# Quick WAF check for multiple targets
while read domain; do
    result=$(wafw00f "$domain" | grep "behind")
    if [[ -n "$result" ]]; then
        echo "PROTECTED: $domain - $result"
    else
        echo "UNPROTECTED: $domain"
    fi
done < bounty_targets.txt
```

### Security Monitoring

```bash
# Monitor organizational WAF deployment
#!/bin/bash
DOMAINS_FILE="company_domains.txt"
LOG_FILE="waf_monitoring_$(date +%Y%m%d).log"

while read domain; do
    echo "Checking: $domain" >> "$LOG_FILE"
    wafw00f "$domain" >> "$LOG_FILE"
    echo "---" >> "$LOG_FILE"
done < "$DOMAINS_FILE"
```

## Troubleshooting and Validation

### False Positive Reduction

```bash
# Use multiple detection methods
wafw00f -a target.com  # All checks
wafw00f -v target.com  # Verbose mode

# Cross-validate with other tools
whatweb target.com | grep -i waf
nmap --script http-waf-detect target.com
```

### Network Issues

```bash
# Check connectivity before scanning
ping -c 1 target.com
curl -I https://target.com

# Use specific HTTP ports if needed
wafw00f http://target.com:8080
```

## Best Practices

### Ethical Usage

- **Authorization**: Only test domains you own or have permission to test
- **Compliance**: Adhere to applicable laws and regulations
- **Documentation**: Maintain records of authorization and findings

### Operational Efficiency

- **Batch Processing**: Use file input for multiple domains
- **Output Management**: Save results for analysis and reporting
- **Integration**: Combine with other reconnaissance tools

### Technical Considerations

- **Validation**: Cross-check results with multiple methods
- **Context**: Consider business impact of WAF presence
- **Evolution**: Keep tool updated for new WAF signatures

## Limitations and Considerations

### Detection Challenges

- **Custom WAFs**: Proprietary solutions may not be detected
- **Configuration Variations**: Same WAF may behave differently
- **Evasion Techniques**: Some WAFs may attempt to hide their presence

### Environmental Factors

- **Network Conditions**: Firewalls and proxies may affect detection
- **Geographic Variations**: CDN configurations may differ by region
- **Timing**: WAF rules may change based on traffic patterns

## Next Steps

After WAF detection, proceed to:

- **Subdomain Enumeration**: Expand attack surface mapping
- **Service Discovery**: Identify additional services
- **Vulnerability Assessment**: WAF-specific testing approaches
- **Security Control Bypass**: Research identified WAF solutions

---

_WAF detection is a fundamental step in authorized security assessments. Always ensure proper authorization and ethical usage of these techniques for legitimate security testing purposes._
