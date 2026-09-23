# Linux Privilege Escalation Attack Path Analysis

> **Purpose:** Learn how to connect individual enumeration findings into a complete, evidence-based privilege-escalation attack path.

---

## Core Question

A privilege-escalation assessment rarely presents one obvious answer.

Instead, you may discover:

* a writable script
* a root-owned service
* a SUID binary
* a credential
* a Docker group
* a suspicious cron job
* a dangerous capability
* a vulnerable application
* an unusual configuration

The important question is not:

> **"Is this interesting?"**

The important question is:

> **"Can these conditions form a complete path from my current privileges to a higher-privileged action?"**

---

## Attack Path Mental Model

Use this model:

```text
CURRENT USER
     ↓
ACCESS
     ↓
CONTROL
     ↓
TRUST
     ↓
PRIVILEGED ACTION
     ↓
PRIVILEGE BOUNDARY
     ↓
HIGHER PRIVILEGE
```

A finding becomes an attack-path candidate when it connects two or more of these stages.

---

## Finding vs Attack Path

A finding is an observation.

An attack path is a connected sequence of observations and actions.

Example:

```text
Writable file
```

is only a finding.

But:

```text
Current user
    ↓
Can modify script
    ↓
Root-owned cron executes script
    ↓
Cron executes modified script
    ↓
Script runs as root
```

is an attack path.

### Remember

> **A finding tells you what exists. An attack path explains how privilege can change.**

---

## The Five Questions

For every interesting finding, ask:

### 1. What Is Privileged?

Identify:

* root-owned process
* root-owned service
* SUID binary
* privileged scheduled task
* privileged container runtime
* privileged account
* privileged group
* privileged capability

### 2. Who Controls It?

Determine whether the current user can influence:

* executable
* script
* configuration
* arguments
* environment
* directory
* library
* input
* credentials
* IPC endpoint
* mounted resource

### 3. What Does It Trust?

Identify dependencies such as:

* PATH
* environment variables
* configuration files
* scripts
* libraries
* files
* directories
* usernames
* credentials
* network services

### 4. When Does It Execute?

Determine whether the action occurs:

* immediately
* when manually invoked
* periodically
* at boot
* when a service starts
* when a user connects
* when an application processes input
* when another process triggers it

### 5. What Privilege Does It Produce?

Determine whether the resulting action provides:

* another user's access
* group-level access
* root access
* access to privileged files
* access to a privileged service
* another route toward root

---

## Attack Path Construction

Build the path from left to right.

```text
[CURRENT ACCESS]
       ↓
[USER CONTROL]
       ↓
[TRUSTED COMPONENT]
       ↓
[PRIVILEGED EXECUTION]
       ↓
[PRIVILEGE CHANGE]
```

For example:

```text
www-data
   ↓
Writable script
   ↓
Root cron job
   ↓
Cron executes script
   ↓
Root context
```

The goal is to prove every arrow.

---

## Path Node Analysis

Treat every attack path as a collection of nodes.

### Node Types

Common nodes include:

```text
USER
GROUP
FILE
DIRECTORY
PROCESS
SERVICE
SCRIPT
BINARY
CONFIGURATION
CREDENTIAL
SOCKET
CONTAINER
SCHEDULED TASK
NETWORK SERVICE
PRIVILEGE
```

Example:

```text
User
 ↓
Group
 ↓
Docker Socket
 ↓
Container Runtime
 ↓
Host Resource
 ↓
Root
```

---

## Edge Analysis

The arrows between nodes are equally important.

An edge represents a relationship such as:

```text
CAN_EXECUTE
CAN_READ
CAN_WRITE
OWNS
EXECUTES
LOADS
TRUSTS
CALLS
AUTHENTICATES_TO
MOUNTS
TRIGGERS
```

Example:

```text
User
  │
  └── CAN_WRITE ──→ Script
                         │
                         └── EXECUTED_BY ──→ Root Cron
```

The attack path exists because the relationships connect.

---

## Ownership Analysis

Ownership often determines whether a path is possible.

Check:

```bash
ls -la <file>
stat <file>
```

Ask:

```text
Who owns it?
What group owns it?
Who can write it?
Who can execute it?
Who consumes it?
```

Do not confuse ownership with control.

A file may be:

```text
root-owned
```

but still writable by the current user because of its permissions.

---

## Permission Analysis

For each path node, identify:

```text
READ
WRITE
EXECUTE
```

Example:

```text
Root-owned script
      ↓
User has write permission
      ↓
Root executes script
```

This is materially different from:

```text
Root-owned script
      ↓
User can only read it
      ↓
No direct modification
```

---

## Trust Relationship Analysis

Many privilege-escalation paths are trust failures.

Examples:

```text
Root service
     ↓
Trusts configuration
     ↓
Configuration writable by user
```

Or:

```text
Root script
     ↓
Trusts PATH
     ↓
PATH contains user-controlled directory
```

Or:

```text
Privileged application
     ↓
Trusts credential
     ↓
Credential discovered by current user
```

The key question is:

> **What privileged component trusts something that the current user can influence?**

---

## Execution Chain

An execution chain describes how an input reaches privileged execution.

Example:

```text
User-controlled file
       ↓
Cron job
       ↓
Shell
       ↓
Command
       ↓
Root process
```

Another:

```text
User
 ↓
sudo permission
 ↓
Allowed binary
 ↓
Interpreter
 ↓
Root process
```

Another:

```text
User
 ↓
Docker group
 ↓
Docker socket
 ↓
Container runtime
 ↓
Host resource
```

---

## Attack Path Completeness

A path should answer every question below:

```text
[ ] Who starts the path?
[ ] What does the user control?
[ ] What trusted component receives that control?
[ ] Who executes the component?
[ ] What privilege does it have?
[ ] What happens to the controlled input?
[ ] When does execution occur?
[ ] What privilege boundary is crossed?
[ ] What result should occur?
[ ] How will the result be verified?
```

If several answers are unknown, the path is not yet validated.

---

## Example: Cron Attack Path

Suppose you discover:

```text
/etc/cron.d/backup
```

First inspect it:

```bash
cat /etc/cron.d/backup
```

Suppose it launches:

```text
/usr/local/bin/backup.sh
```

Then inspect:

```bash
ls -la /usr/local/bin/backup.sh
```

Now construct:

```text
Current User
     ↓
Can write backup.sh
     ↓
Cron configuration invokes backup.sh
     ↓
Cron executes as root
     ↓
Modified script executes
     ↓
Root privilege
```

The important discovery is not merely:

```text
Cron exists
```

It is:

```text
User-controlled resource
        ↓
Trusted by
        ↓
Privileged scheduler
```

---

## Example: SUID Attack Path

Suppose:

```bash
find / -perm -4000 -type f 2>/dev/null
```

returns an unusual binary.

Do not immediately exploit it.

Build the path:

```text
Current User
     ↓
Can execute SUID binary
     ↓
Binary runs with elevated effective privilege
     ↓
Binary accepts controllable input
     ↓
Input changes privileged behavior
     ↓
Privilege boundary crossed
```

If the binary does not provide a controllable path, the finding may be irrelevant.

---

## Example: Sudo Attack Path

Start with:

```bash
sudo -l
```

Suppose the user may run a particular program as root.

Construct:

```text
Current User
     ↓
sudo authorization
     ↓
Allowed program
     ↓
Program executes as root
     ↓
Program exposes controllable behavior
     ↓
Root-level action
```

The key question is not:

> "Can I run sudo?"

It is:

> **"What can the allowed program do while running with elevated privilege?"**

---

## Example: Capability Attack Path

Suppose:

```bash
getcap -r / 2>/dev/null
```

returns a binary with a powerful capability.

Construct:

```text
Current User
     ↓
Can execute binary
     ↓
Binary possesses capability
     ↓
Capability affects privileged operation
     ↓
Binary behavior is controllable
     ↓
Privilege boundary affected
```

A capability is meaningful only in the context of what the binary can actually do.

---

## Example: Credential Attack Path

Suppose a configuration file contains credentials.

The path may be:

```text
Current User
     ↓
Can read configuration
     ↓
Credential discovered
     ↓
Credential belongs to privileged account
     ↓
Authorized authentication
     ↓
Higher-privileged access
```

