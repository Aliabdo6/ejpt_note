# Leaked Password Databases Guide

## Overview

Leaked password databases provide critical intelligence for security assessments by revealing credentials exposed in data breaches. This passive reconnaissance technique helps identify compromised employee accounts and assess organizational password security practices.

## Have I Been Pwned (HIBP)

### Primary Resource

**Website:** `https://haveibeenpwned.com`

### Key Features

- **Free Public Access**: No registration required
- **Comprehensive Database**: Aggregates data from numerous breaches
- **API Integration**: Programmatic access available
- **Regular Updates**: Continuously updated with new breaches

### Basic Usage

#### Web Interface

```bash
# Manual email checking via web browser
https://haveibeenpwned.com

# Search for specific email
https://haveibeenpwned.com/account/{email}
```

#### API Integration

```bash
# Check email via API
curl -s "https://haveibeenpwned.com/api/v3/breachedaccount/{email}" \
  -H "hibp-api-key: YOUR_API_KEY"

# Check password (k-anonymity)
curl -s "https://api.pwnedpasswords.com/range/{first5hashchars}"
```

## Integration with Reconnaissance Workflow

### Email Harvesting to Breach Checking

```bash
#!/bin/bash

# Automated breach checking for harvested emails
DOMAIN="$1"
EMAIL_FILE="harvested_emails_${DOMAIN}.txt"
BREACH_RESULTS="breach_results_${DOMAIN}.txt"

echo "[*] Checking breached emails for: $DOMAIN"

# Check each harvested email
while read email; do
    echo "[+] Checking: $email"

    # URL encode email for API call
    encoded_email=$(echo "$email" | jq -s -R -r @uri)

    # Make API request (conceptual - requires API key)
    response=$(curl -s "https://haveibeenpwned.com/api/v3/breachedaccount/$encoded_email" \
      -H "hibp-api-key: $HIBP_API_KEY")

    if [[ "$response" != "" ]]; then
        echo "BREACHED: $email" >> "$BREACH_RESULTS"
        echo "$response" | jq -r '.[].Name' >> "$BREACH_RESULTS"
    else
        echo "CLEAN: $email" >> "$BREACH_RESULTS"
    fi

    # Rate limiting
    sleep 2
done < "$EMAIL_FILE"

echo "[+] Breach analysis complete: $BREACH_RESULTS"
```

## Operational Implementation

### Ethical Breach Checking Script

```bash
#!/bin/bash

# Ethical breach checking with authorization verification
DOMAIN="$1"
AUTHORIZATION_FILE="authorization_${DOMAIN}.txt"

echo "[*] Starting ethical breach analysis for: $DOMAIN"

# Authorization verification
if [[ ! -f "$AUTHORIZATION_FILE" ]]; then
    echo "[!] ERROR: No authorization file found for $DOMAIN"
    echo "[!] Create $AUTHORIZATION_FILE with proof of authorization"
    exit 1
fi

echo "[+] Authorization verified: $(cat "$AUTHORIZATION_FILE")"

# Proceed with breach checking
./breach_checker.sh "$DOMAIN"
```

### Comprehensive Password Analysis

```bash
#!/bin/bash

# Multi-source breach intelligence
DOMAIN="$1"
OUTPUT_DIR="breach_intel_${DOMAIN}"

mkdir -p "$OUTPUT_DIR"

echo "[*] Comprehensive breach analysis for: $DOMAIN"

# 1. Check HIBP for emails
echo "[+] Checking Have I Been Pwned..."
# Implementation would require API integration

# 2. Check DeHashed (requires subscription)
echo "[+] Checking DeHashed database..."
# Commercial alternative with more detailed results

# 3. Analyze password patterns
echo "[+] Analyzing password reuse patterns..."
# Custom analysis based on discovered breaches

echo "[+] Breach analysis complete: $OUTPUT_DIR"
```

## Security Assessment Applications

### Organizational Risk Assessment

```bash
#!/bin/bash

# Employee password security assessment
COMPANY_DOMAIN="$1"
EMPLOYEE_EMAILS="employees_${COMPANY_DOMAIN}.txt"

echo "[*] Employee Password Security Assessment"
echo "[*] Target: $COMPANY_DOMAIN"

# Check breach status for discovered emails
while read email; do
    breach_status=$(check_breach "$email")  # Custom function

    case "$breach_status" in
        "CLEAN")
            echo "✅ $email - No known breaches"
            ;;
        "BREACHED")
            echo "❌ $email - Compromised in data breach"
            ;;
        "UNKNOWN")
            echo "⚠️  $email - Unable to verify"
            ;;
    esac
done < "$EMPLOYEE_EMAILS"
```

### Password Policy Compliance

```bash
#!/bin/bash

# Assess password policy effectiveness
DOMAIN="$1"
BREACHED_EMAILS="breached_${DOMAIN}.txt"

echo "[*] Password Policy Compliance Assessment"

if [[ -s "$BREACHED_EMAILS" ]]; then
    echo "[!] CRITICAL: $(wc -l < "$BREACHED_EMAILS") employees using breached passwords"
    echo "[!] Recommended Actions:"
    echo "    1. Implement mandatory password changes"
    echo "    2. Enforce password complexity requirements"
    echo "    3. Deploy multi-factor authentication"
    echo "    4. Conduct security awareness training"
else
    echo "[+] No breached passwords detected"
fi
```

## Advanced Techniques

### Password Spray Attack Preparation

