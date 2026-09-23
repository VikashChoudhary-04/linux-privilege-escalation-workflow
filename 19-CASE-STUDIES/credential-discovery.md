# Credential Discovery — Case Study

> **Purpose:** Demonstrate how credentials discovered during Linux enumeration can become a privilege-escalation path when they belong to a more privileged account or provide access to a privileged service.

---

## Scenario

A low-privileged user discovers a credential, secret, key, or authentication material somewhere on the system.

The discovery itself is not the escalation.

The investigation is:

```text id="7m4q2x"
What was discovered?
        ↓
Who does it belong to?
        ↓
Where can it be used?
        ↓
What privilege does that account have?
        ↓
Can the credential be used within the authorized scope?
        ↓
Can the resulting privilege be verified?
```

The lab should be performed only in an authorized environment.

---

## Initial Context

Start with:

```bash id="5x8m2q"
whoami
id
hostname
```

Record:

```text id="9q4m7x"
Initial User:
UID:
Groups:
Hostname:
```

Establish system context:

```bash id="3m6q8x"
uname -a
cat /etc/os-release
uname -m
```

Record:

```text id="8x5m2q"
OS:
Version:
Kernel:
Architecture:
```

---

## Credential Sources

Credentials can appear in many locations.

Common areas to investigate include:

```text id="6q3m9x"
/home/
/opt/
/etc/
/var/
/usr/local/
```

Potential sources include:

```text id="4m7x2q"
Shell history
Configuration files
Application configuration
Service configuration
Backup files
SSH keys
Environment variables
Scripts
Database configuration
API tokens
Credential stores
```

The exact sources depend on the system.

---

## Command History

Check relevant history files where authorized:

```bash id="8x2q5m"
ls -la ~
```

Common history files include:

```text id="5m9q3x"
~/.bash_history
~/.zsh_history
```

If a history file exists:

```bash id="7x4m8q"
cat ~/.bash_history
```

Look for evidence of:

```text id="2m8q6x"
Passwords
SSH commands
Administrative commands
Database credentials
Service credentials
Tokens
Configuration paths
```

Do not unnecessarily expose or copy discovered secrets.

---

## Environment Variables

Inspect the environment:

```bash id="9m3q5x"
env
```

Also:

```bash id="6x8m2q"
printenv
```

Look for application-related variables such as:

```text id="3q7m9x"
PASSWORD
PASS
TOKEN
SECRET
KEY
API_KEY
DATABASE_URL
```

Names vary widely.

A variable that looks sensitive must still be analyzed before being treated as a valid credential.

---

## Configuration Files

Search relevant application directories:

```bash id="5x4m8q"
ls -la /opt/
ls -la /var/www/ 2>/dev/null
ls -la /usr/local/ 2>/dev/null
```

Then inspect known configuration files.

Examples of potentially interesting extensions:

```text id="8m2q7x"
.conf
.config
.ini
.env
.yaml
.yml
.json
.xml
```

Do not blindly dump entire directories.

Start with files associated with services or applications discovered during enumeration.

---

## File Permissions Matter

Finding a credential is only the first step.

Check:

```bash id="7m3x9q"
ls -l /path/to/file
stat /path/to/file
```

Ask:

```text id="4x8m6q"
Why can the current user read this?
Who owns the file?
Who owns the credential?
Is the credential still valid?
```

---

## Identify the Credential Owner

The most important question is:

> Who does this credential authenticate as?

Possible identities include:

```text id="6q5m2x"
Current user
Another normal user
Service account
Administrator
Root
Database account
Application account
```

Do not assume that a discovered password belongs to the file owner.

Establish the relationship from context.

---

## Determine the Privilege Level

If the credential belongs to a local Linux account, investigate that account where appropriate.

For example:

```bash id="9x3m7q"
grep '^username:' /etc/passwd
```

Then determine group membership where authorized:

```bash id="5m8q2x"
id username
```

The goal is to determine:

```text id="7q4m9x"
Credential
   ↓
Account
   ↓
Groups / Privilege
```

---

## Credential Reuse

