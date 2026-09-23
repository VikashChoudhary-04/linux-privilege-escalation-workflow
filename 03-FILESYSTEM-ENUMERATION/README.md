## 03 — Filesystem Enumeration

> **Find files, directories, permissions, ownership, mounts, and trusted paths that may create a controllable privilege boundary.**

The filesystem is one of the most important areas to investigate after obtaining a low-privileged Linux shell.

But the objective is **not** simply:

```text
Find writable files.
```

The real question is:

> **Can I influence a file, directory, executable, configuration, or dependency that a more privileged process trusts or executes?**

That distinction is fundamental.

---

## 🎯 Objectives

By the end of this section, you should be able to:

* Understand file ownership
* Understand permission relationships
* Identify writable files
* Identify writable directories
* Identify executable files
* Identify privileged files
* Locate interesting configuration files
* Identify backup files
* Identify scripts
* Identify unusual files
* Investigate mount points
* Analyze trusted execution paths
* Determine whether filesystem access creates a privilege boundary

---

## 🧠 Core Mental Model

Think about filesystem privilege escalation as:

```text
Privileged Process
       │
       ▼
Trusted File / Directory / Path
       │
       ▼
Can current user influence it?
       │
       ├── NO ──► Continue
       │
       └── YES
             │
             ▼
        Can influence affect
        privileged execution?
             │
             ├── NO ──► Continue
             │
             └── YES
                   │
                   ▼
                Validate
```

This pattern appears repeatedly throughout Linux privilege escalation.

---

## 🧭 Filesystem Workflow

Follow this progression:

```text
1. Identify current location
        ↓
2. Inspect permissions
        ↓
3. Inspect ownership
        ↓
4. Understand writable locations
        ↓
5. Search for interesting files
        ↓
6. Find scripts
        ↓
7. Find configuration files
        ↓
8. Find backups
        ↓
9. Investigate privileged files
        ↓
10. Investigate trusted execution paths
        ↓
11. Inspect mounts
        ↓
12. Validate interesting findings
```

---

## 1. Identify the Current Directory

### Command

```bash id="k3lq9j"
pwd
```

### Why?

Before examining files, establish where your shell is currently located.

This gives context for subsequent filesystem operations.

### Next Command

```bash id="2c5v8h"
ls -la
```

---

## 2. Inspect the Current Directory

### Command

```bash id="j9g0jv"
ls -la
```

### Why?

This shows:

* Files
* Directories
* Hidden files
* Ownership
* Permissions
* Timestamps

### Look For

Pay attention to:

```text
Owner
Group
Permissions
Filename
```

For example:

```text
-rwxrwxr-- 1 root dev 1234 script.sh
```

The important question is:

> **Can my current user modify or influence this file?**

---

## 3. Understand File Permissions

A permission string such as:

```text
-rwxr-xr--
```

can be interpreted as:

```text
- | rwx | r-x | r--
    │      │      │
  owner   group  others
```

Meaning:

```text
Owner  → read/write/execute
Group  → read/execute
Others → read
```

For privilege escalation, always connect permissions to ownership.

For example:

```text
root-owned + user-writable
```

is far more interesting than:

```text
user-owned + user-writable
```

---

## 4. Inspect a Specific File

### Command

```bash id="l5u7i9"
ls -la /path/to/file
```

For more detailed metadata:

```bash id="c4q4ga"
stat /path/to/file
```

### Why?

`stat` provides detailed information about:

* Owner
* Group
* Permissions
* Timestamps
* File type

### Example

```text id="f7r7cw"
stat /path/to/script.sh
```

### Look For

Ask:

```text
Who owns it?
Who owns the group?
Can I write it?
Can I execute it?
Who executes it?
What uses it?
```

---

## 5. Inspect Parent Directory Permissions

A common mistake is checking only the file.

For:

```text id="6w2w4h"
/opt/app/config.conf
```

also inspect:

```bash id="e5oq9n"
ls -ld /opt
ls -ld /opt/app
ls -la /opt/app
```

### Why?

Directory permissions determine whether users can:

* Enter the directory
* Create files
* Delete files
* Rename files
* Modify directory contents

A file may be protected while its containing directory is writable.

---

## 6. Understand Directory Permissions

For directories, the meaning of permissions differs from regular files.

### Read

Allows listing directory contents.

### Write

Allows modifying directory entries, such as creating or deleting files, subject to other restrictions.

