# Linux Privilege Escalation Decision Tree

> **Purpose:** Provide a fast, decision-driven map for determining what to investigate next during Linux privilege-escalation testing.

---

## Core Question

At any point during an assessment, ask:

> **What did I just discover, what privilege does it involve, and what should I check next?**

The decision tree follows:

```text
DISCOVER
   ↓
CLASSIFY
   ↓
UNDERSTAND
   ↓
CHECK USER CONTROL
   ↓
VALIDATE
   ↓
EXPLOIT
   ↓
VERIFY
```

Do not treat every finding as an exploit.

---

## Start Here

When you obtain a low-privileged shell:

```bash id="8w1qwm"
whoami
id
hostname
uname -a
cat /etc/os-release
```

Then ask:

```text id="x98cne"
Who am I?
What privilege do I have?
What system am I on?
What is running?
What privileged mechanisms exist?
```

---

## Master Decision Tree

```text id="x2q3pm"
START
  │
  ▼
Establish identity
  │
  ▼
Establish baseline
  │
  ▼
Enumerate privileged mechanisms
  │
  ├── SUDO
  │
  ├── SUID / SGID
  │
  ├── CAPABILITIES
  │
  ├── PROCESSES / SERVICES
  │
  ├── CRON / TIMERS
  │
  ├── CREDENTIALS
  │
  ├── WRITABLE RESOURCES
  │
  ├── CONTAINERS / SPECIAL GROUPS
  │
  └── VULNERABLE SOFTWARE
```

Every branch follows:

```text id="5z9qpr"
Finding
   ↓
Who owns it?
   ↓
Who executes it?
   ↓
Who can modify/control it?
   ↓
Can it affect a privileged action?
   ↓
Validate
```

---

## Identity Decision

Start with:

```bash id="x3st50"
whoami
id
```

### If UID Is 0

```text id="9s0c0x"
UID 0
 ↓
Verify context
 ↓
Document current privilege
 ↓
Move to post-exploitation analysis
```

Do not continue privilege escalation blindly if the intended privileged identity has already been obtained.

### If UID Is Not 0

Continue enumeration.

---

## Sudo Decision

Run:

```bash id="lyspq7"
sudo -l
```

### If No Useful Sudo Permission Exists

```text id="b5rc4m"
No useful sudo path
       ↓
Move to SUID / capabilities
```

### If Sudo Permission Exists

Ask:

```text id="zx9c6u"
What command?
As which user?
What arguments?
What restrictions?
Can the command execute another program?
Can it read/write attacker-controlled content?
Can it influence the environment?
```

Then:

```text id="3gshv7"
Controllable privileged behavior?
       │
       ├── NO → Move on
       │
       └── YES
             ↓
          Validate
```

---

## SUID Decision

Find SUID files:

```bash id="yq0lqt"
find / -perm -4000 -type f 2>/dev/null
```

### For Each Interesting Binary

Ask:

```text id="8f8oj3"
Who owns it?
What does it do?
Why does it require SUID?
What inputs can I control?
Can it execute another program?
Can it modify privileged resources?
```

Then:

```text id="1dr5s6"
Privileged behavior controllable?
       │
       ├── NO → Move on
       │
       └── YES
             ↓
          Validate
```

---

## SGID Decision

Find SGID files:

```bash id="3w5d6u"
find / -perm -2000 -type f 2>/dev/null
```

Ask:

```text id="q7sp9h"
Which group does it execute as?
What resources does that group control?
Can the current user influence its behavior?
```

Then:

```text id="2cyt90"
User-controlled privileged group action?
       │
       ├── NO → Move on
       │
       └── YES
             ↓
          Validate
```

---

## Capability Decision

Enumerate:

```bash id="g0v5z1"
getcap -r / 2>/dev/null
```

Ask:

```text id="x0o8sg"
What capability?
What operation does it permit?
Which binary has it?
Can I invoke the binary?
Can I control its behavior?
Does the capability affect the privilege boundary?
```

Then:

```text id="2c3z6j"
Useful controllable capability?
       │
       ├── NO → Move on
       │
       └── YES
             ↓
          Validate
```

---

## Process Decision

List processes:

```bash id="5gqz5p"
ps aux
```

### If a Root Process Looks Interesting

Ask:

```text id="h7z2x6"
What executable?
What arguments?
What configuration?
What files?
What directories?
What dependencies?
What environment?
```

Then:

```text id="p2i5e4"
Can current user influence execution?
       │
       ├── NO → Move on
       │
       └── YES
             ↓
          Validate
```

---

## Service Decision

List services:

```bash id="g2k9ak"
systemctl list-units --type=service --state=running
```

Inspect:

```bash id="5ck9zq"
systemctl status <service>
systemctl cat <service>
```

Ask:

```text id="az3d7x"
Who runs the service?
What executable?
What configuration?
What scripts?
What dependencies?
Who can modify them?
```

Then:

```text id="3q4c0j"
Privileged controllable execution?
       │
       ├── NO → Move on
       │
       └── YES
             ↓
          Validate
```

---

## Cron Decision

Inspect:

```bash id="xk9bqn"
cat /etc/crontab
ls -la /etc/cron.d/
```

Also inspect:

```bash id="2o0f5v"
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
```

Ask:

```text id="t5av98"
Who executes the task?
What does it execute?
Who owns the script?
Who owns the directory?
Can I modify it?
When does it run?
```

Then:

```text id="7o8e2d"
Root executes controllable resource?
       │
       ├── NO → Move on
       │
       └── YES
             ↓
          Validate
```

---

## Systemd Timer Decision

List timers:

```bash id="n0h8fr"
systemctl list-timers --all
```

For an interesting timer:

```bash id="2p5njw"
systemctl cat <timer>
```

Then identify the associated service.

Ask:

```text id="w6b9r7"
What service does it trigger?
Who runs the service?
What does the service execute?
Can the current user influence it?
```

Then validate the complete chain.

---

## Writable File Decision

When you find a writable file, do **not** immediately modify it.

Ask:

```text id="e3a7o8"
Who owns it?
Who uses it?
When is it used?
Is it consumed by a privileged process?
Can modifying it affect privileged execution?
```

The key distinction:

```text id="b2d5kq"
Writable file
    ↓
No privileged consumer
    ↓
Probably irrelevant
```

versus:

```text id="9d5l1x"
Writable file
    ↓
Privileged consumer
    ↓
User-controlled influence
    ↓
Attack-path candidate
```

---

## Writable Directory Decision

When you find a writable directory, ask:

```text id="qj2e9h"
What files are created there?
Who creates them?
Who reads them?
Who executes them?
Does a privileged process trust the directory?
```

Investigate:

```text id="9j8jpl"
Writable directory
      ↓
Privileged consumer?
      │
      ├── NO → Move on
      │
      └── YES
            ↓
       Analyze execution path
```

---

## PATH Decision

Inspect:

```bash id="ux8qtd"
echo "$PATH"
```

For a privileged process, determine whether it invokes commands without absolute paths.

Ask:

```text id="1p6t1m"
Does privileged code use a relative command?
Which PATH is used?
Can the current user influence the relevant directory?
Will the privileged process search that directory?
```

The complete relationship must be proven.

---

## Library Loading Decision

When a privileged program loads libraries, ask:

```text id="k9k6hy"
Which library?
Where is it loaded from?
Who controls the path?
Can the current user modify the library or search path?
Does the privileged process actually load it?
```

Do not treat the presence of a library as proof of exploitability.

---

## Environment Variable Decision

If a privileged process depends on environment variables:

```text id="x47sl0"
Which variable?
Who sets it?
Is it preserved?
Does the privileged process trust it?
Can the current user influence it?
```

Then validate.

---

## Command Substitution Decision

If privileged scripts use dynamically constructed commands, ask:

```text id="bq1y3f"
What input controls the command?
How is the input processed?
Is shell interpretation involved?
Does the resulting command execute with higher privilege?
```

A command containing user input is a candidate for analysis, not automatic exploitation.

---

## Wildcard Decision

When privileged commands use wildcards, ask:

```text id="2u6f3c"
What command?
Which directory?
Who controls the directory?
How does the program interpret matched filenames?
Can filename-controlled arguments change behavior?
```

Validate the exact behavior before exploitation.

---

## Credential Decision

When credentials are discovered:

```text id="8udj7d"
Credential found
      ↓
Who owns account?
      ↓
What service?
      ↓
What privilege?
      ↓
Is reuse authorized?
      ↓
Can access be validated?
```

