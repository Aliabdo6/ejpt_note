# Subdomain Enumeration with Sublist3r Guide

## Overview

Sublist3r is a powerful open-source intelligence (OSINT) tool designed for passive subdomain enumeration. It leverages multiple search engines and public databases to discover subdomains without direct interaction with target systems, making it ideal for authorized reconnaissance activities.

## Installation

### Kali Linux Installation

```bash
# Install from Kali repositories
sudo apt update
sudo apt install sublist3r
```

### Manual Installation

```bash
# Clone from GitHub
git clone https://github.com/aboul3la/Sublist3r.git
cd Sublist3r

# Install Python dependencies
pip install -r requirements.txt

# Verify installation
python sublist3r.py --help
```

## Basic Usage

### Simple Domain Enumeration

```bash
# Basic subdomain discovery
sublist3r -d target-domain.com
```

### Specifying Search Engines

```bash
# Use specific search engines
sublist3r -d target-domain.com -e google,bing,yahoo

# Available search engines: google, bing, yahoo, baidu, ask, netcraft, dnsdumpster, virustotal, threatcrowd
```

## Advanced Usage

### Comprehensive Enumeration

```bash
# All search engines with verbose output
sublist3r -d target-domain.com -v

# Save results to file
sublist3r -d target-domain.com -o subdomains.txt

# Enable all enumeration methods
sublist3r -d target-domain.com -b -t 10
```

### Rate Limiting and Stealth

```bash
# Control threads to avoid detection
sublist3r -d target-domain.com -t 5

# Use specific ports for verification
sublist3r -d target-domain.com -p 80,443,8080
```

## Operational Considerations

### Search Engine Limitations

- **Rate Limiting**: Search engines may block excessive requests
- **CAPTCHA Challenges**: Automated queries may trigger verification
- **IP Rotation**: Use VPN services to bypass restrictions

### VPN Integration

```bash
# Example with VPN rotation
# First location
sublist3r -d target-domain.com -e google,bing

# Switch VPN location
sublist3r -d target-domain.com -e yahoo,baidu
```

## Practical Implementation

### Basic Enumeration Script

```bash
#!/bin/bash

# Sublist3r Enumeration Script
# Usage: ./subdomain_enum.sh -d domain.com -o output_file

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

echo "[*] Starting subdomain enumeration for: $DOMAIN"
echo "[*] Timestamp: $(date)"

# Perform subdomain enumeration
if [[ -n "$OUTPUT" ]]; then
    sublist3r -d "$DOMAIN" -v -o "$OUTPUT"
else
    sublist3r -d "$DOMAIN" -v
fi

echo "[+] Enumeration complete"
```

### Multi-Domain Campaign Analysis

```bash
#!/bin/bash

# Analyze multiple related domains
DOMAINS=("primary.com" "secondary.org" "tertiary.net")
OUTPUT_DIR="subdomain_analysis_$(date +%Y%m%d)"

mkdir -p "$OUTPUT_DIR"

for domain in "${DOMAINS[@]}"; do
    echo "[*] Enumerating: $domain"
    sublist3r -d "$domain" -o "$OUTPUT_DIR/${domain}_subdomains.txt"

    # Count discovered subdomains
    count=$(grep -c "." "$OUTPUT_DIR/${domain}_subdomains.txt" 2>/dev/null || echo "0")
    echo "[+] Found $count subdomains for $domain"
done

echo "[+] Campaign analysis complete: $OUTPUT_DIR"
```

## Integration with Reconnaissance Workflow

### Comprehensive Subdomain Discovery

```bash
#!/bin/bash

# Integrated subdomain reconnaissance
DOMAIN="$1"
OUTPUT_DIR="subdomain_recon_$DOMAIN"

mkdir -p "$OUTPUT_DIR"

echo "[*] Starting comprehensive subdomain discovery for: $DOMAIN"

# Passive enumeration with Sublist3r
echo "[+] Passive enumeration with Sublist3r..."
sublist3r -d "$DOMAIN" -o "$OUTPUT_DIR/passive_subdomains.txt"

# DNS-based enumeration
echo "[+] DNS reconnaissance..."
dnsrecon -d "$DOMAIN" -t brt > "$OUTPUT_DIR/dns_subdomains.txt"

# Certificate transparency logs
echo "[+] Certificate transparency check..."
curl -s "https://crt.sh/?q=%25.$DOMAIN&output=json" | jq -r '.[].name_value' | sort -u > "$OUTPUT_DIR/cert_subdomains.txt"

# Combine and deduplicate results
cat "$OUTPUT_DIR/"*_subdomains.txt | sort -u > "$OUTPUT_DIR/all_subdomains.txt"

echo "[+] Subdomain discovery complete: $(wc -l < "$OUTPUT_DIR/all_subdomains.txt") unique subdomains found"
```

## Output Analysis and Processing

### Subdomain Categorization

```bash
#!/bin/bash

# Categorize discovered subdomains
INPUT_FILE="$1"

if [[ -z "$INPUT_FILE" ]]; then
    echo "Usage: $0 subdomains_file"
    exit 1
fi

echo "[*] Categorizing subdomains from: $INPUT_FILE"

# Common subdomain patterns
echo "[+] Common service subdomains:"
grep -E "(api|admin|mail|ftp|ssh|vpn|portal|dashboard|app|dev|staging|test)" "$INPUT_FILE"

echo "[+] Infrastructure subdomains:"
grep -E "(ns[0-9]|mail|smtp|pop|imap|ftp|cdn|waf|firewall|proxy)" "$INPUT_FILE"

echo "[+] Geographic/regional subdomains:"
grep -E "(us|uk|eu|de|fr|jp|cn|au|ca|mx|br|in)" "$INPUT_FILE"
```

