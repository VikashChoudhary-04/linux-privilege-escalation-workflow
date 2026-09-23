# Linux Privilege Escalation Labs

> **Purpose:** Apply the Linux privilege-escalation workflow against controlled labs and deliberately vulnerable systems.

---

## Purpose of This Section

The workflow becomes useful when it can be applied repeatedly.

This section converts the methodology from:

```text
Understand
   ↓
Enumerate
   ↓
Recognize
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

into practical exercises.

The goal is not simply to obtain root.

The goal is to understand:

* Why the escalation was possible
* Which enumeration step exposed it
* Which privilege boundary was crossed
* What trust relationship made it possible
* How the finding was validated
* How the result was verified
* How the attack path should be documented

---

## Lab Philosophy

A good privilege-escalation lab should answer more than:

> "Can I become root?"

It should also answer:

> "Could I have recognized this path systematically without already knowing the solution?"

Each lab should therefore be treated as an investigation.

```text
Initial Access
      ↓
Context
      ↓
Enumeration
      ↓
Candidate Finding
      ↓
Investigation
      ↓
Validation
      ↓
Exploitation
      ↓
Root Verification
      ↓
Documentation
```

---

## Authorized Environment

Only use these workflows against systems you are authorized to test.

Suitable environments include:

* Intentionally vulnerable training machines
* TryHackMe rooms
* Hack The Box machines
* Proving Grounds labs
* OWASP training environments
* Local virtual machines
* Personal test infrastructure
* Other explicitly authorized assessment targets

Do not apply these techniques to systems outside the permitted scope.

---

## Lab Categories

This repository organizes labs into four categories.

### TryHackMe

Location:

```text
18-LABS/tryhackme/
```

Use this section for TryHackMe rooms involving Linux privilege escalation.

Focus on documenting the methodology rather than simply copying walkthrough answers.

---

### Hack The Box

Location:

```text
18-LABS/hackthebox/
```

Use this section for Hack The Box machines and challenges involving Linux privilege escalation.

Where appropriate, separate:

```text
Initial Access
```

from:

```text
Privilege Escalation
```

so the escalation path remains clear.

---

### Proving Grounds

Location:

```text
18-LABS/proving-grounds/
```

Use this section for Proving Grounds machines and practice environments.

Document the privilege boundary and reasoning used to identify the escalation path.

---

### Local Labs

Location:

```text
18-LABS/local-labs/
```

Use this section for intentionally vulnerable systems created or hosted locally.

Examples include:

* Vulnerable Linux virtual machines
* Custom privilege-escalation exercises
* Docker-based training environments
* Purpose-built service misconfigurations
* Controlled filesystem and permission scenarios

---

## Standard Lab Workflow

Every lab should follow the same high-level process.

### Phase 1 — Establish Context

Record:

```bash
whoami
id
hostname
```

Then identify:

```text
OS
Kernel
Architecture
Groups
Shell
Environment
```

---

### Phase 2 — Enumerate

Use the repository workflow systematically.

Start with high-value checks:

```text
sudo
SUID / SGID
Capabilities
Processes
Services
Cron / Timers
Writable Resources
Credentials
Network
Special Environments
Software Versions
```

Do not blindly execute every available command.

Ask:

> What question does this command answer?

---

### Phase 3 — Build a Candidate List

Record interesting findings as they appear.

Example:

```text
Candidate #1
Finding: Root cron job
Resource: /opt/backup.sh
Owner: root
User control: Yes
Execution: Every minute
Status: Needs validation
```

Example:

```text
Candidate #2
Finding: SUID binary
Binary: /usr/local/bin/example
Owner: root
User execution: Yes
Status: Investigating
```

---

### Phase 4 — Investigate

For each candidate determine:

```text
Who owns it?
Who executes it?
Who can modify it?
What does it trust?
When does it execute?
What privilege does it have?
Can the current user influence it?
```

This converts an observation into an attack-path hypothesis.

---

### Phase 5 — Validate

Do not immediately exploit.

First establish:

```text
Finding
   ↓
User Control
   ↓
Privileged Execution
   ↓
Privilege Impact
```

Reject candidates that cannot cross the privilege boundary.

---

### Phase 6 — Exploit

Only after validation:

```text
Validated Finding
       ↓
Controlled Exploitation
       ↓
Privilege Change
```

Use the smallest effective action.

Avoid unnecessary:

* Persistence
* Destructive changes
* System modifications
* Credential exposure
* Service disruption

---

### Phase 7 — Verify

Run:

```bash
whoami
id
id -u
```

Confirm the actual privilege level.

For root:

```text
UID = 0
```

Do not assume successful command execution means successful privilege escalation.

---

### Phase 8 — Document

Record:

```text
Initial User
Initial Privilege
Finding
Root Cause
Attack Path
Validation
Exploitation
Final Privilege
Evidence
Cleanup
```

The final write-up should allow another tester to understand the path without relying on guesswork.

---

## Lab Documentation Standard

Each completed lab should contain enough information to answer:

### Initial Access

```text
How did the session begin?
Who was the initial user?
What was the initial privilege level?
```

### Enumeration

```text
What was checked?
Which finding mattered?
Why did it matter?
```

### Analysis

```text
What privileged component was involved?
Who owned it?
Who could influence it?
What did it trust?
```

### Validation

```text
How was the finding confirmed?
Why was it considered exploitable?
```

### Exploitation

```text
What controlled action crossed the privilege boundary?
```

### Verification

```text
How was the final privilege verified?
```

### Root Cause

```text
Why was the system vulnerable?
```

### Cleanup

```text
What temporary changes or artifacts were removed?
```

---

## Lab Evidence

Where appropriate, preserve evidence such as:

```text
whoami output
id output
Relevant permissions
Ownership information
Service configuration
Cron configuration
Sudo configuration
Capability output
Process information
Validation result
Final UID
```

Avoid collecting unnecessary sensitive information.

Use sanitized examples when publishing write-ups.

---

## Recommended Lab Record

Use this structure for each lab:

```text
# Lab Name

