# Linux Privilege Escalation Quick Reference Decision Tree

> **Purpose:** Provide a compact decision tree for rapidly choosing the next enumeration or validation step during an authorized Linux privilege-escalation assessment.

---

## Start Here

After obtaining a shell:

```text id="8k3m7p"
START
  ↓
WHO AM I?
  ↓
WHAT SYSTEM AM I ON?
  ↓
WHAT GROUPS DO I HAVE?
  ↓
WHAT PRIVILEGED MECHANISMS EXIST?
```

Run:

```bash id="5q8m2x"
whoami
id
hostname
uname -a
cat /etc/os-release
```

---

## Master Decision Tree

```text id="7m4x9q"
LOW-PRIVILEGED SHELL
        │
        ▼
     WHOAMI?
        │
        ▼
      UID 0?
     ┌──┴──┐
    YES    NO
     │      │
     │      ▼
     │    SUDO?
     │   ┌──┴──┐
     │  YES    NO
     │   │      │
     │   ▼      ▼
     │ INVESTIGATE
     │        SUID / SGID
     │             │
     │             ▼
     │        CAPABILITIES
     │             │
     │             ▼
     │     PROCESSES / SERVICES
     │             │
     │             ▼
     │       CRON / TIMERS
     │             │
     │             ▼
     │      WRITABLE RESOURCES
     │             │
     │             ▼
     │        CREDENTIALS
     │             │
     │             ▼
     │ NETWORK / SPECIAL ENVIRONMENTS
     │             │
     │             ▼
     │     VULNERABLE SOFTWARE
     │             │
     │             ▼
     │       AUTOMATED ENUMERATION
     │
     ▼
VERIFY + DOCUMENT
```

---

## Decision 1 — Is the Current User Already Root?

Run:

```bash id="2x7m5p"
id -u
```

### If `0`

```text id="5m8q3x"
Verify:
    whoami
    id
```

Then:

```text id="7p4m9x"
Document
   ↓
Do not perform unnecessary escalation
```

### If Not `0`

Continue.

---

## Decision 2 — Are Sudo Permissions Useful?

Run:

```bash id="9q3m6x"
sudo -l
```

### If Useful

Ask:

```text id="4x8m2p"
What command?
As which user?
What restrictions?
Can its behavior be controlled?
```

Then:

```text id="6m7q4v"
Candidate
   ↓
Validate
```

### If Not Useful

Move on.

---

## Decision 3 — Are There Interesting SUID Files?

Run:

```bash id="3p8m5q"
find / -perm -4000 -type f 2>/dev/null
```

For an unusual binary:

```bash id="7x2m9p"
ls -la <binary>
file <binary>
```

Ask:

```text id="5m4q8x"
Can I execute it?
What does it do?
Can I influence its behavior?
```

### If No Control

Move on.

### If Control Exists

Validate.

---

## Decision 4 — Are There Interesting Capabilities?

Run:

```bash id="8q5m3x"
getcap -r / 2>/dev/null
```

Ask:

```text id="2m7p6v"
What capability?
Which binary?
What does the binary do?
Can I execute it?
Can I influence it?
```

### If Relevant

Validate.

### If Not

Move on.

---

## Decision 5 — Is a Root Process Interesting?

Run:

```bash id="4x7m2p"
ps aux
```

Find root processes.

Ask:

```text id="9m5q3x"
What executable?
What arguments?
What configuration?
What files?
What dependencies?
Who can modify them?
```

### If User Control Exists

Trace the execution path.

### If No User Control

Move on.

---

## Decision 6 — Is a Root Service Interesting?

Run:

```bash id="6p8m4q"
systemctl list-units --type=service --state=running
```

Inspect:

```bash id="3m7x9p"
systemctl status <service>
systemctl cat <service>
```

Ask:

```text id="8q2m5v"
Who runs it?
What does it execute?
What does it trust?
Who can modify dependencies?
```

### If User Control Exists

Validate.

### If Not

Move on.

---

## Decision 7 — Is Scheduled Execution Interesting?

### Cron

```bash id="5x9m2q"
cat /etc/crontab
ls -la /etc/cron.d/
```

### Timers

```bash id="7m4p8x"
systemctl list-timers --all
```

Ask:

```text id="2q6m9v"
Who executes the task?
What resource is executed?
Can I modify it?
When does it execute?
```

### If Root Executes User-Controlled Content

Prioritize the candidate.

### If Not

Move on.

---

## Decision 8 — Is a Writable Resource Relevant?

Finding something writable is not enough.

Ask:

```text id="8m3x7q"
Who uses it?
Who executes it?
Is the consumer privileged?
When is it consumed?
```

Decision:

```text id="5q9m2p"
Writable Resource
      │
      ▼
Privileged Consumer?
   ┌──┴──┐
  NO    YES
   │      │
MOVE     USER CONTROL
ON        │
          ▼
       VALIDATE
```