### Execute

Allows traversal/access through the directory.

Therefore:

```text
Directory:
r + w + x
```

must be interpreted differently from:

```text
Regular file:
r + w + x
```

This distinction is important when investigating writable directories.

---

## 7. Find Writable Files

A broad starting point:

```bash id="axm8k5"
find / -type f -writable 2>/dev/null
```

### Why?

This identifies files that the current user can write to.

### Important

This command may produce a lot of output.

Do not assume every result is useful.

The important question is:

> **Is the writable file trusted or executed by something more privileged?**

---

## 8. Prioritize Writable Files

Instead of treating every writable file equally, investigate based on context.

### High-interest pattern

```text id="u0q8a2"
Writable file
     +
Privileged owner/process
     +
Trusted execution/read path
```

For example:

```text
root-owned script
       +
user can modify it
       +
root executes it
```

This deserves immediate investigation.

### Low-interest example

```text
user-owned personal note
```

This normally does not create a privilege boundary.

---

## 9. Find Writable Directories

### Command

```bash id="a1ps3n"
find / -type d -writable 2>/dev/null
```

### Why?

Writable directories can become relevant when a privileged process:

* Executes files from them
* Loads libraries from them
* Reads configuration from them
* Searches them for commands
* Creates files inside them

### Decision

```text
Writable directory found
        │
        ▼
Who owns it?
        │
        ▼
Who uses it?
        │
        ▼
Is a privileged process involved?
        │
        ├── NO → Continue
        │
        └── YES
              ↓
          Investigate execution path
```

---

## 10. Inspect Temporary Directories

Common temporary locations include:

```text id="f8m1p8"
/tmp
/var/tmp
/dev/shm
```

Inspect them:

```bash id="8xw0pr"
ls -ld /tmp /var/tmp /dev/shm
```

Then:

```bash id="6w6qot"
ls -la /tmp
```

### Why?

Temporary directories are often writable by multiple users and may be used by applications or scripts.

### Important

A writable temporary directory alone is not an escalation.

You need a privileged process that trusts or executes something from that location.

---

## 11. Search for Scripts

Scripts are particularly interesting because their contents can often be understood and, depending on permissions, influenced.

Search common script types:

```bash id="6s0a7f"
find / -type f \( -name "*.sh" -o -name "*.py" -o -name "*.pl" -o -name "*.rb" \) 2>/dev/null
```

### Why?

Scripts may be executed by:

* Root
* Cron
* Systemd
* Services
* Administrative tools

### Decision

```text
Script found
   ↓
Who owns it?
   ↓
Who can modify it?
   ↓
Who executes it?
   ↓
When does it execute?
   ↓
What does it call?
```

### Next Command

For an interesting script:

```bash id="jgnp2d"
ls -la /path/to/script
```

Then:

```bash id="1d0p9j"
sed -n '1,200p' /path/to/script
```

---

## 12. Inspect Script Dependencies

Suppose a privileged script contains:

```text id="7r2k2s"
some-command
```

Do not stop at the script itself.

Ask:

> **How is `some-command` resolved?**

Check:

```bash id="ezf0r8"
command -v some-command
```

Then:

```bash id="p1j1yd"
ls -la "$(command -v some-command)"
```

### Why?

A privileged script may rely on:

* PATH lookup
* Relative paths
* Writable directories
* External programs
* Configuration files

This creates potential execution-abuse paths.

---

## 13. Search for Configuration Files

Common configuration locations include:

```text id="q8c2tc"
/etc
/opt
/usr/local/etc
/var/www
/home
```

A targeted search can be more useful than searching the entire filesystem.

For example:

```bash id="b1f2z0"
find /etc /opt /usr/local/etc -type f \( -name "*.conf" -o -name "*.cfg" -o -name "*.ini" \) 2>/dev/null
```

### Why?

Configuration files may reveal:

* Service configuration
* Credentials
* Paths
* Scripts
* Application behavior
* Privileged execution settings

### Decision

```text
Interesting configuration
        ↓
Who owns it?
        ↓
Can current user read it?
        ↓
Can current user modify it?
        ↓
What consumes it?
        ↓
Does that consumer run with higher privilege?
```

---

## 14. Search for Environment / Application Configuration

Useful names can include:

```text id="p5n3vr"
.env
config
settings
credentials
secrets
```

A targeted example:

```bash id="0n5jzv"
find /opt /var/www /home -type f \( -name ".env" -o -name "*config*" -o -name "*credentials*" -o -name "*secret*" \) 2>/dev/null
```

### Why?

Applications frequently store configuration outside `/etc`.

### Important

Treat discovered secrets as **evidence requiring validation**, not automatic escalation.

---

## 15. Search for Backup Files

Backups can contain:

* Old configurations
* Credentials
* Scripts
* Database dumps
* SSH material
* Previous versions of applications

Search common backup extensions:

```bash id="5i5f48"
find /etc /opt /var/www /home -type f \( -name "*.bak" -o -name "*.backup" -o -name "*.old" -o -name "*.save" -o -name "*~" \) 2>/dev/null
```

### Why?

A backup may contain information that is no longer present in the active configuration.

### Next

For an interesting file:

```bash id="o8m6r4"
ls -la /path/to/file
```

Then inspect it according to its type.

---

## 16. Search for Recently Modified Files

### Command

```bash id="b0r2p6"
find / -type f -mtime -7 2>/dev/null
```

### Why?

Recently modified files can reveal:

* Active application changes
* Recently deployed scripts
* Temporary files
* Administrative activity
* Newly created configuration

### Important

This can produce a large amount of output.

Use it as a clue, not as a definitive escalation technique.

---

## 17. Search for Root-Owned Files

A useful investigation is to identify files owned by root.

For example:

```bash id="5b8v7a"
find / -user root -type f 2>/dev/null
```

This can produce an enormous amount of output.

Therefore, narrow the search when possible.

For example:

```bash id="y8t0ul"
find /opt /usr/local /var/www -user root -type f 2>/dev/null
```

### Why?

Root-owned files become particularly interesting when the current user can influence them or something around them.

The useful pattern is:

```text
Root ownership
      +
User influence
      +
Privileged execution/trust
```

---

## 18. Find SUID Files

SUID is important enough to have its own section later, but filesystem enumeration should identify it.

### Command

```bash id="4kzq5c"
find / -perm -4000 -type f 2>/dev/null
```

### Why?

This identifies files with the SUID permission bit set.

### Look For

Record unusual or custom binaries.

For each interesting result:

```bash id="3d5j1p"
ls -la /path/to/binary
```

Then:

```bash id="0j0qkh"
file /path/to/binary
```

Do not immediately exploit it.

Detailed SUID analysis will be covered in:

```text
06-PRIVILEGE-MECHANISMS/
```

---

## 19. Find SGID Files

### Command

```bash id="j3b4o9"
find / -perm -2000 -type f 2>/dev/null
```

### Why?

SGID executables can execute with the privileges of their owning group.

### Next

For an interesting binary:

```bash id="8w3h4f"
ls -la /path/to/binary
```

Then:

```bash id="q7o5sm"
file /path/to/binary
```

Investigate the owning group and program behavior.

---

## 20. Search for Executables in Interesting Locations

For example:

```bash id="y3k9ax"
find /opt /usr/local/bin /usr/local/sbin -type f -executable 2>/dev/null
```

### Why?

Custom software is often installed outside standard system paths.

Custom binaries deserve closer inspection because their security properties may differ from standard system utilities.

### Decision

```text
Custom executable
       ↓
Who owns it?
       ↓
Who can modify it?
       ↓
Who executes it?
       ↓
What does it call?
       ↓
What files does it trust?
```

---

## 21. Inspect File Types

For an interesting file:

```bash id="m4d8b1"
file /path/to/file
```

### Why?

This determines whether you are dealing with:

* Text
* Script
* ELF executable
* Shared library
* Archive
* Symlink
* Other file type

### Example

```text id="9m7g7g"
ELF 64-bit LSB executable
```

If it is an executable, continue with binary analysis.

If it is a script, inspect its contents.

If it is a symlink, inspect its target.

---

## 22. Inspect Symbolic Links

### Command

```bash id="p2h8h8"
ls -la /path/to/file
```

For a symlink, identify the target:

```bash id="e4p1g6"
readlink -f /path/to/file
```

### Why?

A privileged process may follow a symbolic link to a location that a low-privileged user can influence.

The key questions are:

```text
Who creates the link?
Who follows it?
Who owns the target?
Can the target be changed?
```

---

## 23. Inspect File Access More Precisely

For a path such as:

```text id="v9g5q5"
/opt/app/config.conf
```

inspect every parent directory:

```bash id="r0p4p5"
namei -l /opt/app/config.conf
```

### Why?

`namei -l` helps identify permissions along the entire path.

This is extremely useful because a file may appear protected while a parent directory is writable.

### Think in Paths

```text
/
 ↓
/opt
 ↓
/opt/app
 ↓
/opt/app/config.conf
```

Every component matters.

---

## 24. Inspect ACLs

Some files use Access Control Lists in addition to traditional Unix permissions.

Check:

```bash id="d3h5p7"
getfacl /path/to/file
```

For a directory:

```bash id="7j9f3x"
getfacl /path/to/directory
```

### Why?

Traditional `ls -la` output may not tell the complete permission story.

ACLs can grant or restrict access beyond the basic owner/group/other model.

### Look For

Entries granting the current user or group:

```text
read
write
execute
```

---

## 25. Inspect Mounts

### Command

```bash id="5k0f3n"
findmnt
```

For filesystem type information:

```bash id="h3u4e2"
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS
```

### Why?

Mounts can reveal:

* Shared filesystems
* NFS
* Bind mounts
* Container filesystems
* Writable filesystems
* Unusual mount options

### Decision

```text
Interesting mount
      ↓
What filesystem?
      ↓
Who owns the mount?
      ↓
What options?
      ↓
Who can write?
      ↓
Is privileged software using it?
```

---

## 26. Investigate NFS

First identify NFS mounts:

```bash id="5k5zly"
findmnt -t nfs,nfs4
```

### Why?

NFS configurations can create interesting trust relationships between systems.

The key questions are:

```text
What is exported?
Who can access it?
What permissions are applied?
Are root identities treated specially?
```

Detailed NFS analysis belongs in:

```text
10-SPECIAL-ENVIRONMENTS/
```

---

## 27. Identify World-Writable Locations

A broad search:

```bash id="6q5t8u"
find / -type d -perm -0002 -print 2>/dev/null
```

### Why?

World-writable directories can become interesting when privileged processes use them.

### Important

Do not equate:

```text
world-writable
```

with:

```text
vulnerable
```

The context matters.

Ask:

```text
Who uses this directory?
What gets created here?
What executes from here?
Does a privileged process trust it?
```

---

## 28. Search for World-Writable Files

### Command

```bash id="k7z1qf"
find / -type f -perm -0002 -print 2>/dev/null
```

### Why?

World-writable files can represent weak permissions.

But again:

> **A writable file matters when it affects something important.**

Prioritize files that are:

* Root-owned
* Executed by privileged processes
* Read as privileged configuration
* Used by services
* Used by scheduled tasks

---

## 29. Inspect Interesting Paths with `namei`

Whenever you find something interesting, use:

```bash id="8q2p6w"
namei -l /path/to/file
```

### Why?

This gives you the permission chain from `/` to the target.

This is particularly useful for answering:

> **Can the current user actually reach and influence this path?**

---

## 30. Build the Privileged Execution Chain

Suppose you discover:

```text id="j9q8gp"
/opt/scripts/backup.sh
```

Do not immediately modify it.

Instead ask:

```text id="7m4n2c"
Who owns it?
       ↓
Who can write it?
       ↓
Who executes it?
       ↓
When is it executed?
       ↓
What does it call?
       ↓
What files does it read?
       ↓
What paths does it use?
       ↓
Does the execution happen with higher privilege?
```

This is the central filesystem investigation pattern.

---

## 🧠 The Filesystem Decision Tree

```text id="j8f5uw"
START
  │
  ▼
Inspect current location
  │
  ▼
Permissions + ownership
  │
  ▼
Writable file?
  │
  ├── YES
  │    ↓
  │  Is it trusted/executed by a privileged process?
  │    │
  │    ├── YES → Investigate
  │    └── NO  → Continue
  │
  ▼
Writable directory?
  │
  ├── YES
  │    ↓
  │  Is it used by privileged software?
  │    │
  │    ├── YES → Investigate
  │    └── NO  → Continue
  │
  ▼
Interesting script?
  │
  ├── YES → Identify owner/executor/dependencies
  │
  ▼
Interesting configuration?
  │
  ├── YES → Identify consumer and privilege
  │
  ▼
Interesting backup?
  │
  ├── YES → Inspect for useful configuration/secrets
  │
  ▼
SUID / SGID?
  │
  ├── YES → Investigate privilege mechanism
  │
  ▼
Interesting mount?
  │
  ├── YES → Investigate filesystem/trust relationship
  │
  ▼
CONTINUE
```

