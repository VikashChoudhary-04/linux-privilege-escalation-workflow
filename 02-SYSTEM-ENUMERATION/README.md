# 02 — System Enumeration

> **Identify the operating system, kernel, architecture, users, groups, software, environment, and system configuration that define the host's privilege landscape.**

The first five minutes gave us a basic picture of the machine.

Now we go deeper.

The objective is not to collect information for its own sake. Every command should answer a question that can help us identify a potential privilege boundary.

---

## 🎯 Objectives

By the end of this section, you should be able to determine:

* Operating system and version
* Kernel and architecture
* Host configuration
* Current and other users
* Group memberships
* Login shells
* Home directories
* Installed software
* Package versions
* Environment variables
* Important system configuration
* Potentially interesting system-level information

---

## 🧠 Core Question

During system enumeration, repeatedly ask:

> **Does this system-level information reveal a configuration, privilege, software component, or trust relationship that I can investigate further?**

The workflow is:

```text id="gk9qut"
Identify OS
    ↓
Identify Kernel
    ↓
Identify Architecture
    ↓
Enumerate Users
    ↓
Enumerate Groups
    ↓
Review User Configuration
    ↓
Enumerate Installed Software
    ↓
Review Environment
    ↓
Review Important Configuration
    ↓
Identify Investigation Targets
```

---

## 1. Identify the Operating System

We already used:

```bash id="m1mqqk"
cat /etc/os-release
```

Now extract the important fields:

```bash id="4d3v5u"
grep -E '^(NAME|VERSION|ID|VERSION_ID)=' /etc/os-release
```

### Why?

The distribution and version influence:

* Default configuration
* Package management
* Service management
* File locations
* Security controls
* Vulnerability applicability

### Look For

```text id="sq1mni"
NAME=
VERSION=
ID=
VERSION_ID=
```

### Decision

```text id="6t5rpo"
Known distribution?
       │
       ├── YES → Continue distribution-specific enumeration
       │
       └── NO  → Inspect /etc/os-release manually
```

### Next Command

```bash id="x3u1uj"
uname -a
```

---

## 2. Identify the Kernel

### Command

```bash id="aqw7w7"
uname -a
```

For the kernel release alone:

```bash id="o3j2u3"
uname -r
```

### Why?

Kernel information can become relevant when investigating:

* Kernel vulnerabilities
* Architecture-specific behavior
* Security controls
* Compatibility of potential exploits

### Look For

```text id="2y8d6b"
Kernel release
Architecture
```

Example:

```text id="m9m4jp"
6.x.x-amd64
x86_64
```

### Decision

```text id="jddg6j"
Kernel appears unusual/outdated?
       │
       ├── YES → Record it for later vulnerability analysis
       │
       └── NO  → Continue enumeration
```

#### Important

Do not jump directly from:

```text id="3o5w5u"
old kernel
```

to:

```text id="2z07ov"
kernel exploit
```

The kernel version alone does not establish exploitability.

The potential vulnerability must be validated against the actual environment.

### Next Command

```bash id="qg8x8x"
uname -m
```

---

## 3. Identify Architecture

### Command

```bash id="eqk0h4"
uname -m
```

### Why?

Architecture affects:

* Available binaries
* Package architecture
* Exploit compatibility
* Compiled payloads
* System behavior

Common values include:

```text id="n8e2vu"
x86_64
aarch64
i386
```

### Decision

Record the architecture.

Then inspect CPU information if necessary.

### Next Command

```bash id="vby7qj"
lscpu
```

---

## 4. Inspect CPU and Virtualization Information

### Command

```bash id="36b7a0"
lscpu
```

### Why?

This provides additional system information and can reveal:

* CPU architecture
* Virtualization support
* CPU configuration
* Hypervisor information on some systems

### Look For

Useful fields include:

```text id="6g4q7d"
Architecture
CPU op-mode(s)
Virtualization
Hypervisor vendor
```

### Decision

