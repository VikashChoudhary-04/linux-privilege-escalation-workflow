# Linux Privilege Escalation Methodology

> **Purpose:** Turn the individual enumeration and exploitation techniques in this repository into one repeatable, decision-driven Linux privilege-escalation methodology.

---

## Core Question

The methodology should answer:

> **Given a low-privileged Linux shell, what should I investigate first, why should I investigate it, and when should I move on?**

A good methodology prevents two common problems:

```text
Random enumeration
       ↓
Huge amount of output
       ↓
No clear attack path
```

and:

```text
Memorized exploit
       ↓
Doesn't match target
       ↓
Guessing
```

Instead, use:

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

---

## Master Mental Model

Every privilege-escalation investigation revolves around five questions:

```text
1. WHO AM I?

2. WHAT IS RUNNING?

3. WHO OWNS IT?

4. WHO CAN MODIFY IT?

5. CAN I INFLUENCE A PRIVILEGED ACTION?
```

These questions apply to almost every escalation mechanism.

For example:

```text
Root cron job
    ↓
What does it execute?
    ↓
Who owns the script?
    ↓
Can I modify it?
    ↓
Will root execute my modification?
```

Or:

```text
SUID binary
    ↓
What binary is privileged?
    ↓
What does it do?
    ↓
What input can I control?
    ↓
Can that control cross the privilege boundary?
```

---

## Complete Methodology

The repository follows this overall sequence:

```text
01 First 5 Minutes
        ↓
02 System Enumeration
        ↓
03 Filesystem Enumeration
        ↓
04 Process & Service Enumeration
        ↓
05 Network Enumeration
        ↓
06 Privilege Mechanisms
        ↓
07 Scheduled Execution
        ↓
08 Credentials & Secrets
        ↓
09 Execution Abuse
        ↓
10 Special Environments
        ↓
11 Vulnerable Software
        ↓
12 Automated Enumeration
        ↓
13 Finding Validation
        ↓
14 Exploitation
        ↓
15 Root Verification
```

This is a **decision framework**, not a requirement to blindly execute every command in every environment.

---

## Enumeration Philosophy

### Enumeration Is Question-Driven

Do not think:

```text
"What commands do I know?"
```

Think:

```text
"What do I need to know?"
```

Then select the command that answers that question.

For example:

```text
Question:
What identity am I?

Command:
whoami
```

```text
Question:
What groups do I belong to?

Command:
id
```

```text
Question:
What processes are running?

Command:
ps aux
```

The command is secondary.

The question comes first.

---

## Enumeration in Layers

Use four broad layers:

```text
Layer 1 — Context
Layer 2 — Attack Surface
Layer 3 — Trust Relationships
Layer 4 — Validation
```

### Context

Determine:

```text
User
Groups
OS
Kernel
Architecture
Hostname
Environment
```

### Attack Surface

Look for:

```text
Files
Processes
Services
Scheduled tasks
Network services
SUID
Capabilities
Sudo
Containers
Credentials
Software versions
```

### Trust Relationships

Ask:

```text
Who executes this?
Who owns this?
Who can modify this?
What does this process trust?
What input does it trust?
```

### Validation

Determine:

```text
Can the current user influence the privileged action?
```

---

## Finding Prioritization

Not every finding deserves equal attention.

Prioritize investigation using:

```text
Privilege impact
+
User control
+
Reliability
+
Clarity
+
Ease of validation
```

For example:

```text
Root executes script
        +
Current user can modify script
        =
Strong attack-path candidate
```

Compare that with:

```text
Old package version
        +
Unknown patch state
        +
Unknown exploitability
        =
Requires further investigation
```

The purpose of prioritization is to spend time where the evidence is strongest.

---

## Finding vs Attack Path

### Finding

A finding is an individual observation:

```text
Writable file
```

or:

```text
SUID binary
```

or:

```text
Old package version
```

A finding alone does not prove privilege escalation.

### Attack Path

An attack path connects the finding to a privileged action:

```text
Writable file
      ↓
Root service reads file
      ↓
Service executes controlled content
      ↓
Higher privilege
```

The relationship between the components is what matters.

---

## Trust Relationship Analysis

For every interesting privileged object, ask:

```text
Who trusts it?
```

Then:

```text
Who controls it?
```

Then:

```text
Can a lower-privileged user influence it?
```

This pattern works across:

```text
Cron
Systemd
Sudo
SUID
Services
PATH
Libraries
Configuration files
Containers
Credentials
```

---

## Privilege Boundary Analysis

### Identify the Current Boundary

Determine:

```text
Current UID
Current groups
Current privileges
```

Use:

```bash
whoami
id
```

### Identify the Target Boundary

Ask:

```text
What higher privilege am I trying to reach?
```

Usually:

```text
UID 0 / root
```

But other privileged identities may matter depending on the target.

### Identify the Crossing Mechanism

Ask:

> **What exact action moves me from the current privilege level to the target privilege level?**

That is the core of the attack path.

---

## Decision-Driven Enumeration

Use:

```text
Question
   ↓
Command
   ↓
Result
   ↓
Interpret
   ↓
Decision
   ↓
Next question
```

Example:

```text
Check sudo
   ↓
Sudo permission exists?
   │
   ├── YES → Analyze permission
   │
   └── NO  → Move to next mechanism
```

Another:

```text
Find SUID binary
   ↓
Unusual?
   │
   ├── YES → Analyze behavior
   │
   └── NO  → Continue
```

---

## The Escalation Loop

For every interesting finding:

```text
FOUND
  ↓
WHO OWNS IT?
  ↓
WHO EXECUTES IT?
  ↓
WHO CAN MODIFY IT?
  ↓
WHEN IS IT USED?
  ↓
WHAT DOES IT TRUST?
  ↓
CAN I INFLUENCE IT?
  ↓
VALIDATE
```

If the answer is no:

```text
MOVE ON
```

If the answer is yes:

```text
INVESTIGATE
      ↓
VALIDATE
```

Only after validation should exploitation begin.

---

## Stop Conditions

Knowing when to stop investigating a path is an important skill.

Move on when there is:

```text
No privileged relationship
No user control
No relevant execution path
No applicable vulnerability
No exploitable trust relationship
```

Do not spend excessive time forcing an unpromising finding.

---

## Complete Workflow

### Phase 1 — Establish Context

Start with:

```bash
whoami
id
hostname
uname -a
cat /etc/os-release
```

Determine:

```text
Identity
Groups
Hostname
OS
Kernel
Architecture
```

### Phase 2 — Understand the Environment

Investigate:

```text
Processes
Services
Network
Filesystem
Environment
Installed software
```

The objective is to understand what the machine actually does.

### Phase 3 — Find Privileged Execution

Look for:

```text
Sudo
SUID
Capabilities
Root processes
Root services
Cron
Systemd timers
Privileged containers
```

The key question is:

> **What executes with more privilege than I currently have?**

### Phase 4 — Find User Control

Once a privileged component is found, determine:

```text
Can I modify it?
Can I influence its input?
Can I control a referenced file?
Can I control its execution environment?
Can I influence a command it runs?
Can I influence a dependency?
```

This is where an interesting finding becomes an attack-path candidate.

### Phase 5 — Validate

Confirm:

```text
Privileged execution
+
User control
+
Trusted relationship
+
Required timing/preconditions
```

Then determine whether the path actually crosses the privilege boundary.

### Phase 6 — Exploit

Use the minimum effective technique.

Follow:

```text
Validated finding
      ↓
Controlled exploitation
      ↓
Privilege verification
```

### Phase 7 — Verify

Run:

```bash
whoami
id
id -u
```

Compare against the original baseline.

### Phase 8 — Document

Record:

```text
Finding
Root cause
Validation
Exploitation
Privilege gained
Evidence
Remediation
```

---

## Enumeration Priority

When time is limited, use a priority model.

### Priority 1 — Direct Privilege Mechanisms

Check:

```text
sudo
SUID
Capabilities
Special groups
```

These can provide relatively direct privilege boundaries.

### Priority 2 — Privileged Automation

Check:

```text
Cron
Systemd timers
Root services
Scheduled scripts
```

### Priority 3 — User-Controlled Execution

Check:

```text
PATH
Writable scripts
Writable service files
Configuration
Command substitution
Library loading
```

### Priority 4 — Credentials

