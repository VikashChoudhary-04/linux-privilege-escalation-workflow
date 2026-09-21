# 01 — First 5 MinuteD 

> **You have a low-privileged Linux shell. What should you do first?**

The first few minutes after obtaining a shell are critical.

Do not immediately start searching for exploits.

First establish:

```text
WHO am I?
WHERE am I?
WHAT system am I on?
WHAT privileges do I have?
WHAT environment am I operating in?
```

This section provides a practical, command-by-command starting workflow.

---

## 🎯 Objective

By the end of this workflow, you should have a basic understanding of:

* Current user
* UID and GID
* Group memberships
* Hostname
* Operating system
* Kernel
* Architecture
* Current working directory
* PATH
* Environment
* Sudo permissions
* Basic process state
* Basic network state

The output from this phase determines where deeper enumeration should begin.

---

## 🧭 The First 5-Minute Workflow

Follow this order:

```text
1. whoami
      ↓
2. id
      ↓
3. hostname
      ↓
4. pwd
      ↓
5. uname -a
      ↓
6. cat /etc/os-release
      ↓
7. groups
      ↓
8. sudo -l
      ↓
9. ps aux
      ↓
10. ss -tulpn
      ↓
11. echo $PATH
      ↓
12. env
```

Do not treat this as a blind checklist.

After each command, briefly interpret the result.

---

## 1. Identify the Current User

### Command

```bash
whoami
```

### Why?

Before investigating privileges, determine which account you currently control.

### Example

```text
user
```

### Look For

The username tells you which account is currently active.

However, the username alone is not enough.

Immediately continue with:

```bash
id
```

---

## 2. Identify UID, GID and Groups

### Command

```bash
id
```

### Why?

`id` provides a more complete picture of the current identity.

It shows:

* UID
* GID
* Supplementary groups

### Example

```text
uid=1000(user) gid=1000(user) groups=1000(user),1001(dev)
```

### Look For

Pay attention to:

```text
uid=
gid=
groups=
```

### Decision

```text
Interesting group?
       │
       ├── YES ──► Investigate what that group controls
       │
       └── NO ───► Continue
```

Do not assume a group is useful simply because its name sounds privileged.

Determine what access it actually grants.

### Next Command

```bash
hostname
```

---

## 3. Identify the Host

### Command

```bash
hostname
```

### Why?

The hostname helps establish which system you are operating on.

This becomes particularly useful when working with multiple machines, containers, or pivoted environments.

### Look For

Record the hostname.

Then determine the operating system.

### Next Command

```bash
uname -a
```

---

## 4. Identify Kernel and Architecture

### Command

```bash
uname -a
```

### Why?

This provides information about:

* Kernel
* Kernel release
* Architecture
* Host information

### Example

```text
Linux target 6.x.x-generic x86_64 GNU/Linux
```

### Look For

Pay attention to:

```text
Kernel version
Architecture
```

The kernel version can later become relevant when investigating potential kernel vulnerabilities.

#### Important

Do **not** immediately assume:

```text
Old kernel = vulnerable = root
```

Kernel exploitation requires additional validation.

### Next Command

```bash
cat /etc/os-release
```

---

## 5. Identify the Distribution

### Command

```bash
cat /etc/os-release
```

### Why?

The distribution and version provide important context for later enumeration.

They can affect:

* Installed tools
* Configuration locations
* Package managers
* Service management
* Default security settings
* Vulnerability applicability

### Look For

Typical fields include:

```text
NAME=
VERSION=
ID=
VERSION_ID=
```

### Next Step

Record the distribution and version.

Then return to identity-related information.

### Next Command

```bash
groups
```

---

## 6. Review Group Memberships

### Command

```bash
groups
```

### Why?

Some Linux groups provide access to resources that can become relevant to privilege escalation.

### Example

```text
user sudo docker
```

### Look For

Investigate groups that provide access to:

* Administrative functionality
* Devices
* Containers
* Sensitive files
* System management
* Security-relevant resources

