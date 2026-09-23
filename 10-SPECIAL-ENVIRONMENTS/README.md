# 10 — Special Environments

Special environments can create privilege-escalation paths that do not look like traditional Linux permission problems.

Examples include:

* Docker
* LXD
* Containers
* NFS
* Mounted filesystems
* Privileged sockets
* Host-mounted directories
* Container management interfaces

The important principle is:

> **A special environment becomes interesting when it gives a low-privileged user control over a resource that has greater privilege or trust than the user's current account.**

---

## Core Question

> **Does this environment give my current user access to a more privileged execution context, filesystem, socket, or trust boundary?**

Do not stop at:

```text
"I am in the docker group."
```

Instead determine:

```text
What does this group control?
        ↓
What interface does it provide?
        ↓
What privilege does that interface have?
        ↓
Can I influence a privileged resource?
        ↓
Does that cross the Linux privilege boundary?
```

---

## Mental Model

Use this chain:

```text
SPECIAL ENVIRONMENT
        ↓
ACCESS CONTROL
        ↓
CONTROL INTERFACE
        ↓
PRIVILEGED RESOURCE
        ↓
CAN CURRENT USER INFLUENCE IT?
        ↓
WHAT PRIVILEGE DOES IT PROVIDE?
        ↓
VALIDATE THE BOUNDARY
```

Common examples:

```text
Docker group
    ↓
Docker socket
    ↓
Container management
    ↓
Potential host-level impact
```

```text
LXD access
    ↓
Container management
    ↓
Privileged container capability
    ↓
Potential host filesystem interaction
```

```text
NFS export
    ↓
Remote filesystem access
    ↓
Export options
    ↓
Potential trust/permission weakness
```

```text
Mounted filesystem
    ↓
Accessible files
    ↓
Unexpected permissions or trust
    ↓
Potential privilege path
```

---

## 1. Identify Special Environments

Start by checking groups and available tools.

### 🔎 ENUMERATE

Check identity:

```bash id="n8p4w1"
id
```

Check groups:

```bash id="m3k7x5"
groups
```

Search for high-interest groups:

```bash id="c6q2v9"
grep -Ei '^(docker|lxd|disk|adm|libvirt|kvm):' /etc/group
```

Check installed commands:

```bash id="r5x8k2"
command -v docker
command -v lxc
command -v lxd
command -v mount
command -v showmount
```

Check mounted filesystems:

```bash id="v1m6q4"
findmnt
```

### 🧠 WHY

You are trying to determine whether the current user has access to:

* Container management
* Block devices
* Virtualization
* Filesystem mounts
* Network filesystems
* Privileged service interfaces

---

## 2. Docker

Docker can create a significant privilege boundary because Docker management access can allow control over containers and their resources.

### 🔎 ENUMERATE

Check whether Docker is installed:

```bash id="2c7n5m"
command -v docker
```

Check Docker version:

```bash id="7p4x1k"
docker --version
```

Check whether the daemon is running:

```bash id="9m3q6v"
systemctl status docker 2>/dev/null
```

Check the Docker socket:

```bash id="f8k2r5"
ls -la /var/run/docker.sock 2>/dev/null
```

Check Docker group membership:

```bash id="x4v7c1"
getent group docker
```

Check whether the current user can communicate with Docker:

```bash id="w6m2p8"
docker info
```

### 🧠 WHY

The important relationship is:

```text
Current user
      ↓
Docker access
      ↓
Docker daemon
      ↓
Container management
      ↓
Privileged host interaction
```

The Docker daemon itself normally operates with high privilege.

### ⚠️ LOOK FOR

High-interest indicators:

```text id="q1k8z4"
User belongs to docker group
```

```text id="a6m3v9"
Docker socket accessible
```

```text id="t7p2x5"
docker info works without elevated privileges
```

These are findings that require further validation.

---

## 3. Docker Socket

The Docker Unix socket is an important trust boundary.

### 🔎 ENUMERATE

```bash id="e4m9q2"
ls -la /var/run/docker.sock
```

Also check:

```bash id="j7x3c8"
ls -la /run/docker.sock 2>/dev/null
```

Determine socket ownership:

```bash id="b5n8w1"
stat /var/run/docker.sock 2>/dev/null
```

Check groups:

```bash id="z2q6k4"
getent group docker
```

### 🧠 WHY

A socket may look like a normal file, but it represents an IPC interface to a service.

