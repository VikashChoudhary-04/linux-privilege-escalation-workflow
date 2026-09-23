# Linux Privilege Escalation — 15-Minute Enumeration

> **Purpose:** Provide a structured 15-minute Linux privilege-escalation enumeration workflow that expands beyond the first five minutes while keeping the investigation focused on meaningful privilege boundaries.

---

## Objective

The first five minutes establish context.

The next ten minutes answer:

```text
WHAT ELSE IS PRIVILEGED?
WHAT DOES IT TRUST?
WHAT CAN I CONTROL?
WHAT EXECUTES AUTOMATICALLY?
ARE THERE CREDENTIALS?
ARE THERE SPECIAL ENVIRONMENTS?
```

The workflow is:

```text
0–5 MINUTES
Fast discovery
      ↓
5–10 MINUTES
Filesystem + scheduled execution + credentials
      ↓
10–15 MINUTES
Network + special environments + correlation
      ↓
DECISION
Validate the strongest candidate
```

---

## Before You Start

Confirm:

```text
[ ] Authorized target
[ ] Known scope
[ ] Current shell established
[ ] First-five-minute checks completed
```

This workflow assumes the initial context from:

`17-QUICK-REFERENCE/first-5-minutes.md`

has already been collected.

---

## Minute 0–5 — Initial Baseline

Run the first-five-minute workflow:

```bash
whoami
id
hostname
uname -a
cat /etc/os-release
sudo -l
ps aux
systemctl list-units --type=service --state=running
find / -perm -4000 -type f 2>/dev/null
getcap -r / 2>/dev/null
```

At the five-minute point, identify:

```text
Known privileged components
Known user-controlled resources
Potential credentials
Interesting groups
Candidate attack paths
```

Do not restart this phase repeatedly.

---

## Minute 5–7 — Scheduled Execution

### Cron

Inspect:

```bash
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
```

Look for:

```text
root execution
custom scripts
unusual commands
user-controlled paths
```

For a referenced script:

```bash
ls -la <script>
stat <script>
```

Ask:

```text
Who executes it?
Can I modify it?
When does it execute?
```

---

## Minute 7–8 — Systemd Timers

Run:

```bash
systemctl list-timers --all
```

For an interesting timer:

```bash
systemctl cat <timer>
```

Trace the associated service:

```text
Timer
  ↓
Service
  ↓
Executable
  ↓
Script / configuration
  ↓
Dependencies
```

Then ask:

```text
Can the current user influence any part of this chain?
```

---

## Minute 8–10 — Filesystem and Writable Resources

Inspect high-value locations:

```bash
ls -la /
ls -la /opt/
ls -la /usr/local/
ls -la /tmp/
```

Look for:

```text
Custom applications
Scripts
Backups
Configuration files
Unusual binaries
Writable directories
```

For a candidate:

```bash
ls -la <path>
stat <path>
```

The critical question is:

> **Who consumes this resource?**

A writable file without a privileged consumer may be irrelevant.

---

## Minute 10–11 — History and Environment

Check:

```bash
history 2>/dev/null
env
echo "$PATH"
```

Look for:

```text
Credentials
Tokens
Internal paths
Custom commands
Sensitive application variables
Unusual PATH entries
```

Do not unnecessarily display or copy secret values.

---

## Minute 11–12 — Configuration and Credentials

Focus on user-accessible configuration and application directories.

Useful locations include:

```text
/etc/
/opt/
/usr/local/
/home/
/var/www/
```

Look for:

```text
Database credentials
Application secrets
Service credentials
SSH material
Backup files
Configuration files
```

For every credential:

```text
Who owns it?
Where is it used?
What privilege does that account have?
Is its use authorized?
```

---

## Minute 12–13 — Network

Run:

```bash
ip addr
ip route
ss -lntup
```

Prioritize:

```text
localhost services
custom applications
administrative interfaces
privileged services
```

Ask:

```text
Does this service provide another trust relationship?
Does it run with higher privilege?
Can the current user interact with it?
```

---

## Minute 13–14 — Mounts and Special Environments

Run:

```bash
findmnt
```

Check for indicators of:

```text
NFS
Docker
LXD
Containers
Unusual mounts
Host filesystem exposure
```

Also check groups again:

```bash
id
```

If a relevant group appears, investigate what access it actually grants.

---

## Minute 14–15 — Correlate Findings

Stop collecting broad information.

Build a short candidate list:

```text
Candidate A:
Candidate B:
Candidate C:
```

For each candidate answer:

```text
Privileged component?
User control?
Trust relationship?
Execution?
Privilege impact?
```

Example:

```text
Candidate A
Root cron
+
Writable script
+
Known execution
=
Strong candidate
```

---

## 15-Minute Decision Point

At minute 15, choose one of three paths.

### Path A — Complete Candidate Exists

```text
Candidate
   ↓
Known privilege
   ↓
Known user control
   ↓
Known execution
```

Action:

```text
STOP BROAD ENUMERATION
        ↓
TRACE
        ↓
VALIDATE
```

---

### Path B — Candidate Exists but One Relationship Is Unknown

Example:

```text
Root service
   ↓
Unknown configuration ownership
```

Action:

```text
Investigate the missing relationship
        ↓
Validate or reject
```

---

### Path C — No Credible Candidate

Action:

```text
Run automated enumeration
        ↓
Review custom services
        ↓
Review credentials
        ↓
Review software versions
        ↓
Search for indirect trust relationships
```

---

## Candidate Analysis Template

For every interesting finding:

```text
Finding:
Privileged Component:
Owner:
Execution User:

Current User Control:
Controlled Resource:

Trust Relationship:
Execution Trigger:

Expected Privilege:
Prerequisites:

Validation Status:
Next Action:
```

---

## High-Value Checks