---

## 🧪 Validation Pattern

Whenever you find something interesting, stop and validate.

Use this model:

```text id="6z3k8a"
FINDING
   ↓
OWNERSHIP
   ↓
PERMISSIONS
   ↓
EXECUTION CONTEXT
   ↓
PRIVILEGED CONSUMER
   ↓
USER INFLUENCE
   ↓
ACTUAL PRIVILEGE IMPACT
```

For example:

```text id="h0o6j5"
Writable script
      ↓
Root-owned
      ↓
Root executes it
      ↓
Current user can modify it
      ↓
Modification affects execution
      ↓
Privilege boundary exists
```

That is a meaningful escalation path.

---

## ⚠️ Common Mistakes

### 1. Treating every writable file as interesting

Most writable files are irrelevant.

Always ask:

> **Who trusts this file?**

---

### 2. Forgetting parent directories

Always inspect:

```bash id="k2x9qt"
namei -l /path/to/file
```

A path is a chain of permissions.

---

### 3. Searching without prioritizing

Huge recursive searches create noise.

Prefer:

```text id="n3q2gn"
Context
 ↓
Targeted search
 ↓
Interpretation
 ↓
Investigation
```

---

### 4. Modifying a file before understanding it

Never blindly modify an interesting privileged file.

First establish:

```text
Owner
Permissions
Executor
Execution frequency
Dependencies
Privilege context
```

Then validate.

---

### 5. Confusing ownership with execution privilege

A root-owned file is not necessarily executed as root.

Likewise, a user-owned file can become relevant if a privileged process trusts it.

Always determine the **execution context**.

---

## ⚡ Filesystem Enumeration — Command Reference

```bash id="v9qjzq"
# Current location
pwd
ls -la

# File metadata
stat /path/to/file
file /path/to/file

# Path permissions
namei -l /path/to/file

# Writable files
find / -type f -writable 2>/dev/null

# Writable directories
find / -type d -writable 2>/dev/null

# World-writable directories
find / -type d -perm -0002 -print 2>/dev/null

# World-writable files
find / -type f -perm -0002 -print 2>/dev/null

# Scripts
find / -type f \( -name "*.sh" -o -name "*.py" -o -name "*.pl" -o -name "*.rb" \) 2>/dev/null

# Configuration
find /etc /opt /usr/local/etc -type f \( -name "*.conf" -o -name "*.cfg" -o -name "*.ini" \) 2>/dev/null

# Application configuration
find /opt /var/www /home -type f \( -name ".env" -o -name "*config*" -o -name "*credentials*" -o -name "*secret*" \) 2>/dev/null

# Backups
find /etc /opt /var/www /home -type f \( -name "*.bak" -o -name "*.backup" -o -name "*.old" -o -name "*.save" -o -name "*~" \) 2>/dev/null

# Recent files
find / -type f -mtime -7 2>/dev/null

# Root-owned files
find /opt /usr/local /var/www -user root -type f 2>/dev/null

# SUID
find / -perm -4000 -type f 2>/dev/null

# SGID
find / -perm -2000 -type f 2>/dev/null

# Custom executables
find /opt /usr/local/bin /usr/local/sbin -type f -executable 2>/dev/null

# Symlinks
readlink -f /path/to/file

# ACLs
getfacl /path/to/file
getfacl /path/to/directory

# Mounts
findmnt
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS
findmnt -t nfs,nfs4

# Temporary locations
ls -ld /tmp /var/tmp /dev/shm
ls -la /tmp
```

---

## 🧠 Remember

The filesystem is not just a collection of files.

Think of it as a network of **trust relationships**:

```text
Privileged Process
       │
       ├── executable
       ├── script
       ├── configuration
       ├── library
       ├── directory
       ├── temporary file
       └── dependency
              │
              ▼
       Can I influence it?
              │
              ▼
       Does that influence
       affect privileged execution?
```

The strongest filesystem findings usually come from identifying this relationship.

---

## 🚀 Next Stage

Once filesystem enumeration is complete, move to:

```text
04-PROCESS-SERVICE-ENUMERATION/
```

The next phase changes the perspective.

Instead of starting with:

> **"What files can I modify?"**

we start with:

> **"What privileged processes and services are running, what do they execute, and what do they trust?"**
