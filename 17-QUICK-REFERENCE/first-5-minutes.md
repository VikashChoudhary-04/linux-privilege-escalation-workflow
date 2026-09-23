# Linux Privilege Escalation — First 5 Minutes

> **Purpose:** Provide a compact, repeatable workflow for the first five minutes after obtaining an authorized low-privileged Linux shell.

---

## Objective

The first five minutes are not about finding every possible vulnerability.

They are about answering:

```text id="6s4r0j"
WHO AM I?
WHAT SYSTEM AM I ON?
WHAT GROUPS DO I HAVE?
WHAT PRIVILEGED MECHANISMS EXIST?
IS THERE AN OBVIOUS ATTACK PATH?
```

The workflow is:

```text id="5m9d2q"
IDENTITY
   ↓
SYSTEM
   ↓
GROUPS
   ↓
SUDO
   ↓
PROCESSES / SERVICES
   ↓
SUID / CAPABILITIES
   ↓
DECIDE
```

---

## 0:00–1:00 — Establish Identity

Run:

```bash id="0m6k4a"
whoami
id
id -u
id -g
```

Record:

```text id="r9p5x3"
Username
UID
Primary group
Supplementary groups
```

### Decision

```text id="6k7v1p"
UID 0?
 │
 ├── YES → Verify privileged context
 │
 └── NO  → Continue
```

If UID 0 is already confirmed, do not continue generic privilege escalation enumeration unnecessarily.

---

## 1:00–2:00 — Establish System Context

Run:

```bash id="7x3n9w"
hostname
uname -a
cat /etc/os-release
uname -m
```

Determine:

```text id="4k8q2v"
Hostname
Distribution
Version
Kernel
Architecture
```

### Why It Matters

System context helps determine:

* available functionality
* compatibility
* service configuration
* architecture-specific behavior
* later vulnerability applicability

Do not treat the kernel version alone as an exploit finding.

---

## 2:00–2:30 — Check Groups

Run:

```bash id="f3r8y5"
id
groups
```

Look for unusual memberships.

Examples:

```text id="q7w2m1"
docker
lxd
disk
libvirt
```

These names are indicators for investigation, not automatic proof of escalation.

Ask:

```text id="j9c4z6"
What access does this group provide?
Does that access cross the privilege boundary?
```

---

## 2:30–3:00 — Check Sudo

Run:

```bash id="h4x7m2"
sudo -l
```

Look for:

```text id="8s5p1c"
Commands executable as root
Commands executable as another user
Unexpected command restrictions
Dangerous interpreters
Programs capable of reading or modifying privileged resources
```

### Decision

```text id="2v9n5k"
Useful sudo permission?
 │
 ├── YES → Investigate immediately
 │
 └── NO  → Continue
```

Do not ignore a useful sudo result just because other enumeration has not been completed.

---

## 3:00–3:45 — Check Processes

Run:

```bash id="y8c3f1"
ps aux
```

Look for:

```text id="m4k7q2"
root processes
custom applications
scripts
unexpected commands
interesting arguments
```

A root process is only a candidate.

Ask:

```text id="n6j2x5"
What executable?
What configuration?
What files?
Who can modify its dependencies?
```

---

## 3:45–4:15 — Check Services

Run:

```bash id="w2q7m9"
systemctl list-units --type=service --state=running
```

For an interesting service:

```bash id="e5v8r3"
systemctl status <service>
systemctl cat <service>
```

Prioritize:

```text id="9x3m6k"
Custom
Third-party
Root-running
Script-based
Unexpected
```

Ask:

```text id="g7p2v4"
Who runs it?
What does it execute?
Can I influence anything it trusts?
```

---

## 4:15–4:45 — Check SUID

Run:

```bash id="k5m8x2"
find / -perm -4000 -type f 2>/dev/null
```

Look for:

```text id="w4q9j7"
Unusual binaries
Custom binaries
Unexpected privileged programs
```

Do not investigate every standard system SUID binary.

For an interesting result:

```text id="6c2n8v"
Owner?
Purpose?
Can I execute it?
Can I control its behavior?
```

