# Linux Privilege Escalation Command Reference

> **Purpose:** Provide a structured command reference for Linux privilege-escalation enumeration, investigation, validation, and verification. Use commands to answer specific questions rather than executing the entire list blindly.

---

## How to Use This Reference

The command reference is organized around questions:

```text id="1g4n8p"
IDENTITY
   ↓
SYSTEM
   ↓
USERS / GROUPS
   ↓
PRIVILEGE
   ↓
PROCESSES / SERVICES
   ↓
FILESYSTEM
   ↓
SCHEDULED EXECUTION
   ↓
CREDENTIALS
   ↓
NETWORK
   ↓
SPECIAL ENVIRONMENTS
   ↓
VULNERABLE SOFTWARE
   ↓
VALIDATION
   ↓
VERIFICATION
```

For every command, ask:

```text id="7x2m9k"
What question does this answer?
What result matters?
What should I check next?
```

---

## Identity

### Current User

```bash id="2x6m4v"
whoami
```

Answers:

```text id="q8j4s1"
Which account am I using?
```

---

### User and Group IDs

```bash id="9f3k7p"
id
```

Answers:

```text id="w5m8r2"
UID
GID
Supplementary groups
```

---

### UID Only

```bash id="6p4z1c"
id -u
```

Root is:

```text id="h7m2q9"
UID 0
```

---

### Primary Group

```bash id="4v8k3x"
id -g
```

---

### Groups

```bash id="n2c7y5"
groups
```

Use this to identify group-based access.

---

## System Information

### Hostname

```bash id="7q5m1v"
hostname
```

---

### Kernel

```bash id="k4x8p2"
uname -a
```

---

### Kernel Release

```bash id="w6m3j9"
uname -r
```

---

### Architecture

```bash id="9x5c2m"
uname -m
```

---

### Distribution

```bash id="3j8v6q"
cat /etc/os-release
```

---

### CPU Architecture Details

```bash id="m7p2z4"
lscpu
```

If available, this can provide:

* architecture
* CPU information
* virtualization details
* kernel-relevant characteristics

---

## Users

### Current User Database

```bash id="5n8x3q"
cat /etc/passwd
```

Look for:

```text id="c6v2m7"
Usernames
UIDs
Login shells
Service accounts
```

---

### Search for UID 0 Accounts

```bash id="1q7w5m"
awk -F: '$3 == 0 {print $1}' /etc/passwd
```

Question:

```text id="8m3r6x"
Which accounts have UID 0?
```

---

### Login Shells

```bash id="9z2k5p"
awk -F: '$7 !~ /(nologin|false)$/ {print $1 ":" $7}' /etc/passwd
```

This helps identify accounts configured with interactive shells.

---

## Groups

### Group Database

```bash id="4m7x9c"
cat /etc/group
```

---

### Group Membership

```bash id="6k2p8v"
id
groups
```

Look for groups that may grant access to:

```text id="j5n3w8"
Containers
Devices
Virtualization
Storage
Administrative resources
```

Always verify what the group actually permits on the target.

---

## Sudo

### Sudo Permissions

```bash id="0r6x4m"
sudo -l
```

Questions:

```text id="y8p2k5"
What can I execute?
As which user?
With what restrictions?
```

---

### Sudo Version

```bash id="5v9m2q"
sudo --version
```

Useful when investigating sudo-specific behavior or version-dependent issues.

---

## Processes

### Full Process Listing

```bash id="7m3x8k"
ps aux
```

---

### Process Tree

```bash id="2q6v9p"
ps auxf
```

A process tree can help identify:

```text id="x8m4r1"
Parent-child relationships
Service launch chains
Scripts
Interpreters
```

---

### Alternative Process Format

```bash id="9p5k3m"
ps -ef
```

---

### Process Search

```bash id="4x7n2v"
ps aux | grep -i <keyword>
```

Avoid treating the `grep` process itself as a finding.

---

## Process Details

For a known PID:

```bash id="6m8q1x"
cat /proc/<PID>/status
```

Executable:

```bash id="7v2p5k"
readlink -f /proc/<PID>/exe
```

Command line:

```bash id="3x9m6q"
tr '\0' ' ' < /proc/<PID>/cmdline
```

Environment, if permitted:

```bash id="8k4n2v"
tr '\0' '\n' < /proc/<PID>/environ
```

Ask:

```text id="p6m1z8"
Who owns the process?
What executable is running?
What arguments are used?
What environment matters?
```

---

## Services

### Running Services

```bash id="1m7q5x"
systemctl list-units --type=service --state=running
```

---

### All Services

```bash id="6v2p8m"
systemctl list-unit-files --type=service
```

