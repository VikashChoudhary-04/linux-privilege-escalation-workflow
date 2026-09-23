# Linux Capabilities — Case Study

> **Purpose:** Demonstrate how Linux file capabilities can grant privileged operations to an executable and how to determine whether those capabilities create a real privilege-escalation path.

---

## Scenario

A low-privileged user discovers a binary with Linux capabilities assigned to it.

Unlike traditional SUID execution, capabilities can grant a process specific privileged powers.

The investigation is:

```text id="7m4q2x"
What capability is assigned?
        ↓
Which binary has it?
        ↓
What does the binary do?
        ↓
Can the current user execute it?
        ↓
What privileged operation does the capability permit?
        ↓
Can that operation cross the privilege boundary?
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

Establish system context:

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

## What Are Linux Capabilities?

Linux capabilities divide traditionally privileged operations into separate units.

Instead of requiring a process to have unrestricted root privilege, a process can be granted specific capabilities.

Conceptually:

```text id="6q3m9x"
Traditional model

Root
 ↓
Broad privileged access
```

versus:

```text id="4m7x2q"
Capability model

Process
 ↓
Specific capability
 ↓
Specific privileged operation
```

Examples include capabilities associated with:

```text id="8x2q5m"
Network administration
Raw network access
Changing file ownership
Changing file permissions
Binding to privileged ports
Changing process identity
```

The exact effect depends on the capability and execution context.

---

## Enumeration

Search for file capabilities:

```bash id="5m9q3x"
getcap -r / 2>/dev/null
```

Example output might resemble:

```text id="7x4m8q"
/usr/bin/example cap_example=ep
```

The exact capability and binary vary by system.

---

## What to Record

For each interesting result:

```text id="2m8q6x"
Binary:
Capability:
Flags:
Owner:
Permissions:
```

Check the file:

```bash id="9m3q5x"
ls -l /path/to/binary
file /path/to/binary
```

Then:

```bash id="6x8m2q"
getcap /path/to/binary
```

---

## Capability Flags

Capability output can contain flags such as:

```text id="3q7m9x"
e
p
i
```

For practical privilege-escalation analysis, understand what the displayed capability state means rather than treating the presence of any capability as automatically dangerous.

A capability must be evaluated together with:

```text id="5x4m8q"
The binary
+
The capability
+
How the binary behaves
+
How the user can invoke it
```

---

## Identify the Binary

For a candidate:

```bash id="8m2q7x"
ls -l /path/to/binary
file /path/to/binary
```

Record:

```text id="7m3x9q"
Path:
Owner:
Group:
Permissions:
Architecture:
```

Determine whether the current user can execute it.

---

## Investigate the Program

Understand what the binary actually does.

Where appropriate:

```bash id="4x8m6q"
/path/to/binary --help
```

You can also inspect basic binary information:

```bash id="9q5m2x"
strings /path/to/binary 2>/dev/null
```

For dynamically linked programs:

```bash id="6m4q8x"
ldd /path/to/binary 2>/dev/null
```

These commands provide clues.

They do not by themselves prove exploitability.

---

## Identify the Privileged Operation

The central question is:

> What can this capability allow the program to do that an ordinary process could not?

For example:

```text id="8x3m5q"
Capability
   ↓
Privileged Operation
   ↓
Program Behavior
```

You must connect the capability to actual program behavior.

---

## Identify User Control

Determine what the current user can influence:

```text id="5q7m3x"
Command-line arguments
Input files
Output files
Environment
PATH
Configuration
Working directory
Program dependencies
```

Then ask:

> Can this control cause the capability-enabled program to perform a privileged operation?

---

## Trust Relationship

A capability-based attack path can be represented as:

```text id="7m9x2q"
Low-Privilege User
       ↓
Capability-Enabled Binary
       ↓
User-Controlled Input
       ↓
Capability-Backed Operation
       ↓
Privilege Boundary
```

The capability is the privileged mechanism.

The program's behavior determines whether the mechanism is useful.

---

## Validation

Before exploitation, confirm:

```text id="3x6m8q"
[ ] Capability exists
[ ] Capability is attached to the intended binary
[ ] Current user can execute the binary
[ ] Capability meaning is understood
[ ] Program behavior is understood
[ ] User-controlled input is identified
[ ] Capability-backed operation is reachable
[ ] Privilege impact is understood
```

Do not treat:

```text id="4m8q2x"
Capability found
```

as equivalent to:

```text id="9q5m3x"
Privilege escalation validated
```

---

## Safe Behavioral Testing

Begin with normal functionality.

For example:

```bash id="6x2m8q"
/path/to/binary --help
```

or another harmless operation supported by the lab.

Observe:

```text id="5m9q4x"
What does the program access?
What operations does it perform?
Which inputs affect behavior?
```

The objective is to establish the relationship between:

```text id="7q3m8x"
Capability
 ↓
Program
 ↓
Operation
```

before attempting privilege-impacting actions.

---

## Capability and Effective Privilege

Capabilities are not simply another representation of UID 0.

A process may retain:

```text id="8m4x2q"
Real UID ≠ 0
```

while possessing a capability that permits a specific privileged operation.

Therefore, do not use:

```text id="3x7m9q"
whoami
```

alone to determine whether a capability is relevant.

The capability itself must be analyzed.

---

## Exploitation

Once the attack path is validated:

```text id="5q8m3x"
Current User
      ↓
Executable With Capability
      ↓
Controlled Input
      ↓