Do not expose secrets unnecessarily.

---

## SSH Key Decision

When an SSH private key is discovered:

```text id="2aqe2g"
Identify key owner
      ↓
Identify target account
      ↓
Determine privilege
      ↓
Check whether use is authorized
      ↓
Validate access
```

A private key is a credential, not automatically a privilege-escalation path.

---

## Container Decision

If Docker is present:

```bash id="t3v7yk"
docker ps
```

Check the socket:

```bash id="b1j9x4"
ls -l /var/run/docker.sock 2>/dev/null
```

Ask:

```text id="7m5g5h"
Who can access the runtime?
What privileges does that access provide?
What host resources are exposed?
Does it cross the relevant boundary?
```

Do not assume:

```text id="xxxy4a"
Container root = host root
```

---

## NFS Decision

When NFS is present, ask:

```text id="0okvce"
What is exported?
Who can access it?
What export options are configured?
How are permissions represented?
Can the current user influence files used by a privileged process?
```

NFS presence alone is not a vulnerability.

---

## Mounted Filesystem Decision

Inspect mounts:

```bash id="w1j5x8"
findmnt
```

Ask:

```text id="11i9gk"
What is mounted?
Who controls it?
What permissions exist?
Is it a host filesystem?
Can the current user modify privileged resources through the mount?
```

---

## Special Group Decision

After:

```bash id="f85mmb"
id
```

If you see an unusual group, ask:

```text id="d4y5zq"
What resources does this group control?
Does membership grant privileged access?
Can that access cross the current privilege boundary?
```

Examples may include groups associated with:

```text id="4n2h8q"
Docker
Disk
Video
Libvirt
LXD
```

Group membership must be analyzed in context.

---

## Kernel Vulnerability Decision

Start with:

```bash id="2o7y3r"
uname -a
cat /etc/os-release
```

Then:

```text id="pqx8m7"
Candidate vulnerability
       ↓
Affected version?
       ↓
Distribution patch?
       ↓
Architecture?
       ↓
Configuration?
       ↓
Prerequisites?
       ↓
Privilege impact?
```

Only then should you consider exploitation.

---

## Automated Tool Decision

If LinPEAS or another enumeration tool reports a finding:

```text id="l9s8f1"
Tool finding
      ↓
What exactly did it detect?
      ↓
Can I reproduce it manually?
      ↓
Is a privileged component involved?
      ↓
Can I control it?
      ↓
Is exploitation applicable?
```

The rule is:

> **Automation finds; manual analysis explains; validation proves.**

---

## Finding Validation Decision

Every promising finding should reach this decision:

```text id="w9m0lq"
Does a privileged component exist?
       │
       ├── NO → Move on
       │
       └── YES
             ↓
Can the current user influence it?
             │
             ├── NO → Move on
             │
             └── YES
                   ↓
Does the influence reach privileged execution?
                   │
                   ├── NO → Move on
                   │
                   └── YES
                         ↓
                   Validate prerequisites
                         │
                         ▼
                   Exploitable?
```

---

## Exploitation Decision

Only after validation:

```text id="8ecqv8"
Validated finding
      ↓
What is the minimum effective action?
      ↓
What privilege should result?
      ↓
Is exploitation authorized?
      ↓
Controlled exploitation
      ↓
Verify
```

---

## Verification Decision

After exploitation:

```bash id="xq7q5j"
whoami
id
id -u
```

Ask:

```text id="i2f9kp"
Did privilege actually change?
```

### If Yes

```text id="qf0v6n"
Record evidence
      ↓
Document path
      ↓
Cleanup if authorized
```

### If No

```text id="c5h8y7"
Do not guess
      ↓
Analyze failure
      ↓
Return to validation
```

---

## Complete Master Decision Tree