The key question is:

> **Who can communicate with the privileged service through this socket?**

### ⚠️ LOOK FOR

Example relationship:

```text
docker.sock
    ↓
owned by root
    ↓
group docker
    ↓
current user belongs to docker
    ↓
Docker API access
```

The socket's permissions alone do not tell the whole story.

---

## 4. Docker Containers

Enumerate existing containers when authorized.

### 🔎 ENUMERATE

```bash id="m6q8r3"
docker ps
```

List all containers:

```bash id="y4c1v7"
docker ps -a
```

Inspect a container:

```bash id="p8x2m5"
docker inspect <container>
```

List images:

```bash id="v7k3q1"
docker images
```

List volumes:

```bash id="c9m4w6"
docker volume ls
```

List networks:

```bash id="r2x8n5"
docker network ls
```

### 🧠 WHY

You are looking for:

* Privileged containers
* Host filesystem mounts
* Sensitive volumes
* Docker socket mounts
* Host networking
* Sensitive environment variables
* Containers running as root

### ⚠️ LOOK FOR

Inspect container configuration for:

```text id="g6q2v9"
Privileged mode
Host mounts
Docker socket
Host network
Root user
Sensitive capabilities
Sensitive environment variables
```

---

## 5. Container Mounts

Container mounts can expose host resources.

### 🔎 ENUMERATE

Inspect mounts:

```bash id="k4v8m2"
docker inspect <container> \
--format '{{json .Mounts}}'
```

Inspect all configuration:

```bash id="x7n3q5"
docker inspect <container>
```

### 🧠 WHY

A container may have access to:

```text
Host directory
      ↓
Container mount
      ↓
Container process
```

The security impact depends on:

* Which host path is mounted
* Mount mode
* Container privilege
* Container user
* What the process can do with the mount

### ⚠️ LOOK FOR

High-interest relationships include:

```text id="w2c6m8"
/host/path → /container/path
```

especially when the host path contains sensitive system resources.

---

## 6. Docker Socket Mounted Inside a Container

A container may have access to the Docker API through the host socket.

### 🔎 ENUMERATE

Search container configuration:

```bash id="q5m8x1"
docker inspect <container> | grep -i docker.sock
```

More targeted:

```bash id="z4c7n2"
docker inspect <container> \
--format '{{json .Mounts}}' | grep -i docker.sock
```

### 🧠 WHY

The relationship becomes:

```text
Container
    ↓
/var/run/docker.sock
    ↓
Docker daemon
    ↓
Container management
```

Access to the Docker API can be equivalent to substantial control over the Docker host.

### ⚠️ LOOK FOR

```text
/var/run/docker.sock
```

mounted inside a container.

This is a high-value trust relationship to investigate.

---

## 7. Docker Privileged Containers

A container can be configured with elevated capabilities.

### 🔎 ENUMERATE

Inspect the container:

```bash id="h3w7p9"
docker inspect <container>
```

Search for privileged mode:

```bash id="n6q2x4"
docker inspect <container> | grep -i privileged
```

Inspect security configuration:

```bash id="b8m5c1"
docker inspect <container> | grep -Ei 'capadd|capdrop|securityopt|privileged'
```

### 🧠 WHY

Privileged containers have substantially different security boundaries from ordinary containers.

### ⚠️ LOOK FOR

```text
"Privileged": true
```

or unusually broad capabilities.

### ❌ COMMON MISTAKE

Do not assume:

```text
container = root on host
```

Container root and host root are not automatically the same security context.

The configuration determines the actual boundary.

---

## 8. Container Users

Determine which user a container process runs as.

### 🔎 ENUMERATE

Inspect configuration:

```bash id="j4r8q2"
docker inspect <container> \
--format '{{.Config.User}}'
```

Inspect processes:

```bash id="m7x3c9"
docker top <container>
```

### 🧠 WHY

You need to distinguish:

```text
Container root
```

from:

```text
Host root
```

and understand whether additional container privileges or host resources are available.

---

## 9. Docker Volumes

Volumes can contain sensitive information.

### 🔎 ENUMERATE

List volumes:

```bash id="t5n2w8"
docker volume ls
```

Inspect a volume:

```bash id="q8c4m1"
docker volume inspect <volume>
```

### ⚠️ LOOK FOR

Potentially sensitive data:

```text
Application configuration
Database data
Credentials
SSH material
Backups
Application secrets
```