Usually this is contextual information rather than an immediate escalation path.

Record anything unusual and continue.

### Next Command

```bash id="bquc4n"
cat /proc/version
```

---

## 5. Inspect Kernel Build Information

### Command

```bash id="ydv4hw"
cat /proc/version
```

### Why?

`/proc/version` provides kernel build information that can supplement `uname`.

It may reveal:

* Kernel version
* Compiler information
* Build details

### Decision

Use this to confirm kernel information when required.

Do not treat compiler/build information alone as an exploit indicator.

### Next Command

```bash id="2wqaz9"
hostnamectl
```

---

## 6. Inspect Host Configuration

### Command

```bash id="fxxj8u"
hostnamectl
```

### Why?

On systems using systemd, `hostnamectl` can provide:

* Hostname
* Operating system
* Kernel
* Architecture

This can consolidate information already collected.

#### Important

`hostnamectl` may not exist or may not work in:

* Minimal environments
* Containers
* Restricted shells
* Non-systemd systems

If it fails, continue with information already collected.

### Next Command

```bash id="q0gh9a"
cat /etc/hostname
```

---

## 7. Enumerate Local Users

### Command

```bash id="0f4hkn"
cat /etc/passwd
```

### Why?

`/etc/passwd` provides information about local accounts.

It can reveal:

* Usernames
* UIDs
* GIDs
* Home directories
* Login shells

### Example

```text id="qg9ivk"
user:x:1000:1000:User:/home/user:/bin/bash
```

The structure is:

```text id="7m6o1s"
username
   :
password field
   :
UID
   :
GID
   :
comment
   :
home directory
   :
login shell
```

### Look For

Pay particular attention to:

* UID `0` accounts
* Human users
* Service accounts
* Unusual shells
* Unusual home directories

### First Filter

Find accounts with UID `0`:

```bash id="c84b3g"
awk -F: '$3 == 0 {print}' /etc/passwd
```

### Why?

UID `0` represents root-level identity.

Multiple UID `0` accounts can therefore warrant investigation.

#### Decision

```text id="6iv9cx"
Additional UID 0 account?
       │
       ├── YES → Investigate account configuration
       │
       └── NO  → Continue
```

Do not assume an account is exploitable merely because it exists.

### Next Command

```bash id="9v4vst"
cut -d: -f1,3,6,7 /etc/passwd
```

---

## 8. Review Usernames, UIDs, Homes and Shells

### Command

```bash id="z5p4k3"
cut -d: -f1,3,6,7 /etc/passwd
```

### Why?

This produces a more focused view:

```text
username
UID
home directory
shell
```

### Look For

Questions to ask:

```text id="e2m5js"
Which users have interactive shells?
Which users have home directories?
Are there unusual accounts?
Are there service accounts with unexpected shells?
Are there multiple administrative accounts?
```

### Filter for interactive shells

A useful starting point:

```bash id="v1n9q4"
grep -E '/(bash|sh|zsh|fish)$' /etc/passwd
```

### Decision

```text id="m8d84y"
Interesting user?
       │
       ├── YES → Investigate home directory, groups and configuration
       │
       └── NO  → Continue
```

### Next Command

```bash id="c2w4b5"
cat /etc/group
```

---

## 9. Enumerate Groups

### Command

```bash id="y7k6nt"
cat /etc/group
```

### Why?

Groups define many access relationships on a Linux system.

This helps identify:

* Administrative groups
* Application groups
* Device access
* Service-related groups
* Container-related groups

### Focused Search

Look for commonly interesting groups:

```bash id="knr1g7"
grep -Ei '^(sudo|wheel|docker|lxd|disk|adm|shadow|video|audio):' /etc/group
```

The exact groups present depend on the distribution.

### Decision

```text id="qfmx0t"
Interesting group?
       │
       ├── YES → Determine what resources the group controls
       │
       └── NO  → Continue
```

Remember:

> **Group name → access → security impact → validation**

Do not stop at the group name.

