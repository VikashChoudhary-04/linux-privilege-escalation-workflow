# Linux Privilege Escalation Quick Reference

> **Purpose:** Provide a fast operational reference for Linux privilege-escalation assessment without replacing the detailed methodology, validation, or lab documentation.

---

## What This Section Is For

Use `17-QUICK-REFERENCE/` when you already understand the methodology and need a fast reminder of:

* what to check
* which command answers the question
* what to look for
* when to investigate further
* when to move on

This section is designed for:

```text
FAST REFERENCE
     ↓
TARGETED ENUMERATION
     ↓
FINDING
     ↓
VALIDATION
     ↓
EXPLOITATION
     ↓
VERIFICATION
```

---

## Core Rule

> **Do not run a command just because it is listed here. Run it because you have a question that the command can answer.**

The most important questions remain:

```text
WHO AM I?
WHAT IS PRIVILEGED?
WHO OWNS IT?
WHO CAN MODIFY IT?
WHAT DOES IT TRUST?
WHEN DOES IT EXECUTE?
CAN I INFLUENCE IT?
```

---

## Quick Workflow

```text
1. Establish identity
        ↓
2. Establish system context
        ↓
3. Check sudo and groups
        ↓
4. Check privileged processes
        ↓
5. Check SUID / SGID
        ↓
6. Check capabilities
        ↓
7. Check cron / timers
        ↓
8. Check writable resources
        ↓
9. Check credentials
        ↓
10. Check network / special environments
        ↓
11. Run automated enumeration
        ↓
12. Correlate findings
        ↓
13. Validate
        ↓
14. Exploit within scope
        ↓
15. Verify
        ↓
16. Document and clean up
```

---

## First Commands

### Identity

```bash
whoami
id
id -u
id -g
```

### System

```bash
hostname
uname -a
cat /etc/os-release
uname -m
```

### Groups

```bash
groups
```

### Environment

```bash
echo "$PATH"
env
```

---

## Privilege Checks

### Sudo

```bash
sudo -l
```

Look for:

```text
Commands allowed as root
Commands allowed as another user
Dangerous interpreters
File manipulation utilities
Environment-related permissions
Unexpected command arguments
```

---

## SUID

```bash
find / -perm -4000 -type f 2>/dev/null
```

Ask:

```text
What binary?
Who owns it?
Why is it SUID?
Can I execute it?
Can I control its behavior?
```

---

## SGID

```bash
find / -perm -2000 -type f 2>/dev/null
```

Ask:

```text
Which group?
What resources does the group control?
Can the current user influence execution?
```

---

## Capabilities

```bash
getcap -r / 2>/dev/null
```

For an interesting binary:

```bash
getcap <binary>
```

Ask:

```text
Which capability?
What does it permit?
Can I execute the binary?
Can I control its behavior?
```

---

## Processes

```bash
ps aux
```

More focused:

```bash
ps -ef
```

Look for:

```text
Root processes
Custom applications
Scripts
Unexpected services
Interesting command-line arguments
```

---

## Root Processes

A process running as root is not automatically vulnerable.

Ask:

```text
What executable?
What arguments?
What configuration?
What files?
What dependencies?
Who can modify those resources?
```

---

## Services

List running services:

```bash
systemctl list-units --type=service --state=running
```

Inspect one:

```bash
systemctl status <service>
systemctl cat <service>
```

Ask:

```text
Who runs it?
What does it execute?
What does it trust?
Who can modify its dependencies?
```

---

## Cron

Inspect:

```bash
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
```

Ask:

```text
Who executes the task?
What does it execute?
Can the current user modify it?
When does it execute?
```

---

## Systemd Timers

```bash
systemctl list-timers --all
```

Inspect:

```bash
systemctl cat <timer>
```

Then inspect the associated service.

---

## Writable Files

For a known file:

```bash
ls -la <file>
stat <file>
```

Ask:

```text
Who owns it?
Who can write it?
Who consumes it?
Is the consumer privileged?
```

---

## Writable Directories

Inspect:

```bash
ls -ld <directory>
```

Ask:

```text
Who can write here?
Who creates files here?
Who reads files here?
Who executes files here?
Does a privileged process trust this directory?
```

---

## Interesting Files

Common areas to inspect:

```text
/etc/
/opt/
/usr/local/
/var/
/home/
/tmp/
```

Look for:

```text
Configuration
Scripts
Backups
Credentials
Custom applications
Logs
Keys
```

