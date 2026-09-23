# 11 — Vulnerable Software

Privilege escalation does not always come from a Linux misconfiguration.

Sometimes the escalation path exists because a privileged application, binary, service, or kernel component contains a known security vulnerability.

The challenge is not simply finding an old version.

The real workflow is:

```text
Identify software
      ↓
Identify exact version
      ↓
Identify affected component
      ↓
Research vulnerability
      ↓
Check applicability
      ↓
Validate safely
      ↓
Determine privilege impact
      ↓
Exploit only when authorized
```

---

## Core Question

> **Is privileged software running on this system affected by a vulnerability that can actually cross the current privilege boundary?**

The key word is **actually**.

An old package is not automatically exploitable.

A CVE affecting a product is not automatically applicable to the installed version.

An applicable CVE is not automatically a privilege-escalation vulnerability.

---

## Mental Model

Use this chain:

```text id="8x4m2q"
SOFTWARE FOUND
      ↓
EXACT VERSION
      ↓
AFFECTED VERSION?
      ↓
VULNERABILITY TYPE
      ↓
LOCAL OR REMOTE?
      ↓
PRIVILEGE REQUIRED
      ↓
PRIVILEGE GAINED
      ↓
ENVIRONMENT MATCH?
      ↓
SAFE VALIDATION
      ↓
AUTHORIZED EXPLOITATION
```

Always ask:

```text id="3p7k9c"
What software?
What exact version?
What component?
What vulnerability?
What prerequisites?
What privilege does it require?
What privilege does it provide?
Does it apply to this system?
```

---

## 1. Identify Installed Software

Start by determining what package-management ecosystem the system uses.

### 🔎 ENUMERATE

Check common package managers:

```bash id="7n4m2x"
command -v apt
command -v dpkg
command -v dnf
command -v yum
command -v rpm
command -v pacman
command -v apk
```

On Debian-based systems:

```bash id="m8q1v5"
dpkg -l
```

List package names and versions:

```bash id="c3x7k9"
dpkg-query -W -f='${Package} ${Version}\n'
```

On RPM-based systems:

```bash id="p6m2w8"
rpm -qa
```

### 🧠 WHY

You need the **exact installed version**, not just the application name.

For example:

```text id="x9c4n7"
Package: example-service
Version: 1.2.3-4
```

The package version determines whether a particular vulnerability may apply.

---

## 2. Identify Security-Relevant Software

Do not investigate every installed package equally.

Prioritize software that:

* Runs as root
* Provides a network service
* Handles authentication
* Processes untrusted input
* Manages containers
* Manages filesystems
* Provides system administration functionality
* Executes commands
* Interacts with privileged resources

### 🔎 ENUMERATE

List running services:

```bash id="4v8m1q"
systemctl --type=service --state=running
```

List processes:

```bash id="k2x6p9"
ps -eo user,pid,ppid,args
```

Identify root processes:

```bash id="q5m7c3"
ps -eo user,pid,ppid,args | awk '$1 == "root"'
```

### 🧠 WHY

A vulnerable application running as an unprivileged user may have limited privilege-escalation impact.

A vulnerable application running as root deserves much closer investigation.

---

## 3. Determine Exact Versions

### 🔎 ENUMERATE

For a known Debian package:

```bash id="t3q8m5"
dpkg-query -W -f='${Package} ${Version}\n' <package>
```

For RPM:

```bash id="w6n2c7"
rpm -q <package>
```

For common command-line programs:

```bash id="z4m8p1"
<command> --version
```

or:

```bash id="n7x3k9"
<command> -V
```

Check the binary:

```bash id="c1q6v8"
command -v <command>
```

```bash id="r5m2x7"
readlink -f "$(command -v <command>)"
```

### ⚠️ LOOK FOR

Record:

```text id="4k8q2m"
Product
Version
Build/revision
Package version
Architecture
Binary path
```

These details matter when comparing against vulnerability information.