### ➡️ NEXT STEP

Determine:

```text id="v3m7x9"
Who can access the volume?
Where is it mounted?
Which container uses it?
What privilege does that container have?
```

---

## 10. LXD

LXD is a Linux container manager.

Access to LXD can represent a significant trust boundary because LXD manages containers and interacts closely with host resources.

### 🔎 ENUMERATE

Check for LXD:

```bash id="k6x2p8"
command -v lxc
```

```bash id="n4m7c1"
command -v lxd
```

Check group membership:

```bash id="z8q3w5"
getent group lxd
```

Check current groups:

```bash id="r1v6k9"
id
```

Check LXD state:

```bash id="y5m2x7"
lxc list
```

### 🧠 WHY

The relationship to investigate is:

```text
Current user
      ↓
LXD access
      ↓
Container management
      ↓
Host resource access
```

### ⚠️ LOOK FOR

```text
Current user belongs to lxd
```

and:

```text
lxc list
```

works without additional authorization.

---

## 11. LXD Configuration

If LXD access exists, inspect the available configuration.

### 🔎 ENUMERATE

```bash id="f4x8m2"
lxc list
```

Inspect a container:

```bash id="p7c3n9"
lxc config show <container>
```

List profiles:

```bash id="m5w1q6"
lxc profile list
```

Inspect a profile:

```bash id="x2k7v4"
lxc profile show <profile>
```

### 🧠 WHY

You are looking for:

* Device mappings
* Disk devices
* Network interfaces
* Security configuration
* Privileged container settings
* Host filesystem exposure

### ⚠️ LOOK FOR

Configuration that connects:

```text
Container
    ↓
Host filesystem/device
```

or grants unusually broad container privileges.

---

## 12. Containers in General

Not every container runtime is Docker or LXD.

### 🔎 ENUMERATE

Check common tools:

```bash id="q6m3x8"
command -v podman
```

```bash id="v8k1c5"
command -v nerdctl
```

```bash id="t4r7n2"
command -v crictl
```

Check running processes:

```bash id="c2x9m6"
ps aux | grep -Ei '[c]ontainerd|[d]ockerd|[p]odman|[c]rio|[k]ubelet'
```

Check listening sockets:

```bash id="j5w8p3"
ss -lxnp
```

### 🧠 WHY

The important question is not:

```text
"Which container technology is installed?"
```

It is:

```text
"What container control interface is accessible to my user?"
```

---

## 13. NFS

Network File System can create important trust relationships between systems.

### 🔎 ENUMERATE

Check mounted NFS filesystems:

```bash id="n3m6x8"
findmnt -t nfs,nfs4
```

Check mounts:

```bash id="c7q2p5"
mount | grep -Ei 'nfs|nfs4'
```

Check NFS-related processes:

```bash id="w4x8k1"
ps aux | grep -Ei '[n]fs|[r]pc'
```

Check exported filesystems when authorized:

```bash id="m9v3c6"
showmount -e <server>
```

### 🧠 WHY

NFS security depends heavily on:

* Export configuration
* Client restrictions
* UID/GID mapping
* Root squashing
* Network trust
* Mount permissions

### ⚠️ LOOK FOR

Potentially interesting conditions:

```text id="f2q7m4"
Writable export
        +
Weak client restrictions
```

or:

```text id="x6n1c8"
Unexpected filesystem access
        +
Privileged files
```

---

## 14. NFS Export Options

If an NFS export is discovered, determine how it is configured.

### 🔎 ENUMERATE

On an authorized NFS server:

```bash id="k3w8p2"
cat /etc/exports
```

Inspect export information:

```bash id="m6x1q4"
exportfs -v
```

### 🧠 WHY

Important export options include:

```text
ro
rw
root_squash
no_root_squash
sync
async
```

The exact security impact depends on the complete configuration.

### ⚠️ LOOK FOR

A particularly important configuration relationship is:

```text
NFS export
     ↓
Writable
     ↓
Client access
     ↓
Root mapping behavior
     ↓
Can privileged files be influenced?
```

### ❌ COMMON MISTAKE

Do not conclude:

```text
NFS = privilege escalation
```

NFS itself is not the vulnerability.

The export configuration and trust relationship determine the security impact.

---

## 15. Mounted Filesystems

A mounted filesystem may expose unexpected resources.

### 🔎 ENUMERATE

List mounts:

```bash id="h8m2q5"
findmnt
```

Show filesystem types:

```bash id="w3x7c1"
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS
```

Show disk usage:

```bash id="p6k9m4"
df -h
```

Show mount information:

```bash id="v2c8n5"
cat /proc/mounts
```

### 🧠 WHY

Look for:

* NFS
* CIFS
* External disks
* Bind mounts
* Container mounts
* Unusual filesystem types
* Writable privileged locations

### ⚠️ LOOK FOR

```text id="r5m1x8"
Mount
  ↓
Who owns mounted files?
  ↓
What permissions apply?
  ↓
Can current user modify them?
  ↓
Who trusts those files?
```

---

## 16. Block Devices and Special Groups

Some groups provide access to powerful system resources.

### 🔎 ENUMERATE

Check groups:

```bash id="x8q2m5"
id
```

```bash id="n4c7w1"
getent group disk
```

Check devices:

```bash id="m7p3x9"
ls -la /dev
```

Look for block devices:

```bash id="t2k6v4"
ls -la /dev | grep -E 'sd[a-z]|nvme|vd[a-z]'
```

### 🧠 WHY

Direct device access can bypass normal filesystem permissions.

### ⚠️ LOOK FOR

Membership in groups with powerful device access.

For example:

```text
disk
```

requires careful investigation because device access can expose raw storage.

### ❌ COMMON MISTAKE

Do not modify raw block devices during validation.

First establish exactly what access exists and what the security impact would be.

---

## 17. Virtualization Groups

Check for virtualization-related group membership.

### 🔎 ENUMERATE

```bash id="q9x3m7"
getent group libvirt
```

```bash id="z5c8n2"
getent group kvm
```

Check installed tools:

```bash id="w1m6p4"
command -v virsh
```

```bash id="f7k2c9"
command -v qemu-system-x86_64
```

### 🧠 WHY

Virtualization management can provide access to:

* Virtual machines
* VM disks
* Virtual hardware
* Host resources

The actual impact depends on the configuration and permissions.

---

## 18. Privileged Unix Sockets

Unix sockets are often overlooked.

### 🔎 ENUMERATE

List sockets:

```bash id="b4m8x2"
ss -lxnp
```

Search common socket directories:

```bash id="j6q3v9"
find /run /var/run /tmp -type s -ls 2>/dev/null
```

Inspect a suspicious socket:

```bash id="r8c1m5"
ls -la /path/to/socket
```

### 🧠 WHY

A Unix socket may provide access to a privileged service.

Examples include:

```text
Docker
Database
Container runtime
Management daemon
Application service
```

### ⚠️ LOOK FOR

```text id="p2x7k4"
Privileged service
       ↓
Unix socket
       ↓
Current user can connect
       ↓
Powerful API
```

This is often more important than the socket filename itself.

---

## 19. Mounted Host Resources

Special environments may expose host resources directly.

### 🔎 ENUMERATE

For Docker:

```bash id="x5m9c2"
docker inspect <container>
```

For LXD:

```bash id="v3k7q1"
lxc config show <container>
```

For mounts:

```bash id="n8p4w6"
findmnt
```

### ⚠️ LOOK FOR

Relationships such as:

```text
Host filesystem
      ↓
Container mount
      ↓
Privileged process
```

or:

```text
Host device
      ↓
Container/device access
      ↓
Container process
```

The exact security impact depends on what resource is exposed and what controls exist around it.

---

## 20. Special Environment Decision Tree

```text id="7m3q8x"
                  START
                    │
                    ▼
          CHECK GROUPS / TOOLS
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      Docker       LXD         NFS
        │           │           │
        ▼           ▼           ▼
      Socket      LXD ACL     Export
        │           │           │
        ▼           ▼           ▼
     Container   Container    Mount
        │           │           │
        └───────────┼───────────┘
                    ▼
             PRIVILEGED RESOURCE?
                │           │
               NO          YES
                │           │
                ▼           ▼
             MOVE ON   CAN I CONTROL IT?
                              │
                              ▼
                       WHAT PRIVILEGE?
                              │
                              ▼
                       VALIDATE BOUNDARY
```

---

## 21. Finding vs Vulnerability vs Exploitable Path

### Finding

```text id="q4m8x2"
Current user belongs to the docker group.
```

This is an observation.

### Vulnerability

```text id="n7c2v5"
Docker management access allows the low-privileged user
to control privileged container resources.
```

