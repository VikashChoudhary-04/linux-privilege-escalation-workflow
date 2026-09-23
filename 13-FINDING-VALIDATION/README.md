# 13 — Finding Validation

Finding something interesting is not the same as finding a vulnerability.

Finding a vulnerability is not the same as finding an exploitable privilege-escalation path.

This stage exists to answer one question:

> **Can I prove that the finding creates a real, reproducible path across the privilege boundary?**

The workflow is:

```text id="v6m2q8"
FINDING
   ↓
REPRODUCE
   ↓
UNDERSTAND
   ↓
IDENTIFY TRUST
   ↓
IDENTIFY USER CONTROL
   ↓
CONFIRM PRIVILEGED ACTION
   ↓
TEST APPLICABILITY
   ↓
VALIDATE IMPACT
   ↓
DOCUMENT
```

---

## Core Question

> **Does this finding actually allow the current user to influence a more privileged action?**

Every candidate should be reduced to a simple relationship:

```text id="p8x3m7"
PRIVILEGED ACTION
        +
USER CONTROL
        +
TRUST RELATIONSHIP
        +
REPRODUCIBLE IMPACT
        =
VALIDATED ESCALATION PATH
```

If one of these links is missing, continue investigating before calling the finding exploitable.

---

## Mental Model

Use the following questions:

```text id="k4q9m2"
WHAT DID I FIND?
      ↓
WHY IS IT INTERESTING?
      ↓
WHO OWNS IT?
      ↓
WHO CAN CONTROL IT?
      ↓
WHO TRUSTS IT?
      ↓
WHEN IS IT USED?
      ↓
WHAT PRIVILEGE DOES THE TRUSTED ACTION HAVE?
      ↓
CAN MY CONTROL CHANGE THE RESULT?
      ↓
WHAT PRIVILEGE DO I GAIN?
```

The most important question is:

> **Where exactly does the privilege boundary get crossed?**

---

## 1. Finding vs Vulnerability vs Exploitable Path

This distinction is the foundation of the entire stage.

### Finding

A finding is an observation.

Example:

```text id="n6m3x8"
Current user can write to:
/opt/scripts/backup.sh
```

This is interesting, but incomplete.

### Vulnerability

A vulnerability exists when the writable file is trusted by a privileged process.

```text id="q2x7m5"
Current user
      ↓
Can modify backup.sh
      ↓
root executes backup.sh
```

Now there is a security weakness.

### Exploitable Path

The path becomes exploitable when the user's control can produce a measurable privilege increase.

```text id="w8c4n1"
Current user
      ↓
Controls backup.sh
      ↓
root executes backup.sh
      ↓
Controlled behavior occurs as root
      ↓
Privilege boundary crossed
```

### 💡 REMEMBER

```text id="m5k8x2"
Finding ≠ Vulnerability ≠ Exploitable Path
```

---

## 2. Establish the Baseline

Before validating a finding, record the current identity.

### 🔎 ENUMERATE

```bash id="r3x7m9"
whoami
```

```bash id="c8q2n5"
id
```

Check the shell:

```bash id="v6m1k4"
echo "$SHELL"
```

Check the hostname:

```bash id="p9x3w7"
hostname
```

### 🧠 WHY

You need a baseline to compare against.

For example:

```text id="t4m8q2"
BEFORE

User: vikash
UID: 1000
Groups: users
```

After validation:

```text id="n7c2m5"
AFTER

User: root
UID: 0
```

The difference demonstrates the privilege boundary.

---

## 3. Reproduce the Finding

Never validate from a vague description.

Reproduce the actual condition.

### Example

Automated tool reports:

```text id="x6m3q8"
Writable root-owned script.
```

Manually verify:

```bash id="j4n8c2"
ls -la /path/to/script
```

```bash id="w7m2p5"
stat /path/to/script
```

### 🧠 WHY

This confirms:

```text id="q3x8m1"
The file exists
+
The permissions are what the tool reported
+
The current user can actually modify it
```

If reproduction fails:

```text id="f5k2v9"
Finding may be:
False positive
Stale
Permission-dependent
Environment-dependent
```

---

## 4. Identify the Trusted Consumer

A controlled file only matters if something trusts or consumes it.

### 🔎 ENUMERATE

Search for references:

```bash id="m8q4x2"
grep -Rni '/path/to/file' /etc /opt /usr/local 2>/dev/null
```

Inspect services:

```bash id="r6c1v8"
systemctl cat <service>
```

Inspect scheduled tasks:

```bash id="k9x3m5"
cat /etc/crontab
```

Inspect processes:

```bash id="p2w7n4"
ps -eo user,pid,ppid,args
```

