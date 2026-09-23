# Linux Privilege Escalation Enumeration Priority

> **Purpose:** Decide what to enumerate first, what to investigate deeply, and when to stop pursuing a finding.

---

## Core Principle

Privilege escalation enumeration should not be:

```text
Run every command
        ↓
Collect everything
        ↓
Hope something works
```

It should be:

```text
Understand
    ↓
Prioritize
    ↓
Enumerate
    ↓
Interpret
    ↓
Investigate
    ↓
Validate
```

The objective is not to collect the largest amount of information.

The objective is to find the **most relevant privilege boundary with the least wasted effort**.

---

## The Priority Question

After every discovery, ask:

> **"Does this information help me identify something privileged that I can influence?"**

If the answer is clearly no, reduce its priority.

---

## Master Priority Model

Use this order:

```text
1. IDENTITY
      ↓
2. SUDO / TRUST
      ↓
3. PRIVILEGED EXECUTION
      ↓
4. USER CONTROL
      ↓
5. CREDENTIALS
      ↓
6. SCHEDULED EXECUTION
      ↓
7. FILESYSTEM
      ↓
8. NETWORK / SPECIAL ENVIRONMENTS
      ↓
9. VULNERABLE SOFTWARE
      ↓
10. AUTOMATED ENUMERATION
```

This is a practical starting order, not a rigid rule.

A strong finding discovered later can immediately become the highest-priority path.

---

## Priority Rule

A finding deserves more attention when it has more of these properties:

```text
PRIVILEGE
+
USER CONTROL
+
TRUST
+
EXECUTION
+
IMPACT
```

A useful mental model is:

```text
Priority
   =
Privileged Impact
   ×
User Control
   ×
Execution Relevance
```

This is a reasoning model, not a numerical scoring system.

---

## Priority Levels

### Immediate Investigation

Characteristics:

```text
Known privileged component
+
Known user control
+
Clear execution relationship
```

Examples:

* dangerous `sudo` permission
* writable root-executed script
* controllable privileged service
* usable privileged credential
* directly relevant container runtime access

Action:

```text
Stop broad enumeration
        ↓
Investigate deeply
        ↓
Validate
```

---

### High Priority

Characteristics:

```text
Privileged component identified
+
Possible user control
+
Execution relationship likely
```

Examples:

* unusual SUID binary
* suspicious root service
* writable configuration used by root
* privileged scheduled task
* dangerous capability

Action:

```text
Investigate ownership
        ↓
Inspect permissions
        ↓
Trace execution
```

---

### Medium Priority

Characteristics:

```text
Interesting system condition
+
Privilege relationship unclear
```

Examples:

* unusual process
* writable directory
* uncommon group
* unusual mount
* internal service
* potentially sensitive configuration

Action:

```text
Collect enough context
        ↓
Determine whether a privilege relationship exists
```

---

### Low Priority

Characteristics:

```text
Interesting
but no visible privilege relationship
```

Examples:

* ordinary user files
* normal system services
* unrelated open ports
* common binaries
* harmless configuration

Action:

```text
Record if useful
        ↓
Continue
```

---

## Phase 1: Identity First

Start with:

```bash id="m2l5qg"
whoami
id
id -u
id -g
```

Determine:

```text
Username
UID
Primary group
Supplementary groups
Current privilege
```

### Why It Is High Priority

Everything else depends on the current security context.

For example:

```text
User A
```

may not be able to access something that:

```text
User B
```

can access.

---

## Phase 2: Sudo and Trust

Check:

```bash id="5l6z9v"
sudo -l
```

If usable sudo permissions exist, investigate immediately.

Ask:

```text
What command?
Which target user?
What arguments?
What restrictions?
What environment?
What files?
What interpreters?
```

A direct privileged execution path can eliminate the need for extensive enumeration.

---

## Phase 3: Privileged Processes and Services

Inspect:

```bash id="t9u7a8"
ps aux
```

Then services:

```bash id="y8x2wl"
systemctl list-units --type=service --state=running
```

