# Types of Security Audits

## Overview

Security audits can be categorized based on scope, methodology, and organizational focus. Understanding these different types helps penetration testers tailor their testing strategies and provides context for how audit findings inform security testing.

## Primary Audit Categories

### 1. Internal Audits

**Objective**: Evaluate effectiveness of internal security controls and compliance with internal policies.

**Conducted By**: Organization's internal audit team or security professionals

**Importance**:

- Provides insight into organization's self-assessment capabilities
- Highlights areas requiring more in-depth testing
- Identifies policy compliance gaps
- Supports continuous internal improvement

**Example**:

- Reviewing user access controls to ensure only authorized personnel access sensitive data
- Verifying compliance with internal password policies
- Assessing employee adherence to security procedures

**Implementation Script**:

```bash
#!/bin/bash

# internal_audit.sh
# Usage: ./internal_audit.sh -d <domain> -p <policy_file>

check_internal_compliance() {
    local domain=$1
    local policy_file=$2
    local output_file="internal_audit_$(date +%Y%m%d).txt"

    echo "Internal Security Audit Report" > "$output_file"
    echo "=============================" >> "$output_file"
    echo "Domain: $domain" >> "$output_file"
    echo "Policy Reference: $policy_file" >> "$output_file"
    echo "Date: $(date)" >> "$output_file"
    echo "" >> "$output_file"

    # Check password policy compliance
    echo "Password Policy Compliance Check:" >> "$output_file"
    check_password_compliance >> "$output_file"

    # Check access controls
    echo "Access Control Review:" >> "$output_file"
    check_access_controls >> "$output_file"

    # Check system configurations
    echo "System Configuration Audit:" >> "$output_file"
    check_system_configs >> "$output_file"

    echo "Internal audit completed: $output_file"
}

check_password_compliance() {
    # Check local password policies
    echo "- Password expiration: $(chage -l root 2>/dev/null | grep "Maximum" | cut -d: -f2)"
    echo "- Minimum password length: $(grep -i "minlen" /etc/security/pwquality.conf 2>/dev/null || echo "Not configured")"
    echo "- Password complexity: $(grep -i "minclass" /etc/security/pwquality.conf 2>/dev/null || echo "Not configured")"
}

check_access_controls() {
    # Check file permissions
    echo "- Critical file permissions:"
    ls -l /etc/passwd /etc/shadow /etc/sudoers 2>/dev/null | awk '{print $1 " " $9}'

    # Check sudo access
    echo "- Users with sudo access:"
    getent group sudo | cut -d: -f4
}

main() {
    local domain=""
    local policy_file=""

    while getopts "d:p:" opt; do
        case $opt in
            d) domain="$OPTARG" ;;
            p) policy_file="$OPTARG" ;;
        esac
    done

    if [[ -z "$domain" ]]; then
        echo "Usage: $0 -d <domain> -p <policy_file>"
        exit 1
    fi

    check_internal_compliance "$domain" "$policy_file"
}

main "$@"
```

### 2. External Audits

**Objective**: Provide unbiased evaluation of security measures and compliance with external standards.

**Conducted By**: Independent third-party auditors

**Importance**:

- Delivers unbiased security assessment
- Serves as compliance benchmark
- Provides objective security effectiveness measurement
- Findings guide penetration testing efforts

**Example**:

- PCI DSS compliance audit for payment processing systems
- ISO 27001 certification audit
- SOC 2 Type II examination

**External Audit Preparation**:

```bash
#!/bin/bash

# external_audit_prep.sh
# Usage: ./external_audit_prep.sh -s <standard> -o <output_dir>

prepare_for_external_audit() {
    local standard=$1
    local output_dir=$2

    echo "Preparing for $standard External Audit"
    echo "======================================"

    # Gather evidence based on standard
    case $standard in
        pci)
            gather_pci_evidence "$output_dir"
            ;;
        iso27001)
            gather_iso_evidence "$output_dir"
            ;;
        soc2)
            gather_soc2_evidence "$output_dir"
            ;;
        *)
            echo "Unsupported standard: $standard"
            return 1
            ;;
    esac

    echo "Audit preparation completed in: $output_dir"
}

gather_pci_evidence() {
    local dir=$1/pci_evidence

    mkdir -p "$dir"

    echo "Gathering PCI DSS Evidence:"
    echo "- Network diagrams and data flows"
    echo "- Firewall and router configurations"
    echo "- Access control lists"
    echo "- Encryption implementation details"
    echo "- Security policy documentation"

    # Example evidence collection
    iptables-save > "$dir/iptables_rules.txt" 2>/dev/null
    find /etc/ -name "*.conf" -exec cp {} "$dir/" \; 2>/dev/null
}

main() {
    local standard=""
    local output_dir="audit_prep_$(date +%Y%m%d)"

    while getopts "s:o:" opt; do
        case $opt in
            s) standard="$OPTARG" ;;
            o) output_dir="$OPTARG" ;;
        esac
    done

    if [[ -z "$standard" ]]; then
        echo "Usage: $0 -s <standard> [-o output_dir]"
        echo "Supported standards: pci, iso27001, soc2"
        exit 1
    fi

    prepare_for_external_audit "$standard" "$output_dir"
}

main "$@"
```