---

## 4:45–5:00 — Check Capabilities

Run:

```bash id="u3x6p9"
getcap -r / 2>/dev/null
```

Look for:

```text id="m7v2c5"
Unusual binaries
Powerful capabilities
User-executable binaries
```

Ask:

```text id="2f8k4r"
What capability?
What does the binary do?
Can I execute it?
Can I control its behavior?
```

---

## The 5-Minute Decision Point

At exactly five minutes, stop and ask:

```text id="s9m4x2"
Do I have a credible privilege-escalation candidate?
```

A strong candidate generally has:

```text id="k2q7m8"
PRIVILEGED COMPONENT
        +
USER CONTROL
        +
EXECUTION RELATIONSHIP
```

### If Yes

```text id="p5r8y3"
STOP BROAD ENUMERATION
        ↓
TRACE THE ATTACK PATH
        ↓
VALIDATE
```

### If No

Continue with:

```text id="w6c3n9"
Cron
Timers
Filesystem
Credentials
Network
Special environments
Automated enumeration
```

---

## Compact Command Block

For quick copy/paste during an authorized assessment:

```bash id="q8v2m5"
whoami
id
hostname
uname -a
cat /etc/os-release
groups
sudo -l
ps aux
systemctl list-units --type=service --state=running
find / -perm -4000 -type f 2>/dev/null
getcap -r / 2>/dev/null
```

Run commands individually when you need to interpret the result before continuing.

---

## What to Record

Keep a minimal five-minute record:

```text id="j5x8q1"
User:
UID:
Groups:

Hostname:
OS:
Kernel:
Architecture:

Sudo:
Interesting Processes:
Interesting Services:
SUID:
Capabilities:

Primary Candidate:
Next Question:
```

---

## What Not to Do

### Do Not Run Everything

The first five minutes are for high-value discovery.

### Do Not Exploit Immediately

A finding must be understood and validated first.

### Do Not Treat Tool Output as Proof

Enumeration output identifies candidates.

### Do Not Ignore Groups

Group membership can expose important trust relationships.

### Do Not Assume SUID Means Vulnerable

SUID is a privilege mechanism, not automatically an exploitable condition.

### Do Not Assume Root Process Means Vulnerable

You need user influence over the privileged execution path.

### Do Not Jump to Kernel Exploitation

First investigate simpler, directly controllable paths.

---

## First 5 Minutes Decision Tree

```text id="b3r7k9"
LOW-PRIVILEGED SHELL
        │
        ▼
WHO AM I?
        │
        ▼
WHAT SYSTEM?
        │
        ▼
WHAT GROUPS?
        │
        ▼
SUDO?
   ┌────┴────┐
  YES        NO
   │          │
Investigate   ▼
             PROCESSES
                │
                ▼
             SERVICES
                │
                ▼
               SUID
                │
                ▼
          CAPABILITIES
                │
                ▼
        CREDIBLE CANDIDATE?
          ┌─────┴─────┐
         YES          NO
          │            │
       VALIDATE     ENUMERATE
```

---

## The Five Questions

Memorize these:

```text id="v7m2c8"
1. WHO AM I?
2. WHAT IS PRIVILEGED?
3. WHO OWNS IT?
4. WHO CAN MODIFY IT?
5. CAN MY CONTROL REACH THE PRIVILEGED ACTION?
```

If you cannot answer question 5, you do not yet have a proven attack path.

---

## When to Move On

Move on from a finding when:

```text id="r4x8n2"
No privileged component
```

or:

```text id="c6m3q7"
No user control
```

or:

```text id="p8v5j1"
No execution relationship
```

or:

```text id="z2k9m4"
No meaningful privilege impact
```

Do not force a finding into an exploit path.

---

## Five-Minute Mental Model

```text id="g5n8r2"
IDENTITY
   ↓
CONTEXT
   ↓
PRIVILEGE
   ↓
CONTROL
   ↓
CANDIDATE
   ↓
VALIDATE
```

The goal of the first five minutes is **not**:

> Find root immediately.

The goal is:

> **Find the strongest question worth investigating next.**