Do not recursively inspect everything without a question.

---

## Credentials

Potential sources:

```text
Shell history
Configuration files
Application files
Environment variables
Backups
SSH material
Scripts
Logs
```

Useful commands:

```bash
history 2>/dev/null
env
```

For discovered credentials, determine:

```text
Who owns them?
Where are they used?
What privilege do they provide?
Is their use authorized?
```

---

## SSH

Check user SSH material where authorized:

```bash
ls -la ~/.ssh/
```

Look for:

```text
authorized_keys
config
known_hosts
private keys
```

Do not expose private key contents unnecessarily.

---

## PATH

Check:

```bash
echo "$PATH"
```

For a privileged command, determine:

```text
Does it use absolute paths?
Does it trust PATH?
Can the current user influence a searched directory?
```

---

## Environment Variables

Check:

```bash
env
```

Ask:

```text
Which variables affect privileged execution?
Who controls them?
Are they preserved?
Does the privileged program trust them?
```

---

## Wildcards

If a privileged command uses a wildcard, determine:

```text
What command?
Which directory?
Who controls the directory?
How are matched filenames interpreted?
Can filenames affect command behavior?
```

Do not assume wildcard use is automatically exploitable.

---

## Command Substitution

For scripts or commands containing dynamic input, ask:

```text
What controls the input?
How is it processed?
Is a shell involved?
Does the resulting command run with higher privilege?
```

---

## Services and Scripts

Trace the complete chain:

```text
Service
 ↓
Executable
 ↓
Arguments
 ↓
Configuration
 ↓
Scripts
 ↓
Libraries / dependencies
 ↓
Files / directories
```

Then determine which components the current user can influence.

---

## Network

### Interfaces

```bash
ip addr
```

### Routes

```bash
ip route
```

### Listening Services

```bash
ss -lntup
```

Ask:

```text
What service?
Who owns it?
Is it local-only?
Is it privileged?
Does it expose a useful trust relationship?
```

---

## Localhost Services

A service listening only on localhost may still matter.

Investigate:

```text
Application
Authentication
Privilege
Input
Trust relationship
```

Example reasoning:

```text
Current User
    ↓
Can access localhost service
    ↓
Service runs with higher privilege
    ↓
User-controlled input
    ↓
Privileged action
```

---

## Mounts

```bash
findmnt
```

Look for:

```text
NFS
Unusual mounts
Writable filesystems
Container mounts
Host filesystem exposure
```

---

## Docker

Check:

```bash
id
docker ps
```

Socket:

```bash
ls -l /var/run/docker.sock 2>/dev/null
```

Ask:

```text
Can the current user control the runtime?
What host resources are exposed?
Does the runtime access cross the host privilege boundary?
```

---

## LXD / Containers

Determine:

```text
Is the runtime installed?
Is the current user authorized to interact with it?
What privileges does the runtime provide?
Can it affect host resources?
```

Container presence alone is not enough.

---

## NFS

Determine:

```text
What is exported?
Who can access it?
What export options exist?
Can the current user influence files used by privileged processes?
```

---

## Special Groups

Run:

```bash
id
```

Pay attention to unusual memberships.

Examples include:

```text
docker
lxd
disk
libvirt
```

Then determine exactly what access the group grants on the target system.

---

## Vulnerable Software

Collect:

```bash
uname -a
cat /etc/os-release
```

Then establish:

```text
Exact version
Distribution patch status
Architecture
Configuration
Exposure
Exploit prerequisites
Privilege impact
```

Never treat an old version alone as proof of exploitability.

---

## Automated Enumeration

Useful tools include:

```text
LinPEAS
Linux Smart Enumeration
pspy
```

Workflow:

```text
Manual baseline
      ↓
Automated enumeration
      ↓
Interesting findings
      ↓
Manual reproduction
      ↓
Validation
```

---

## Finding Triage

Prioritize:

```text
1. Sudo
2. Root services
3. Root processes
4. Writable root-executed resources
5. SUID / SGID
6. Capabilities
7. Cron / timers
8. Credentials
9. Privileged groups
10. Containers
11. Network trust
12. Vulnerable software
```

This is a practical starting order, not a universal ranking.

A newly discovered complete attack path should immediately receive attention.

---

## Finding → Attack Path

Use:

```text
Finding
   ↓
Privileged component?
   ↓
User control?
   ↓
Trust relationship?
   ↓
Execution?
   ↓
Privilege impact?
   ↓
Validation
```

