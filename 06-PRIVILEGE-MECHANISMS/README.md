# 06 — Privilege Mechanisms

> **Goal:** Identify Linux mechanisms that allow programs, users, or groups to perform actions with privileges beyond the current user's normal permissions, then determine whether any of those mechanisms are misconfigured and exploitable.

## 🎯 Core Question

> **What legitimate Linux mechanism allows privileged execution, and can my current user influence or abuse it?**

Linux provides many mechanisms that allow processes to operate with privileges different from the invoking user's normal privileges.

Important examples include:

```text
sudo
SUID
SGID
Linux capabilities
Privileged groups
Polkit
```

These mechanisms are **not vulnerabilities by themselves**.

The real question is:

```text
Privilege mechanism
        ↓
What privilege does it provide?
        ↓
Who can use it?
        ↓
What can they execute/access/control?
        ↓
Can the current user influence that behavior?
        ↓
Does it cross the privilege boundary?
```

## 🧠 The Mental Model

For every privilege mechanism, ask:

```text
WHO gets the privilege?
        ↓
WHAT privilege do they receive?
        ↓
UNDER WHAT CONDITIONS?
        ↓
WHAT program/action is trusted?
        ↓
CAN I influence that program/action?
        ↓
CAN I cross the privilege boundary?
```

This prevents a common mistake:

> **Finding a privileged mechanism does not automatically mean finding a privilege-escalation vulnerability.**

## 1. Start With Identity

Before analyzing privilege mechanisms, establish your current identity.

### 🔎 ENUMERATE

```bash
whoami
id
groups
```

### WHY

You need to know:

* Current username
* UID
* Primary GID
* Supplementary groups
* Whether you are already root
* Which privileged groups you belong to

### LOOK FOR

Example:

```text
uid=1000(user) gid=1000(user) groups=1000(user),27(sudo)
```

The important clue here is:

```text
27(sudo)
```

This means the user belongs to the `sudo` group.

But group membership alone does not tell you exactly what the user can execute.

That is why you continue with `sudo -l`.

## 2. Check Sudo Permissions

This is one of the highest-priority privilege checks.

### 🔎 ENUMERATE

```bash
sudo -l
```

### WHY

`sudo -l` asks the system which commands the current user is permitted to execute through `sudo`.

Possible outcomes include:

```text
User may run specific commands
```

or:

```text
(ALL) ALL
```

or:

```text
(root) /usr/bin/some-command
```

## 3. Understand Sudo Output

Suppose you see:

```text
User user may run the following commands:
    (root) /usr/bin/vim
```

This means:

```text
Current user
      ↓
sudo
      ↓
/usr/bin/vim
      ↓
root
```

The important question becomes:

> **What can this permitted program do when executed with elevated privileges?**

Do not immediately classify every permitted binary as exploitable.

## 4. Check Whether Sudo Requires a Password

Run:

```bash
sudo -l
```

Look carefully at the rule.

For example:

```text
NOPASSWD: /usr/bin/example
```

means the command may be executed without an interactive sudo password prompt.

### 🧠 WHY IT MATTERS

`NOPASSWD` can change the practical usability of a sudo rule.

But:

```text
NOPASSWD
```

does **not** automatically mean:

```text
privilege escalation
```

You still need to understand the command and its permitted behavior.

## 5. Identify Wildcards in Sudo Rules

A sudo rule may contain patterns such as:

```text
/usr/bin/example *
```

or:

```text
/opt/tools/*
```

### 🔎 ENUMERATE

```bash
sudo -l
```

### LOOK FOR

* `*`
* `?`
* Broad path patterns
* Multiple allowed arguments
* User-controlled paths
* Writable scripts
* Commands that accept arbitrary files

### 🧠 IMPORTANT

Wildcard behavior in sudoers can be subtle.

Do not assume:

```text
* = unrestricted access
```

Analyze exactly what arguments and paths the rule permits.

## 6. Inspect Sudo Version

### 🔎 ENUMERATE

```bash
sudo --version
```

### WHY

Version information can identify:

* Sudo implementation
* Configuration features
* Potentially relevant historical vulnerabilities

### ⚠️ IMPORTANT

A version number is a **clue**, not proof of vulnerability.

The correct process is:

