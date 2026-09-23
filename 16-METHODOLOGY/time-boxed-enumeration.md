# Linux Privilege Escalation Time-Boxed Enumeration

> **Purpose:** Perform structured Linux privilege-escalation enumeration under time constraints without sacrificing reasoning, validation, or attack-path analysis.

---

## Core Principle

Enumeration should not consume the entire assessment.

The goal is to reach:

```text id="m8zq3d"
CONTEXT
   ↓
DISCOVERY
   ↓
PRIORITIZATION
   ↓
VALIDATION
   ↓
EXPLOITATION
   ↓
VERIFICATION
```

A useful assessment is not the one where the most commands were executed.

It is the one where the most relevant privilege relationships were identified and validated.

---

## Why Time-Box Enumeration

Without time limits, enumeration can become:

```text id="4j6q0p"
Command
  ↓
More output
  ↓
More files
  ↓
More possibilities
  ↓
More commands
  ↓
No decision
```

Time-boxing forces:

```text id="j8x3s7"
Question
  ↓
Command
  ↓
Interpretation
  ↓
Decision
```

The objective is to prevent endless enumeration.

---

## The Master Time Model

Use four stages:

```text id="2k9j7p"
0–5 MINUTES
Context + Fast Wins

5–15 MINUTES
Structured Enumeration

15–30 MINUTES
Deep Investigation

30+ MINUTES
Targeted Validation / Exploitation
```

These are practical guidance, not rigid deadlines.

A strong finding can immediately change the schedule.

---

## Phase 0: Before Enumeration

Before touching the system, confirm:

```text id="8x3qg2"
[ ] Authorization exists
[ ] Scope is known
[ ] Target is known
[ ] Testing constraints are known
[ ] Destructive actions are prohibited unless explicitly authorized
```

Do not start exploitation before understanding the engagement boundaries.

---

## Phase 1: First 5 Minutes

### Objective

Answer:

```text id="v6x3r0"
Who am I?
What system am I on?
What groups do I have?
What obvious privileged mechanisms exist?
```

---

## Minute 0–1: Identity

Run:

```bash id="0f5rj2"
whoami
id
id -u
id -g
```

Record:

```text id="6ax3zq"
Username
UID
Primary group
Supplementary groups
```

### Decision

```text id="q9b6dm"
UID 0?
 ├── YES → Verify and document
 └── NO  → Continue
```

---

## Minute 1–2: System Context

Run:

```bash id="p8y2x0"
hostname
uname -a
cat /etc/os-release
uname -m
```

Determine:

```text id="d0v6qt"
Hostname
Distribution
Version
Kernel
Architecture
```

### Why

This gives immediate context for:

* software compatibility
* kernel research
* architecture-specific behavior
* system configuration
* later vulnerability validation

---

## Minute 2–3: Sudo and Groups

Run:

```bash id="m9h1vr"
sudo -l
id
```

Look for:

```text id="8x4l7b"
Sudo permissions
Privileged groups
Unusual memberships
```

If `sudo -l` reveals a clearly useful authorized path, investigate it immediately.

---

## Minute 3–4: Privileged Execution

Run:

```bash id="t2c6f1"
ps aux
```

Then:

```bash id="z5e7v2"
systemctl list-units --type=service --state=running
```

Look for:

```text id="4f1n8m"
Root-owned custom processes
Custom services
Unusual applications
Suspicious scripts
```

---

## Minute 4–5: SUID and Capabilities

Run:

```bash id="6z0f4n"
find / -perm -4000 -type f 2>/dev/null
```

Then:

```bash id="y3m9s2"
getcap -r / 2>/dev/null
```

Do not investigate every result.

Look for:

```text id="9k3b6x"
Unusual
+
Privileged
+
Potentially controllable
```

---

## Five-Minute Decision Point

At the end of five minutes, ask:

```text id="8d4q4p"
Did I find a direct candidate?
```

### If Yes

Stop broad enumeration.

```text id="r1w9s8"
Candidate
   ↓
Investigate
   ↓
Validate
```

### If No

Continue structured enumeration.

---

## Phase 2: 5–15 Minutes

### Objective

Expand coverage without going deeply into every result.

Focus on:

```text id="w7p3k2"
Filesystem
Scheduled execution
Credentials
Network
Special environments
Writable resources
```

