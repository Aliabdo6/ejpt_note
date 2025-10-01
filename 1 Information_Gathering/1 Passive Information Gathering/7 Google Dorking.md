# Google Dorking Guide

## Overview

Google Dorking (also known as Google Hacking) is an advanced search technique that utilizes Google's search operators to find specific information, exposed files, and vulnerable systems that are publicly accessible but not easily discoverable through normal searches.

## Core Search Operators

### Domain and Site Targeting

**Limit to Specific Domain:**

```
site:target-domain.com
```

**Subdomain Enumeration:**

```
site:*.target-domain.com
```

**Specific URL Patterns:**

```
inurl:admin site:target-domain.com
inurl:forum site:target-domain.com
```

### File Type Searching

**Find Specific File Types:**

```
filetype:pdf site:target-domain.com
filetype:xlsx site:target-domain.com
filetype:doc site:target-domain.com
filetype:zip site:target-domain.com
```

**Common File Extensions:**

- pdf, doc, docx, xls, xlsx, ppt, pptx
- txt, csv, json, xml
- zip, rar, tar, gz
- sql, bak, old

### Content and Title Searching

**Search Within Page Titles:**

```
intitle:"index of" site:target-domain.com
intitle:"admin" site:target-domain.com
intitle:"login" site:target-domain.com
```

**Search Within Page Content:**

```
intext:"password" site:target-domain.com
intext:"confidential" site:target-domain.com
```

## Practical Dorking Examples

### Subdomain Discovery

```bash
# Manual subdomain enumeration
site:*.example.com

# Combine with other operators
site:*.example.com intitle:"admin"
```

### Exposed Directory Listings

```bash
# Find directory listings
intitle:"index of" site:example.com

# Directory listings with specific file types
intitle:"index of" "parent directory" filetype:pdf
```

### Configuration File Discovery

```bash
# WordPress configuration backups
inurl:"wp-config.php" filetype:php
intitle:"index of" "wp-config.php.bak"

# General configuration files
filetype:env site:example.com
filetype:config site:example.com
```

### Credential and Sensitive Data Hunting

```bash
# Password files
inurl:"password.txt"
filetype:txt "password"

# Authentication files
inurl:"auth_user_file.txt"
filetype:htpasswd
```

### Technology-Specific Dorks

```bash
# WordPress specific
site:example.com inurl:wp-admin
site:example.com inurl:wp-content

# Database dumps
filetype:sql "INSERT INTO" site:example.com
```

## Advanced Dorking Techniques

### Combined Operators

```bash
# Multiple conditions
site:gov.* intitle:"index of" filetype:csv
site:example.com (inurl:admin | inurl:login) filetype:php
```

### Exclusion Operators

```bash
# Exclude specific terms
site:example.com -inurl:blog
site:example.com filetype:pdf -inurl:public
```

### Date Range Queries

```bash
# Historical data (via cache)
cache:example.com

# Note: Traditional date range operators have been deprecated
# Use Wayback Machine for historical analysis
```

## Integration with Reconnaissance

### Automated Dorking Script

```bash
#!/bin/bash

# Google Dorking Automation Script
DOMAIN="$1"
OUTPUT_FILE="dorking_results_${DOMAIN}.txt"

echo "[*] Starting Google Dorking for: $DOMAIN"
echo "[*] Results will be saved to: $OUTPUT_FILE"

# Common dorks to test
DORKS=(
    "site:${DOMAIN}"
    "site:*.${DOMAIN}"
    "site:${DOMAIN} filetype:pdf"
    "site:${DOMAIN} filetype:doc"
    "site:${DOMAIN} filetype:xlsx"
    "site:${DOMAIN} inurl:admin"
    "site:${DOMAIN} inurl:login"
    "site:${DOMAIN} intitle:\"index of\""
    "site:${DOMAIN} intext:\"password\""
    "site:${DOMAIN} intext:\"confidential\""
)

for dork in "${DORKS[@]}"; do
    echo "[+] Testing: $dork" | tee -a "$OUTPUT_FILE"
    # Note: Actual Google queries would require API integration
    # This is a conceptual implementation
    echo "   Manual Query: https://www.google.com/search?q=${dork// /+}" | tee -a "$OUTPUT_FILE"
    echo "---" | tee -a "$OUTPUT_FILE"
done

echo "[+] Dorking complete. Review $OUTPUT_FILE for manual queries"
```

### Domain-Focused Reconnaissance

```bash
#!/bin/bash

# Comprehensive domain dorking
DOMAIN="$1"

echo "[*] Comprehensive Google Dorking for: $DOMAIN"

echo "[+] Subdomain Discovery"
echo "    site:*.${DOMAIN}"

echo "[+] File Discovery"
echo "    site:${DOMAIN} filetype:pdf"
echo "    site:${DOMAIN} filetype:doc"
echo "    site:${DOMAIN} filetype:xlsx"
echo "    site:${DOMAIN} filetype:sql"

echo "[+] Administrative Interfaces"
echo "    site:${DOMAIN} inurl:admin"
echo "    site:${DOMAIN} inurl:login"
echo "    site:${DOMAIN} intitle:\"admin\""

echo "[+] Directory Listings"
echo "    site:${DOMAIN} intitle:\"index of\""

echo "[+] Sensitive Information"
echo "    site:${DOMAIN} intext:\"password\""
echo "    site:${DOMAIN} intext:\"confidential\""
```

