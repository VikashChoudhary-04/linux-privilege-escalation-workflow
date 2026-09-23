# 09 — Execution Abuse

Execution abuse occurs when a privileged process trusts something that a lower-privileged user can influence.

The vulnerable component may be:

* A command
* A script
* A binary
* A directory
* A library
* An environment variable
* A wildcard expansion
* A shell function
* A command substitution
* A configuration value

The key idea is:

> **A privileged process is only as secure as the things it trusts and the things an unprivileged user can control.**

---

## Core Question

> **Does a privileged process execute something that I can influence?**

Do not start with:

```text
"What exploit can I run?"
```

Start with:

```text
"What does the privileged process trust?"
        ↓
"Can I modify or influence it?"
        ↓
"Will the privileged process use my modification?"
        ↓
"Does that execution cross the privilege boundary?"
```

---

## Mental Model

Use this chain:

```text id="2k9w4e"
PRIVILEGED PROCESS
        ↓
WHAT DOES IT EXECUTE?
        ↓
WHAT DOES THAT EXECUTABLE TRUST?
        ↓
CAN I CONTROL THAT TRUSTED COMPONENT?
        ↓
WHEN IS IT EXECUTED?
        ↓
UNDER WHICH USER?
        ↓
CAN MY CONTROL CHANGE THE RESULT?
        ↓
VALIDATE PRIVILEGE BOUNDARY
```

The most important relationship is:

```text id="m0p4e7"
PRIVILEGED EXECUTION
        +
UNPRIVILEGED CONTROL
        =
POSSIBLE EXECUTION ABUSE
```

---

## 1. Identify Privileged Execution

Before investigating execution abuse, identify processes that run with higher privileges.

### 🔎 ENUMERATE

List processes:

```bash id="x5y3f7"
ps aux
```

Show users and commands:

```bash id="1r6v8k"
ps -eo user,pid,ppid,args
```

Show root-owned processes:

```bash id="j8x0qa"
ps -eo user,pid,ppid,args | awk '$1 == "root"'
```

Show process hierarchy:

```bash id="6z2c1m"
ps -eo pid,ppid,user,args --forest
```

### 🧠 WHY

You are looking for:

```text
root process
    ↓
script
    ↓
command
    ↓
file/config/dependency
```

That chain gives you something to investigate.

---

## 2. Identify the Executable

Once an interesting process is found, determine exactly what binary is running.

### 🔎 ENUMERATE

For a known PID:

```bash id="q8j3wx"
readlink -f /proc/<PID>/exe
```

Check its working directory:

```bash id="a9w1cz"
readlink -f /proc/<PID>/cwd
```

Check the command line:

```bash id="6q2r9s"
tr '\0' ' ' < /proc/<PID>/cmdline
```

Check open files:

```bash id="0b7rta"
lsof -p <PID>
```

### 🧠 WHY

The visible process name may not tell you:

* Where the binary lives
* Which script launched it
* Which directory it uses
* Which files it opens
* Which commands it executes

### ➡️ NEXT STEP

Build:

```text id="0y4l2f"
Process
   ↓
Executable
   ↓
Arguments
   ↓
Working directory
   ↓
Configuration
   ↓
Dependencies
```

---

## 3. Inspect File Ownership and Permissions

Once you identify an executable, inspect it.

### 🔎 ENUMERATE

```bash id="1t4v6a"
ls -la /path/to/executable
```

```bash id="k5j9w1"
stat /path/to/executable
```

```bash id="6m4p8x"
namei -l /path/to/executable
```

### 🧠 WHY

You need to know:

```text id="q3a5z8"
Who owns it?
Can I modify it?
Can I replace it?
Can I modify its parent directory?
Can I influence its dependencies?
```

### ⚠️ LOOK FOR

High-interest relationship:

```text id="kq2v4j"
Privileged process
       ↓
Root-owned executable
       ↓
Unprivileged user can modify parent directory
```

The file itself does not necessarily need to be writable.

A writable parent directory can sometimes be equally important.

