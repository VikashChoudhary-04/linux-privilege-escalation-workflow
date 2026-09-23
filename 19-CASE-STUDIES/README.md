# Linux Privilege Escalation Case Studies

> **Purpose:** Break common Linux privilege-escalation attack paths into focused, realistic case studies that demonstrate how individual findings become complete, validated attack paths.

---

## Purpose

The workflow sections explain **how to investigate**.

The lab section provides **hands-on practice**.

Case studies explain **why specific privilege-escalation paths work**.

Each case study isolates one technique and follows it from:

```text id="7m4q2x"
Initial Context
      ↓
Enumeration
      ↓
Finding
      ↓
Analysis
      ↓
Validation
      ↓
Exploitation
      ↓
Verification
      ↓
Root Cause
```

The objective is not to memorize individual exploits.

The objective is to recognize the same underlying pattern on unfamiliar systems.

---

## Case Study Structure

This section contains focused examples of common privilege-escalation mechanisms.

```text id="5x8m3q"
19-CASE-STUDIES/
│
├── README.md
├── sudo-misconfiguration.md
├── suid-binary.md
├── cron-job.md
├── capabilities.md
└── credential-discovery.md
```

Each case study should be understandable independently while still connecting back to the main workflow.

---

## What Makes a Good Case Study

A useful case study should answer:

```text id="9q4m7x"
What was the initial privilege?
What was discovered?
Why was it interesting?
Who owned the privileged component?
Who could influence it?
What did it trust?
When did it execute?
How was the finding validated?
How was the privilege boundary crossed?
How was the result verified?
Why did the vulnerability exist?
```

---

## Standard Case Study Workflow

Every case study should follow the same reasoning model.

### 1. Establish Context

```bash id="3x7m9q"
whoami
id
hostname
```

Record the initial privilege level.

---

### 2. Identify the Privileged Component

Determine what operates with greater privilege.

Examples:

```text id="8m5q2x"
sudo command
SUID binary
root cron job
capability-enabled binary
credential belonging to privileged account
```

---

### 3. Identify User Control

Ask:

> What part of this execution path can the current user influence?

Examples:

```text id="6q3x8m"
Command arguments
Input files
Environment
PATH
Script
Configuration
Credential
Executable
Working directory
```

---

### 4. Analyze Trust

Determine why the privileged component accepts the controlled input.

```text id="4m9q5x"
USER CONTROL
     ↓
TRUSTED INPUT
     ↓
PRIVILEGED COMPONENT
```

---

### 5. Identify the Trigger

Determine how the privileged action occurs.

```text id="7x2m6q"
Manual execution
Scheduled execution
Service startup
Authentication
Program invocation
Container management
```

---

### 6. Validate

Prove the attack path before exploitation.

```text id="5q8m3x"
Can I control it?
      ↓
Does the privileged component use it?
      ↓
Does execution occur with higher privilege?
      ↓
Does that influence cross the privilege boundary?
```

---

### 7. Exploit

Perform the minimum controlled action necessary to demonstrate the vulnerability.

---

### 8. Verify

Confirm the resulting privilege level:

```bash id="9m4x7q"
whoami
id
id -u
```

---

### 9. Explain the Root Cause

The final explanation should identify the underlying security failure.

Examples:

```text id="2x6m8q"
Excessive sudo delegation
Insecure SUID program
Writable root-executed script
Overly powerful capability
Exposed privileged credential
```

---

## Case Study 1 — Sudo Misconfiguration

File:

```text id="6m3q9x"
sudo-misconfiguration.md
```

Core pattern:

```text id="8x5m2q"
Low-Privilege User
      ↓
Sudo Permission
      ↓
Privileged Command
      ↓
Unsafe / Excessive Capability
      ↓
Privilege Escalation
```

Questions to investigate:

```text id="4q7m9x"
What does sudo -l allow?
Who can run the command?
As which user?
Are restrictions present?
What does the allowed program actually do?
Can its behavior cross the privilege boundary?
```

Relevant workflow sections:

```text id="7m3x8q"
06-PRIVILEGE-MECHANISMS
13-FINDING-VALIDATION
14-EXPLOITATION
15-ROOT-VERIFICATION
```

---

## Case Study 2 — SUID Binary

File:

```text id="5x9q3m"
suid-binary.md
```

Core pattern:

```text id="2m8x6q"
User
 ↓
SUID Binary
 ↓
Root-Owned Execution
 ↓
Privileged Behavior
 ↓
Privilege Impact
```

Questions:

```text id="9q4m5x"
Who owns the binary?
Why does it have SUID?
What functionality does it expose?
Can the current user execute it?
What inputs can the user control?
Does the controlled behavior execute with elevated privilege?
```

Relevant workflow sections:

```text id="6x3m8q"
06-PRIVILEGE-MECHANISMS
13-FINDING-VALIDATION
14-EXPLOITATION
```

---

## Case Study 3 — Writable Cron Job

File:

```text id="8m2q7x"
cron-job.md
```

Core pattern:

```text id="3x9m5q"
User
 ↓
Writable Script
 ↓
Cron
 ↓
Root Execution
 ↓
Privilege Escalation
```

Questions:

```text id="5q7m3x"
Who owns the cron job?
Who executes it?
What file does it execute?
Can the current user modify that file?
When does it run?
Can the effect be observed safely?
```

Relevant workflow sections:

```text id="7m4x9q"
07-SCHEDULED-EXECUTION
13-FINDING-VALIDATION
14-EXPLOITATION
```

---

## Case Study 4 — Linux Capability

File:

```text id="4x8m6q"
capabilities.md
```

Core pattern:

```text id="9m3q5x"
User
 ↓
Capability-Enabled Binary
 ↓
Privileged Operation
 ↓
Privilege Boundary
```

Questions:

```text id="6q2m8x"
Which capability is present?
Which binary has it?
Why does the binary need it?
Can the current user execute the binary?
What privileged operation can the capability enable?
Does that operation affect the privilege boundary?
```

Relevant workflow sections:

```text id="5x7m9q"
06-PRIVILEGE-MECHANISMS
13-FINDING-VALIDATION
14-EXPLOITATION
```

---

## Case Study 5 — Credential Discovery

File:

```text id="8q4m2x"
credential-discovery.md
```

Core pattern:

```text id="7m3x9q"
Low-Privilege User
      ↓
Credential Discovery
      ↓
Credential Reuse
      ↓
Higher-Privilege Account
      ↓
Privilege Escalation
```

Questions:

```text id="5x8m4q"
Where was the credential found?
Who does it belong to?
What services accept it?
What privilege does that account have?
Is reuse authorized within the lab?
Can the resulting privilege be verified?
```

Relevant workflow sections:

```text id="9m4q7x"
08-CREDENTIALS-AND-SECRETS
13-FINDING-VALIDATION
14-EXPLOITATION
15-ROOT-VERIFICATION
```

---

## Finding vs Attack Path

A case study should clearly distinguish:

```text id="2x6m9q"
Finding
```

from:

```text id="7q3m5x"
Attack Path
```

Example:

```text id="8m4x2q"
Finding:
A script is writable by the current user.

Attack Path:
The script is executed by a root-owned cron job,
allowing controlled content to execute with root privilege.
```

The second statement establishes the complete relationship.

---

## Validation Before Exploitation

Every case study should demonstrate this sequence:

```text id="5m7q3x"
OBSERVE
  ↓
INVESTIGATE
  ↓
VALIDATE
  ↓
EXPLOIT
```

Avoid writing case studies that jump directly from:

```text id="9x4m6q"
"Found X"
```

to:

```text id="3m8q5x"
"Ran exploit Y"
```

The missing analysis is where most of the useful learning occurs.

---

## Root Cause Analysis

The case study should finish by answering:

> Why was this possible?

A strong root-cause explanation identifies the configuration or design weakness.

Examples:

| Technique   | Possible Root Cause                         |
| ----------- | ------------------------------------------- |
| Sudo        | Excessive privileged delegation             |
| SUID        | Unsafe privileged program behavior          |
| Cron        | Root-executed writable resource             |
| Capability  | Excessive capability assigned to executable |
| Credentials | Secret exposure or unsafe credential reuse  |

The exact root cause must be based on the case rather than assumed from the technique.

---

## Evidence Model

Each case study should preserve enough evidence to support the conclusion.

Useful evidence includes:

```text id="6x2m9q"
Initial identity
Relevant permissions
Ownership
Sudo configuration
SUID/capability output
Cron/service configuration
Credential location
Validation result
Final identity
```

Avoid publishing unnecessary secrets.

Use sanitized values when a case study contains credentials or tokens.

