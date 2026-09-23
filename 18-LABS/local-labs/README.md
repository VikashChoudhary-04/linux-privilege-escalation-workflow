# Local Linux Privilege Escalation Labs

> **Purpose:** Build, test, and document intentionally vulnerable Linux environments locally so privilege-escalation techniques can be practiced without relying on external platforms.

---

## Purpose

Local labs provide a controlled environment for testing individual privilege-escalation mechanisms.

They are especially useful when you want to isolate one concept:

```text id="7m4x2q"
Sudo
SUID
Capabilities
Cron
Services
Writable Files
Credentials
PATH Abuse
Docker
Kernel Vulnerabilities
```

Instead of learning several concepts simultaneously, a local lab can deliberately create one privilege boundary and allow it to be investigated from start to finish.

---

## Local Lab Philosophy

A good local lab should make the vulnerability understandable.

The objective is not:

```text id="8q3m6x"
Create a machine
      ↓
Hide the answer
      ↓
Find root
```

The objective is:

```text id="5x7m9p"
Create a Privilege Boundary
        ↓
Enumerate the System
        ↓
Discover the Boundary
        ↓
Analyze the Trust Relationship
        ↓
Validate the Weakness
        ↓
Exploit the Weakness
        ↓
Verify the Result
        ↓
Understand the Root Cause
```

This makes the lab reusable for learning and testing.

---

## Authorized Environment

Local labs should be isolated from systems that are not part of the exercise.

Recommended environments include:

* Virtual machines
* Containers
* Dedicated Linux test systems
* Host-only virtual networks
* Intentionally vulnerable applications
* Purpose-built privilege-escalation scenarios

Keep intentionally vulnerable configurations separate from production systems.

---

## Recommended Lab Structure

A local lab can be organized like this:

```text id="3x8m6q"
18-LABS/
└── local-labs/
    ├── README.md
    └── <lab-name>/
        └── README.md
```

Each lab should describe:

```text id="9m4q7x"
Objective
Environment
Configuration
Initial User
Intended Finding
Enumeration
Validation
Exploitation
Verification
Root Cause
Reset / Cleanup
Lessons Learned
```

---

## Lab Design Principles

### One Primary Learning Objective

A beginner lab should preferably have one main privilege boundary.

Example:

```text id="2x6m8q"
Root Cron Job
```

The learner should be able to understand:

```text id="7q3m5x"
Who runs the job?
What does it execute?
Who controls the executed resource?
When does it execute?
```

---

### Make the Privilege Boundary Explicit in the Design

Every lab should contain a clear relationship:

```text id="8m4q2x"
LOW PRIVILEGE USER
       ↓
USER-CONTROLLED RESOURCE
       ↓
PRIVILEGED COMPONENT
       ↓
PRIVILEGED ACTION
```

The learner's task is to discover this relationship through enumeration.

---

### Avoid Accidental Vulnerabilities

A lab should not contain unrelated weaknesses that make the intended path ambiguous unless the purpose of the exercise is multi-path analysis.

After creating a lab, test it from the perspective of the low-privileged user.

---

## Recommended Local Lab Categories

### Sudo Labs

Focus on:

```text id="6x9m3q"
sudo -l
Allowed commands
Command restrictions
Execution context
Trusted programs
```

Learning objective:

> Understand how delegated privileged execution can create a privilege boundary.

---

### SUID Labs

Focus on:

```text id="4m7q2x"
SUID binary
Owner
Permissions
Program behavior
User-controlled input
Privileged execution
```

Learning objective:

> Understand why a root-owned executable running with elevated privileges deserves investigation.

---

### Capability Labs

Focus on:

```text id="9x5m6q"
File capability
Binary behavior
Effective capability
User execution
Privilege impact
```

Learning objective:

> Understand how Linux capabilities can grant privileged operations without traditional SUID behavior.

---

### Cron Labs

Focus on:

```text id="3q8m5x"
Scheduled task
Execution user
Script ownership
Script permissions
Execution interval
User-controlled resource
```

Learning objective:

> Understand how trusted scheduled execution can cross a privilege boundary.

---

### Service Labs

Focus on:

```text id="7m2x9q"
Root service
Service definition
Executable
Configuration
Dependencies
Permissions
Restart / execution behavior
```

Learning objective:

> Understand how service configuration and execution dependencies can create escalation paths.

---

### Filesystem Labs

Focus on:

```text id="5x8q3m"
Writable files
Writable directories
Ownership
Permissions
Privileged consumers
Configuration files
Backups
```

Learning objective:

> Learn to distinguish arbitrary write access from write access that actually influences privileged behavior.

---

### Credential Labs

Focus on:

```text id="8m4x6q"
History
Configuration files
SSH material
Application secrets
Credential reuse
Account privilege
```

Learning objective:

> Understand how credentials can become an indirect privilege-escalation path.

---

### Execution-Abuse Labs

Focus on:

```text id="2q7m9x"
PATH
Wildcards
Environment variables
Libraries
Command substitution
Shell functions
```

Learning objective:

> Understand how privileged programs can trust attacker-controlled execution or environment behavior.

---

### Container Labs

Focus on:

```text id="6x3m8q"
Docker access
Container privileges
Runtime permissions
Host interaction
Privilege boundary
```

Learning objective:

> Understand when container-management access changes the effective privilege boundary of the host.

---

## Lab Difficulty Progression

A useful progression is:

```text id="9m5q2x"
Level 1
Single Misconfiguration
        ↓
Level 2
Single Privilege Mechanism
        ↓
Level 3
Multiple Candidates
        ↓
Level 4
Multi-Step Attack Path
        ↓
Level 5
Chained Privilege Escalation
```

Difficulty should come from reasoning rather than unnecessary complexity.

---

## Standard Lab Workflow

### Step 1 — Start as the Low-Privileged User

Record:

```bash id="4x8m6q"
whoami
id
hostname
```

Then:

```bash id="7m3q9x"
uname -a
cat /etc/os-release
```

---

### Step 2 — Perform Baseline Enumeration

Check:

```text id="5q2x8m"
Users
Groups
Sudo
Processes
Services
SUID
SGID
Capabilities
Cron
Filesystem
Credentials
Network
Special Environments
```

Do not reveal the intended vulnerability directly to the learner.

---

### Step 3 — Identify Candidates

Record observations such as:

```text id="8m4q7x"
Root-owned script
Writable by current user
Executed periodically
```

or:

```text id="3x6m9q"
Root-owned SUID binary
Current user can execute it
Program behavior appears controllable
```

---

### Step 4 — Analyze the Trust Relationship

Ask:

```text id="6q8m2x"
Who owns it?
Who executes it?
What does it trust?
Who controls the trusted resource?
When is it executed?
```

This is the central learning step.

---

### Step 5 — Validate

Prove the relevant relationship before exploitation.

```text id="9x3m5q"
Finding
   ↓
User Control
   ↓
Privileged Consumer
   ↓
Execution
   ↓
Privilege Impact
```

---

### Step 6 — Exploit

Perform only the controlled action necessary to demonstrate the vulnerability.

Record:

```text id="2m7q8x"
Technique:
Command/action:
Expected result:
Observed result:
```

---

### Step 7 — Verify

Run:

```bash id="5x9m3q"
whoami
id
id -u
```

Confirm the actual privilege level.

---

### Step 8 — Reset the Lab

A reusable lab should be recoverable.

Restore:

```text id="7q4m2x"
Modified files
Permissions
Services
Cron entries
Accounts
Credentials
Temporary artifacts
Container state
```

The reset process should return the environment to its original vulnerable state.

---

## Lab Write-Up Template

Use this structure for individual local labs:

```text id="8m3x6q"
# Lab Name

## Objective

What the lab teaches.

## Environment

OS:
Version:
Architecture:
Virtualization / Container:

## Initial User

Username:
UID:
Groups:

## Intended Vulnerability

High-level description without revealing unnecessary implementation details.

## Enumeration

Commands:
Observations:

## Candidate Finding

Finding:
Owner:
Permissions:
Privileged context:
User control:
Execution trigger:

## Validation

Validation steps:
Expected result:
Observed result:

## Exploitation

Technique:
Action:
Result:

## Verification

whoami:
id:
UID:

## Root Cause

Why the vulnerability exists.

## Reset

How to restore the lab.

## Lessons Learned

Key concepts:

## Repository Mapping

Relevant workflow sections:
```

---

## Builder's Checklist

When creating a new local lab:

```text id="4x7m9q"
[ ] Learning objective defined
[ ] Low-privileged user created
[ ] Privilege boundary created
[ ] Intended privileged component identified
[ ] User-controlled resource identified
[ ] Trigger defined
[ ] Expected attack path documented
[ ] Lab tested from low privilege
[ ] Exploitation validated
[ ] Verification method defined
[ ] Reset procedure tested
[ ] No unintended escalation path introduced
[ ] Documentation prepared
```

---

## Testing the Lab Before Publishing

Test every lab from a clean state.

