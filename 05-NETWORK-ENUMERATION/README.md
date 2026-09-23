# 05 — Network Enumeration

> **Goal:** Understand the host's network position, identify reachable services, discover locally exposed services, and determine whether network trust or service exposure creates a privilege-escalation opportunity.

---

## 🎯 Core Question

> **What can this machine communicate with, what is listening, and does any of it create a path to higher privilege?**

Network enumeration during Linux privilege escalation is different from external reconnaissance.

You are already on the machine.

The objective is to understand the machine's **local network context**:

```text
WHO AM I?
    ↓
WHAT INTERFACES EXIST?
    ↓
WHAT NETWORKS CAN I REACH?
    ↓
WHAT SERVICES ARE LISTENING?
    ↓
WHAT IS ONLY AVAILABLE LOCALLY?
    ↓
WHO CAN ACCESS THOSE SERVICES?
    ↓
DO ANY SERVICES TRUST THIS HOST/USER?
    ↓
CAN THAT TRUST LEAD TO PRIVILEGE ESCALATION?
```

---

## 🧠 Why Network Enumeration Matters

A compromised Linux host may expose services that are:

* available only on `localhost`
* available on a private interface
* available on all interfaces
* running as `root`
* running as another privileged account
* configured with weak authentication
* communicating with trusted internal systems
* connected to containers
* exposing management interfaces
* using unusual ports

A service that is invisible from an external scan may be extremely useful once you already have a shell.

Example:

```text
External attacker
       X
       │
       │ cannot reach
       ▼
127.0.0.1:8080
       ▲
       │
Your shell
       │
       ▼
Local service
```

This is why **localhost enumeration matters**.

---

## 1. Identify Network Interfaces

Start with the network interfaces.

### 🔎 ENUMERATE

```bash
ip addr
```

A shorter version:

```bash
ip -br addr
```

### WHY

You want to understand:

* Which interfaces exist
* Which interfaces are active
* IPv4 addresses
* IPv6 addresses
* Loopback
* Private networks
* Container interfaces
* Virtual interfaces

### LOOK FOR

Example:

```text
lo       UNKNOWN        127.0.0.1/8
eth0     UP             10.10.10.25/24
```

This immediately tells you:

```text
Loopback = 127.0.0.1
Host IP  = 10.10.10.25
Network  = 10.10.10.0/24
```

---

## 2. Understand the Loopback Interface

### 🔎 ENUMERATE

```bash
ip addr show lo
```

Normally you will see:

```text
127.0.0.1
```

### 🧠 WHY IT MATTERS

Services bound to:

```text
127.0.0.1
```

are normally accessible only from the local machine.

For example:

```text
127.0.0.1:8080
```

may not be reachable from another machine but **is reachable from your current shell**.

This makes localhost services an important privilege-escalation investigation target.

---

## 3. Identify the Default Route

### 🔎 ENUMERATE

```bash
ip route
```

Or:

```bash
ip route show
```

### WHY

The routing table tells you where traffic is sent.

Example:

```text
default via 10.10.10.1 dev eth0
10.10.10.0/24 dev eth0 proto kernel scope link src 10.10.10.25
```

Interpretation:

```text
Default gateway = 10.10.10.1
Local network   = 10.10.10.0/24
Host address    = 10.10.10.25
```

### LOOK FOR

* Default gateway
* Internal networks
* Multiple interfaces
* Unexpected routes
* VPN interfaces
* Container networks

---

## 4. Inspect Routes in More Detail

### 🔎 ENUMERATE

```bash
ip route show table main
```

If available:

```bash
ip -6 route
```

### WHY

IPv6 can sometimes reveal additional network connectivity that you would miss if you only inspect IPv4.

Do not assume IPv6 is irrelevant.

---

## 5. Identify the Host's IP Addresses

### 🔎 ENUMERATE

```bash
hostname -I
```

Or:

```bash
ip -br addr
```

### WHY

This gives you a quick overview of the addresses assigned to the host.

### LOOK FOR

Multiple addresses can indicate:

```text
multiple network interfaces
VPN connectivity
containers
virtual networks
management networks
```

---

## 6. Identify Interface Details

### 🔎 ENUMERATE

```bash
ip link
```

### WHY

This identifies interfaces even when they do not have an IP address.

You may discover:

```text
eth0
ens33
enp0s3
lo
docker0
tun0
br0
vethXXXX
```

