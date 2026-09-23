# SUID Binary — Case Study

> **Purpose:** Demonstrate how a root-owned SUID executable can become a Linux privilege-escalation path when its privileged behavior can be meaningfully influenced by a lower-privileged user.

---

## Scenario

A low-privileged user discovers an executable with the SUID permission bit set.

The file is owned by `root`.

At first glance:

```text id="7m4q2x"
SUID binary exists
        ↓
Interesting
```

But that is not enough.

The investigation must determine:

```text id="5x8m3q"
Who owns it?
Who can execute it?
What does it do?
What does it trust?
What can the current user control?
Does that control affect privileged behavior?
```

---

## Initial Context

Start with:

```bash id="8q3m6x"
whoami
id
hostname
```

Record:

```text id="4m7x9q"
Initial User:
UID:
Groups:
Hostname:
```

Establish system context:

```bash id="6x2m8q"
uname -a
cat /etc/os-release
uname -m
```

Record:

```text id="9m5q3x"
OS:
Version:
Kernel:
Architecture:
```

---

## Enumeration

Search for SUID executables:

```bash id="3x8m6q"
find / -perm -4000 -type f 2>/dev/null
```

A more explicit form is:

```bash id="7q4m2x"
find / -type f -perm -u=s 2>/dev/null
```

For SGID files:

```bash id="5m9x3q"
find / -perm -2000 -type f 2>/dev/null
```

The goal is to identify unusual or potentially relevant privileged executables.

---

## What SUID Means

For an executable owned by `root` with SUID set, execution can cause the process's effective user ID to become the file owner, subject to the operating system and execution conditions.

Conceptually:

```text id="8x3m6q"
Normal executable:

User
 ↓
Process
 ↓
User privilege


Root-owned SUID executable:

User
 ↓
SUID executable
 ↓
Process with elevated effective UID
```

This does **not** mean every SUID binary is exploitable.

The program's behavior still matters.

---

## Identify the Binary

For an interesting candidate:

```bash id="6m4q8x"
ls -l /path/to/binary
file /path/to/binary
```

Record:

```text id="2x7m9q"
Path:
Owner:
Group:
Permissions:
Architecture:
File type:
```

Example permission pattern:

```text id="9m3q5x"
-rwsr-xr-x
```

The `s` in the owner execute position indicates SUID.

---

## Determine Whether It Is Actually SUID

Use:

```bash id="4q8m2x"
stat /path/to/binary
```

Look for the permission information.

The important relationships are:

```text id="7x5m9q"
Owner
+
SUID
+
User can execute
```

All three should be considered.

---

## Investigate the Program

Do not immediately search for an exploit.

First understand what the binary does.

Try normal informational options where appropriate:

```bash id="3m7q8x"
/path/to/binary --help
```

Also consider:

```bash id="8x4m6q"
strings /path/to/binary 2>/dev/null
```

For dynamically linked binaries:

```bash id="5q9x2m"
ldd /path/to/binary 2>/dev/null
```

Use these as investigative aids, not as automatic proof of vulnerability.

---

## Identify Program Behavior

Determine whether the program:

```text id="6m3q8x"
Executes commands
Reads files
Writes files
Loads libraries
Uses configuration files
Searches PATH
Processes filenames
Uses environment variables
Invokes interpreters
Creates files
Accesses privileged resources
```

The important question is:

> Which of these behaviors can the current user influence?

---

## Identify User Control

For the candidate binary, investigate:

```text id="9x5m2q"
Command-line arguments
Input files
Output paths
Environment
PATH
Working directory
Configuration
Referenced executables
Referenced libraries
File names
```

Then connect the control to privileged execution.

---

## Ownership and Permissions

Check the binary:

```bash id="7m4q9x"
ls -l /path/to/binary
```

Then investigate resources it references.

For example:

```bash id="2x8m6q"
ls -la /path/to/resource
stat /path/to/resource
```

The critical question is:

> Can the low-privileged user modify something that the SUID process trusts?

---

## Trust Relationship

A useful model is:

```text id="5q3m9x"
Low-Privilege User
       ↓
User-Controlled Input
       ↓
SUID Program
       ↓
Elevated Effective Privilege
       ↓
Privileged Action
```

The escalation depends on the relationship between the input and the privileged action.

---

## Validation

Before exploitation, establish:

```text id="8x6m4q"
[ ] Binary is actually SUID
[ ] Binary is owned by a privileged account
[ ] Current user can execute it
[ ] Program behavior is understood
[ ] Relevant input is user-controlled
[ ] Input reaches privileged behavior
[ ] Privilege impact is understood
```

A SUID bit alone is insufficient.

---

## Safe Behavioral Testing

Start with benign behavior.

For example:

```bash id="4m7x2q"
/path/to/binary --help
```

or a harmless input supported by the application.

Observe:

```text id="9q5m3x"
What files does it access?
What programs does it invoke?
What output does it produce?
What user context does the process use?
```

The objective is to prove the behavior before attempting privilege-impacting actions.

---

## Process Context

When useful in an authorized lab, inspect the process while it executes.

For example:

```bash id="6x8m4q"
ps aux | grep '[b]inary'
```

The exact process details depend on the program.

The important distinction is:

```text id="7m3q9x"
Real UID
vs.
Effective UID
```

A privileged process may retain the invoking user's real UID while using the file owner's effective UID.

---

## Exploitation

Once the attack path is validated:

```text id="5x9q2m"
User
 ↓
SUID Binary
 ↓
Controlled Input
 ↓
Privileged Operation
 ↓
Privilege Escalation
```

