# 07 — Scheduled Execution

> **Goal:** Identify commands, scripts, and services that execute automatically on a schedule, determine what privileges they run with, and validate whether the current user can influence anything trusted by that scheduled execution.

## 🎯 Core Question

> **What executes automatically, with what privileges, and can I influence what it executes?**

Scheduled execution is an important Linux privilege-escalation area because a low-privileged user may not need to execute a privileged command directly.

Instead, the system may execute it automatically:

```text
Low-privileged user
        ↓
Influences trusted file/script/configuration
        ↓
Scheduled task executes automatically
        ↓
Task runs as root or another privileged account
        ↓
Privilege boundary is crossed
```

The key relationship is:

```text
AUTOMATIC EXECUTION
        +
PRIVILEGED CONTEXT
        +
USER INFLUENCE
        =
POTENTIAL ESCALATION PATH
```

## 🧠 The Mental Model

For every scheduled task, ask:

```text
WHAT runs?
    ↓
WHEN does it run?
    ↓
WHO runs it?
    ↓
WHERE is the script/binary?
    ↓
WHAT files does it trust?
    ↓
WHAT commands does it execute?
    ↓
CAN I modify any trusted component?
    ↓
DOES that influence cross the privilege boundary?
```

Do not focus only on finding cron jobs.

Scheduled execution can come from:

```text
cron
systemd timers
at
application schedulers
custom daemons
service managers
scripts launched by other scheduled tasks
```

## 1. Identify Cron

Start by checking whether cron is running.

### 🔎 ENUMERATE

```bash
ps aux | grep -E '[c]ron|[c]rond'
```

Or:

```bash
pgrep -a cron
```

And:

```bash
pgrep -a crond
```

### WHY

This tells you whether a cron daemon is currently active.

### LOOK FOR

Examples:

```text
/usr/sbin/cron
/usr/sbin/crond
```

If cron is present, continue with cron enumeration.

### ⚠️ IMPORTANT

A missing cron daemon does not mean scheduled execution is impossible.

Continue checking other mechanisms such as systemd timers.

## 2. Understand Cron Locations

Common cron locations include:

```text
/etc/crontab
/etc/cron.d/
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
/var/spool/cron/
/var/spool/cron/crontabs/
```

The exact locations vary by distribution.

## 3. Inspect `/etc/crontab`

### 🔎 ENUMERATE

```bash
cat /etc/crontab
```

### WHY

Unlike a user's personal crontab, `/etc/crontab` includes a user field.

Example:

```text
17 * * * * root cd / && run-parts --report /etc/cron.hourly
```

The important information is:

```text
schedule
user
command
```

### LOOK FOR

Pay particular attention to:

```text
root
custom scripts
/opt/
usr/local/
home/
temporary directories
relative commands
```

## 4. Understand Cron Syntax

A standard cron entry has:

```text
minute hour day-of-month month day-of-week user command
```

Example:

```text
0 * * * * root /opt/scripts/backup.sh
```

means:

```text
Every hour
    ↓
run /opt/scripts/backup.sh
    ↓
as root
```

### 🧠 REMEMBER

For privilege escalation, the important fields are:

```text
WHO
+
WHAT
+
WHEN
```

## 5. Inspect `/etc/cron.d/`

### 🔎 ENUMERATE

```bash
ls -la /etc/cron.d/
```

Then inspect readable files:

```bash
cat /etc/cron.d/*
```

If there are many files:

```bash
find /etc/cron.d -type f -maxdepth 1 -print 2>/dev/null
```

### WHY

Custom scheduled tasks are often placed in `/etc/cron.d/`.

### LOOK FOR

Example:

```text
*/5 * * * * root /opt/scripts/maintenance.sh
```

Now you have:

```text
root
  ↓
/opt/scripts/maintenance.sh
```

The next step is permission analysis.

## 6. Inspect Cron Scripts

Suppose you find:

```text
*/5 * * * * root /opt/scripts/maintenance.sh
```

Run:

```bash
ls -la /opt/scripts/maintenance.sh
```

Then:

```bash
stat /opt/scripts/maintenance.sh
```

Then:

```bash
namei -l /opt/scripts/maintenance.sh
```

### WHY

You need to determine:

* Who owns the script?
* Who can write it?
* Who can modify its parent directories?
* Is it executable?
* Is any part of the path user-controlled?

## 7. Check Whether the Scheduled Script Is Writable

### 🔎 ENUMERATE

```bash
test -w /opt/scripts/maintenance.sh && echo "Writable" || echo "Not writable"
```

Or:

```bash
ls -la /opt/scripts/maintenance.sh
```

### HIGH-INTEREST PATTERN

```text
root cron job
      ↓
/opt/scripts/maintenance.sh
      ↓
current user can modify script
```

This is a strong candidate for validation.

### 🧪 VALIDATE

Before changing anything, establish:

```text
Who runs the script?
How often?
Can the current user modify it?
Does cron actually execute it?
What does the script do?
```

## 8. Check Parent Directory Permissions

A script itself may not be writable.

For example:

```text
-rwxr-xr-x root root maintenance.sh
```

That does not end the investigation.

Run:

```bash
namei -l /opt/scripts/maintenance.sh
```

### WHY

You need to inspect every directory in the path.

Potential pattern:

```text
/opt
  ↓
/opt/scripts
  ↓
maintenance.sh
```

If an unprivileged user can modify a relevant parent directory, the analysis changes.

## 9. Enumerate System Cron Directories

### 🔎 ENUMERATE

```bash
ls -la /etc/cron.hourly/
ls -la /etc/cron.daily/
ls -la /etc/cron.weekly/
ls -la /etc/cron.monthly/
```

### WHY

These directories can contain scripts executed automatically by the system.

### LOOK FOR

* Custom scripts
* Unusual filenames
* Root-owned scripts
* Scripts in unusual locations
* Writable files
* Writable parent directories

## 10. Search Cron-Related Files

A broad search:

```bash
find /etc -type f \( -name '*cron*' -o -path '/etc/cron.d/*' \) 2>/dev/null
```

### WHY

This can identify additional cron configuration files.

Do not assume every result is a scheduled task. Inspect the contents.

## 11. Enumerate User Crontabs

Depending on permissions, inspect the cron spool.

### 🔎 ENUMERATE

```bash
ls -la /var/spool/cron/ 2>/dev/null
```

And:

```bash
ls -la /var/spool/cron/crontabs/ 2>/dev/null
```

### WHY

User-specific cron jobs may reveal:

* Application maintenance
* Backup scripts
* Automation
* Privileged jobs
* Interesting command paths

### ⚠️ IMPORTANT

Do not assume you can read every user's crontab.

Access restrictions are normal.

## 12. Check Your Own Crontab

### 🔎 ENUMERATE

```bash
crontab -l
```

### WHY

Your own crontab can reveal:

* Existing scheduled tasks
* Scripts you control
* Environment assumptions
* Potential paths that interact with privileged services

This is mostly contextual unless the task interacts with a higher-privileged process.

## 13. Check Root's Crontab

If you have legitimate permission to inspect it:

```bash
sudo crontab -l
```

### WHY

A root crontab can reveal scheduled privileged commands.

If `sudo` access does not permit this, do not attempt to bypass the restriction.

The goal is enumeration within your authorized privileges.

## 14. Inspect Cron Scripts for Relative Commands

Suppose:

```text
root cron
    ↓
/opt/scripts/backup.sh
```

and inside:

```bash
tar -czf /backup/data.tar.gz /var/www
```

The command:

```text
tar
```

is not an absolute path.

### 🔎 ENUMERATE

```bash
command -v tar
```

Then inspect the script:

```bash
sed -n '1,240p' /opt/scripts/backup.sh
```

### WHY

You need to understand how commands are resolved.

### LOOK FOR

```text
tar
cp
mv
rsync
python
perl
bash
custom-tool
```