Check:

```text
History
Configuration
SSH
Application secrets
Environment variables
Backups
```

### Priority 5 — Special Environments

Check:

```text
Docker
LXD
Containers
NFS
Mounted filesystems
Privileged sockets
```

### Priority 6 — Vulnerable Software

Investigate:

```text
Running services
Installed software
Kernel
Relevant vulnerabilities
```

The order can change depending on the target.

---

## Time-Boxed Enumeration

When working under a time constraint, divide the investigation into phases.

For example:

```text
0–5 minutes
    ↓
Context + identity

5–10 minutes
    ↓
Processes + services + sudo

10–15 minutes
    ↓
SUID + capabilities + cron

15–20 minutes
    ↓
Filesystem + credentials + network

20+ minutes
    ↓
Special environments + vulnerable software + deeper analysis
```

The exact timing is flexible.

The principle is:

> **Do not spend the entire assessment on one uncertain lead before checking obvious privilege mechanisms.**

---

## Manual vs Automated Enumeration

Automation should accelerate methodology, not replace it.

Use:

```text
Manual enumeration
      +
Automated enumeration
      ↓
Candidate findings
      ↓
Manual validation
```

Useful enumeration tools include:

```text
LinPEAS
Linux Smart Enumeration
pspy
```

But their output is not automatically proof.

---

## Tool Output Methodology

When an automated tool highlights something:

```text
Tool finding
    ↓
Understand what it detected
    ↓
Reproduce manually
    ↓
Determine privilege context
    ↓
Determine user control
    ↓
Validate exploitability
```

Never use:

```text
Tool says vulnerable
    ↓
Automatically exploit
```

---

## False Positive Handling

A finding becomes a false positive when the apparent path does not actually provide the expected privilege impact.

Examples:

```text
SUID binary
    ↓
No controllable privileged behavior
```

```text
Writable file
    ↓
No privileged consumer
```

```text
Old package
    ↓
Distribution patch already applied
```

```text
Container-related access
    ↓
No demonstrated path across the relevant privilege boundary
```

Record useful false positives.

They improve future enumeration decisions.

---

## Attack Path Analysis

When multiple findings exist, connect them into possible chains.

Use:

```text
Current privilege
      ↓
User-controlled condition
      ↓
Trusted privileged component
      ↓
Privileged action
      ↓
Higher privilege
```

For example:

```text
Low-privileged shell
      ↓
Writable script
      ↓
Root cron job
      ↓
Script execution
      ↓
Root
```

The important question is:

> **What exact relationship connects the current user to the privileged action?**

---

## Attack Path Prioritization

When multiple validated or promising paths exist, consider:

```text
Reliability
Privilege impact
Number of prerequisites
Environmental dependencies
Potential system impact
Evidence quality
Reproducibility
```

The goal is not to blindly select the most complicated path.

The goal is to understand the available paths and choose an appropriate authorized testing path.

---

## Methodology Decision Tree

```text
START
  │
  ▼
Establish identity
  │
  ▼
Understand host
  │
  ▼
What privileged mechanisms exist?
  │
  ├── Sudo ───────────────► Analyze
  │
  ├── SUID ───────────────► Analyze
  │
  ├── Capabilities ───────► Analyze
  │
  ├── Cron/Timers ────────► Analyze
  │
  ├── Services ───────────► Analyze
  │
  ├── Credentials ────────► Analyze
  │
  ├── Containers ─────────► Analyze
  │
  └── Vulnerable software ► Analyze
                              │
                              ▼
                       User control?
                              │
                    ┌─────────┴─────────┐
                    │                   │
                   NO                  YES
                    │                   │
                 Move on             Validate
                                        │
                                        ▼
                                  Exploitable?
                                        │
                              ┌─────────┴─────────┐
                              │                   │
                             NO                  YES
                              │                   │
                           Move on             Exploit
                                                  │
                                                  ▼
                                               Verify
                                                  │
                                                  ▼
                                             Document
```

---

## Common Methodology Mistakes

### Running Commands Without a Question

Bad:

```text
Run every command I know.
```

Better:

```text
What do I need to know?
        ↓
Choose command
        ↓
Interpret result
        ↓
Decide next step
```

