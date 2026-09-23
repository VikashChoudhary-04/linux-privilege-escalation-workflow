# 04 — Process & Service Enumeration

> **Goal:** Identify processes and services running with higher privileges, understand how they are started, determine what files and commands they trust, and find whether your current user can influence them.

---

## 🎯 Core Question

> **What is running with more privilege, and can I influence it?**

A privileged process by itself is **not** a vulnerability.

The interesting situation is:

```text
Privileged process
        ↓
Uses a file / script / command / configuration / dependency
        ↓
Current user can influence it
        ↓
Influence crosses a privilege boundary
        ↓
Potential privilege-escalation path
```

The objective of this stage is therefore not simply to find `root` processes.

You need to understand:

* What is running?
* Who owns it?
* How was it started?
* What executable does it run?
* What configuration does it use?
* What files does it depend on?
* What commands does it execute?
* Can the current user modify any of those components?

---

## 🧠 The Mental Model

For every interesting process or service, ask:

```text
WHO runs it?
    ↓
WHAT does it execute?
    ↓
WHERE is the executable?
    ↓
WHAT configuration does it use?
    ↓
WHAT files does it access?
    ↓
WHAT commands does it call?
    ↓
CAN I modify any trusted component?
    ↓
CAN that influence cross the privilege boundary?
```

This prevents the common mistake of seeing a root-owned service and immediately assuming it is exploitable.

---

## 1. Identify Your Current Process

Start by understanding the shell you currently control.

### 🔎 ENUMERATE

```bash
ps -p $$ -o pid,ppid,user,group,args
```

### WHY

This identifies:

* Your shell PID
* Parent PID
* User
* Group
* Command being executed

### LOOK FOR

Example:

```text
PID   PPID USER     GROUP    COMMAND
1234  1200 www-data www-data /bin/bash
```

You now know the shell context in which your commands execute.

### ➡️ NEXT STEP

Move from your shell to the complete process list.

---

## 2. List All Processes

### 🔎 ENUMERATE

```bash
ps aux
```

### WHY

This gives a broad view of processes running on the system.

Look for:

* Root-owned processes
* Custom applications
* Scripts
* Databases
* Web applications
* Backup processes
* Monitoring software
* Unusual binaries
* Processes running from `/opt`
* Processes running from `/usr/local`
* Processes using files in writable locations

### ⚠️ LOOK FOR

Example:

```text
root   1200  0.0  ... /opt/scripts/backup.sh
root   1300  0.1  ... /usr/local/bin/custom-service
```

These deserve investigation.

### 🧠 IMPORTANT

Do **not** assume:

```text
root process = vulnerability
```

Instead:

```text
root process
    ↓
what does it execute?
    ↓
can I influence that execution?
```

---

## 3. Show Processes With More Useful Columns

The default `ps aux` output can be difficult to interpret.

Use:

```bash
ps -eo user,pid,ppid,%cpu,%mem,lstart,args
```

### WHY

This makes ownership, process IDs, parent processes, start time, and command line easier to analyze.

### LOOK FOR

Focus on:

```text
USER
PID
PPID
ARGS
```

Especially:

```text
USER = root
```

combined with:

```text
custom executable
custom script
unusual path
writable dependency
```

---

## 4. Focus on Root-Owned Processes

### 🔎 ENUMERATE

```bash
ps -eo user,pid,ppid,args | awk '$1 == "root"'
```

### WHY

This reduces the process list to processes running as `root`.

### LOOK FOR

Examples:

```text
root  1000  1 /usr/sbin/sshd
root  1200  1 /opt/app/service
root  1300  1 /usr/local/bin/backup
```

Standard system services are not automatically interesting.

Custom processes deserve closer investigation.

---

## 5. Identify Parent Processes

Processes often inherit their execution context from another process.

### 🔎 ENUMERATE

```bash
ps -eo pid,ppid,user,args --forest
```

### WHY

The process tree helps answer:

> **Who started this process?**

### LOOK FOR

Example:

```text
init
 └── service
      └── /opt/app/app.py
```