## Historical Analysis with Wayback Machine

### Accessing Historical Data

```bash
# Wayback Machine URL format
https://web.archive.org/web/*/https://example.com

# Specific timestamp
https://web.archive.org/web/20160101000000/https://example.com
```

### Historical Analysis Script

```bash
#!/bin/bash

# Wayback Machine Analysis
DOMAIN="$1"
YEAR="${2:-2015}"

echo "[*] Analyzing historical data for: $DOMAIN"
echo "[*] Focus year: $YEAR"

WAYBACK_URL="https://web.archive.org/web/${YEAR}0101000000/https://${DOMAIN}"

echo "[+] Wayback URL: $WAYBACK_URL"
echo "[+] Open this URL in browser to view historical snapshots"

# Extract available snapshots (conceptual)
echo "[+] Available snapshots can be viewed at:"
echo "    https://web.archive.org/web/*/https://${DOMAIN}"
```

## Google Hacking Database (GHDB)

### About GHDB

The Google Hacking Database, maintained by Offensive Security, contains curated dorks for finding vulnerable systems and sensitive information.

### Common GHDB Categories

- **Juicy Information**: Sensitive documents and data
- **File Containers**: Exposed directories and files
- **Vulnerable Files**: Configuration backups and exposed data
- **Web Server Detection**: Server version disclosures
- **Advisories and Vulnerabilities**: Known vulnerability patterns

### Using GHDB Effectively

```bash
# Conceptual GHDB integration
# Visit: https://www.exploit-db.com/google-hacking-database

# Example dorks from GHDB:
# Government sites with directory listings
site:gov.* intitle:"index of" filetype:csv

# WordPress config backups
intitle:"index of" "wp-config.php.bak"

# Exposed password files
inurl:"password.txt" filetype:txt
```

## Operational Security

### Rate Limiting Considerations

- **Manual Queries**: Space out searches to avoid CAPTCHAs
- **Automated Tools**: Use official APIs when available
- **VPN Rotation**: Change IP addresses if blocked

### Legal and Ethical Usage

```bash
#!/bin/bash

# Ethical dorking checklist
echo "[*] Ethical Google Dorking Checklist"
echo "[ ] 1. Verify authorization for target domain"
echo "[ ] 2. Review Google Terms of Service"
echo "[ ] 3. Document findings for authorized purposes only"
echo "[ ] 4. Report discovered vulnerabilities responsibly"
echo "[ ] 5. Do not access or download unauthorized data"
echo ""
echo "[*] Proceed only after completing checklist"
```

## Advanced Techniques

### Dork Chaining

```bash
# Combine multiple dorks for precision
site:example.com (inurl:admin | inurl:login) (filetype:php | filetype:html)
```

### Negative Filtering

```bash
# Exclude common false positives
site:example.com intitle:"index of" -inurl:wp-content -inurl:public
```

### Geographic Targeting

```bash
# Note: Traditional geographic operators are deprecated
# Use location-specific sites or ccTLDs
site:example.co.uk
site:example.de
```

## Integration with Other Tools

### Dorking in Reconnaissance Workflow

```bash
#!/bin/bash

# Integrated reconnaissance with dorking
DOMAIN="$1"

echo "[*] Starting comprehensive reconnaissance for: $DOMAIN"

# Passive subdomain discovery
echo "[+] Passive subdomain enumeration..."
sublist3r -d "$DOMAIN" -o subdomains.txt

# Google dorking for content discovery
echo "[+] Google dorking for sensitive content..."
echo "    Manual queries required for:"
echo "    - site:${DOMAIN} filetype:pdf"
echo "    - site:${DOMAIN} inurl:admin"
echo "    - site:${DOMAIN} intitle:\"index of\""

# Historical analysis
echo "[+] Historical analysis via Wayback Machine..."
echo "    https://web.archive.org/web/*/https://${DOMAIN}"

echo "[+] Reconnaissance phase complete"
```

## Best Practices

### Effective Dork Construction

1. **Start Broad**: Begin with basic site operators
2. **Progressively Refine**: Add filters to narrow results
3. **Use Multiple Variations**: Try different operator combinations
4. **Document Successful Dorks**: Build a personal database
5. **Validate Findings**: Verify discovered information

### Operational Security

- **Legal Compliance**: Only search authorized targets
- **Terms of Service**: Adhere to search engine policies
- **Data Handling**: Securely store and handle discovered information
- **Responsible Disclosure**: Report vulnerabilities appropriately

## Limitations and Considerations

### Search Engine Limitations

- **Rate Limiting**: Excessive queries may trigger blocks
- **Index Coverage**: Not all content is indexed by search engines
- **Dynamic Content**: Modern web applications may not be fully indexed
- **API Changes**: Search operators may change over time

### Ethical Boundaries

- **Authorization**: Only conduct dorking on authorized targets
- **Data Access**: Do not access or download sensitive information
- **Scope Compliance**: Stay within authorized testing boundaries
- **Professional Conduct**: Maintain professional ethics throughout

## Next Steps

After Google dorking, proceed to:

- **Email Harvesting**: Collect contact information
- **Social Media Intelligence**: Gather personnel information
- **Technical Validation**: Verify discovered services
- **Vulnerability Assessment**: Test identified systems

---

_Google dorking is a powerful technique for authorized security assessments. Always ensure proper authorization, adhere to search engine terms of service, and maintain ethical standards throughout your reconnaissance activities._