---

## 4. Kernel Version

The Linux kernel is itself privileged software.

### 🔎 ENUMERATE

```bash id="p8m4x2"
uname -a
```

Kernel release:

```bash id="v3q7n5"
uname -r
```

Architecture:

```bash id="m6k1c8"
uname -m
```

Distribution information:

```bash id="x2n9w4"
cat /etc/os-release
```

Kernel build information:

```bash id="q7c3m1"
cat /proc/version
```

### 🧠 WHY

Kernel vulnerabilities can have significant privilege impact.

However:

> **Kernel version matching is only the beginning of kernel vulnerability analysis.**

Distribution backports can make simple version comparison misleading.

---

## 5. Distribution Backports

A common mistake is assuming:

```text id="8c2m5x"
Old-looking kernel version
=
Known vulnerable kernel
```

Linux distributions frequently backport security fixes into maintained package versions.

### 🧠 CONCEPT

You may encounter:

```text id="1x7q4n"
Upstream version
        ↓
Distribution package
        ↓
Security patches backported
```

Therefore, always consider:

```text id="y6m3p8"
Distribution
Package version
Kernel build
Security update status
```

### 🔎 ENUMERATE

On Debian-based systems:

```bash id="n4w8c2"
apt-cache policy linux-image-$(uname -r) 2>/dev/null
```

Check installed kernel packages:

```bash id="q2m7x5"
dpkg -l | grep -E '^ii[[:space:]]+linux-image'
```

On RPM-based systems:

```bash id="k8p1v6"
rpm -q kernel
```

---

## 6. Identify Vulnerability Type

Not every vulnerability is useful for privilege escalation.

Common categories include:

```text id="r9x4m2"
Remote Code Execution
Local Privilege Escalation
Arbitrary File Read
Arbitrary File Write
Authentication Bypass
Command Injection
Memory Corruption
Path Traversal
Improper Access Control
Container Escape
Kernel Privilege Escalation
```

### 🧠 WHY

For local privilege escalation, prioritize vulnerabilities where:

```text id="w5k2n7"
Current user
     ↓
Local vulnerability
     ↓
More privileged execution
```

A remote-only vulnerability may not be relevant after obtaining a local shell.

---

## 7. Local vs Remote Vulnerabilities

### 🧠 CONCEPT

A vulnerability may require:

```text id="x6m1p9"
Remote network access
```

or:

```text id="b8q3c5"
Local user access
```

or:

```text id="z4n7w2"
Specific application privileges
```

### ⚠️ LOOK FOR

For privilege escalation, determine:

```text id="j2m6x8"
Attack vector
Privileges required
User interaction
Scope
Privileges gained
```

### ➡️ NEXT STEP

If the vulnerability requires privileges you do not have:

```text id="v8c3m1"
Not directly applicable
```

Do not force an exploit simply because the CVE exists.

---

## 8. Version Matching

This is one of the most important steps.

### 🔎 ENUMERATE

Record:

```text id="q6m9x4"
Installed:
Product = ?
Version = ?
Architecture = ?
Distribution = ?
```

Then compare against the vulnerability's affected versions.

### 🧠 WHY

You need to establish:

```text id="s3n7p2"
Installed version
        ↓
Falls within affected range?
        ↓
YES
        ↓
Other prerequisites satisfied?
```

### ❌ COMMON MISTAKE

Do not use:

```text id="t1x8m5"
"Version looks old"
```

as proof.

Use exact version information.

---

## 9. Architecture Matching

Some vulnerabilities or proof-of-concept code depend on architecture.

### 🔎 ENUMERATE

```bash id="f7m2c8"
uname -m
```

```bash id="k4x9n1"
lscpu
```

For a binary:

```bash id="p3w6q8"
file /path/to/binary
```

### 🧠 WHY

You may encounter:

```text id="n5q2v7"
x86
x86_64
ARM
AArch64
```

A proof of concept built for one architecture may not work on another.