---

## Decision 9 — Is a Credential Relevant?

If a credential is discovered:

```text id="3x8m5p"
Credential
   ↓
Who owns it?
   ↓
Where is it used?
   ↓
What privilege does the account have?
```

Decision:

```text id="7m2q9x"
Higher privilege?
 ┌────┴────┐
NO        YES
 │          │
MOVE       Validate
ON
```

Do not assume the credential provides escalation simply because it is valid.

---

## Decision 10 — Is PATH Relevant?

Run:

```bash id="4q8m3p"
echo "$PATH"
```

Ask:

```text id="6m5x9v"
Does privileged execution use PATH?
Can the current user control a searched directory?
Does the privileged process actually invoke the affected command?
```

All relationships must be established.

---

## Decision 11 — Is a Wildcard Relevant?

If a privileged command uses wildcards:

```text id="8x2m7q"
What command?
Which directory?
Who controls that directory?
How are filenames interpreted?
Can filename-controlled input alter execution?
```

If the relationship cannot be demonstrated:

```text id="5m9p3x"
Move on.
```

---

## Decision 12 — Is a Library or Environment Dependency Relevant?

Ask:

```text id="2q7m4x"
What dependency?
Who controls it?
Who loads it?
Under what privilege?
When is it loaded?
```

Decision:

```text id="9m3x8p"
User Control
     ↓
Privileged Loader
     ↓
Actual Load
     ↓
Validate
```

---

## Decision 13 — Is a Special Group Relevant?

Run:

```bash id="6x4m8q"
id
```

If an unusual group exists:

```text id="3m7p5v"
What resources does the group control?
Can those resources affect privileged execution?
```

Examples:

```text id="8q2m6x"
docker
lxd
disk
libvirt
```

Group membership must be evaluated on the actual system.

---

## Decision 14 — Is Docker Relevant?

Check:

```bash id="5m8x3q"
docker ps
ls -l /var/run/docker.sock 2>/dev/null
```

Then ask:

```text id="7q4m9p"
Can the current user control the runtime?
Can that control affect host resources?
```

If yes:

```text id="2x6m8v"
Prioritize
   ↓
Validate
```

---

## Decision 15 — Is NFS Relevant?

Ask:

```text id="9m3q7x"
What is exported?
Who can access it?
What options are configured?
Can the current user influence privileged resources through it?
```

NFS presence alone is not sufficient.

---

## Decision 16 — Is a Mounted Filesystem Relevant?

Run:

```bash id="4x8m2p"
findmnt
```

Ask:

```text id="6m5q9v"
What is mounted?
Who controls it?
What files are exposed?
Can the mount affect privileged execution?
```

---

## Decision 17 — Is a Kernel Vulnerability Relevant?

Collect:

```bash id="3q7m8x"
uname -a
cat /etc/os-release
uname -m
```

Then check:

```text id="5m2p9q"
Exact version
Distribution patch status
Architecture
Configuration
Prerequisites
Privilege impact
```

Decision:

```text id="8x4m6v"
Applicable?
 ┌────┴────┐
NO        YES
 │          │
MOVE      Validate
ON
```

---

## Decision 18 — No Obvious Path

If all major mechanisms appear unproductive:

```text id="7m5q2x"
Run automated enumeration
```

Then:

```text id="9p3m8v"
Tool finding
   ↓
Manual reproduction
   ↓
Attack-path analysis
   ↓
Validation
```

Do not treat automated output as proof.

---

## Finding Decision

Every finding enters this branch:

```text id="2x6m4p"
FINDING
   ↓
PRIVILEGED COMPONENT?
   │
 ┌─┴─┐
NO  YES
│     │
MOVE  ↓
ON   USER CONTROL?
       │
      ┌┴┐
     NO YES
      │  │
     MOVE ↓
     ON  EXECUTION?
          │
         ┌┴┐
        NO YES
         │  │
        MOVE ↓
        ON  PRIVILEGE IMPACT?
             │
            ┌┴┐
           NO YES
            │  │
           MOVE ↓
           ON VALIDATE
```

---

## Validation Decision

Before exploitation:

```text id="5q8m3x"
Can I reproduce the finding?
        ↓
Can I prove user control?
        ↓
Can I prove privileged execution?
        ↓
Can I prove the trust relationship?
        ↓
Can I predict the privilege impact?
```

### If Any Critical Answer Is No

Investigate the missing relationship.

### If All Are Yes

Proceed to controlled exploitation within scope.

---

## Exploitation Decision

```text id="8m4p6q"
VALIDATED PATH
      ↓
AUTHORIZED?
   ┌──┴──┐
  NO    YES
   │      │
STOP     ↓
       MINIMAL ACTION
           ↓
       VERIFY RESULT
```

