# Security Auditing

## Overview of Security Auditing

Security auditing is a systematic process of evaluating and verifying the security measures and controls within an organization to ensure they are effective, appropriate, and compliant with relevant standards, policies, and regulations.

### What is Security Auditing?

Security auditing involves reviewing various aspects of an organization's information systems, networks, applications, and operational procedures to identify vulnerabilities, weaknesses, and areas for improvement. It assesses whether an organization is compliant with its own established standards and policies.

## Importance of Security Auditing

### 1. Identifying Vulnerabilities and Weaknesses

Security audits help uncover vulnerabilities in an organization's information systems and infrastructure that could be exploited by attackers. Regular audits ensure security controls remain effective and up-to-date, minimizing breach risks.

**Key Differentiator**: Unlike penetration testing that focuses on exploiting technical vulnerabilities, security auditing starts with assessing the foundational security layers - organizational processes, employee adherence to policies, and operational procedures.

### 2. Ensuring Compliance

Organizations must comply with various regulatory requirements and industry standards to protect sensitive data and maintain trust. Security audits verify compliance with standards such as:

- **GDPR** (General Data Protection Regulation) - For organizations handling EU citizen data
- **HIPAA** (Health Insurance Portability and Accountability Act) - For healthcare organizations
- **PCI DSS** (Payment Card Industry Data Security Standard) - For financial transaction processing
- **ISO 27001** - General information security management

Non-compliance can result in serious legal and financial penalties, especially following data breaches.

### 3. Enhancing Risk Management

Audits provide comprehensive assessments of an organization's security posture, allowing companies to:

- Identify and quantify vulnerabilities
- Prioritize risks based on potential impact
- Develop effective risk management strategies
- Focus resources on high-impact risks

### 4. Improving Security Policies and Procedures

Regular audits help organizations:

- Ensure adherence to existing security policies
- Identify areas for policy improvement
- Maintain updated and robust security procedures
- Foster a strong security culture within the organization

### 5. Supporting Business Objectives

A strong security posture supports business objectives by:

- Protecting critical operations from security disruptions
- Building customer trust and confidence
- Demonstrating responsible data handling practices
- Enhancing organizational reputation

### 6. Enabling Continuous Improvement

Security auditing is an ongoing process that promotes continuous improvement by:

- Ensuring security measures evolve to address new threats
- Maintaining a proactive security approach
- Keeping policies updated with current threat landscapes

## Practical Security Auditing Tools and Techniques

### Basic Security Audit Script

```bash
#!/bin/bash

# Basic security audit tool
# Usage: ./security_audit.sh [target] [options]

TARGET=""
OUTPUT_FILE=""
SCAN_TYPE="basic"

usage() {
    echo "Usage: $0 -t <target> [-o output_file] [-s scan_type]"
    echo "  -t  Target IP or hostname"
    echo "  -o  Output file (default: audit_<target>_<date>.txt)"
    echo "  -s  Scan type: basic|comprehensive (default: basic)"
    exit 1
}

while getopts "t:o:s:" opt; do
    case $opt in
        t) TARGET="$OPTARG" ;;
        o) OUTPUT_FILE="$OPTARG" ;;
        s) SCAN_TYPE="$OPTARG" ;;
        *) usage ;;
    esac
done

if [[ -z "$TARGET" ]]; then
    usage
fi

if [[ -z "$OUTPUT_FILE" ]]; then
    OUTPUT_FILE="audit_${TARGET}_$(date +%Y%m%d_%H%M%S).txt"
fi

echo "Starting security audit for: $TARGET" | tee "$OUTPUT_FILE"
echo "Scan type: $SCAN_TYPE" | tee -a "$OUTPUT_FILE"
echo "Date: $(date)" | tee -a "$OUTPUT_FILE"
echo "==========================================" | tee -a "$OUTPUT_FILE"

# Basic network reconnaissance
echo "[*] Running basic network scan..." | tee -a "$OUTPUT_FILE"
nmap -sS -T4 "$TARGET" 2>/dev/null | tee -a "$OUTPUT_FILE"

if [[ "$SCAN_TYPE" == "comprehensive" ]]; then
    echo "[*] Running comprehensive vulnerability scan..." | tee -a "$OUTPUT_FILE"
    nmap -sV -sC --script vuln "$TARGET" 2>/dev/null | tee -a "$OUTPUT_FILE"

    echo "[*] Checking common web vulnerabilities..." | tee -a "$OUTPUT_FILE"
    nikto -h "$TARGET" 2>/dev/null | tee -a "$OUTPUT_FILE"
fi

echo "[*] Audit completed. Results saved to: $OUTPUT_FILE" | tee -a "$OUTPUT_FILE"
```