Architecture mismatch can be a reason to reject an otherwise interesting lead.

---

## 10. Identify the Privileged Component

A vulnerability matters for privilege escalation only if the vulnerable component has an appropriate privilege relationship.

### 🔎 ENUMERATE

Identify process owner:

```bash id="m8c4x2"
ps -eo user,pid,ppid,args
```

For a specific PID:

```bash id="q7n3w5"
ps -p <PID> -o user,pid,ppid,args
```

Identify executable:

```bash id="v1m6k9"
readlink -f /proc/<PID>/exe
```

Check executable:

```bash id="x5p2c8"
ls -la /path/to/binary
```

### 🧠 WHY

Build the chain:

```text id="a7q3m5"
Vulnerable software
        ↓
Privileged process
        ↓
Local attack surface
        ↓
Privilege boundary
```

---

## 11. Search for Vulnerability Information

Once exact software and version information is known, research the relevant vulnerability.

### 🔎 RESEARCH

Search for:

```text id="z9m4x2"
<Product> <version> vulnerability
<Product> <version> local privilege escalation
<CVE-ID> affected versions
<CVE-ID> advisory
```

Prefer authoritative sources such as:

* Vendor security advisories
* Distribution security trackers
* NVD
* CVE records
* Official project advisories
* Security research with reproducible technical details

### 🧠 WHY

You need more than a CVE number.

Determine:

```text id="c6n8p1"
Affected versions
Attack vector
Required privileges
Affected component
Fixed versions
Known prerequisites
```

---

## 12. CVE Identification

A CVE identifier provides a standardized reference to a vulnerability.

Example format:

```text id="w2m7x5"
CVE-YYYY-NNNNN
```

### 🧠 CONCEPT

A CVE record may tell you:

* Vulnerability description
* Affected products
* Severity information
* References
* Publication/update information

But a CVE record alone may not establish that the vulnerability is exploitable on your specific machine.

### ⚠️ LOOK FOR

Always connect:

```text id="d5q8m2"
CVE
 ↓
Affected product
 ↓
Affected version
 ↓
Installed version
 ↓
System configuration
 ↓
Privilege impact
```

---

## 13. CVE Validation Matrix

Use a small matrix instead of guessing.

| Question                        | Result         |
| ------------------------------- | -------------- |
| Correct product?                | Yes / No       |
| Correct component?              | Yes / No       |
| Installed version affected?     | Yes / No       |
| Correct architecture?           | Yes / No / N/A |
| Local attack possible?          | Yes / No       |
| Required privileges available?  | Yes / No       |
| Required configuration present? | Yes / No       |
| Vulnerable process privileged?  | Yes / No       |
| Privilege escalation impact?    | Yes / No       |
| Safe validation possible?       | Yes / No       |

If several critical fields are unknown, the finding is not yet validated.

---

## 14. Installed Package vs Running Binary

Do not assume the package version and running binary are always identical in practice.

### 🔎 ENUMERATE

Find the executable:

```bash id="h3m8q1"
command -v <command>
```

Resolve it:

```bash id="r7c2x9"
readlink -f "$(command -v <command>)"
```

Inspect the process:

```bash id="m5n1v6"
ps -eo pid,user,args | grep '[p]rocess-name'
```

Inspect the executable:

```bash id="x8q4k2"
file /path/to/binary
```

### 🧠 WHY

You want to connect:

```text id="p9c6m4"
Installed package
      ↓
Actual executable
      ↓
Running process
      ↓
Privilege level
```

This prevents investigating a vulnerable package that is not actually used by the privileged process.

---

## 15. Vulnerable Services

Privileged services deserve special attention.

### 🔎 ENUMERATE

List running services:

```bash id="q4m7x2"
systemctl --type=service --state=running
```

Inspect a service:

```bash id="n8c3p5"
systemctl status <service>
```

Inspect configuration:

```bash id="v6x1m9"
systemctl cat <service>
```