### Test 1 — Initial State

Confirm:

```text id="6m2q8x"
[ ] Initial user has expected privileges
[ ] Intended configuration exists
[ ] Lab starts correctly
```

### Test 2 — Enumeration

Ask:

> Can the vulnerability be discovered through the repository workflow?

### Test 3 — Validation

Ask:

> Can the learner prove the finding before exploiting it?

### Test 4 — Exploitation

Ask:

> Does the intended technique produce the expected privilege change?

### Test 5 — Verification

Ask:

> Is the resulting privilege unambiguous?

### Test 6 — Reset

Ask:

> Can the lab be returned to its original state?

---

## Multi-Path Labs

More advanced labs may intentionally contain multiple findings.

Example:

```text id="7x5m3q"
Candidate A
   ↓
False Positive

Candidate B
   ↓
Valid Finding
   ↓
Partial Attack Path

Candidate C
   ↓
Valid Finding
   ↓
Complete Attack Path
```

These labs teach prioritization and validation.

The objective becomes:

> Find the complete attack path, not merely an interesting configuration.

---

## Chained Labs

Advanced labs can require multiple stages:

```text id="9m4q6x"
Initial User
    ↓
Privilege Boundary #1
    ↓
Intermediate User
    ↓
Privilege Boundary #2
    ↓
Root
```

Each transition should be independently explainable.

For every stage document:

```text id="3x8m2q"
Current User
↓
Privileged Resource
↓
Control
↓
Execution
↓
New Privilege
```

---

## Reset Philosophy

A vulnerable lab should be deliberately reproducible.

Prefer reset mechanisms that restore the known state rather than manually guessing which changes need to be reversed.

Useful approaches include:

```text id="5q7m9x"
VM Snapshot
Container Recreation
Configuration Reset
Scripted Reset
Filesystem Restore
```

The chosen method depends on the lab environment.

---

## Safety Rules

```text id="8x3m6q"
[ ] Keep vulnerable systems isolated
[ ] Do not expose intentionally vulnerable services unnecessarily
[ ] Do not reuse real credentials
[ ] Do not use production data
[ ] Do not connect test privileges to real infrastructure
[ ] Keep destructive testing inside the lab
[ ] Reset the environment after testing
```

---

## Suggested Local Lab Roadmap

Build labs in increasing conceptual difficulty.

```text id="2m8q4x"
01 — Sudo Misconfiguration
        ↓
02 — SUID Binary
        ↓
03 — Writable Cron Script
        ↓
04 — Linux Capability
        ↓
05 — Root Service
        ↓
06 — Credential Discovery
        ↓
07 — PATH / Execution Abuse
        ↓
08 — Docker Privilege Boundary
        ↓
09 — Multiple Findings
        ↓
10 — Chained Attack Path
```

The exact order can change as the repository grows.

---

## Lessons to Extract From Every Lab

After completing a lab, answer:

```text id="6x9m2q"
What did I initially observe?

What command exposed the finding?

Why was the finding interesting?

Who owned the privileged component?

Who could influence it?

What did it trust?

When did it execute?

What proved exploitability?

How was privilege verified?

What was the root cause?

What would I check first on another machine?
```

The last question is especially important.

It converts an individual lab into reusable methodology.

---

## Lab Completion Checklist

```text id="4m7x8q"
[ ] Objective understood
[ ] Environment understood
[ ] Initial user documented
[ ] Initial privilege documented
[ ] System context recorded
[ ] Enumeration completed
[ ] Candidate findings recorded
[ ] Privileged component identified
[ ] User control identified
[ ] Trust relationship understood
[ ] Execution trigger understood
[ ] Finding validated
[ ] Exploitation completed
[ ] Final privilege verified
[ ] Root cause documented
[ ] Evidence preserved
[ ] Lab reset
[ ] Lessons learned recorded
```

---

## Final Mental Model

Local labs should reinforce one principle:

```text id="9q3m6x"
PRIVILEGE ESCALATION
is not
"finding a magic command"

It is:

OBSERVE
  ↓
UNDERSTAND
  ↓
IDENTIFY PRIVILEGE
  ↓
IDENTIFY CONTROL
  ↓
IDENTIFY TRUST
  ↓
TRACE EXECUTION
  ↓
VALIDATE
  ↓
EXPLOIT
  ↓
VERIFY
  ↓
EXPLAIN
```

> **The strongest local lab is one where the learner can discover the privilege boundary, prove why it is exploitable, reproduce the result, reset the environment, and explain the root cause without relying on a walkthrough.**