---

## Minute 5–7: Scheduled Execution

Inspect:

```bash id="3v7n6c"
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
```

Then:

```bash id="q0m7s8"
systemctl list-timers --all
```

Look for:

```text id="5x9p4d"
Root execution
+
Custom scripts
+
Writable resources
```

---

## Minute 7–9: Filesystem

Inspect important locations:

```bash id="w4r2k8"
ls -la /
ls -la /opt/
ls -la /usr/local/
ls -la /tmp/
```

Look for:

```text id="z5y8n1"
Custom applications
Scripts
Backups
Configuration files
Writable directories
Unusual binaries
```

The goal is not to recursively inspect the entire filesystem.

---

## Minute 9–11: Credentials

Check obvious locations and context:

```bash id="4v6r9k"
history 2>/dev/null
env
```

Then investigate application configuration and user-accessible files where appropriate.

Prioritize credentials belonging to:

```text id="9w2d7s"
Root
Privileged users
Service accounts
Administrative applications
```

Do not unnecessarily print or expose secret values.

---

## Minute 11–13: Network

Run:

```bash id="n4c7p3"
ip addr
ip route
ss -lntup
```

Focus on:

```text id="3d8y5m"
Localhost services
Custom services
Administrative interfaces
Privileged applications
```

---

## Minute 13–15: Special Environments

Check:

```bash id="f5n2h7"
id
findmnt
```

Look for indicators of:

```text id="m1p6v8"
Docker
LXD
NFS
Containers
Special mounts
Virtualization
Privileged groups
```

If a relevant runtime is found, investigate it instead of continuing generic enumeration.

---

## Fifteen-Minute Decision Point

Ask:

```text id="8h0k5c"
Do I have a credible attack-path candidate?
```

A credible candidate should have at least:

```text id="x2m7k9"
Privileged component
+
Some user control
+
A plausible execution relationship
```

### If Yes

Move into deep investigation.

### If No

Run automated enumeration and expand targeted coverage.

---

## Phase 3: 15–30 Minutes

### Objective

Stop collecting broad information.

Start connecting findings.

The workflow becomes:

```text id="n0j6r4"
FINDING
   ↓
OWNERSHIP
   ↓
PERMISSIONS
   ↓
TRUST
   ↓
EXECUTION
   ↓
PRIVILEGE
```

---

## Candidate Investigation

For each candidate, answer:

```text id="v5s3h2"
What is privileged?
Who owns it?
Who executes it?
Who can modify it?
What does it trust?
When does it execute?
What privilege does it provide?
```

---

## Root Service Investigation

If you find an interesting service:

```bash id="u6w8j1"
systemctl status <service>
systemctl cat <service>
```

Then inspect referenced resources:

```bash id="b4k0x7"
ls -la <resource>
stat <resource>
```

Decision:

```text id="x6m8v0"
Root service
   ↓
User-controlled dependency?
   ├── NO → Move on
   └── YES → Validate
```

---

## Cron Investigation

If you find a suspicious cron entry:

```bash id="f4v7s3"
cat /etc/crontab
```

Trace the referenced command or script.

Then:

```bash id="y8k2d6"
ls -la <script>
stat <script>
```

Ask:

```text id="r4m1j5"
Who runs it?
Can I modify it?
Does it execute?
When does it execute?
```

---

## SUID Investigation

For an unusual SUID binary:

```bash id="p8x5m9"
ls -la <binary>
file <binary>
```

Determine:

```text id="w3v9c2"
Owner
Permissions
Architecture
Purpose
Expected behavior
```

Then determine whether the current user can influence its privileged behavior.

---

## Capability Investigation

For an unusual capability:

```bash id="a7n2f5"
getcap <binary>
```

Then determine:

```text id="j0m8r6"
What capability?
Why is it present?
What can the binary do?
Can the current user execute it?
Can its behavior be controlled?
```

---

## Credential Investigation

For a discovered credential:

```text id="z4v7h3"
Identify owner
     ↓
Identify service/account
     ↓
Determine privilege
     ↓
Determine whether authorized use is possible
     ↓
Validate
```

Do not assume a credential automatically provides escalation.

---

## Phase 4: 30+ Minutes

At this point, the assessment should become highly targeted.

Possible activities:

```text id="c8s5w2"
Deep validation
Controlled exploitation
Alternative attack-path testing
Targeted vulnerability research
Evidence collection
Privilege verification
```

Avoid returning to unrestricted enumeration unless new evidence requires it.

---

## When to Stop Enumerating

Stop broad enumeration when:

```text id="n8c1v6"
A complete candidate path exists
```

For example:

```text id="k5y7q2"
Current User
   ↓
Writable Resource
   ↓
Root Process
   ↓
Known Execution
   ↓
Known Privilege Impact
```

At this point, additional generic enumeration may have lower value than validation.

---

## When to Continue Enumerating

Continue when:

```text id="j3p6s8"
No credible path exists
```

or:

```text id="w8d4q0"
Current candidate has a broken relationship
```

or:

```text id="m5r7n2"
Required prerequisite is missing
```

Then return to the enumeration tree.

---

## Time-Box Failure Recovery

If one candidate consumes too much time:

```text id="v4y8c1"
Candidate
   ↓
Investigation
   ↓
No clear progress
```

Set it aside.

Then:

```text id="r6k0p3"
Return to candidate list
   ↓
Select next credible path
```

Do not allow one uncertain finding to consume the entire assessment.

---

## The 10-Minute Rule

A useful practical rule:

> **If a candidate is producing no new evidence after several focused checks, pause it and investigate another candidate.**

This is not an absolute timer.

The rule exists to prevent tunnel vision.

A high-value finding can justify substantially more investigation.

---

## Candidate Queue

Maintain a simple queue:

| Candidate     | Evidence          | Confidence | Next Question                  |
| ------------- | ----------------- | ---------- | ------------------------------ |
| Root service  | Root execution    | High       | Who controls dependency?       |
| SUID binary   | SUID permission   | Medium     | What behavior is controllable? |
| Credential    | Secret discovered | Medium     | Which account owns it?         |
| Writable file | User write access | Low        | Who consumes it?               |

Update the queue as evidence changes.

---

## Time-Boxed Workflow

```text id="v0d6m1"
0–5 MIN
│
├── Identity
├── System
├── Groups
├── Sudo
├── Processes
├── Services
├── SUID
└── Capabilities
       │
       ▼
5–15 MIN
│
├── Cron
├── Timers
├── Filesystem
├── Credentials
├── Network
└── Special Environments
       │
       ▼
15–30 MIN
│
├── Correlate Findings
├── Trace Trust
├── Trace Execution
├── Validate Candidates
└── Select Attack Path
       │
       ▼
30+ MIN
│
├── Controlled Exploitation
├── Verification
├── Evidence
└── Documentation
```

---

## Automated Enumeration Timing

Automated tools are most useful after establishing a basic context.

Recommended sequence:

```text id="b7s2q4"
Manual Baseline
      ↓
Fast Enumeration
      ↓
Automated Enumeration
      ↓
Correlate Findings
      ↓
Manual Validation
```

Do not run an automated tool and blindly follow its entire output.

---

## Tool Output Triage

When a tool produces large output, first search for:

```text id="m8w2c5"
sudo
SUID
SGID
capabilities
writable
cron
timer
root
password
credential
docker
lxd
ssh
service
```

Then inspect the surrounding context.

The keyword itself is not the conclusion.

---

## Time-Boxed Validation

When a candidate looks promising:

```text id="q4j6z9"
1. Reproduce the finding
2. Confirm privilege context
3. Confirm user control
4. Confirm execution
5. Confirm expected impact
6. Perform minimal controlled validation
7. Verify result
```

Do not skip directly from:

```text id="h5v2p7"
Interesting
```

to:

```text id="d8n4s1"
Exploit
```

---

## Time-Boxed Exploitation

Before exploitation:

```text id="g1m7r3"
[ ] Authorized
[ ] Finding validated
[ ] Preconditions satisfied
[ ] Expected privilege known
[ ] Minimal action selected
[ ] Verification method ready
[ ] Cleanup understood
```

Then perform the smallest effective action appropriate to the authorized environment.

---

## Time-Boxed Verification

Immediately verify:

```bash id="x6s8k2"
whoami
id
id -u
```

Then determine:

```text id="n2f5v7"
Did the privilege boundary actually change?
```

If not:

```text id="q7w1c9"
Do not assume failure reason.
Return to validation.
```