The credential itself is a finding.

The authentication relationship creates the attack path.

---

## Example: Container Attack Path

A possible chain:

```text
Current User
     ↓
Membership in privileged container group
     ↓
Access to container runtime
     ↓
Control over container creation
     ↓
Privileged host interaction
     ↓
Host privilege boundary
```

The exact path depends on runtime configuration and host exposure.

Do not assume that every container membership produces the same result.

---

## Example: Service Attack Path

A service may run as root:

```text
Root Service
     ↓
Executes /opt/app/start.sh
```

Check:

```bash
ls -la /opt/app/start.sh
```

If the current user can modify the script:

```text
Current User
     ↓
Writable service script
     ↓
Root service executes script
     ↓
Privileged execution
```

The attack path depends on the service actually executing the modified resource.

---

## Attack Path Graph

For complicated systems, draw a graph.

```text
                   ┌──────────────┐
                   │ Current User │
                   └──────┬───────┘
                          │
                    CAN WRITE
                          │
                          ▼
                   ┌──────────────┐
                   │    Script    │
                   └──────┬───────┘
                          │
                      EXECUTED BY
                          │
                          ▼
                   ┌──────────────┐
                   │ Root Service │
                   └──────┬───────┘
                          │
                       RUNS AS
                          │
                          ▼
                   ┌──────────────┐
                   │     Root     │
                   └──────────────┘
```

This makes the privilege boundary visible.

---

## Multiple Possible Paths

A host may contain several candidate paths.

Example:

```text
Path A
User → SUID binary → elevated execution

Path B
User → writable cron script → root cron → root

Path C
User → credentials → privileged account

Path D
User → Docker group → container runtime → host
```

Do not automatically pursue every path.

First determine:

```text
Is the path complete?
Is it applicable?
Can it be validated?
What privilege does it provide?
What prerequisites exist?
```

---

## Path Prioritization

Use evidence rather than guesswork.

### High-Confidence Candidate

A path has:

```text
Known privileged component
+
Known user control
+
Known trust relationship
+
Known execution path
+
Known privilege impact
```

### Medium-Confidence Candidate

Some relationship is known, but one important dependency remains uncertain.

```text
Privileged component
+
Possible user control
+
Unknown execution behavior
```

### Low-Confidence Candidate

The finding is interesting but the privilege relationship is unclear.

```text
Unusual binary
or
Unusual file
or
Unusual configuration
```

Do not confuse "interesting" with "exploitable."

---

## Path Validation Matrix

Use a simple matrix:

| Question                         | Evidence                 | Status            |
| -------------------------------- | ------------------------ | ----------------- |
| Current user identified?         | `id`                     | Confirmed         |
| Privileged component identified? | Service/process/config   | Confirmed         |
| User control identified?         | Permission/configuration | Confirmed         |
| Trust relationship identified?   | Execution path           | Confirmed         |
| Trigger identified?              | Cron/service/manual      | Confirmed         |
| Privilege impact identified?     | Execution context        | Confirmed         |
| Exploitation validated?          | Controlled test          | Pending/Confirmed |
| Privilege verified?              | `id` / `whoami`          | Pending/Confirmed |

This prevents assumptions from becoming conclusions.

---

## Attack Path vs Exploit

These are different.

### Attack Path

Explains:

```text
How the privilege boundary can be crossed.
```

### Exploit

Is the actual action used to cross it.

Example:

```text
Attack Path:
User → writable script → root cron → root execution

Exploit:
Controlled modification of the authorized lab resource
```

The path should be understood before the exploit is selected.

---

## Attack Path vs Vulnerability

A vulnerability may exist without providing a usable path.

Example:

```text
Old software
```

may be vulnerable in theory.

But:

```text
Current version
+
patched distribution
+
missing prerequisite
+
wrong architecture
```

may prevent exploitation.

Therefore:

```text
Finding
   ≠
Vulnerability
   ≠
Attack Path
   ≠
Exploit
```

---

## Broken Attack Paths

A path can fail at any edge.

Example:

```text
User
 ↓
Writable script
 ↓
Root service
 X
Service never executes script
```