### 3. Compliance Audits

**Objective**: Verify adherence to specific regulatory requirements and industry standards.

**Focus Areas**:

- Regulatory requirements (GDPR, HIPAA, PCI DSS)
- Industry standards (ISO 27001, NIST frameworks)
- Legal obligations and certifications

**Importance**:

- Identifies regulatory compliance gaps
- Helps avoid legal and financial penalties
- Provides framework for targeted security testing
- Demonstrates due diligence to stakeholders

**Examples**:

- **HIPAA**: Healthcare data protection compliance
- **GDPR**: EU data privacy regulation adherence
- **PCI DSS**: Payment card industry security standards

**Compliance Check Script**:

```bash
#!/bin/bash

# compliance_checker.sh
# Usage: ./compliance_checker.sh -s <standard> -t <target>

check_compliance() {
    local standard=$1
    local target=$2
    local report_file="compliance_${standard}_$(date +%Y%m%d).txt"

    echo "Compliance Audit: $standard" > "$report_file"
    echo "Target: $target" >> "$report_file"
    echo "Date: $(date)" >> "$report_file"
    echo "===============================" >> "$report_file"

    case $standard in
        gdpr)
            check_gdpr_compliance "$target" >> "$report_file"
            ;;
        hipaa)
            check_hipaa_compliance "$target" >> "$report_file"
            ;;
        pci)
            check_pci_compliance "$target" >> "$report_file"
            ;;
        *)
            echo "Unsupported standard: $standard" >> "$report_file"
            ;;
    esac

    echo "Compliance report generated: $report_file"
}

check_gdpr_compliance() {
    local target=$1
    echo "GDPR Compliance Checks:"
    echo "- Data encryption in transit: $(check_encryption "$target")"
    echo "- Data retention policies: $(check_retention_policies)"
    echo "- User consent mechanisms: $(check_consent_mechanisms)"
    echo "- Data access controls: $(check_access_controls)"
    echo "- Breach notification procedures: $(check_breach_procedures)"
}

check_encryption() {
    local target=$1
    # Check if HTTPS is enforced
    curl -s -I "https://$target" | grep -i "strict-transport-security" && echo "HSTS Enabled" || echo "HSTS Missing"
}

main() {
    local standard=""
    local target=""

    while getopts "s:t:" opt; do
        case $opt in
            s) standard="$OPTARG" ;;
            t) target="$OPTARG" ;;
        esac
    done

    if [[ -z "$standard" || -z "$target" ]]; then
        echo "Usage: $0 -s <standard> -t <target>"
        echo "Supported standards: gdpr, hipaa, pci"
        exit 1
    fi

    check_compliance "$standard" "$target"
}

main "$@"
```

## Technical Audit Specializations

### 4. Technical Audits

**Objective**: Assess technical aspects of IT infrastructure including hardware, software, and network configurations.

**Focus Areas**:

- System configurations and hardening
- Security control implementations
- Technical policy enforcement
- Infrastructure security posture

**Importance**:

- Provides detailed view of technical controls
- Identifies configuration weaknesses
- Highlights areas for penetration testing
- Supports technical risk assessment

**Example**:

- Reviewing firewall configurations for proper rule sets
- Auditing server hardening against CIS benchmarks
- Verifying encryption implementations

**Technical Audit Script**:

```bash
#!/bin/bash

# technical_audit.sh
# Usage: ./technical_audit.sh -t <target> -c <check_type>

perform_technical_audit() {
    local target=$1
    local check_type=$2
    local output_file="technical_audit_${target}_$(date +%Y%m%d).txt"

    echo "Technical Security Audit" > "$output_file"
    echo "=======================" >> "$output_file"
    echo "Target: $target" >> "$output_file"
    echo "Check Type: $check_type" >> "$output_file"
    echo "Date: $(date)" >> "$output_file"
    echo "" >> "$output_file"

    case $check_type in
        system)
            perform_system_audit "$target" >> "$output_file"
            ;;
        network)
            perform_network_audit "$target" >> "$output_file"
            ;;
        application)
            perform_application_audit "$target" >> "$output_file"
            ;;
        full)
            perform_full_audit "$target" >> "$output_file"
            ;;
    esac

    echo "Technical audit completed: $output_file"
}

perform_system_audit() {
    local target=$1
    echo "System Security Audit:"
    echo "- OS version and patch level"
    echo "- Running services and ports"
    echo "- User accounts and privileges"
    echo "- File permissions and ownership"
    echo "- Logging and monitoring configuration"

    # Example system checks
    ssh "$target" "uname -a; whoami; ps aux | head -10" 2>/dev/null
}

perform_network_audit() {
    local target=$1
    echo "Network Security Audit:"
    echo "- Open ports and services: $(nmap -sS "$target" | grep open)"
    echo "- Firewall configuration review"
    echo "- Network segmentation verification"
    echo "- Wireless security assessment"
}

main() {
    local target=""
    local check_type="full"

    while getopts "t:c:" opt; do
        case $opt in
            t) target="$OPTARG" ;;
            c) check_type="$OPTARG" ;;
        esac
    done

    if [[ -z "$target" ]]; then
        echo "Usage: $0 -t <target> [-c check_type]"
        echo "Check types: system, network, application, full"
        exit 1
    fi

    perform_technical_audit "$target" "$check_type"
}

main "$@"
```

### 5. Network Audits

**Objective**: Assess security of network infrastructure including routers, switches, firewalls, and network devices.

**Focus Areas**:

- Network architecture and segmentation
- Device configurations and hardening
- Traffic flow and access controls
- Network monitoring and logging

**Importance**:

- Reveals network design vulnerabilities
- Identifies misconfigurations in network devices
- Provides basis for network penetration testing
- Supports network security optimization

**Example**:

- Identifying insecure protocols (Telnet, SNMP v1/v2)
- Reviewing firewall rule bases for excessive permissions
- Assessing network segmentation effectiveness

**Network Audit Script**:

```bash
#!/bin/bash

# network_audit.sh
# Usage: ./network_audit.sh -n <network_cidr> -o <output_dir>

audit_network() {
    local network=$1
    local output_dir=$2

    echo "Network Security Audit: $network"
    echo "==============================="

    # Network discovery
    echo "[1/4] Network Discovery"
    nmap -sn "$network" > "$output_dir/network_discovery.txt"

    # Port scanning
    echo "[2/4] Port Scanning"
    nmap -sS -sV -O "$network" > "$output_dir/port_scan.txt"

    # Vulnerability assessment
    echo "[3/4] Vulnerability Scanning"
    nmap --script vuln "$network" > "$output_dir/vulnerability_scan.txt"

    # Network device assessment
    echo "[4/4] Network Device Analysis"
    analyze_network_devices "$network" "$output_dir"

    echo "Network audit completed in: $output_dir"
}

analyze_network_devices() {
    local network=$1
    local output_dir=$2/network_devices

    mkdir -p "$output_dir"

    # Check for common network services
    echo "Checking network services:"
    nmap -sS -p 21,22,23,80,443,161,162,443,3389 "$network" | grep open > "$output_dir/network_services.txt"

    # SNMP assessment (if found)
    if grep -q "161/open" "$output_dir/network_services.txt"; then
        echo "SNMP found - checking for public community string"
        onesixtyone "$network" > "$output_dir/snmp_check.txt" 2>/dev/null
    fi
}

main() {
    local network=""
    local output_dir="network_audit_$(date +%Y%m%d)"

    while getopts "n:o:" opt; do
        case $opt in
            n) network="$OPTARG" ;;
            o) output_dir="$OPTARG" ;;
        esac
    done

    if [[ -z "$network" ]]; then
        echo "Usage: $0 -n <network_cidr> [-o output_dir]"
        echo "Example: $0 -n 192.168.1.0/24"
        exit 1
    fi

    mkdir -p "$output_dir"
    aud it_network "$network" "$output_dir"
}

main "$@"
```