### 🧠 INTERPRET

Common examples:

```text
tun0
```

may indicate a VPN or tunnel.

```text
docker0
```

may indicate Docker networking.

```text
br0
```

may indicate a bridge.

```text
vethXXXX
```

often indicates container/network namespaces.

These are **clues**, not automatic vulnerabilities.

---

## 7. Identify Listening Services

This is one of the highest-value network checks.

### 🔎 ENUMERATE

```bash
ss -tulpn
```

### WHY

This can show:

* TCP listeners
* UDP listeners
* Local addresses
* Ports
* Process information

Example:

```text
tcp   LISTEN   0   128   127.0.0.1:8080   0.0.0.0:*   users:(("python3",pid=1234,fd=3))
```

This tells you:

```text
Service  = python3
PID      = 1234
Port     = 8080
Binding  = 127.0.0.1
```

---

## 8. Understand Service Binding

A service's bind address is extremely important.

Consider:

```text
127.0.0.1:8080
```

versus:

```text
0.0.0.0:8080
```

### `127.0.0.1`

Usually means:

```text
local machine only
```

### `0.0.0.0`

Usually means:

```text
all IPv4 interfaces
```

### Specific IP

Example:

```text
10.10.10.25:8080
```

Usually means the service is bound to that interface/address.

### 🧠 Remember

> **Port number tells you what port is used. Bind address tells you who can potentially reach it.**

---

## 9. Separate TCP and UDP Listeners

### 🔎 TCP

```bash
ss -ltnp
```

### 🔎 UDP

```bash
ss -lunp
```

### WHY

Some services use UDP rather than TCP.

You do not want your enumeration to depend on one protocol.

---

## 10. Show Listening Services Without Process Names

If process information is unavailable because of permissions:

```bash
ss -ltn
```

And:

```bash
ss -lun
```

### WHY

Even without process names, ports and addresses still provide useful information.

You can investigate the owning process separately.

---

## 11. Identify the Process Behind a Port

Suppose you find:

```text
127.0.0.1:8080
```

Search the process list:

```bash
ps aux | grep '[8]080'
```

Better, if the PID is visible from `ss`:

```bash
ps -p <PID> -o pid,ppid,user,group,args
```

Then:

```bash
readlink -f /proc/<PID>/exe
```

### WHY

You want to connect:

```text
PORT
  ↓
PROCESS
  ↓
USER
  ↓
EXECUTABLE
  ↓
CONFIGURATION
```

This turns a network observation into a process/service investigation.

---

## 12. Check Common Localhost Services

If you find:

```text
127.0.0.1:8080
```

test whether it actually responds.

For HTTP:

```bash
curl -I http://127.0.0.1:8080
```

If that works:

```bash
curl -s http://127.0.0.1:8080
```

### WHY

A listening port does not automatically mean there is a useful application behind it.

You need to determine:

```text
Is it HTTP?
Is it an API?
Is it a management interface?
Is authentication required?
What application is running?
```

---

## 13. Test Common HTTP Ports

If you discover HTTP-like services:

```bash
curl -I http://127.0.0.1:<PORT>
```

Then:

```bash
curl -s http://127.0.0.1:<PORT> | head
```

For HTTPS:

```bash
curl -k -I https://127.0.0.1:<PORT>
```

### ⚠️ IMPORTANT

Do not blindly send destructive requests.

Start with:

```text
HEAD
GET
```

and understand the application before interacting further.

---

## 14. Inspect HTTP Response Headers

### 🔎 ENUMERATE

```bash
curl -I http://127.0.0.1:<PORT>
```

### LOOK FOR

Headers may reveal:

```text
Server
X-Powered-By
Location
WWW-Authenticate
Content-Type
```

These can provide clues about:

* Web server
* Framework
* Authentication mechanism
* Redirects
* Application type

Treat version information as a clue, not proof of a vulnerability.

---

## 15. Identify Services by Port

Common ports can provide quick clues.

| Port | Common Service           |
| ---: | ------------------------ |
|   22 | SSH                      |
|   23 | Telnet                   |
|   25 | SMTP                     |
|   53 | DNS                      |
|   80 | HTTP                     |
|  111 | RPC                      |
|  139 | SMB/NetBIOS              |
|  443 | HTTPS                    |
|  445 | SMB                      |
| 3306 | MySQL                    |
| 5432 | PostgreSQL               |
| 6379 | Redis                    |
| 8080 | Common HTTP alternative  |
| 8443 | Common HTTPS alternative |
| 9000 | Application-specific     |