### 🧠 WHY

You need to establish:

```text id="v3m8q1"
Controlled object
      ↓
Trusted consumer
      ↓
Execution/access
      ↓
Privilege
```

Without a trusted consumer, the finding may not lead anywhere.

---

## 5. Identify the Privilege Context

Determine who performs the trusted action.

### 🔎 ENUMERATE

For a process:

```bash id="n5c8x2"
ps -p <PID> -o user,pid,ppid,args
```

For a service:

```bash id="q7m3v1"
systemctl show -p User,Group <service>
```

For a scheduled task:

```bash id="w4k9m6"
cat /etc/crontab
```

### 🧠 WHY

Compare:

```text id="t2x6m8"
Current user
      ↓
Privileged consumer
```

A process running as your current user does not normally provide a user-to-root privilege boundary.

---

## 6. Identify User Control

The next question is:

> **Exactly what can the current user influence?**

Possible control points include:

```text id="c7m2x8"
File contents
File name
Directory contents
Directory permissions
PATH
Environment variables
Command arguments
Wildcard expansion
Library search path
Configuration
Credentials
Container configuration
Service inputs
```

### 🔎 ENUMERATE

For files:

```bash id="x4m8q1"
ls -la /path/to/file
```

For directories:

```bash id="p6c2n9"
ls -ld /path/to/directory
```

For path components:

```bash id="v7m3k5"
namei -l /path/to/file
```

For environment:

```bash id="r2x8c4"
env | sort
```

For command resolution:

```bash id="m5q1w7"
command -v <command>
```

---

## 7. Identify the Trust Relationship

A vulnerability often exists because a privileged process trusts something controlled by a lower-privileged user.

### 🧠 CONCEPT

Common trust relationships:

```text id="h4m9x2"
Root script → user-writable file
```

```text id="q8c3m5"
Root process → user-controlled environment
```

```text id="z6v1n7"
Privileged binary → writable library
```

```text id="j2x7p4"
Scheduled task → writable script
```

```text id="c5m8q1"
Container runtime → user-controlled container configuration
```

```text id="n9w3k6"
Credential → privileged account
```

The exact relationship matters more than the vulnerability label.

---

## 8. Determine Execution Timing

A finding is more useful when you know when the privileged action occurs.

### 🔎 ENUMERATE

For cron:

```bash id="k4x8m2"
cat /etc/crontab
```

For systemd timers:

```bash id="r7c3n9"
systemctl list-timers --all
```

For services:

```bash id="m2q6v5"
systemctl status <service>
```

For dynamic execution:

```text id="w8n4c1"
pspy
```

when available in the authorized environment.

### 🧠 WHY

You need to know:

```text id="v5m2x8"
Does the action happen automatically?
Does it happen only at startup?
Does it happen periodically?
Does it happen when a request arrives?
Does it happen only under a specific condition?
```

---

## 9. Determine the Complete Execution Path

Map every step.

### 🧠 CONCEPT

Use:

```text id="q3m7x9"
TRIGGER
  ↓
PROCESS
  ↓
USER
  ↓
EXECUTABLE
  ↓
ARGUMENTS
  ↓
DEPENDENCIES
  ↓
FILES / CONFIG
  ↓
USER CONTROL
  ↓
RESULT
```

### Example

```text id="n8c4m2"
Cron
 ↓
root
 ↓
/opt/backup.sh
 ↓
tar *
 ↓
/opt/backups/
 ↓
user controls filenames
 ↓
unexpected privileged behavior
```

This gives you an attack-path model rather than a disconnected finding.

---

## 10. Check Preconditions

Many vulnerabilities only work when specific conditions exist.

### 🔎 ENUMERATE

Build a prerequisite list:

```text id="w5x1m8"
[ ] Correct software version
[ ] Correct architecture
[ ] Required file exists
[ ] Required permission exists
[ ] Required service is running
[ ] Required user/group access exists
[ ] Required configuration exists
[ ] Required execution path exists
```

### 🧠 WHY

If a prerequisite is missing:

```text id="p7m3c9"
The finding may not be exploitable.
```

Do not compensate for a missing prerequisite by blindly changing the system.

---

## 11. Validate Permissions

Permissions are often the deciding factor.

### 🔎 ENUMERATE

```bash id="x8q2m5"
ls -la /path/to/file
```

```bash id="m4c7n1"
stat /path/to/file
```

```bash id="v9w3k6"
namei -l /path/to/file
```

For directories:

```bash id="q5m8x2"
ls -ld /path/to/directory
```

### 🧠 WHY

You need to prove:

```text id="k2x6m9"
Current user
      ↓
Can actually modify/access object
```

Do not infer access from filenames alone.

---

## 12. Validate Execution

Permission alone does not prove exploitation.

You must establish that the trusted component is actually used.

### 🔎 ENUMERATE

For a service:

```bash id="f3m7x1"
systemctl status <service>
```

For a process:

```bash id="n8q4c6"
ps -eo user,pid,ppid,args
```

For a scheduled task:

```bash id="x2m5v9"
cat /etc/crontab
```

For dynamic observation:

```text id="w6k3p8"
pspy
```

### 🧠 WHY

The relationship must be:

```text id="r5x9m2"
User control
      ↓
Trusted component
      ↓
Actual execution
```

Not merely:

```text id="c7q2m4"
User control
```

---

## 13. Validate Privilege Impact

The final question is:

> **What privilege does the controlled action actually receive?**

### 🔎 ENUMERATE

Before testing:

```bash id="m8x3q7"
whoami
```

```bash id="p4n9c2"
id
```

After authorized validation:

```bash id="v6k1w5"
whoami
```

```bash id="x2q8m4"
id
```

### 🧠 WHY

You need measurable evidence.

For example:

```text id="j5m7x2"
Before:
UID 1000

After:
UID 0
```

This is much stronger than:

```text id="q9c3v8"
"Exploit worked."
```

---

## 14. Validate Without Full Exploitation

Whenever possible, prove the privilege boundary with the smallest safe action.

### 🧪 VALIDATE

Examples of harmless validation objectives:

```text id="m7x2q8"
Confirm execution identity
Confirm file access
Confirm command execution context
Confirm service identity
Confirm container context
Confirm effective UID
```

A validation payload should answer:

```text id="n4c8m1"
"Did my controlled input execute with the expected privilege?"
```

rather than immediately performing unrelated actions.

---

## 15. False Positives

A false positive is a reported condition that does not create the expected security impact.

Common examples:

```text id="c5m8x2"
Writable file
    ↓
Never executed
```

```text id="q7n3v6"
Old software
    ↓
Security patch backported
```

```text id="x4m9k1"
SUID binary
    ↓
No controllable behavior
```

```text id="w2c6p8"
Credential discovered
    ↓
Invalid or expired
```

```text id="r8m3q5"
Docker group
    ↓
Additional controls prevent expected access
```

### 💡 REMEMBER

A false positive is useful information.

It tells you:

```text id="p6x1m9"
What looked dangerous
        ≠
What is actually exploitable
```

---

## 16. Privilege Boundary Analysis

Every validation should explicitly identify the two sides of the boundary.

### 🧠 CONCEPT

```text id="v4m8x2"
BEFORE
Current User
UID 1000
      │
      │  ← privilege boundary
      │
AFTER
Root
UID 0
```

Or:

```text id="q7c3n9"
Container User
      │
      │ ← container/host boundary
      │
Host Privileged Context
```

Or:

```text id="m2x6w8"
Low-privileged Service
      │
      │ ← trust boundary
      │
Root Service
```

### 💡 REMEMBER

Do not simply ask:

> "Did something execute?"

Ask:

> **"Did execution move across a privilege boundary?"**

---

## 17. Validation Matrix

Use this matrix for important findings.

| Question                                      | Result |
| --------------------------------------------- | ------ |
| What is the finding?                          |        |
| Why is it interesting?                        |        |
| Who owns the trusted component?               |        |
| Who controls the input?                       |        |
| What process consumes it?                     |        |
| What user runs that process?                  |        |
| When does it execute?                         |        |
| What prerequisites exist?                     |        |
| Can the behavior be reproduced?               |        |
| Does the behavior cross a privilege boundary? |        |
| What evidence proves it?                      |        |
| What remains uncertain?                       |        |

This turns vague findings into structured evidence.

---

## 18. Validation by Finding Type

Different findings require different proof.

### Sudo

```text id="z5m8q2"
sudo rule
   ↓
Allowed command
   ↓
Command behavior
   ↓
Can user influence execution?
   ↓
Privilege result
```

### SUID

```text id="c7x2m9"
SUID binary
   ↓
Behavior
   ↓
User-controlled input
   ↓
Privileged execution
```

### Cron

```text id="w4n8p1"
Cron entry
   ↓
Root execution
   ↓
Script/command
   ↓
User control
   ↓
Execution
```

### Service

```text id="m6q3v8"
Service
   ↓
Execution user
   ↓
Executable/config/dependency
   ↓
User control
   ↓
Service restart/start
```

### Credential

```text id="r2x7c5"
Credential
   ↓
Account
   ↓
Authentication
   ↓
Privilege
```