---

## Case Study Documentation Template

Use this structure for the individual files:

```text id="4m8x7q"
# Case Study Name

## Scenario

Describe the controlled environment.

## Initial Context

Initial user:
Initial privilege:
Target:

## Enumeration

Commands:
Observations:

## Finding

What was discovered?

## Privilege Boundary

Privileged component:
Owner:
Execution context:

## User Control

What can the current user influence?

## Trust Relationship

What does the privileged component trust?

## Trigger

How does the privileged action occur?

## Validation

How was the finding confirmed?

## Exploitation

How was the validated weakness demonstrated?

## Verification

whoami:
id:
UID:

## Root Cause

Why was the system vulnerable?

## Evidence

Important evidence:

## Cleanup

Actions taken:

## Methodology Mapping

Relevant repository sections.

## Lessons Learned

Key lessons:

## Final Mental Model

Short explanation of the attack path.
```

---

## Comparing Case Studies

The techniques differ, but the reasoning pattern remains similar.

| Technique  | Privileged Component      | User-Controlled Element | Trigger             |
| ---------- | ------------------------- | ----------------------- | ------------------- |
| Sudo       | Sudo-authorized program   | Command/input           | User invocation     |
| SUID       | SUID executable           | Program input/behavior  | Program execution   |
| Cron       | Cron job                  | Script/resource         | Scheduled execution |
| Capability | Capability-enabled binary | Program input/behavior  | Program execution   |
| Credential | Privileged account        | Credential use          | Authentication      |

The important common factor is:

```text id="8m5q3x"
USER CONTROL
     ↓
TRUST
     ↓
PRIVILEGED ACTION
```

---

## Case Study Learning Cycle

After reading or completing a case study:

```text id="7x4m2q"
1. Identify the privileged component
2. Identify user control
3. Identify trust
4. Identify execution trigger
5. Predict the attack path
6. Validate the prediction
7. Verify the result
8. Explain the root cause
```

Then ask:

> What would I enumerate first if this mechanism appeared on an unfamiliar machine?

---

## Common Mistakes

### Memorizing Exploit Commands

Commands are implementation details.

The underlying relationship matters more.

---

### Ignoring Ownership

Always determine:

```text id="5m8q2x"
Owner
Group
Permissions
Execution User
```

---

### Ignoring Execution Timing

Scheduled and service-based attacks depend on when the privileged action occurs.

---

### Assuming Write Access Is Enough

A writable resource matters only if a privileged consumer uses it in a meaningful way.

---

### Skipping Validation

A plausible finding is not automatically an exploitable attack path.

---

### Failing to Explain Root Cause

A case study should teach why the vulnerability existed, not merely how it was exploited.

---

## Case Study Completion Checklist

```text id="9q3m6x"
[ ] Scenario defined
[ ] Initial privilege documented
[ ] Privileged component identified
[ ] User-controlled element identified
[ ] Ownership documented
[ ] Permissions documented
[ ] Trust relationship explained
[ ] Trigger identified
[ ] Finding validated
[ ] Exploitation documented
[ ] Final privilege verified
[ ] Root cause explained
[ ] Evidence recorded
[ ] Cleanup documented
[ ] Methodology mapping added
[ ] Lessons learned recorded
```

---

## Repository Integration

Case studies connect the repository's major layers:

```text id="6x8m4q"
FOUNDATIONS
     ↓
WORKFLOW
     ↓
METHODOLOGY
     ↓
QUICK REFERENCE
     ↓
LABS
     ↓
CASE STUDIES
```

The same technique can therefore be understood at multiple levels:

```text id="3m7q9x"
WHY?
Foundations

WHAT?
Workflow

WHEN / WHY NEXT?
Methodology

HOW FAST?
Quick Reference

CAN I PRACTICE IT?
Labs

CAN I UNDERSTAND THE FULL PATH?
Case Study
```

---

## Final Mental Model

The five initial case studies demonstrate different mechanisms, but they all reduce to the same question:

```text id="8q5m2x"
What privileged action exists?
          ↓
What does it trust?
          ↓
Who can influence that trust?
          ↓
When does the privileged action occur?
          ↓
Can the influence cross the privilege boundary?
          ↓
Can I validate it safely?
          ↓
Can I verify the result?
```

> **The technique changes. The reasoning does not.**

That is the central lesson of the case-study section.