Then ask:

```text
What PATH does cron provide?
Can the command resolution be influenced?
```

## 15. Understand Cron Environment

Cron does not necessarily use the same interactive environment as your shell.

A cron job may have a limited `PATH`.

For example:

```text
PATH=/usr/bin:/bin
```

### WHY

This matters when a scheduled script executes commands without absolute paths.

The question is not:

> "Does the script use `tar`?"

The question is:

> **"Which `tar` will the privileged scheduled process actually execute?"**

## 16. Inspect Cron's PATH

Check the system cron configuration:

```bash
grep -E '^PATH=' /etc/crontab 2>/dev/null
```

Also inspect relevant cron configuration:

```bash
grep -RniE '^PATH=' /etc/cron* 2>/dev/null
```

### ⚠️ IMPORTANT

Do not assume the output represents every possible cron implementation.

Verify the specific scheduled task's environment when necessary.

## 17. Investigate Wildcards in Scheduled Commands

Suppose a privileged cron script contains:

```bash
tar -czf /backup/archive.tar.gz /opt/data/*
```

The wildcard:

```text
*
```

deserves investigation.

### WHY

Shell wildcard expansion can produce unexpected behavior depending on:

* filenames
* command syntax
* current directory
* command options
* file permissions

This is covered in more detail later under:

```text
09-EXECUTION-ABUSE/wildcard-abuse.md
```

At this stage:

```text
Identify
    ↓
Record
    ↓
Validate later
```

## 18. Search for Writable Scheduled Scripts

A useful targeted search:

```bash
find /etc/cron* /opt /usr/local -type f -writable 2>/dev/null
```

### WHY

This helps identify writable files in locations commonly associated with scheduled execution.

### ⚠️ IMPORTANT

A writable file is not automatically a scheduled task.

You still need to establish:

```text
Is it referenced by a scheduled task?
Who executes it?
With what privileges?
```

## 19. Search for Scripts Referenced by Cron

Suppose `/etc/crontab` contains:

```text
*/10 * * * * root /opt/scripts/check.sh
```

Search:

```bash
grep -RniF '/opt/scripts/check.sh' /etc 2>/dev/null
```

### WHY

This can reveal other configuration references.

Then inspect:

```bash
ls -la /opt/scripts/check.sh
stat /opt/scripts/check.sh
namei -l /opt/scripts/check.sh
```

## 20. Understand Systemd Timers

Modern Linux systems frequently use systemd timers instead of traditional cron.

### 🔎 ENUMERATE

```bash
systemctl list-timers --all
```

### WHY

This lists scheduled systemd units.

Example:

```text
NEXT
LEFT
LAST
PASSED
UNIT
ACTIVATES
```

The important relationship is:

```text
Timer
  ↓
Activates
  ↓
Service
```

## 21. Identify Interesting Timers

Look for:

```text
custom names
backup tasks
maintenance tasks
application-specific tasks
scripts under /opt
scripts under /usr/local
```

Example:

```text
backup.timer
```

may activate:

```text
backup.service
```

### ➡️ NEXT STEP

Inspect the associated service.

## 22. Inspect a Systemd Timer

Suppose:

```text
backup.timer
```

Run:

```bash
systemctl status backup.timer
```

Then:

```bash
systemctl cat backup.timer
```

### LOOK FOR

```text
OnBootSec=
OnUnitActiveSec=
OnCalendar=
Unit=
```

The important question is:

> **What service does this timer activate?**

## 23. Inspect the Service Activated by a Timer

If:

```text
Unit=backup.service
```

run:

```bash
systemctl status backup.service
```

Then:

```bash
systemctl cat backup.service
```

Look for:

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

### 🧠 Mental Model

```text
backup.timer
     ↓
backup.service
     ↓
ExecStart
     ↓
script/binary
     ↓
files/commands/dependencies
```

This is the same trust-chain analysis used in process/service enumeration.

## 24. Identify Timers Running With Elevated Privileges

Systemd timers themselves do not necessarily indicate privilege.

