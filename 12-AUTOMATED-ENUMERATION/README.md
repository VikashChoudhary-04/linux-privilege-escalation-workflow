# 12 — Automated Enumeration

Automated enumeration tools can dramatically reduce the time required to discover potential Linux privilege-escalation paths.

Common tools include:

* LinPEAS
* Linux Smart Enumeration
* pspy
* Native Linux utilities combined into custom scripts

But automation should **accelerate reasoning, not replace it**.

The correct workflow is:

```text id="a7k2m9"
MANUAL ENUMERATION
       ↓
AUTOMATED ENUMERATION
       ↓
HIGH-VALUE FINDINGS
       ↓
MANUAL VALIDATION
       ↓
EXPLOITATION
       ↓
VERIFICATION
```

---

## Core Question

> **Can automated tools help me discover privilege-escalation candidates faster, and can I prove which findings are actually exploitable?**

The objective is not:

```text id="z3q8m1"
"Run LinPEAS and find root."
```

The objective is:

```text id="m6x1c7"
Run automation
      ↓
Identify interesting findings
      ↓
Understand why they matter
      ↓
Reproduce manually
      ↓
Validate the privilege boundary
```

---

## Mental Model

Use this chain:

```text id="p4n8w2"
AUTOMATED ENUMERATION
        ↓
MANY OBSERVATIONS
        ↓
FILTER NOISE
        ↓
IDENTIFY HIGH-VALUE FINDINGS
        ↓
UNDERSTAND THE FINDING
        ↓
MANUALLY REPRODUCE
        ↓
VALIDATE
        ↓
EXPLOIT
```

Remember:

> **Automation finds candidates. You prove the attack path.**

---

## 1. Why Automated Enumeration Matters

Manual enumeration is essential, but a complex Linux system may contain:

* Hundreds of processes
* Thousands of files
* Many packages
* Multiple services
* Complex permissions
* Containers
* Custom applications
* Scheduled tasks
* Environment variables
* Credentials
* Kernel information

A good enumeration tool can correlate many of these details quickly.

### 🧠 WHY

Automation helps answer:

```text id="y6m2p9"
What did I miss?
What should I investigate next?
Which findings deserve priority?
```

It should not replace understanding.

---

## 2. Manual vs Automated Enumeration

### Manual Enumeration

Advantages:

* Precise
* Easy to understand
* Low noise
* Good for learning
* Easy to reproduce
* Helps build methodology

Disadvantages:

* Slower
* Easy to forget a category
* Harder to correlate large systems

### Automated Enumeration

Advantages:

* Fast
* Broad coverage
* Correlates information
* Finds unusual configurations
* Useful for time-boxed assessments

Disadvantages:

* Can generate noise
* Can produce false positives
* May miss custom logic
* Findings require interpretation
* Tool output can be overwhelming

### 💡 REMEMBER

Use both:

```text id="v9c3m7"
Manual = Understand
Automation = Accelerate
Validation = Prove
```

---

## 3. When to Run Automation

Automated enumeration can be used at different points.

### Option 1 — Early Automation

```text id="k4m8x2"
Initial shell
    ↓
Basic manual context
    ↓
Automated scan
    ↓
Prioritize findings
```

Useful when:

* Time is limited
* The system is unfamiliar
* You need broad coverage quickly

### Option 2 — After Manual Enumeration

```text id="x7q2n5"
Initial shell
    ↓
Manual enumeration
    ↓
Automated scan
    ↓
Compare results
```

Useful for learning and methodology development.

### Option 3 — Both

The strongest workflow for learning is often:

```text id="p8m3c6"
Manual first
    ↓
Automation
    ↓
Compare
    ↓
Investigate differences
```

---

## 4. LinPEAS

LinPEAS is a widely used Linux privilege-escalation enumeration tool.

Its purpose is to collect and highlight information that may reveal possible escalation paths.

### 🔎 ENUMERATE

First identify whether it already exists:

```bash id="n2x7m4"
command -v linpeas
```

If the script is available locally:

```bash id="w5c8q1"
ls -la /path/to/linpeas.sh
```

Inspect its help/options when appropriate:

```bash id="r7m3p9"
bash /path/to/linpeas.sh -h
```

### 🧠 WHY

LinPEAS can examine areas such as:

* System information
* Users and groups
* Sudo configuration
* SUID/SGID files
* Capabilities
* Processes
* Services
* Cron
* Writable files
* Credentials
* Containers
* Network information
* Kernel information

Exact output varies by version and system.

