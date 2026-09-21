# Linux Privilege Escalation Workflow

> A practical, decision-driven guide to Linux privilege escalation — from obtaining a low-privileged shell to identifying, validating, and exploiting privilege boundaries in authorized environments.

---

## 📌 About

**Linux Privilege Escalation Workflow** is a practical methodology for learning and performing Linux privilege escalation.

Instead of presenting hundreds of commands as an isolated cheatsheet, this repository follows a structured workflow:

```text
Low-Privileged Shell
        │
        ▼
   Establish Context
        │
        ▼
     Enumerate
        │
        ▼
 Identify Interesting Findings
        │
        ▼
    Investigate
        │
        ▼
     Validate
        │
        ▼
     Exploit
        │
        ▼
   Verify Privileges
        │
        ▼
   Document the Path
```

The goal is simple:

> **Know what to run, know why you are running it, understand what you found, and know what to do next.**

---

## 🎯 Who Is This For?

This repository is designed for:

* Students learning Linux privilege escalation
* Beginner and intermediate penetration testers
* CTF and cybersecurity lab learners
* Security practitioners who want a structured Linux enumeration workflow
* Anyone who wants to understand **why** privilege escalation techniques work rather than simply memorizing commands

---

## 🧠 The Core Mindset

Linux privilege escalation is not about blindly running commands until something works.

The central question is:

> **Can I influence something that a more privileged process, user, or service trusts or executes?**

Throughout the workflow, repeatedly ask:

```text
WHO am I?
WHAT system am I on?
WHAT privileges do I have?
WHAT is running?
WHAT runs with more privilege?
WHO owns it?
WHO can modify it?
WHAT does it trust?
CAN I influence it?
DOES that influence cross the privilege boundary?
```

These questions are more important than memorizing individual commands.

---

## 🔄 The Workflow

The repository follows this progression:

```text
00  Foundations
 ↓
01  First 5 Minutes
 ↓
02  System Enumeration
 ↓
03  Filesystem Enumeration
 ↓
04  Process & Service Enumeration
 ↓
05  Network Enumeration
 ↓
06  Privilege Mechanisms
 ↓
07  Scheduled Execution
 ↓
08  Credentials & Secrets
 ↓
09  Execution Abuse
 ↓
10  Special Environments
 ↓
11  Vulnerable Software
 ↓
12  Automated Enumeration
 ↓
13  Finding Validation
 ↓
14  Exploitation
 ↓
15  Root Verification
 ↓
16  Methodology
 ↓
17  Quick Reference
 ↓
18  Labs
 ↓
19  Case Studies
```

---

## 🧭 How to Use This Repository

There are two ways to use this project.

### 🟢 Guided Mode

If you are learning or working through a machine from scratch:

```text
Start at 00-FOUNDATIONS
        ↓
Follow the workflow sequentially
        ↓
Run each command
        ↓
Interpret the result
        ↓
Follow the appropriate branch
        ↓
Validate findings
        ↓
Escalate
        ↓
Verify
```

This mode teaches the **methodology**.

### ⚡ Quick Reference Mode

If you already understand the concepts and need a fast reference:

```text
17-QUICK-REFERENCE
```

This section compresses the workflow into practical command references, checklists, and decision trees.

---

## 🧩 The Command-Chain Method

Every important practical workflow follows the same pattern:

```text
┌─────────────┐
│    RUN      │
│  Command    │
└──────┬──────┘
       ↓
┌─────────────┐
│    WHY      │
│ Short reason│
└──────┬──────┘
       ↓
┌─────────────┐
│  LOOK FOR   │
│ Interesting │
│   output    │
└──────┬──────┘
       ↓
┌─────────────┐
│  INTERPRET  │
│ What does   │
│ it mean?    │
└──────┬──────┘
       ↓
   ┌───┴───┐
   ▼       ▼
   X       Y
   │       │
   ▼       ▼
NEXT X   NEXT Y
   │       │
   └───┬───┘
       ▼
┌─────────────┐
│  VALIDATE   │
└──────┬──────┘
       ↓
┌─────────────┐
│  EXPLOIT    │
└──────┬──────┘
       ↓
┌─────────────┐
│   VERIFY    │
└─────────────┘
```

For example:

```text
Run:
sudo -l

        ↓

Why:
Determine whether the current user can execute
commands through sudo.

        ↓

Look for:
Commands executable with elevated privileges.

        ↓

Interpret:
Is the permission actually useful?
What binary and arguments are allowed?

        ↓

Next:
Investigate the permitted command.

        ↓

Validate:
Confirm that the permission crosses the
intended privilege boundary.

        ↓

Exploit:
Use the applicable technique in an
authorized environment.

        ↓

Verify:
Confirm the resulting privileges.
```

This pattern is used throughout the repository.

---

## 📚 Repository Structure

```text
linux-privilege-escalation-workflow/
│
├── 00-FOUNDATIONS/
│   ├── README.md
│   ├── linux-privilege-model.md
│   ├── users-groups-and-ids.md
│   ├── file-permissions.md
│   ├── processes-and-services.md
│   ├── sudo-and-trust.md
│   └── privilege-escalation-mindset.md
│
├── 01-FIRST-5-MINUTES/
│   ├── README.md
│   ├── establish-context.md
│   ├── identity.md
│   ├── host-information.md
│   ├── environment.md
│   └── first-5-minutes-checklist.md
│
├── 02-SYSTEM-ENUMERATION/
│
├── 03-FILESYSTEM-ENUMERATION/
│
├── 04-PROCESS-SERVICE-ENUMERATION/
│
├── 05-NETWORK-ENUMERATION/
│
├── 06-PRIVILEGE-MECHANISMS/
│
├── 07-SCHEDULED-EXECUTION/
│
├── 08-CREDENTIALS-AND-SECRETS/
│
├── 09-EXECUTION-ABUSE/
│
├── 10-SPECIAL-ENVIRONMENTS/
│
├── 11-VULNERABLE-SOFTWARE/
│
├── 12-AUTOMATED-ENUMERATION/
│
├── 13-FINDING-VALIDATION/
│
├── 14-EXPLOITATION/
│
├── 15-ROOT-VERIFICATION/
│
├── 16-METHODOLOGY/
│
├── 17-QUICK-REFERENCE/
│
├── 18-LABS/
│
├── 19-CASE-STUDIES/
│
├── LICENSE
├── CONTRIBUTING.md
└── CHANGELOG.md
```

The directories will be created progressively as the guide is built.

---

## 🧠 The Six Questions

When approaching a Linux host, keep these questions in mind:

```text
1. WHO am I?

2. WHAT am I running?

3. WHO owns it?

4. WHO can modify it?

5. WHAT runs with more privilege?

6. CAN I influence that execution?
```

These questions form the mental model behind the workflow.

---

## 🔎 What Counts as an Interesting Finding?

A finding becomes particularly interesting when it creates a potential path across a pr
