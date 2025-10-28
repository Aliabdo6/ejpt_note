# Web Server Enumeration with Metasploit

## Overview

Web servers are software applications that serve website content over HTTP/HTTPS protocols. This guide covers comprehensive web server enumeration techniques using Metasploit auxiliary modules to identify server versions, discover hidden directories and files, analyze HTTP headers, and perform authentication brute force attacks.

## Web Server Fundamentals

- **Ports**: 80/HTTP, 443/HTTPS
- **Common Servers**: Apache, Nginx, Microsoft IIS
- **Protocol**: HTTP/HTTPS for client-server communication
- **Key Files**: robots.txt, .htaccess, web.config
- **Authentication**: Basic, Digest, Form-based

## Lab Environment Setup

For practical exercises, use the **TryHackMe** or **Hack The Box** platforms:

**Lab Recommendations**:

- TryHackMe: [Basic Pentesting](https://tryhackme.com/room/basicpentestingjt)
- Hack The Box: Active machines with web services

**Setup Requirements**:

- Target machine with web service running
- Network connectivity to target
- Metasploit Framework installed

## Initial Setup and Configuration

### Starting Metasploit and Workspace

```bash
# Start database and Metasploit
sudo systemctl start postgresql
msfconsole

# Create dedicated workspace
msf6 > workspace -a web_enum
msf6 > workspace web_enum

# Set global RHOSTS variable
msf6 > setg RHOSTS 192.168.1.100
msf6 > setg RHOST 192.168.1.100
```

### Dynamic Target Configuration Script

```bash
#!/bin/bash

TARGET=$1
WORKSPACE="web_scan_$(date +%Y%m%d_%H%M%S)"

echo "[*] Starting web server enumeration"
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
chmod +x web_setup.sh
./web_setup.sh 192.168.1.100
```

## Service Discovery and Version Detection

### HTTP Version Detection

```bash
msf6 > search type:auxiliary name:http

msf6 > use auxiliary/scanner/http/http_version
msf6 auxiliary(scanner/http/http_version) > show options

Module options (auxiliary/scanner/http/http_version):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                    yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT    80               yes       The target port (TCP)
   SSL      false            no        Negotiate SSL/TLS for outgoing connections
   THREADS  1                yes       The number of concurrent threads
   VHOST                     no        HTTP server virtual host

msf6 auxiliary(scanner/http/http_version) > run

[*] 192.168.1.100:80 - Server: Apache/2.4.18 (Ubuntu)
[*] Scanned 1 of 1 hosts (100% complete)
```

### HTTPS Version Detection

```bash
msf6 auxiliary(scanner/http/http_version) > set RPORT 443
msf6 auxiliary(scanner/http/http_version) > set SSL true
msf6 auxiliary(scanner/http/http_version) > run
```

## HTTP Header Analysis

### HTTP Header Scanner

```bash
msf6 > use auxiliary/scanner/http/http_header
msf6 auxiliary(scanner/http/http_header) > show options

Module options (auxiliary/scanner/http/http_header):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   IGN_HEADER  Date, Content-Length, Connection, E-Tag, X-Powered-By  no        A list of headers to ignore, separated by comma
   METHOD      HEAD             yes       HTTP method to use: HEAD or GET (HEAD the default)
   Proxies                      no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                       yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT       80               yes       The target port (TCP)
   SSL         false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI   /                yes       The base path
   VHOST                        no        HTTP server virtual host

msf6 auxiliary(scanner/http/http_header) > run

[*] 192.168.1.100:80 - Analyzing https://192.168.1.100:80/
[*] 192.168.1.100:80 - Server: Apache/2.4.18 (Ubuntu)
[*] 192.168.1.100:80 - Powered By: PHP/7.2.24-0ubuntu0.18.04.17
[*] 192.168.1.100:80 - Content-Type: text/html; charset=UTF-8
```

### Comprehensive Header Analysis Script

```bash
#!/bin/bash

TARGET=$1
PORT=${2:-80}
SSL=${3:-false}

echo "[*] Analyzing HTTP headers for $TARGET:$PORT"

msfconsole -q -x "
workspace -a header_analysis;
setg RHOSTS $TARGET;
setg RPORT $PORT;
setg SSL $SSL;
use auxiliary/scanner/http/http_header;
set IGN_HEADER '';
run;
exit"
```

## Robots.txt Analysis

### Robots.txt Scanner

```bash
msf6 > use auxiliary/scanner/http/robots_txt
msf6 auxiliary(scanner/http/robots_txt) > show options

Module options (auxiliary/scanner/http/robots_txt):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   PATH        /                yes       The test path to find robots.txt
   Proxies                      no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                       yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT       80               yes       The target port (TCP)
   SSL         false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI   /                yes       The base path
   VHOST                        no        HTTP server virtual host

msf6 auxiliary(scanner/http/robots_txt) > run

[*] 192.168.1.100:80 - Found robots.txt
[*] 192.168.1.100:80 - Contents of robots.txt:
User-agent: *
Disallow: /data/
Disallow: /secure/
```

### Manual Verification with curl

```bash
# Check discovered directories
curl http://192.168.1.100/data/
curl http://192.168.1.100/secure/

# Check HTTP status codes
curl -I http://192.168.1.100/secure/
# Returns: HTTP/1.1 401 Unauthorized
```

## Directory Brute Forcing

### Directory Scanner

```bash
msf6 > use auxiliary/scanner/http/dir_scanner
msf6 auxiliary(scanner/http/dir_scanner) > show options

Module options (auxiliary/scanner/http/dir_scanner):

   Name        Current Setting                                  Required  Description
   ----        ---------------                                  --------  -----------
   DICTIONARY  /usr/share/metasploit-framework/data/wmap/wmap_dirs.txt  no        Path of word dictionary to use
   PATH        /                                                yes       The path to identify directories
   Proxies                                                      no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                                                       yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT       80                                               yes       The target port (TCP)
   SSL         false                                            no        Negotiate SSL/TLS for outgoing connections
   THREADS     10                                               yes       The number of concurrent threads
   VHOST                                                        no        HTTP server virtual host

msf6 auxiliary(scanner/http/dir_scanner) > run

[*] 192.168.1.100:80 - Found directory: /data/ 200
[*] 192.168.1.100:80 - Found directory: /secure/ 401
[*] 192.168.1.100:80 - Found directory: /downloads/ 200
[*] 192.168.1.100:80 - Found directory: /webdev/ 401
[*] 192.168.1.100:80 - Found directory: /webmail/ 401
[*] 192.168.1.100:80 - Found directory: /webdb/ 401
```

### Advanced Directory Scanning

```bash
# Use different wordlists
msf6 auxiliary(scanner/http/dir_scanner) > set DICTIONARY /usr/share/wordlists/dirb/common.txt
msf6 auxiliary(scanner/http/dir_scanner) > set THREADS 20
msf6 auxiliary(scanner/http/dir_scanner) > run
```

## File Brute Forcing

### File/Directory Scanner

```bash
msf6 > use auxiliary/scanner/http/files_dir
msf6 auxiliary(scanner/http/files_dir) > show options

Module options (auxiliary/scanner/http/files_dir):

   Name        Current Setting                                  Required  Description
   ----        ---------------                                  --------  -----------
   DICTIONARY  /usr/share/metasploit-framework/data/wmap/wmap_files.txt  no        Path of word dictionary to use
   EXTensions                                                   no        A list of extensions to check, separated by comma
   PATH        /                                                yes       The path to identify files
   Proxies                                                      no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                                                       yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT       80                                               yes       The target port (TCP)
   SSL         false                                            no        Negotiate SSL/TLS for outgoing connections
   THREADS     10                                               yes       The number of concurrent threads
   VHOST                                                        no        HTTP server virtual host

msf6 auxiliary(scanner/http/files_dir) > run

[*] 192.168.1.100:80 - Found file: /code.c 200
[*] 192.168.1.100:80 - Found file: /index.html 200
[*] 192.168.1.100:80 - Found file: /test.php 200
```

### Targeted File Discovery

```bash
# Search for specific file types
msf6 auxiliary(scanner/http/files_dir) > set EXTensions php,html,txt
msf6 auxiliary(scanner/http/files_dir) > set DICTIONARY /usr/share/wordlists/dirb/common.txt
msf6 auxiliary(scanner/http/files_dir) > run
```

## Authentication Brute Forcing

### HTTP Login Scanner

```bash
msf6 > use auxiliary/scanner/http/http_login
msf6 auxiliary(scanner/http/http_login) > show options

Module options (auxiliary/scanner/http/http_login):

   Name              Current Setting                      Required  Description
   ----              ---------------                      --------  -----------
   AUTH_URI          /secure/                             yes       The URI to authenticate against
   BLANK_PASSWORDS   false                                no        Try blank passwords for all users
   BRUTEFORCE_SPEED  5                                    yes       How fast to bruteforce, from 0 to 5
   DB_ALL_CREDS      false                                no        Try each user/password couple stored in the current database
   DB_ALL_PASS       false                                no        Add all passwords in the current database to the list
   DB_ALL_USERS      false                                no        Add all users in the current database to the list
   PASSWORD                                           no        A specific password to authenticate with
   PASS_FILE         /usr/share/metasploit-framework/data/wmap/wmap_pass.txt  no        File containing passwords, one per line
   Proxies                                                 no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                                                yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT             80                                  yes       The target port (TCP)
   SSL               false                                no        Negotiate SSL/TLS for outgoing connections
   STOP_ON_SUCCESS   false                                yes       Stop guessing when a credential works for a host
   THREADS           1                                   yes       The number of concurrent threads per host
   USERPASS_FILE                                       no        File containing users and passwords separated by space, one pair per line
   USER_AS_PASS      false                                no        Try the username as the password for all users
   USER_FILE         /usr/share/metasploit-framework/data/wmap/wmap_users.txt  no        File containing usernames, one per line
   VERBOSE           true                                 yes       Whether to print output for all attempts
   VHOST                                                 no        HTTP server virtual host
```

### Targeted Authentication Attack

```bash
# Configure for specific directory
msf6 auxiliary(scanner/http/http_login) > set AUTH_URI /secure/
msf6 auxiliary(scanner/http/http_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
msf6 auxiliary(scanner/http/http_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
msf6 auxiliary(scanner/http/http_login) > set STOP_ON_SUCCESS true
msf6 auxiliary(scanner/http/http_login) > set THREADS 5
msf6 auxiliary(scanner/http/http_login) > set VERBOSE false
msf6 auxiliary(scanner/http/http_login) > run
```

## User Enumeration

### Apache User Directory Enumeration

```bash
msf6 > use auxiliary/scanner/http/apache_userdir_enum
msf6 auxiliary(scanner/http/apache_userdir_enum) > show options

Module options (auxiliary/scanner/http/apache_userdir_enum):

   Name        Current Setting                                  Required  Description
   ----        ---------------                                  --------  -----------
   Proxies                                                      no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                                                       yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT       80                                               yes       The target port (TCP)
   SSL         false                                            no        Negotiate SSL/TLS for outgoing connections
   THREADS     1                                                yes       The number of concurrent threads
   USER_FILE   /usr/share/metasploit-framework/data/wordlists/common_users.txt  no        File containing users, one per line
   VHOST                                                        no        HTTP server virtual host

msf6 auxiliary(scanner/http/apache_userdir_enum) > run

[*] 192.168.1.100:80 - User found: rooty
```

### Advanced User Enumeration Script

```bash
#!/bin/bash

TARGET=$1
USER_LIST=${2:-"/usr/share/metasploit-framework/data/wordlists/common_users.txt"}

echo "[*] Enumerating web server users on $TARGET"

msfconsole -q -x "
workspace -a user_enum;
setg RHOSTS $TARGET;
use auxiliary/scanner/http/apache_userdir_enum;
set USER_FILE $USER_LIST;
run;
exit"
```

## Complete Web Server Enumeration Workflow

### Comprehensive Assessment Script

```bash
#!/bin/bash

TARGET=$1
OUTPUT_DIR="web_full_enum_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT_DIR

echo "[*] Starting comprehensive web server enumeration"
echo "[*] Target: $TARGET"
echo "[*] Output directory: $OUTPUT_DIR"

msfconsole -q -x "
workspace -a full_web_scan;
setg RHOSTS $TARGET;

echo '[+] Detecting web server version';
use auxiliary/scanner/http/http_version;
run;

echo '[+] Analyzing HTTP headers';
use auxiliary/scanner/http/http_header;
set IGN_HEADER '';
run;

echo '[+] Checking robots.txt';
use auxiliary/scanner/http/robots_txt;
run;

echo '[+] Performing directory brute force';
use auxiliary/scanner/http/dir_scanner;
set THREADS 10;
run;

echo '[+] Performing file brute force';
use auxiliary/scanner/http/files_dir;
set THREADS 10;
run;

echo '[+] Enumerating users';
use auxiliary/scanner/http/apache_userdir_enum;
run;

echo '[+] Web server enumeration complete';
services;
exit" > $OUTPUT_DIR/full_report.txt

echo "[+] Comprehensive report saved to $OUTPUT_DIR/full_report.txt"
```

**Usage**:

```bash
chmod +x web_complete_enum.sh
./web_complete_enum.sh 192.168.1.100
```

## Advanced Enumeration Techniques

### Backup File Discovery

```bash
msf6 > use auxiliary/scanner/http/backup_file
msf6 auxiliary(scanner/http/backup_file) > show options

Module options (auxiliary/scanner/http/backup_file):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   EXTensions  bak              no        A list of extensions to check, separated by comma
   PATH        /                yes       The path to identify files
   Proxies                      no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                       yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT       80               yes       The target port (TCP)
   SSL         false            no        Negotiate SSL/TLS for outgoing connections
   THREADS     1                yes       The number of concurrent threads
   VHOST                        no        HTTP server virtual host

msf6 auxiliary(scanner/http/backup_file) > set EXTensions bak,old,backup,tmp
msf6 auxiliary(scanner/http/backup_file) > run
```

### HTTP Method Enumeration

```bash
msf6 > use auxiliary/scanner/http/http_methods
msf6 auxiliary(scanner/http/http_methods) > show options

Module options (auxiliary/scanner/http/http_methods):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   Proxies                      no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                       yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT       80               yes       The target port (TCP)
   SSL         false            no        Negotiate SSL/TLS for outgoing connections
   THREADS     1                yes       The number of concurrent threads
   VHOST                        no        HTTP server virtual host

msf6 auxiliary(scanner/http/http_methods) > run
```

## Manual Verification Techniques

### Using curl for Manual Testing

```bash
# Check directory listing
curl http://192.168.1.100/data/

# Check authentication requirements
curl -I http://192.168.1.100/secure/

# Test specific files
curl http://192.168.1.100/test.php

# Check with different HTTP methods
curl -X OPTIONS http://192.168.1.100/
```

### Automated Manual Testing Script

```bash
#!/bin/bash

TARGET=$1

echo "[*] Manual verification for $TARGET"

# Check common endpoints
for endpoint in /robots.txt /.htaccess /web.config /phpinfo.php; do
    echo "[*] Testing $endpoint"
    curl -s -I "http://$TARGET$endpoint" | head -1
done

# Check discovered directories
for dir in /data/ /secure/ /downloads/; do
    echo "[*] Testing directory $dir"
    curl -s -I "http://$TARGET$dir" | head -1
done
```

## Best Practices and Security Considerations

### Optimization Tips

```bash
# For stealth operations
msf6 auxiliary(scanner/http/dir_scanner) > set THREADS 5
msf6 auxiliary(scanner/http/http_login) > set BRUTEFORCE_SPEED 3

# For comprehensive testing
msf6 auxiliary(scanner/http/http_login) > set BLANK_PASSWORDS true
msf6 auxiliary(scanner/http/http_login) > set USER_AS_PASS true
```

### Common Web Server Security Issues

1. **Information Disclosure**: Version numbers, directory listings
2. **Default Files**: Backup files, configuration files
3. **Weak Authentication**: Default credentials, weak passwords
4. **Misconfigurations**: Directory traversal, HTTP methods
5. **Outdated Software**: Known vulnerabilities in web server software

### Defensive Recommendations

- Disable directory listings
- Remove version information from headers
- Implement proper access controls
- Use strong authentication mechanisms
- Regularly update web server software
- Monitor for suspicious activity

## Information Usage Strategy

### Leveraging Web Server Findings

1. **Version Information**: Research known vulnerabilities
2. **Directory Structure**: Map application architecture
3. **Authentication Points**: Target for credential attacks
4. **File Discoveries**: Analyze for sensitive information
5. **User Enumeration**: Build target lists for further attacks

### Integration with Other Attack Vectors

```bash
# Chain web enumeration with other services
./web_complete_enum.sh 192.168.1.100
grep "401" web_results.txt | awk -F'/' '{print $3}' > auth_directories.txt
./http_login_attack.sh 192.168.1.100 auth_directories.txt
```

## Next Steps

After successful web server enumeration:

1. Research vulnerabilities for identified web server versions
2. Test discovered files and directories for sensitive information
3. Attempt authentication bypass on protected directories
4. Look for web application vulnerabilities (SQLi, XSS, etc.)
5. Move to database enumeration if web applications are present

This web server enumeration methodology provides a thorough approach to assessing web services, from initial discovery to detailed directory and file analysis, enabling comprehensive penetration testing of web infrastructure.
