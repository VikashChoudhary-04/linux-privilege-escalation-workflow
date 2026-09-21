# 00 — Foundations

> **Before learning how to escalate privileges, understand what privileges are, where they come from, and what creates a privilege boundary.**

Linux privilege escalation becomes much easier when you understand the underlying system instead of memorizing individual exploits.

This section establishes the mental model used throughout the repository.

---

## 🎯 Objectives

By the end of this section, you should understand:

* Linux users and UIDs
* Groups and GIDs
* Root and effective privileges
* File ownership and permissions
* SUID and SGID
* Processes and process ownership
* Services and privileged execution
* `sudo`
* Linux capabilities
* Privilege boundaries
* The difference between a finding and an exploitable path
* The mindset required for privilege escalation

---

## 🧠 The Core Idea

Privilege escalation is fundamentally about **crossing a privilege boundary**.

A low-privileged user normally cannot perform certain actions that a privileged user can.

For example:

```text
Low-Privileged User
        │
        │  cannot normally
        ▼
Privileged Resource
```

A vulnerability or misconfiguration may create a controllable path:

```text
Low-Privileged User
        │
        │  influence
        ▼
Privileged Process / File / Service
        │
        ▼
Higher Privilege
```

Therefore, throughout this repository, ask:

> **What privileged action exists, and can my current user influence it?**

---

## 👤 Users

Linux identifies users primarily through a **UID (User ID)**.

The username is mainly a human-readable representation of that identity.

Check the current user:

```bash
whoami
```

Check the current identity in more detail:

```bash
id
```

Example:

```text
uid=1000(user) gid=1000(user) groups=1000(user)
```

The important pieces are:

```text
uid=1000(user)
gid=1000(user)
groups=1000(user)
```

### Why does this matter?

Privilege escalation begins with understanding **what identity you currently control**.

You cannot determine whether you have gained additional privileges until you know your starting privileges.

---

## 👑 Root

The traditional Linux superuser is:

```text
root
```

Root normally has extensive control over the operating system.

Its UID is:

```text
0
```

Check:

```bash
id
```

A root shell commonly reports:

```text
uid=0(root)
```

### Important

Do not rely only on:

```bash
whoami
```

When verifying privileges, also inspect:

```bash
id
```

because UID, GID, supplementary groups, and effective privileges can all matter.

---

## 👥 Groups

A user can belong to multiple groups.

Check them with:

```bash
groups
```

or:

```bash
id
```

Groups can grant access to resources that the user would not otherwise be able to access.

For privilege escalation, this makes group membership important.

Examples of groups that may warrant investigation depending on the environment include:

```text
docker
lxd
disk
adm
sudo
```

Membership alone does **not** automatically mean privilege escalation.

The correct workflow is:

```text
Group discovered
      ↓
Understand what the group controls
      ↓
Determine what access it grants
      ↓
Determine whether that access crosses
a privilege boundary
      ↓
Validate
```

---

## 📁 File Ownership

Every file has an owner and a group.

View them with:

```bash
ls -la /path/to/file
```

Example:

```text
-rwxr-xr-- 1 root root 1234 example
```

The important fields include:

```text
owner → root
group → root
permissions → rwxr-xr--
```

This matters because privileged files and directories may be part of an escalation path if a low-privileged user can influence them.

---

## 🔐 Linux File Permissions

Linux commonly represents basic permissions as:

```text
r = read
w = write
x = execute
```

They are applied to:

```text
owner
group
others
```

Example:

```text
-rwxr-xr--
```

Break it down:

```text
- | rwx | r-x | r--
    │      │      │
  owner   group  others
```

Therefore:

```text
Owner  → read + write + execute
Group  → read + execute
Others → read
```

---

## 🧠 Why Permissions Matter for Privilege Escalation

Suppose a privileged process executes:

```text
/opt/scripts/backup.sh
```

and:

```text
backup.sh → owned by root
backup.sh → writable by user
```

The important question is not simply:

> "Is this file writable?"

The important question is:

> **"Who executes this file, and can my modification influence that privileged execution?"**

That distinction is fundamental.

---

## ⚙️ Processes

A process is a running instance of a program.

View processes with:

```bash
ps aux
```

A process has an associated user.

For example:

```text
root      ... /usr/bin/example
user      ... /bin/bash
```

This immediately gives us an important privilege-escalation question:

> **What is running as root that I can influence?**

That question will repeatedly appear throughout the workflow.

---

## 🔄 Privileged Execution

Many escalation paths can be reduced to this pattern:

```text
Privileged Process
       │
       ├── executes a file
       ├── loads a library
       ├── reads a configuration
       ├── calls another command
       ├── accesses a directory
       └── uses an environment variable
                │
                ▼
        Can the current user
        influence any of it?
```

If the answer is yes, investigate further.

---

## 🛡️ `sudo`

`sudo` allows permitted users to execute commands with another user's privileges, commonly root.

Check the current user's sudo permissions with:

```bash
sudo -l
```

This is one of the first commands to consider during privilege-escalation enumeration.

The important question is not simply:

> "Does sudo exist?"

Instead:

> **"What is this user permitted to execute, as whom, and under what conditions?"**

The workflow becomes:

```text
sudo -l
   ↓
Any permitted commands?
   │
   ├── No
   │    ↓
   │  Continue enumeration
   │
   └── Yes
        ↓
   Identify command
        ↓
   Understand behavior
        ↓
   Check arguments/environment/files
        ↓
   Determine whether privilege boundary
   can be crossed
        ↓
   Validate
```

---

## 🔑 SUID

SUID is a special permission bit that can cause an executable to run with the privileges of its file owner.

This is important because a root-owned SUID executable may execute with root privileges even when launched by a lower-privileged user.

Conceptually:

```text
User
 │
 │ executes
 ▼
SUID executable
 │
 │ runs with owner's privileges
 ▼
Potentially privileged execution
```

The important question is:

> **Can the user influence what this privileged executable does?**

SUID will be covered in detail later in:

```text
06-PRIVILEGE-MECHANISMS/
```

---

## 🔐 SGID

SGID is another special permission mechanism.

Depending on context, SGID can affect:

* Executable behavior
* Group ownership inheritance for directories

For privilege escalation, SGID executables can deserve investigation because they may execute with the privileges of their owning group.

Again:

```text
SGID found
    ↓
Who owns the group?
    ↓
What does the program do?
    ↓
Can the user influence its behavior?
    ↓
Does this create a useful privilege boundary?
```

---

## 🧩 Linux Capabilities

Linux capabilities divide certain traditionally root-level powers into more granular privileges.

Instead of giving a process complete root privileges, a capability can grant a specific privileged operation.

This creates another important enumeration area:

```text
Process / Binary
       ↓
Capabilities
       ↓
What privileged operation is granted?
       ↓
Can the current user influence the process?
       ↓
Potential escalation path?
```

Capabilities will be covered in detail later.

---

## ⏰ Scheduled Execution

Linux systems can automatically execute programs through mechanisms such as:

* Cron
* Systemd timers
* Other scheduled tasks

A common escalation pattern is:

```text
Privileged scheduled task
        │
        ▼
Executes script/program
        │
        ▼
Low-privileged user can modify
something involved in execution
        │
        ▼
Potential privilege boundary
```

The key question is:

> **Can I influence something that a privileged scheduled task will execute or trust?**

---

## ⚙️ Services

Services often run continuously with elevated privileges.

For example:

```text
root
 │
 ▼
service
 │
 ▼
application
 │
 ├── executable
 ├── configuration
 ├── libraries
 ├── scripts
 └── files
```

If a low-privileged user can influence one of those trusted components, investigate it.

The important mental model is:

> **Privileged service + controllable dependency = investigate.**

---

## 🌐 Network Services

A service listening on a local interface can also be relevant.

For example:

```text
127.0.0.1:8080
```

may expose an application that is not accessible externally.

Therefore:

```text
Listening service
       ↓
Who owns it?
       ↓
What application?
       ↓
What privileges?
       ↓
Can the current user interact with it?
       ↓
Does interaction provide a path to higher privilege?
```

Network enumeration is therefore part of privilege-escalation analysis, not simply reconnaissance.

---

## 🔑 Credentials and Secrets

Credentials can sometimes provide a path to a more privileged identity.

Examples include:

* Passwords
* SSH keys
* Configuration credentials
* Application secrets
* Tokens
* Environment variables
* Shell history

But remember:

```text
Secret found
     ↓
Is it valid?
     ↓
What identity does it belong to?
     ↓
What access does that identity have?
     ↓
Does it provide higher privilege?
```

Finding a credential is **not automatically privilege escalation**.

---

## 🧠 The Universal Privilege-Escalation Pattern

Many seemingly different techniques follow the same structure:

```text
1. Something has higher privileges.
             ↓
2. It performs an action.
             ↓
3. The action depends on something.
             ↓
4. The current user can influence that dependency.
             ↓
5. The influence changes privileged behavior.
             ↓
6. The privilege boundary is crossed.
```

Examples:

```text
SUID
    privileged executable
          ↓
    controllable behavior
```

```text
Cron
    privileged scheduled task
          ↓
    controllable script/input
```

```text
Service
    privileged service
          ↓
    controllable executable/config/dependency
```

```text
Sudo
    privileged command execution
          ↓
    controllable command behavior
```

```text
Capability
    privileged capability
          ↓
    controllable process/binary
```

The mechanism changes.

The reasoning stays similar.

---

## 🔎 Finding vs. Vulnerability

This distinction is critical.

### Finding

Something interesting was discovered.

Example:

```text
A SUID binary exists.
```

### Vulnerability / Misconfiguration

The finding creates a security weakness.

Example:

```text
The SUID binary provides a controllable path
to privileged execution.
```

### Exploitable Path

The weakness can actually be used under the current conditions.

Therefore:

```text
Finding
   ↓
Investigation
   ↓
Validation
   ↓
Exploitable Path
```

Never assume:

```text
Finding = Root
```

---

## 🧪 Validation Mindset

Before exploitation, ask:

```text
Can I reproduce the behavior?
        ↓
Can I control the relevant input?
        ↓
Does the relevant component run with
higher privileges?
        ↓
Does my influence reach that component?
        ↓
Does the resulting action cross the
privilege boundary?
```

This prevents blind exploitation.

---

## 🧭 The Enumeration Mindset

Good enumeration is not:

```text
Run everything.
```

Good enumeration is:

```text
Observe
   ↓
Ask a question
   ↓
Run a relevant command
   ↓
Interpret the result
   ↓
Form a hypothesis
   ↓
Investigate
```

For example:

```text
id
 ↓
Interesting group discovered
 ↓
What does this group control?
 ↓
Investigate group
 ↓
Does it provide privileged access?
```

---

## 🧠 The Six Questions

When you are lost during a privilege-escalation assessment, return to these:

```text
1. WHO am I?

2. WHAT am I running?

3. WHO owns it?

4. WHO can modify it?

5. WHAT runs with more privilege?

6. CAN I influence it?
```

If you can answer these questions consistently, you can reason about most privilege-escalation situations without relying on memorized exploit lists.

---

## 🔄 Foundation → Workflow

The concepts in this section become the building blocks for the practical workflow.

```text
Foundations
    │
    ├── Users / Groups
    │        ↓
    │    Identity Enumeration
    │
    ├── Permissions
    │        ↓
    │    Filesystem Enumeration
    │
    ├── Processes
    │        ↓
    │    Process Enumeration
    │
    ├── Services
    │        ↓
    │    Service Enumeration
    │
    ├── Sudo
    │        ↓
    │    Privilege Enumeration
    │
    ├── SUID / SGID
    │        ↓
    │    Special Permission Enumeration
    │
    └── Trust Boundaries
             ↓
        Finding Validation
             ↓
        Privilege Escalation
```

---

## 📝 Foundation Checklist

Before moving into the practical workflow, make sure you understand:

* [ ] UID and GID
* [ ] Root and UID `0`
* [ ] Users and groups
* [ ] File ownership
* [ ] Linux permissions
* [ ] SUID
* [ ] SGID
* [ ] Processes
* [ ] Services
* [ ] `sudo`
* [ ] Linux capabilities
* [ ] Privilege boundaries
* [ ] Findings vs. vulnerabilities
* [ ] Validation before exploitation
* [ ] The six core questions

---

## 🚀 Next Stage

Once these concepts are understood, move to:

```text
01-FIRST-5-MINUTES/
```

The next stage turns the concepts above into a **real command-by-command workflow** beginning from an obtained low-privileged shell.
