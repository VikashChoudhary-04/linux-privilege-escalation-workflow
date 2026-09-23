# TryHackMe Linux Privilege Escalation Labs

> **Purpose:** Document TryHackMe rooms and tasks that provide practical Linux privilege-escalation experience using the workflow defined in this repository.

---

## Purpose

TryHackMe provides controlled environments where Linux privilege-escalation techniques can be practiced repeatedly.

This directory focuses on extracting the **methodology** behind each room rather than simply recording the final answer.

The goal is to move from:

```text id="5m2x8q"
Walkthrough Dependency
        ↓
Command Memorization
```

toward:

```text id="7q4m9x"
System Understanding
        ↓
Question-Driven Enumeration
        ↓
Finding Recognition
        ↓
Attack Path Analysis
        ↓
Validation
        ↓
Controlled Exploitation
        ↓
Verification
```

---

## Directory Purpose

This directory contains write-ups for relevant TryHackMe rooms.

Recommended structure:

```text id="3x8m6q"
18-LABS/
└── tryhackme/
    ├── README.md
    └── <room-name>/
        └── README.md
```

Each room should have its own directory when a complete write-up is required.

---

## What to Record

Every TryHackMe privilege-escalation lab should attempt to capture:

```text id="9m4q7x"
Room
Task
Target
Initial Access
Initial User
Initial Privilege
Enumeration
Finding
Validation
Exploitation
Privilege Verification
Root Cause
Evidence
Lessons Learned
```

The exact fields may be reduced for very small tasks.

---

## Standard TryHackMe Workflow

Use the repository's normal workflow.

```text id="2x7m5q"
START
  ↓
Read Task Objectives
  ↓
Understand Scope
  ↓
Establish Access
  ↓
Identify Current User
  ↓
Identify System
  ↓
Enumerate
  ↓
Build Candidate List
  ↓
Investigate
  ↓
Validate
  ↓
Exploit
  ↓
Verify
  ↓
Document
```

---

## Phase 1 — Understand the Task

Before executing commands:

```text id="6q3m8x"
[ ] Task objective understood
[ ] Target identified
[ ] Required credentials understood
[ ] Expected access level understood
[ ] Questions provided by the room recorded
```

Do not skip the task description.

It often provides important context.

---

## Phase 2 — Establish the Initial Session

Once access is available:

```bash id="8m4x2q"
whoami
id
hostname
```

Record:

```text id="5x9q3m"
Username:
UID:
GID:
Groups:
Hostname:
```

Then establish the operating-system context:

```bash id="7m2q8x"
uname -a
cat /etc/os-release
uname -m
```

---

## Phase 3 — Enumerate Privilege Mechanisms

Start with high-value mechanisms.

### Sudo

```bash id="3q6m9x"
sudo -l
```

### SUID

```bash id="9x4m7q"
find / -perm -4000 -type f 2>/dev/null
```

### Capabilities

```bash id="6m8q2x"
getcap -r / 2>/dev/null
```

### Processes

```bash id="4x7m5q"
ps aux
```

### Services

```bash id="8q3m6x"
systemctl list-units --type=service --state=running
```

### Cron

```bash id="5m9x2q"
cat /etc/crontab
ls -la /etc/cron.d/
```

Do not treat this as a mandatory blind command dump.

Prioritize based on what the system reveals.

---

## Phase 4 — Build a Candidate List

Use a small table while working.

| Candidate   | Privileged Context   | User Control | Trigger        | Status        |
| ----------- | -------------------- | ------------ | -------------- | ------------- |
| Sudo rule   | Root                 | Yes/No       | Manual         | Investigating |
| SUID binary | Root                 | Yes/No       | Execution      | Investigating |
| Cron script | Root                 | Yes/No       | Schedule       | Investigating |
| Service     | Root                 | Yes/No       | Service start  | Investigating |
| Credential  | Another account/root | Yes          | Authentication | Investigating |

The exact candidates will vary by room.

---

## Phase 5 — Investigate the Finding

For each candidate ask:

```text id="2q8m5x"
Who owns it?
Who executes it?
Who can modify it?
What does it trust?
When does it execute?
What privilege does it have?
Can my current user influence it?
```

This is where enumeration becomes analysis.

---

## Phase 6 — Validate the Attack Path

A candidate should become an exploitation target only after the relevant relationships are understood.

Use:

```text id="7x3m9q"
Finding
   ↓
Privileged Component
   ↓
User Control
   ↓
Trusted Input / Action
   ↓
Execution
   ↓
Privilege Impact
```

If one of these links is missing, investigate further.

If the required relationship does not exist, move to the next candidate.

---

## Phase 7 — Exploit

Only exploit validated findings within the authorized TryHackMe environment.

Record:

```text id="5q8m2x"
Technique:
Preconditions:
Action:
Expected Result:
Observed Result:
```

Prefer the smallest effective exploitation step.

---

## Phase 8 — Verify

Do not stop immediately after the exploit appears successful.

Run:

```bash id="8m4x6q"
whoami
id
id -u
```

Record the result.

For a root shell:

```text id="3x7m9q"
UID = 0
```

If the room uses a flag as the final objective, record the flag separately from privilege verification.

---

## Phase 9 — Document the Root Cause

Do not document only:

```text id="6m2q8x"
"Used command X."
```

Document:

```text id="9x5m3q"
The initial user could influence RESOURCE.
RESOURCE was trusted by COMPONENT.
COMPONENT executed with PRIVILEGE.
The controlled input reached the privileged action.
The resulting privilege change was verified.
```

This makes the write-up reusable.

---

## Room Write-Up Template

Use the following structure for individual room write-ups.