```bash
#!/bin/bash

# Prepare for authorized password spraying
DOMAIN="$1"
BREACHED_CREDS="breached_credentials_${DOMAIN}.txt"

echo "[*] Preparing password spray data for: $DOMAIN"

# Extract common passwords from breach data
echo "[+] Analyzing password patterns..."
# This would involve processing breach data to identify
# commonly used passwords among organization employees

# Generate password spray list
echo "[+] Creating targeted password list..."
# Focus on passwords actually used by organization members

echo "[+] Password spray preparation complete"
echo "[!] REMINDER: Only use for authorized testing"
```

### Historical Breach Analysis

```bash
#!/bin/bash

# Analyze breach patterns over time
DOMAIN="$1"

echo "[*] Historical Breach Analysis: $DOMAIN"

# Check breach timeline
echo "[+] Breach timeline analysis..."
# Identify when employees were most affected
# Correlate with organizational changes

# Password aging assessment
echo "[+] Password reuse patterns..."
# Analyze how long compromised passwords remained in use

echo "[+] Historical analysis complete"
```

## Legal and Ethical Considerations

### Authorization Framework

```bash
#!/bin/bash

# Authorization verification script
DOMAIN="$1"

echo "[*] Legal and Ethical Compliance Check"

# Check for proper authorization
if [[ ! -f "authorization_${DOMAIN}.txt" ]]; then
    echo "[!] ERROR: Missing authorization documentation"
    echo "[!] Required: Signed testing agreement for $DOMAIN"
    exit 1
fi

# Verify scope compliance
if ! grep -q "password_breach_analysis" "scope_${DOMAIN}.txt"; then
    echo "[!] ERROR: Breach analysis not in scope"
    echo "[!] Update scope documentation before proceeding"
    exit 1
fi

echo "[+] Legal and ethical compliance verified"
```

### Responsible Disclosure

```bash
#!/bin/bash

# Breach discovery reporting
DOMAIN="$1"
BREACH_FINDINGS="breach_findings_${DOMAIN}.txt"

echo "[*] Preparing responsible disclosure report"

{
echo "DATA BREACH FINDINGS REPORT"
echo "==========================="
echo "Organization: $DOMAIN"
echo "Report Date: $(date)"
echo ""
echo "EXECUTIVE SUMMARY"
echo "-----------------"
echo "Number of compromised accounts: $(grep -c "BREACHED" "$BREACH_FINDINGS")"
echo ""
echo "RECOMMENDATIONS"
echo "---------------"
echo "1. Immediate password reset for affected accounts"
echo "2. Implement mandatory MFA"
echo "3. Security awareness training"
echo "4. Regular credential monitoring"
} > "breach_report_${DOMAIN}.md"

echo "[+] Responsible disclosure report generated"
```

## Alternative Resources

### Commercial Services

- **DeHashed**: Comprehensive database with detailed results
- **BreachDirectory**: Additional breach data sources
- **Intelx.io**: Advanced search capabilities

### Open Source Alternatives

```bash
# Local breach database tools
# Note: Maintaining local copies requires significant storage
# and may have legal implications

# Example conceptual implementation
check_local_breach_db() {
    local email="$1"
    local breach_db="/opt/breach_database/data.json"

    # Search local database
    jq -r ".[] | select(.email == \"$email\")" "$breach_db"
}
```

## Best Practices

### Operational Security

1. **API Rate Limiting**: Respect service limits
2. **Data Minimization**: Only check necessary emails
3. **Secure Storage**: Encrypt results
4. **Proper Disposal**: Securely delete after assessment

### Ethical Guidelines

```bash
#!/bin/bash

# Ethical usage checklist
echo "[*] Ethical Breach Database Usage Checklist"
echo "[ ] 1. Verify explicit authorization for target domain"
echo "[ ] 2. Confirm breach checking is within testing scope"
echo "[ ] 3. Use data only for authorized security assessment"
echo "[ ] 4. Implement proper data protection measures"
echo "[ ] 5. Follow responsible disclosure procedures"
echo "[ ] 6. Document all activities for audit purposes"
echo ""
echo "[*] Complete checklist before proceeding"
```

## Limitations and Considerations

### Data Coverage

- **Incomplete Data**: Not all breaches are included
- **Timeliness**: Some breaches may not be immediately added
- **Geographic Variations**: Coverage may vary by region

### Legal Constraints

- **Terms of Service**: Adhere to database provider terms
- **Jurisdictional Laws**: Comply with local regulations
- **Data Protection**: Follow GDPR and similar regulations

## Integration with Overall Assessment

### Comprehensive Security Posture

```bash
#!/bin/bash

# Integrate breach data with overall assessment
DOMAIN="$1"

echo "[*] Integrated Security Assessment: $DOMAIN"

# 1. Technical reconnaissance
echo "[+] Technical infrastructure mapping..."
./technical_recon.sh "$DOMAIN"

# 2. Employee intelligence
echo "[+] Employee and email enumeration..."
./email_harvesting.sh "$DOMAIN"

# 3. Breach analysis
echo "[+] Breached credential analysis..."
./breach_analysis.sh "$DOMAIN"

# 4. Risk assessment
echo "[+] Comprehensive risk scoring..."
./risk_assessment.sh "$DOMAIN"

echo "[+] Integrated assessment complete"
```

## Next Steps

After breach analysis, proceed to:

- **Active Directory Assessment**: Test identified credentials
- **Multi-Factor Authentication**: Evaluate MFA implementation
- **Security Control Testing**: Assess defensive measures
- **Social Engineering**: Develop targeted awareness training

---

_Breach database analysis is a powerful technique for authorized security assessments. Always ensure proper authorization, adhere to legal and ethical guidelines, and use discovered information responsibly to improve organizational security posture._