### ⚠️ IMPORTANT

Ports are **not proof of service identity**.

For example:

```text
8080 = probably HTTP
```

is only a clue.

Verify what is actually listening.

---

## 16. Inspect Localhost-Only Services

Search specifically for loopback listeners:

```bash id="m2w3qz"
ss -ltnp | grep '127.0.0.1'
```

IPv6 loopback:

```bash id="s5j7kw"
ss -ltnp | grep '\[::1\]'
```

### WHY

These services may not appear in an external scan.

But because you already have local access, you can communicate with them.

### High-value pattern

```text
localhost service
       ↓
accessible from your shell
       ↓
management/API interface
       ↓
weak or missing authentication
       ↓
privileged backend
```

This requires validation before being treated as a privilege-escalation path.

---

## 17. Search for All Localhost Listeners

A practical command:

```bash id="6km4xa"
ss -lntup | grep -E '127\.0\.0\.1|\[::1\]'
```

### LOOK FOR

Examples:

```text
127.0.0.1:3306
127.0.0.1:6379
127.0.0.1:8080
127.0.0.1:9000
```

Investigate unfamiliar services first.

---

## 18. Inspect Unix Domain Sockets

Not all local services use TCP.

### 🔎 ENUMERATE

```bash
ss -lx
```

For more information:

```bash
ss -lxnp
```

### WHY

Unix sockets provide local inter-process communication.

You may encounter:

```text
/run/docker.sock
/run/systemd/private
/run/... 
```

### ⚠️ IMPORTANT

A Unix socket can be extremely sensitive depending on:

* Ownership
* Group
* Permissions
* Service behind it

For example, access to a privileged management socket may have significant consequences.

Always inspect permissions before assuming impact.

---

## 19. Investigate Unix Socket Permissions

Suppose you find:

```text
/run/example.sock
```

Run:

```bash id="gq5w6q"
ls -la /run/example.sock
```

Then:

```bash id="9r9t9s"
stat /run/example.sock
```

### ASK

```text
Who owns it?
What group owns it?
Can my user access it?
What service uses it?
What operations does the socket expose?
```

---

## 20. Inspect Active Connections

Listening services are only part of the picture.

### 🔎 ENUMERATE

```bash
ss -tunap
```

### WHY

This shows active network connections.

You may discover:

```text
database connections
internal APIs
management systems
remote services
unexpected outbound connections
```

### LOOK FOR

Especially:

```text
127.0.0.1
private IP addresses
unusual ports
connections owned by root
```

---

## 21. Show Established Connections

```bash id="a1k4qz"
ss -tnp state established
```

### WHY

This focuses on currently established TCP connections.

Example:

```text
10.10.10.25:49152
10.10.10.50:5432
```

This suggests the host is communicating with a PostgreSQL service at:

```text
10.10.10.50:5432
```

That may reveal application architecture.

---

## 22. Inspect UDP Connections

```bash id="v9z1d5"
ss -unp
```

### WHY

UDP services can be overlooked when only TCP is examined.

Look for:

* DNS
* SNMP
* DHCP
* Custom UDP services

---

## 23. Identify DNS Configuration

### 🔎 ENUMERATE

```bash id="i1x8y5"
cat /etc/resolv.conf
```

### WHY

This tells you which DNS resolvers the system uses.

Example:

```text
nameserver 10.10.10.1
```

This can reveal internal network architecture.

---

## 24. Check the Hosts File

### 🔎 ENUMERATE

```bash id="f5f9b8"
cat /etc/hosts
```

### WHY

Static hostname mappings can reveal:

```text
internal servers
development systems
application names
management hosts
```

Example:

```text
10.10.10.20   db.internal
10.10.10.30   admin.internal
```

These are useful architectural clues.

---

## 25. Identify Network Neighbors

### 🔎 ENUMERATE

```bash id="0h1u0d"
ip neigh
```

### WHY

The ARP/neighbor table may show nearby systems that the host has recently communicated with.

Example:

```text
10.10.10.1 dev eth0 lladdr aa:bb:cc:dd:ee:ff REACHABLE
10.10.10.50 dev eth0 lladdr 11:22:33:44:55:66 STALE
```

### 🧠 INTERPRET