```text id="4m8q2x"
# Room Name

## Overview

Platform:
Room:
Difficulty:
Focus:

## Objective

What the room is designed to teach.

## Initial Access

How access was obtained.

Initial user:
Initial UID:
Initial groups:

## System Context

OS:
Kernel:
Architecture:
Hostname:

## Enumeration

Commands and important observations.

## Candidate Findings

### Candidate 1

Finding:
Owner:
Permissions:
Privileged context:
User control:
Execution trigger:
Status:

## Validation

How the candidate was validated.

## Exploitation

Technique:
Execution:
Result:

## Privilege Verification

whoami:
id:
UID:

## Root Cause

Why the privilege boundary could be crossed.

## Lessons Learned

Key concepts learned from the room.

## Methodology Mapping

Relevant repository sections.

## Quick Takeaway

The most important lesson from the room.
```

---

## Methodology Mapping

Each room should be connected to the repository's workflow.

Example:

| Room Finding            | Repository Section           |
| ----------------------- | ---------------------------- |
| Sudo misconfiguration   | `06-PRIVILEGE-MECHANISMS`    |
| SUID binary             | `06-PRIVILEGE-MECHANISMS`    |
| Cron job                | `07-SCHEDULED-EXECUTION`     |
| Credentials             | `08-CREDENTIALS-AND-SECRETS` |
| PATH hijacking          | `09-EXECUTION-ABUSE`         |
| Docker access           | `10-SPECIAL-ENVIRONMENTS`    |
| Kernel vulnerability    | `11-VULNERABLE-SOFTWARE`     |
| LinPEAS discovery       | `12-AUTOMATED-ENUMERATION`   |
| False positive analysis | `13-FINDING-VALIDATION`      |
| Exploitation            | `14-EXPLOITATION`            |
| Root verification       | `15-ROOT-VERIFICATION`       |
| Complete reasoning      | `16-METHODOLOGY`             |

This makes the lab section reinforce the rest of the repository.

---

## Avoiding Walkthrough Dependency

When solving a room, try this order:

```text id="7m5q2x"
1. Read the task
2. Establish context
3. Enumerate manually
4. Build candidates
5. Investigate candidates
6. Validate
7. Use automated enumeration if needed
8. Compare findings
9. Exploit
10. Review the intended lesson
```

If a walkthrough is necessary, use it to understand the missed reasoning rather than merely copying commands.

Ask:

> What observation should have led me to this technique?

---

## Learning From Failure

A failed attempt is useful when documented.

Record:

```text id="3x9m6q"
Candidate:
Why it looked promising:
What I tested:
Why it failed:
What assumption was wrong:
What I investigated next:
```

This develops troubleshooting ability.

---

## Common TryHackMe Mistakes

### Starting With the Flag

Finding the final flag is not the same as understanding privilege escalation.

---

### Searching for Kernel Exploits Immediately

First investigate simpler privilege mechanisms and misconfigurations.

---

### Treating Enumeration Output as Proof

A tool finding is a candidate.

Manual analysis determines whether it is relevant.

---

### Ignoring Ownership

Always ask:

```text id="8m4q2x"
Who owns this?
Who can modify this?
Who executes this?
```

---

### Ignoring Execution Context

A writable file is not automatically useful.

Determine who executes it and with what privilege.

---

### Forgetting Verification

Always verify the actual final privilege level.

---

## Progress Tracking

A useful external or repository-level record can contain:

| Date       | Room | Difficulty | Initial Access | PrivEsc Technique | Root Cause       | Status |
| ---------- | ---- | ---------- | -------------- | ----------------- | ---------------- | ------ |
| YYYY-MM-DD | Room | Easy       | Method         | Technique         | Misconfiguration | Solved |

Keep the record factual.

The purpose is to measure exposure to different techniques, not simply count completed rooms.

---

## Recommended Technique Coverage

Over time, try to encounter different privilege-escalation mechanisms.

```text id="5x7m9q"
[ ] Sudo
[ ] SUID
[ ] SGID
[ ] Capabilities
[ ] Cron
[ ] Systemd
[ ] Writable Scripts
[ ] Writable Services
[ ] PATH Hijacking
[ ] Wildcard Abuse
[ ] Credentials
[ ] SSH Keys
[ ] Special Groups
[ ] Docker
[ ] NFS
[ ] Kernel Vulnerability
[ ] Automated Enumeration
[ ] Multi-Step Attack Path
```

The list is a learning map, not a requirement that every room contain every technique.

---

## Lab Completion Checklist

```text id="9m3q6x"
[ ] Room objective understood
[ ] Initial access documented
[ ] Initial user documented
[ ] Initial privilege documented
[ ] OS and kernel identified
[ ] Relevant privilege mechanisms enumerated
[ ] Candidate findings recorded
[ ] Strongest candidate investigated
[ ] Attack path understood
[ ] Finding validated
[ ] Exploitation performed within scope
[ ] Final privilege verified
[ ] Root cause documented
[ ] Evidence recorded
[ ] Lessons learned written
[ ] Methodology section mapped
```

---

## Final Mental Model

Every TryHackMe Linux privilege-escalation room should reinforce this sequence:

```text id="2q8m4x"
ACCESS
  ↓
CONTEXT
  ↓
ENUMERATION
  ↓
FINDING
  ↓
WHO CONTROLS IT?
  ↓
WHAT DOES IT TRUST?
  ↓
WHEN DOES IT EXECUTE?
  ↓
VALIDATION
  ↓
EXPLOITATION
  ↓
VERIFICATION
  ↓
ROOT CAUSE
  ↓
LESSON
```

> **Do not measure a lab only by whether you reached root. Measure it by whether you can explain the complete privilege boundary that made root reachable.**