Focus on:

```text
root-owned
custom
third-party
recently installed
unusual
misconfigured
```

Do not spend equal time on every process.

---

## Phase 4: SUID and Capabilities

SUID:

```bash id="j5t8cx"
find / -perm -4000 -type f 2>/dev/null
```

Capabilities:

```bash id="8x2f4w"
getcap -r / 2>/dev/null
```

Prioritize:

```text
Unusual binary
+
Elevated execution
+
Controllable behavior
```

Do not automatically investigate every standard SUID binary.

---

## Phase 5: Scheduled Execution

Inspect:

```bash id="v8y2sh"
cat /etc/crontab
ls -la /etc/cron.d/
systemctl list-timers --all
```

Focus on:

```text
root execution
+
custom scripts
+
user-controlled resources
```

A root cron entry pointing to a user-writable script is substantially more important than an ordinary system maintenance task.

---

## Phase 6: Filesystem

Filesystem enumeration should answer specific questions.

### Permissions

```bash id="b4m7c9"
ls -la
stat <file>
```

### Writable Resources

Look for:

```text
Writable scripts
Writable configurations
Writable service files
Writable directories
Writable binaries
```

But always ask:

> **Who uses the resource?**

---

## Phase 7: Credentials and Secrets

Search selectively.

Potential sources include:

```text
Shell history
Configuration files
Application configuration
Backups
Environment variables
SSH material
Scripts
Logs
```

Prioritize credentials associated with:

```text
Root
Administrators
Service accounts
Privileged applications
Other higher-privileged users
```

A credential for the current user's account may be useful, but it does not automatically represent escalation.

---

## Phase 8: Network Enumeration

Inspect:

```bash id="e2k0k1"
ip addr
ip route
ss -lntup
```

Focus on:

```text
localhost-only services
privileged services
custom applications
administrative interfaces
services associated with credentials
```

Network findings may provide an indirect path.

Example:

```text
Current user
    ↓
Local service
    ↓
Weak trust relationship
    ↓
Privileged action
```

---

## Phase 9: Special Environments

Check for:

```text
Docker
LXD
containers
NFS
unusual mounts
special groups
virtualization interfaces
```

Start with context:

```bash id="5l8r1q"
id
findmnt
```

Then inspect the relevant runtime.

Do not investigate containers simply because they exist.

Ask:

> **Can this environment affect the host privilege boundary?**

---

## Phase 10: Vulnerable Software

Only after understanding the system should you investigate software vulnerabilities deeply.

Collect:

```bash id="7q4j0x"
uname -a
cat /etc/os-release
```

Then determine:

```text
Software version
Distribution patches
Architecture
Configuration
Exposure
Exploit prerequisites
```

Do not turn:

```text
Old version
```

into:

```text
Automatically exploitable
```

---

## Automated Enumeration

Use tools such as:

```text
LinPEAS
Linux Smart Enumeration
pspy
```

for acceleration.

The correct workflow is:

```text
Manual baseline
      ↓
Automated enumeration
      ↓
Interesting findings
      ↓
Manual reproduction
      ↓
Validation
```

Automation should reduce search time, not replace reasoning.

---

## What to Prioritize in Tool Output

When automated tools produce hundreds of findings, look first for:

```text
SUDO
SUID
CAPABILITIES
ROOT PROCESSES
ROOT SERVICES
CRON
WRITABLE ROOT-EXECUTED FILES
CREDENTIALS
PRIVILEGED GROUPS
CONTAINER ACCESS
INTERESTING CONFIGURATIONS
RELEVANT SOFTWARE VERSIONS
```

Ignore noise until the high-value paths are investigated.

---

## Privilege Boundary Priority

The strongest findings usually sit directly next to a privilege boundary.

Example:

```text
LOW PRIVILEGE
     │
     │ user-controlled
     ▼
TRUSTED RESOURCE
     │
     │ executed by
     ▼
HIGH PRIVILEGE
```

Compare that with:

```text
LOW PRIVILEGE
     │
     ▼
UNUSUAL FILE
```