### Terminal Usage Instructions

#### Basic Usage

```bash
# Make script executable
chmod +x security_audit.sh

# Run basic audit against target
./security_audit.sh -t 192.168.1.100

# Run with custom output file
./security_audit.sh -t example.com -o my_audit_results.txt

# Run comprehensive audit
./security_audit.sh -t 10.0.1.50 -s comprehensive -o detailed_audit.txt
```

#### Advanced Usage Example

```bash
#!/bin/bash
# Advanced audit automation script

TARGETS_FILE="targets.txt"
AUDIT_DIR="audit_results_$(date +%Y%m%d)"

mkdir -p "$AUDIT_DIR"

while IFS= read -r target; do
    if [[ -n "$target" ]]; then
        echo "Auditing target: $target"
        ./security_audit.sh -t "$target" -o "$AUDIT_DIR/audit_${target//\//_}.txt" -s comprehensive
    fi
done < "$TARGETS_FILE"

# Generate summary report
echo "Generating summary report..."
find "$AUDIT_DIR" -name "*.txt" -exec grep -l "open" {} \; > "$AUDIT_DIR/vulnerable_hosts.txt"
```

### Practical Usage Example

**Scenario**: Auditing a web application for common security issues

```bash
# Create targets file
echo "webapp.example.com" > targets.txt
echo "api.example.com" >> targets.txt

# Run automated audit
./advanced_audit.sh

# Check results
ls -la audit_results_20231201/
cat audit_results_20231201/vulnerable_hosts.txt
```

**Expected Output**:

```
Starting security audit for: webapp.example.com
Scan type: comprehensive
Date: Fri Dec 1 14:30:45 UTC 2023
==========================================
[*] Running basic network scan...
Nmap scan report for webapp.example.com (192.168.1.100)
Host is up (0.045s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https

[*] Running comprehensive vulnerability scan...
[*] Checking common web vulnerabilities...
- Nikto v2.1.6
+ Target IP:          192.168.1.100
+ Target Hostname:    webapp.example.com
+ Target Port:        80
+ Start Time:         2023-12-01 14:30:46
...
```

## Hands-on Lab Exercise

### PortSwigger Web Security Academy Lab

**Lab**: [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data)

**Lab Setup**:

1. Access the lab URL (free account required)
2. No additional credentials needed - lab provides temporary instances
3. Lab resets after completion or timeout

**Audit Objective**: Identify and demonstrate SQL injection vulnerabilities

**Audit Commands**:

```bash
# Audit the lab application
./security_audit.sh -t [LAB_URL] -s comprehensive -o sql_injection_audit.txt

# Manual testing with curl for SQL injection
curl "https://[LAB_URL]/filter?category=Gifts' OR 1=1--"
```

## Key Takeaways

- Security auditing is essential for maintaining organizational security posture
- Regular audits help identify compliance gaps and policy violations
- Automated tools combined with manual testing provide comprehensive coverage
- Continuous auditing enables proactive security improvement
- Proper documentation and reporting are crucial for audit effectiveness

## Next Steps

- Implement regular automated security scans
- Develop organization-specific audit checklists
- Establish remediation workflows for identified issues
- Schedule periodic comprehensive security assessments
- Maintain audit trails for compliance and improvement tracking