Or:

```text
cron
 └── /bin/sh
      └── /opt/scripts/backup.sh
```

The parent process can reveal the mechanism responsible for execution.

---

## 6. Investigate an Interesting PID

Suppose you find:

```text
root  2450  1 /opt/scripts/backup.sh
```

Start with:

```bash
ps -p 2450 -o pid,ppid,user,group,args
```

Then inspect the process directory:

```bash
ls -la /proc/2450
```

### WHY

`/proc/<PID>` exposes information about the running process.

Useful locations include:

```text
/proc/2450/cmdline
/proc/2450/cwd
/proc/2450/exe
/proc/2450/environ
/proc/2450/fd
```

---

## 7. Inspect the Executable

### 🔎 ENUMERATE

```bash
readlink -f /proc/2450/exe
```

### WHY

This identifies the actual executable associated with the process.

### Example

```text
/usr/local/bin/custom-service
```

Now inspect it:

```bash
ls -la /usr/local/bin/custom-service
```

Then:

```bash
file /usr/local/bin/custom-service
```

### LOOK FOR

You want to know:

* Who owns it?
* Is it writable?
* Is it a script?
* Is it a custom binary?
* Is it located in an unusual directory?

---

## 8. Inspect the Current Working Directory

### 🔎 ENUMERATE

```bash
readlink -f /proc/2450/cwd
```

### WHY

A process may execute or load files relative to its working directory.

### LOOK FOR

Interesting locations such as:

```text
/tmp
/opt/app
/home/user/app
/var/www
/custom/path
```

If a privileged process works inside a directory that an unprivileged user can modify, investigate further.

---

## 9. Inspect the Process Command Line

### 🔎 ENUMERATE

```bash
tr '\0' ' ' < /proc/2450/cmdline
echo
```

Or:

```bash
cat /proc/2450/cmdline | tr '\0' ' '
echo
```

### WHY

The command line can reveal:

* Configuration files
* Script paths
* Arguments
* Log paths
* Working directories
* Environment-specific options

### Example

```text
/usr/bin/python3 /opt/app/backup.py --config /opt/app/config.ini
```

Now you have multiple investigation targets:

```text
backup.py
config.ini
/usr/bin/python3
```

---

## 10. Inspect the Process Environment

### 🔎 ENUMERATE

```bash
tr '\0' '\n' < /proc/2450/environ
```

### WHY

The environment may reveal:

* `PATH`
* Application directories
* Configuration paths
* Library paths
* Runtime settings
* Credentials or tokens accidentally exposed by a service

### ⚠️ LOOK FOR

Variables such as:

```text
PATH=
HOME=
PWD=
LD_LIBRARY_PATH=
PYTHONPATH=
```

Do not automatically treat the presence of these variables as exploitable.

First determine whether the privileged process actually uses them in a controllable way.

---

## 11. Inspect Open Files

### 🔎 ENUMERATE

If available:

```bash
lsof -p 2450
```

Alternatively:

```bash
ls -la /proc/2450/fd
```

### WHY

This can reveal files currently used by the process.

You may discover:

```text
configuration files
logs
sockets
temporary files
libraries
input/output files
```

### LOOK FOR

A particularly interesting pattern is:

```text
root process
    ↓
opens file
    ↓
file is writable by current user
```

That requires validation.

---

## 12. Inspect Service Status

On systems using `systemd`:

### 🔎 ENUMERATE

```bash
systemctl --type=service --state=running
```

### WHY

This provides a service-oriented view rather than only looking at processes.

### LOOK FOR

Focus on:

* Custom services
* Third-party applications
* Services running as root
* Services using files under `/opt`
* Services using custom scripts
* Unusual service names

---

## 13. List All Service Units

### 🔎 ENUMERATE

```bash
systemctl list-units --type=service
```

For installed service definitions:

```bash
systemctl list-unit-files --type=service
```

### WHY

A service may not currently be running but may still be configured to start automatically.

---

## 14. Inspect an Interesting Service

Suppose you find:

```text
custom-backup.service
```

Run:

```bash
systemctl status custom-backup.service
```

Then:

```bash
systemctl cat custom-backup.service
```

### WHY

You want to discover the service's actual execution configuration.

### LOOK FOR

Especially:

```text
User=
Group=
ExecStart=
ExecStartPre=
ExecStartPost=
WorkingDirectory=
Environment=
EnvironmentFile=
```

Example:

```ini
[Service]
User=root
ExecStart=/opt/scripts/backup.sh
```

This gives you a direct investigation path:

```text
root service
    ↓
ExecStart=/opt/scripts/backup.sh
    ↓
inspect backup.sh
```

---

## 15. Inspect the Service Executable

If the service contains:

```text
ExecStart=/opt/scripts/backup.sh
```

run:

```bash
ls -la /opt/scripts/backup.sh
```

Then:

```bash
stat /opt/scripts/backup.sh
```

Then:

```bash
namei -l /opt/scripts/backup.sh
```

### WHY

You need to check both:

1. The file itself
2. Every directory in its path

### LOOK FOR

Potentially interesting:

```text
-rwxrwxrwx
```

or:

```text
-rwxrwxr-x
```

or ownership by an unprivileged user/group.

Also inspect parent directories.

A root-owned file inside a user-writable directory can be just as important as a writable file.

---

## 16. Check Service File Permissions

First locate the service file:

```bash
systemctl show -p FragmentPath custom-backup.service
```

Then inspect it:

```bash
ls -la /path/to/service-file
```

And:

```bash
namei -l /path/to/service-file
```

### WHY

A privileged service depends on its unit configuration.

If an unprivileged user can modify a trusted service definition, the service configuration itself may become an attack path.

---

## 17. Search for Custom Services

### 🔎 ENUMERATE

```bash
find /etc/systemd /lib/systemd /usr/lib/systemd -type f -name "*.service" 2>/dev/null
```

### WHY

This can reveal service definitions that deserve manual review.

### LOOK FOR

Prioritize:

```text
custom names
organization-specific services
scripts under /opt
scripts under /usr/local
services using unusual binaries
```

Do not waste time manually analyzing every standard distribution service.

---

## 18. Investigate Scripts Used by Services

Suppose:

```text
ExecStart=/opt/scripts/backup.sh
```

Inspect it:

```bash
sed -n '1,240p' /opt/scripts/backup.sh
```

Then:

```bash
ls -la /opt/scripts/backup.sh
```

Then:

```bash
namei -l /opt/scripts/backup.sh
```

### WHY

A root service may execute a script containing commands that rely on:

* Relative paths
* Other scripts
* External commands
* Configuration files
* Temporary files
* Environment variables
* Wildcards
* Writable directories

---

## 19. Trace Commands Used by a Script

Suppose the script contains:

```bash
tar ...
cp ...
rsync ...
python ...
backup-tool ...
```

For each command:

### 🔎 ENUMERATE

```bash
command -v tar
command -v cp
command -v rsync
command -v python
```

### WHY

This identifies which executable will normally be resolved.

Then inspect it:

```bash
ls -la "$(command -v tar)"
```

### 🧠 IMPORTANT

The question is not:

> "Does the script call `tar`?"

The question is:

> **"How does the privileged process resolve and execute `tar`, and can I influence that resolution?"**

This leads directly into execution-abuse analysis later.

---

## 20. Investigate PATH Used by a Service

If the service uses commands without absolute paths:

```text
tar
cp
python
backup-tool
```

inspect the service environment:

```bash
systemctl show custom-backup.service -p Environment
```

And:

```bash
systemctl show custom-backup.service -p EnvironmentFiles
```

If necessary, inspect the unit:

```bash
systemctl cat custom-backup.service
```

### LOOK FOR

Example:

```text
ExecStart=/opt/scripts/backup.sh
```

and inside the script:

```bash
tar -czf ...
```

The next question becomes:

```text
What PATH does the service use?
```

If an attacker-controlled directory can appear earlier in the privileged process's search path, investigate the trust relationship carefully.

Do not assume PATH presence alone proves exploitability.