---

## 4. PATH Hijacking

PATH hijacking occurs when a privileged program executes a command without specifying an absolute path and the command lookup can be influenced.

### 🧠 CONCEPT

Suppose a privileged script contains:

```bash id="7x1w5k"
backup
```

instead of:

```bash id="f8v2zq"
/usr/bin/backup
```

The shell may search directories listed in `PATH`.

The question becomes:

```text id="h8p1y6"
Which PATH is used?
        ↓
Which directory is searched first?
        ↓
Can I write to that directory?
        ↓
What command is being resolved?
```

### 🔎 ENUMERATE

Check your PATH:

```bash id="d9f5j2"
echo "$PATH"
```

Split it into directories:

```bash id="1b7x8p"
echo "$PATH" | tr ':' '\n'
```

Inspect each directory:

```bash id="r4n6yt"
for dir in $(echo "$PATH" | tr ':' ' '); do
    ls -ld "$dir" 2>/dev/null
done
```

Look for command resolution:

```bash id="9z3c6k"
command -v backup
```

```bash id="u8p5l1"
type -a backup
```

### ⚠️ LOOK FOR

A potential pattern:

```text id="4m7k9c"
Privileged script
       ↓
Runs "backup"
       ↓
No absolute path
       ↓
PATH-controlled lookup
       ↓
Writable search directory
```

### 🧪 VALIDATE

Do not immediately modify a privileged system.

First determine:

```text id="b3j6w0"
What command is expected?
What executable would normally run?
What PATH is used by the privileged process?
Can the relevant directory be influenced?
Does the privileged process actually reach the command?
```

A safe validation can use a harmless test executable in an isolated authorized lab.

---

## 5. PATH vs Shell Environment

A common mistake is assuming that your current PATH is automatically the PATH used by a privileged process.

It may not be.

### 🧠 CONCEPT

The execution chain may be:

```text id="7e3q9n"
Your shell PATH
      ≠
Sudo PATH
      ≠
Cron PATH
      ≠
Systemd environment
```

### 🔎 ENUMERATE

For sudo:

```bash id="w7j4b2"
sudo -l
```

For a script:

```bash id="2q5c8h"
sed -n '1,240p' /path/to/script
```

For systemd:

```bash id="8v6x0m"
systemctl cat <service>
```

For cron:

```bash id="4p8n1k"
cat /etc/crontab
```

### 💡 REMEMBER

> **Always analyze the environment of the privileged execution context, not only your own shell.**

---

## 6. Writable Scripts

A privileged script that an unprivileged user can modify is a high-value finding.

### 🔎 ENUMERATE

Find scripts:

```bash id="v5f2w8"
find /etc /opt /usr/local /var/www -type f \
\( -name "*.sh" -o -name "*.py" -o -name "*.pl" -o -name "*.rb" \) \
2>/dev/null
```

Inspect ownership:

```bash id="p6n3x9"
ls -la /path/to/script
```

```bash id="m7q1s4"
stat /path/to/script
```

Inspect contents:

```bash id="a4z8k2"
sed -n '1,240p' /path/to/script
```

### 🧠 WHY

You need all three conditions:

```text id="1w8m4c"
Privileged execution
        +
User can modify script
        +
Script is actually executed
```

Only then does the finding become a likely privilege-escalation path.

### ⚠️ LOOK FOR

Examples:

```text id="5s8x1n"
root executes /opt/backup.sh
        ↓
backup.sh writable by current user
```

or:

```text id="r7c2p4"
root executes script
        ↓
script sources writable configuration
```

---

## 7. Writable Parent Directories

Do not check only the target file.

Check every directory in its path.

### 🔎 ENUMERATE

```bash id="8k3z6d"
namei -l /path/to/script
```

For each directory:

```bash id="4x9p2m"
ls -ld /path
```

### 🧠 WHY

Consider:

```text id="1c7v4q"
/opt/tools/backup.sh
```

Even if:

```text id="0f8d3m"
backup.sh → root-owned, not writable
```