---

### Service Status

```bash id="4k9x3q"
systemctl status <service>
```

---

### Service Definition

```bash id="8m5v2p"
systemctl cat <service>
```

---

### Service Properties

```bash id="3q7n6x"
systemctl show <service>
```

Useful properties include:

```text id="p5x8m1"
User
Group
ExecStart
Environment
WorkingDirectory
```

---

## SUID

### Find SUID Files

```bash id="9k2m5v"
find / -perm -4000 -type f 2>/dev/null
```

---

### Inspect a Candidate

```bash id="4x6q8n"
ls -la <binary>
file <binary>
```

Questions:

```text id="s3m7p2"
Who owns it?
What is it?
Can I execute it?
Does it expose controllable privileged behavior?
```

---

## SGID

### Find SGID Files

```bash id="7m4p9x"
find / -perm -2000 -type f 2>/dev/null
```

---

## Capabilities

### Recursive Capability Search

```bash id="2x8k5m"
getcap -r / 2>/dev/null
```

---

### Specific Binary

```bash id="6p3v9q"
getcap <binary>
```

Questions:

```text id="1m8x5k"
Which capability?
Why does the binary have it?
Can I execute it?
What privileged operation can it perform?
```

---

## Filesystem

### Root Directory

```bash id="8q4m6x"
ls -la /
```

---

### Important Application Directories

```bash id="5v2n8p"
ls -la /opt/
ls -la /usr/local/
```

---

### Temporary Directory

```bash id="3x7k1m"
ls -la /tmp/
```

---

### File Metadata

```bash id="9m4p6x"
stat <file>
```

---

### File Permissions

```bash id="7q2v5k"
ls -la <file>
```

---

### Directory Permissions

```bash id="4m8x3p"
ls -ld <directory>
```

---

### Find Writable Files

A targeted example:

```bash id="6x9p2m"
find /opt /usr/local /var/tmp /tmp -type f -writable 2>/dev/null
```

Use targeted paths where possible to reduce unnecessary output.

---

### Find Writable Directories

```bash id="2k7m5v"
find /opt /usr/local /var/tmp /tmp -type d -writable 2>/dev/null
```

Then ask:

```text id="9x3p6m"
Who consumes resources from this directory?
```

---

## Interesting Files

### Configuration Files

```bash id="5m8q2x"
find /etc /opt /usr/local -type f \
  \( -name "*.conf" -o -name "*.cfg" -o -name "*.ini" \) \
  2>/dev/null
```

---

### Backup Files

```bash id="7p4m9x"
find /etc /opt /usr/local /var \
  -type f \
  \( -name "*.bak" -o -name "*.backup" -o -name "*.old" \) \
  2>/dev/null
```

---

### Scripts

```bash id="3x6k8m"
find /opt /usr/local /var /home \
  -type f \
  \( -name "*.sh" -o -name "*.py" -o -name "*.pl" \) \
  2>/dev/null
```

Treat script discovery as a starting point, not a vulnerability.

---

## File Ownership

```bash id="8m5q1v"
find /path -type f -user root -ls 2>/dev/null
```

Combine ownership with:

```text id="1x7p4m"
Who can modify it?
Who executes it?
```

---

## Scheduled Execution

### Cron Configuration

```bash id="6q3m8x"
cat /etc/crontab
```

---

### Cron Directory

```bash id="2v9p5k"
ls -la /etc/cron.d/
```

---

### Periodic Cron Directories

```bash id="4m7x2q"
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
ls -la /etc/cron.weekly/
ls -la /etc/cron.monthly/
```

---

### Systemd Timers

```bash id="9p4m6x"
systemctl list-timers --all
```

---

### Timer Definition

```bash id="7x2k5m"
systemctl cat <timer>
```

Trace the associated service.

---

## PATH

### Current PATH

```bash id="5m8q3v"
echo "$PATH"
```

---

### PATH Components

```bash id="2x7p9m"
printf '%s\n' "$PATH" | tr ':' '\n'
```

Ask:

```text id="6k4m8x"
Which directories are trusted?
Who can write to them?
Does privileged execution depend on them?
```

---

## Environment

### Environment Variables

```bash id="8p3m6q"
env
```

---

### Shell

```bash id="4x7k2m"
echo "$SHELL"
```

---

### Home Directory

```bash id="9m5p8v"
echo "$HOME"
```

Environment values become important when a privileged process explicitly trusts them.

---

## Command History

```bash id="3q6x9m"
history 2>/dev/null
```

Potentially interesting items include:

```text id="7m2p5k"
Passwords
Tokens
Administrative commands
Internal paths
Credentials
```