---

## 5. Running LinPEAS

In an authorized environment, run the tool from a location where you are permitted to execute it.

Example:

```bash id="c6m1x8"
bash ./linpeas.sh
```

If executable permissions are appropriate:

```bash id="q4n8v2"
./linpeas.sh
```

### ⚠️ LOOK FOR

Do not try to read every line immediately.

Prioritize sections related to:

```text id="t9x3m5"
Sudo
SUID / SGID
Capabilities
Writable files
Cron
Services
Processes
Credentials
Containers
Kernel
```

---

## 6. LinPEAS Output Strategy

Large automated output can become overwhelming.

Use a three-pass method.

### Pass 1 — Scan

Look for obvious high-interest findings.

```text id="m7q2c9"
root
sudo
NOPASSWD
SUID
cap_
writable
cron
docker
lxd
password
token
ssh
kernel
```

### Pass 2 — Categorize

Put each finding into a category:

```text id="x3k8n1"
Privilege mechanism
Filesystem
Scheduled execution
Credentials
Execution abuse
Special environment
Software vulnerability
```

### Pass 3 — Validate

Return to the original system and manually verify the finding.

---

## 7. Linux Smart Enumeration

Linux Smart Enumeration, commonly referred to as LSE, is another enumeration tool designed to identify Linux privilege-escalation opportunities.

### 🔎 ENUMERATE

If available locally:

```bash id="p5m9x2"
command -v lse
```

Inspect help:

```bash id="v8c3q6"
bash ./lse.sh -h
```

Run it according to the tool's documented options in the authorized environment.

### 🧠 WHY

LSE can help investigate areas such as:

* System information
* Users
* Groups
* Sudo
* Files
* Processes
* Services
* Credentials
* Network configuration
* Scheduled tasks

The exact output depends on the version and selected checks.

---

## 8. LSE Output Interpretation

Do not treat every highlighted result as an exploit.

Use:

```text id="w2m6q9"
Interesting result
       ↓
What category?
       ↓
Why is it interesting?
       ↓
Can I reproduce it manually?
       ↓
Does it create user control?
       ↓
Does it cross privilege boundary?
```

### 💡 REMEMBER

A scanner's warning means:

> **Investigate this.**

It does not necessarily mean:

> **Exploit this.**

---

## 9. pspy

pspy is useful for observing processes without requiring root privileges in many environments.

Its main value is **dynamic enumeration**.

Static enumeration asks:

```text id="h8x2m4"
What is configured?
```

pspy helps answer:

```text id="c5q7n1"
What actually executes?
```

### 🔎 ENUMERATE

If pspy is available:

```bash id="r3m8v2"
ls -la ./pspy*
```

Run the appropriate binary for the target architecture in the authorized environment.

For example:

```bash id="k6x1p9"
uname -m
```

Then select the compatible build.

### 🧠 WHY

pspy can reveal activity such as:

* Cron jobs
* Scheduled scripts
* Service activity
* Automated commands
* Scripts that execute periodically
* Processes triggered by events

---

## 10. Static vs Dynamic Enumeration

This distinction is extremely important.

### Static Enumeration

You inspect configuration:

```text id="q7m3x8"
Cron file
Service file
Script
Permissions
Configuration
```

### Dynamic Enumeration

You observe what actually happens:

```text id="n4c8p2"
Process starts
    ↓
Command executes
    ↓
Script runs
    ↓
File accessed
```

### 💡 REMEMBER

Use:

```text id="w6m2q9"
Static = "What should happen?"
Dynamic = "What actually happens?"
```

Combining both provides stronger validation.

---

## 11. pspy Workflow

Start pspy in an authorized lab:

```text id="x3q7m1"
pspy
   ↓
Observe processes
   ↓
Identify recurring commands
   ↓
Identify execution user
   ↓
Identify scripts
   ↓
Inspect referenced files
   ↓
Check permissions
   ↓
Validate
```

### ⚠️ LOOK FOR

Examples:

```text id="p9m4c7"
root → /opt/scripts/backup.sh
```

or:

```text id="j2x8n5"
root → /usr/bin/python3 /opt/task.py
```

Then investigate:

```text id="v6q1m3"
Who owns the script?
Can I modify it?
What does it call?
What files does it trust?
```

---

## 12. Automated Tool Selection

Use the tool according to the question.

