# Sudo Misconfiguration — Case Study

> **Purpose:** Demonstrate how excessive or unsafe `sudo` delegation can create a Linux privilege-escalation path, and how to analyze the complete privilege boundary rather than simply looking for a known exploit.

---

## Scenario

A low-privileged Linux user has access to a command through `sudo`.

The command appears legitimate and may even be intentionally delegated for administrative convenience.

The investigation is:

```text id="7m3q8x"
What can the user run?
        ↓
As which user?
        ↓
With which restrictions?
        ↓
What can the allowed program actually do?
        ↓
Can its behavior cross the privilege boundary?
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

Then establish the operating-system context:

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

The primary enumeration step is:

```bash id="6q3m9x"
sudo -l
```

This asks:

> What commands is the current user permitted to execute through `sudo`?

Possible output may resemble:

```text id="4m7x2q"
User may run the following commands:
    (root) /usr/bin/example
```

Or:

```text id="8x2q5m"
(root) /usr/bin/example *
```

The exact output varies by configuration.

---

## What to Record

For every interesting `sudo` rule, record:

```text id="5m9q3x"
Command:
Run-As User:
Run-As Group:
Arguments:
Restrictions:
Environment Restrictions:
NOPASSWD:
```

Do not assume that every `sudo` permission is exploitable.

---

## Identify the Privileged Component

The first important question is:

> What program is actually being executed with elevated privilege?

Example:

```text id="7x4m8q"
Allowed program:
/usr/bin/example

Run as:
root
```

Now investigate the program.

Useful commands:

```bash id="2m8q6x"
ls -l /usr/bin/example
file /usr/bin/example
command -v example
```

If appropriate:

```bash id="9x3m5q"
example --help
```

The objective is to understand the program's functionality.

---

## Determine the Execution Context

A `sudo` rule changes the execution context.

Conceptually:

```text id="6m4q8x"
Current User
     ↓
sudo authorization
     ↓
Allowed Program
     ↓
Run-As User
```

The important question is:

> Does the allowed program perform a privileged action that the original user should not otherwise be able to perform?

---

## Analyze the Program

Investigate whether the allowed program can:

```text id="5x7m2q"
Read arbitrary files
Write arbitrary files
Execute other programs
Load plugins
Invoke a shell
Modify configuration
Access privileged resources
Interpret scripts
Process attacker-controlled input
```

Not every capability creates an escalation path.

The next step is determining whether the behavior is actually usable from the current user's context.

---

## Identify User Control

Ask:

```text id="8q3m6x"
What can I control?
```

Potential control points include:

```text id="4m9x2q"
Command arguments
Input files
Output files
Configuration
Environment
Working directory
Referenced programs
Plugins
Scripts
```

Then ask:

> Does this controlled element influence a privileged operation?

---

## Trust Relationship

The attack path often looks like:

```text id="7x5m2q"
Current User
      ↓
sudo permission
      ↓
Privileged Program
      ↓
Trusted Input / Functionality
      ↓
Privileged Action
```

The key security question is:

> Why does the privileged program trust something the low-privileged user controls?

That relationship is more important than the `sudo` command itself.

---

## Validation

Before exploitation, establish:

```text id="3m8q6x"
[ ] The command can be executed through sudo
[ ] The command runs with elevated privilege
[ ] The relevant behavior is available
[ ] The current user controls the required input
[ ] The behavior reaches a privileged action
[ ] The expected privilege impact is understood
```

Use the smallest safe test necessary.

---

## Controlled Validation Example

Suppose the authorized `sudo` rule allows a program that reads a user-supplied file.

First establish normal behavior:

```bash id="6x2m9q"
sudo /usr/bin/example <test-file>
```

Observe:

```text id="5q8m3x"
What does the program read?
What does it modify?
What privilege does it operate with?
```

Then determine whether the controlled input can influence a privileged operation.

Do not jump directly to an unrestricted payload.

---

## Exploitation

Once the relationship has been validated:

```text id="9m4q7x"
Sudo Permission
      ↓
Allowed Program
      ↓
User-Controlled Input
      ↓
Privileged Behavior
      ↓