### Treating Every Finding as Vulnerable

Remember:

```text
Finding
≠
Vulnerability
≠
Exploitable Path
```

### Staying Too Long on One Lead

If a finding cannot establish:

```text
Privileged action
+
User control
```

move on.

### Ignoring Relationships

A writable file alone may mean nothing.

The important question is:

```text
Who uses it?
```

### Trusting Automated Tools Blindly

Tools identify candidates.

You prove the attack path.

### Exploiting Without Evidence

A successful exploit should leave you able to explain:

```text
What
Why
How
Result
```

---

## Five-Minute Methodology

When first landing on a Linux host:

```bash
whoami
id
hostname
uname -a
cat /etc/os-release
sudo -l
```

Then quickly investigate:

```text
Processes
Services
SUID
Capabilities
Cron
Interesting writable files
Credentials
Containers
```

The objective is not to finish the assessment in five minutes.

The objective is to build an initial map.

---

## Fifteen-Minute Methodology

Use this sequence:

```text
0–5 min
Context + identity + OS

5–10 min
Sudo + processes + services + SUID

10–15 min
Capabilities + cron + filesystem + credentials
```

Then prioritize the strongest candidate paths.

---

## Methodology Under Pressure

When stuck, return to the core questions:

```text
WHO AM I?
     ↓
WHAT IS PRIVILEGED?
     ↓
WHO OWNS IT?
     ↓
WHO CAN MODIFY IT?
     ↓
WHEN DOES IT EXECUTE?
     ↓
WHAT DOES IT TRUST?
     ↓
CAN I CONTROL IT?
```

If you cannot answer these questions, you probably need more enumeration.

---

## Methodology for Labs

For training environments, focus on learning the reasoning.

After solving a lab, ask:

```text
What did I notice?
Why did it matter?
What command exposed it?
What assumption did I make?
What confirmed it?
What actually crossed the boundary?
What would I check first next time?
```

This turns individual machine solutions into reusable knowledge.

---

## Methodology for Professional Assessments

In authorized professional environments, add:

```text
Scope verification
Evidence collection
Change tracking
Risk awareness
Minimal-impact testing
Clear reporting
Remediation guidance
```

The technical workflow remains:

```text
Enumerate
   ↓
Validate
   ↓
Exploit
   ↓
Verify
```

Operational discipline becomes equally important.

---

## Attack Path Documentation

Use this structure for every confirmed path:

```text
Initial Access
    ↓
Current Identity
    ↓
Enumeration Finding
    ↓
Privileged Component
    ↓
User-Controlled Element
    ↓
Validation
    ↓
Exploitation
    ↓
Privilege Verification
    ↓
Root Cause
    ↓
Remediation
```

This creates a reusable case study.

---

## Methodology Checklist

```text
[ ] Establish current identity
[ ] Establish current privileges
[ ] Identify OS and kernel
[ ] Understand running processes
[ ] Understand services
[ ] Check sudo
[ ] Check SUID
[ ] Check capabilities
[ ] Check scheduled execution
[ ] Check writable privileged resources
[ ] Check credentials and secrets
[ ] Check special environments
[ ] Check relevant software vulnerabilities
[ ] Run automated enumeration
[ ] Validate important findings
[ ] Select an appropriate exploitation path
[ ] Verify privilege change
[ ] Preserve evidence
[ ] Document root cause
[ ] Document remediation
```

---

## Final Mental Model

The complete methodology can be remembered as:

```text
UNDERSTAND
    ↓
ENUMERATE
    ↓
RECOGNIZE
    ↓
INVESTIGATE
    ↓
VALIDATE
    ↓
EXPLOIT
    ↓
VERIFY
    ↓
DOCUMENT
```

Or even more simply:

```text
WHAT EXISTS?
     ↓
WHAT IS PRIVILEGED?
     ↓
WHAT CAN I CONTROL?
     ↓
CAN I CROSS THE BOUNDARY?
     ↓
CAN I PROVE IT?
```

The strongest Linux privilege-escalation methodology is not the one containing the most commands.

It is the one that consistently answers:

> **What am I looking at, why does it matter, what can I control, and what should I investigate next?**

That is the difference between **running enumeration commands** and **performing privilege-escalation analysis**.