---

## Verification Decision

Run:

```bash id="3x7m9p"
whoami
id
id -u
```

Ask:

```text id="6m2q8v"
Did the privilege boundary actually change?
```

### Yes

```text id="5p9m3x"
Evidence
   ↓
Document
   ↓
Cleanup
```

### No

```text id="7q4x8m"
Do not guess
   ↓
Return to validation
```

---

## Stuck Decision

If investigation stalls:

```text id="2m6p9x"
What question remains unanswered?
```

Return to:

```text id="8q3m5v"
WHO?
WHAT?
WHO OWNS IT?
WHO CAN MODIFY IT?
WHAT DOES IT TRUST?
WHEN DOES IT EXECUTE?
CAN I INFLUENCE IT?
```

---

## Fast 5-Minute Branch

```text id="4x7m2q"
whoami
   ↓
id
   ↓
uname -a
   ↓
cat /etc/os-release
   ↓
sudo -l
   ↓
ps aux
   ↓
systemctl list-units --type=service --state=running
   ↓
find / -perm -4000 -type f 2>/dev/null
   ↓
getcap -r / 2>/dev/null
```

Then:

```text id="9m5p8x"
CANDIDATE?
 ┌───┴───┐
YES     NO
 │       │
TRACE   CRON
 │       │
VALIDATE TIMERS
         │
         FILESYSTEM
         │
         CREDENTIALS
         │
         NETWORK
```

---

## Fast 15-Minute Branch

```text id="7q2m4p"
FIRST 5 MINUTES
      ↓
CRON
      ↓
SYSTEMD TIMERS
      ↓
WRITABLE RESOURCES
      ↓
CONFIGURATIONS
      ↓
CREDENTIALS
      ↓
NETWORK
      ↓
MOUNTS
      ↓
CONTAINERS
      ↓
CORRELATE
      ↓
VALIDATE
```

---

## Complete Decision Tree

```text id="3m8x5q"
LOW-PRIVILEGED SHELL
        │
        ▼
    IDENTITY
        │
        ▼
     CONTEXT
        │
        ▼
    PRIVILEGE
        │
        ├───────────────┐
        ▼               ▼
      SUDO          GROUPS
        │               │
        └───────┬───────┘
                ▼
       PRIVILEGED EXECUTION
                │
        ┌───────┼────────┐
        ▼       ▼        ▼
       SUID   SERVICES   CAPS
        │       │        │
        └───────┼────────┘
                ▼
       SCHEDULED EXECUTION
                │
           ┌────┴────┐
           ▼         ▼
         CRON      TIMERS
           │         │
           └────┬────┘
                ▼
       USER-CONTROLLED RESOURCE?
                │
             ┌──┴──┐
            NO    YES
             │      │
          MOVE ON   ▼
                 VALIDATE
                    │
                    ▼
               PRIVILEGE IMPACT?
                    │
                 ┌──┴──┐
                NO    YES
                 │      │
              MOVE ON   ▼
                      EXPLOIT
                         │
                         ▼
                      VERIFY
                         │
                         ▼
                     DOCUMENT
```

---

## Decision Rules to Memorize

### Rule 1

> **A finding is not an attack path.**

### Rule 2

> **A privileged component matters only when you can identify a relevant influence over it.**

### Rule 3

> **A writable resource matters when a privileged consumer trusts it.**

### Rule 4

> **A credential matters when it provides access to a more privileged context.**

### Rule 5

> **An exploit is the final step, not the first question.**

### Rule 6

> **If a path cannot be validated, do not force it.**

---

## Quick Decision Checklist

```text id="5x8m2q"
[ ] Who am I?
[ ] What UID?
[ ] What groups?
[ ] What OS?
[ ] What kernel?
[ ] Is sudo useful?
[ ] What runs as root?
[ ] What is SUID?
[ ] What capabilities exist?
[ ] What executes automatically?
[ ] What can I write?
[ ] What credentials can I access?
[ ] What services are listening?
[ ] Are containers relevant?
[ ] Are mounts relevant?
[ ] Is vulnerable software applicable?
[ ] What is the strongest candidate?
[ ] Can I prove the attack path?
[ ] Can I validate it safely?
[ ] Did privilege actually change?
[ ] Did I document the path?
```

---

## Final Mental Model

The entire decision tree reduces to:

```text id="8m3q6x"
FIND
 ↓
PRIVILEGED?
 ↓
CONTROLLED?
 ↓
TRUSTED?
 ↓
EXECUTED?
 ↓
IMPACTFUL?
 ↓
VALIDATE
 ↓
EXPLOIT
 ↓
VERIFY
```

> **When you are unsure what to do next, do not ask "Which command should I run?" Ask "Which relationship is still unproven?"**