### 6. Application Audits

**Objective**: Evaluate security of software applications focusing on code quality, input validation, authentication, and data handling.

**Focus Areas**:

- Code review and static analysis
- Authentication and authorization mechanisms
- Input validation and sanitization
- Data protection and encryption
- Session management

**Importance**:

- Identifies application-level vulnerabilities
- Supports secure development practices
- Provides basis for application penetration testing
- Helps prevent common web vulnerabilities (OWASP Top 10)

**Example**:

- Identifying SQL injection vulnerabilities
- Testing for cross-site scripting (XSS) flaws
- Assessing authentication bypass possibilities
- Reviewing data encryption implementations

**Application Audit Script**:

```bash
#!/bin/bash

# application_audit.sh
# Usage: ./application_audit.sh -u <url> -t <app_type>

audit_application() {
    local url=$1
    local app_type=$2
    local output_dir="app_audit_$(echo "$url" | sed 's|https?://||' | tr '/' '_')"

    mkdir -p "$output_dir"

    echo "Application Security Audit: $url"
    echo "================================"

    # Basic reconnaissance
    echo "[1/5] Application Reconnaissance"
    whatweb "$url" > "$output_dir/recon.txt"

    # Vulnerability scanning
    echo "[2/5] Vulnerability Scanning"
    nikto -h "$url" > "$output_dir/nikto_scan.txt" 2>/dev/null

    # Technology stack analysis
    echo "[3/5] Technology Analysis"
    wappalyzer "$url" > "$output_dir/tech_stack.txt" 2>/dev/null

    # Security headers check
    echo "[4/5] Security Headers Check"
    check_headers "$url" > "$output_dir/security_headers.txt"

    # OWASP Top 10 assessment
    echo "[5/5] OWASP Top 10 Assessment"
    perform_owasp_checks "$url" "$output_dir"

    echo "Application audit completed in: $output_dir"
}

check_headers() {
    local url=$1
    curl -s -I "$url" | grep -E "(Strict-Transport-Security|X-Frame-Options|X-Content-Type-Options|Content-Security-Policy)"
}

perform_owasp_checks() {
    local url=$1
    local output_dir=$2/owasp_checks

    mkdir -p "$output_dir"

    echo "OWASP Top 10 Checks:"
    echo "- Injection flaws: $(test_sql_injection "$url")"
    echo "- Broken authentication: $(test_auth_issues "$url")"
    echo "- Sensitive data exposure: $(test_data_exposure "$url")"
    echo "- XML external entities: $(test_xxe "$url")"
    echo "- Broken access control: $(test_access_control "$url")"
}

main() {
    local url=""
    local app_type="web"

    while getopts "u:t:" opt; do
        case $opt in
            u) url="$OPTARG" ;;
            t) app_type="$OPTARG" ;;
        esac
    done

    if [[ -z "$url" ]]; then
        echo "Usage: $0 -u <url> [-t app_type]"
        echo "App types: web, api, mobile"
        exit 1
    fi

    aud it_application "$url" "$app_type"
}

main "$@"
```

## Integrated Audit Management

### Multi-Type Audit Coordinator