### Container

```text id="p9m4k2"
Runtime access
   ↓
Control interface
   ↓
Privileged resource
   ↓
Boundary
```

### Kernel Vulnerability

```text id="x8c1m6"
Kernel
   ↓
Exact version
   ↓
Applicable CVE
   ↓
Prerequisites
   ↓
Privilege impact
```

---

## 19. Validation Documentation

For each confirmed path, record:

```text id="n5q8m3"
Finding:
What was discovered?

Evidence:
What commands prove it?

Control:
What does the current user control?

Trusted Consumer:
What privileged process uses it?

Privilege:
Who executes the trusted action?

Trigger:
When/how does execution occur?

Impact:
What privilege boundary is crossed?

Validation:
How was the finding safely confirmed?
```

This structure can later be used in the reporting stage.

---

## 20. Safe Validation Principles

### 🧪 Principle 1 — Minimize Impact

Use the smallest test that proves the relationship.

### 🧪 Principle 2 — Preserve the Environment

Do not unnecessarily:

* Delete files
* Modify system configuration
* Stop services
* Kill processes
* Destroy containers
* Alter credentials

### 🧪 Principle 3 — Prove One Thing at a Time

For example:

```text id="k3x7m1"
First:
Can I modify the file?

Then:
Is it executed by root?

Then:
Does my controlled change execute?

Then:
What privilege does it receive?
```

### 🧪 Principle 4 — Record Before and After

Capture:

```text id="m8c2q6"
Identity
Permissions
Process
Configuration
Result
```

---

## 21. When Validation Fails

Not every interesting finding deserves endless investigation.

If validation fails:

```text id="q6m3x8"
Finding
   ↓
Reproduce
   ↓
Prerequisite missing?
   ↓
YES
   ↓
Mark as not currently exploitable
   ↓
Move to next candidate
```

Possible reasons:

```text id="v1x7n4"
Wrong version
Wrong architecture
No execution
No user control
Insufficient permissions
Credential invalid
Service not running
Configuration mismatch
Patch already applied
No privilege boundary
```

---

## 22. Validation Priority

When multiple findings exist, prioritize those with the strongest relationship.

### High-Value Validation Targets

```text id="r5c8m2"
Privileged execution
+
Direct user control
```

```text id="x7m3q9"
Valid privileged credential
```

```text id="k2n6v4"
Powerful management interface
+
User access
```

```text id="p8q1m5"
Applicable local vulnerability
+
Privileged component
```

### Lower-Confidence Leads

```text id="w4m7x2"
Old software
```

```text id="n9c3k6"
Interesting filename
```

```text id="v5x8q1"
Writable file with no known consumer
```

The purpose is to spend validation time where the evidence is strongest.

---

## 23. Complete Validation Workflow

```text id="m3x7q9"
                FINDING
                   │
                   ▼
             REPRODUCE IT
                   │
                   ▼
             WHAT IS IT?
                   │
                   ▼
          WHO CONTROLS IT?
                   │
                   ▼
          WHO TRUSTS IT?
                   │
                   ▼
         WHAT EXECUTES/ACCESSES IT?
                   │
                   ▼
         UNDER WHICH PRIVILEGE?
                   │
                   ▼
        CAN USER CONTROL CHANGE
             THE RESULT?
              │         │
             NO        YES
              │         │
              ▼         ▼
           MOVE ON   VALIDATE
                         │
                         ▼
                PRIVILEGE BOUNDARY?
                    │         │
                   NO        YES
                    │         │
                    ▼         ▼
                 MOVE ON    DOCUMENT
                              │
                              ▼
                          EXPLOIT
```

---

## 24. Common Mistakes

### ❌ Mistake 1 — Calling Every Finding a Vulnerability

A finding requires analysis.

### ❌ Mistake 2 — Skipping Reproduction

Always verify the actual condition.

### ❌ Mistake 3 — Ignoring the Trusted Consumer

A writable object with no privileged consumer may be irrelevant.

### ❌ Mistake 4 — Ignoring Execution Context

Determine who actually performs the action.

### ❌ Mistake 5 — Ignoring Timing

A scheduled task may only execute under specific conditions.

### ❌ Mistake 6 — Proving Execution but Not Privilege

Execution alone is not privilege escalation.

### ❌ Mistake 7 — Over-Exploiting

Once the privilege boundary is proven, additional unnecessary actions add risk without improving the finding.

### ❌ Mistake 8 — Failing to Record Evidence

A finding you cannot explain later is difficult to reproduce and report.

---

## 25. Five-Minute Validation Checklist

For any interesting finding:

```bash id="q8m2x4"
whoami
```

```bash id="m5c7n1"
id
```

Then determine:

```text id="v3x9k6"
[ ] What did I find?
[ ] Can I reproduce it?
[ ] Who owns it?
[ ] Who controls it?
[ ] Who trusts it?
[ ] What process uses it?
[ ] Who runs that process?
[ ] When does it execute?
[ ] Can my control change the result?
[ ] What privilege results?
```

If you cannot answer these questions:

```text id="p6m2w8"
Not validated yet.
```

---

## 26. Quick Command Reference

| Goal                  | Command                                    |
| --------------------- | ------------------------------------------ |
| Current user          | `whoami`                                   |
| Identity and groups   | `id`                                       |
| File permissions      | `ls -la /path/to/file`                     |
| File metadata         | `stat /path/to/file`                       |
| Path permissions      | `namei -l /path/to/file`                   |
| Processes             | `ps -eo user,pid,ppid,args`                |
| Process identity      | `ps -p <PID> -o user,pid,ppid,args`        |
| Process executable    | `readlink -f /proc/<PID>/exe`              |
| Service configuration | `systemctl cat <service>`                  |
| Service user          | `systemctl show -p User,Group <service>`   |
| Running services      | `systemctl --type=service --state=running` |
| Timers                | `systemctl list-timers --all`              |
| Cron configuration    | `cat /etc/crontab`                         |
| Environment           | `env \| sort`                              |
| Command resolution    | `command -v <command>`                     |
| Mounts                | `findmnt`                                  |
| Network listeners     | `ss -lntup`                                |

---

## 27. Complete Checklist

### 🔎 Discovery

```text id="x7m3p8"
[ ] Identify the finding
[ ] Reproduce the finding
[ ] Determine why it matters
```

### 🧠 Trust Analysis

```text id="c4n8q2"
[ ] Identify owner
[ ] Identify user control
[ ] Identify trusted consumer
[ ] Identify execution/access path
[ ] Identify execution user
[ ] Identify trigger
```

### 🧪 Validation

```text id="m9x2k6"
[ ] Confirm prerequisites
[ ] Confirm reproducibility
[ ] Confirm privileged action
[ ] Confirm user-controlled influence
[ ] Confirm resulting behavior
[ ] Confirm privilege boundary
[ ] Record evidence
```

### 🚀 Before Exploitation

```text id="p5w7c3"
[ ] Environment is authorized
[ ] Impact is understood
[ ] Validation is sufficient
[ ] Exploitation is necessary
[ ] Minimal-impact approach selected
```

---

## 💡 Remember

The most important distinction in this entire repository is:

```text id="n8q3m6"
FOUND
  ≠
VULNERABLE
  ≠
EXPLOITABLE
```

Use this sentence whenever you get stuck:

> **What does the privileged process trust, and can I control that trusted component?**

Then prove every link:

```text id="r4m8x2"
CURRENT USER
     ↓
USER CONTROL
     ↓
TRUSTED COMPONENT
     ↓
PRIVILEGED ACTION
     ↓
RESULT
     ↓
PRIVILEGE BOUNDARY
```

If you can prove that chain, you understand the finding.

---

## Where Findings Lead

This stage is the bridge between enumeration and exploitation:

```text id="x6m2q9"
00–12
Enumeration
   ↓
13
Finding Validation
   ↓
Confirmed Attack Path
   ↓
14
Exploitation
   ↓
15
Root Verification
```

It also feeds back into earlier stages when additional context is required:

```text id="k3c8v1"
Finding
   ↓
Missing context
   ↓
Return to appropriate enumeration stage
   ↓
Collect evidence
   ↓
Validate again
```

This prevents premature exploitation.

---

## Final Mental Model

```text id="v8q2m5"
                  FINDING
                     │
                     ▼
                REPRODUCE
                     │
                     ▼
              UNDERSTAND IT
                     │
                     ▼
             WHO CONTROLS IT?
                     │
                     ▼
             WHO TRUSTS IT?
                     │
                     ▼
          WHAT PRIVILEGED ACTION?
                     │
                     ▼
          CAN CONTROL CHANGE IT?
                  │        │
                 NO       YES
                  │        │
                  ▼        ▼
               MOVE ON   VALIDATE
                           │
                           ▼
                  PRIVILEGE BOUNDARY?
                       │       │
                      NO      YES
                       │       │
                       ▼       ▼
                    MOVE ON  DOCUMENT
                                 │
                                 ▼
                              EXPLOIT
                                 │
                                 ▼
                              VERIFY
```

> **Enumeration discovers possibilities. Validation turns possibilities into evidence.**