### Decision

```text
Interesting group?
       │
       ├── YES
       │    ↓
       │  Determine exactly what access it provides
       │
       └── NO
            ↓
          Continue
```

### Next Command

```bash
sudo -l
```

---

## 7. Check Sudo Permissions

### Command

```bash
sudo -l
```

### Why?

This checks what commands the current user is permitted to execute through `sudo`.

This is one of the highest-value early checks.

### Possible Outcomes

#### Case A — No sudo access

You may see an error indicating that the user is not permitted to run sudo.

#### Action

Continue enumeration.

---

#### Case B — Sudo permissions exist

You may see something similar to:

```text
User user may run the following commands:
    (root) /usr/bin/example
```

Do not immediately exploit it.

First ask:

```text
What command?
      ↓
As which user?
      ↓
With what arguments?
      ↓
Under what conditions?
      ↓
Can the command's behavior cross the privilege boundary?
```

Then investigate the specific permission.

---

#### Case C — Password required

A sudo rule may exist but require authentication.

Record that condition because it affects exploitability.

---

### Decision

```text
Useful sudo permission?
       │
       ├── YES ──► Investigate sudo path
       │
       └── NO ───► Continue enumeration
```

### Next Command

```bash
ps aux
```

---

## 8. Enumerate Running Processes

### Command

```bash
ps aux
```

### Why?

Processes reveal what is currently running and which users own those processes.

This is especially useful for identifying:

* Root processes
* Custom applications
* Security software
* Scheduled activity
* Unusual services
* Applications running from unusual paths

### Look For

Focus on processes running as:

```text
root
```

or other privileged identities.

### Decision

```text
Interesting privileged process?
       │
       ├── YES
       │    ↓
       │  Identify:
       │  - executable
       │  - arguments
       │  - owner
       │  - configuration
       │  - files/dependencies
       │
       └── NO
            ↓
          Continue
```

Do not assume a root process is exploitable.

The important question is:

> **Can the current user influence the process or something it trusts?**

### Next Command

```bash
ss -tulpn
```

---

## 9. Enumerate Listening Services

### Command

```bash
ss -tulpn
```

### Why?

This shows listening network sockets and can reveal services that are:

* Locally accessible
* Network accessible
* Running with elevated privileges
* Not obvious from the user's normal workflow

### Look For

Pay attention to:

```text
Local Address
Port
Process
```

Examples:

```text
127.0.0.1:8080
0.0.0.0:80
127.0.0.1:3306
```

A service bound only to localhost can still be important.

### Decision

```text
Interesting local service?
       │
       ├── YES
       │    ↓
       │  Identify the application
       │
       └── NO
            ↓
          Continue
```

Later, investigate interesting services in:

```text
04-PROCESS-SERVICE-ENUMERATION/
05-NETWORK-ENUMERATION/
```

### Next Command

```bash
echo $PATH
```

---

## 10. Inspect PATH

### Command

```bash
echo $PATH
```

### Why?

`PATH` determines where the shell looks for commands when an absolute path is not supplied.

It becomes particularly important when a privileged program executes another command using a relative command name.

### Look For

Review each directory in the PATH.

Questions to ask:

```text
Is a directory writable by my user?
Is an unusual directory present?
Does a privileged process rely on this PATH?
```

#### Important

A writable PATH directory alone does **not** prove an escalation path.

The critical question is:

> **Does privileged execution depend on a command lookup that I can influence?**

This will be investigated later under execution abuse.

### Next Command

```bash
env
```

---

## 11. Inspect Environment Variables

### Command

```bash
env
```

### Why?

Environment variables can reveal useful context such as:

* PATH
* Home directory
* Shell
* Application configuration
* Runtime information
* Potentially exposed secrets

### Look For

Pay particular attention to variables related to:

```text
PATH
HOME
SHELL
USER
PWD
```

and application-specific variables.

#### Important

Environment variables can contain secrets.

Do not automatically treat every variable as exploitable.

Instead:

```text
Interesting variable
       ↓
What consumes it?
       ↓
Who executes that program?
       ↓
Can I influence privileged behavior?
```

---

## 12. Confirm the Current Directory

### Command

```bash
pwd
```

### Why?

Knowing your current location provides context for the files and applications around your shell.

It can also reveal whether you started in:

* A user's home directory
* An application directory
* A temporary directory
* A deployment directory
* A mounted filesystem

### Next

List the directory contents:

```bash
ls -la
```

This begins the deeper filesystem investigation.

---

## 🧠 The First 5-Minute Decision Tree

The initial workflow can be summarized as:

```text
START
  │
  ▼
whoami
  │
  ▼
id
  │
  ├── Interesting group?
  │        └──► Investigate group
  │
  ▼
hostname
  │
  ▼
uname -a
  │
  ▼
cat /etc/os-release
  │
  ▼
groups
  │
  ├── Interesting group?
  │        └──► Investigate group
  │
  ▼
sudo -l
  │
  ├── Interesting permission?
  │        └──► Investigate sudo
  │
  ▼
ps aux
  │
  ├── Interesting privileged process?
  │        └──► Investigate process
  │
  ▼
ss -tulpn
  │
  ├── Interesting service?
  │        └──► Investigate service
  │
  ▼
echo $PATH
  │
  ├── Suspicious/writable path?
  │        └──► Investigate execution path
  │
  ▼
env
  │
  ├── Interesting variable?
  │        └──► Investigate consumer
  │
  ▼
pwd
  │
  ▼
ls -la
  │
  ▼
DEEP ENUMERATION
```

---

## ⚠️ Don't Make These Mistakes

### Mistake 1 — Running commands without reading the output

Bad workflow:

```text
Run command
↓
Copy output
↓
Run next command
```

Better:

```text
Run command
↓
Understand output
↓
Ask what it means
↓
Choose next action
```

---

### Mistake 2 — Assuming every unusual result is exploitable

For example:

```text
Root process found
```

does not mean:

```text
Root process = privilege escalation
```

You still need to determine whether you can influence it.

---

### Mistake 3 — Jumping directly to kernel exploits

An old kernel may look exciting.

But always investigate simpler privilege boundaries first.

The goal is not:

> **Find the most sophisticated exploit.**

The goal is:

> **Identify and validate a viable privilege-escalation path.**

---

### Mistake 4 — Ignoring groups

A single group membership can sometimes be more important than dozens of individual file findings.

Always inspect:

```bash
id
groups
```

early.

---

### Mistake 5 — Treating automated tools as the answer

Automated tools are excellent for discovery.

But the workflow should be:

```text
Tool finding
    ↓
Understand finding
    ↓
Manually validate
    ↓
Determine exploitability
```

---

## ⚡ First 5 Minutes — Command Reference

For a quick reminder:

```bash
# Identity
whoami
id
groups

# Host
hostname
uname -a
cat /etc/os-release

# Privileges
sudo -l

# Processes
ps aux

# Network
ss -tulpn

# Environment
echo $PATH
env

# Location
pwd
ls -la
```

**Do not blindly execute this list and move on.**

The purpose of the workflow is to interpret each result and branch into the appropriate investigation.

---

## 🧠 Remember

The first five minutes are not about finding root immediately.

They are about building a **mental model of the machine**.

```text
IDENTITY
   ↓
SYSTEM
   ↓
PRIVILEGES
   ↓
PROCESSES
   ↓
NETWORK
   ↓
ENVIRONMENT
   ↓
FILESYSTEM
   ↓
DEEP ENUMERATION
```

The most important question remains:

> **What can this user influence that something more privileged trusts?**

---

## 🚀 Next Stage

Once the initial five-minute assessment is complete, move to:

```text
02-SYSTEM-ENUMERATION/
```

There we will go deeper into the operating system, kernel, architecture, users, groups, installed software, environment, and system configuration.

The next stage should be driven by what we discovered here—not by blindly running every available command.