### Subdomain Validation

```bash
#!/bin/bash

# Validate discovered subdomains
INPUT_FILE="$1"
OUTPUT_FILE="${INPUT_FILE%.*}_validated.txt"

if [[ -z "$INPUT_FILE" ]]; then
    echo "Usage: $0 subdomains_file"
    exit 1
fi

echo "[*] Validating subdomains from: $INPUT_FILE"

while read subdomain; do
    if host "$subdomain" &>/dev/null; then
        echo "$subdomain" >> "$OUTPUT_FILE"
        echo "[+] Valid: $subdomain"
    else
        echo "[-] Invalid: $subdomain"
    fi
done < "$INPUT_FILE"

echo "[+] Validation complete: $(wc -l < "$OUTPUT_FILE") valid subdomains"
```

## Advanced Techniques

### Search Engine Optimization

```bash
# Stagger requests to avoid detection
for engine in google bing yahoo baidu; do
    echo "[*] Using search engine: $engine"
    sublist3r -d target-domain.com -e "$engine" -t 2
    sleep 10
done
```

### Integration with Other Tools

```bash
#!/bin/bash

# Multi-tool subdomain enumeration
DOMAIN="$1"

echo "[*] Multi-source subdomain enumeration for: $DOMAIN"

# Sublist3r
sublist3r -d "$DOMAIN" -o sublist3r.txt

# Amass (if installed)
amass enum -passive -d "$DOMAIN" -o amass.txt

# AssetFinder (if installed)
assetfinder --subs-only "$DOMAIN" > assetfinder.txt

# Combine and deduplicate
cat *.txt | sort -u > "all_${DOMAIN}_subdomains.txt"

echo "[+] Found $(wc -l < "all_${DOMAIN}_subdomains.txt") unique subdomains"
```

## Operational Security

### Stealth Considerations

- **Rate Limiting**: Use `-t` option to control threads
- **Time Delays**: Add delays between search engine queries
- **IP Rotation**: Utilize VPN services for repeated queries
- **User Agents**: Consider custom user agent rotation

### Legal and Ethical Usage

```bash
#!/bin/bash

# Responsible usage checklist
DOMAIN="$1"

echo "[*] Legal and Ethical Subdomain Enumeration Checklist"
echo "[ ] 1. Verify authorization for target domain: $DOMAIN"
echo "[ ] 2. Review search engine terms of service"
echo "[ ] 3. Implement rate limiting to avoid service disruption"
echo "[ ] 4. Document authorization and scope"
echo "[ ] 5. Secure collected data appropriately"
echo ""
echo "[*] Proceed only after completing checklist"
```

## Troubleshooting Common Issues

### Search Engine Blocks

```bash
# Symptoms: No results, CAPTCHA requests, connection timeouts
# Solutions:
# 1. Reduce thread count: sublist3r -d domain.com -t 2
# 2. Use different search engines: -e bing,yahoo,baidu
# 3. Implement delays between requests
# 4. Use VPN or proxy rotation
```

### Performance Optimization

```bash
# Balance between speed and detection
# Fast but risky
sublist3r -d domain.com -t 20

# Slower but stealthier
sublist3r -d domain.com -t 5
```

## Best Practices

### Enumeration Strategy

1. **Start Passive**: Begin with OSINT tools like Sublist3r
2. **Validate Results**: Verify discovered subdomains
3. **Categorize Findings**: Group by function and importance
4. **Document Everything**: Maintain detailed records
5. **Respect Boundaries**: Stay within authorized scope

### Data Management

```bash
# Organized output structure
DOMAIN="target-domain.com"
DATE=$(date +%Y%m%d)
OUTPUT_DIR="recon_${DOMAIN}_${DATE}"

mkdir -p "$OUTPUT_DIR"/{subdomains,ports,services,screenshots}

sublist3r -d "$DOMAIN" -o "$OUTPUT_DIR/subdomains/passive.txt"
```

## Integration with Reporting

### Automated Report Generation

```bash
#!/bin/bash

# Generate subdomain enumeration report
DOMAIN="$1"
INPUT_FILE="all_${DOMAIN}_subdomains.txt"
REPORT_FILE="subdomain_report_${DOMAIN}.md"

{
echo "# Subdomain Enumeration Report: $DOMAIN"
echo "## Generated: $(date)"
echo ""
echo "## Summary"
echo "- Total Subdomains Discovered: $(wc -l < "$INPUT_FILE")"
echo "- Enumeration Methods: Sublist3r, DNS Recon, Certificate Transparency"
echo ""
echo "## Discovered Subdomains"
echo "\`\`\`"
cat "$INPUT_FILE"
echo "\`\`\`"
} > "$REPORT_FILE"

echo "[+] Report generated: $REPORT_FILE"
```

## Next Steps

After subdomain enumeration:

- **Service Discovery**: Port scanning and service identification
- **Technology Profiling**: Identify software and frameworks
- **Vulnerability Assessment**: Test for common vulnerabilities
- **Content Discovery**: Spidering and directory enumeration

---

_Sublist3r is designed for legitimate security assessment and research purposes. Always ensure proper authorization before conducting any enumeration activities and respect all applicable laws and terms of service._
