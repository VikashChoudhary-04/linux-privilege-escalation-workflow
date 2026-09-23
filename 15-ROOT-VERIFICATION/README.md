# Root Verification Workflow

> **Purpose:** Prove that privilege escalation succeeded, capture reproducible evidence, document the exact attack path, and safely return the environment to an appropriate state.

---

## Core Question

After exploitation, do not assume success.

Ask:

> **Did my privilege actually change, and can I prove exactly how it changed?**

The correct workflow is:

```text
Exploitation
     ↓
Verify Identity
     ↓
Verify Privilege
     ↓
Collect Evidence
     ↓
Document Attack Path
     ↓
Cleanup
```

---

## Verification Mental Model

Use this chain:

```text
WHO AM I?
    ↓
WHAT GROUPS DO I HAVE?
    ↓
WHAT UID/GID DO I HAVE?
    ↓
WHAT PRIVILEGES DO I CURRENTLY POSSESS?
    ↓
WHAT CHANGED?
    ↓
CAN I ATTRIBUTE THE CHANGE TO THE VALIDATED FINDING?
```

The final question is important.

You should be able to explain:

> **This privilege increase occurred because this specific validated weakness allowed this specific privileged action.**

---

## Step 1 — Establish the Current Identity

Run:

```bash
whoami
```

Then:

```bash
id
```

And:

```bash
id -u
id -g
```

Useful information includes:

```text
Username
UID
Primary GID
Supplementary groups
```

### Expected Result

A successful root escalation commonly produces:

```text
whoami → root
UID    → 0
```

But do not rely on `whoami` alone.

Use multiple indicators.

---

## Step 2 — Verify UID 0

Run:

```bash
id -u
```

A result of:

```text
0
```

means the current process has UID 0.

Verify the complete identity:

```bash
id
```

A typical privileged result may resemble:

```text
uid=0(root) gid=0(root) groups=0(root)
```

The exact output depends on the environment.

### Important Distinction

Do not confuse:

```text
UID 0
```

with merely:

```text
username containing "root"
```

The UID is the important identity attribute.

---

## Step 3 — Check Effective Identity

Run:

```bash
id
```

Pay attention to:

```text
uid=
gid=
groups=
```

For unusual cases, inspect the current process credentials:

```bash
grep -E '^(Uid|Gid):' /proc/self/status
```

This helps distinguish real and effective credential information when investigating special execution contexts.

---

## Step 4 — Verify Privileged Access

Once higher privilege is confirmed, perform a **minimal** access check appropriate to the authorized lab.

For most privilege-escalation demonstrations, these are sufficient:

```bash
whoami
id
id -u
```

Do not perform unnecessary privileged operations merely to prove that privilege exists.

---

## Before-and-After Verification

Always compare the state before exploitation with the state afterward.

### Before

```bash
whoami
id
id -u
```

Record the result.

### After

```bash
whoami
id
id -u
```

Compare the two states.

### Example

```text
BEFORE

whoami
student

id
uid=1001(student) gid=1001(student) groups=1001(student)
```

After exploitation:

```text
AFTER

whoami
root

id
uid=0(root) gid=0(root) groups=0(root)
```

This provides clear evidence of the privilege-boundary crossing.

---

## Verification by Technique

Different exploitation mechanisms may require slightly different verification.

### Sudo

Verify:

```bash
whoami
id
```

Then document:

```text
sudo permission
        ↓
Allowed privileged command
        ↓
Controlled execution
        ↓
Higher privilege
```

### SUID

Verify:

```bash
whoami
id
```

Document:

```text
SUID binary
        ↓
Privileged execution
        ↓
User-controlled behavior
        ↓
Privilege increase
```

### Capabilities

Verify:

```bash
whoami
id
```

Then document the capability that enabled the privileged operation.

### Cron

Verify:

```bash
whoami
id
```

after the scheduled action has executed.

Document:

```text
Scheduled task
        ↓
Privileged execution
        ↓
User-controlled component
        ↓
Privilege increase
```

### Service

Verify:

```bash
whoami
id
```

Document:

```text
Privileged service
        ↓
Controllable execution path
        ↓
Service execution
        ↓
Privilege increase
```

### Credentials

Verify the target account or privilege level:

```bash
whoami
id
```

Do not expose unnecessary credentials in notes or reports.

Document the credential source without unnecessarily reproducing sensitive secret material.

### Container

Verify the resulting privilege context.

Do not assume:

```text
container root
=
host root
```

Determine which environment the verified identity belongs to.

Document the boundary explicitly:

```text
Container privilege
vs
Host privilege
```

### Kernel

Verify:

```bash
whoami
id
```

Then document:

```text
Kernel version
Vulnerability
Exploit applicability
Privilege change
```

---

## Evidence Collection

### Core Principle

Evidence should answer:

> **What happened, why did it happen, and what proves it?**

Collect only what is necessary.

### Minimum Evidence