```text
Version
   ↓
Identify relevant vulnerability
   ↓
Check affected versions
   ↓
Check configuration
   ↓
Validate applicability
```

## 7. Check Sudo Configuration

The primary configuration file is:

```bash
ls -la /etc/sudoers
```

You can inspect readable configuration with:

```bash
sudo -l
```

If you have legitimate permission to inspect the configuration:

```bash
sudo cat /etc/sudoers
```

Also investigate included configuration:

```bash
ls -la /etc/sudoers.d/
```

### WHY

Sudo rules may be defined in:

```text
/etc/sudoers
/etc/sudoers.d/*
```

### ⚠️ IMPORTANT

Do not modify sudo configuration during enumeration.

## 8. Identify SUID Files

SUID is one of the classic Linux privilege mechanisms.

### 🔎 ENUMERATE

```bash
find / -perm -4000 -type f 2>/dev/null
```

A more explicit form:

```bash
find / -type f -perm -u=s 2>/dev/null
```

### WHY

An executable with the SUID bit may execute with the privileges of its **file owner** rather than the privileges of the user launching it.

Example:

```text
-rwsr-xr-x 1 root root /path/to/program
```

The `s` in the owner execute position indicates SUID.

## 9. Understand SUID

Normal executable:

```text
-rwxr-xr-x
```

SUID executable:

```text
-rwsr-xr-x
```

The important difference is:

```text
s
```

### 🧠 Mental Model

```text
User
  ↓
executes SUID program
  ↓
program receives effective privileges of file owner
```

If the owner is `root`:

```text
User
  ↓
SUID root-owned program
  ↓
effective UID may become root
```

But this does **not** mean every SUID binary gives a root shell.

The program must provide some exploitable behavior or weakness.

## 10. Inspect Every Interesting SUID Binary

Suppose you find:

```text
/usr/local/bin/custom-tool
```

Start with:

```bash
ls -la /usr/local/bin/custom-tool
```

Then:

```bash
file /usr/local/bin/custom-tool
```

Then:

```bash
stat /usr/local/bin/custom-tool
```

Then:

```bash
strings /usr/local/bin/custom-tool | head -n 50
```

### WHY

You want to determine:

* What type of binary is it?
* Who owns it?
* Is it custom?
* What does it appear to do?
* Is it a standard system utility?
* Does it reference commands or paths?

## 11. Identify Custom SUID Binaries

Standard system SUID binaries are common.

Prioritize unusual paths:

```text
/opt/
/usr/local/bin/
/usr/local/sbin/
/home/
/tmp/
```

Search:

```bash
find /opt /usr/local /home -type f -perm -4000 2>/dev/null
```

### 🧠 WHY

A custom root-owned SUID binary is often more interesting than a standard distribution binary because it may contain application-specific logic.

## 12. Check SUID Binary Dependencies

For an ELF binary:

```bash
ldd /path/to/binary
```

### WHY

This can reveal dynamically linked libraries.

Example:

```text
libsomething.so => /opt/app/lib/libsomething.so
```

Now investigate:

```bash
ls -la /opt/app/lib/libsomething.so
```

### ⚠️ IMPORTANT

`ldd` should be used carefully with untrusted binaries. For safer static inspection, use:

```bash
readelf -d /path/to/binary
```

or:

```bash
objdump -p /path/to/binary
```

## 13. Inspect SUID Binary Strings

### 🔎 ENUMERATE

```bash
strings /path/to/binary
```

For a smaller initial view:

```bash
strings /path/to/binary | head -n 100
```

### LOOK FOR

Interesting strings such as:

```text
/bin/sh
/bin/bash
system(
exec
popen
/tmp/
/opt/
/usr/local/
config
```

### 🧠 IMPORTANT

Strings are clues.

A string appearing in a binary does not prove that the corresponding code path is reachable or exploitable.

## 14. Inspect SUID Binary System Calls

If available:

```bash
strace /path/to/binary
```

### WHY

This can reveal:

* Files opened
* Commands executed
* Libraries loaded
* System calls
* Paths accessed

For a SUID binary, this can help answer:

> **What does this privileged program actually do?**

### ⚠️ IMPORTANT

Tracing privileged programs can behave differently depending on system security controls and may produce large output.

Use focused tracing when possible.

## 15. Identify SGID Files

SGID on an executable can cause the program to run with the file's group privileges.