The important question is which service they activate and which user that service runs as.

### 🔎 ENUMERATE

```bash
systemctl cat <timer>
```

Then:

```bash
systemctl cat <service>
```

Look for:

```text
User=root
```

or the absence of `User=` where the service defaults to the system manager's privileged context.

### ⚠️ IMPORTANT

Always verify the actual service configuration.

## 25. Search Systemd Timer Files

Common locations include:

```text
/etc/systemd/system/
/usr/lib/systemd/system/
/lib/systemd/system/
```

Search:

```bash
find /etc/systemd /usr/lib/systemd /lib/systemd -type f -name '*.timer' 2>/dev/null
```

### WHY

This can reveal custom timer definitions.

Prioritize unusual or organization-specific timers.

## 26. Inspect Timer File Permissions

Suppose:

```text
/etc/systemd/system/backup.timer
```

Run:

```bash
ls -la /etc/systemd/system/backup.timer
```

Then:

```bash
stat /etc/systemd/system/backup.timer
```

Then:

```bash
namei -l /etc/systemd/system/backup.timer
```

### WHY

A scheduled task depends on its timer configuration.

If an unprivileged user can modify a trusted timer or its execution chain, investigate further.

## 27. Inspect Service File Permissions

Suppose:

```text
backup.service
```

Run:

```bash
systemctl show -p FragmentPath backup.service
```

Then inspect the returned path:

```bash
ls -la /path/to/backup.service
```

And:

```bash
namei -l /path/to/backup.service
```

### LOOK FOR

```text
root-owned
not writable
parent directories not writable
```

or suspicious deviations.

## 28. Search for Writable Timer/Service Components

A useful targeted check:

```bash
find /etc/systemd/system /usr/local /opt -type f -writable 2>/dev/null
```

Then determine whether any result is actually referenced by:

```text
timer
service
scheduled script
```

### 🧠 IMPORTANT

Do not equate:

```text
writable file
```

with:

```text
privilege escalation
```

You need the complete execution chain.

## 29. Investigate `at`

Some systems may have the `at` scheduler installed.

### 🔎 ENUMERATE

```bash
command -v at
```

Then:

```bash
atq
```

### WHY

`at` schedules one-time jobs.

It is less common than cron/systemd timers in many environments, but it can still be relevant.

## 30. Check for `anacron`

Some systems use `anacron` for periodic tasks.

### 🔎 ENUMERATE

```bash
command -v anacron
```

If available:

```bash
cat /etc/anacrontab
```

### WHY

Anacron can execute scheduled commands when the system becomes available after a scheduled interval.

## 31. Search for Application-Level Schedulers

Not all scheduled execution is handled by the operating system.

Applications may use:

```text
Celery
Jenkins
Quartz
custom Python schedulers
Node.js workers
database jobs
application-specific task runners
```

### 🔎 ENUMERATE

Search running processes:

```bash
ps aux | grep -Ei 'celery|jenkins|worker|scheduler|cron|queue'
```

Then investigate unusual applications.

### WHY

An application scheduler may execute commands or scripts with the privileges of the application service account.

## 32. Identify Scheduled Scripts in Application Directories

Search common application locations:

```bash
find /opt /var/www /usr/local -type f \( -name '*.sh' -o -name '*.py' -o -name '*.pl' -o -name '*.rb' \) 2>/dev/null
```

### WHY

Custom applications frequently keep automation scripts under:

```text
/opt
/usr/local
/var/www
```

A script only becomes interesting when you establish that it is automatically executed.

## 33. Investigate Recently Modified Scheduled Files

### 🔎 ENUMERATE

```bash
find /etc/cron* /etc/systemd /opt /usr/local -type f -mtime -7 2>/dev/null
```

### WHY

Recently modified scripts or scheduler definitions can provide useful clues in a lab environment.

Prioritize files that are:

```text
root-owned
executed automatically
custom
writable
```

## 34. Monitor Scheduled Execution With `pspy`