---

## 21. Search for Recently Modified Service-Related Files

### 🔎 ENUMERATE

```bash
find /etc/systemd /opt /usr/local -type f -mtime -7 2>/dev/null
```

### WHY

Recently modified custom scripts or service definitions can be useful clues during a lab.

### LOOK FOR

Files that are:

* Recently changed
* Root-owned
* Executed by root
* Located in custom directories

Then inspect their permissions.

---

## 22. Look for Processes Running From Writable Locations

A powerful pattern is:

```text
Privileged process
        ↓
Executable/script/config
        ↓
Writable by current user
```

Start with:

```bash
ps -eo user,pid,args | awk '$1 == "root"'
```

Identify an interesting executable.

Then:

```bash
readlink -f /proc/<PID>/exe
```

Then:

```bash
ls -la /path/to/executable
```

Then:

```bash
namei -l /path/to/executable
```

### 🧪 VALIDATE

Determine:

```text
Who owns the executable?
Who can write it?
Who can write its parent directories?
Is it actually executed with elevated privileges?
```

---

## 23. Investigate Root-Owned Processes Using User-Writable Files

Suppose you discover:

```text
root process
    ↓
/opt/app/config.ini
```

Check:

```bash
ls -la /opt/app/config.ini
```

Then:

```bash
namei -l /opt/app/config.ini
```

If writable:

```text
root process
    ↓
configuration file
    ↓
current user can modify it
```

This is a **candidate finding**.

It is not automatically an exploitable vulnerability.

You must determine:

```text
Does the process trust the modified setting?
Does the process execute the affected behavior?
Does that behavior cross the privilege boundary?
```

---

## 24. Look for Suspicious Temporary-File Usage

Privileged services sometimes interact with temporary locations.

Search process command lines:

```bash
ps aux | grep -E '/tmp|/var/tmp|/dev/shm'
```

Then investigate relevant processes.

Check:

```bash
ls -la /tmp
ls -la /var/tmp
ls -la /dev/shm
```

### ⚠️ LOOK FOR

A suspicious pattern might be:

```text
root process
    ↓
uses predictable temporary file
    ↓
file location is writable
```

This requires careful validation because temporary-file behavior varies significantly between applications.

---

## 25. Check for Service Managers Other Than systemd

Not every Linux system uses systemd.

Check:

```bash
ps -p 1 -o pid,user,args
```

### INTERPRET

If you see:

```text
systemd
```

use `systemctl`.

If you see another init/service manager, investigate the platform's service mechanism accordingly.

---

## 26. Check Cron-Started Processes

Cron is covered in detail later, but process enumeration can reveal cron activity.

### 🔎 ENUMERATE

```bash
ps aux | grep -E '[c]ron|[c]rond'
```

Then:

```bash
ps -ef --forest | grep -A 10 -B 5 '[c]ron'
```

### WHY

You may discover:

```text
cron
 └── script
```

This creates a bridge to:

```text
07-SCHEDULED-EXECUTION
```

Do not fully investigate cron here unless a running process gives you a reason to.

---

## 27. Process-Service Investigation Workflow

When you discover an interesting process:

```text
1. Identify PID
       ↓
2. Identify owner
       ↓
3. Identify parent process
       ↓
4. Identify executable
       ↓
5. Identify command line
       ↓
6. Identify working directory
       ↓
7. Identify environment
       ↓
8. Identify open files
       ↓
9. Identify configuration
       ↓
10. Check permissions
       ↓
11. Determine user influence
       ↓
12. Validate privilege impact
```

---

## 28. The Practical Decision Tree