Record:

```text
Initial user
Initial UID
Finding
Privileged component
Validation result
Exploitation technique
Final user
Final UID
```

### Recommended Evidence Format

```text
Initial identity:
student

Initial UID:
1001

Finding:
Writable script executed by root cron job

Validation:
Confirmed current user can modify the script and root executes it

Exploitation:
Controlled modification performed in authorized lab

Final identity:
root

Final UID:
0
```

---

## Command Evidence

Useful commands include:

```bash
whoami
id
id -u
id -g
```

For the target process or environment, additional evidence may include:

```bash
ps -ef
```

or:

```bash
ps aux
```

Use only commands necessary to establish the result.

---

## Capture the Attack Path

A good privilege-escalation finding should be explainable as a chain.

Use:

```text
Initial Access
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
```

For example:

```text
Low-privileged shell
      ↓
Root cron job discovered
      ↓
Cron script writable by current user
      ↓
Execution relationship validated
      ↓
Controlled modification
      ↓
Cron executes as root
      ↓
UID 0 verified
```

This is much more useful than simply writing:

```text
"Got root through cron."
```

---

## Attribution of the Privilege Increase

This is one of the most important verification steps.

Ask:

> **Can I prove that the exploited finding caused the privilege increase?**

Consider whether another mechanism could have caused the result.

For example:

```text
Finding A → exploited
Finding B → unrelated
Finding C → unrelated
```

The report should clearly identify:

```text
Finding A
   ↓
Exploitation
   ↓
Privilege increase
```

Do not claim causation merely because exploitation and privilege change happened around the same time.

---

## Verification Checklist

```text
[ ] Initial identity recorded
[ ] Initial UID recorded
[ ] Finding identified
[ ] Finding validated
[ ] Exploitation performed
[ ] Final identity checked
[ ] Final UID checked
[ ] Privilege difference confirmed
[ ] Exploitation path understood
[ ] Evidence recorded
```

---

## Root Does Not Always Mean Done

Obtaining root is a milestone, not the end of the assessment workflow.

After verification, determine:

```text
What caused the escalation?
What configuration enabled it?
What files were changed?
What processes were started?
What evidence should be preserved?
What should be cleaned up?
```

The technical result should be converted into a reproducible finding.

---

## Document the Vulnerability

Use this structure:

```text
### Title

[Short description of the privilege-escalation weakness]

### Initial Access

[How the low-privileged shell/account was obtained]

### Finding

[What was discovered]

### Root Cause

[Why the privileged action was controllable]

### Validation

[How exploitability was confirmed]

### Exploitation

[Minimal exploitation technique]

### Verification

[Evidence showing the privilege change]

### Impact

[What privilege or access was obtained]

### Remediation

[How the underlying weakness should be corrected]
```

---

## Root Cause vs Exploit

Keep these concepts separate.

### Root Cause

The condition that made escalation possible.

Examples:

```text
Writable root-owned script
Unsafe sudo permission
Misconfigured SUID binary
Excessive container privilege
Exposed privileged service
```

### Exploit

The technique used to take advantage of that condition.

Examples:

```text
Controlled script modification
Permitted command execution
SUID behavior abuse
Container boundary abuse
```

### Result

The resulting privilege.

```text
UID 0
Privileged service account
Higher-privileged local account
```

A strong report explains all three.

---

## Cleanup

### Core Question

Ask:

> **What did I change while proving the vulnerability, and does it need to be restored?**

Cleanup depends on the authorized assessment scope.

### Review Changes

Consider:

```text
Files modified
Configuration changed
Processes started
Temporary files created
Services changed
Scheduled tasks modified
Test accounts or credentials created
Network changes
```

### Cleanup Principle

Do not blindly delete everything.

First determine:

```text
Was the change part of the original environment?
Was the change introduced by testing?
Is the evidence required?
Does the scope require restoration?
```

Preserve evidence before removing it.

---

## File Cleanup

If you created temporary files during an authorized test, identify them first.

For example:

```bash
ls -la /tmp
```

Do not indiscriminately remove files from shared systems.

Only remove artifacts you created and are authorized to remove.

---

## Process Cleanup

If your test launched a temporary process, identify it before terminating anything.

For example:

```bash
ps aux
```

Confirm:

```text
Process
Owner
Purpose
Whether it belongs to the test
```

Then clean it up only if authorized.

---

## Service Cleanup

If testing required a service restart or temporary configuration change:

```text
Record original state
      ↓
Perform test
      ↓
Collect evidence
      ↓
Restore authorized original state
      ↓
Verify service state
```

Do not assume the original state.

Record it before changing anything.

---

## Scheduled Task Cleanup

If a scheduled-task test introduced a temporary change:

```text
Identify test modification
        ↓
Preserve evidence
        ↓
Restore original content
        ↓
Verify permissions
        ↓
Verify scheduled task
```

Be particularly careful with root-owned scheduled tasks.