If `pspy` is available, it can help reveal commands executed by processes without requiring root privileges.

### 🔎 ENUMERATE

Run:

```bash
pspy
```

Or the appropriate `pspy` binary available in your authorized lab.

### WHY

You may observe:

```text
cron
systemd
scripts
commands
short-lived processes
```

that are difficult to catch with a static process listing.

### 🧠 IMPORTANT

`pspy` is a discovery tool.

It tells you:

```text
WHAT executed
WHEN it executed
```

You still need to determine:

```text
WHO executed it
WHAT it trusts
CAN I influence it
```

## 35. Combine Static and Dynamic Enumeration

Static enumeration:

```text
cron files
systemd timers
service files
permissions
```

Dynamic enumeration:

```text
pspy
ps
process monitoring
```

Use both when possible.

### 🧠 Mental Model

```text
STATIC
  ↓
What SHOULD execute?
  ↓
DYNAMIC
  ↓
What ACTUALLY executes?
  ↓
COMPARE
  ↓
INVESTIGATE differences
```

This is particularly useful when tasks are generated dynamically.

## 36. Identify the Complete Scheduled Execution Chain

For every interesting scheduled task, document:

```text
SCHEDULER
    ↓
SCHEDULE
    ↓
USER
    ↓
SERVICE/SCRIPT
    ↓
EXECUTABLE
    ↓
COMMANDS
    ↓
FILES
    ↓
DEPENDENCIES
    ↓
USER CONTROL
    ↓
PRIVILEGE IMPACT
```

Example:

```text
systemd timer
    ↓
every 5 minutes
    ↓
root
    ↓
backup.service
    ↓
/opt/scripts/backup.sh
    ↓
tar
    ↓
/opt/data/*
    ↓
current user can modify backup.sh
    ↓
potential privilege boundary
```

This is much more useful than simply recording:

```text
Found cron job.
```

## 37. Scheduled Execution Decision Tree

```text
Start
  │
  ▼
Check cron
  │
  ├── Interesting job?
  │       │
  │       ▼
  │   Identify user
  │       │
  │       ▼
  │   Identify command
  │       │
  │       ▼
  │   Inspect script/binary
  │       │
  │       ▼
  │   Check permissions
  │       │
  │       ▼
  │   Check dependencies
  │       │
  │       ▼
  │   Can current user influence it?
  │       │
  │      YES
  │       ↓
  │   Validate
  │
  ▼
Check systemd timers
  │
  ▼
Identify timer
  │
  ▼
Identify service
  │
  ▼
Identify ExecStart
  │
  ▼
Identify user
  │
  ▼
Check trusted components
  │
  ▼
Can current user influence?
  │
  ▼
Validate
  │
  ▼
Check other schedulers
  │
  ▼
Document findings
```

## 38. High-Value Patterns

### Root Cron + Writable Script

```text
root cron
    ↓
/opt/scripts/backup.sh
    ↓
user can modify script
```

This is a strong candidate for validation.

### Root Cron + Writable Parent Directory

```text
root cron
    ↓
/opt/scripts/backup.sh
    ↓
/opt/scripts writable
```

Investigate how the script is resolved and whether it can be replaced or altered.

### Root Scheduled Script + Writable Configuration

```text
root scheduled task
    ↓
/opt/app/config.ini
    ↓
current user can modify config
```

Determine whether the configuration influences privileged behavior.

### Root Scheduled Script + Relative Command

```text
root scheduled script
    ↓
runs "custom-tool"
    ↓
command resolution
    ↓
potential execution-path issue
```

Investigate under:

```text
09-EXECUTION-ABUSE/path-hijacking.md
```

### Root Scheduled Task + Wildcard

```text
root scheduled task
    ↓
command uses wildcard
    ↓
filename expansion
    ↓
potential command-option/file-name interaction
```

Investigate under:

```text
09-EXECUTION-ABUSE/wildcard-abuse.md
```

## 39. Finding vs Vulnerability

Keep these separate.

### Finding