### 🔎 ENUMERATE

```bash
find / -perm -2000 -type f 2>/dev/null
```

Or:

```bash
find / -type f -perm -g=s 2>/dev/null
```

### WHY

SGID can provide access to resources controlled by a privileged group.

Example:

```text
-rwxr-sr-x root shadow /path/to/program
```

The important part is:

```text
group execute position = s
```

## 16. Understand SGID

The basic model is:

```text
User
  ↓
SGID executable
  ↓
effective group = file's group
```

This can matter when the group has access to:

* sensitive files
* administrative resources
* device files
* application data

### 🧠 REMEMBER

> **SUID changes effective user privilege. SGID changes effective group privilege.**

## 17. Investigate SGID Programs

For an interesting SGID binary:

```bash
ls -la /path/to/binary
```

Then:

```bash
file /path/to/binary
```

Then:

```bash
stat /path/to/binary
```

Ask:

```text
What group does it run as?
What resources does that group control?
Can I influence its behavior?
```

## 18. Enumerate Linux Capabilities

Capabilities split traditional root privileges into smaller units.

### 🔎 ENUMERATE

```bash
getcap -r / 2>/dev/null
```

### WHY

You may find programs with capabilities such as:

```text
cap_setuid
cap_setgid
cap_dac_override
cap_dac_read_search
cap_net_raw
cap_net_admin
```

A capability can grant a process specific privileged powers without giving it all traditional root privileges.

## 19. Understand Capability Output

Example:

```text
/usr/bin/example cap_setuid+ep
```

Break it down:

```text
cap_setuid
```

Capability:

```text
+ep
```

Effective and permitted capability sets.

### 🧠 IMPORTANT

Do not simply memorize capability names.

Ask:

```text
What does this capability allow?
How is the binary designed to use it?
Can I control the binary's behavior?
Does that control cross the privilege boundary?
```

## 20. Investigate High-Interest Capabilities

Some capabilities deserve immediate attention.

Examples include:

```text
cap_setuid
cap_setgid
cap_dac_override
cap_dac_read_search
cap_sys_admin
cap_sys_ptrace
```

But the actual risk depends on:

```text
Capability
+
Program
+
Arguments
+
User influence
+
Execution path
```

There is no universal rule that:

```text
capability = root
```

## 21. Inspect Capability-Bearing Programs

Suppose:

```text
/usr/local/bin/custom-tool cap_setuid+ep
```

Run:

```bash
ls -la /usr/local/bin/custom-tool
```

Then:

```bash
file /usr/local/bin/custom-tool
```

Then:

```bash
getcap /usr/local/bin/custom-tool
```

Then investigate its behavior.

### Questions

```text
Can I execute it?
What does it do?
Does it accept arguments?
Can I influence files?
Does it change UID/GID?
Does it invoke another program?
```

## 22. Enumerate Privileged Groups

Groups can provide access to powerful system resources.

Start with:

```bash
id
```

Then:

```bash
groups
```

Look for potentially sensitive groups such as:

```text
sudo
wheel
docker
lxd
disk
adm
shadow
video
audio
libvirt
systemd-journal
```

### ⚠️ IMPORTANT

Group names vary by distribution and configuration.

A group being present does not automatically mean it provides privilege escalation.

## 23. Investigate the `docker` Group

If:

```bash
id
```

shows:

```text
docker
```

check:

```bash
getent group docker
```

Then:

```bash
ls -la /var/run/docker.sock 2>/dev/null
```

### WHY

Membership in a container-management group can provide substantial control over containers.

The exact security impact depends on the host configuration.

Detailed container-specific analysis belongs in:

```text
10-SPECIAL-ENVIRONMENTS/docker.md
```

## 24. Investigate the `lxd` Group

If present:

```bash
getent group lxd
```

### WHY

LXD/container management privileges can have significant security implications.

Detailed container-specific analysis belongs in:

```text
10-SPECIAL-ENVIRONMENTS/lxd.md
```

At this stage, record the finding and continue systematic enumeration.

## 25. Investigate the `disk` Group

If the current user belongs to:

```text
disk
```

investigate what block devices and resources are exposed.

### 🔎 ENUMERATE

```bash
getent group disk
```

Then:

```bash
ls -l /dev | head
```

And:

```bash
findmnt
```

### 🧠 WHY

Groups that provide direct access to devices can potentially bypass normal filesystem permissions.

Do not access or modify raw devices unnecessarily.

The objective here is to understand the privilege boundary and validate exposure safely.

## 26. Investigate the `adm` Group

Check:

```bash
getent group adm
```

### WHY

On some distributions, members of `adm` can read system logs.

Logs may contain:

* usernames
* application errors
* internal paths
* service information
* authentication-related events
* accidentally logged secrets

This connects to:

```text
08-CREDENTIALS-AND-SECRETS
```

## 27. Investigate the `shadow` Group

Check:

```bash
getent group shadow
```

### WHY

Access to password-hash data can have security implications.

Check membership first.

Do not modify authentication databases during enumeration.

## 28. Understand Polkit

Polkit is a framework used by Linux systems to control authorization for privileged operations.

The general model is:

```text
Unprivileged process
       ↓
requests privileged action
       ↓
Polkit authorization
       ↓
privileged service performs action
```

### 🧠 WHY

Misconfigured authorization policies or vulnerable components can create privilege-escalation paths.

## 29. Identify Polkit

### 🔎 ENUMERATE

```bash
command -v pkexec
```

Then:

```bash
pkexec --version
```

If available:

```bash
pkaction --version
```

### WHY

This establishes whether common Polkit tooling exists.

## 30. Inspect Polkit Actions

If available:

```bash
pkaction
```

For detailed information:

```bash
pkaction --verbose
```

### LOOK FOR

Interesting privileged actions and authorization policies.

Do not assume an action is exploitable simply because it exists.

## 31. Inspect Polkit Rules

Common locations include:

```text
/etc/polkit-1/rules.d/
/etc/polkit-1/localauthority/
```

Check:

```bash
ls -la /etc/polkit-1/ 2>/dev/null
```

And:

```bash
find /etc/polkit-1 -type f 2>/dev/null
```

### WHY

Custom rules can change authorization behavior.

## 32. Investigate Polkit Configuration

If a custom rule exists:

```bash
sed -n '1,240p' /path/to/rule
```

Ask:

```text
Who is authorized?
For what action?
Under what conditions?
Can my user satisfy those conditions?
Does the action result in privileged behavior?
```

## 33. Privilege Mechanism Decision Tree

```text
Start
  │
  ▼
Check sudo -l
  │
  ├── Interesting sudo rule?
  │       │
  │       ├── NO ───────────────┐
  │       │                     │
  │       ▼                     │
  │     Analyze command         │
  │       │                     │
  │       ▼                     │
  │   Can influence it?         │
  │       │                     │
  │      YES                    │
  │       ↓                     │
  │   Validate                  │
  │                             │
  ▼                             │
Find SUID files                 │
  │                             │
  ▼                             │
Custom/interesting SUID?        │
  │                             │
  ├── NO ───────────────────────┤
  │                             │
  ▼                             │
Analyze binary                  │
  │                             │
  ▼                             │
Can influence behavior?         │
  │                             │
  ▼                             │
Validate                        │
                                │
  ▼                             │
Find SGID files                 │
  │                             │
  ▼                             │
Analyze interesting programs    │
                                │
  ▼                             │
Find capabilities               │
  │                             │
  ▼                             │
Analyze capability + program    │
                                │
  ▼                             │
Check privileged groups         │
  │                             │
  ▼                             │
Investigate relevant groups     │
                                │
  ▼                             │
Check Polkit                    │
  │                             │
  ▼                             │
Analyze authorization           │
                                │
  └───────────────┬─────────────┘
                  ▼
          Validate privilege
              boundary
                  │
          ┌───────┴────────┐
          ▼                ▼
         NO               YES
          │                │
          ▼                ▼
       Move on          Document
                           ↓
                       Exploit
                           ↓
                       Verify
```

## 34. Sudo Investigation Workflow

Use this compact workflow:

```text
sudo -l
   ↓
What commands are allowed?
   ↓
Which user may execute them?
   ↓
Which arguments are permitted?
   ↓
NOPASSWD?
   ↓
Wildcard/path restrictions?
   ↓
What does the command actually do?
   ↓
Can current user influence it?
   ↓
Does it cross the privilege boundary?
```

## 35. SUID Investigation Workflow

