# Linux Privilege Escalation Final Checklist

> **Purpose:** Provide a final end-to-end checklist for reviewing enumeration, validation, exploitation, verification, evidence, documentation, and cleanup during an authorized Linux privilege-escalation assessment.

---

## How to Use This Checklist

Use this file at the end of an assessment or before closing a lab.

It answers:

```text id="7m4x2q"
Did I understand the system?
Did I enumerate the important privilege mechanisms?
Did I identify a real attack path?
Did I validate it?
Did I verify the result?
Did I document what happened?
Did I clean up?
```

Do not mark an item complete merely because a command was executed.

Mark it complete when the underlying question has been answered.

---

## Phase 1 — Authorization and Scope

```text id="8q3m6x"
[ ] Target is authorized
[ ] Scope is understood
[ ] Testing boundaries are known
[ ] Exploitation is permitted where required
[ ] Destructive actions are prohibited unless explicitly authorized
[ ] Persistence requirements are understood
[ ] Evidence requirements are understood
```

---

## Phase 2 — Initial Access Context

```text id="5x7m9p"
[ ] Initial access method understood
[ ] Current shell identified
[ ] Current user identified
[ ] Initial privilege recorded
[ ] Target hostname recorded
[ ] Initial evidence preserved
```

Run:

```bash id="2m8q4v"
whoami
id
hostname
```

---

## Phase 3 — System Context

```text id="6p3x8m"
[ ] Operating system identified
[ ] OS version identified
[ ] Kernel identified
[ ] Architecture identified
[ ] Relevant package ecosystem identified
[ ] Virtualization/container context considered
```

Run:

```bash id="9m4q7x"
uname -a
cat /etc/os-release
uname -m
```

---

## Phase 4 — Users and Groups

```text id="3x6m9q"
[ ] Current UID identified
[ ] Current GID identified
[ ] Supplementary groups identified
[ ] Interesting privileged groups investigated
[ ] UID 0 accounts reviewed where relevant
[ ] Service accounts considered
```

Run:

```bash id="7q2m5x"
id
groups
awk -F: '$3 == 0 {print $1}' /etc/passwd
```

Do not assume a group is privileged without determining what access it actually grants.

---

## Phase 5 — Sudo

```text id="8m5x3q"
[ ] `sudo -l` checked
[ ] Allowed commands recorded
[ ] Target users identified
[ ] Restrictions understood
[ ] Environment behavior considered
[ ] Allowed programs investigated
[ ] Sudo candidate validated
```

Run:

```bash id="4p7m2x"
sudo -l
```

---

## Phase 6 — Processes

```text id="9x3m6q"
[ ] Running processes reviewed
[ ] Root processes identified
[ ] Custom processes identified
[ ] Interesting command-line arguments reviewed
[ ] Parent-child relationships considered
[ ] Process dependencies investigated
[ ] User control over relevant resources checked
```

Run:

```bash id="5m8q2x"
ps aux
ps auxf
```

---

## Phase 7 — Services

```text id="7x4m9p"
[ ] Running services identified
[ ] Root-running services identified
[ ] Custom services investigated
[ ] Service definitions reviewed
[ ] Executables identified
[ ] Scripts identified
[ ] Configuration identified
[ ] Dependencies identified
[ ] Service permissions checked
```

Run:

```bash id="2m6x8q"
systemctl list-units --type=service --state=running
systemctl status <service>
systemctl cat <service>
```

---

## Phase 8 — SUID and SGID

### SUID

```text id="6q3m7x"
[ ] SUID files enumerated
[ ] Standard binaries separated from unusual binaries
[ ] Ownership checked
[ ] Permissions checked
[ ] Purpose understood
[ ] User control investigated
[ ] Privileged behavior validated
```

Run:

```bash id="8m4x2p"
find / -perm -4000 -type f 2>/dev/null
```

### SGID

```text id="5x9m3q"
[ ] SGID files enumerated
[ ] Group ownership understood
[ ] Group-level privilege identified
[ ] User control investigated
```

Run:

```bash id="3m7q8x"
find / -perm -2000 -type f 2>/dev/null
```

---

## Phase 9 — Capabilities

```text id="7p4m2x"
[ ] Capabilities enumerated
[ ] Interesting binaries identified
[ ] Capability purpose understood
[ ] Binary behavior understood
[ ] User execution confirmed
[ ] User control confirmed
[ ] Privilege impact validated
```

Run:

```bash id="9x5m6q"
getcap -r / 2>/dev/null
```