```text
root executes /opt/scripts/backup.sh every 5 minutes.
```

This is an observation.

### Potential Vulnerability

```text
The current user can modify /opt/scripts/backup.sh.
```

Now there is a privilege-boundary concern.

### Exploitable Path

```text
root scheduler
    ↓
executes user-controlled script
    ↓
script executes with root privileges
    ↓
user-controlled behavior occurs
    ↓
privilege boundary crossed
```

Only after validation should the path be considered exploitable.

## 40. Validation Checklist

Before modifying or exploiting a scheduled task, establish:

```text
[ ] What scheduler executes it?
[ ] What is the schedule?
[ ] Which user executes it?
[ ] What script/binary is executed?
[ ] Is the task actually active?
[ ] Is the script/binary writable?
[ ] Are parent directories writable?
[ ] What configuration does it use?
[ ] What commands does it execute?
[ ] Are command paths absolute?
[ ] What PATH/environment does it use?
[ ] Does it use wildcards?
[ ] What dependencies does it trust?
[ ] Can the current user influence the execution?
[ ] Does that influence cross the privilege boundary?
```

## 41. Common Mistakes

### ❌ Mistake 1 — Finding a cron job and immediately calling it vulnerable

A cron job is normal system functionality.

You need:

```text
privileged execution
+
user influence
+
trusted execution path
```

### ❌ Mistake 2 — Checking only `/etc/crontab`

Scheduled execution can exist in:

```text
/etc/cron.d/
/etc/cron.daily/
/var/spool/cron/
/etc/systemd/system/
/usr/lib/systemd/system/
```

and application-specific schedulers.

### ❌ Mistake 3 — Ignoring systemd timers

Modern Linux systems often rely heavily on systemd.

Always run:

```bash
systemctl list-timers --all
```

when systemd is present.

### ❌ Mistake 4 — Ignoring parent directory permissions

Always use:

```bash
namei -l /path/to/script
```

### ❌ Mistake 5 — Assuming writable files are scheduled

A writable script means nothing unless you prove that a privileged scheduled task executes it.

### ❌ Mistake 6 — Ignoring dynamic execution

Static configuration may not reveal short-lived processes.

Use `pspy` where appropriate.

### ❌ Mistake 7 — Modifying production scheduled tasks

Do not alter cron jobs, systemd timers, or privileged scripts on systems where you do not have explicit authorization.

For labs, use controlled validation.

## 42. The 5-Minute Scheduled Execution Check

If time is limited:

### Step 1 — Cron

```bash
cat /etc/crontab 2>/dev/null
```

### Step 2 — Cron directories

```bash
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/ 2>/dev/null
```

### Step 3 — Your crontab

```bash
crontab -l 2>/dev/null
```

### Step 4 — Systemd timers

```bash
systemctl list-timers --all 2>/dev/null
```

### Step 5 — Interesting scripts

For each candidate:

```bash
ls -la /path/to/script
stat /path/to/script
namei -l /path/to/script
```

### Step 6 — Check content

```bash
sed -n '1,240p' /path/to/script
```

### Step 7 — Dynamic observation

If available:

```bash
pspy
```

Then reduce every finding to:

```text
WHAT
  ↓
WHO
  ↓
WHEN
  ↓
WHERE
  ↓
WHAT DOES IT TRUST?
  ↓
CAN I CONTROL IT?
  ↓
PRIVILEGE IMPACT?
```

## 43. Quick Command Reference

### Cron

```bash
ps aux | grep -E '[c]ron|[c]rond'
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.hourly/
ls -la /etc/cron.daily/
ls -la /etc/cron.weekly/
ls -la /etc/cron.monthly/
crontab -l
```

### Cron spool

```bash
ls -la /var/spool/cron/ 2>/dev/null
ls -la /var/spool/cron/crontabs/ 2>/dev/null
```

### Script permissions

```bash
ls -la /path/to/script
stat /path/to/script
namei -l /path/to/script
```

### Script inspection

```bash
sed -n '1,240p' /path/to/script
command -v <command>
```