Privilege Boundary
```

Perform the minimum action required to demonstrate the escalation in the authorized lab.

The exact exploitation method depends entirely on the allowed program.

There is no universal `sudo` exploit.

---

## Verification

After the controlled exploitation attempt:

```bash id="4x8m2q"
whoami
id
id -u
```

If the expected result is root:

```text id="7m5q3x"
UID = 0
```

Verify the actual execution context rather than assuming the exploit worked.

---

## Root Cause

Possible root causes include:

```text id="8m3q6x"
Excessive sudo delegation
Unsafe command delegation
Insufficient command restrictions
Privileged program with dangerous functionality
Unnecessary administrative privilege
```

The correct root cause depends on the actual configuration.

Do not simply label every case:

> "Sudo is misconfigured."

Explain exactly what trust relationship created the privilege boundary.

---

## Attack Path

Document the complete path:

```text id="5q7m2x"
Initial User
     ↓
Allowed sudo rule
     ↓
Privileged program
     ↓
User-controlled functionality
     ↓
Privileged operation
     ↓
Verified privilege escalation
```

Example documentation:

```text id="3x9m6q"
The initial user was permitted to execute PROGRAM through sudo
as root. PROGRAM exposed FUNCTIONALITY controlled by the user.
That functionality caused ACTION to execute in the root context.
The resulting privilege change was verified using id and UID.
```

---

## Evidence

Capture only the evidence necessary to demonstrate the finding.

Useful evidence:

```bash id="6m4q9x"
sudo -l
id
```

And relevant information about the allowed program:

```bash id="8x2m5q"
ls -l /path/to/program
file /path/to/program
```

Record the final verification:

```bash id="9q3m7x"
whoami
id
id -u
```

Sanitize sensitive information before publishing the case study.

---

## False Positives

Not every `sudo` rule is dangerous.

Examples:

```text id="7m5x2q"
Allowed command has no dangerous functionality
Arguments are tightly restricted
Program cannot access attacker-controlled resources
Program runs with a non-privileged account
Required functionality is unavailable
Privilege boundary cannot be crossed
```

The correct conclusion may therefore be:

```text id="4x8m6q"
Interesting sudo permission
        ↓
Investigated
        ↓
No exploitable path
```

Rejecting a false positive is part of successful enumeration.

---

## Common Mistakes

### Stopping at `sudo -l`

Seeing:

```bash id="2q7m9x"
sudo -l
```

is only the beginning.

The allowed program must be analyzed.

---

### Assuming Root Execution Means Exploitable

A program running as root does not automatically provide unrestricted control.

Determine what the user can influence.

---

### Searching for a Known Exploit Too Early

First understand the actual command and its behavior.

---

### Ignoring Arguments

A command may be safe in one invocation but dangerous when specific arguments or inputs are allowed.

---

### Ignoring Restrictions

Check the complete `sudo` rule.

A command restricted to a fixed argument set is different from unrestricted execution.

---

### Failing to Verify

Always confirm the resulting identity.

---

## Case Study Checklist

```text id="5m3q8x"
[ ] Initial user identified
[ ] Initial UID recorded
[ ] Groups recorded
[ ] OS identified
[ ] sudo -l executed
[ ] Relevant sudo rule recorded
[ ] Run-As user identified
[ ] Restrictions identified
[ ] Allowed program identified
[ ] Program behavior investigated
[ ] User-controlled input identified
[ ] Trust relationship understood
[ ] Privileged action identified
[ ] Finding validated
[ ] Exploitation performed in scope
[ ] Final privilege verified
[ ] Root cause documented
[ ] Evidence preserved
[ ] Cleanup completed
```

---

## Methodology Mapping

This case study reinforces:

```text id="8x4m6q"
06-PRIVILEGE-MECHANISMS/
└── sudo.md

13-FINDING-VALIDATION/
├── finding-vs-vulnerability.md
├── exploitability.md
└── privilege-boundary-analysis.md

14-EXPLOITATION/
└── sudo-exploitation.md

15-ROOT-VERIFICATION/
├── verify-privileges.md
└── collect-evidence.md

16-METHODOLOGY/
└── attack-path-analysis.md
```

---

## What to Remember

The most important lesson is not:

> "Check `sudo -l`."

It is:

```text id="6q9m2x"
Check sudo
    ↓
Understand the allowed program
    ↓
Identify what the user controls
    ↓
Identify what the program trusts
    ↓
Determine what executes with privilege
    ↓
Validate the boundary
    ↓
Exploit only after validation
    ↓
Verify
```

> **A sudo permission becomes an escalation path when the delegated privileged functionality gives the low-privileged user meaningful control over a privileged action.**