### Next Command

```bash id="m1b3pr"
getent group
```

---

## 10. Enumerate Groups Through NSS

### Command

```bash id="p8o4cp"
getent group
```

### Why?

`/etc/group` shows local group configuration.

`getent` queries the system's configured Name Service Switch sources.

Depending on the environment, this may include information from sources beyond local files.

### Why does this matter?

It can provide a more complete picture of group membership and identity configuration.

### Next

Compare relevant results with:

```bash id="r5ihpe"
id
```

The current user's actual supplementary groups are especially important.

---

## 11. Inspect Current User Configuration

First identify the current user's home directory:

```bash id="jql1kt"
echo $HOME
```

Then inspect it:

```bash id="k0d7xa"
ls -la "$HOME"
```

### Why?

Home directories can contain:

* Shell history
* Configuration files
* SSH material
* Application configuration
* Scripts
* Credentials
* Backups

### Look For

Files such as:

```text id="3v7u9j"
.bash_history
.profile
.bashrc
.ssh/
.config/
```

Do not assume that every hidden file contains useful information.

### Next Command

```bash id="4j5h5x"
ls -la "$HOME/.ssh" 2>/dev/null
```

---

## 12. Inspect SSH Configuration for the Current User

### Command

```bash id="6afjzq"
ls -la "$HOME/.ssh" 2>/dev/null
```

### Why?

SSH configuration may contain:

* Private keys
* Public keys
* Known hosts
* Client configuration

### Look For

Potentially relevant files:

```text id="wx0fpf"
id_rsa
id_ed25519
config
authorized_keys
known_hosts
```

### Important

The existence of a key does not automatically mean it can be used.

You must determine:

```text id="z3o3cs"
Who owns the key?
 ↓
Are permissions appropriate?
 ↓
Is the key usable?
 ↓
Which account does it authenticate to?
 ↓
What privileges does that account have?
```

Credential investigation will be covered more thoroughly later.

---

## 13. Enumerate Installed Software

The exact command depends on the distribution.

For Debian-based systems:

```bash id="m0x5es"
dpkg -l
```

For a concise package list:

```bash id="u8g7gl"
dpkg-query -W -f='${Package} ${Version}\n'
```

For RPM-based systems:

```bash id="4e0w0b"
rpm -qa
```

### Why?

Installed software can reveal:

* Applications
* Services
* Development tools
* Security software
* Custom packages
* Potentially vulnerable components

### Decision

```text id="5cl4b6"
Unusual software?
       │
       ├── YES → Identify version and configuration
       │
       └── NO  → Continue
```

### Important

Do not treat:

```text id="b6f2zq"
software found
```

as:

```text id="gibmri"
vulnerability confirmed
```

Version and configuration must be evaluated.

---

## 14. Identify Package Managers

Determine which package manager is available:

```bash id="oj5u1k"
command -v apt
command -v apt-get
command -v dnf
command -v yum
command -v pacman
command -v apk
```

### Why?

This helps determine:

* Distribution family
* Installed package ecosystem
* Appropriate package queries
* Software-version investigation methods

### Next

Use the appropriate package manager for the system.

---

## 15. Search for Specific Installed Software

Once an interesting application is identified, determine its version.

Examples:

```bash id="n1z0id"
command -v <program>
```

Then:

```bash id="m9z4mm"
<program> --version
```

or:

```bash id="9gjb0x"
<program> -V
```

### Why?

The binary's version can be compared against known vulnerabilities later.

### Important

A version match is only a starting point.

Always validate:

```text id="7h6m9u"
Version
   ↓
Affected version range?
   ↓
Correct configuration?
   ↓
Exploit prerequisites?
   ↓
Applicable to this environment?
```

---

## 16. Inspect Environment Variables

### Command

```bash id="x8v8gd"
env
```

Also inspect:

```bash id="1j4b4q"
printenv
```

### Why?

Environment variables can reveal:

* Execution paths
* User context
* Application configuration
* Runtime settings
* Potential secrets

### Focused Review

```bash id="v7f2hi"
env | sort
```

### Look For

Pay attention to:

```text id="6z3xqg"
PATH
HOME
SHELL
USER
PWD
LOGNAME
```

and application-specific variables.

### Decision

```text id="v2p8aa"
Interesting variable?
       │
       ├── YES → Identify what consumes it
       │
       └── NO  → Continue
```

---

## 17. Inspect PATH Directories

First:

```bash id="6x6dwm"
echo "$PATH"
```

Then split it into individual entries:

```bash id="q9f9q3"
echo "$PATH" | tr ':' '\n'
```

### Why?

Individual PATH directories are easier to inspect.

## Check permissions

For each interesting directory:

```bash id="v3d1gc"
ls -ld /path/to/directory
```

### Look For

Ask:

```text id="h9a3tx"
Is the directory writable?
Is it unusual?
Is it user-controlled?
Does a privileged process use this PATH?
```

### Important

A writable PATH directory is not automatically exploitable.

It becomes interesting when a privileged execution context relies on command lookup through that path.

This will be investigated in:

```text id="q1tq4o"
09-EXECUTION-ABUSE/
```

---

## 18. Inspect System Information Files

Several files can provide useful context.

### Kernel information

```bash id="w5v3ht"
cat /proc/version
```

### CPU information

```bash id="r2x1c8"
cat /proc/cpuinfo
```

### Memory information

```bash id="m2m5h1"
cat /proc/meminfo
```

### Mounted filesystems

```bash id="y9l8tc"
cat /proc/mounts
```

### Why?

These files can reveal:

* System configuration
* Mounted filesystems
* Hardware information
* Runtime information
* Potentially interesting mounts

Not every result is directly relevant to escalation.

Use the information to form investigation hypotheses.

---

## 19. Review Mounted Filesystems

### Command

```bash id="j4q7zq"
findmnt
```

Alternative:

```bash id="i8l0pj"
mount
```

### Why?

Mounted filesystems can reveal:

* NFS
* Bind mounts
* Shared directories
* Container filesystems
* Unusual storage
* Writable locations

### Look For

Pay attention to:

```text id="j9y2ba"
Filesystem
Mount point
Options
```

### Decision

```text id="v4f5e8"
Interesting mount?
       │
       ├── YES → Investigate ownership, permissions and mount options
       │
       └── NO  → Continue
```

NFS and filesystem-specific privilege issues will be covered later.

---

## 20. Inspect System-Wide Configuration

Start with:

```bash id="q6y9kw"
ls -la /etc
```

### Why?

`/etc` contains a large amount of system configuration.

Do not recursively read everything.

Instead, investigate configuration relevant to something you have already discovered.

For example:

```text id="9e2y7g"
Interesting service
      ↓
Identify service configuration
      ↓
Inspect relevant files
```

This is more efficient than indiscriminate searching.

---

## 21. Identify the Shell

### Command

```bash id="4k5t0h"
echo "$SHELL"
```

Also inspect the current process:

```bash id="h4e9m7"
ps -p $$ -o pid,ppid,user,group,args
```

### Why?

Understanding the shell can help explain:

* Command behavior
* Environment inheritance
* Shell configuration
* Execution context

### Look For

```text id="g1h9l8"
Shell path
Process owner
Parent process
```

This becomes especially useful when investigating unusual shells or restricted environments.

---

## 22. Check Resource Limits

### Command

```bash id="3y2zj4"
ulimit -a
```

### Why?

Resource limits define restrictions placed on the current shell/process environment.

They are usually contextual information rather than immediate escalation findings, but unusual limits can help explain process behavior.

Record anything relevant and continue.

---

## 🧠 System Enumeration Decision Tree

The overall workflow is:

```text id="p4t1si"
START
  │
  ▼
OS / Version
  │
  ▼
Kernel
  │
  ├── Interesting version?
  │       └──► Record for later validation
  │
  ▼
Architecture
  │
  ▼
Users
  │
  ├── UID 0 account?
  │       └──► Investigate
  │
  ▼
Groups
  │
  ├── Interesting group?
  │       └──► Investigate access
  │
  ▼
User Configuration
  │
  ├── Interesting SSH/config/history?
  │       └──► Investigate
  │
  ▼
Installed Software
  │
  ├── Interesting application?
  │       └──► Identify version/configuration
  │
  ▼
Environment
  │
  ├── Interesting variable/PATH?
  │       └──► Identify privileged consumer
  │
  ▼
Mounts
  │
  ├── Interesting filesystem?
  │       └──► Investigate permissions/options
  │
  ▼
System Configuration
  │
  ▼
DEEPER ENUMERATION
```

---

## ⚠️ Common Mistakes

### 1. Collecting information without prioritizing it

Not every system detail is an escalation path.

Ask:

> **What does this information allow me to investigate?**

---

### 2. Assuming an old version is automatically vulnerable

A version is a clue.

It is not proof.

Always validate:

```text id="5x9j3n"
Version
 ↓
Affected?
 ↓
Configuration affected?
 ↓
Prerequisites present?
 ↓
Actually exploitable?
```

---

### 3. Ignoring UID 0 accounts

Always check:

```bash id="x8zzq6"
awk -F: '$3 == 0 {print}' /etc/passwd
```

Multiple UID 0 accounts deserve investigation.

---

### 4. Searching every file blindly

Do not immediately perform enormous recursive searches.

Instead:

```text id="b7ef2u"
Finding
  ↓
Hypothesis
  ↓
Targeted search
```

Targeted enumeration is faster and produces less noise.

---

### 5. Treating credentials as automatically useful

A credential is only useful if:

```text id="h4aj3w"
Valid
 ↓
Accessible
 ↓
Belongs to useful identity
 ↓
Provides relevant privileges
```

---

## ⚡ System Enumeration — Command Reference

```bash id="b3oq6w"
# OS
cat /etc/os-release
grep -E '^(NAME|VERSION|ID|VERSION_ID)=' /etc/os-release

# Kernel
uname -a
uname -r
cat /proc/version

# Architecture / CPU
uname -m
lscpu

# Host
hostname
hostnamectl
cat /etc/hostname

# Users
cat /etc/passwd
awk -F: '$3 == 0 {print}' /etc/passwd
cut -d: -f1,3,6,7 /etc/passwd
grep -E '/(bash|sh|zsh|fish)$' /etc/passwd

# Groups
cat /etc/group
getent group
groups
id

# Home directory
echo "$HOME"
ls -la "$HOME"

# SSH
ls -la "$HOME/.ssh" 2>/dev/null

# Packages
dpkg -l
rpm -qa

# Package managers
command -v apt
command -v apt-get
command -v dnf
command -v yum
command -v pacman
command -v apk

# Environment
env
printenv
env | sort

# PATH
echo "$PATH"
echo "$PATH" | tr ':' '\n'

# Mounts
findmnt
mount
cat /proc/mounts

# Shell
echo "$SHELL"
ps -p $$ -o pid,ppid,user,group,args

# Resource limits
ulimit -a
```

---

## 🧠 Remember

System enumeration is not about memorizing commands.

It is about building a model:

```text id="q6f8jy"
WHAT system is this?
       ↓
WHO exists?
       ↓
WHAT groups exist?
       ↓
WHAT software is installed?
       ↓
WHAT configuration exists?
       ↓
WHAT is unusual?
       ↓
WHAT deserves investigation?
```

The objective is to turn raw system information into **investigation targets**.

---

## 🚀 Next Stage

Once system-level information has been collected, move to:

```text id="1s4h4w"
03-FILESYSTEM-ENUMERATION/
```

The next phase asks a different question:

> **What files and directories can I read, modify, execute, or otherwise influence—and are any of them involved in privileged operations?**
 