### Systemd timers

```bash
systemctl list-timers --all
systemctl status <timer>
systemctl cat <timer>
```

### Timer → service

```bash
systemctl cat <timer>
systemctl status <service>
systemctl cat <service>
systemctl show -p FragmentPath <service>
```

### Search timer files

```bash
find /etc/systemd /usr/lib/systemd /lib/systemd -type f -name '*.timer' 2>/dev/null
```

### Other schedulers

```bash
command -v at
atq
command -v anacron
cat /etc/anacrontab 2>/dev/null
```

### Dynamic monitoring

```bash
pspy
```

## 44. Complete Scheduled Execution Checklist

```text
[ ] Check whether cron is running
[ ] Inspect /etc/crontab
[ ] Inspect /etc/cron.d/
[ ] Inspect cron.daily/hourly/weekly/monthly
[ ] Check accessible user crontabs
[ ] Check current user's crontab
[ ] Identify root scheduled tasks where authorized
[ ] Identify scheduled scripts
[ ] Check script ownership
[ ] Check script permissions
[ ] Check parent directory permissions
[ ] Inspect script contents
[ ] Identify commands used by scripts
[ ] Check absolute vs relative command paths
[ ] Understand scheduled-task environment
[ ] Check PATH where relevant
[ ] Identify wildcard usage
[ ] Enumerate systemd timers
[ ] Map timers to services
[ ] Inspect service execution user
[ ] Inspect ExecStart
[ ] Check timer/service file permissions
[ ] Search for other schedulers
[ ] Check application-level schedulers
[ ] Use pspy when useful
[ ] Identify user-controlled components
[ ] Validate privilege impact
[ ] Document the execution chain
```

## 45. Master Decision Model

Reduce every scheduled execution finding to:

```text
1. WHAT executes?
        ↓
2. WHEN does it execute?
        ↓
3. WHO executes it?
        ↓
4. WHERE is the trusted component?
        ↓
5. WHAT does it depend on?
        ↓
6. CAN I influence it?
        ↓
7. DOES that influence cross the privilege boundary?
```

Example:

```text
cron
 ↓
every 5 minutes
 ↓
root
 ↓
/opt/scripts/backup.sh
 ↓
current user can modify script
 ↓
script executes as root
 ↓
validated privilege-escalation path
```

## 💡 Remember

> **Scheduled execution turns time into an execution mechanism.**

You do not always need to execute a privileged action yourself.

Sometimes the system will execute it for you.

The critical pattern is:

```text
SCHEDULED
    +
PRIVILEGED
    +
TRUSTED COMPONENT
    +
USER CONTROL
    =
POTENTIAL PRIVILEGE-ESCALATION PATH
```

And the workflow remains:

```text
ENUMERATE
    ↓
IDENTIFY
    ↓
UNDERSTAND
    ↓
CHECK PERMISSIONS
    ↓
TRACE TRUST
    ↓
VALIDATE
    ↓
EXPLOIT
    ↓
VERIFY
```

The most important question is:

> **Can I influence something that a privileged scheduled task trusts?**

## 🔗 Where These Findings Lead

```text
Scheduled Execution
       │
       ├── Writable script
       │      ↓
       │ 03-FILESYSTEM-ENUMERATION
       │
       ├── Relative command / PATH issue
       │      ↓
       │ 09-EXECUTION-ABUSE
       │
       ├── Wildcard issue
       │      ↓
       │ 09-EXECUTION-ABUSE
       │
       ├── Service configuration
       │      ↓
       │ 04-PROCESS-SERVICE-ENUMERATION
       │
       ├── Credential/configuration discovery
       │      ↓
       │ 08-CREDENTIALS-AND-SECRETS
       │
       └── Validated scheduled-task weakness
              ↓
          14-EXPLOITATION
```

**The goal is not to find every scheduled task.**

The goal is to identify **privileged automatic execution that trusts something you can control**, then validate whether that relationship actually crosses the privilege boundary.