This does **not** mean those hosts are necessarily vulnerable.

It simply provides network context.

---

## 26. Check for VPN or Tunnel Interfaces

### 🔎 ENUMERATE

```bash
ip -br link
```

Look for:

```text
tun0
tap0
wg0
```

You can inspect them with:

```bash
ip addr show tun0
```

or:

```bash
ip addr show wg0
```

### WHY

Tunnel interfaces can provide access to networks that are not reachable through the primary interface.

---

## 27. Identify Container Networking

Check interfaces:

```bash id="a5h0b4"
ip -br addr
```

Look for:

```text
docker0
br-xxxx
vethxxxx
```

Then inspect routes:

```bash id="4ax4jc"
ip route
```

### WHY

Container networking can reveal:

```text
container subnets
bridges
internal services
host/container relationships
```

This connects to the later:

```text
10-SPECIAL-ENVIRONMENTS
```

section.

---

## 28. Check Docker's Unix Socket

If Docker is installed, check:

```bash id="q1v0km"
ls -la /var/run/docker.sock 2>/dev/null
```

Also:

```bash
stat /var/run/docker.sock 2>/dev/null
```

### WHY

Access to Docker's control socket can have major security implications because it may provide control over containers and, depending on configuration, the host.

### 🧠 IMPORTANT

Do not simply conclude:

```text
docker.sock exists = root
```

Check:

```text
Who owns it?
What group controls it?
Is the current user a member?
Can the socket actually be used?
```

Detailed container-related privilege escalation is handled later.

---

## 29. Identify Network Services Owned by Root

### 🔎 ENUMERATE

```bash id="z4xq9j"
ss -lntup
```

Look at the process information.

For an interesting PID:

```bash id="5q0o5c"
ps -p <PID> -o pid,ppid,user,group,args
```

### High-value pattern

```text
root process
    ↓
network listener
    ↓
local or internal access
    ↓
application functionality
    ↓
potential privilege boundary
```

Again, this is a **candidate path**, not automatically an exploit.

---

## 30. Map Port → Process → User

This is one of the most useful habits in network enumeration.

Suppose:

```text
127.0.0.1:8080
```

Start:

```bash id="l5bqkj"
ss -lntp | grep ':8080'
```

Find the PID.

Then:

```bash id="b4e6xm"
ps -p <PID> -o pid,ppid,user,group,args
```

Then:

```bash id="7p1w4j"
readlink -f /proc/<PID>/exe
```

Then:

```bash id="h9g1tq"
readlink -f /proc/<PID>/cwd
```

Then:

```bash id="c5w7yb"
tr '\0' '\n' < /proc/<PID>/environ
```

You now have:

```text
PORT
 ↓
PROCESS
 ↓
USER
 ↓
EXECUTABLE
 ↓
WORKING DIRECTORY
 ↓
ENVIRONMENT
```

This connects network enumeration with process/service enumeration.

---

## 31. Investigate a Local Web Application

Suppose you find:

```text
127.0.0.1:8080
```

Start safely:

```bash id="i5p8w1"
curl -I http://127.0.0.1:8080
```

Then:

```bash id="z8w0o7"
curl -s http://127.0.0.1:8080/ | head -n 30
```

Look for:

```text
login pages
admin interfaces
API endpoints
application names
framework indicators
error messages
documentation
```

### NEXT QUESTION

If the service appears to be an application:

```text
Who owns the application?
What privileges does it have?
What files does it access?
Does it expose administrative functionality?
Does it trust local users?
```

---

## 32. Check Common Local API Paths

If an HTTP service is identified, inspect only safe, read-oriented paths first.

Examples:

```bash id="yp4v4x"
curl -s http://127.0.0.1:<PORT>/
curl -s http://127.0.0.1:<PORT>/robots.txt
curl -s http://127.0.0.1:<PORT>/api/
```

### ⚠️ IMPORTANT

Do not assume these endpoints exist.

The purpose is to identify the application's structure without immediately performing state-changing actions.

---

## 33. Identify Authentication Requirements

If a service responds with:

```text
401 Unauthorized
```

inspect:

```bash id="6x4czi"
curl -I http://127.0.0.1:<PORT>
```

Look for:

```text
WWW-Authenticate
```

If it returns:

```text
403 Forbidden
```

the service exists but access is restricted.

### 🧠 INTERPRET

Do not treat:

```text
401
403
```

as failures.