```text
Find SUID
   ↓
Identify unusual binaries
   ↓
Check owner
   ↓
Check permissions
   ↓
Check file type
   ↓
Understand behavior
   ↓
Inspect dependencies
   ↓
Inspect referenced paths
   ↓
Can current user influence execution?
   ↓
Validate
```

## 36. Capability Investigation Workflow

```text
getcap -r /
      ↓
Find capability-bearing binary
      ↓
Identify capability
      ↓
Understand capability
      ↓
Understand binary behavior
      ↓
Can user influence behavior?
      ↓
Does capability affect privilege boundary?
      ↓
Validate
```

## 37. Group Investigation Workflow

```text
id
 ↓
Interesting group?
 ↓
What resource does group control?
 ↓
Can current user access resource?
 ↓
What actions does access permit?
 ↓
Does it cross privilege boundary?
 ↓
Validate
```

## 38. Privilege Mechanism Priority

A practical investigation order is:

```text
1. sudo -l
2. Interesting SUID binaries
3. Linux capabilities
4. Privileged groups
5. SGID binaries
6. Polkit
```

### 🧠 WHY

This order prioritizes mechanisms that can quickly reveal direct privilege boundaries while keeping the investigation structured.

It is not a universal rule.

If another finding is clearly more relevant to the host, follow the evidence.

## 39. Finding vs Vulnerability

Keep these concepts separate.

### Finding

```text
/usr/bin/example has SUID set.
```

This is a fact.

### Potential Vulnerability

```text
The SUID binary performs privileged operations
using user-controlled input.
```

This is a security concern.

### Exploitable Path

```text
Current user
    ↓
can control relevant input
    ↓
SUID program trusts that input
    ↓
privileged operation occurs
    ↓
privilege boundary crossed
```

That is the path you validate.

## 40. Common Mistakes

### ❌ Mistake 1 — Treating all SUID binaries as vulnerable

Most systems have legitimate SUID binaries.

For example:

```text
/usr/bin/passwd
/usr/bin/su
```

may be perfectly normal.

Focus on:

* unusual binaries
* custom programs
* unexpected locations
* known vulnerable versions
* dangerous behavior

### ❌ Mistake 2 — Treating `sudo -l` as enough

Finding:

```text
(root) /usr/bin/example
```

is only the beginning.

You still need to understand:

```text
what the program does
what arguments it accepts
what files it accesses
what the sudo rule actually permits
```

### ❌ Mistake 3 — Assuming every capability means root

Capabilities are granular.

For example:

```text
cap_net_raw
```

does not provide the same privileges as:

```text
cap_setuid
```

Always understand the specific capability.

### ❌ Mistake 4 — Ignoring group membership

A user may have no useful SUID finding but belong to:

```text
docker
lxd
disk
adm
```

Groups can create privilege boundaries that are not visible through ordinary file permissions.

### ❌ Mistake 5 — Ignoring file ownership

When investigating a privileged mechanism, always check:

```bash
ls -la /path/to/file
```

and:

```bash
namei -l /path/to/file
```

You need to know who controls the complete execution path.

### ❌ Mistake 6 — Exploiting before validating

Do not immediately execute a suspected exploit.

First establish:

```text
mechanism
→
privilege
→
trusted component
→
user influence
→
boundary crossing
```

Then validate safely.

## 41. The 5-Minute Privilege Mechanism Check

If time is limited, run:

### Step 1 — Identity

```bash
id
groups
```

### Step 2 — Sudo

```bash
sudo -l
```

### Step 3 — SUID

```bash
find / -perm -4000 -type f 2>/dev/null
```

### Step 4 — SGID

```bash
find / -perm -2000 -type f 2>/dev/null
```

### Step 5 — Capabilities

```bash
getcap -r / 2>/dev/null
```

### Step 6 — Interesting Groups

```bash
getent group sudo wheel docker lxd disk adm shadow 2>/dev/null
```

### Step 7 — Polkit

```bash
command -v pkexec
pkexec --version 2>/dev/null
```

Then investigate the highest-value findings.

## 42. Quick Command Reference

### Identity

```bash
whoami
id
groups
```

### Sudo

```bash
sudo -l
sudo --version
ls -la /etc/sudoers
ls -la /etc/sudoers.d/
```

### SUID

