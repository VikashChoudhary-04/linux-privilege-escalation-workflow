# Complete Linux Privilege Escalation Workflow

> **Purpose:** Provide one complete, repeatable workflow for moving from an initial low-privileged Linux shell through enumeration, finding analysis, validation, exploitation, verification, and documentation.

---

## Core Workflow

The complete workflow is:

```text
INITIAL SHELL
     ↓
ESTABLISH CONTEXT
     ↓
IDENTIFY CURRENT PRIVILEGE
     ↓
ENUMERATE SYSTEM
     ↓
ENUMERATE FILESYSTEM
     ↓
ENUMERATE PROCESSES & SERVICES
     ↓
ENUMERATE NETWORK
     ↓
ENUMERATE PRIVILEGE MECHANISMS
     ↓
ENUMERATE SCHEDULED EXECUTION
     ↓
SEARCH CREDENTIALS & SECRETS
     ↓
ANALYZE EXECUTION ABUSE
     ↓
CHECK SPECIAL ENVIRONMENTS
     ↓
CHECK VULNERABLE SOFTWARE
     ↓
AUTOMATE ENUMERATION
     ↓
PRIORITIZE FINDINGS
     ↓
VALIDATE ATTACK PATH
     ↓
EXPLOIT
     ↓
VERIFY PRIVILEGE
     ↓
DOCUMENT
     ↓
CLEAN UP IF AUTHORIZED
```

The important principle is:

> **Do not execute the entire workflow mechanically. Use each result to decide what to investigate next.**

---

## Phase 0 — Confirm Authorization

Before performing privilege-escalation testing, confirm that the system is within your authorized scope.

Verify:

```text
[ ] Target is authorized
[ ] Testing is permitted
[ ] Privilege escalation is permitted
[ ] Exploitation is permitted
[ ] Scope limitations are understood
[ ] Destructive actions are prohibited unless explicitly authorized
```

For personal labs and training machines, this means using systems you control or are explicitly permitted to test.

---

## Phase 1 — Establish Context

### Objective

Determine:

> **Where am I, who am I, and what kind of Linux environment am I dealing with?**

Start with:

```bash
whoami
id
hostname
uname -a
cat /etc/os-release
```

Then inspect the architecture:

```bash
uname -m
```

Record:

```text
Username
UID
Groups
Hostname
Distribution
Kernel
Architecture
```

---

## Phase 2 — Establish the Baseline

Before changing anything, record the current privilege state.

Run:

```bash
whoami
id
id -u
id -g
```

Create a mental baseline:

```text
Current user:
Current UID:
Current GID:
Current groups:
Expected target privilege:
```

For a typical root-escalation lab:

```text
Current:
UID 1000+

Target:
UID 0
```

The exact values depend on the environment.

---

## Phase 3 — Inspect the Environment

### Objective

Understand the machine before searching for vulnerabilities.

Check:

```bash
pwd
echo "$SHELL"
echo "$PATH"
env
```

Then inspect important directories:

```bash
ls -la
ls -la /tmp
ls -la /var/tmp
```

Ask:

```text
What applications are present?
What directories are interesting?
What environment variables matter?
What does PATH contain?
```

---

## Phase 4 — System Enumeration

### Objective

Determine:

> **What operating system and software environment am I working with?**

Collect:

```bash
uname -a
cat /etc/os-release
uname -m
```

Then inspect:

```text
Kernel
Distribution
Architecture
Installed software
Configuration
Users
Groups
Environment
```

The result should be a basic system profile.

---

## Phase 5 — User and Group Enumeration

### Objective

Determine:

> **What identities and trust relationships exist on the system?**

Inspect users:

```bash
cat /etc/passwd
```

Inspect groups:

```bash
cat /etc/group
```

Check your current groups:

```bash
id
```

Look for membership in security-relevant groups.

Examples may include groups associated with:

```text
Docker
Disk access
Administration
Virtualization
Device access
Special services
```

Group membership is a finding.

Determine what the membership actually permits before treating it as an escalation path.

---

## Phase 6 — Sudo Enumeration

### Objective

Determine:

> **Can the current user execute anything with higher privilege through sudo?**