| Question                    | Useful approach             |
| --------------------------- | --------------------------- |
| Broad Linux enumeration     | LinPEAS                     |
| Structured Linux checks     | LSE                         |
| Dynamic process discovery   | pspy                        |
| Exact validation            | Native commands             |
| Custom application behavior | Manual analysis             |
| Service configuration       | `systemctl`                 |
| Scheduled execution         | `cron` + `systemctl` + pspy |
| Filesystem permissions      | `find`, `stat`, `namei`     |
| Privilege mechanisms        | `sudo`, `find`, `getcap`    |

---

## 13. The Automation-to-Manual Loop

Every interesting automated result should enter this loop:

```text id="q1m7x5"
AUTOMATED FINDING
       ↓
READ THE FINDING
       ↓
UNDERSTAND WHY IT MATTERS
       ↓
RUN MANUAL COMMAND
       ↓
CONFIRM RESULT
       ↓
TRACE TRUST RELATIONSHIP
       ↓
VALIDATE PRIVILEGE IMPACT
```

### Example

Suppose an automated tool reports:

```text id="a8c3m6"
Writable script executed by root
```

Do not immediately exploit.

Manually verify:

```bash id="y7n2q4"
ls -la /path/to/script
```

```bash id="m5x8c1"
stat /path/to/script
```

```bash id="v3q6p9"
sed -n '1,240p' /path/to/script
```

Then determine:

```text id="k9w4m2"
Who executes it?
When?
How?
Can I modify it?
Does modifying it affect privileged execution?
```

---

## 14. False Positives

Automated tools intentionally report many possibilities.

Examples:

```text id="x2m8q4"
Old package
Writable file
Interesting group
SUID binary
Unusual port
Readable configuration
```

None of these automatically proves exploitation.

### 🧠 WHY FALSE POSITIVES HAPPEN

A tool may not know:

* Application-specific logic
* Actual execution path
* Whether a service is reachable
* Whether a credential is valid
* Whether a file is trusted
* Whether a configuration is actually used
* Whether distribution patches apply

### 💡 REMEMBER

> **Automation reports evidence. Human analysis establishes meaning.**

---

## 15. Prioritizing Automated Findings

When output contains hundreds of findings, prioritize using this order.

### Priority 1 — Direct Privilege Relationships

Examples:

```text id="m7q2x9"
Sudo permissions
Writable root script
Privileged scheduled task
Privileged service using writable component
```

### Priority 2 — Powerful Access Mechanisms

Examples:

```text id="c5n8p3"
Docker
LXD
Capabilities
Powerful groups
Privileged sockets
```

### Priority 3 — Credentials

Examples:

```text id="r1x6m4"
SSH keys
Passwords
Tokens
Database credentials
```

### Priority 4 — Execution Abuse

Examples:

```text id="v8q3k5"
PATH
Wildcard
Relative path
Writable library
Sourced file
```

### Priority 5 — Software Vulnerabilities

Examples:

```text id="p2m7x9"
Known vulnerable service
Known local vulnerability
Kernel vulnerability
```

This is a prioritization workflow, not a guarantee that one category will always produce an escalation.

---

## 16. Manual Reproduction

Every important automated finding should be reproducible manually.

### 🧠 CONCEPT

If a tool reports:

```text id="x9m4q2"
Writable /opt/scripts/task.sh
```

you should be able to prove it with:

```bash id="f6c2n8"
ls -la /opt/scripts/task.sh
```

If it reports:

```text id="m3q7v1"
SUID binary
```

verify:

```bash id="w8x2c5"
ls -la /path/to/binary
```

If it reports:

```text id="n5k9p3"
Capability
```

verify:

```bash id="q4m1x7"
getcap /path/to/binary
```

### 💡 REMEMBER

> **If you cannot reproduce the finding manually, you do not fully understand it yet.**

---

## 17. Tool Output vs Evidence

Automated output is useful, but the final evidence should preferably contain the relevant native command output.

### Example

Automated tool:

```text id="j8m2c6"
Possible writable root script
```

Manual evidence:

```bash id="p5x7n3"
ls -la /opt/scripts/task.sh
```

Then:

```bash id="c4q9m1"
stat /opt/scripts/task.sh
```

Then:

```bash id="v6k2x8"
sed -n '1,240p' /opt/scripts/task.sh
```

This produces a clearer finding:

```text id="r7m3q9"
Root executes the script
+
Current user can modify it
+
Script is actually executed
```

---

## 18. Automated Enumeration Workflow

Use this sequence:

```text id="w3k8m2"
                 LOW-PRIVILEGED SHELL
                          │
                          ▼
                 BASIC MANUAL CONTEXT
                          │
                          ▼
                  RUN AUTOMATION
                          │
            ┌─────────────┼─────────────┐
            ▼             ▼             ▼
         LinPEAS         LSE           pspy
            │             │             │
            └─────────────┼─────────────┘
                          ▼
                  COLLECT FINDINGS
                          │
                          ▼
                    FILTER NOISE
                          │
                          ▼
                 PRIORITIZE FINDINGS
                          │
                          ▼
                  MANUAL REPRODUCTION
                          │
                          ▼
                    VALIDATION
                          │
                          ▼
                    EXPLOITATION
                          │
                          ▼
                    VERIFICATION
```

---

## 19. Comparing Multiple Tools

Running several tools does not mean blindly combining every output.

Instead compare them.

### 🧠 CONCEPT

```text id="k5q1m8"
LinPEAS
   +
LSE
   +
Manual enumeration
   +
pspy
   ↓
Correlated evidence
```

If two tools report the same finding:

```text id="p8m3x7"
Check it manually.
```

If one tool reports something unexpected:

```text id="c2n6q9"
Investigate the difference.
```

If manual enumeration finds something automation missed:

```text id="v4x8m1"
Understand why.
```

---

## 20. Why Tools Miss Things

No enumeration tool has perfect visibility.

A tool may miss a finding because:

* The application is custom
* The configuration is unusual
* A required command is missing
* Permissions prevent access
* The vulnerability requires contextual reasoning
* The execution happens only periodically
* The trust relationship is application-specific
* The tool does not understand the environment

### 💡 REMEMBER

> **Tool coverage is not attack-path coverage.**

---

## 21. Automated Enumeration in Time-Boxed Assessments

When time is limited:

```text id="m9c4x7"
0–5 minutes
    ↓
Manual context
    ↓
5–15 minutes
    ↓
Automated enumeration
    ↓
15+ minutes
    ↓
Validate highest-value findings
```

The exact timing can change based on the environment.

The principle remains:

```text id="r2x6k8"
Broad discovery
      ↓
Focused investigation
      ↓
Validation
```

---

## 22. Common Mistakes

### ❌ Mistake 1 — Running a Tool and Stopping

```text id="n4m8q2"
LinPEAS finished
```

does not mean:

```text id="p6x1c7"
Privilege escalation found
```

### ❌ Mistake 2 — Treating Red/Highlighted Output as Exploitable

Colors are indicators, not proof.

### ❌ Mistake 3 — Ignoring Manual Enumeration

You lose understanding and may miss contextual findings.

### ❌ Mistake 4 — Running Every Tool Available

More output is not necessarily better.

Use tools purposefully.

### ❌ Mistake 5 — Not Recording Evidence

Save important findings in a structured way.

### ❌ Mistake 6 — Trusting Version-Based Warnings Blindly

Verify exact software, version, distribution, and prerequisites.

### ❌ Mistake 7 — Exploiting Immediately

Validation comes first.

---

## 23. Safe Validation Workflow

For every automated finding:

### 🧪 VALIDATE

```text id="q7m3x8"
[ ] Read the finding
[ ] Identify the category
[ ] Understand why it matters
[ ] Reproduce manually
[ ] Identify the privileged component
[ ] Identify the user-controlled component
[ ] Confirm the trust relationship
[ ] Confirm execution/access path
[ ] Confirm privilege impact
[ ] Record evidence
```

If validation fails:

```text id="t4x8m1"
Mark as:
False positive / Not applicable / Insufficient evidence
```

Then move on.

---

## 24. Finding vs Vulnerability vs Exploitable Path

### Finding

```text id="m8q2c5"
LinPEAS reports a writable root-owned script.
```

### Vulnerability

```text id="v3n7x9"
The script is executed with root privileges
and the current user can modify it.
```

### Exploitable Path

```text id="p6k1m4"
The modified trusted component is actually executed
with root privileges, crossing the privilege boundary.
```

The tool only helped discover the candidate.

Your validation establishes the result.

---

## 25. Five-Minute Automated Enumeration Check

### 🔎 ENUMERATE

Start with manual context:

```bash id="c8m3x7"
id
```

```bash id="r4n7q2"
uname -a
```

```bash id="w6x1m9"
sudo -l
```

Then identify available tools:

```bash id="p2m8k5"
command -v linpeas
command -v lse
command -v pspy
```

If available, run the appropriate enumeration tool in the authorized environment.

Then immediately return to:

```text id="k9c4m2"
Finding
   ↓
Manual command
   ↓
Interpretation
   ↓
Validation
```

---

## 26. Quick Command Reference

| Goal                 | Command                                    |
| -------------------- | ------------------------------------------ |
| Current identity     | `id`                                       |
| Kernel               | `uname -a`                                 |
| Sudo permissions     | `sudo -l`                                  |
| Find LinPEAS         | `command -v linpeas`                       |
| Find LSE             | `command -v lse`                           |
| Find pspy            | `command -v pspy`                          |
| Processes            | `ps -eo user,pid,ppid,args`                |
| Services             | `systemctl --type=service --state=running` |
| Timers               | `systemctl list-timers --all`              |
| Cron                 | `cat /etc/crontab`                         |
| SUID                 | `find / -perm -4000 -type f 2>/dev/null`   |
| Capabilities         | `getcap -r / 2>/dev/null`                  |
| Writable files       | `find / -type f -writable 2>/dev/null`     |
| Writable directories | `find / -type d -writable 2>/dev/null`     |
| Mounted filesystems  | `findmnt`                                  |
| Network listeners    | `ss -lntup`                                |

---

## 27. Complete Checklist

### 🔎 Tool Preparation

```text id="x3m7q9"
[ ] Establish basic manual context
[ ] Identify architecture
[ ] Identify available tools
[ ] Select appropriate enumeration tool
[ ] Confirm authorized environment
```

### 🔎 Automated Enumeration

```text id="q8c2n5"
[ ] Run LinPEAS when appropriate
[ ] Run LSE when appropriate
[ ] Run pspy for dynamic observation
[ ] Record important findings
[ ] Avoid drowning in low-value output
```

### 🧠 Analysis

```text id="m5x9p3"
[ ] Categorize findings
[ ] Prioritize high-value findings
[ ] Reproduce manually
[ ] Identify trust relationships
[ ] Check privilege context
[ ] Identify false positives
```

### 🧪 Validation

```text id="v7k1c4"
[ ] Confirm finding
[ ] Confirm user control
[ ] Confirm privileged execution/access
[ ] Confirm actual attack path
[ ] Confirm privilege boundary
[ ] Record evidence
```

---

## 💡 Remember

The best way to use automated enumeration is:

```text id="n2m8q6"
AUTOMATION
     ↓
DISCOVERY
     ↓
REASONING
     ↓
MANUAL REPRODUCTION
     ↓
VALIDATION
```

Not:

```text id="w5c1x9"
AUTOMATION
     ↓
COPY EXPLOIT
```

Three rules should stay in your head:

> **Automation finds.**

> **Manual analysis explains.**

> **Validation proves.**

---

## Where Findings Lead

Automated enumeration is a bridge between discovery and validation:

```text id="p4x8m2"
12-AUTOMATED-ENUMERATION
          ↓
     Candidate Finding
          ↓
13-FINDING-VALIDATION
          ↓
     Confirmed Path
          ↓
14-EXPLOITATION
          ↓
15-ROOT-VERIFICATION
```

Individual findings may also return to earlier stages:

```text id="q6m3n8"
LinPEAS
  ↓
Sudo finding
  ↓
06-PRIVILEGE-MECHANISMS
```

```text id="r1v7c4"
LSE
  ↓
Writable script
  ↓
03-FILESYSTEM-ENUMERATION
```

```text id="x9k2m5"
pspy
  ↓
Recurring root process
  ↓
04-PROCESS-SERVICE-ENUMERATION
07-SCHEDULED-EXECUTION
```

```text id="c5n8q1"
LinPEAS
  ↓
Credential
  ↓
08-CREDENTIALS-AND-SECRETS
```

```text id="m7x3p9"
Automated finding
  ↓
PATH / wildcard / library
  ↓
09-EXECUTION-ABUSE
```

---

## Final Mental Model

```text id="v8m2q6"
              LOW-PRIVILEGED SHELL
                       │
                       ▼
                MANUAL CONTEXT
                       │
                       ▼
             AUTOMATED ENUMERATION
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       LinPEAS         LSE          pspy
          │            │            │
          └────────────┼────────────┘
                       ▼
                 FINDINGS
                       │
                       ▼
                 FILTER NOISE
                       │
                       ▼
               MANUAL REPRODUCTION
                       │
                       ▼
                  VALIDATION
                       │
                       ▼
                 EXPLOITATION
                       │
                       ▼
                  VERIFICATION
```

> **Use automation to make your enumeration wider and faster—not to make your reasoning weaker.**