```bash
find / -perm -4000 -type f 2>/dev/null
find /opt /usr/local /home -type f -perm -4000 2>/dev/null
```

### SGID

```bash
find / -perm -2000 -type f 2>/dev/null
```

### File analysis

```bash
ls -la /path/to/file
stat /path/to/file
file /path/to/file
namei -l /path/to/file
```

### Binary analysis

```bash
strings /path/to/binary
readelf -d /path/to/binary
objdump -p /path/to/binary
```

### Capabilities

```bash
getcap -r / 2>/dev/null
getcap /path/to/binary
```

### Groups

```bash
getent group
getent group docker
getent group lxd
getent group disk
getent group adm
```

### Polkit

```bash
command -v pkexec
pkexec --version
pkaction
pkaction --verbose
```

## 43. Complete Privilege Mechanism Checklist

```text
[ ] Run whoami
[ ] Run id
[ ] Run groups
[ ] Run sudo -l
[ ] Understand every sudo rule
[ ] Check NOPASSWD rules
[ ] Check sudo wildcards
[ ] Check sudo version
[ ] Enumerate SUID files
[ ] Identify unusual SUID binaries
[ ] Inspect SUID ownership
[ ] Inspect SUID behavior
[ ] Inspect SUID dependencies
[ ] Enumerate SGID files
[ ] Identify unusual SGID binaries
[ ] Enumerate Linux capabilities
[ ] Identify high-impact capabilities
[ ] Inspect capability-bearing binaries
[ ] Enumerate privileged groups
[ ] Investigate docker membership
[ ] Investigate lxd membership
[ ] Investigate disk membership
[ ] Investigate adm membership
[ ] Investigate shadow membership
[ ] Identify Polkit
[ ] Inspect relevant Polkit actions/rules
[ ] Determine user influence
[ ] Validate privilege boundary
[ ] Document the finding
```

## 44. Master Decision Model

When you find a privilege mechanism, reduce it to four questions:

```text
1. WHAT PRIVILEGE?
       ↓
2. WHO GETS IT?
       ↓
3. WHAT CAN THEY CONTROL?
       ↓
4. CAN THAT CONTROL CROSS THE PRIVILEGE BOUNDARY?
```

Example:

```text
SUID root binary
       ↓
root effective privileges
       ↓
accepts user-controlled file
       ↓
file influences privileged behavior
       ↓
potential privilege escalation
```

Another example:

```text
docker group
       ↓
access to Docker control interface
       ↓
container-management privileges
       ↓
potential host-level impact
       ↓
validate actual configuration
```

## 💡 Remember

> **Privilege mechanisms are not vulnerabilities. Misconfigured privilege mechanisms are where the investigation begins.**

The most important pattern in this entire section is:

```text
PRIVILEGE
    +
TRUST
    +
USER CONTROL
    +
PRIVILEGE BOUNDARY
    =
POTENTIAL ESCALATION PATH
```

And the workflow remains:

```text
ENUMERATE
    ↓
IDENTIFY
    ↓
UNDERSTAND
    ↓
CHECK CONTROL
    ↓
VALIDATE
    ↓
EXPLOIT
    ↓
VERIFY
```

Do not memorize:

```text
sudo = root
SUID = root
docker = root
capability = root
```

Memorize the better rule:

> **Find the privilege mechanism → understand exactly what privilege it provides → determine what you control → prove whether your control crosses the privilege boundary.**

## 🔗 Where These Findings Lead

```text
Privilege Mechanism
       │
       ├── Sudo
       │     ↓
       │ 14-EXPLOITATION/sudo-exploitation.md
       │
       ├── SUID
       │     ↓
       │ 14-EXPLOITATION/suid-exploitation.md
       │
       ├── Capabilities
       │     ↓
       │ 14-EXPLOITATION/capability-exploitation.md
       │
       ├── Privileged Groups
       │     ↓
       │ 10-SPECIAL-ENVIRONMENTS
       │
       ├── Polkit
       │     ↓
       │ 13-FINDING-VALIDATION
       │
       └── Nothing useful
             ↓
         Continue to
      07-SCHEDULED-EXECUTION
```

**The objective of this stage is not to exploit everything you find.**

The objective is to identify which Linux privilege mechanisms actually create a **credible, validated path across the privilege boundary**.