Do not expose secrets unnecessarily.

---

## SSH

### SSH Directory

```bash id="6x4m8p"
ls -la ~/.ssh/
```

Potential files:

```text id="2q7m5v"
authorized_keys
config
known_hosts
private keys
```

A discovered key must be evaluated in context.

---

## Network

### Interfaces

```bash id="8m3x6q"
ip addr
```

---

### Routes

```bash id="5p7k2m"
ip route
```

---

### Listening TCP/UDP Services

```bash id="9x4m8v"
ss -lntup
```

If permission limits process information, use:

```bash id="3m6q2x"
ss -lntu
```

---

### Active Connections

```bash id="7p5m9k"
ss -antup
```

---

## Localhost Services

Identify listeners:

```bash id="2x8m4q"
ss -lntup
```

Then investigate services bound to:

```text id="j5m7p2"
127.0.0.1
::1
```

Ask:

```text id="6x3m9v"
Who can access it?
Who owns it?
What privilege does it have?
What input does it accept?
```

---

## Mounts

### Mounted Filesystems

```bash id="8q5m2x"
findmnt
```

---

### Filesystem Table

```bash id="4x7p9m"
cat /etc/fstab
```

Look for:

```text id="m6k2v8"
NFS
Writable mounts
Unusual options
Application mounts
Container-related mounts
```

---

## Docker

### Group Context

```bash id="7m3x5q"
id
```

### Running Containers

```bash id="9p6m2x"
docker ps
```

### All Containers

```bash id="2x8k4m"
docker ps -a
```

### Docker Socket

```bash id="5q7m3v"
ls -l /var/run/docker.sock 2>/dev/null
```

The important question is:

```text id="8m4x6p"
What host-level control does runtime access provide?
```

---

## LXD / Container Runtime

First determine whether the runtime exists and whether the current user can interact with it.

Then analyze:

```text id="3x9m5q"
User
 ↓
Runtime access
 ↓
Container privileges
 ↓
Host interaction
```

Do not assume all installations have identical security properties.

---

## NFS

Identify relevant configuration:

```bash id="6m2x8p"
cat /etc/exports 2>/dev/null
```

Then consider:

```text id="7q4m9x"
Exported paths
Client restrictions
Export options
File ownership
Privilege relationships
```

---

## Kernel / Software Context

### Kernel

```bash id="5x8m3q"
uname -a
uname -r
```

### OS

```bash id="2m7p4x"
cat /etc/os-release
```

### Architecture

```bash id="9q5m6v"
uname -m
```

Use this information to determine whether a specific vulnerability is applicable.

---

## Package Enumeration

### Debian / Ubuntu

```bash id="4x7m2p"
dpkg -l
```

Search:

```bash id="6m9q3x"
dpkg -l | grep -i <keyword>
```

### RPM-Based Systems

```bash id="8p5m4v"
rpm -qa
```

Use the package manager appropriate to the target.

---

## Process Monitoring

If event-driven execution needs investigation, a process-monitoring utility such as `pspy` can help observe processes created during normal system activity.

Reasoning:

```text id="3m7x9q"
Unknown execution
      ↓
Observe process
      ↓
Identify command
      ↓
Identify user
      ↓
Trace resource
      ↓
Validate
```

---

## Automated Enumeration

Useful tools:

```text id="7x4m2p"
LinPEAS
Linux Smart Enumeration
pspy
```

Recommended workflow:

```text id="8m6q3v"
Manual Baseline
      ↓
Automated Tool
      ↓
Candidate Finding
      ↓
Manual Reproduction
      ↓
Validation
```

---

## Search Helpers

### Search Files by Name

```bash id="2q8m5x"
find /path -name "filename" 2>/dev/null
```

---

### Search Case-Insensitive

```bash id="6m4x9p"
find /path -iname "*keyword*" 2>/dev/null
```

---

### Search Text in Files

```bash id="9x3m7q"
grep -Rni "keyword" /path 2>/dev/null
```

Use targeted paths to avoid unnecessary output.

---

### Locate Executable

```bash id="4m8p2x"
command -v <command>
```

---

### Resolve Symlink

```bash id="7q5m3v"
readlink -f <path>
```

---

## Permissions and Ownership

### Long Listing

```bash id="2x6m8q"
ls -la <path>
```

### Numeric Permissions

```bash id="5m9p3x"
stat -c '%A %a %U %G %n' <file>
```

This shows:

```text id="8q4m7v"
Symbolic permissions
Numeric permissions
Owner
Group
Path
```

---

## File Type

```bash id="3x7m5q"
file <path>
```

Useful for distinguishing:

```text id="9m2p6x"
ELF binaries
Scripts
Archives
Text files
Symlinks
Libraries
```

---

## Symlinks

### Inspect

```bash id="6q8m4x"
ls -la <link>
readlink -f <link>
```

Ask:

```text id="5m7p2q"
Who controls the link?
Who follows it?
What privilege does the consumer have?
```

---

## Privilege Verification

### Current User

```bash id="8x3m6q"
whoami
```

### Full Identity

```bash id="4p7m2x"
id
```

### UID

```bash id="9m5q8v"
id -u
```

### Process Status

```bash id="2x6m4p"
grep -E '^(Uid|Gid):' /proc/self/status
```

---

## Finding Validation

For a candidate finding, use:

```text id="7m3x9q"
1. Reproduce
2. Identify owner
3. Identify execution user
4. Identify user control
5. Trace execution
6. Confirm privilege impact
7. Perform minimal controlled validation
```

Useful supporting commands:

```bash id="5x8m2v"
ls -la <resource>
stat <resource>
ps aux
systemctl cat <service>
sudo -l
```

---

## Attack Path Construction

Use:

```text id="8q4m6x"
CURRENT USER
      ↓
CONTROLLED RESOURCE
      ↓
TRUSTED COMPONENT
      ↓
PRIVILEGED EXECUTION
      ↓
PRIVILEGE IMPACT
```

For each arrow, identify evidence.

---

## Common Command Combinations

### Identity + System

```bash id="3m7x9p"
whoami
id
hostname
uname -a
cat /etc/os-release
```

### Privilege Mechanisms

```bash id="6x2m8q"
sudo -l
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
getcap -r / 2>/dev/null
```

### Processes + Services

```bash id="9p4m5x"
ps aux
systemctl list-units --type=service --state=running
```

### Scheduled Execution

```bash id="7m8q2v"
cat /etc/crontab
ls -la /etc/cron.d/
systemctl list-timers --all
```

### Network

```bash id="4x6m9p"
ip addr
ip route
ss -lntup
```

### Mounts

```bash id="2m7x5q"
findmnt
cat /etc/fstab
```

---

## Commands That Need Caution

Some commands can generate substantial output or interact with sensitive information.

Use targeted paths and filtering where possible for:

```text id="8p3m6x"
find /
grep -R /
cat sensitive configuration
environment inspection
process environment inspection
```

Avoid unnecessary disclosure of:

```text id="6q4m9v"
Passwords
Private keys
API tokens
Session tokens
Other secrets
```

---

## Command Selection Rule

Before running a command, complete this sentence:

> **"I am running this because I need to know ______."**

Examples:

```text id="3x8m5q"
"I need to know which groups I belong to."
        ↓
id

"I need to know which services are running."
        ↓
systemctl list-units --type=service --state=running

"I need to know which files are SUID."
        ↓
find / -perm -4000 -type f 2>/dev/null
```

This simple habit prevents command-dump enumeration.

---

## Final Quick Reference

```text id="7m2q8x"
IDENTITY
whoami
id

SYSTEM
hostname
uname -a
cat /etc/os-release
uname -m

SUDO
sudo -l

PROCESSES
ps aux
ps -ef

SERVICES
systemctl list-units --type=service --state=running
systemctl status <service>
systemctl cat <service>

SUID
find / -perm -4000 -type f 2>/dev/null

SGID
find / -perm -2000 -type f 2>/dev/null

CAPABILITIES
getcap -r / 2>/dev/null

CRON
cat /etc/crontab
ls -la /etc/cron.d/

TIMERS
systemctl list-timers --all

FILES
ls -la <path>
stat <file>
file <path>

NETWORK
ip addr
ip route
ss -lntup

MOUNTS
findmnt

ENVIRONMENT
env
echo "$PATH"

CREDENTIAL CONTEXT
history 2>/dev/null
ls -la ~/.ssh/

CONTAINERS
docker ps
ls -l /var/run/docker.sock

VERIFICATION
whoami
id
id -u
```

---

## Final Mental Model

Do not memorize this file as a list of commands.

Memorize the questions:

```text id="4x7m9q"
WHO AM I?
     ↓
WHAT IS RUNNING?
     ↓
WHAT IS PRIVILEGED?
     ↓
WHO OWNS IT?
     ↓
WHO CAN MODIFY IT?
     ↓
WHAT DOES IT TRUST?
     ↓
WHEN DOES IT EXECUTE?
     ↓
CAN I INFLUENCE IT?
     ↓
CAN I VALIDATE THE PATH?
     ↓
DID PRIVILEGE ACTUALLY CHANGE?
```

> **Commands are tools for answering questions. The methodology determines which questions are worth asking.**