---

## Phase 10 — Scheduled Execution

### Cron

```text id="4m8q2x"
[ ] `/etc/crontab` reviewed
[ ] `/etc/cron.d/` reviewed
[ ] Periodic cron directories reviewed
[ ] Root-executed jobs identified
[ ] Referenced scripts identified
[ ] Script ownership checked
[ ] Script permissions checked
[ ] Execution timing understood
```

Run:

```bash id="6x3m9p"
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
```

### Systemd Timers

```text id="8m5q3x"
[ ] Timers enumerated
[ ] Interesting timers identified
[ ] Associated services identified
[ ] Execution user identified
[ ] Executed resources investigated
```

Run:

```bash id="2q7m4x"
systemctl list-timers --all
```

---

## Phase 11 — Filesystem

```text id="5x8m2q"
[ ] Root filesystem reviewed
[ ] `/opt` reviewed
[ ] `/usr/local` reviewed
[ ] Relevant `/var` paths reviewed
[ ] User home directories considered
[ ] Temporary directories considered
[ ] Writable files investigated
[ ] Writable directories investigated
[ ] Custom scripts identified
[ ] Configuration files identified
[ ] Backups identified
[ ] Mount points reviewed
```

Useful commands:

```bash id="7m3x9p"
ls -la /
ls -la /opt/
ls -la /usr/local/
findmnt
```

For candidates:

```bash id="9q4m6x"
ls -la <path>
stat <path>
file <path>
```

---

## Phase 12 — Writable Resource Analysis

For every interesting writable resource:

```text id="3x7m5q"
[ ] Owner identified
[ ] Group identified
[ ] Permissions identified
[ ] Privileged consumer identified
[ ] Consumer execution context identified
[ ] Trigger identified
[ ] User influence proven
[ ] Privilege impact understood
```

Remember:

```text id="8m2q4x"
Writable
   ≠
Vulnerable
```

The consumer matters.

---

## Phase 13 — Credentials and Secrets

```text id="6p9m3x"
[ ] History reviewed
[ ] Environment reviewed
[ ] Application configuration reviewed
[ ] Service configuration reviewed
[ ] Backup files considered
[ ] SSH material considered
[ ] Relevant credentials identified
[ ] Credential owner identified
[ ] Credential privilege identified
[ ] Authorized use considered
```

Potential sources:

```text id="4x8m2q"
/home/
/opt/
/etc/
/var/
/usr/local/
```

Do not unnecessarily copy or expose secrets.

---

## Phase 14 — PATH and Execution Abuse

```text id="7m5q9x"
[ ] PATH reviewed
[ ] Privileged commands using PATH identified
[ ] User-controlled PATH directories identified
[ ] Absolute vs relative command resolution understood
[ ] Wildcards investigated where relevant
[ ] Environment dependencies investigated
[ ] Library dependencies investigated
[ ] Command substitution investigated where relevant
```

Run:

```bash id="2x6m8q"
echo "$PATH"
env
```

Only continue when a privileged execution relationship exists.

---

## Phase 15 — Network

```text id="5m3q7x"
[ ] Interfaces identified
[ ] Routes identified
[ ] Listening services identified
[ ] Localhost services identified
[ ] Custom services identified
[ ] Privileged services investigated
[ ] Network trust relationships considered
```

Run:

```bash id="8x4m9p"
ip addr
ip route
ss -lntup
```

---

## Phase 16 — Special Environments

### Containers

```text id="6m8q2x"
[ ] Container runtime identified
[ ] Current user access checked
[ ] Runtime permissions understood
[ ] Host interaction considered
[ ] Privilege boundary analyzed
```

### Docker

```bash id="3x7m5q"
docker ps
ls -l /var/run/docker.sock 2>/dev/null
```

### Mounts

```bash id="9m4x6q"
findmnt
```

### NFS

```text id="5q8m2x"
[ ] Exports identified
[ ] Access restrictions understood
[ ] Export options reviewed
[ ] Privileged resource relationships investigated
```

---

## Phase 17 — Vulnerable Software

```text id="7x3m9q"
[ ] Kernel version identified
[ ] OS version identified
[ ] Architecture identified
[ ] Software version identified
[ ] Distribution patches considered
[ ] Configuration considered
[ ] Exposure confirmed
[ ] Prerequisites checked
[ ] Privilege impact understood
```

Never conclude:

```text id="2m6q8x"
Old version
=
Automatically exploitable
```

---

## Phase 18 — Automated Enumeration

