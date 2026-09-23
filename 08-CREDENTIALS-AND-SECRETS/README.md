# 08 — Credentials and Secrets

Credentials and secrets are one of the most important privilege-escalation discovery areas.

A low-privileged user may not need a technical exploit at all if they can discover:

* A password
* An SSH private key
* An API token
* A database credential
* A service credential
* A configuration secret
* A password stored in command history
* A reusable credential belonging to another user
* A secret exposed through an environment variable

The goal is not simply to **find secrets**.

The goal is to determine whether a discovered secret can cross a privilege boundary.

---

## Core Question

> **Can I discover a credential or secret that gives me access to a more privileged account, service, or resource?**

Finding a password is not automatically privilege escalation.

Finding a password for another user is a **finding**.

Determining that the password authenticates as a more privileged user creates a possible **privilege-escalation path**.

---

## Mental Model

Use this chain:

```text
FIND SECRET
     ↓
IDENTIFY OWNER
     ↓
IDENTIFY PURPOSE
     ↓
TEST WHETHER IT IS VALID
     ↓
DETERMINE PRIVILEGE LEVEL
     ↓
CHECK WHETHER IT IS REUSABLE
     ↓
VALIDATE PRIVILEGE BOUNDARY
```

Always ask:

```text
What is the secret?
Who does it belong to?
Where is it used?
Can I authenticate with it?
What privileges does it provide?
Does it cross the current privilege boundary?
```

---

## 1. Command History

Command history can expose credentials accidentally supplied directly on the command line.

### 🔎 ENUMERATE

Start with:

```bash
history
```

Then inspect common history files:

```bash
ls -la "$HOME"/.*history 2>/dev/null
```

```bash
cat "$HOME/.bash_history" 2>/dev/null
```

```bash
cat "$HOME/.zsh_history" 2>/dev/null
```

Search for potentially sensitive command patterns:

```bash
grep -Ei 'pass(word)?|passwd|secret|token|api[_-]?key|auth|credential|ssh|mysql|psql' "$HOME/.bash_history" 2>/dev/null
```

### 🧠 WHY

Users sometimes execute commands containing:

* Passwords
* Tokens
* Database credentials
* API keys
* SSH commands
* Administrative commands

### ⚠️ LOOK FOR

Examples:

```text
mysql -u admin -p'SecretPassword'
```

```text
curl -H "Authorization: Bearer TOKEN"
```

```text
ssh user@host
```

```text
psql postgresql://user:password@host/db
```

The exact format varies by application.

### ➡️ NEXT STEP

If a possible credential appears:

```text
Secret found
    ↓
Identify account/service
    ↓
Determine whether credential is still valid
    ↓
Determine privilege level
    ↓
Validate authorized access
```

Do not assume that a historical credential is still valid.

---

## 2. Environment Variables

Applications frequently receive secrets through environment variables.

### 🔎 ENUMERATE

Start with:

```bash
env
```

Then:

```bash
printenv
```

Sort the environment:

```bash
env | sort
```

Search for likely secret names:

```bash
env | grep -Ei 'pass(word)?|secret|token|key|api|auth|credential|database|db_'
```

Check the current process environment:

```bash
tr '\0' '\n' < /proc/$$/environ
```

### 🧠 WHY

Environment variables may contain:

* Database passwords
* API keys
* Access tokens
* Application secrets
* Service credentials
* Configuration values

### ⚠️ LOOK FOR

Examples:

```text
DB_PASSWORD=...
API_TOKEN=...
SECRET_KEY=...
AWS_ACCESS_KEY_ID=...
```

A variable name alone does not prove that the value is sensitive.

### ➡️ NEXT STEP

For an interesting variable:

```bash
echo "$VARIABLE_NAME"
```

Then determine:

```text
Who uses it?
Which application uses it?
What account runs that application?
Can the credential authenticate somewhere?
```

---

## 3. Configuration Files

Configuration files are one of the most common places to discover application credentials.

### 🔎 ENUMERATE

Search common configuration locations:

```bash
find /etc /opt /usr/local /var/www -type f \
\( -name "*.conf" -o -name "*.cfg" -o -name "*.ini" -o -name "*.yaml" -o -name "*.yml" -o -name "*.json" \) \
2>/dev/null
```

Search for credential-related filenames:

```bash
find /etc /opt /var/www /home -type f \
\( -iname "*password*" -o -iname "*credential*" -o -iname "*secret*" -o -iname "*config*" \) \
2>/dev/null
```