They are information about the service's access-control model.

---

## 34. Investigate Service-to-Service Trust

Network enumeration can reveal architecture such as:

```text
Web application
      ↓
Database
      ↓
Internal API
```

For example:

```text
127.0.0.1:8080
        ↓
10.10.10.50:5432
```

This suggests:

```text
Application → PostgreSQL
```

### NEXT STEP

Search the application configuration for connection details.

For example:

```bash id="q9f0v7"
grep -RniE 'host|hostname|database|db_|postgres|mysql|redis' /opt /var/www 2>/dev/null
```

Use this carefully on authorized systems because recursive searches can generate significant output.

---

## 35. Identify Services Bound to All Interfaces

### 🔎 ENUMERATE

```bash id="h6a2qs"
ss -lntp | grep '0.0.0.0'
```

IPv6:

```bash id="u2c7mv"
ss -lntp | grep '\[::\]'
```

### WHY

These services may be reachable through multiple interfaces.

You should understand:

```text
Which interface?
Which service?
Which user?
Which authentication?
Which application?
```

---

## 36. Look for Management Services

High-interest examples can include:

```text
SSH
database administration interfaces
monitoring systems
application dashboards
container APIs
custom management services
```

But remember:

> **Interesting does not mean vulnerable.**

The correct workflow is:

```text
DISCOVER
   ↓
IDENTIFY
   ↓
UNDERSTAND
   ↓
CHECK ACCESS
   ↓
CHECK PRIVILEGE
   ↓
VALIDATE
```

---

## 37. Check SSH Configuration

SSH itself is covered more deeply in credentials and other sections, but network enumeration should identify it.

### 🔎 ENUMERATE

```bash id="0xkqfr"
ss -lntp | grep ':22'
```

Then, if permitted:

```bash id="h5z2q7"
sshd -T 2>/dev/null | head -n 50
```

### WHY

This can confirm that SSH is running and provide configuration clues.

Do not change SSH configuration during enumeration.

---

## 38. Check for NFS-Related Services

Look for RPC:

```bash id="y8n2r7"
ss -lntup | grep -E ':111|:2049'
```

If available:

```bash id="1o0tqz"
rpcinfo -p 2>/dev/null
```

### WHY

NFS and RPC can reveal mounted or exported filesystems.

Detailed NFS analysis belongs in:

```text
10-SPECIAL-ENVIRONMENTS/nfs.md
```

---

## 39. Check SMB-Related Services

Look for:

```bash id="8j5m0e"
ss -lntup | grep -E ':139|:445'
```

### WHY

This identifies potential SMB exposure.

Detailed SMB analysis is outside the main Linux privilege-escalation workflow unless it directly contributes to the current escalation path.

---

## 40. Check Database Listeners

Common examples:

```bash id="7d8y3r"
ss -lntup | grep -E ':3306|:5432|:6379|:27017'
```

Possible services:

```text
3306  MySQL/MariaDB
5432  PostgreSQL
6379  Redis
27017 MongoDB
```

### NEXT STEP

Do not immediately attempt authentication attacks.

First determine:

```text
Is it local?
Who owns the service?
Does the application use it?
Are credentials stored locally?
Is authentication required?
```

---

## 41. Look for Unexpected High Ports

Do not restrict enumeration to well-known ports.

### 🔎 ENUMERATE

```bash id="c0x7py"
ss -lntup
```

Look for:

```text
8000
8080
8081
8443
9000
9090
10000
random high ports
```

### WHY

Custom applications often use high-numbered ports.

A service on:

```text
127.0.0.1:49152
```

may be more interesting than a standard SSH listener.

---

## 42. Network Enumeration Decision Tree

```text id="w4m2jg"
Start
  │
  ▼
Identify interfaces
  │
  ▼
Identify routes
  │
  ▼
Identify listening services
  │
  ├── No listeners of interest
  │       ↓
  │    Continue other enumeration
  │
  └── Interesting listener
          ↓
      Identify bind address
          │
          ├── 127.0.0.1 / ::1
          │       ↓
          │   Test locally
          │
          └── External interface
                  ↓
              Identify service
                  ↓
              Identify process
                  ↓
              Identify user
                  ↓
              Inspect configuration
                  ↓
              Check authentication
                  ↓
              Check trust relationship
                  ↓
              Can current user influence it?
                  │
             ┌────┴────┐
             ▼         ▼
            NO        YES
             │         │
             ▼         ▼
          Move on   Validate
                       │
                       ▼
               Privilege boundary?
                    │       │
                   NO      YES
                    │       │
                    ▼       ▼
                 Move on  Document
                              ↓
                           Exploit
                              ↓
                           Verify
```