A credential may be useful because it is reused across services.

Conceptually:

```text id="3x6m8q"
Credential Discovered
       ↓
Account Identified
       ↓
Service Identified
       ↓
Authentication
       ↓
Higher Privilege
```

However:

> Credential reuse must be tested only against services within the authorized lab scope.

---

## SSH Keys

Private SSH keys can be sensitive authentication material.

Common locations include:

```bash id="8m4q2x"
ls -la ~/.ssh/
```

Potential files include:

```text id="6x9m3q"
id_rsa
id_ed25519
authorized_keys
config
```

The presence of a private key does not automatically establish that it is usable.

Investigate:

```text id="5q7x2m"
Who owns the key?
Which account uses it?
Is the key protected?
What host/service is it associated with?
Does the authorized lab permit testing it?
```

---

## Application Credentials

Applications frequently use credentials for backend services.

Examples include:

```text id="7m3q8x"
Database
SMTP
LDAP
API
Cloud service
Internal service
```

The investigation should determine:

```text id="2x8m6q"
Credential
   ↓
Service
   ↓
Account
   ↓
Privilege
```

A database credential may provide access to data without providing operating-system privilege.

Do not confuse application privilege with root privilege.

---

## Service Credentials

Service configuration can contain credentials used by background processes.

Investigate relevant configurations:

```text id="9q4m5x"
/etc/
/etc/systemd/
/etc/default/
/etc/<service>/
/opt/<application>/
```

The exact location depends on the service.

Determine:

```text id="6x3m8q"
Which service uses the credential?
Which account does it authenticate as?
What privileges does that account possess?
```

---

## Validation

Before using discovered credentials, confirm:

```text id="5m8q2x"
[ ] Credential source identified
[ ] Credential owner identified
[ ] Account/service identified
[ ] Privilege level understood
[ ] Credential appears relevant
[ ] Target service is in scope
[ ] Authentication method is understood
[ ] Expected privilege impact is understood
```

This avoids treating every string resembling a password as a valid escalation path.

---

## Controlled Authentication

In an authorized lab, use the discovered credential only against the identified in-scope account or service.

Record:

```text id="8x4m6q"
Credential Source:
Account:
Service:
Authentication Method:
Expected Privilege:
Observed Result:
```

Do not attempt broad credential spraying or unrelated authentication attempts.

---

## Exploitation

A credential-based privilege-escalation path typically looks like:

```text id="7q3m9x"
Low-Privilege User
       ↓
Credential Discovery
       ↓
Higher-Privilege Account
       ↓
Authorized Authentication
       ↓
Higher Privilege
```

The credential is the mechanism connecting the two privilege contexts.

---

## Verification

After successful authentication, verify the resulting identity.

For a local shell:

```bash id="6m2x8q"
whoami
id
id -u
```

For another account:

```text id="5x9q3m"
Username:
UID:
Groups:
Privilege:
```

If the intended result is root:

```text id="8m4q7x"
UID = 0
```

Do not assume that successful authentication means root access.

---

## Credential vs Privilege

This distinction is essential.

```text id="3x7m5q"
Credential
   ≠
Root Privilege
```

A discovered credential may provide:

```text id="9m2q6x"
Application access
Database access
Service access
Normal-user access
Administrative access
Root access
```

Determine which one actually applies.

---

## Root Cause

Possible root causes include:

```text id="4x8m2q"
Plaintext credential storage
Insecure configuration permissions
Credential reuse
Exposed backup
Weak secret management
Private key exposure
Secrets stored in scripts
Unnecessary credential persistence
```

The actual root cause depends on the case.

---

## Attack Path

Document the complete chain:

```text id="7m3q9x"
Initial User
     ↓
Readable Credential
     ↓
Credential Owner Identified
     ↓
Privileged Account / Service
     ↓
Authorized Authentication
     ↓
Higher Privilege
     ↓
Verified Identity
```

If the credential only provides application-level access, document that accurately instead of claiming operating-system escalation.

---

## False Positives

Credential findings can fail for many reasons.

Examples:

```text id="5q8m2x"
Credential is expired
Credential is incorrect
Credential belongs to current user
Credential belongs to low-privileged account
Credential is only used by an application
Service is unavailable
Authentication method differs
Credential requires another factor
```

Therefore:

```text id="6x4m9q"
Secret Found
    ↓
Identify Owner
    ↓
Identify Service
    ↓
Determine Privilege
    ↓
Validate
```

---

## Common Credential Locations

Use targeted investigation rather than indiscriminate searching.

```text id="8m5q2x"
User Home Directories
Application Directories
Service Configurations
Web Application Files
Backup Files
Shell History
Environment Variables
SSH Configuration
Database Configuration
```

Prioritize locations associated with services and applications already identified during enumeration.

---

## Common Mistakes

### Treating Every Password-Looking String as Valid

A string may be:

```text id="3x7m8q"
Old
Example
Placeholder
Expired
Unrelated
```

Validate its context.

---

### Ignoring Credential Ownership

Determine who the credential belongs to.

---

### Ignoring Service Context

A credential may authenticate to a database without granting operating-system access.

---

### Using Credentials Outside Scope

Authentication testing must remain within the authorized environment.

---

### Dumping Secrets Unnecessarily

Collect only what is needed to demonstrate the attack path.

Sanitize published evidence.

---

### Forgetting Verification

Always establish the actual resulting identity and privilege.

---

## Evidence

Useful evidence includes:

```text id="7m4x9q"
Credential location
File permissions
Relevant configuration
Credential owner
Target service
Authentication result
Final identity
```

For local account verification:

```bash id="5q8m2x"
id <username>
```

For the final session:

```bash id="8x3m6q"
whoami
id
id -u
```

Never publish real credentials.

Replace them with placeholders such as:

```text id="4m7q9x"
REDACTED_PASSWORD
REDACTED_PRIVATE_KEY
REDACTED_TOKEN
```

---

## Cleanup

Credential-based labs require careful cleanup.

```text id="6x2m8q"
[ ] Test credentials removed where appropriate
[ ] Temporary accounts removed
[ ] Temporary keys removed
[ ] Temporary authentication changes reverted
[ ] Test files removed
[ ] Published evidence sanitized
[ ] Original lab state restored
```

If the credential is intentionally part of the vulnerable baseline, restore it to the documented lab state rather than deleting the vulnerability.

---

## Case Study Checklist

```text id="9m5q3x"
[ ] Initial user identified
[ ] Initial privilege recorded
[ ] Credential source identified
[ ] Credential context understood
[ ] Credential owner identified
[ ] Account/service identified
[ ] Account privilege identified
[ ] Target service confirmed
[ ] Scope confirmed
[ ] Credential validated
[ ] Authentication performed in scope
[ ] Result verified
[ ] Root cause documented
[ ] Evidence sanitized
[ ] Cleanup completed
```

---

## Methodology Mapping

This case study reinforces:

```text id="7x4m2q"
08-CREDENTIALS-AND-SECRETS/
├── command-history.md
├── configuration-secrets.md
├── passwords.md
├── ssh-keys.md
└── credential-reuse.md

13-FINDING-VALIDATION/
├── finding-vs-vulnerability.md
├── exploitability.md
└── privilege-boundary-analysis.md

14-EXPLOITATION/
└── credential-based-escalation.md

15-ROOT-VERIFICATION/
├── verify-privileges.md
└── collect-evidence.md

16-METHODOLOGY/
└── attack-path-analysis.md
```

---

## What to Remember

The memorable rule is:

```text id="5q8m3x"
SECRET FOUND
    ↓
WHO OWNS IT?
    ↓
WHERE IS IT USED?
    ↓
WHAT PRIVILEGE DOES IT PROVIDE?
    ↓
IS THE TARGET IN SCOPE?
    ↓
VALIDATE
    ↓
AUTHENTICATE
    ↓
VERIFY
```

> **Finding a credential is only the beginning. A credential becomes a privilege-escalation path when its owner, usage, privilege, scope, and resulting access are all established.**