If a branch fails:

```text
Move on.
```

---

## Validation Gate

Before exploitation:

```text
[ ] Finding reproduced
[ ] Current privilege confirmed
[ ] Privileged component confirmed
[ ] User control confirmed
[ ] Trust relationship understood
[ ] Execution path confirmed
[ ] Preconditions satisfied
[ ] Expected impact understood
[ ] Exploitation authorized
```

---

## Exploitation Gate

Use the smallest effective action.

Before proceeding:

```text
[ ] Scope permits exploitation
[ ] Finding is validated
[ ] Expected privilege is known
[ ] Impact is understood
[ ] Verification is ready
[ ] Cleanup is understood
```

---

## Root Verification

After exploitation:

```bash
whoami
id
id -u
```

Expected elevated context should be verified rather than assumed.

For root:

```text
UID = 0
```

---

## Evidence

Capture enough evidence to prove:

```text
Initial user
Initial privilege
Finding
Control
Trust
Execution
Privilege change
Final user
Final privilege
```

Useful commands:

```bash
whoami
id
id -u
ls -la <resource>
stat <resource>
sudo -l
systemctl cat <service>
```

---

## Cleanup

After authorized testing:

```text
Remove temporary files
Stop temporary processes
Restore temporary configuration changes
Remove temporary accounts or keys
Remove temporary services
Preserve required evidence
```

Never destroy evidence required for reporting.

---

## Common Mistakes

### Running Commands Without a Question

Every command should answer something.

### Treating Findings as Vulnerabilities

A finding is an observation.

### Treating Vulnerabilities as Attack Paths

A vulnerability may not be reachable or applicable.

### Skipping Validation

Never assume the final result.

### Ignoring Groups

Group membership can materially change access.

### Ignoring Timing

Scheduled execution depends on when and how a task runs.

### Tunnel Vision

If one path stalls, return to the candidate list.

---

## Quick Decision Tree

```text
LOW-PRIVILEGED SHELL
        ↓
WHO AM I?
        ↓
WHAT IS PRIVILEGED?
        ↓
CAN I CONTROL IT?
        ↓
WHAT DOES IT TRUST?
        ↓
WHEN DOES IT EXECUTE?
        ↓
CAN CONTROL REACH PRIVILEGE?
        │
    ┌───┴───┐
   NO      YES
    │        │
 MOVE ON   VALIDATE
             │
             ▼
          EXPLOIT
             │
             ▼
          VERIFY
             │
             ▼
         DOCUMENT
```

---

## Quick Checklist

```text
IDENTITY
[ ] whoami
[ ] id
[ ] groups

SYSTEM
[ ] hostname
[ ] uname -a
[ ] /etc/os-release
[ ] architecture

PRIVILEGE
[ ] sudo -l
[ ] SUID
[ ] SGID
[ ] capabilities
[ ] privileged groups

EXECUTION
[ ] processes
[ ] services
[ ] cron
[ ] timers
[ ] scripts

FILESYSTEM
[ ] permissions
[ ] writable files
[ ] writable directories
[ ] configurations
[ ] backups
[ ] mounts

SECRETS
[ ] history
[ ] environment
[ ] credentials
[ ] SSH material
[ ] application secrets

NETWORK
[ ] interfaces
[ ] routes
[ ] listening services
[ ] localhost services

SPECIAL ENVIRONMENTS
[ ] Docker
[ ] LXD
[ ] containers
[ ] NFS
[ ] special mounts

VALIDATION
[ ] finding reproduced
[ ] privilege relationship proven
[ ] user control proven
[ ] execution proven
[ ] impact understood

FINAL
[ ] exploit
[ ] verify
[ ] evidence
[ ] document
[ ] cleanup
```

---

## Where to Go Next

Use the other files in this directory for focused workflows:

```text
first-5-minutes.md
    ↓
Fast initial enumeration

15-minute-enumeration.md
    ↓
Structured short assessment

command-reference.md
    ↓
Command lookup

decision-tree.md
    ↓
"What should I check next?"

final-checklist.md
    ↓
Final assessment verification
```

For detailed reasoning, return to:

```text
00-FOUNDATIONS/
16-METHODOLOGY/
```

For practical application:

```text
18-LABS/
19-CASE-STUDIES/
```

---

## Final Mental Model

Keep this sequence in memory:

```text
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
```

And remember:

> **The quick reference tells you what to check. The methodology tells you why. The validation process tells you whether it actually works.**