Search configuration content:

```bash
grep -RniE 'password|passwd|secret|token|api[_-]?key|credential' \
/etc /opt /var/www 2>/dev/null
```

### 🧠 WHY

Applications often need credentials to communicate with:

* Databases
* APIs
* Internal services
* Message queues
* Cloud services
* Other servers

### ⚠️ LOOK FOR

Typical patterns:

```text
username=...
password=...
```

```text
DB_USER=...
DB_PASSWORD=...
```

```text
api_key=...
```

```text
secret_key=...
```

### ➡️ NEXT STEP

Do not immediately treat every matching line as exploitable.

Determine:

```text
What application uses this?
Which account runs it?
What does the credential authenticate to?
What privileges does that account have?
```

---

## 4. Web Application Secrets

Web applications frequently store credentials inside application directories.

### 🔎 ENUMERATE

Check common locations:

```bash
ls -la /var/www 2>/dev/null
```

```bash
find /var/www -type f \
\( -name ".env" -o -name "*config*" -o -name "*database*" -o -name "*credential*" \) \
2>/dev/null
```

Search for likely secrets:

```bash
grep -RniE 'DB_PASSWORD|PASSWORD|SECRET_KEY|API_KEY|TOKEN' \
/var/www 2>/dev/null
```

Check application directories:

```bash
find /opt /srv /var/www -maxdepth 3 -type f 2>/dev/null
```

### 🧠 WHY

Applications commonly contain:

* Database credentials
* Session secrets
* API keys
* Service credentials
* Deployment credentials

### ⚠️ LOOK FOR

Pay particular attention to:

```text
.env
config.php
settings.py
application.yml
application.properties
database.yml
wp-config.php
```

The actual filenames depend on the application.

### ➡️ NEXT STEP

Follow the credential:

```text
Application
    ↓
Credential
    ↓
Database/service/account
    ↓
Privilege level
    ↓
Possible reuse
```

---

## 5. SSH Keys

SSH private keys can provide authentication without a password.

### 🔎 ENUMERATE

Inspect your own SSH directory:

```bash
ls -la "$HOME/.ssh" 2>/dev/null
```

Look for private-key filenames:

```bash
find "$HOME/.ssh" -type f \
\( -name "id_rsa" -o -name "id_ed25519" -o -name "id_ecdsa" -o -name "id_dsa" \) \
2>/dev/null
```

Search accessible home directories:

```bash
find /home -type f \
\( -name "id_rsa" -o -name "id_ed25519" -o -name "id_ecdsa" \) \
2>/dev/null
```

Inspect permissions:

```bash
ls -la "$HOME/.ssh" 2>/dev/null
```

```bash
stat "$HOME/.ssh/id_rsa" 2>/dev/null
```

### 🧠 WHY

An accessible private key may authenticate as another account.

### ⚠️ LOOK FOR

Potential private keys usually begin with:

```text
-----BEGIN OPENSSH PRIVATE KEY-----
```

or another private-key header.

Do not confuse:

```text
authorized_keys
```

with:

```text
id_rsa
id_ed25519
```

`authorized_keys` contains public keys accepted for authentication.

### ➡️ NEXT STEP

Determine:

```text
Who owns the key?
Is the private key readable?
Which account is it associated with?
Where could it authenticate?
What privilege level does that account have?
```

Only test authentication against systems you are authorized to access.

---

## 6. SSH Configuration

SSH configuration can reveal useful trust relationships.

### 🔎 ENUMERATE

Inspect your SSH configuration:

```bash
ls -la "$HOME/.ssh" 2>/dev/null
```

```bash
cat "$HOME/.ssh/config" 2>/dev/null
```

Check known hosts:

```bash
cat "$HOME/.ssh/known_hosts" 2>/dev/null
```

Check authorized keys:

```bash
cat "$HOME/.ssh/authorized_keys" 2>/dev/null
```

Inspect system SSH configuration:

```bash
cat /etc/ssh/ssh_config 2>/dev/null
```

```bash
cat /etc/ssh/sshd_config 2>/dev/null
```

### 🧠 WHY

SSH configuration can reveal:

* Hosts
* Usernames
* Aliases
* Identity files
* Authentication configuration
* Trust relationships

### ⚠️ LOOK FOR

Examples:

```text
Host internal-server
    User admin
    IdentityFile ~/.ssh/id_ed25519
```