The second finding may be interesting, but the first contains a complete privilege relationship.

---

## Control Before Complexity

Prefer a simple, proven path over a complicated theoretical path.

Example:

```text
Writable root script
```

should generally be investigated before:

```text
Potential kernel vulnerability
```

when the first path is clearly applicable.

The reason is not that one technique is universally superior.

The reason is that **certainty and direct control reduce unnecessary complexity**.

---

## Direct vs Indirect Paths

### Direct Path

```text
User
 ↓
Sudo
 ↓
Root
```

### Indirect Path

```text
User
 ↓
Credential
 ↓
Service
 ↓
Another account
 ↓
Sudo
 ↓
Root
```

Both can be valid.

The indirect path requires more assumptions and therefore requires more validation.

---

## Enumeration Depth

Not every finding deserves the same investigation depth.

Use:

```text
DISCOVER
   ↓
IS IT PRIVILEGED?
   ↓
CAN I CONTROL IT?
   ↓
DOES IT EXECUTE?
```

If the answer becomes "no" at any point:

```text
Stop investigating that branch.
```

If all answers are "yes":

```text
Deep investigation
        ↓
Validation
```

---

## Stop Conditions

Stop investigating a branch when:

### No Privilege

```text
Resource
 ↓
No privileged consumer
```

Move on.

### No Control

```text
Privileged component
 ↓
Current user cannot influence it
```

Move on.

### No Execution

```text
User-controlled resource
 ↓
Never executed
```

Move on.

### No Impact

```text
Privileged execution
 ↓
No meaningful privilege increase
```

Move on.

### Preconditions Fail

```text
Potential exploit
 ↓
Required condition absent
```

Move on unless another path exists.

---

## Escalation of Investigation

Use three levels.

### Level 1 — Discovery

Answer:

```text
Does something interesting exist?
```

Example:

```bash id="i8o5x2"
find / -perm -4000 -type f 2>/dev/null
```

### Level 2 — Relationship

Answer:

```text
Does it interact with privilege?
```

Example:

```bash id="3f5h9c"
ls -la <binary>
```

### Level 3 — Exploitability

Answer:

```text
Can my control actually cross the privilege boundary?
```

This is where validation occurs.

---

## Enumeration Priority by Finding Type

| Finding              | First Question                                      | Priority                  |
| -------------------- | --------------------------------------------------- | ------------------------- |
| `sudo -l`            | Can a permitted command provide privileged control? | Immediate if useful       |
| Root service         | Can I influence execution?                          | High                      |
| Writable root script | Is it executed by root?                             | High                      |
| SUID binary          | Does it expose controllable privileged behavior?    | High                      |
| Capability           | Does the capability create useful control?          | High                      |
| Cron job             | Does root execute user-controlled content?          | High                      |
| Credential           | Does it belong to a higher-privileged account?      | High                      |
| Docker access        | Can runtime control affect host privilege?          | High                      |
| Writable directory   | Is it trusted by privileged execution?              | Medium                    |
| Open port            | Is there a relevant privileged service?             | Medium                    |
| Old software         | Is the exact system vulnerable and exploitable?     | Medium                    |
| Ordinary file        | Does it participate in privileged execution?        | Low until proven relevant |

---

## Decision-Driven Enumeration

Instead of:

```text
"What's the next command?"
```

ask:

```text
"What question remains unanswered?"
```

Examples:

### You Found a Root Script

Question:

```text
Who executes it?
```

Next action:

```bash id="p2p4j5"
grep -R "/path/to/script" /etc/cron* /etc/systemd 2>/dev/null
```

### You Found a Writable Service Configuration

Question:

```text
Does the service actually load it?
```

Next action:

```bash id="u8j7i6"
systemctl cat <service>
```

### You Found a Credential

Question:

```text
Which account owns it?
```

Next action:

```text
Trace the configuration or service context.
```

### You Found a SUID Binary

Question:

```text
What does it actually do?
```

Next action:

```text
Inspect its identity, permissions, behavior, and documented purpose.
```

---

## Time-Boxed Priority

When time is limited, use:

### First 5 Minutes

```text
Identity
System
Groups
Sudo
Processes
Services
SUID
Capabilities
Cron
```

### Next 10 Minutes

```text
Filesystem
Writable resources
Credentials
Network
Containers
Special groups
Timers
```

### Then

```text
Automated enumeration
Finding correlation
Validation
Exploitation
```

This prevents spending the entire assessment searching for one obscure possibility.

---

## Priority Reassessment

Priority can change.

Example:

```text
Initial:
Old software → Medium

New discovery:
Known vulnerable version
+
Required configuration
+
Relevant privilege
```

Priority becomes:

```text
Immediate investigation
```

Likewise:

```text
Initial:
Writable file → High
```

may become:

```text
Low
```

after discovering:

```text
No privileged process consumes it.
```

Always update your priority based on evidence.

---

## Avoid Tunnel Vision

A common mistake is finding one interesting path and spending the entire assessment on it.

Use:

```text
Candidate Path A
      ↓
Validate quickly
      ↓
Valid?
 ┌────┴────┐
YES       NO
 │         │
Deep      Move on
 │
Exploit
```

If validation is taking too long without producing evidence, temporarily return to the broader enumeration tree.

---

## Parallel Candidates

Sometimes several paths deserve investigation.

Example:

```text
Candidate A
Writable root script

Candidate B
Interesting SUID binary

Candidate C
Privileged credential

Candidate D
Docker group
```

Maintain a small candidate list:

| Candidate | Privileged? | User Control? | Execution Known? | Next Step        |
| --------- | ----------: | ------------: | ---------------: | ---------------- |
| A         |         Yes |           Yes |              Yes | Validate         |
| B         |         Yes |       Unknown |          Unknown | Inspect binary   |
| C         |    Possible |           Yes |          Unknown | Identify account |
| D         |         Yes |           Yes |              Yes | Analyze runtime  |

This prevents losing useful findings while investigating one path.

---

## Common Mistakes

### Enumerating Without a Question

Bad:

```text
Run commands because they are in a cheat sheet.
```

Better:

```text
Identify the unanswered question first.
```

### Treating Output as Conclusions

Bad:

```text
LinPEAS says suspicious.
Therefore exploitable.
```

Better:

```text
Tool found suspicious condition.
Reproduce it.
Understand it.
Validate it.
```

### Spending Too Long on Low-Value Findings

A normal file should not receive the same investigation depth as a root-executed writable script.

### Ignoring the Current User

Every permission decision depends on the current identity.

### Ignoring Groups

Group membership can create access that is not obvious from file permissions alone.

### Jumping to Kernel Exploitation

Kernel exploitation should not be the automatic answer to an old kernel.

First check simpler, more directly controllable paths.

---

## Priority Checklist

```text
[ ] Who am I?
[ ] What groups do I have?
[ ] What is the current privilege?
[ ] Is sudo useful?
[ ] What executes as root?
[ ] What can I control?
[ ] Are there useful SUID files?
[ ] Are there useful capabilities?
[ ] Are there root cron jobs?
[ ] Are there systemd timers?
[ ] Are there writable privileged resources?
[ ] Are credentials exposed?
[ ] Are privileged services reachable?
[ ] Are special groups present?
[ ] Are containers relevant?
[ ] Are mounted filesystems relevant?
[ ] Is vulnerable software actually applicable?
[ ] What did automated enumeration reveal?
[ ] Which finding has the strongest complete path?
[ ] Has the path been validated?
```

---

## Final Mental Model

Remember enumeration priority as:

```text
IDENTITY
   ↓
PRIVILEGE
   ↓
CONTROL
   ↓
TRUST
   ↓
EXECUTION
   ↓
IMPACT
```

And remember the most important prioritization rule:

> **Investigate the shortest, clearest path from something you control to something privileged.**

Do not optimize for the number of commands executed.

Optimize for the number of **useful questions answered**.