Check executable:

```bash id="k2w8q4"
systemctl show -p ExecStart <service>
```

### 🧠 WHY

Determine:

```text id="b5m3x7"
Service
 ↓
Binary
 ↓
Version
 ↓
User
 ↓
Configuration
 ↓
Known vulnerability
```

---

## 16. Vulnerable Binaries

A local binary may contain a vulnerability even when no network service is exposed.

### 🔎 ENUMERATE

Find custom executables:

```bash id="m7q2c5"
find /opt /usr/local/bin /usr/local/sbin -type f -executable 2>/dev/null
```

Inspect candidates:

```bash id="x4n8p1"
file /path/to/binary
```

```bash id="j6m3v9"
ls -la /path/to/binary
```

Check libraries:

```bash id="r8c5k2"
ldd /path/to/binary
```

### 🧠 WHY

Prioritize binaries that:

* Run as root
* Have SUID
* Have capabilities
* Process untrusted input
* Load external files
* Execute commands

---

## 17. Kernel Vulnerability Analysis

Kernel exploitation should be treated as a separate category.

### 🔎 ENUMERATE

```bash id="c9m2x7"
uname -a
```

```bash id="p4w8n1"
uname -r
```

```bash id="v6q3m5"
cat /etc/os-release
```

Check architecture:

```bash id="k1x7c4"
uname -m
```

### 🧠 WHY

Kernel vulnerabilities can cross the user-to-root boundary directly.

However, kernel exploitation is usually a later option because:

* It can be unstable
* It can crash the system
* It may depend on exact kernel configuration
* Distribution patches can change applicability
* The exploit may have destructive side effects

### ⚠️ LOOK FOR

A credible kernel lead should have:

```text id="n3m8q5"
Exact kernel family
+
Affected version range
+
Correct distribution context
+
Correct architecture
+
Required configuration
+
Local attack vector
+
Privilege escalation impact
```

---

## 18. Exploit Selection

Finding a public proof of concept does not mean it should be the first thing you run.

### 🧠 CONCEPT

Select an exploit only after answering:

```text id="w4k9m2"
Is the software affected?
        ↓
Does the vulnerability apply locally?
        ↓
Are prerequisites satisfied?
        ↓
Does it provide the required privilege?
        ↓
Is it safe to test?
        ↓
Is the environment authorized?
```

### ⚠️ LOOK FOR

Prefer an approach that is:

* Applicable
* Understandable
* Reproducible
* Stable
* Minimal in impact
* Appropriate for the authorized lab

Do not choose an exploit merely because it is the newest or easiest-looking result.

---

## 19. Proof of Concept vs Exploit

A proof of concept demonstrates that a vulnerability can be triggered.

An exploit attempts to turn that vulnerability into a meaningful security impact.

### 🧠 CONCEPT

For example:

```text id="a4m7x1"
PoC:
Demonstrates vulnerable behavior
```

versus:

```text id="c8q2n5"
Exploit:
Uses the vulnerability to cross the privilege boundary
```

This distinction matters when documenting findings.

---

## 20. Safe Validation Strategy

Before running an exploit, validate the prerequisites.

### 🧪 VALIDATE

Check:

```text id="y5m8q2"
[ ] Product matches
[ ] Version matches
[ ] Component matches
[ ] Architecture matches
[ ] Required privileges available
[ ] Required configuration exists
[ ] Vulnerable process is running
[ ] Vulnerability is local
[ ] Privilege impact is relevant
[ ] Test environment is authorized
```

If one of the critical prerequisites fails:

```text id="s3n7k1"
Stop.
```

Move to another enumeration path.

---

## 21. Finding vs Vulnerability vs Exploitable Path

### Finding

```text id="q8m2x5"
The system is running software version X.
```

This is an observation.

### Vulnerability

```text id="v4c7n9"
Version X falls within a documented affected range
for a known local vulnerability.
```