Now there is a security weakness to analyze.

### Exploitable Path

```text id="x5k9m3"
The available Docker configuration provides a practical,
authorized path from the current user to greater host privilege.
```

The actual path must be validated against the specific environment.

---

## 22. Safe Validation

Before exploitation, document the trust boundary.

### 🧪 VALIDATE

For Docker:

```text id="d6q1m8"
[ ] Docker installed
[ ] Docker daemon running
[ ] Socket accessible
[ ] Current user has Docker access
[ ] Container configuration understood
[ ] Host resources identified
[ ] Privilege boundary identified
```

For LXD:

```text id="f3x8k2"
[ ] LXD installed
[ ] Current user has LXD access
[ ] Container configuration understood
[ ] Profiles understood
[ ] Host resources identified
[ ] Privilege boundary identified
```

For NFS:

```text id="m9c4w7"
[ ] Export identified
[ ] Export options identified
[ ] Client restrictions understood
[ ] UID/GID behavior understood
[ ] Root mapping understood
[ ] Writable resources identified
```

For mounted filesystems:

```text id="r2v6n8"
[ ] Mount identified
[ ] Filesystem type identified
[ ] Ownership identified
[ ] Permissions identified
[ ] Trusted consumer identified
[ ] Privilege boundary identified
```

---

## 23. Common Mistakes

### ❌ Mistake 1 — Assuming Container Root Equals Host Root

These are different security contexts.

### ❌ Mistake 2 — Treating Group Membership as the Final Finding

A group is only the beginning.

Ask:

```text
What does the group control?
```

### ❌ Mistake 3 — Ignoring Unix Sockets

A privileged API may be exposed through a Unix socket rather than a TCP port.

### ❌ Mistake 4 — Assuming Every NFS Export Is Vulnerable

The export options and trust relationships determine the impact.

### ❌ Mistake 5 — Ignoring Mount Options

Always inspect:

```bash id="w5k2m8"
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS
```

### ❌ Mistake 6 — Modifying Host Resources During Enumeration

Enumeration should first establish the relationship.

Do not alter production or unauthorized systems.

### ❌ Mistake 7 — Exploiting Before Mapping the Boundary

First determine:

```text
User
 ↓
Interface
 ↓
Resource
 ↓
Privilege
 ↓
Boundary
```

---

## 24. Five-Minute Special Environment Check

### 🔎 ENUMERATE

```bash id="c8m2x5"
id
```

```bash id="n7q4w1"
command -v docker lxc lxd podman virsh
```

```bash id="r3k8p6"
ss -lxnp
```

```bash id="y5m1v9"
findmnt
```

```bash id="h2x7c4"
findmnt -t nfs,nfs4
```

Check Docker:

```bash id="m8q3k1"
ls -la /var/run/docker.sock 2>/dev/null
```

```bash id="p4x9n2"
docker info 2>/dev/null
```

Check LXD:

```bash id="v6c2m8"
getent group lxd
```

```bash id="z1w7q5"
lxc list 2>/dev/null
```

Then ask:

```text id="b9m4x2"
Special environment found?
        │
       YES
        │
        ▼
What interface is accessible?
        │
        ▼
What privileged resource does it control?
        │
        ▼
Can my user influence that resource?
        │
        ▼
What privilege does it provide?
        │
        ▼
Validate
```

---

## 25. Quick Command Reference

| Goal                 | Command                                           |
| -------------------- | ------------------------------------------------- |
| Current groups       | `id`                                              |
| Docker binary        | `command -v docker`                               |
| Docker version       | `docker --version`                                |
| Docker socket        | `ls -la /var/run/docker.sock`                     |
| Docker group         | `getent group docker`                             |
| Docker containers    | `docker ps -a`                                    |
| Docker images        | `docker images`                                   |
| Docker volumes       | `docker volume ls`                                |
| Docker networks      | `docker network ls`                               |
| Docker inspection    | `docker inspect <container>`                      |
| LXD binary           | `command -v lxc`                                  |
| LXD group            | `getent group lxd`                                |
| LXD containers       | `lxc list`                                        |
| LXD config           | `lxc config show <container>`                     |
| LXD profiles         | `lxc profile list`                                |
| Unix sockets         | `ss -lxnp`                                        |
| Find sockets         | `find /run /var/run /tmp -type s -ls 2>/dev/null` |
| Mounts               | `findmnt`                                         |
| Mount options        | `findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS`         |
| NFS mounts           | `findmnt -t nfs,nfs4`                             |
| NFS mounts via mount | `mount \| grep -Ei 'nfs\|nfs4'`                   |
| NFS exports          | `showmount -e <server>`                           |
| Export config        | `cat /etc/exports`                                |
| Export details       | `exportfs -v`                                     |
| Block devices        | `ls -la /dev`                                     |
| KVM group            | `getent group kvm`                                |
| Libvirt group        | `getent group libvirt`                            |