The file is writable, but the path is broken.

Another:

```text
User
 ↓
Credential
 ↓
Account
 X
Account has same privilege
```

The credential works, but does not provide escalation.

Another:

```text
User
 ↓
SUID binary
 X
No controllable privileged behavior
```

SUID exists, but no usable path is established.

---

## Broken Path Recovery

When a path fails:

```text
Do not force the exploit.
```

Instead:

```text
Identify failed edge
      ↓
Understand why it failed
      ↓
Look for another edge
      ↓
Re-evaluate the finding
```

Example:

```text
Writable file
      ↓
Not consumed by root
      ↓
Path rejected
      ↓
Search for another privileged consumer
```

---

## Attack Path Documentation

For every validated path, document:

```text
Initial User:
Current Privilege:
Target Privilege:

Entry Point:
Controlled Resource:
Privileged Component:
Trust Relationship:
Execution Trigger:
Privilege Boundary:

Validation:
Exploit:
Verification:

Root Cause:
Evidence:
Cleanup:
```

This makes the result reproducible.

---

## Evidence Collection

Useful evidence includes:

```bash
whoami
id
ls -la <resource>
stat <resource>
ps aux
systemctl status <service>
systemctl cat <service>
sudo -l
findmnt
```

Record only what is necessary.

Avoid unnecessary exposure of:

* passwords
* private keys
* API tokens
* session secrets

---

## Attack Path Review

Before exploitation, review the entire path:

```text
[ ] Initial access known
[ ] Current privilege known
[ ] Target privilege known
[ ] Privileged component identified
[ ] User-controlled input identified
[ ] Trust relationship identified
[ ] Execution path understood
[ ] Trigger understood
[ ] Preconditions satisfied
[ ] Exploitation authorized
[ ] Expected result known
[ ] Verification method defined
```

If an important item is unknown, investigate before proceeding.

---

## Common Mistakes

### Mistake 1: Treating Every Finding as an Exploit

```text
Interesting
≠
Exploitable
```

### Mistake 2: Ignoring the Consumer

A writable file means little unless something important consumes it.

### Mistake 3: Ignoring Execution Context

Always determine:

```text
Who executes this?
```

### Mistake 4: Ignoring Timing

A script may be writable but never executed during the assessment.

### Mistake 5: Ignoring Preconditions

A vulnerability may require:

* specific version
* configuration
* architecture
* authentication
* group membership
* network access

### Mistake 6: Skipping Validation

Never turn an assumption into an exploit attempt without proving the relationship.

### Mistake 7: Losing the Original Path

After exploitation, document exactly how the privilege boundary was crossed.

---

## Fast Attack Path Workflow

When time is limited:

```text
1. Identify current user
        ↓
2. Find privileged components
        ↓
3. Find user-controlled resources
        ↓
4. Connect them
        ↓
5. Identify trust relationship
        ↓
6. Identify execution trigger
        ↓
7. Validate
        ↓
8. Exploit
        ↓
9. Verify
        ↓
10. Document
```

---

## 5-Minute Attack Path Test

For a suspicious finding, answer:

```text
WHO?
What user am I?

WHAT?
What privileged component exists?

CONTROL?
What can I modify or influence?

TRUST?
What does the privileged component trust?

TRIGGER?
When does it execute?

IMPACT?
What privilege should result?

VERIFY?
How will I prove it?
```

If all seven answers are clear, you likely have a well-defined path.

---

## Final Mental Model

Remember attack-path analysis as:

```text
FINDING
   ↓
WHO OWNS IT?
   ↓
WHO EXECUTES IT?
   ↓
WHO CAN MODIFY IT?
   ↓
WHAT DOES IT TRUST?
   ↓
WHEN DOES IT RUN?
   ↓
WHAT PRIVILEGE DOES IT HAVE?
   ↓
CAN MY CONTROL REACH THAT PRIVILEGE?
   ↓
VALIDATE
   ↓
EXPLOIT
   ↓
VERIFY
```

The central lesson is:

> **Privilege escalation is not about finding the most interesting vulnerability. It is about proving a chain of control that crosses a privilege boundary.**