```text id="v2yqpr"
LOW-PRIVILEGED SHELL
        │
        ▼
ESTABLISH IDENTITY
        │
        ▼
WHAT IS MY CURRENT PRIVILEGE?
        │
        ▼
ENUMERATE
        │
        ├── SUDO
        │
        ├── SUID / SGID
        │
        ├── CAPABILITIES
        │
        ├── PROCESSES
        │
        ├── SERVICES
        │
        ├── CRON / TIMERS
        │
        ├── WRITABLE RESOURCES
        │
        ├── CREDENTIALS
        │
        ├── EXECUTION ABUSE
        │
        ├── CONTAINERS
        │
        └── VULNERABLE SOFTWARE
        │
        ▼
INTERESTING FINDING?
        │
        ├── NO
        │    ↓
        │  CONTINUE ENUMERATION
        │
        └── YES
             ↓
        PRIVILEGED COMPONENT?
             │
             ├── NO → MOVE ON
             │
             └── YES
                  ↓
             USER CONTROL?
                  │
                  ├── NO → MOVE ON
                  │
                  └── YES
                       ↓
                  TRUST RELATIONSHIP?
                       │
                       ├── NO → MOVE ON
                       │
                       └── YES
                            ↓
                       VALIDATE
                            │
                            ▼
                       EXPLOITABLE?
                            │
                       ┌────┴────┐
                       │         │
                      NO        YES
                       │         │
                    MOVE ON   EXPLOIT
                                 │
                                 ▼
                              VERIFY
                                 │
                                 ▼
                         PRIVILEGE CHANGED?
                                 │
                          ┌──────┴──────┐
                          │             │
                         NO            YES
                          │             │
                    RE-VALIDATE     DOCUMENT
                                        │
                                        ▼
                                   CLEAN UP
```

---

## Stuck-State Recovery

When the investigation stops producing useful findings, do not start guessing.

Return to:

```text id="t4ihf8"
WHO AM I?
     ↓
WHAT IS PRIVILEGED?
     ↓
WHO OWNS IT?
     ↓
WHO CAN MODIFY IT?
     ↓
WHAT DOES IT TRUST?
     ↓
WHEN DOES IT EXECUTE?
     ↓
CAN I INFLUENCE IT?
```

Then identify the missing question.

---

## Quick Reference

### Identity

```bash id="c7c2qp"
whoami
id
id -u
id -g
```

### System

```bash id="b1yy1a"
uname -a
cat /etc/os-release
uname -m
```

### Sudo

```bash id="c2g3k5"
sudo -l
```

### Processes

```bash id="49jzde"
ps aux
```

### Services

```bash id="z4h7fj"
systemctl list-units --type=service --state=running
```

### SUID

```bash id="7rj8me"
find / -perm -4000 -type f 2>/dev/null
```

### Capabilities

```bash id="u2l6wl"
getcap -r / 2>/dev/null
```

### Cron

```bash id="h2j93y"
cat /etc/crontab
ls -la /etc/cron.d/
```

### Timers

```bash id="6s4p4s"
systemctl list-timers --all
```

### Network

```bash id="q9r7mi"
ip addr
ip route
ss -lntup
```

### Mounts

```bash id="8k2s6g"
findmnt
```

### Verification

```bash id="w5wq0g"
whoami
id
id -u
```

---

## Decision Tree Checklist

```text id="t17m7n"
[ ] Establish identity
[ ] Establish baseline
[ ] Check sudo
[ ] Check SUID
[ ] Check SGID
[ ] Check capabilities
[ ] Check processes
[ ] Check services
[ ] Check cron
[ ] Check timers
[ ] Check writable resources
[ ] Check credentials
[ ] Check execution abuse
[ ] Check containers
[ ] Check special groups
[ ] Check mounts
[ ] Check network services
[ ] Check vulnerable software
[ ] Run automated enumeration
[ ] Validate promising findings
[ ] Select exploitation path
[ ] Exploit within scope
[ ] Verify privilege
[ ] Preserve evidence
[ ] Document
[ ] Clean up if authorized
```

---

## Final Mental Model

The decision tree can be remembered as:

```text id="n4m4br"
FIND
 ↓
CLASSIFY
 ↓
ASK WHO?
 ↓
ASK WHAT?
 ↓
ASK WHO CAN CONTROL IT?
 ↓
ASK WHEN IT RUNS
 ↓
ASK WHAT IT TRUSTS
 ↓
VALIDATE
 ↓
EXPLOIT
 ↓
VERIFY
```

The most important decision is not:

> **"Which exploit should I try?"**

It is:

> **"Which privileged action can the current user actually influence?"**

Once that relationship is proven, the correct exploitation path becomes much easier to identify.
