# Writable Cron Job — Case Study

> **Purpose:** Demonstrate how a writable resource used by a privileged scheduled task can create a Linux privilege-escalation path, and how to prove the complete execution chain before exploitation.

---

## Scenario

A low-privileged Linux user discovers a scheduled task that executes with elevated privileges.

The task itself may be legitimate.

The security question is:

```text id="7m3q8x"
What executes?
        ↓
Who executes it?
        ↓
What resource does it use?
        ↓
Who can modify that resource?
        ↓
When does it execute?
        ↓
Can the modification influence privileged execution?
```

The lab should be performed only in an authorized environment.

---

## Initial Context

Start with:

```bash id="5x8m2q"
whoami
id
hostname
```

Record:

```text id="9q4m7x"
Initial User:
UID:
Groups:
Hostname:
```

Establish the system context:

```bash id="3m6q8x"
uname -a
cat /etc/os-release
uname -m
```

Record:

```text id="8x5m2q"
OS:
Version:
Kernel:
Architecture:
```

---

## Enumeration

Start with the system-wide crontab:

```bash id="6q3m9x"
cat /etc/crontab
```

Then inspect common cron directories:

```bash id="4m7x2q"
ls -la /etc/cron.d/
ls -la /etc/cron.hourly/
ls -la /etc/cron.daily/
ls -la /etc/cron.weekly/
ls -la /etc/cron.monthly/
```

User-specific cron entries may also be relevant:

```bash id="8x2q5m"
crontab -l 2>/dev/null
```

The exact locations depend on the system.

---

## What to Look For

A potentially interesting scheduled task has several characteristics:

```text id="5m9q3x"
Privileged execution
        +
Known executable/script
        +
User influence
        +
Predictable trigger
```

For example:

```text id="7x4m8q"
root
 ↓
/opt/backup.sh
 ↓
runs every minute
```

Now investigate whether `/opt/backup.sh` can be influenced by the current user.

---

## Identify the Scheduled Task

For an interesting entry, record:

```text id="2m8q6x"
Schedule:
User:
Command:
Script:
Arguments:
Working Directory:
```

Example:

```text id="9m3q5x"
*/1 * * * * root /opt/backup.sh
```

This tells you:

```text id="6x8m2q"
Trigger:
Every minute

Execution user:
root

Target:
 /opt/backup.sh
```

But this is still only a finding.

---

## Inspect the Script

Check the file:

```bash id="3q7m9x"
ls -l /opt/backup.sh
stat /opt/backup.sh
file /opt/backup.sh
```

Read it where permitted:

```bash id="5x4m8q"
cat /opt/backup.sh
```

Record:

```text id="8m2q7x"
Owner:
Group:
Permissions:
Interpreter:
Commands:
Referenced files:
Referenced programs:
```

---

## Ownership and Permissions

The critical question is:

> Can the current user modify the resource executed by the privileged cron job?

For example:

```text id="7m3x9q"
-rwxrwxrwx
```

would indicate that the file is writable by everyone.

But permissions alone are not enough.

Confirm the current user's actual access.

---

## Determine User Control

Check:

```bash id="4x8m6q"
ls -l /opt/backup.sh
```

Then inspect the parent directory:

```bash id="9q5m2x"
ls -ld /opt
```

A file may appear writable while directory permissions or other controls affect what the user can actually do.

The complete question is:

```text id="6m4q8x"
Can the current user modify the exact resource
that the privileged task executes?
```

---

## Trace the Execution Chain

A cron attack path should be represented as:

```text id="8x3m5q"
Current User
      ↓
Writable Script
      ↓
Cron
      ↓
Root Execution
      ↓
Privileged Action
```

Every arrow needs evidence.

---

## Inspect Script Dependencies

A script may execute additional programs or access additional resources.

For example:

```bash id="5q7m3x"
grep -nE '(^|[[:space:]])(cp|mv|tar|find|chmod|chown|bash|sh|python|perl|ruby|php)([[:space:]]|$)' /opt/backup.sh
```

The exact analysis depends on the script.

Investigate:

```text id="7m9x2q"
Commands
Arguments
Absolute vs relative paths
Referenced files
Referenced directories
Environment assumptions
PATH usage
```

The goal is to identify the actual trusted resource.

---

## PATH Considerations

If the script executes commands using relative names:

```text id="3x8m6q"
command
```

instead of:

```text id="6m2q9x"
/usr/bin/command
```

investigate how command resolution occurs.

Check:

```bash id="8q4m7x"
echo "$PATH"
```

However:

> A PATH entry alone is not a vulnerability.

You need to establish that the privileged execution actually uses a controllable command resolution path.

---

## Trigger Analysis

Cron introduces an important variable:

> When does the privileged action occur?

Possible schedules include:

```text id="5m3q8x"
Every minute
Every hour
Daily
Weekly
At reboot
At a specific time
```

Determine the trigger from the cron configuration.

Do not assume execution frequency.

---

## Validation

Before exploitation, confirm:

```text id="9x7m4q"
[ ] Cron entry exists
[ ] Privileged execution confirmed
[ ] Target resource identified
[ ] Current user can influence resource
[ ] Resource is actually used by the scheduled task
[ ] Execution trigger understood
[ ] Expected privilege impact understood
```

This converts:

```text id="4m8q2x"
Writable File
```

into:

```text id="7q3m9x"
Writable Root-Executed File
```

which is a meaningful attack-path candidate.

---

## Safe Validation

Where possible, use a harmless observable effect to confirm execution.