## Target

Platform:
Machine:
Difficulty:
IP:
URL:

## Initial Access

Initial user:
Initial privilege:
Access method:

## Enumeration

Important findings:

## Candidate Findings

### Candidate 1

Finding:
Owner:
Permissions:
User control:
Privileged consumer:
Execution trigger:
Status:

## Validation

Validation performed:
Expected result:
Observed result:

## Exploitation

Technique:
Privilege boundary:
Result:

## Verification

whoami:
id:
UID:

## Root Cause

Why the escalation was possible:

## Evidence

Relevant evidence:

## Cleanup

Actions performed:

## Lessons Learned

Key lesson:
Enumeration step that exposed the path:
```

---

## Finding Quality

A useful lab write-up should distinguish between:

```text
Observation
    ↓
Finding
    ↓
Potential Vulnerability
    ↓
Validated Attack Path
    ↓
Successful Exploitation
```

For example:

```text
SUID binary exists
```

is an observation.

```text
The SUID binary performs a privileged action
```

is a finding.

```text
The current user can influence the privileged action
```

is stronger evidence.

```text
The influence crosses the privilege boundary
```

establishes an attack path.

```text
Controlled exploitation produces UID 0
```

establishes successful escalation.

---

## Manual Before Automated

When practical:

```text
Manual Enumeration
       ↓
Candidate Identification
       ↓
Automated Enumeration
       ↓
Correlation
       ↓
Manual Validation
```

Automated tools should help discover missed candidates.

They should not replace understanding.

Recommended tools for authorized labs include:

```text
LinPEAS
Linux Smart Enumeration
pspy
```

---

## Learning Objectives

After completing labs in this section, you should be able to:

```text
[ ] Establish Linux privilege context quickly
[ ] Enumerate common privilege mechanisms
[ ] Recognize high-value findings
[ ] Analyze ownership and permissions
[ ] Identify trusted privileged execution
[ ] Build complete attack paths
[ ] Reject false positives
[ ] Validate findings safely
[ ] Select an appropriate exploitation path
[ ] Verify privilege changes
[ ] Document root causes
[ ] Clean up after testing
```

---

## Progress Tracking

Track completed labs using a simple record:

```text
Date
Platform
Machine
Difficulty
Initial Access
Privilege Escalation
Root Cause
Technique
Status
Notes
```

A more detailed progress record can be maintained outside the individual write-ups when needed.

---

## Difficulty Progression

A useful learning progression is:

```text
Beginner
   ↓
Basic Enumeration
   ↓
Single-Mechanism Escalation
   ↓
Multi-Step Escalation
   ↓
Chained Attack Paths
   ↓
Complex Privilege Boundaries
```

Do not judge progress only by the number of machines completed.

A stronger indicator is whether you can identify and explain the attack path without relying on a walkthrough.

---

## Common Lab Mistakes

### Looking for Exploits Too Early

Do not immediately search for kernel exploits.

First determine whether a simpler local misconfiguration exists.

---

### Treating Every Finding as Exploitable

A finding becomes valuable only when it can influence a privileged action.

---

### Running Tools Without Reading Output

Automated enumeration produces candidates, not conclusions.

---

### Following Walkthroughs Blindly

If a walkthrough says:

```text
Run this command
```

ask:

```text
Why?
What did the previous step establish?
What would prove this is the correct path?
```

---

### Forgetting Verification

Always verify the final privilege level.

---

### Poor Documentation

A successful shell without a documented attack path provides limited learning value.

---

## Lab Completion Criteria

Consider a lab complete when you can explain:

```text
1. How access was obtained
2. Who the initial user was
3. What was enumerated
4. Which finding mattered
5. Why the finding mattered
6. Who controlled the relevant resource
7. What privileged component trusted it
8. How the privilege boundary was crossed
9. How the result was verified
10. Why the system was vulnerable
11. What evidence proves the path
12. What cleanup was performed
```

---

## Quick Lab Flow

```text
START
  ↓
Establish Context
  ↓
Identify User
  ↓
Identify System
  ↓
Enumerate Privilege Mechanisms
  ↓
Find Interesting Resources
  ↓
Build Candidate List
  ↓
Investigate Highest-Value Candidate
  ↓
Can User Influence Privileged Action?
  ├── NO → Reject / Move On
  └── YES
        ↓
      Validate
        ↓
      Exploit
        ↓
      Verify
        ↓
      Document
        ↓
      Clean Up
        ↓
       END
```

---

## Where This Leads

The lab section connects the workflow to real practice:

```text
00–17
Methodology + Workflow + Quick Reference
        ↓
18
Hands-On Labs
        ↓
19
Case Studies
```

Labs demonstrate that the methodology works against real training scenarios.

Case studies then isolate individual privilege-escalation mechanisms and explain them in greater depth.

---

## Final Mental Model

Do not approach a lab as:

> "Where is the root shell?"

Approach it as:

```text
Who am I?
     ↓
What is running?
     ↓
What is privileged?
     ↓
Who owns it?
     ↓
Who can modify it?
     ↓
What does it trust?
     ↓
When does it execute?
     ↓
Can I influence it?
     ↓
Can I validate the path?
     ↓
Can I safely cross the boundary?
     ↓
Can I prove the result?
```

> **The objective of a privilege-escalation lab is not merely to reach root. The objective is to understand the system well enough to explain exactly why root was reachable.**