```text id="8m4x7p"
[ ] Manual baseline completed
[ ] Automated enumeration performed where useful
[ ] Tool output triaged
[ ] High-value findings extracted
[ ] Findings manually reproduced
[ ] False positives rejected
[ ] Attack paths constructed
```

Typical tools:

```text id="5x9m3q"
LinPEAS
Linux Smart Enumeration
pspy
```

The rule:

> **Automation accelerates discovery; it does not replace validation.**

---

## Phase 19 — Finding Classification

For every finding, classify it:

```text id="3q7m8x"
FINDING
   ↓
VULNERABILITY?
   ↓
ATTACK PATH?
   ↓
EXPLOITABLE?
```

Do not collapse these into one conclusion.

---

## Phase 20 — Attack Path Analysis

For the strongest candidate:

```text id="6m2x9q"
[ ] Initial user identified
[ ] Current privilege identified
[ ] Privileged component identified
[ ] User-controlled resource identified
[ ] Ownership identified
[ ] Permissions identified
[ ] Trust relationship identified
[ ] Execution trigger identified
[ ] Execution context identified
[ ] Privilege boundary identified
[ ] Expected impact identified
```

Construct:

```text id="8x5m3p"
CURRENT USER
      ↓
CONTROL
      ↓
TRUSTED COMPONENT
      ↓
PRIVILEGED EXECUTION
      ↓
PRIVILEGE IMPACT
```

Every arrow should have evidence.

---

## Phase 21 — Validation

Before exploitation:

```text id="4m7q2x"
[ ] Finding reproduced
[ ] User control proven
[ ] Privileged execution proven
[ ] Trust relationship proven
[ ] Preconditions satisfied
[ ] Expected result understood
[ ] False-positive possibility considered
[ ] Validation performed with minimal impact
```

If validation fails:

```text id="7x3m8q"
Identify failed relationship
      ↓
Understand why
      ↓
Reject or revise candidate
```

Do not force the path.

---

## Phase 22 — Exploitation

Before controlled exploitation:

```text id="5q9m4x"
[ ] Authorization confirmed
[ ] Scope permits exploitation
[ ] Finding validated
[ ] Preconditions satisfied
[ ] Expected privilege known
[ ] Minimal effective action selected
[ ] Verification method prepared
[ ] Cleanup plan understood
```

During exploitation:

```text id="8m2x6p"
[ ] Minimize changes
[ ] Avoid unnecessary persistence
[ ] Avoid destructive actions
[ ] Preserve evidence
[ ] Stop when objective is achieved
```

---

## Phase 23 — Root / Privilege Verification

Run:

```bash id="3x7m9q"
whoami
id
id -u
```

For root:

```text id="6m4q8x"
UID = 0
```

Verify the actual privilege context.

Do not treat:

```text id="2m8x5q"
Successful command
```

as equivalent to:

```text id="7q3m9x"
Verified privilege escalation
```

---

## Phase 24 — Evidence

Record:

```text id="5x7m2q"
[ ] Initial user
[ ] Initial UID
[ ] Initial groups
[ ] Finding
[ ] Privileged component
[ ] User control
[ ] Trust relationship
[ ] Execution path
[ ] Validation evidence
[ ] Exploitation evidence
[ ] Final user
[ ] Final UID
```

Preserve only evidence needed to demonstrate the result.

---

## Phase 25 — Document the Attack Path

Write the path in plain language:

```text id="8m3q6x"
The initial user could control [RESOURCE].
[PRIVILEGED COMPONENT] trusted or executed that resource
under [PRIVILEGE].
This allowed the controlled input to reach [PRIVILEGED ACTION].
The resulting privilege change was verified using [EVIDENCE].
```

Separate:

```text id="4x7m9p"
Root Cause
   ↓
Finding
   ↓
Attack Path
   ↓
Exploit
   ↓
Impact
```

---

## Phase 26 — Cleanup

```text id="6m2q8x"
[ ] Temporary files removed
[ ] Temporary processes stopped
[ ] Temporary services removed
[ ] Temporary configuration changes reverted
[ ] Temporary credentials/keys removed
[ ] Temporary accounts removed
[ ] Test artifacts removed where authorized
[ ] Required evidence preserved
```

Do not remove legitimate system artifacts unnecessarily.

---

## Phase 27 — Final Verification

After cleanup:

```text id="9x4m7q"
[ ] Original system state restored where required
[ ] No unintended persistence remains
[ ] Temporary processes gone
[ ] Temporary files gone
[ ] Temporary accounts/keys gone
[ ] Evidence remains available
[ ] Final privilege state documented
```