For example, in a purpose-built lab, a test marker can be used:

```bash id="5x9m3q"
printf '%s\n' "cron-test" > /tmp/cron-test-marker
```

The exact validation depends on the lab design.

The principle is:

```text id="8m4q6x"
Prove execution
without causing unnecessary system impact.
```

---

## Exploitation

Once the path is validated:

```text id="3x7m9q"
Current User
      ↓
Writable Resource
      ↓
Privileged Cron Job
      ↓
Controlled Action
      ↓
Privilege Escalation
```

In an authorized lab, modify the vulnerable resource in the smallest way necessary to demonstrate the privilege boundary.

Do not introduce persistence or unrelated system changes.

---

## Verification

After the scheduled task executes, verify the resulting privilege.

Run:

```bash id="6m2q8x"
whoami
id
id -u
```

If the expected result is root:

```text id="9x4m5q"
UID = 0
```

If the lab uses another controlled indicator, verify that separately.

---

## Timing and Troubleshooting

If the expected result does not appear immediately:

```text id="7m3q9x"
Check cron schedule
       ↓
Check file permissions
       ↓
Check script syntax
       ↓
Check script path
       ↓
Check execution user
       ↓
Check script dependencies
       ↓
Check whether the task actually runs
```

Do not repeatedly modify the payload without first determining which part of the chain failed.

---

## Root Cause

Typical root causes include:

```text id="5x8m2q"
Root executes a user-writable script
Root trusts user-writable configuration
Privileged scheduled task uses user-controlled resource
Unsafe script dependency
Insecure execution environment
```

The correct root cause must match the actual case.

---

## Attack Path

Document it clearly:

```text id="8q4m6x"
Initial User
     ↓
Can modify RESOURCE
     ↓
RESOURCE is executed by CRON
     ↓
CRON runs as root
     ↓
Controlled content executes with root privilege
     ↓
UID 0 verified
```

This is much stronger than writing:

> "Cron was vulnerable."

---

## False Positives

Not every cron entry is exploitable.

Examples:

```text id="3m7x9q"
Root cron job exists
        ↓
Script is root-owned
        ↓
User cannot modify it
        ↓
No useful control
```

Another example:

```text id="6x4m8q"
User can modify a file
        ↓
Cron never consumes it
        ↓
No attack path
```

The complete relationship matters.

---

## Common Mistakes

### Only Checking `/etc/crontab`

Cron configuration can exist in multiple locations.

---

### Seeing a Root Cron Job and Stopping

A root cron job is a candidate, not automatically a vulnerability.

---

### Ignoring the Parent Directory

File access should be considered together with directory permissions and ownership.

---

### Ignoring Script Dependencies

The vulnerable resource may be a file, command, directory, environment variable, or another dependency used by the script.

---

### Forgetting Timing

A correct modification may appear ineffective simply because the scheduled task has not executed yet.

---

### Modifying More Than Necessary

Use the smallest change that proves the path.

---

## Evidence

Useful evidence includes:

```bash id="2m8q6x"
cat /etc/crontab
ls -la /etc/cron.d/
ls -l /path/to/script
stat /path/to/script
```

And the final verification:

```bash id="7x3m5q"
whoami
id
id -u
```

Record:

```text id="5q9m3x"
Cron schedule
Execution user
Target resource
Ownership
Permissions
User control
Validation result
Execution result
Final privilege
```

---

## Cleanup

Restore the vulnerable resource to its intended lab state.

```text id="8m4x2q"
[ ] Test modifications removed
[ ] Temporary files removed
[ ] Temporary processes stopped
[ ] Cron configuration restored if changed
[ ] Original permissions restored
[ ] Evidence preserved
```

If the lab intentionally remains vulnerable for another exercise, restore it to the documented baseline rather than hardening it permanently.

---

## Case Study Checklist

```text id="6x3m9q"
[ ] Initial user identified
[ ] Initial privilege recorded
[ ] Cron configuration enumerated
[ ] Interesting scheduled task identified
[ ] Execution user identified
[ ] Target resource identified
[ ] Resource ownership checked
[ ] Resource permissions checked
[ ] User control confirmed
[ ] Script dependencies investigated
[ ] Execution timing understood
[ ] Attack path validated
[ ] Exploitation performed in scope
[ ] Privilege verified
[ ] Root cause documented
[ ] Evidence preserved
[ ] Lab restored
```

---

## Methodology Mapping

This case study reinforces:

```text id="4m7x8q"
07-SCHEDULED-EXECUTION/
├── cron.md
├── cron-jobs.md
└── writable-scheduled-tasks.md

13-FINDING-VALIDATION/
├── finding-vs-vulnerability.md
├── exploitability.md
└── privilege-boundary-analysis.md

14-EXPLOITATION/
└── cron-exploitation.md

15-ROOT-VERIFICATION/
└── verify-privileges.md

16-METHODOLOGY/
├── attack-path-analysis.md
└── time-boxed-enumeration.md
```

---

## What to Remember

The memorable rule is:

```text id="9q5m3x"
CRON
 ↓
WHO RUNS IT?
 ↓
WHAT DOES IT RUN?
 ↓
WHO CONTROLS THAT RESOURCE?
 ↓
WHEN DOES IT RUN?
 ↓
CAN MY CONTROL REACH PRIVILEGED EXECUTION?
 ↓
VALIDATE
 ↓
EXPLOIT
 ↓
VERIFY
```

> **A scheduled task becomes an escalation path when a higher-privileged execution context consumes something the lower-privileged user can meaningfully control.**