Capability-Enabled Operation
      ↓
Privilege Impact
```

Perform only the minimum controlled action necessary in the authorized lab.

The exact exploitation technique depends on:

```text id="9m4x7q"
Capability
Binary
Version
Program behavior
Environment
```

There is no universal capability exploit.

---

## Verification

After exploitation, verify the actual result:

```bash id="6x3m8q"
whoami
id
id -u
```

If the lab is intended to produce root:

```text id="8q5m2x"
UID = 0
```

If the capability instead grants a specific privileged operation without changing UID, verify that operation directly.

This distinction is important.

```text id="7m4x9q"
Privileged capability
≠
Automatically UID 0
```

---

## Root Cause

Possible root causes include:

```text id="5x8m3q"
Excessive capability assignment
Capability assigned to unnecessary executable
Privileged operation exposed through user-controlled functionality
Unsafe capability-enabled program
Incorrect security boundary design
```

The actual root cause depends on the case.

---

## Attack Path

Document the relationship:

```text id="2q7m9x"
Initial User
     ↓
Can execute binary
     ↓
Binary possesses capability
     ↓
Capability permits privileged operation
     ↓
User controls relevant program behavior
     ↓
Privileged operation occurs
     ↓
Impact verified
```

If the result is UID 0, document that explicitly.

If the result is a privileged operation without UID 0, document that instead.

---

## False Positives

A capability finding may not provide an escalation path.

Examples:

```text id="8m4x6q"
Capability exists
        ↓
Binary is not executable by current user
```

Or:

```text id="3x9q5m"
Capability exists
        ↓
Capability is unrelated to useful privileged behavior
```

Or:

```text id="7m2x8q"
Capability exists
        ↓
Program does not expose controllable functionality
```

Therefore:

```text id="5q4m9x"
Capability
   ↓
Investigate
   ↓
Understand
   ↓
Validate
```

---

## Common Investigation Paths

### Network-Related Capabilities

Investigate whether the binary can perform privileged network operations and whether those operations meaningfully affect the target.

Do not assume network privilege automatically means root access.

---

### File-Related Capabilities

Investigate whether the capability permits operations on files that the current user otherwise could not modify.

Then determine whether that access can cross the intended privilege boundary.

---

### Process-Related Capabilities

Determine what privileged process operations are permitted and whether the current user can meaningfully influence a privileged process.

---

## Common Mistakes

### Treating Every Capability as Dangerous

Capabilities are granular.

Determine what the specific capability actually permits.

---

### Ignoring the Binary

The same capability can have very different security implications depending on the executable.

---

### Assuming Capabilities Equal Root

A capability grants specific powers.

It does not automatically create an unrestricted root shell.

---

### Ignoring User Control

A privileged operation matters only if the current user can reach or influence it.

---

### Skipping Validation

Always establish the complete attack path before exploitation.

---

### Using `getcap` as the Final Answer

Enumeration identifies candidates.

Analysis identifies attack paths.

---

## Evidence

Useful evidence includes:

```bash id="4m8q2x"
getcap -r / 2>/dev/null
getcap /path/to/binary
ls -l /path/to/binary
file /path/to/binary
```

Where relevant:

```bash id="6x3m9q"
strings /path/to/binary 2>/dev/null
ldd /path/to/binary 2>/dev/null
```

Record:

```text id="9m5x2q"
Binary
Capability
Flags
Owner
Permissions
Relevant behavior
User-controlled input
Validation result
Privilege impact
```

---

## Cleanup

After testing:

```text id="7q4m8x"
[ ] Temporary files removed
[ ] Modified resources restored
[ ] Temporary processes stopped
[ ] Configuration restored
[ ] Evidence preserved
```

Do not remove or alter the capability unless the lab specifically requires it.

---

## Case Study Checklist

```text id="5x8m3q"
[ ] Initial user identified
[ ] Initial privilege recorded
[ ] Capabilities enumerated
[ ] Candidate binary identified
[ ] Capability confirmed
[ ] Capability meaning understood
[ ] Binary ownership checked
[ ] Binary permissions checked
[ ] Program behavior investigated
[ ] User control identified
[ ] Privileged operation identified
[ ] Attack path validated
[ ] Exploitation performed in scope
[ ] Result verified
[ ] Root cause documented
[ ] Evidence preserved
[ ] Cleanup completed
```

---

## Methodology Mapping

This case study reinforces:

```text id="8m4q6x"
06-PRIVILEGE-MECHANISMS/
└── capabilities.md

13-FINDING-VALIDATION/
├── finding-vs-vulnerability.md
├── exploitability.md
└── privilege-boundary-analysis.md

14-EXPLOITATION/
└── capability-exploitation.md

15-ROOT-VERIFICATION/
└── verify-privileges.md

16-METHODOLOGY/
└── attack-path-analysis.md
```

---

## What to Remember

Use this mental model:

```text id="6q3m8x"
CAPABILITY
   ↓
WHAT POWER DOES IT GRANT?
   ↓
WHICH BINARY HAS IT?
   ↓
WHAT DOES THE BINARY DO?
   ↓
WHAT CAN I CONTROL?
   ↓
CAN MY CONTROL REACH THE PRIVILEGED OPERATION?
   ↓
VALIDATE
   ↓
EXPLOIT
   ↓
VERIFY
```

> **A capability is a privileged permission, not automatically a privilege-escalation path. The decisive step is connecting that permission to user-controlled program behavior and a meaningful privilege boundary.**