a writable:

```text id="3y6j1a"
/opt/tools/
```

may create a different security relationship.

The important question is:

> **Can the current user influence the object that the privileged process eventually executes?**

---

## 8. Command Substitution

Shell command substitution executes another command and inserts its output.

Examples include:

```bash id="5u8z3q"
$(command)
```

and:

```bash id="7m1r5x"
`command`
```

### 🧠 CONCEPT

If a privileged script contains:

```bash id="d4p6q2"
echo "$(hostname)"
```

then `hostname` is another command being executed.

The investigation becomes:

```text id="z6y3k9"
Privileged script
      ↓
Command substitution
      ↓
Command
      ↓
How is command resolved?
      ↓
Can execution be influenced?
```

### 🔎 ENUMERATE

Search scripts:

```bash id="x3c7n8"
grep -RniE '\$\([^)]*\)|`[^`]+`' \
/etc /opt /usr/local /var/www 2>/dev/null
```

Inspect suspicious scripts manually.

### ⚠️ LOOK FOR

Focus on:

* Commands executed by root
* Commands resolved through PATH
* User-controlled variables
* User-controlled files
* Commands using relative paths

---

## 9. Wildcard Abuse

Wildcards can become dangerous when privileged commands pass expanded filenames to utilities that interpret certain arguments as options.

### 🧠 CONCEPT

A command such as:

```bash id="f0m3v7"
tar -cf backup.tar *
```

causes the shell to expand `*` before `tar` receives the arguments.

The resulting argument list may depend on files present in the directory.

The investigation is:

```text id="9x4c2p"
Privileged command
      ↓
Wildcard
      ↓
Shell expands filenames
      ↓
Program receives generated arguments
      ↓
Can attacker-controlled filenames alter behavior?
```

### 🔎 ENUMERATE

Search scripts for wildcards:

```bash id="v1n8k5"
grep -RniE '(^|[[:space:]])(\*|\?|\[[^]]+\])([[:space:]]|$)' \
/etc /opt /usr/local 2>/dev/null
```

Then inspect the full command.

### ⚠️ LOOK FOR

Pay attention to privileged use of commands such as:

* `tar`
* `cp`
* `rsync`
* `chmod`
* `chown`
* `find`
* Other utilities where generated filenames may be interpreted as options

The presence of `*` alone does **not** prove vulnerability.

### 🧪 VALIDATE

Determine:

```text id="k7x1b6"
What command receives the expanded arguments?
Can I create files in the relevant directory?
How does the target utility parse those arguments?
Does the resulting behavior cross the privilege boundary?
```

---

## 10. Library Loading

Programs may load shared libraries at runtime.

### 🔎 ENUMERATE

Inspect an ELF executable:

```bash id="6c4w9p"
file /path/to/binary
```

Inspect dynamic dependencies:

```bash id="3m8x7q"
ldd /path/to/binary
```

Inspect ELF dynamic information:

```bash id="y6n2v8"
readelf -d /path/to/binary
```

Search for RPATH/RUNPATH:

```bash id="9f1c4a"
readelf -d /path/to/binary | grep -Ei 'rpath|runpath'
```

### 🧠 WHY

You are looking for relationships such as:

```text id="q8w3m6"
Privileged binary
       ↓
Loads library
       ↓
Library search path
       ↓
Writable location
```

### ⚠️ LOOK FOR

Potentially interesting indicators:

```text id="s2p5j8"
RPATH
RUNPATH
relative library path
writable library directory
unexpected library location
```

### 🧪 VALIDATE

Confirm:

```text id="1n7c4b"
Which library is actually loaded?
Where is it loaded from?
Can the current user modify or replace it?
Does the privileged process load it?
```

Do not replace system libraries on a real system during validation.

Use an authorized lab environment.

---

## 11. Environment Abuse

Environment variables can influence application behavior.

### 🔎 ENUMERATE

Inspect the current environment:

```bash id="m9x4q2"
env | sort
```

Inspect privileged service configuration:

```bash id="h5k7p1"
systemctl show <service>
```

Look for environment-related configuration:

```bash id="t2w8c6"
systemctl cat <service>
```

### 🧠 WHY

Potentially relevant variables depend on the application.

Examples can include:

```text id="3r6m1z"
PATH
LD_LIBRARY_PATH
PYTHONPATH
PERL5LIB
RUBYLIB
```

Not every variable is honored by every privileged execution mechanism.

### ⚠️ LOOK FOR

The important relationship is:

```text id="8k2d7v"
Privileged application
       ↓
Environment-controlled behavior
       ↓
Unprivileged user can influence environment
       ↓
Application trusts that influence
```

### ❌ COMMON MISTAKE

Do not assume:

```text id="0q6s1p"
Environment variable exists
        =
Environment variable is exploitable
```

You must establish that the privileged application actually uses it.

---

## 12. Shell Function Abuse

Shell functions can influence command behavior within a shell environment.

### 🔎 ENUMERATE

List functions in the current shell:

```bash id="e5n8r3"
declare -F
```

Inspect a specific function:

```bash id="u4j7c1"
declare -f function_name
```

Check exported functions where supported:

```bash id="7b2x9m"
export -p
```

### 🧠 WHY

A function may affect how a shell resolves a command.

However, privileged execution mechanisms commonly sanitize or restrict environments.

### ⚠️ LOOK FOR

The important questions are:

```text id="m3k8q1"
Is a shell actually involved?
Which shell?
Does it inherit the relevant environment?
Can the current user influence that environment?
```

Do not treat shell functions as a universal privilege-escalation mechanism.

---

## 13. Relative Paths

Relative paths can create trust problems.

### 🧠 CONCEPT

Compare:

```bash id="x4k7z2"
./backup.sh
```

with:

```bash id="m8c1v5"
/opt/tools/backup.sh
```

A relative path depends on the current working directory.

### 🔎 ENUMERATE

Inspect the privileged process:

```bash id="e7p2q9"
readlink -f /proc/<PID>/cwd
```

Inspect service configuration:

```bash id="4n6s8w"
systemctl cat <service>
```

Inspect scripts:

```bash id="z1c5r7"
grep -nE '(^|[[:space:]])\./|(^|[[:space:]])[^/[:space:]]+' /path/to/script
```

### ⚠️ LOOK FOR

Potential relationship:

```text id="3k9v6x"
Privileged execution
       ↓
Relative executable
       ↓
Predictable working directory
       ↓
User can influence working directory or referenced file
```

### 🧪 VALIDATE

Determine exactly:

```text id="q2m7c4"
What is the working directory?
What file is resolved?
Who controls that location?
When does execution occur?
```

---

## 14. Sourced Files

Shell scripts can import another file using commands such as:

```bash id="r4k6p9"
source file
```

or:

```bash id="w3n8x2"
. file
```

### 🔎 ENUMERATE

Search scripts:

```bash id="2c7m5v"
grep -RniE '(^|[[:space:]])(source|\.)[[:space:]]+' \
/etc /opt /usr/local 2>/dev/null
```

Inspect the referenced file:

```bash id="9q1x6d"
ls -la /path/to/file
```

```bash id="7m4k2p"
stat /path/to/file
```

### 🧠 WHY

A privileged script may be secure itself but source a configuration or helper file that an unprivileged user can modify.

The chain becomes:

```text id="b6r2z8"
Root script
    ↓
source config.sh
    ↓
config.sh writable
    ↓
Root executes attacker-controlled content
```

This is a high-value relationship to investigate.

---

## 15. Temporary Files

Privileged applications sometimes create temporary files.

### 🔎 ENUMERATE

Inspect temporary directories:

```bash id="f7k2m4"
ls -la /tmp
```

```bash id="p3x8v1"
ls -la /var/tmp
```

```bash id="n6q4c9"
ls -la /dev/shm
```

Find recently modified files:

```bash id="2w9s5j"
find /tmp /var/tmp /dev/shm -type f -mmin -30 2>/dev/null
```

### 🧠 WHY

Investigate whether a privileged process:

* Creates predictable temporary files
* Reads temporary files
* Executes temporary files
* Uses predictable paths
* Follows attacker-controlled symlinks

### ⚠️ LOOK FOR

The important relationship is:

```text id="k4m8s2"
Privileged process
       ↓
Predictable temporary file
       ↓
Unprivileged user can influence it
       ↓
Privileged process trusts it
```

Do not modify live system files during testing unless the environment explicitly authorizes it.

---

## 16. Execution Chain Analysis

When investigating an execution finding, map the complete chain.

### 🧠 CONCEPT

Use:

```text id="3q7m1k"
WHO
 ↓
WHAT
 ↓
WHERE
 ↓
HOW
 ↓
WHEN
 ↓
WHAT DOES IT TRUST?
 ↓
CAN I CONTROL IT?
 ↓
WHAT PRIVILEGE RESULTS?
```

For example:

```text id="w5r8n2"
root
 ↓
/opt/scripts/backup.sh
 ↓
/opt/scripts/
 ↓
cron
 ↓
every 5 minutes
 ↓
backup.sh calls "tar"
 ↓
PATH + wildcard
 ↓
current user controls execution directory
 ↓
potential privilege boundary
```

This is much more useful than simply recording:

```text
"cron vulnerability found"
```

---

## 17. Finding vs Vulnerability vs Exploitable Path

### Finding

```text id="p2v6m8"
A root-owned script uses a relative command.
```

This is an observation.

### Vulnerability

```text id="h7c1x4"
The relative command resolves through a location
that the unprivileged user can influence.
```

Now there is a security weakness.

### Exploitable Path

```text id="m9w3k7"
The privileged process actually executes the influenced command,
and the resulting execution crosses the privilege boundary.
```

That is a validated escalation path.

---

## 18. Validation Before Exploitation

Before attempting exploitation, answer:

```text id="s5n8q2"
[ ] What process is privileged?
[ ] What exactly does it execute?
[ ] Which user controls the target?
[ ] Which directories are involved?
[ ] Which environment is used?
[ ] When does execution occur?
[ ] Can I influence the trusted component?
[ ] Does the process actually use my controlled component?
[ ] What privilege will result?
```

### 🧪 VALIDATE

Use harmless validation whenever possible.

Examples:

```text id="x8q3m5"
Confirm command resolution
Confirm file ownership
Confirm directory permissions
Confirm service configuration
Confirm execution timing
Confirm process identity
Confirm dependency resolution
```

The goal is to prove the relationship before attempting a privileged action.

---

## 19. Execution Abuse Decision Tree

```text id="8q3m7c"
                 PRIVILEGED PROCESS
                        │
                        ▼
                 WHAT DOES IT RUN?
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       SCRIPT         BINARY        COMMAND
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                WHAT DOES IT TRUST?
                        │
        ┌───────────────┼────────────────┐
        ▼               ▼                ▼
       PATH          FILE/DIR          LIBRARY
        │               │                │
        ▼               ▼                ▼
     Writable?       Writable?       Writable?
        │               │                │
       YES             YES              YES
        │               │                │
        └───────────────┼────────────────┘
                        ▼
                 VALIDATE CONTROL
                        │
                        ▼
                 DOES EXECUTION
                 CROSS BOUNDARY?
                    │       │
                   NO      YES
                    │       │
                    ▼       ▼
                 MOVE ON   VERIFY
```

---

## 20. High-Value Patterns

### 🔎 Pattern 1 — Root Script + Writable Script

```text id="6j4m8x"
root executes script
        +
current user can modify script
        =
high-priority validation target
```

---

### 🔎 Pattern 2 — Root Script + Writable Parent Directory

```text id="7c2n5p"
root executes /opt/tools/task
        +
current user controls /opt/tools/
        =
investigate execution path
```

---

### 🔎 Pattern 3 — Root Command + PATH Lookup

```text id="9v1k4m"
root script
     ↓
calls command without absolute path
     ↓
PATH resolution
     ↓
user-controlled search location
```

---

### 🔎 Pattern 4 — Root Binary + Writable Library Location

```text id="2m6x8q"
root binary
     ↓
loads shared library
     ↓
library resolved from writable location
```

---

### 🔎 Pattern 5 — Root Script + Writable Sourced File

```text id="5p9r3w"
root script
     ↓
source helper/config
     ↓
helper/config writable
```

---

### 🔎 Pattern 6 — Root Scheduled Command + Wildcard

```text id="4x7n2c"
scheduled root command
        ↓
wildcard expansion
        ↓
attacker-controlled filenames
        ↓
unexpected argument interpretation
```

---

## 21. Common Mistakes

### ❌ Mistake 1 — Finding a Writable File and Stopping

A writable file matters only if a privileged process trusts or executes it.

### ❌ Mistake 2 — Assuming Root-Owned Means Vulnerable

```text id="v3k8m1"
root-owned file
```

does not automatically mean:

```text id="x7q2p5"
privilege escalation
```

You need a user-controlled relationship.

### ❌ Mistake 3 — Assuming PATH Is Universal

Different execution mechanisms may use different environments.

### ❌ Mistake 4 — Treating Wildcards as Automatically Vulnerable

The specific utility and argument parsing matter.

### ❌ Mistake 5 — Ignoring Parent Directories

Always inspect the entire path:

```bash id="1k8m4q"
namei -l /path/to/file
```

### ❌ Mistake 6 — Ignoring Execution Timing

A vulnerable script that never executes is not a practical escalation path.

### ❌ Mistake 7 — Exploiting Before Understanding

First establish:

```text id="q6r9c3"
WHAT
WHO
WHERE
WHEN
HOW
WHY
```

Then validate.

---

## 22. Five-Minute Execution Abuse Check

### 🔎 ENUMERATE

```bash id="m3k7x1"
ps -eo user,pid,ppid,args | awk '$1 == "root"'
```

```bash id="w8q2c6"
systemctl --type=service --state=running
```

```bash id="j5n9r4"
systemctl list-timers --all
```

```bash id="p7x1v3"
cat /etc/crontab
```

```bash id="z4m8q2"
find /etc/cron.d -type f -maxdepth 1 -print 2>/dev/null
```

Inspect scripts discovered during enumeration:

```bash id="c6r2n8"
ls -la /path/to/script
```

```bash id="y9w3k5"
namei -l /path/to/script
```

```bash id="f1q7m4"
sed -n '1,240p' /path/to/script
```

Search for common execution-abuse indicators:

```bash id="2n6x8p"
grep -nE '\$PATH|source |^\s*\.\s|(^|[[:space:]])\./|\*' \
/path/to/script 2>/dev/null
```

Then ask:

```text id="8k4m1q"
Privileged execution found?
        │
       YES
        │
        ▼
What does it execute?
        │
        ▼
What does it trust?
        │
        ▼
Can I influence it?
        │
        ▼
Does that influence cross the privilege boundary?
```

---

## 23. Quick Command Reference

| Goal                      | Command                                                      |
| ------------------------- | ------------------------------------------------------------ |
| All processes             | `ps aux`                                                     |
| Root processes            | `ps -eo user,pid,ppid,args \| awk '$1 == "root"'`            |
| Process executable        | `readlink -f /proc/<PID>/exe`                                |
| Process working directory | `readlink -f /proc/<PID>/cwd`                                |
| Process command line      | `tr '\0' ' ' < /proc/<PID>/cmdline`                          |
| Process open files        | `lsof -p <PID>`                                              |
| Service configuration     | `systemctl cat <service>`                                    |
| Running services          | `systemctl --type=service --state=running`                   |
| Timers                    | `systemctl list-timers --all`                                |
| Cron configuration        | `cat /etc/crontab`                                           |
| Cron directory            | `ls -la /etc/cron.d/`                                        |
| PATH                      | `echo "$PATH"`                                               |
| Split PATH                | `echo "$PATH" \| tr ':' '\n'`                                |
| Command resolution        | `command -v <command>`                                       |
| All command locations     | `type -a <command>`                                          |
| File permissions          | `stat /path/to/file`                                         |
| Path permissions          | `namei -l /path/to/file`                                     |
| ELF dependencies          | `ldd /path/to/binary`                                        |
| ELF dynamic info          | `readelf -d /path/to/binary`                                 |
| Find scripts              | `find /etc /opt /usr/local -type f -name "*.sh" 2>/dev/null` |
| Temporary files           | `find /tmp /var/tmp /dev/shm -type f 2>/dev/null`            |

---

## 24. Complete Checklist

### 🔎 Privileged Execution

```text id="6w2k9p"
[ ] Identify root processes
[ ] Identify root services
[ ] Identify root scheduled tasks
[ ] Identify privileged scripts
[ ] Identify privileged binaries
```

### 🔎 Trust Relationships

```text id="1r5m8c"
[ ] Check executable ownership
[ ] Check executable permissions
[ ] Check parent-directory permissions
[ ] Check configuration files
[ ] Check sourced files
[ ] Check PATH usage
[ ] Check relative paths
[ ] Check wildcard usage
[ ] Check library loading
[ ] Check environment dependencies
[ ] Check temporary files
```

### 🧪 Validation

```text id="7x4q1m"
[ ] Confirm privileged execution
[ ] Confirm trusted component
[ ] Confirm user control
[ ] Confirm execution occurs
[ ] Confirm resulting behavior
[ ] Confirm privilege boundary
[ ] Record evidence
```

---

## 💡 Remember

Execution abuse is fundamentally about **trust**.

Do not memorize:

```text
"PATH trick"
"wildcard trick"
"library trick"
```

Instead memorize:

```text
WHAT runs with privilege?
        ↓
WHAT does it trust?
        ↓
WHO controls that trusted component?
        ↓
WHEN is it used?
        ↓
CAN I influence the result?
        ↓
DOES IT CROSS THE PRIVILEGE BOUNDARY?
```

The most important pattern is:

> **Privileged execution + unprivileged control + trusted relationship = investigate.**

---

## Where Findings Lead

Execution-abuse findings commonly connect to:

```text id="5m8q2x"
PATH
 ↓
Command Resolution
 ↓
09-EXECUTION-ABUSE
```

```text id="n4c7p1"
Wildcard
 ↓
Argument Interpretation
 ↓
09-EXECUTION-ABUSE
```

```text id="r6w2k9"
Library
 ↓
Dynamic Loading
 ↓
09-EXECUTION-ABUSE
```

```text id="x3m8q5"
Scheduled Task
 ↓
Privileged Execution
 ↓
07-SCHEDULED-EXECUTION
```

```text id="j1v6c4"
Privileged Service
 ↓
Trusted Component
 ↓
04-PROCESS-SERVICE-ENUMERATION
```

```text id="z8q3m7"
Finding
 ↓
Validation
 ↓
13-FINDING-VALIDATION
 ↓
Authorized Exploitation
 ↓
14-EXPLOITATION
```

---

## Final Mental Model

```text id="4k7m2p"
              PRIVILEGED PROCESS
                      │
                      ▼
                WHAT DOES IT RUN?
                      │
                      ▼
                WHAT DOES IT TRUST?
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
      PATH           FILE          LIBRARY
       │              │              │
       ▼              ▼              ▼
    Writable?      Writable?      Writable?
       │              │              │
       └──────────────┼──────────────┘
                      ▼
               CAN I CONTROL IT?
                      │
                      ▼
               WILL IT BE USED?
                      │
                      ▼
               WHAT PRIVILEGE?
                      │
                      ▼
             BOUNDARY CROSSED?
                  │        │
                 NO       YES
                  │        │
                  ▼        ▼
               MOVE ON   VERIFY
```

> **Don't ask only what is running as root. Ask what that root process trusts.**