This is a lead, not automatically a vulnerability.

---

## 7. Password Files and Account Information

### 🔎 ENUMERATE

Inspect account information:

```bash
cat /etc/passwd
```

Identify UID 0 accounts:

```bash
awk -F: '$3 == 0 {print}' /etc/passwd
```

Inspect password database permissions:

```bash
ls -la /etc/passwd /etc/shadow
```

Check whether the shadow file is accessible:

```bash
cat /etc/shadow 2>/dev/null
```

### 🧠 WHY

The account database identifies:

* Users
* UIDs
* Home directories
* Login shells
* Privileged accounts

`/etc/shadow` contains password hashes and is normally restricted.

### ⚠️ LOOK FOR

Important observations include:

```text
UID 0
```

Unexpectedly readable sensitive files:

```text
/etc/shadow
```

Unexpected account configurations.

### ❌ COMMON MISTAKE

Do not assume:

```text
UID 0 = root username only
```

UID 0 represents root-level privilege regardless of the username.

---

## 8. Database Credentials

Applications often store database credentials locally.

### 🔎 ENUMERATE

Search configuration files:

```bash
grep -RniE 'mysql|mariadb|postgres|postgresql|mongodb|redis|database' \
/etc /opt /var/www 2>/dev/null
```

Search for password fields:

```bash
grep -RniE 'password[[:space:]]*=' \
/etc /opt /var/www 2>/dev/null
```

Check local listeners:

```bash
ss -lntup
```

### 🧠 WHY

A database credential may provide:

* Access to application data
* Access to another account
* Administrative database privileges
* Information useful for another escalation path

### ⚠️ LOOK FOR

The important relationship is:

```text
Credential
    ↓
Database
    ↓
Database user
    ↓
Database privileges
    ↓
Potential impact
```

A database administrator is not automatically the same as Linux root.

Keep the privilege boundaries separate.

---

## 9. Application Credential Reuse

Credential reuse can turn one discovery into another access path.

### 🧠 CONCEPT

Suppose you discover:

```text
Username: admin
Password: discovered-password
```

Do not immediately assume:

```text
admin = root
```

Instead determine:

```text
Which service?
Which account?
Which system?
Which privilege level?
```

### 🔎 ENUMERATE

Identify local authentication services:

```bash
ss -lntup
```

Check known users:

```bash
cut -d: -f1,3,6,7 /etc/passwd
```

Inspect application configuration:

```bash
grep -RniE 'user(name)?|login|password|passwd' \
/etc /opt /var/www 2>/dev/null
```

### ➡️ NEXT STEP

Build the relationship:

```text
Credential
   ↓
Account
   ↓
Service
   ↓
Privilege
```

Only authorized authentication testing should be performed.

---

## 10. History, Logs, and Temporary Files

Sensitive information can also appear in operational artifacts.

### 🔎 ENUMERATE

Check temporary directories:

```bash
ls -la /tmp
```

```bash
ls -la /var/tmp
```

```bash
ls -la /dev/shm
```

Search for potentially interesting files:

```bash
find /tmp /var/tmp /dev/shm -type f \
\( -name "*.log" -o -name "*.conf" -o -name "*.txt" -o -name "*.bak" \) \
2>/dev/null
```

Check common log locations:

```bash
ls -la /var/log
```

Search selectively:

```bash
grep -RniE 'password|token|secret|credential' /var/log 2>/dev/null
```

### ⚠️ LOOK FOR

Examples:

```text
temporary configuration
debug output
deployment logs
application logs
backup files
command output
```

### ❌ COMMON MISTAKE

Do not dump or copy large amounts of sensitive data unnecessarily.

Collect only the evidence required to validate the finding.

---

## 11. Backup Files

Backups can contain older versions of configuration files containing credentials.

### 🔎 ENUMERATE

Search common locations:

```bash
find /etc /opt /var/www /home -type f \
\( -name "*.bak" -o -name "*.backup" -o -name "*.old" -o -name "*.save" -o -name "*~" \) \
2>/dev/null
```

Inspect candidates:

```bash
ls -la /path/to/file
```

```bash
stat /path/to/file
```

```bash
sed -n '1,240p' /path/to/file
```

### 🧠 WHY

A current configuration may have been cleaned while an old backup still contains:

* Passwords
* API keys
* Database credentials
* SSH information
* Internal hostnames