Run:

```bash
sudo -l
```

Analyze:

```text
Allowed command
Run-as identity
Arguments
Restrictions
Environment handling
Password requirements
```

If nothing useful is permitted:

```text
Move on.
```

If a potentially dangerous permission exists:

```text
Investigate
    ↓
Validate
    ↓
Exploit only after validation
```

---

## Phase 7 — Process Enumeration

### Objective

Determine:

> **What is currently running, and which processes have higher privileges?**

Run:

```bash
ps aux
```

or:

```bash
ps -ef
```

Look for:

```text
Root-owned processes
Custom applications
Unusual scripts
Security-sensitive services
Scheduled execution
Custom binaries
```

For a suspicious process, ask:

```text
Who owns it?
What does it execute?
What files does it use?
Can I influence any part of its execution?
```

---

## Phase 8 — Service Enumeration

### Objective

Determine:

> **Which services execute with elevated privileges?**

On systemd systems:

```bash
systemctl list-units --type=service --state=running
```

Inspect interesting services:

```bash
systemctl status <service>
systemctl cat <service>
```

Look for:

```text
Root services
Custom services
Writable executables
Writable configuration
Scripts
Unsafe paths
User-controlled dependencies
```

Do not restart a service merely because it looks interesting.

First understand its execution path.

---

## Phase 9 — Filesystem Enumeration

### Objective

Determine:

> **Can the current user modify something that a privileged process later consumes?**

Start with permissions:

```bash
ls -la
```

Search for writable files where appropriate:

```bash
find / -writable -type f 2>/dev/null
```

Search for writable directories:

```bash
find / -writable -type d 2>/dev/null
```

These commands can generate substantial output.

Prioritize files associated with:

```text
Root processes
Services
Scheduled tasks
Configuration
Executables
Scripts
Applications
```

---

## Phase 10 — SUID Enumeration

### Objective

Determine:

> **Which programs can execute with elevated effective privileges?**

Run:

```bash
find / -perm -4000 -type f 2>/dev/null
```

For each unusual binary:

```text
Identify owner
        ↓
Identify purpose
        ↓
Understand behavior
        ↓
Identify user-controlled input
        ↓
Determine privileged impact
```

Remember:

```text
SUID file
≠
Automatic vulnerability
```

---

## Phase 11 — Capability Enumeration

### Objective

Determine:

> **Do privileged capabilities provide a controllable path to higher privilege?**

Run:

```bash
getcap -r / 2>/dev/null
```

For an interesting binary:

```text
Capability
    ↓
Security effect
    ↓
Program behavior
    ↓
User control
    ↓
Privilege impact
```

Do not treat the existence of a capability as proof of exploitation.

---

## Phase 12 — Scheduled Execution

### Objective

Determine:

> **What executes automatically with elevated privileges?**

Inspect:

```bash
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
```

Also investigate systemd timers:

```bash
systemctl list-timers --all
```

For each relevant task:

```text
Who executes it?
What does it execute?
Where is the executable?
Who owns it?
Who can modify it?
When does it run?
```

---

## Phase 13 — Dynamic Scheduled Execution

Static enumeration shows what is configured.

Dynamic monitoring can reveal what actually executes.

In an authorized lab, tools such as `pspy` can help observe process activity without requiring modification of the target process.

The reasoning is:

```text
Static configuration
       +
Observed execution
       ↓
Confirmed execution relationship
```

This is especially useful when a scheduled process is difficult to identify from configuration alone.

---

## Phase 14 — Credential and Secret Enumeration

### Objective

Determine:

> **Can an exposed credential provide access to a more privileged identity or resource?**

Potential sources:

```text
Command history
Configuration files
Application files
Environment variables
SSH files
Private keys
Database configuration
Service configuration
Backups
Logs
Temporary files
```

Search deliberately.

Do not immediately dump or expose every potentially sensitive value.

For each discovery ask:

```text
Who owns the credential?
What service uses it?
Is it valid?
What privilege does the target account have?
Is reuse authorized to test?
```

---

## Phase 15 — Execution Abuse Analysis

### Objective

Determine:

> **Does a privileged process trust something that the current user can influence?**