```bash
#!/bin/bash

# audit_coordinator.sh
# Usage: ./audit_coordinator.sh -o <organization> -t <audit_types>

coordinate_audits() {
    local organization=$1
    shift
    local audit_types=("$@")
    local audit_id="${organization}_$(date +%Y%m%d)"

    echo "Coordinating Security Audits for: $organization"
    echo "=============================================="

    for audit_type in "${audit_types[@]}"; do
        echo "Processing $audit_type audit..."

        case $audit_type in
            internal)
                ./internal_audit.sh -d "$organization" -p "policies_${organization}.md"
                ;;
            external)
                ./external_audit_prep.sh -s "iso27001" -o "${audit_id}_external"
                ;;
            compliance)
                ./compliance_checker.sh -s "gdpr" -t "$organization"
                ;;
            technical)
                ./technical_audit.sh -t "$organization" -c "full"
                ;;
            network)
                ./network_audit.sh -n "192.168.1.0/24" -o "${audit_id}_network"
                ;;
            application)
                ./application_audit.sh -u "https://$organization" -t "web"
                ;;
        esac
    done

    # Generate consolidated report
    generate_consolidated_report "$audit_id" "${audit_types[@]}"

    echo "Audit coordination completed: $audit_id"
}

generate_consolidated_report() {
    local audit_id=$1
    shift
    local audit_types=("$@")
    local report_file="${audit_id}_consolidated_report.md"

    echo "# Consolidated Security Audit Report" > "$report_file"
    echo "## Organization: ${audit_id%_*}" >> "$report_file"
    echo "## Audit Date: $(date)" >> "$report_file"
    echo "" >> "$report_file"

    for audit_type in "${audit_types[@]}"; do
        echo "### ${audit_type^} Audit Findings" >> "$report_file"
        find . -name "*${audit_type}*" -name "*.txt" -exec cat {} \; >> "$report_file" 2>/dev/null
        echo "" >> "$report_file"
    done

    echo "Consolidated report: $report_file"
}

main() {
    local organization=""
    local audit_types=()

    while getopts "o:t:" opt; do
        case $opt in
            o) organization="$OPTARG" ;;
            t) audit_types+=("$OPTARG") ;;
        esac
    done

    if [[ -z "$organization" || ${#audit_types[@]} -eq 0 ]]; then
        echo "Usage: $0 -o <organization> -t <audit_type1> -t <audit_type2> ..."
        echo "Available types: internal external compliance technical network application"
        exit 1
    fi

    coordinate_audits "$organization" "${audit_types[@]}"
}

main "$@"
```

## Terminal Usage Examples

### Individual Audit Types

```bash
# Internal audit for specific domain
./internal_audit.sh -d company.com -p security_policies.md

# External audit preparation for PCI DSS
./external_audit_prep.sh -s pci -o pci_audit_prep

# Compliance check for GDPR
./compliance_checker.sh -s gdpr -t webapp.company.com

# Technical infrastructure audit
./technical_audit.sh -t server.company.com -c full

# Network security audit
./network_audit.sh -n 10.1.1.0/24 -o network_audit_results

# Web application audit
./application_audit.sh -u https://app.company.com -t web
```

### Coordinated Multi-Audit Approach

```bash
# Run multiple audit types for comprehensive assessment
./audit_coordinator.sh -o "SecurePay Inc" \
  -t internal \
  -t compliance \
  -t technical \
  -t application

# Generate specific audit combination
./audit_coordinator.sh -o "Healthcare Provider" \
  -t compliance \
  -t technical \
  -t network
```

## Hands-on Lab Exercise

### PortSwigger Web Security Academy

**Lab**: [SQL injection with filter bypass via XML encoding](https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding)

**Lab Setup**:

1. Free account at PortSwigger Web Security Academy
2. Temporary, isolated lab instances
3. No additional setup required

**Audit Objectives**:

- Perform application security audit
- Identify SQL injection vulnerabilities
- Test input validation mechanisms
- Assess XML parsing security

**Audit Commands**:

```bash
# Run comprehensive application audit
./application_audit.sh -u [LAB_URL] -t web

# Specific SQL injection testing
sqlmap -u "[LAB_URL]/filter?category=Tech" --batch --level=3

# Manual payload testing
curl -X POST "[LAB_URL]/search" -H "Content-Type: application/xml" -d '<search>test</search>'
```

## Key Audit Relationships

### Audit Type → Penetration Testing Focus

| Audit Type      | Penetration Testing Focus Areas                               |
| --------------- | ------------------------------------------------------------- |
| **Internal**    | Policy validation, access control testing, social engineering |
| **External**    | External perimeter testing, compliance verification           |
| **Compliance**  | Regulatory requirement testing, data protection validation    |
| **Technical**   | Infrastructure testing, configuration validation              |
| **Network**     | Network exploitation, service enumeration                     |
| **Application** | Web app vulnerabilities, API security testing                 |

### Best Practices for Audit Integration

1. **Use audit findings to scope penetration tests**
2. **Prioritize testing based on audit risk ratings**
3. **Validate audit recommendations through exploitation**
4. **Maintain clear documentation between audit and test phases**
5. **Coordinate timing between audit remediation and testing validation**

## Summary

Understanding different security audit types enables security professionals to:

- Select appropriate audit methodologies for organizational needs
- Translate audit findings into targeted penetration testing
- Prioritize security improvements based on audit results
- Maintain comprehensive security assessment coverage
- Demonstrate regulatory compliance through structured auditing

Each audit type serves specific purposes and provides unique insights that collectively contribute to a comprehensive security posture assessment.