### ➡️ NEXT STEP

Determine whether the credential is:

```text
Current
Expired
Reused
Application-specific
Privileged
```

---

## 12. Search Strategy

Avoid blindly searching the entire filesystem for every possible keyword.

Start with high-value locations.

### 🔎 TARGETED SEARCH

```bash
grep -RniE 'password|passwd|secret|token|api[_-]?key|credential' \
/etc /opt /srv /var/www /home 2>/dev/null
```

Then investigate interesting files individually.

### 🧠 WHY

Targeted searches:

* Reduce noise
* Reduce unnecessary data collection
* Make findings easier to understand
* Improve validation speed

### ⚠️ LOOK FOR

Prioritize:

```text
/root
/home
/etc
/opt
/srv
/var/www
/usr/local
```

Then follow application-specific paths discovered during enumeration.

---

## 13. Ownership and Permissions Matter

A credential is only useful if you can access it.

### 🔎 ENUMERATE

For a discovered file:

```bash
ls -la /path/to/file
```

```bash
stat /path/to/file
```

```bash
namei -l /path/to/file
```

### 🧠 WHY

You need to understand:

```text
Who owns the file?
What permissions exist?
Can my user read it?
Can my group read it?
Can another process expose it?
```

### ⚠️ LOOK FOR

Example:

```text
-rw------- 1 root root ... secrets.conf
```

If your user cannot read it, the file itself is not currently an accessible credential source.

Compare with:

```text
-rw-r--r-- 1 root root ... config.conf
```

The second file may expose information to unprivileged users.

---

## 14. Credentials in Processes

A running process may expose useful command-line arguments or environment information.

### 🔎 ENUMERATE

List processes:

```bash
ps aux
```

Inspect command lines:

```bash
ps -eo user,pid,ppid,args
```

Inspect a specific process:

```bash
tr '\0' ' ' < /proc/<PID>/cmdline
```

Inspect its environment:

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

### 🧠 WHY

Some applications may expose configuration through:

* Command-line arguments
* Environment variables
* Process configuration

### ⚠️ LOOK FOR

Patterns such as:

```text
--password
--token
--api-key
```

or environment variables containing secrets.

### ➡️ NEXT STEP

Ask:

```text
Who owns the process?
What service is running?
What privilege does it have?
Can the discovered credential be reused?
```

---

## 15. Credential Discovery Decision Tree

```text
                    START
                      │
                      ▼
             Search high-value sources
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
    History        Configs        SSH Keys
       │              │              │
       └──────────────┼──────────────┘
                      ▼
                Secret Found?
                 │          │
                NO         YES
                 │          │
                 ▼          ▼
            Move on    Identify owner
                              │
                              ▼
                       Identify purpose
                              │
                              ▼
                      Can you access it?
                         │          │
                        NO         YES
                         │          │
                         ▼          ▼
                    Move on    Test validity
                                    │
                                    ▼
                              Identify privilege
                                    │
                         ┌──────────┴──────────┐
                         ▼                     ▼
                    Low privilege        Higher privilege
                         │                     │
                         ▼                     ▼
                    Reassess path       Validate boundary
                                               │
                                               ▼
                                      Authorized escalation
```

---

## 16. High-Value Patterns

### 🔎 Pattern 1 — Privileged Account Credential

```text
Credential discovered
        ↓
Belongs to privileged account
        ↓
Credential is accessible
        ↓
Credential is valid
        ↓
Authentication crosses privilege boundary
```

This requires validation.

---

### 🔎 Pattern 2 — Root-Owned Application Configuration

```text
Root-owned application
        ↓
Configuration readable by current user
        ↓
Configuration contains credential
        ↓
Credential belongs to privileged service/account
```

Investigate the relationship before exploitation.

---

### 🔎 Pattern 3 — SSH Private Key

```text
Private key discovered
        ↓
Key readable by current user
        ↓
Identify associated account
        ↓
Authorized SSH target
        ↓
Determine privilege level
```

---

### 🔎 Pattern 4 — Credential Reuse

```text
Credential discovered
        ↓
Identify account
        ↓
Identify other services
        ↓
Authorized authentication test
        ↓
Higher-privileged account?
```

---

### 🔎 Pattern 5 — Process Exposure

```text
Privileged process
        ↓
Command line/environment inspected
        ↓
Credential exposed
        ↓
Credential belongs to another account/service
        ↓
Validate access
```

---

