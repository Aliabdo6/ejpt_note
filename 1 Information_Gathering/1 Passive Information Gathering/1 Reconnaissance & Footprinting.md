# Passive Reconnaissance & Footprinting Guide

## Overview

This guide covers passive reconnaissance and footprinting techniques for gathering information about target websites without direct interaction. These methods help build a comprehensive understanding of a target's digital footprint while maintaining stealth.

## Core Concepts

### What is Footprinting?

Footprinting is the process of collecting publicly available information about a target to identify critical data points that could be relevant for security assessment or penetration testing.

### Key Information Targets

- IP addresses and server infrastructure
- Hidden directories and files
- Contact information (names, emails, phone numbers)
- Physical addresses
- Web technologies and frameworks
- Content management systems

## Tools & Techniques

### DNS Resolution with `host`

The `host` command performs DNS lookups to resolve domain names to IP addresses:

```bash
host target-domain.com
```

**Example Output:**

```
target-domain.com has address 192.0.2.1
target-domain.com has address 192.0.2.2
target-domain.com has IPv6 address 2001:db8::1
target-domain.com mail is handled by 10 mail.target-domain.com.
```

**Note:** Multiple IP addresses often indicate the site is behind a proxy service like Cloudflare.

### Website Structure Analysis

#### robots.txt

This file tells search engines which directories should not be indexed:

```bash
curl https://target-domain.com/robots.txt
```

**Example Analysis:**

```
User-agent: *
Disallow: /wp-admin/
Disallow: /private/
```

This reveals WordPress usage and restricted directories.

#### sitemap.xml

Provides search engines with an organized structure of the website:

```bash
curl https://target-domain.com/sitemap.xml
```

Sitemaps can reveal:

- Post structures
- Page hierarchies
- Author information
- Hidden categories

### Web Technology Profiling

#### Browser Extensions

- **BuiltWith**: Identifies web technologies, frameworks, and plugins
- **Wappalyzer**: Detects content management systems and web technologies

#### Command Line Tools

**WhatWeb** (pre-installed in Kali):

```bash
whatweb target-domain.com
```

**Example Output:**

```
https://target-domain.com [301 Moved Permanently]
Country[UNITED STATES][US], HTTPServer[cloudflare],
IP[192.0.2.1], jQuery, Modernizr, PHP[7.4.29],
WordPress[5.8], X-Powered-By[PHP/7.4.29]
```

### Website Mirroring with HTTrack

Download entire websites for offline analysis:

```bash
# Install HTTrack
sudo apt-get install webhttrack

# Launch web interface
webhttrack
```

**Usage:**

1. Access `http://localhost:8080`
2. Set project name and base path
3. Add target URL
4. Start mirroring process

## Advanced Usage Script

Here's a comprehensive reconnaissance script that automates these techniques:

```bash
#!/bin/bash

# Passive Reconnaissance Script
# Usage: ./recon.sh -d domain.com -o output_dir

while getopts "d:o:" opt; do
  case $opt in
    d) DOMAIN="$OPTARG" ;;
    o) OUTPUT="$OPTARG" ;;
    *) echo "Usage: $0 -d domain -o output_dir" >&2
       exit 1 ;;
  esac
done

if [[ -z "$DOMAIN" || -z "$OUTPUT" ]]; then
    echo "Domain and output directory required"
    exit 1
fi

mkdir -p "$OUTPUT"

echo "[*] Starting passive reconnaissance for: $DOMAIN"
echo "[*] Output directory: $OUTPUT"

# DNS Resolution
echo "[+] Performing DNS lookup..."
host "$DOMAIN" > "$OUTPUT/dns_info.txt"

# robots.txt analysis
echo "[+] Checking robots.txt..."
curl -s "https://$DOMAIN/robots.txt" -o "$OUTPUT/robots.txt"

# sitemap analysis
echo "[+] Checking sitemap.xml..."
curl -s "https://$DOMAIN/sitemap.xml" -o "$OUTPUT/sitemap.xml"

# Technology profiling
echo "[+] Profiling web technologies..."
whatweb "$DOMAIN" > "$OUTPUT/technology.txt"

echo "[+] Reconnaissance complete. Results saved to: $OUTPUT"
```

## Practical Examples

### Basic Domain Reconnaissance

```bash
# Create output directory
mkdir recon_results

# Run comprehensive scan
./recon.sh -d example.com -o recon_results

# Review results
cat recon_results/dns_info.txt
cat recon_results/technology.txt
```

### Advanced Multi-Domain Analysis

```bash
#!/bin/bash

# Batch domain analysis
DOMAINS=("domain1.com" "domain2.org" "domain3.net")

for domain in "${DOMAINS[@]}"; do
    echo "Analyzing: $domain"
    mkdir -p "output_$domain"
    ./recon.sh -d "$domain" -o "output_$domain"
done
```

## Key Findings Interpretation

### Common Indicators

**WordPress Detection:**

- `/wp-admin/` in robots.txt
- WordPress-specific plugins in technology scan
- Characteristic file structure

**Proxy Services:**

- Multiple IP addresses in DNS resolution
- Cloudflare or similar headers in technology scan

**Information Disclosure:**

- Author names in sitemaps
- Email patterns in downloaded content
- Administrative paths in robots.txt

## Best Practices

1. **Legal Compliance**: Only perform reconnaissance on domains you own or have explicit permission to test
2. **Rate Limiting**: Add delays between requests to avoid triggering security measures
3. **Data Organization**: Maintain clear documentation of findings
4. **Stealth Operations**: Use passive methods that don't interact directly with target systems

## Next Steps

After completing passive reconnaissance, consider:

- WHOIS information enumeration
- Subdomain discovery
- Social media profiling
- Email pattern analysis

This foundation of passive information gathering provides crucial insights for more targeted security assessments while maintaining operational stealth.

---