```text
Find interesting process
        │
        ▼
Who owns it?
        │
        ├── Current user
        │      └── Usually lower priority
        │
        └── root / privileged user
                │
                ▼
        What does it execute?
                │
                ▼
        Script or binary?
                │
        ┌───────┴────────┐
        ▼                ▼
      Script            Binary
        │                │
        ▼                ▼
Inspect script       Inspect executable
        │                │
        └───────┬────────┘
                ▼
        What does it trust?
                │
                ▼
       Config / command / file
                │
                ▼
        Can I modify it?
                │
        ┌───────┴────────┐
        ▼                ▼
       NO               YES
        │                │
        ▼                ▼
 Investigate other     Validate influence
 trusted components       │
                          ▼
                 Does it cross privilege
                      boundary?
                     │         │
                    NO        YES
                     │         │
                     ▼         ▼
                 Move on    Document finding
                              ↓
                           Exploit
                              ↓
                           Verify
```

---

## 29. High-Value Patterns to Remember

### Pattern 1 — Root Service + Writable Script

```text
root service
    ↓
/opt/script.sh
    ↓
current user can modify script
```

**Priority:** High

---

### Pattern 2 — Root Process + Writable Configuration

```text
root process
    ↓
config file
    ↓
current user can modify configuration
```

**Priority:** Investigate and validate.

---

### Pattern 3 — Root Process + User-Writable Directory

```text
root process
    ↓
/opt/app/
    ↓
current user can modify directory
```

Investigate:

* executable replacement
* scripts
* configuration
* dependencies
* generated files

Do not assume the directory alone is enough.

---

### Pattern 4 — Root Script + Relative Command

```text
root script
    ↓
runs "tool"
    ↓
tool resolved through PATH
```

Investigate the execution environment and whether an unprivileged user can influence command resolution.

---

### Pattern 5 — Root Service + User-Writable Dependency

```text
root service
    ↓
loads file/library/script
    ↓
current user controls dependency
```

This can become a privilege-escalation path depending on how the dependency is loaded.

---

## 30. Finding vs Vulnerability vs Exploit

Keep these separate.

### Finding

```text
Root-owned service exists.
```

This is only an observation.

### Potential Vulnerability

```text
Root service executes /opt/backup.sh
and current user can modify /opt/backup.sh.
```

Now there is a privilege-boundary concern.

### Exploitable Path

```text
Root service
    ↓
executes user-modifiable script
    ↓
script executes with root privileges
    ↓
controlled behavior produces elevated execution
```

Now the path has been validated.

---

## 31. Validation Checklist

Before calling a process/service a privilege-escalation vector, verify:

```text
[ ] Is the process actually privileged?
[ ] Is the process actually running?
[ ] What executable does it use?
[ ] What script/configuration does it use?
[ ] Who owns the component?
[ ] Who can modify the component?
[ ] Can I influence its execution?
[ ] Does the process trust that component?
[ ] Does it execute with elevated privileges?
[ ] Can the influence cross the privilege boundary?
[ ] Can the behavior be reproduced safely?
```

---

## 32. Common Mistakes

### ❌ Mistake 1 — "Every root process is exploitable"

Wrong.

Root processes are normal on Linux.

The important question is:

> **Can I influence what the privileged process does?**

---

### ❌ Mistake 2 — Looking only at `ps aux`

A process list is only the beginning.

You still need to investigate:

```text
PID
owner
parent
executable
command line
working directory
environment
open files
configuration
permissions
```

---

### ❌ Mistake 3 — Ignoring parent processes

A process may be launched by:

```text
systemd
cron
supervisor
container runtime
custom launcher
```

The parent often explains how execution occurs.

---

### ❌ Mistake 4 — Ignoring directory permissions

You might see:

```text
-rwxr-xr-x root root script.sh
```

and conclude:

```text
not writable
```

But check:

```bash
namei -l /path/to/script.sh
```

A writable parent directory may change the analysis.

---

### ❌ Mistake 5 — Assuming environment variables are exploitable

Seeing:

```text
LD_LIBRARY_PATH
PATH
PYTHONPATH
```

does not automatically mean privilege escalation.

You must determine:

```text
Does the privileged process use it?
Can the current user control it?
Does that control affect privileged execution?
```

---

### ❌ Mistake 6 — Exploiting before understanding

Do not immediately modify a privileged service.

First establish:

```text
what runs
→
who runs it
→
what it trusts
→
what you control
→
why that crosses the boundary
```

---

## 33. Quick Command Reference