## 17. Finding vs Vulnerability vs Exploitable Path

Always distinguish these three states.

### Finding

```text
Password discovered in a configuration file.
```

This is an observation.

### Vulnerability

```text
The configuration exposes a valid credential to an unauthorized low-privileged user.
```

Now there is a security weakness.

### Exploitable Path

```text
The credential authenticates as a more privileged account
and allows the current user to cross the privilege boundary.
```

That is an actual escalation path.

---

## 18. Safe Validation

Before treating a credential as a privilege-escalation path, validate:

```text
[ ] Credential is actually accessible
[ ] Credential belongs to an identified account/service
[ ] Credential is valid
[ ] Authentication method is understood
[ ] Target is authorized
[ ] Account privilege level is known
[ ] Access crosses the intended privilege boundary
[ ] Evidence has been collected
```

### 🧪 VALIDATE

After authorized authentication, verify identity:

```bash
whoami
```

```bash
id
```

Then compare:

```text
Before:
current-user

After:
authenticated-user
```

The important result is the privilege boundary crossed, not merely successful authentication.

---

## 19. Common Mistakes

### ❌ Mistake 1 — Treating Every Password as Useful

A password may be:

* Expired
* Invalid
* Application-specific
* Low privilege
* Decoy data
* Used only for a non-privileged service

Always validate.

### ❌ Mistake 2 — Ignoring File Permissions

Finding a secret does not matter if you cannot actually access it.

### ❌ Mistake 3 — Assuming Username Equals Privilege

```text
admin
```

does not automatically mean:

```text
root
```

Determine the actual privilege level.

### ❌ Mistake 4 — Ignoring Credential Reuse

A credential may be useful somewhere other than where it was discovered.

### ❌ Mistake 5 — Dumping Everything

Mass collection creates noise and unnecessary exposure.

Use targeted searches.

### ❌ Mistake 6 — Confusing Database Privileges with Linux Privileges

Database administrator privileges and Linux root privileges are different privilege boundaries.

### ❌ Mistake 7 — Skipping Validation

```text
Found password
```

is not the same as:

```text
Confirmed privilege escalation
```

---

## 20. Five-Minute Credential Check

When time is limited:

### 🔎 ENUMERATE

```bash
history
```

```bash
env | grep -Ei 'pass(word)?|secret|token|key|api|auth|credential'
```

```bash
ls -la "$HOME/.ssh" 2>/dev/null
```

```bash
find /var/www /opt /home -type f \
\( -name ".env" -o -name "*config*" -o -name "*credential*" -o -name "*secret*" \) \
2>/dev/null
```

```bash
grep -RniE 'password|passwd|secret|token|api[_-]?key|credential' \
/etc /opt /var/www /home 2>/dev/null
```

```bash
ps -eo user,pid,ppid,args
```

Then ask:

```text
Did I find a credential?
        │
       YES
        │
        ▼
Who owns it?
        │
        ▼
What does it access?
        │
        ▼
What privilege does it provide?
        │
        ▼
Can it cross my privilege boundary?
```

---

## 21. Quick Command Reference

| Goal                          | Command                                                                            |
| ----------------------------- | ---------------------------------------------------------------------------------- |
| Command history               | `history`                                                                          |
| Bash history                  | `cat ~/.bash_history`                                                              |
| Environment                   | `env`                                                                              |
| Search environment            | `env \| grep -Ei 'pass\|secret\|token\|key'`                                       |
| SSH directory                 | `ls -la ~/.ssh`                                                                    |
| SSH private keys              | `find /home -type f \( -name "id_rsa" -o -name "id_ed25519" \) 2>/dev/null`        |
| Account list                  | `cat /etc/passwd`                                                                  |
| UID 0 accounts                | `awk -F: '$3 == 0 {print}' /etc/passwd`                                            |
| Password database permissions | `ls -la /etc/passwd /etc/shadow`                                                   |
| Config search                 | `find /etc /opt /var/www -type f -name "*.conf" 2>/dev/null`                       |
| Secret search                 | `grep -RniE 'password\|secret\|token\|api[_-]?key' /etc /opt /var/www 2>/dev/null` |
| Processes                     | `ps aux`                                                                           |
| Process command lines         | `ps -eo user,pid,ppid,args`                                                        |
| Process environment           | `tr '\0' '\n' < /proc/<PID>/environ`                                               |
| Temporary files               | `find /tmp /var/tmp /dev/shm -type f 2>/dev/null`                                  |
| File permissions              | `stat /path/to/file`                                                               |
| Path permissions              | `namei -l /path/to/file`                                                           |