---

## Phase 28 — Assessment Closure

Before closing:

```text id="5m8q2x"
[ ] Attack path is reproducible
[ ] Root cause is understood
[ ] Exploitation result is verified
[ ] Evidence is sufficient
[ ] Cleanup is complete
[ ] Remaining findings are documented
[ ] Unvalidated candidates are clearly marked
[ ] Scope limitations are recorded
```

---

## Complete End-to-End Checklist

```text id="7x3m9p"
AUTHORIZATION
[ ] Scope
[ ] Rules
[ ] Constraints

IDENTITY
[ ] whoami
[ ] id
[ ] UID
[ ] Groups

SYSTEM
[ ] Hostname
[ ] OS
[ ] Kernel
[ ] Architecture

PRIVILEGE
[ ] sudo
[ ] SUID
[ ] SGID
[ ] Capabilities
[ ] Special groups

EXECUTION
[ ] Processes
[ ] Services
[ ] Cron
[ ] Timers

FILESYSTEM
[ ] Permissions
[ ] Writable files
[ ] Writable directories
[ ] Configurations
[ ] Backups
[ ] Mounts

CREDENTIALS
[ ] History
[ ] Environment
[ ] Application secrets
[ ] SSH material

EXECUTION ABUSE
[ ] PATH
[ ] Wildcards
[ ] Environment
[ ] Libraries
[ ] Command substitution

NETWORK
[ ] Interfaces
[ ] Routes
[ ] Listening services
[ ] Localhost services

SPECIAL ENVIRONMENTS
[ ] Docker
[ ] LXD
[ ] Containers
[ ] NFS
[ ] Special mounts

SOFTWARE
[ ] Versions
[ ] Patches
[ ] Configuration
[ ] Applicability

AUTOMATION
[ ] LinPEAS
[ ] LSE
[ ] pspy
[ ] Manual validation

ATTACK PATH
[ ] Privileged component
[ ] User control
[ ] Trust
[ ] Execution
[ ] Impact

VALIDATION
[ ] Reproduced
[ ] Preconditions
[ ] Safe validation
[ ] Exploitability

EXPLOITATION
[ ] Authorized
[ ] Minimal action
[ ] Expected result

VERIFICATION
[ ] whoami
[ ] id
[ ] UID
[ ] Evidence

DOCUMENTATION
[ ] Root cause
[ ] Attack path
[ ] Exploit
[ ] Impact
[ ] Evidence

CLEANUP
[ ] Temporary artifacts
[ ] Processes
[ ] Services
[ ] Configuration
[ ] Credentials
[ ] Final verification
```

---

## Final Assessment Questions

Before declaring the assessment complete, answer:

### Question 1

> **Who was the initial user?**

```text id="4m7x2q"
____________________________
```

### Question 2

> **What was the initial privilege level?**

```text id="8x3m6q"
____________________________
```

### Question 3

> **What privileged component was involved?**

```text id="5q9m4x"
____________________________
```

### Question 4

> **What could the user control?**

```text id="7m2x8p"
____________________________
```

### Question 5

> **What did the privileged component trust?**

```text id="3x6q9m"
____________________________
```

### Question 6

> **How did execution reach the privilege boundary?**

```text id="9m4x7q"
____________________________
```

### Question 7

> **How was privilege escalation verified?**

```text id="6q8m2x"
____________________________
```

### Question 8

> **What was the root cause?**

```text id="2x5m9p"
____________________________
```

---

## Final Mental Model

The complete assessment can be reduced to:

```text id="7m3q8x"
SCOPE
 ↓
IDENTIFY
 ↓
ENUMERATE
 ↓
PRIORITIZE
 ↓
CONNECT
 ↓
VALIDATE
 ↓
EXPLOIT
 ↓
VERIFY
 ↓
DOCUMENT
 ↓
CLEAN UP
```

And the most important reasoning loop is:

```text id="5x9m2q"
WHAT IS PRIVILEGED?
        ↓
WHO CONTROLS IT?
        ↓
WHAT DOES IT TRUST?
        ↓
WHEN DOES IT EXECUTE?
        ↓
CAN MY CONTROL REACH IT?
        ↓
CAN I PROVE IT?
```

> **A completed privilege-escalation assessment is not defined by how many commands were executed. It is defined by whether the privilege boundary, attack path, result, evidence, and root cause are understood and documented.**