Use the smallest effective action necessary in the authorized environment.

The exploitation method depends on the actual binary.

There is no universal SUID payload.

---

## Verification

After exploitation:

```bash id="8m4x6q"
whoami
id
id -u
```

For a root result:

```text id="3q7m9x"
UID = 0
```

If the objective is a privileged file or another controlled resource, verify that separately.

Do not confuse:

```text id="6m2x8q"
Program executed
```

with:

```text id="9x4m7q"
Privilege boundary successfully crossed
```

---

## Root Cause

Possible root causes include:

```text id="5m8q2x"
Unsafe SUID program design
Dangerous command execution
Trusted user-controlled resource
Unsafe PATH resolution
Unsafe library loading
Insecure configuration handling
Excessive privileged functionality
```

The correct root cause depends on the binary.

The SUID bit itself is not necessarily the vulnerability.

---

## Attack Path

Document the complete chain:

```text id="7x3m9q"
Initial User
     ↓
Root-Owned SUID Binary
     ↓
User-Controlled Input
     ↓
Privileged Program Behavior
     ↓
Privileged Action
     ↓
Verified Result
```

A strong write-up should explain every arrow.

---

## False Positives

Common SUID false positives include:

```text id="4q9m2x"
Standard system binary
No dangerous user-controlled functionality
Restricted behavior
No useful privileged operation
Required input cannot be influenced
Privilege boundary cannot be crossed
```

Therefore:

```text id="8m5x3q"
SUID Found
   ↓
Investigate
   ↓
No Control
   ↓
Move On
```

This is a valid outcome.

---

## Common SUID Investigation Paths

When a SUID binary is interesting, investigate these categories:

### Command Execution

```text id="6x4m8q"
Does the program execute another command?
Is the command path absolute?
Can command selection be influenced?
```

### PATH Resolution

```text id="9m3q7x"
Does it invoke programs without absolute paths?
Can the execution environment influence resolution?
```

### Library Loading

```text id="5q8m2x"
Does it load libraries from locations the user can influence?
Are the relevant loading conditions satisfied?
```

### Configuration

```text id="7m4x9q"
Does the binary read a configuration file?
Who owns the configuration?
Can the current user modify it?
```

### File Operations

```text id="3x6m8q"
Can the program read or write files chosen by the user?
Does that behavior occur with elevated privilege?
```

---

## Common Mistakes

### Treating Every SUID Binary as Vulnerable

SUID is a reason to investigate, not proof of exploitation.

---

### Ignoring Ownership

Always verify:

```bash id="8q2m5x"
ls -l /path/to/binary
```

---

### Ignoring Program Behavior

The permission bits do not explain what the application does.

---

### Jumping Straight to GTFOBins

Reference material can identify known patterns, but first understand the actual binary and environment.

Use external knowledge to confirm a hypothesis, not replace analysis.

---

### Ignoring the Effective UID

Privilege escalation concerns execution context.

Understand the difference between real and effective identity.

---

### Forgetting Validation

A suspicious SUID binary is a candidate until its privileged behavior is proven relevant.

---

## Evidence

Useful evidence includes:

```bash id="4m7x2q"
ls -l /path/to/binary
file /path/to/binary
stat /path/to/binary
```

And, where appropriate:

```bash id="6x3m8q"
strings /path/to/binary 2>/dev/null
ldd /path/to/binary 2>/dev/null
```

Record:

```text id="9m5q2x"
Binary path
Owner
Permissions
Relevant behavior
User-controlled resource
Validation result
Final privilege
```

Sanitize sensitive data before publishing.

---

## Cleanup

After testing:

```text id="7q4m8x"
[ ] Temporary files removed
[ ] Modified test resources restored
[ ] Temporary processes stopped
[ ] Temporary configuration changes reverted
[ ] Evidence preserved
```

Do not alter legitimate system files unnecessarily.

---

## Case Study Checklist

```text id="5x8m9q"
[ ] Initial user identified
[ ] Initial privilege recorded
[ ] SUID binaries enumerated
[ ] Candidate identified
[ ] Ownership checked
[ ] Permissions checked
[ ] SUID state confirmed
[ ] Binary behavior investigated
[ ] User-controlled input identified
[ ] Privileged behavior identified
[ ] Trust relationship understood
[ ] Finding validated
[ ] Exploitation performed in scope
[ ] Final privilege verified
[ ] Root cause documented
[ ] Evidence collected
[ ] Cleanup completed
```

---

## Methodology Mapping

This case study reinforces:

```text id="8m4q2x"
06-PRIVILEGE-MECHANISMS/
└── suid.md

13-FINDING-VALIDATION/
├── finding-vs-vulnerability.md
├── exploitability.md
└── privilege-boundary-analysis.md

14-EXPLOITATION/
└── suid-exploitation.md

15-ROOT-VERIFICATION/
└── verify-privileges.md

16-METHODOLOGY/
├── attack-path-analysis.md
└── enumeration-priority.md
```

---

## What to Remember

The memorable rule is:

```text id="6q3m8x"
SUID
  ↓
WHO OWNS IT?
  ↓
WHAT DOES IT DO?
  ↓
WHAT DOES IT TRUST?
  ↓
WHAT CAN I CONTROL?
  ↓
DOES THAT CONTROL REACH PRIVILEGED CODE?
  ↓
VALIDATE
  ↓
EXPLOIT
  ↓
VERIFY
```

> **A SUID bit tells you that elevated execution is possible. It does not tell you that privilege escalation is possible. The decisive question is whether the current user can influence the privileged behavior.**