---

## Evidence Preservation

Before cleanup, preserve:

```text
Command output
Screenshots
Relevant configuration
Before/after state
Timestamps
Finding details
```

The goal is:

```text
Evidence
   ↓
Cleanup
   ↓
Reproducible report
```

not:

```text
Cleanup
   ↓
Lost evidence
```

---

## When Cleanup Is Not Appropriate

Do not automatically modify a target after exploitation.

Cleanup may be inappropriate when:

```text
System is a dedicated training lab
The lab expects the modification
The assessment requires forensic preservation
The client explicitly requests evidence preservation
Restoration is outside scope
```

Follow the authorized scope.

---

## Failed Exploitation Verification

A failed exploit still provides useful information.

Record:

```text
Finding
Expected result
Technique attempted
Actual result
Failure condition
Reason identified
```

For example:

```text
Expected:
UID 0

Actual:
UID 1001

Reason:
Required privilege condition was not present
```

This prevents an invalid finding from being reported as confirmed.

---

## Common Verification Mistakes

### Mistake 1 — "The Exploit Ran, So It Worked"

Execution does not prove privilege escalation.

Always run:

```bash
whoami
id
id -u
```

### Mistake 2 — Checking Only the Username

A username alone is insufficient in unusual environments.

Use:

```bash
id
```

to inspect UID, GID, and groups.

### Mistake 3 — Forgetting the Baseline

Without a before-state, the privilege change is harder to demonstrate.

Record:

```text
Before
```

and:

```text
After
```

### Mistake 4 — Failing to Attribute the Result

If multiple possible escalation paths exist, identify which one actually produced the privilege increase.

### Mistake 5 — Cleaning Up Too Early

Evidence should be preserved before restoration.

### Mistake 6 — Over-Testing After Success

Once the intended privilege boundary has been demonstrated, unnecessary additional actions increase risk without improving the finding.

---

## Decision Tree

```text
Exploitation completed
        │
        ▼
Check whoami / id
        │
        ├── Expected privilege
        │       │
        │       ▼
        │   Check UID/GID
        │       │
        │       ▼
        │   Record evidence
        │       │
        │       ▼
        │   Attribute privilege change
        │       │
        │       ▼
        │   Preserve evidence
        │       │
        │       ▼
        │   Cleanup if authorized
        │       │
        │       ▼
        │   Document path
        │
        └── Unexpected privilege
                │
                ▼
          Analyze result
                │
                ├── Finding still valid?
                │       ├── YES → Re-analyze exploitation
                │       └── NO  → Record false positive
                │
                └── Cause unknown
                        ↓
                  Return to validation
```

---

## Quick Reference

### Identity

```bash
whoami
id
id -u
id -g
```

### Process Context

```bash
ps aux
```

### Current Credential State

```bash
grep -E '^(Uid|Gid):' /proc/self/status
```

### Before/After Comparison

```text
BEFORE
whoami
id
id -u

AFTER
whoami
id
id -u
```

---

## Root Verification Checklist

```text
[ ] Exploitation completed
[ ] whoami checked
[ ] id checked
[ ] UID checked
[ ] GID checked
[ ] Before/after comparison completed
[ ] Expected privilege confirmed
[ ] Privilege increase attributed to finding
[ ] Evidence preserved
[ ] Test artifacts identified
[ ] Cleanup decision made
[ ] Authorized cleanup completed if required
[ ] Final environment state checked
[ ] Attack path documented
```

---

## Complete Workflow

The entire post-exploitation process is:

```text
VALIDATED FINDING
       ↓
CONTROLLED EXPLOITATION
       ↓
WHOAMI
       ↓
ID
       ↓
UID/GID VERIFICATION
       ↓
BEFORE vs AFTER
       ↓
PRIVILEGE CHANGE CONFIRMED?
       │
       ├── NO → Analyze / Re-validate
       │
       └── YES
             ↓
       COLLECT EVIDENCE
             ↓
       ATTRIBUTE ROOT CAUSE
             ↓
       DOCUMENT ATTACK PATH
             ↓
       PRESERVE REQUIRED EVIDENCE
             ↓
       CLEAN UP IF AUTHORIZED
             ↓
       FINAL STATE CHECK
```

---

## Final Mental Model

Remember:

> **Getting a privileged shell is not the proof. Proving why you got it is the proof.**

The final verification loop is:

```text
BEFORE
  ↓
EXPLOIT
  ↓
AFTER
  ↓
COMPARE
  ↓
PROVE
  ↓
DOCUMENT
  ↓
CLEAN UP
```

The strongest privilege-escalation result can be summarized in one sentence:

> **The low-privileged user could influence [privileged component], causing [privileged action], which resulted in [verified privilege].**

That is the complete bridge from:

```text
Enumeration
```

to:

```text
Validated Privilege Escalation
```

and finally to:

```text
Reproducible Security Finding
```