---

## Time-Boxed Documentation

Record findings while the path is still fresh.

Minimum record:

```text id="p3k6m8"
Initial User:
Initial UID:

Finding:
Privileged Component:
User Control:
Trust Relationship:
Execution Trigger:

Validation:
Exploitation:
Result:

Final User:
Final UID:

Evidence:
Root Cause:
Cleanup:
```

---

## Example: 15-Minute Assessment

### Minutes 0–5

Discovery:

```text id="2k7s4f"
User: low privilege
Groups: docker
Sudo: no useful permissions
SUID: standard binaries
Capabilities: nothing obvious
```

Candidate:

```text id="v6m8q1"
Docker group
```

### Minutes 5–10

Investigate:

```text id="7x3r9c"
Docker runtime available
Socket accessible
```

Now determine:

```text id="c8p5m2"
Does runtime access cross the host privilege boundary?
```

### Minutes 10–15

Validate the relationship in the authorized lab.

If confirmed:

```text id="r2w7h5"
Stop generic enumeration
Document path
Proceed to controlled exploitation
```

The important point is that enumeration stopped because a complete candidate path existed.

---

## Example: No Immediate Path

After 15 minutes:

```text id="8n4m2k"
No useful sudo
No obvious SUID path
No useful capabilities
No writable root scripts
No relevant credentials
No useful container access
```

Do not conclude:

```text id="c5q9v1"
"No privilege escalation exists."
```

Instead:

```text id="w7h3p6"
Run automated enumeration
      ↓
Inspect custom services
      ↓
Inspect application configurations
      ↓
Review software versions
      ↓
Search for indirect trust relationships
```

---

## Time-Boxing Does Not Mean Rushing

The purpose is not:

```text id="e4j6m8"
Do everything quickly.
```

It is:

```text id="r5p8x2"
Spend time where evidence justifies spending time.
```

A clearly validated path deserves attention.

A weak theoretical possibility does not deserve unlimited investigation.

---

## Common Mistakes

### Mistake 1: Spending 30 Minutes on SUID

Finding many SUID files does not mean every binary deserves equal attention.

### Mistake 2: Ignoring Sudo

A single `sudo -l` result can be more valuable than hundreds of filesystem findings.

### Mistake 3: Running Tools Without Interpretation

Large output is not the same as useful information.

### Mistake 4: Never Stopping Enumeration

Once a complete attack path exists, validate it.

### Mistake 5: Exploiting Too Early

A finding without a complete path can waste time and create unnecessary risk.

### Mistake 6: Tunnel Vision

If one candidate stops producing evidence, return to the candidate queue.

### Mistake 7: Treating Time as the Only Priority

A 10-minute-old finding with a complete path can be more valuable than a 30-second-old theoretical vulnerability.

---

## Quick Time-Box Checklist

### First 5 Minutes

```text id="w5g8k1"
[ ] whoami
[ ] id
[ ] hostname
[ ] uname -a
[ ] /etc/os-release
[ ] sudo -l
[ ] groups
[ ] processes
[ ] services
[ ] SUID
[ ] capabilities
```

### 5–15 Minutes

```text id="f3c7z2"
[ ] cron
[ ] systemd timers
[ ] writable resources
[ ] important filesystem locations
[ ] history
[ ] environment
[ ] credentials
[ ] network
[ ] mounts
[ ] containers
```

### 15–30 Minutes

```text id="j8m4r6"
[ ] Correlate findings
[ ] Identify privileged components
[ ] Identify user control
[ ] Trace trust
[ ] Trace execution
[ ] Build attack path
[ ] Validate
```

### 30+ Minutes

```text id="k2p5v9"
[ ] Controlled exploitation
[ ] Verify privilege
[ ] Collect evidence
[ ] Document root cause
[ ] Cleanup
```

---

## Final Mental Model

Remember time-boxed enumeration as:

```text id="y7q3m5"
5 MINUTES
Find the obvious.

        ↓

15 MINUTES
Build the attack surface.

        ↓

30 MINUTES
Connect and validate the strongest paths.

        ↓

AFTER 30
Exploit, verify, document.
```

The deeper rule is:

> **Do not measure enumeration by how much information you collected. Measure it by how quickly you can prove or reject meaningful privilege-escalation paths.**