---

## 22. Complete Checklist

### 🔎 Credential Discovery

```text
[ ] Check command history
[ ] Check shell history files
[ ] Check environment variables
[ ] Check application configuration
[ ] Check web application files
[ ] Check SSH directories
[ ] Check accessible private keys
[ ] Check database configuration
[ ] Check backup files
[ ] Check temporary files
[ ] Check process command lines
[ ] Check process environments
```

### 🧠 Credential Analysis

```text
[ ] Identify the credential
[ ] Identify its owner
[ ] Identify its purpose
[ ] Identify the target service
[ ] Determine whether it is accessible
[ ] Determine whether it is valid
[ ] Determine account privilege
[ ] Check for authorized credential reuse
```

### 🧪 Validation

```text
[ ] Validate only in an authorized environment
[ ] Confirm authentication
[ ] Confirm resulting identity
[ ] Confirm privilege level
[ ] Confirm privilege boundary crossing
[ ] Record evidence
```

---

## 23. Master Decision Model

Use this mental model whenever you discover a credential:

```text
             CREDENTIAL FOUND
                    │
                    ▼
             CAN I ACCESS IT?
              │            │
             NO           YES
              │            │
              ▼            ▼
          MOVE ON     WHO OWNS IT?
                           │
                           ▼
                     WHAT DOES IT
                       ACCESS?
                           │
                           ▼
                    IS IT VALID?
                     │        │
                    NO       YES
                     │        │
                     ▼        ▼
                 MOVE ON   WHAT PRIVILEGE?
                              │
                              ▼
                       CAN IT CROSS THE
                       PRIVILEGE BOUNDARY?
                         │            │
                        NO           YES
                         │            │
                         ▼            ▼
                     MOVE ON      VALIDATE
                                      │
                                      ▼
                                   VERIFY
```

---

## 💡 Remember

Do not memorize hundreds of secret locations.

Memorize the questions:

```text
Where could credentials be stored?
Who owns the credential?
What service uses it?
Can I access it?
Is it valid?
What account does it authenticate as?
What privilege does that account have?
Can it cross the privilege boundary?
```

The key pattern is:

> **Find the secret → identify the owner → identify the service → validate the credential → determine privilege → verify the boundary.**

A credential becomes important for privilege escalation only when it creates a real, validated path to greater privilege.

---

## Where Findings Lead

Credential discoveries can lead to:

```text
Command History
       ↓
Credentials / Secrets
       ↓
Credential Reuse
       ↓
Authentication
       ↓
Higher-Privileged Account
```

or:

```text
Configuration File
       ↓
Service Credential
       ↓
Privileged Service
       ↓
Service Abuse
```

or:

```text
SSH Private Key
       ↓
Account Authentication
       ↓
Higher-Privileged User
       ↓
Privilege Boundary
```

Related stages:

* `02-SYSTEM-ENUMERATION/` — users, groups, accounts, installed software
* `03-FILESYSTEM-ENUMERATION/` — permissions, configuration files, backups
* `04-PROCESS-SERVICE-ENUMERATION/` — processes and privileged services
* `06-PRIVILEGE-MECHANISMS/` — sudo, SUID, capabilities, privileged groups
* `09-EXECUTION-ABUSE/` — abusing trusted execution mechanisms
* `13-FINDING-VALIDATION/` — determining whether a credential creates a real vulnerability
* `14-EXPLOITATION/` — authorized exploitation of validated paths

---

## Final Mental Model

```text
             LOW-PRIVILEGED SHELL
                      │
                      ▼
             SEARCH FOR SECRETS
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       HISTORY      CONFIG      SSH KEY
          │           │           │
          └───────────┼───────────┘
                      ▼
                SECRET FOUND
                      │
                      ▼
                IDENTIFY OWNER
                      │
                      ▼
                IDENTIFY SERVICE
                      │
                      ▼
                 VALIDATE
                      │
                      ▼
             DETERMINE PRIVILEGE
                      │
                      ▼
          PRIVILEGE BOUNDARY CROSSED?
                 │           │
                NO          YES
                 │           │
                 ▼           ▼
              MOVE ON     VERIFY
```

> **Credentials are clues. Validated privilege boundaries are the objective.**