---

## 26. Complete Checklist

### 🔎 Environment Discovery

```text
[ ] Check groups
[ ] Check Docker
[ ] Check LXD
[ ] Check other container runtimes
[ ] Check virtualization tools
[ ] Check Unix sockets
[ ] Check mounted filesystems
[ ] Check NFS
[ ] Check special device access
```

### 🧠 Trust Analysis

```text
[ ] Identify control interface
[ ] Identify owner
[ ] Identify privileged resource
[ ] Identify accessible socket/API
[ ] Identify container privileges
[ ] Identify mounted host resources
[ ] Identify filesystem permissions
[ ] Identify trust relationship
```

### 🧪 Validation

```text
[ ] Confirm current user access
[ ] Confirm actual service/interface
[ ] Confirm privileged resource
[ ] Confirm configuration
[ ] Confirm possible influence
[ ] Confirm resulting privilege
[ ] Confirm privilege boundary
[ ] Record evidence
```

---

## 💡 Remember

Do not memorize:

```text
Docker = root
LXD = root
NFS = vulnerable
```

Instead memorize:

```text
WHO AM I?
     ↓
WHAT SPECIAL ENVIRONMENT EXISTS?
     ↓
WHAT INTERFACE CAN I ACCESS?
     ↓
WHAT RESOURCE DOES IT CONTROL?
     ↓
WHO OWNS THAT RESOURCE?
     ↓
CAN I INFLUENCE IT?
     ↓
WHAT PRIVILEGE DOES IT PROVIDE?
     ↓
DOES IT CROSS THE PRIVILEGE BOUNDARY?
```

The key principle is:

> **Special environments matter because they can expose powerful trust boundaries that ordinary file-permission enumeration may miss.**

---

## Where Findings Lead

Special-environment findings commonly connect to:

```text
Docker
  ↓
Container Configuration
  ↓
Privileged Resource
  ↓
Finding Validation
```

```text
LXD
  ↓
Container Management
  ↓
Host Resource Access
  ↓
Finding Validation
```

```text
NFS
  ↓
Export Configuration
  ↓
Filesystem Trust
  ↓
Finding Validation
```

```text
Mounted Filesystem
  ↓
Unexpected Permissions
  ↓
Trusted Resource
  ↓
Execution / Credential / Filesystem Path
```

Related stages:

* `02-SYSTEM-ENUMERATION/` — groups and installed software
* `03-FILESYSTEM-ENUMERATION/` — permissions and mounts
* `05-NETWORK-ENUMERATION/` — listening services and Unix sockets
* `06-PRIVILEGE-MECHANISMS/` — special groups and privileged access
* `08-CREDENTIALS-AND-SECRETS/` — secrets exposed by applications and containers
* `11-VULNERABLE-SOFTWARE/` — vulnerable container/runtime software
* `13-FINDING-VALIDATION/` — validate the actual privilege boundary
* `14-EXPLOITATION/` — authorized exploitation of validated paths

---

## Final Mental Model

```text id="q7m3x9"
             LOW-PRIVILEGED USER
                     │
                     ▼
          SPECIAL ENVIRONMENT FOUND
                     │
                     ▼
             CONTROL INTERFACE
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        Docker      LXD        NFS
          │          │          │
          └──────────┼──────────┘
                     ▼
             PRIVILEGED RESOURCE
                     │
                     ▼
             CAN USER CONTROL IT?
                  │       │
                 NO      YES
                  │       │
                  ▼       ▼
               MOVE ON  VALIDATE
                           │
                           ▼
                    WHAT PRIVILEGE?
                           │
                           ▼
                  BOUNDARY CROSSED?
                       │       │
                      NO      YES
                       │       │
                       ▼       ▼
                    MOVE ON  VERIFY
```

> **Don't ask only what special technology is installed. Ask what privileged control that technology gives you.**