Investigate:

```text
PATH
Relative paths
Writable scripts
Command substitution
Wildcards
Library loading
Environment variables
Shell functions
Configuration
Temporary files
Dependencies
```

Use the core pattern:

```text
Privileged execution
        +
User-controlled input
        +
Trusted relationship
        =
Attack-path candidate
```

Then validate.

---

## Phase 16 — Network Enumeration

### Objective

Determine:

> **Are there local or network services that change the privilege-escalation attack surface?**

Inspect interfaces:

```bash
ip addr
```

Routes:

```bash
ip route
```

Listening services:

```bash
ss -lntup
```

Ask:

```text
What is listening?
Is it local-only?
Is it privileged?
Is authentication required?
Can the current user interact with it?
```

Network exposure alone does not prove privilege escalation.

---

## Phase 17 — Special Environment Analysis

### Objective

Determine:

> **Does the environment itself provide an unusual privilege boundary?**

Investigate:

```text
Docker
LXD
Containers
NFS
Mounted filesystems
Privileged sockets
Special groups
```

For Docker:

```bash
docker ps
```

Check the Docker socket:

```bash
ls -l /var/run/docker.sock 2>/dev/null
```

For container context:

```bash
cat /proc/1/cgroup 2>/dev/null
cat /proc/1/mountinfo 2>/dev/null
```

Always determine:

```text
Container privilege
vs
Host privilege
```

---

## Phase 18 — Vulnerable Software Analysis

### Objective

Determine:

> **Is a privileged component affected by an applicable vulnerability?**

Collect:

```bash
uname -a
cat /etc/os-release
```

Then identify relevant software versions.

For every candidate vulnerability, verify:

```text
Correct software
Correct version
Correct architecture
Correct distribution
Correct patch status
Correct configuration
Required privileges
Expected privilege impact
```

Use:

```text
Version
   ↓
Candidate vulnerability
   ↓
Applicability
   ↓
Validation
```

Never use:

```text
Old version
   ↓
Automatic exploitation
```

---

## Phase 19 — Automated Enumeration

### Objective

Accelerate discovery.

Use tools such as:

```text
LinPEAS
Linux Smart Enumeration
pspy
```

Treat output as:

```text
Candidate findings
```

not:

```text
Confirmed vulnerabilities
```

The correct loop is:

```text
Tool output
    ↓
Understand finding
    ↓
Reproduce manually
    ↓
Validate
    ↓
Exploit
```

---

## Phase 20 — Build the Finding List

At this point, create a mental or written list.

Example:

```text
Finding 1:
Sudo permission

Finding 2:
Writable cron script

Finding 3:
Unusual SUID binary

Finding 4:
Docker group membership

Finding 5:
Old kernel
```

Do not immediately exploit everything.

Prioritize.

---

## Phase 21 — Prioritize Findings

For every finding ask:

```text
1. Is a privileged component involved?
2. Can I control something it trusts?
3. Can I reproduce the condition?
4. Are the prerequisites satisfied?
5. Does the path cross the privilege boundary?
```

A useful candidate looks like:

```text
Privileged component
        +
User control
        +
Trusted relationship
        +
Applicable conditions
```

---

## Phase 22 — Validate the Attack Path

### Objective

Determine:

> **Is this finding actually exploitable by the current user?**

Use:

```text
Finding
   ↓
Reproduce
   ↓
Understand
   ↓
Identify trusted consumer
   ↓
Identify user control
   ↓
Confirm privileged action
   ↓
Test applicability
   ↓
Validate impact
```

Do not skip this phase.

---

## Phase 23 — Finding vs Vulnerability vs Exploit

Keep these separate.

```text
Finding
   ↓
Something interesting was observed
```

```text
Vulnerability
   ↓
The condition creates a security weakness
```

```text
Exploitability
   ↓
The current user can actually use the weakness
```

```text
Exploitation
   ↓
The validated weakness is used
```

```text
Verification
   ↓
The resulting privilege is proven
```

This distinction is one of the most important concepts in the entire repository.

---

## Phase 24 — Select the Exploitation Path

When multiple validated paths exist, consider:

```text
Reliability
Privilege impact
Prerequisites
Environmental dependencies
Potential impact
Evidence quality
Reproducibility
```

Choose an appropriate authorized technique.

Prefer:

```text
Simple
Controlled
Reproducible
Minimal-impact
```

over unnecessary complexity.

---

## Phase 25 — Record the Baseline

Immediately before exploitation:

```bash
whoami
id
id -u
id -g
```

Record:

```text
Current user
Current UID
Current GID
Current groups
```

This gives you a before-state.

---

## Phase 26 — Controlled Exploitation

Use the validated mechanism.

The basic pattern is:

```text
Validated finding
       ↓
Required preconditions
       ↓
Minimum effective action
       ↓
Privileged execution
```

Avoid unrelated changes.

Avoid persistence unless explicitly authorized.

Avoid destructive actions.

---

## Phase 27 — Verify Privilege

Immediately after exploitation:

```bash
whoami
id
id -u
```

Compare:

```text
BEFORE
UID 1000
    ↓
EXPLOIT
    ↓
AFTER
UID 0
```

Do not declare success merely because a command executed.

---

## Phase 28 — Attribute the Result

Ask:

> **Can I prove that this specific finding caused the privilege increase?**

The final attack path should look like:

```text
Low-privileged user
       ↓
Specific finding
       ↓
Validated user control
       ↓
Privileged action
       ↓
Higher privilege
```

If you cannot explain this chain, return to validation.

---

## Phase 29 — Collect Evidence

Minimum evidence:

```text
Initial identity
Initial UID
Finding
Privileged component
Validation
Exploitation
Final identity
Final UID
Privilege difference
```

Record only the evidence necessary to prove the finding.

---

## Phase 30 — Document the Attack Path

Use:

```text
Initial Access
      ↓
Current Identity
      ↓
Enumeration
      ↓
Finding
      ↓
Validation
      ↓
Exploitation
      ↓
Privilege Verification
      ↓
Root Cause
      ↓
Impact
      ↓
Remediation
```

A good write-up should allow another tester to understand the reasoning without repeating the entire investigation.

---

## Phase 31 — Cleanup

Review what changed:

```text
Files
Configuration
Processes
Services
Scheduled tasks
Temporary artifacts
Network state
```

Before removing anything:

```text
Preserve evidence
      ↓
Identify test-created changes
      ↓
Confirm cleanup is authorized
      ↓
Restore appropriate state
      ↓
Verify final state
```

Do not indiscriminately delete files or processes.

---

## Phase 32 — Final Verification

After cleanup, confirm that:

```text
Evidence preserved
Test artifacts handled
Authorized changes restored
Services functioning as expected
No unintended modifications remain
```

The exact final checks depend on the environment and scope.

---

## Complete Decision Tree

```text
START
  │
  ▼
Authorized target?
  │
  ├── NO ──► STOP
  │
  └── YES
        │
        ▼
Establish identity
        │
        ▼
Establish baseline
        │
        ▼
Enumerate host
        │
        ▼
Find privileged mechanisms
        │
        ▼
Find user-controlled resources
        │
        ▼
Candidate finding?
        │
        ├── NO ──► Continue enumeration
        │
        └── YES
              │
              ▼
        Privileged relationship?
              │
              ├── NO ──► Move on
              │
              └── YES
                    │
                    ▼
              User control?
                    │
                    ├── NO ──► Move on
                    │
                    └── YES
                          │
                          ▼
                    Validate prerequisites
                          │
                          ▼
                    Exploitable?
                          │
                    ┌─────┴─────┐
                    │           │
                   NO          YES
                    │           │
                 Move on     Exploit
                                │
                                ▼
                          Verify privilege
                                │
                                ▼
                          Evidence
                                │
                                ▼
                           Document
                                │
                                ▼
                       Cleanup if authorized
```

---

## When You Are Stuck

Return to the five core questions:

```text
WHO AM I?
```

```text
WHAT IS PRIVILEGED?
```

```text
WHO OWNS IT?
```

```text
WHO CAN MODIFY IT?
```

```text
CAN I INFLUENCE THE PRIVILEGED ACTION?
```