### Processes

```bash
ps aux
ps -ef
ps -eo user,pid,ppid,args
ps -eo pid,ppid,user,args --forest
```

### Root processes

```bash
ps -eo user,pid,ppid,args | awk '$1 == "root"'
```

### Current shell

```bash
ps -p $$ -o pid,ppid,user,group,args
```

### Process executable

```bash
readlink -f /proc/<PID>/exe
```

### Working directory

```bash
readlink -f /proc/<PID>/cwd
```

### Command line

```bash
tr '\0' ' ' < /proc/<PID>/cmdline
```

### Environment

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

### Open files

```bash
lsof -p <PID>
ls -la /proc/<PID>/fd
```

### Services

```bash
systemctl --type=service --state=running
systemctl list-units --type=service
systemctl list-unit-files --type=service
```

### Service investigation

```bash
systemctl status <service>
systemctl cat <service>
systemctl show -p FragmentPath <service>
systemctl show <service>
```

### File investigation

```bash
ls -la /path/to/file
stat /path/to/file
namei -l /path/to/file
```

---

## 34. The 60-Second Process/Service Check

When time is limited:

```bash
ps -eo user,pid,ppid,args | awk '$1 == "root"'
```

Then identify suspicious PIDs.

For each:

```bash
readlink -f /proc/<PID>/exe
readlink -f /proc/<PID>/cwd
tr '\0' ' ' < /proc/<PID>/cmdline; echo
```

If it is a systemd service:

```bash
systemctl status <service>
systemctl cat <service>
```

Then inspect interesting files:

```bash
ls -la /path/to/file
namei -l /path/to/file
```

### 🧠 Remember

```text
ROOT PROCESS
     ↓
WHAT?
     ↓
WHERE?
     ↓
WHO OWNS IT?
     ↓
WHO CAN MODIFY IT?
     ↓
WHAT DOES IT TRUST?
     ↓
CAN I INFLUENCE IT?
     ↓
VALIDATE
```

---

## 35. Process & Service Enumeration Checklist

```text
[ ] Identify current shell process
[ ] List all processes
[ ] Identify root-owned processes
[ ] Understand process hierarchy
[ ] Identify interesting PIDs
[ ] Inspect /proc/<PID>
[ ] Identify executable
[ ] Identify working directory
[ ] Inspect command line
[ ] Inspect environment
[ ] Inspect open files
[ ] Enumerate running services
[ ] Identify custom services
[ ] Inspect service configuration
[ ] Identify ExecStart
[ ] Identify service user/group
[ ] Inspect service scripts
[ ] Inspect service dependencies
[ ] Check permissions
[ ] Check writable paths
[ ] Validate user influence
[ ] Determine privilege impact
[ ] Document interesting findings
```

---

## 36. Where This Leads

Process and service enumeration often reveals paths into several later stages:

```text
Interesting process
        │
        ├── Writable script
        │       ↓
        │   03 Filesystem / 14 Exploitation
        │
        ├── Scheduled execution
        │       ↓
        │   07 Scheduled Execution
        │
        ├── Weak privilege mechanism
        │       ↓
        │   06 Privilege Mechanisms
        │
        ├── Credential/config exposure
        │       ↓
        │   08 Credentials & Secrets
        │
        ├── PATH/dependency abuse
        │       ↓
        │   09 Execution Abuse
        │
        └── Vulnerable software
                ↓
            11 Vulnerable Software
```

The purpose of this stage is therefore **discovery and understanding**, not blindly exploiting every unusual process.

---

## 💡 Remember

> **A process tells you what is running.**
>
> **A service tells you how it is managed.**
>
> **Permissions tell you what you can influence.**
>
> **Validation tells you whether that influence actually crosses the privilege boundary.**

The key pattern is:

```text
PRIVILEGED EXECUTION
        +
USER INFLUENCE
        +
TRUST RELATIONSHIP
        =
POTENTIAL PRIVILEGE-ESCALATION PATH
```

And always remember:

> **Finding something interesting is not the same as proving it is exploitable.**