---

## 43. High-Value Network Patterns

### Pattern 1 — Localhost Management Interface

```text id="g7gq3u"
127.0.0.1:PORT
      ↓
management interface
      ↓
accessible from compromised shell
      ↓
weak access control
      ↓
privileged application
```

Investigate carefully.

---

### Pattern 2 — Root-Owned Network Service

```text id="4km1fy"
root process
      ↓
network listener
      ↓
application functionality
      ↓
user-controlled input
```

Determine whether that input can influence privileged behavior.

---

### Pattern 3 — Local Service + Configuration Secret

```text id="0glkpe"
localhost service
      ↓
application config
      ↓
credentials/API key
      ↓
trusted backend
```

This can connect network enumeration with:

```text
08-CREDENTIALS-AND-SECRETS
```

---

### Pattern 4 — Docker Management Socket

```text id="r3n1kp"
/var/run/docker.sock
      ↓
user can access socket
      ↓
Docker management interface
      ↓
potential host-level impact
```

Validate permissions and actual access before treating this as an escalation path.

---

### Pattern 5 — Internal Service Not Externally Reachable

```text id="b7n0oa"
External scan
      X
      │
      ▼
127.0.0.1:9000
      ▲
      │
Current shell
```

This is one of the biggest reasons local network enumeration matters after obtaining a shell.

---

## 44. Finding vs Vulnerability

Keep these separate.

### Finding

```text id="zv7h6u"
127.0.0.1:8080 is listening.
```

That is an observation.

### Interesting Finding

```text id="3a1t8b"
127.0.0.1:8080 is a management interface
running as root.
```

This deserves investigation.

### Potential Vulnerability

```text id="p7n8js"
The management interface is accessible
without appropriate authorization and exposes
privileged functionality.
```

Now there is a potential security issue.

### Exploitable Privilege-Escalation Path

```text id="x0z5kq"
Current user
    ↓
can interact with local service
    ↓
service performs privileged action
    ↓
user-controlled input reaches that action
    ↓
privilege boundary is crossed
```

Only after validation should it become an exploitation candidate.

---

## 45. Common Mistakes

### ❌ Mistake 1 — Only checking `localhost`

Do not inspect only:

```bash
ss -ltnp | grep 127.0.0.1
```

Start with:

```bash
ss -tulpn
```

Then investigate all relevant listeners.

---

### ❌ Mistake 2 — Only checking common ports

Do not assume:

```text
22
80
443
```

are the only important ports.

Custom applications frequently use high ports.

---

### ❌ Mistake 3 — Assuming a listening port is vulnerable

A listening port only proves:

```text
something is listening
```

It does not prove:

```text
vulnerable
```

---

### ❌ Mistake 4 — Ignoring bind addresses

These are different:

```text
127.0.0.1:8080
0.0.0.0:8080
10.10.10.25:8080
```

Always record the bind address.

---

### ❌ Mistake 5 — Ignoring Unix sockets

Some of the most important local interfaces are not TCP ports.

Always consider:

```bash
ss -lxnp
```

---

### ❌ Mistake 6 — Launching aggressive scans immediately

You already have a shell.

Start with local information:

```text
ip
ss
ps
/proc
systemctl
curl
```

before performing unnecessary network scanning.

---

## 46. Practical 5-Minute Network Workflow

If you have limited time, run:

### Step 1 — Interfaces

```bash id="w5f8f8"
ip -br addr
```

### Step 2 — Routes

```bash id="b6x1ny"
ip route
```

### Step 3 — TCP/UDP listeners

```bash id="6f9l3v"
ss -lntup
```

### Step 4 — Localhost listeners

```bash id="j6v7gd"
ss -lntup | grep -E '127\.0\.0\.1|\[::1\]'
```

### Step 5 — Active connections

```bash id="3a6t1v"
ss -tnp state established
```

### Step 6 — DNS

```bash id="9c4r7y"
cat /etc/resolv.conf
```

### Step 7 — Hosts

```bash id="c4v5mx"
cat /etc/hosts
```

### Step 8 — Neighbors