Prioritize findings involving:

```text
SUDO
ROOT SERVICES
ROOT PROCESSES
SUID / SGID
CAPABILITIES
CRON
SYSTEMD TIMERS
WRITABLE PRIVILEGED RESOURCES
CREDENTIALS
PRIVILEGED GROUPS
CONTAINERS
SPECIAL MOUNTS
```

---

## Fast Command Reference

### Identity

```bash
whoami
id
```

### System

```bash
hostname
uname -a
cat /etc/os-release
```

### Sudo

```bash
sudo -l
```

### Processes

```bash
ps aux
```

### Services

```bash
systemctl list-units --type=service --state=running
```

### SUID

```bash
find / -perm -4000 -type f 2>/dev/null
```

### Capabilities

```bash
getcap -r / 2>/dev/null
```

### Cron

```bash
cat /etc/crontab
ls -la /etc/cron.d/
```

### Timers

```bash
systemctl list-timers --all
```

### Network

```bash
ip addr
ip route
ss -lntup
```

### Mounts

```bash
findmnt
```

### Environment

```bash
env
echo "$PATH"
```

---

## What to Ignore Initially

Do not spend significant time on:

```text
Ordinary system files
Normal root processes with no user control
Standard SUID binaries without interesting behavior
Unrelated listening services
Unprivileged application files
Old software without applicability evidence
```

These may become relevant later if new evidence connects them to a privilege boundary.

---

## Example: Strong 15-Minute Path

Suppose you discover:

```text
User
 ↓
docker group
 ↓
Docker socket accessible
 ↓
Container runtime controlled
```

Do not continue running unrelated filesystem searches merely because fifteen minutes have not elapsed.

Instead:

```text
Analyze runtime access
       ↓
Determine host privilege impact
       ↓
Validate
```

Time-boxing is about prioritization, not forcing a fixed number of commands.

---

## Example: Weak 15-Minute Finding

Suppose you discover:

```text
Writable file:
 /tmp/example.conf
```

Ask:

```text
Who reads it?
```

If:

```text
No privileged consumer
```

then:

```text
Reject candidate
```

and continue.

---

## Example: Incomplete Path

Suppose:

```text
Root service
   ↓
Custom script
   ↓
Unknown permissions
```

The next question is:

```text
Who can modify the script?
```

Run:

```bash
ls -la <script>
stat <script>
```

If the current user cannot modify it:

```text
Candidate rejected
```

If the user can modify it:

```text
Candidate becomes high priority
```

---

## Automated Enumeration

If no credible path has emerged, use an enumeration tool.

Typical workflow:

```text
Manual Baseline
      ↓
LinPEAS / LSE / pspy
      ↓
Interesting Output
      ↓
Manual Reproduction
      ↓
Attack Path Analysis
      ↓
Validation
```

Do not blindly execute every suggestion produced by a tool.

---

## pspy Use

When scheduled or event-driven execution remains unclear, a process-monitoring tool such as `pspy` can help observe processes without requiring root privileges.

The reasoning flow is:

```text
Unknown execution
      ↓
Observe process activity
      ↓
Identify recurring command
      ↓
Identify execution context
      ↓
Trace resource
      ↓
Validate
```

The observation itself is not proof of exploitability.

---

## 15-Minute Stopping Rules

### Stop Broad Enumeration If

```text
Complete attack path exists
```

### Investigate Deeper If

```text
Privileged component
+
User control
+
Unknown relationship
```

### Move On If

```text
No privileged consumer
```

or:

```text
No user control
```

or:

```text
No meaningful privilege impact
```

---

## Common Mistakes

### Mistake 1: Repeating the First Five Minutes

Do not repeatedly rerun the same baseline commands without a reason.

### Mistake 2: Enumerating Everything

The goal is not complete filesystem coverage.

### Mistake 3: Ignoring Scheduled Execution

Cron and systemd timers can reveal privileged execution paths that are invisible from process listings.

### Mistake 4: Ignoring Credentials

Credentials can create indirect attack paths.

### Mistake 5: Treating Open Ports as Vulnerabilities

A listening service is an observation. Determine what it does and whether it creates a relevant trust relationship.

### Mistake 6: Running Automated Tools Without Triage

Tool output must be interpreted.

### Mistake 7: Refusing to Abandon a Weak Finding

A weak candidate should not consume unlimited time.

---

## 15-Minute Checklist

```text
BASELINE
[ ] Identity
[ ] Groups
[ ] OS
[ ] Kernel
[ ] Architecture
[ ] Sudo

PRIVILEGED EXECUTION
[ ] Processes
[ ] Services
[ ] SUID
[ ] SGID
[ ] Capabilities

SCHEDULED EXECUTION
[ ] Cron
[ ] Cron directories
[ ] Systemd timers

FILESYSTEM
[ ] Important directories
[ ] Writable resources
[ ] Configurations
[ ] Backups

CREDENTIALS
[ ] History
[ ] Environment
[ ] Application configuration
[ ] SSH material

NETWORK
[ ] Interfaces
[ ] Routes
[ ] Listening services
[ ] Localhost services

SPECIAL ENVIRONMENTS
[ ] Mounts
[ ] Docker
[ ] LXD
[ ] Containers
[ ] Special groups

DECISION
[ ] Candidate identified
[ ] Privileged component known
[ ] User control known
[ ] Execution relationship known
[ ] Attack path constructed
[ ] Validation started
```

---

## Final Mental Model

Remember the 15-minute workflow as:

```text
5 MINUTES
DISCOVER
   ↓
10 MINUTES
EXPAND
   ↓
15 MINUTES
CONNECT
   ↓
VALIDATE
```

The central rule is:

> **By minute 15, you should be moving from "What exists?" toward "Which of these findings can actually cross the privilege boundary?"**