Now there is a documented vulnerability.

### Exploitable Path

```text id="m1x6p8"
The vulnerability applies to this exact environment,
its prerequisites are satisfied, and it provides a path
to greater privilege.
```

That is the point where exploitation becomes relevant.

---

## 22. Common Mistakes

### ❌ Mistake 1 — "Old Version = Vulnerable"

Version age alone is not proof.

### ❌ Mistake 2 — Ignoring Distribution Patches

A distribution may backport security fixes.

### ❌ Mistake 3 — Searching CVEs Before Identifying Software

Start with the machine.

```text id="z5q8m2"
Software
 ↓
Version
 ↓
Component
 ↓
CVE
```

Not:

```text id="r7c3n6"
Random CVE
 ↓
Try it
```

### ❌ Mistake 4 — Ignoring Privilege Context

A vulnerability in an unprivileged application may not provide privilege escalation.

### ❌ Mistake 5 — Using the Wrong Architecture

A proof of concept may depend on architecture.

### ❌ Mistake 6 — Running Public Exploits Immediately

A public exploit may:

* Crash the system
* Modify files
* Leave artifacts
* Depend on assumptions
* Be unreliable
* Be inappropriate for the environment

### ❌ Mistake 7 — Ignoring Existing Easier Paths

Always compare the software finding with:

* Sudo
* SUID
* Capabilities
* Scheduled tasks
* Credentials
* Writable files
* Services
* Special environments

The objective is a validated privilege path, not simply exploitation of a CVE.

---

## 23. Five-Minute Vulnerable Software Check

### 🔎 ENUMERATE

```bash id="q4x8m2"
uname -a
```

```bash id="n6c3p7"
cat /etc/os-release
```

```bash id="v1m9k5"
ps -eo user,pid,ppid,args
```

```bash id="w7q2x4"
systemctl --type=service --state=running
```

Identify package managers:

```bash id="m8c5n1"
command -v dpkg rpm
```

Debian package versions:

```bash id="x3p7k9"
dpkg-query -W -f='${Package} ${Version}\n' 2>/dev/null
```

RPM package versions:

```bash id="j5n2v8"
rpm -qa 2>/dev/null
```

Then prioritize:

```text id="c6m1x4"
Privileged process
       ↓
Software name
       ↓
Exact version
       ↓
Known vulnerability?
       ↓
Applicable locally?
       ↓
Privilege impact?
```

---

## 24. Quick Command Reference

| Goal                   | Command                                                                    |
| ---------------------- | -------------------------------------------------------------------------- |
| Kernel                 | `uname -a`                                                                 |
| Kernel release         | `uname -r`                                                                 |
| Architecture           | `uname -m`                                                                 |
| OS information         | `cat /etc/os-release`                                                      |
| Kernel build           | `cat /proc/version`                                                        |
| Debian packages        | `dpkg -l`                                                                  |
| Debian package version | `dpkg-query -W -f='${Package} ${Version}\n'`                               |
| RPM packages           | `rpm -qa`                                                                  |
| Package lookup         | `dpkg-query -W <package>`                                                  |
| Running services       | `systemctl --type=service --state=running`                                 |
| Service status         | `systemctl status <service>`                                               |
| Service configuration  | `systemctl cat <service>`                                                  |
| Service executable     | `systemctl show -p ExecStart <service>`                                    |
| Processes              | `ps -eo user,pid,ppid,args`                                                |
| Root processes         | `ps -eo user,pid,ppid,args \| awk '$1 == "root"'`                          |
| Process executable     | `readlink -f /proc/<PID>/exe`                                              |
| Binary information     | `file /path/to/binary`                                                     |
| Binary dependencies    | `ldd /path/to/binary`                                                      |
| ELF information        | `readelf -d /path/to/binary`                                               |
| Custom executables     | `find /opt /usr/local/bin /usr/local/sbin -type f -executable 2>/dev/null` |

---