```bash id="k3z2gq"
ip neigh
```

### Step 9 — Unix sockets

```bash id="4f1yq8"
ss -lxnp
```

### Step 10 — Investigate

For every interesting service:

```text id="e1v8px"
PORT
 ↓
BIND ADDRESS
 ↓
PID
 ↓
PROCESS
 ↓
USER
 ↓
EXECUTABLE
 ↓
CONFIGURATION
 ↓
ACCESS CONTROL
 ↓
PRIVILEGE IMPACT
```

---

## 47. Network Enumeration Checklist

```text id="7j3h6v"
[ ] Identify network interfaces
[ ] Identify IPv4 addresses
[ ] Identify IPv6 addresses
[ ] Identify loopback
[ ] Identify default route
[ ] Identify additional routes
[ ] Check VPN/tunnel interfaces
[ ] Check container interfaces
[ ] Enumerate TCP listeners
[ ] Enumerate UDP listeners
[ ] Identify listening processes
[ ] Identify service owners
[ ] Identify localhost-only services
[ ] Identify services bound to external interfaces
[ ] Check active connections
[ ] Check Unix domain sockets
[ ] Inspect socket permissions
[ ] Inspect /etc/resolv.conf
[ ] Inspect /etc/hosts
[ ] Inspect network neighbors
[ ] Identify unusual ports
[ ] Identify management interfaces
[ ] Identify database listeners
[ ] Check Docker socket if relevant
[ ] Check NFS/RPC if relevant
[ ] Check SMB if relevant
[ ] Validate interesting services
[ ] Determine privilege impact
```

---

## 48. Quick Command Reference

### Interfaces

```bash
ip addr
ip -br addr
ip link
```

### Routes

```bash
ip route
ip -6 route
```

### Listening services

```bash
ss -tulpn
ss -ltnp
ss -lunp
```

### Localhost services

```bash
ss -lntp | grep '127.0.0.1'
ss -lntp | grep '\[::1\]'
```

### Active connections

```bash
ss -tunap
ss -tnp state established
```

### Unix sockets

```bash
ss -lx
ss -lxnp
```

### DNS

```bash
cat /etc/resolv.conf
```

### Hosts

```bash
cat /etc/hosts
```

### Neighbors

```bash
ip neigh
```

### Process behind service

```bash
ps -p <PID> -o pid,ppid,user,group,args
readlink -f /proc/<PID>/exe
readlink -f /proc/<PID>/cwd
```

### Local HTTP service

```bash
curl -I http://127.0.0.1:<PORT>
curl -s http://127.0.0.1:<PORT> | head
```

---

## 49. The Network Enumeration Mental Model

Remember this chain:

```text id="j3n4kp"
INTERFACE
    ↓
ADDRESS
    ↓
ROUTE
    ↓
LISTENER
    ↓
BIND ADDRESS
    ↓
PROCESS
    ↓
USER
    ↓
APPLICATION
    ↓
ACCESS CONTROL
    ↓
TRUST
    ↓
PRIVILEGE IMPACT
```

If you cannot explain the entire chain, you probably have not finished investigating the finding.

---

## 💡 Remember

> **A network port is only the beginning of the investigation.**

The real questions are:

```text
WHAT is listening?
WHO owns it?
WHERE is it bound?
WHO can reach it?
WHAT does it trust?
WHAT can it do?
CAN I influence it?
DOES THAT INFLUENCE CROSS A PRIVILEGE BOUNDARY?
```

The most important habit is:

> **Always connect the port back to the process, the process back to the user, and the user back to the privilege boundary.**

That turns network enumeration from a port-number checklist into an actual privilege-escalation workflow.

---

## 🔗 Where Network Findings Lead

```text id="9h5f2m"
Network finding
      │
      ├── Privileged service
      │       ↓
      │   04 Process & Service Enumeration
      │
      ├── Local management interface
      │       ↓
      │   13 Finding Validation
      │
      ├── Credentials discovered
      │       ↓
      │   08 Credentials & Secrets
      │
      ├── Docker/container exposure
      │       ↓
      │   10 Special Environments
      │
      ├── Vulnerable service
      │       ↓
      │   11 Vulnerable Software
      │
      └── Execution/trust weakness
              ↓
          09 Execution Abuse
```

**The goal is not to enumerate every packet, port, or interface.**

The goal is to identify **network relationships that can be converted into a validated privilege-escalation path**.