Then ask:

```text
What have I not checked yet?
```

Avoid randomly running exploits.

---

## Fast Workflow

When time is extremely limited:

```text
1. whoami
2. id
3. uname -a
4. cat /etc/os-release
5. sudo -l
6. ps aux
7. systemctl list-units --type=service --state=running
8. find / -perm -4000 -type f 2>/dev/null
9. getcap -r / 2>/dev/null
10. cat /etc/crontab
11. Check writable privileged resources
12. Check credentials
13. Check containers/special groups
14. Validate strongest finding
15. Exploit
16. Verify
```

This is a **triage workflow**, not a replacement for deeper enumeration.

---

## Full Workflow Checklist

```text
AUTHORIZATION
[ ] Target authorized
[ ] Privilege escalation permitted
[ ] Exploitation permitted
[ ] Scope understood

CONTEXT
[ ] whoami
[ ] id
[ ] hostname
[ ] OS
[ ] kernel
[ ] architecture
[ ] environment

SYSTEM
[ ] Users
[ ] Groups
[ ] Processes
[ ] Services
[ ] Installed software
[ ] Configuration

FILESYSTEM
[ ] Permissions
[ ] Writable files
[ ] Writable directories
[ ] Interesting files
[ ] Backups
[ ] Mounts

PRIVILEGE MECHANISMS
[ ] Sudo
[ ] SUID
[ ] SGID
[ ] Capabilities
[ ] Special groups
[ ] Polkit

SCHEDULED EXECUTION
[ ] Cron
[ ] Systemd timers
[ ] Scheduled scripts
[ ] Dynamic execution

CREDENTIALS
[ ] History
[ ] Configuration secrets
[ ] SSH
[ ] Application secrets
[ ] Environment variables
[ ] Backups
[ ] Credential reuse

EXECUTION ABUSE
[ ] PATH
[ ] Wildcards
[ ] Library loading
[ ] Environment
[ ] Shell functions
[ ] Command substitution

SPECIAL ENVIRONMENTS
[ ] Docker
[ ] LXD
[ ] Containers
[ ] NFS
[ ] Mounted filesystems
[ ] Privileged sockets

VULNERABLE SOFTWARE
[ ] Running versions
[ ] Kernel
[ ] CVE applicability
[ ] Exploit prerequisites

AUTOMATION
[ ] LinPEAS
[ ] LSE
[ ] pspy
[ ] Manual reproduction

VALIDATION
[ ] Finding reproduced
[ ] Privileged consumer identified
[ ] User control confirmed
[ ] Preconditions confirmed
[ ] Privilege boundary identified

EXPLOITATION
[ ] Baseline recorded
[ ] Minimum technique selected
[ ] Controlled exploitation performed
[ ] Scope respected

VERIFICATION
[ ] whoami
[ ] id
[ ] UID
[ ] Before/after comparison
[ ] Privilege increase attributed

DOCUMENTATION
[ ] Finding
[ ] Root cause
[ ] Validation
[ ] Exploitation
[ ] Evidence
[ ] Impact
[ ] Remediation

CLEANUP
[ ] Evidence preserved
[ ] Test changes identified
[ ] Cleanup authorized
[ ] Appropriate restoration completed
[ ] Final state checked
```

---

## Final Mental Model

The complete Linux privilege-escalation workflow can be reduced to:

```text
UNDERSTAND
    ↓
ENUMERATE
    ↓
FIND
    ↓
ANALYZE
    ↓
VALIDATE
    ↓
EXPLOIT
    ↓
VERIFY
    ↓
DOCUMENT
```

At every stage ask:

> **What question am I answering?**

Then:

> **What does this result tell me?**

Then:

> **What should I investigate next?**

The ultimate goal is not to memorize hundreds of commands.

It is to recognize this relationship:

```text
LOW PRIVILEGE
      ↓
USER CONTROL
      ↓
TRUSTED PRIVILEGED COMPONENT
      ↓
PRIVILEGED ACTION
      ↓
HIGHER PRIVILEGE
```

Once you can consistently identify that chain, Linux privilege escalation becomes a structured investigation rather than a collection of unrelated tricks.