## 25. Complete Checklist

### 🔎 Software Enumeration

```text id="m7x3q8"
[ ] Identify OS
[ ] Identify distribution
[ ] Identify kernel
[ ] Identify architecture
[ ] Identify package manager
[ ] Enumerate installed software
[ ] Identify running services
[ ] Identify privileged processes
```

### 🧠 Vulnerability Analysis

```text id="p4c8n2"
[ ] Identify exact software
[ ] Identify exact version
[ ] Identify affected component
[ ] Identify vulnerability
[ ] Check affected version range
[ ] Check distribution patch status
[ ] Check architecture
[ ] Check local attack requirements
[ ] Check required privileges
[ ] Check resulting privilege
```

### 🧪 Validation

```text id="x2m6v9"
[ ] Confirm software is actually used
[ ] Confirm vulnerable component is running
[ ] Confirm prerequisites
[ ] Confirm vulnerability applies
[ ] Confirm privilege impact
[ ] Confirm authorized test environment
[ ] Validate safely
[ ] Record evidence
```

---

## 💡 Remember

Do not memorize CVE numbers.

Memorize the workflow:

```text id="k5q9m3"
IDENTIFY
   ↓
VERSION
   ↓
RESEARCH
   ↓
MATCH
   ↓
VALIDATE
   ↓
EXPLOIT
   ↓
VERIFY
```

And remember:

> **A CVE is a lead. Applicability is a conclusion.**

The strongest habit is to prove every link:

```text id="r8m2x6"
Software
   ↓
Exact version
   ↓
Affected vulnerability
   ↓
Applicable configuration
   ↓
Privileged component
   ↓
Privilege impact
```

---

## Where Findings Lead

Vulnerable-software findings can connect to:

```text id="c4m8x2"
Software
   ↓
CVE
   ↓
Applicability
   ↓
13-FINDING-VALIDATION
   ↓
14-EXPLOITATION
```

Kernel findings:

```text id="n7x3p5"
Kernel
   ↓
Version / Distribution
   ↓
CVE
   ↓
Kernel Validation
   ↓
Kernel Exploitation
```

Service findings:

```text id="w5q1m9"
Privileged Service
   ↓
Vulnerable Software
   ↓
Service Attack Surface
   ↓
Finding Validation
```

Related stages:

* `02-SYSTEM-ENUMERATION/` — OS, kernel, architecture, installed software
* `04-PROCESS-SERVICE-ENUMERATION/` — privileged processes and services
* `06-PRIVILEGE-MECHANISMS/` — privileged execution mechanisms
* `10-SPECIAL-ENVIRONMENTS/` — container/runtime software
* `12-AUTOMATED-ENUMERATION/` — automated vulnerability discovery
* `13-FINDING-VALIDATION/` — applicability and exploitability
* `14-EXPLOITATION/` — authorized exploitation
* `15-ROOT-VERIFICATION/` — verify resulting privilege

---

## Final Mental Model

```text id="q6m3x8"
             SOFTWARE FOUND
                    │
                    ▼
             EXACT VERSION
                    │
                    ▼
            VULNERABILITY FOUND
                    │
                    ▼
          DOES VERSION MATCH?
                │       │
               NO      YES
                │       │
                ▼       ▼
             MOVE ON  CHECK PREREQUISITES
                           │
                           ▼
                   LOCAL ATTACK POSSIBLE?
                      │          │
                     NO         YES
                      │          │
                      ▼          ▼
                   MOVE ON   CHECK PRIVILEGE
                                  │
                                  ▼
                         PRIVILEGE BOUNDARY?
                            │          │
                           NO         YES
                            │          │
                            ▼          ▼
                         MOVE ON   VALIDATE
                                      │
                                      ▼
                                   EXPLOIT
                                      │
                                      ▼
                                   VERIFY
```

> **Don't hunt for exploits first. Identify the exact software, prove applicability, then decide whether exploitation is justified.**
