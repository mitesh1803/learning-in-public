![Progress](https://img.shields.io/badge/Progress-28%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 28 — Linux Zero to Hero: Full Foundations Sprint

## 📝 Topic: OS Fundamentals, Architecture, Directories, Users, Permissions, Processes, Monitoring, Networking & Disk Management
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 04, 2026

---

## 🎯 Learning Objectives
* Understand why an operating system exists and how Linux fits into its history.
* Understand Linux's layered architecture and what a "distribution" actually is.
* Set up a working Linux environment via WSL or Docker.
* Understand package managers and use APT for software installation.
* Read a terminal prompt and navigate root (`/`) vs. home (`~`).
* Know the purpose of every major top-level Linux directory.
* Understand how `$PATH` resolves commands.
* Create and manage users and groups, and connect remotely via SSH.
* Perform core file management operations and edit files with Vi.
* Read and modify file permissions using `chmod` and `chown`.
* Manage processes — view, kill, prioritize, and control as services.
* Monitor system health with `top`, `htop`, `vmstat`, `free`, `df`, `du`.
* Understand networking fundamentals at a high level (with a pointer to a deeper dedicated resource).
* Perform basic disk management: partition, format, and mount a volume.

> This was a single intensive sprint through the entire **Linux Zero to Hero** series — 10 sections across the course's episodes, condensed into one day of focused notes.

---

## 🖥️ Part 1 — Why an Operating System Exists

```
An OS is the intermediate bridge between:
  Hardware (CPU, memory, disk, devices)
        ↕
  Software / Users (applications, commands, scripts)

Without it, every application would need to manage hardware
resources directly — which doesn't scale or stay secure.
```

### A Brief History

```
1960s → Unix        (the pioneering OS, foundation for what followed)
1980s → Windows      (popularized the GUI for mainstream users)
1990s → Linux        (Linus Torvalds — open-source, highly secure)
```

> **Why Linux dominates production today:** ~90% of production workloads run on Linux — being free, open-source, and backed by a massive global community makes it the default choice for servers, cloud infrastructure, and embedded systems alike.

---

## 🏗️ Part 2 — Linux Architecture & Distributions

```
Layered structure, bottom to top:

  1. Hardware              → the physical machine
  2. Linux Kernel           → the engine: manages processes,
                              memory, and networking directly
  3. System Utilities &     → tools and libraries built on
     Libraries                top of the kernel
  4. Shell / Binaries       → the CLI/GUI layer users
                              actually interact with
```

### What a "Distribution" Actually Is

```
A distribution = the same open-source Linux kernel,
wrapped with a different set of tools, package managers,
and default configurations.

Examples: Ubuntu, Red Hat (RHEL), Debian, Alpine
```

> **Key insight:** the kernel underneath Ubuntu and RHEL is fundamentally the same lineage — the *distribution* is really a curated bundle of decisions layered on top of it (package manager choice, default utilities, release philosophy).

---

## 🐧 Part 3 — Getting Linux Running Locally

### Option 1: WSL (Windows Subsystem for Linux)

```powershell
wsl --install
```

```
Easiest path to a real Linux environment directly on Windows —
no separate VM or dual-boot required.
```

### Option 2: Docker

```bash
docker run -it ubuntu bash
```

```
Why Docker is recommended here:
  → Fully portable — same environment on any machine with Docker
  → Disposable — spin up/tear down instantly, no persistent install
  → Consistent — matches what's actually used in real DevOps workflows
```

---

## 📦 Part 4 — Package Managers

```
Purpose: install, upgrade, and remove software dependencies
(Python, Java, Git, etc.) from centralized, trusted repositories
instead of manually downloading and compiling everything.
```

### APT (Ubuntu/Debian)

```bash
apt update
# Refreshes the local package index from remote repositories
# (does NOT install/upgrade anything by itself)

apt install <package_name>
# Downloads and installs the specified package and its dependencies
```

> **Common beginner mistake to avoid:** running `apt install` without first running `apt update` on a fresh system — the local package index may be stale or empty, causing "package not found" errors even for valid package names.

---

## 💻 Part 5 — Understanding the Terminal Prompt

```
Typical prompt structure:
  user@hostname:path$

Example:
  ubuntu@devbox:~$
    ↑        ↑    ↑
   user   hostname current path
```

| Symbol | Meaning |
|---|---|
| **`/`** | Root directory — the parent of every file and folder on the system |
| **`~`** | Shorthand for the current user's home directory (e.g. `/home/ubuntu`) |

---

## 📂 Part 6 — Key Linux Directories

| Directory | Purpose |
|---|---|
| **`/bin`** | Executable binaries/commands available to all users |
| **`/sbin`** | System administrative binaries — typically root-only tools |
| **`/usr`** | Parent directory for shared user binaries, libraries, and docs; often the actual source behind `/bin` and `/sbin` |
| **`/lib`** | Shared libraries required by the kernel and system utilities |
| **`/etc`** | Configuration files — the most critical directory for system config (password files, network settings, etc.) |
| **`/home`** | Individual user data, one subdirectory per user |
| **`/root`** | The root user's home directory — the one exception that lives outside `/home` |
| **`/opt`** | Third-party applications and optional add-on software |
| **`/var`** | Variable data — log files, caches, anything that changes frequently |
| **`/tmp`** | Temporary files, periodically cleared by the system |
| **`/mnt`** | Temporary mount point for external storage/disks |

> **`useradd` vs. `adduser` note (relevant to `/home`):** `adduser` automatically creates a home directory under `/home` and walks through setup interactively; `useradd` is the lower-level command and does **not** create a home directory by default unless explicitly told to.

---

## 🧭 Part 7 — The `$PATH` Variable

```
When a command is typed (e.g. `ls`):
  1. The shell checks each directory listed in $PATH, in order
  2. If a matching executable is found → it runs
  3. If not found in ANY listed directory → "command not found"
```

```bash
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

> **Why this matters practically:** installing a tool but getting "command not found" almost always means either the binary isn't in a directory listed in `$PATH`, or the shell session needs to be restarted/re-sourced to pick up an updated `$PATH`.

---

## 👤 Part 8 — User Management

### Why It Matters

```
Shared root access = no accountability, no audit trail,
one mistake can affect everyone. Individual users with
scoped permissions is the security baseline for any
multi-user Linux system.
```

### Creating Users

```bash
useradd devuser
# Basic — does NOT create a home directory by default

adduser devuser
# Interactive — creates home directory + prompts for user details
```

### Where User Data Lives

| File | Purpose |
|---|---|
| **`/etc/passwd`** | Stores user account information (username, UID, home dir, shell, etc.) |
| **`/etc/shadow`** | Stores encrypted password information |

```bash
passwd devuser
# Sets or changes a user's password
```

> **Critical to remember:** passwords cannot be decrypted from `/etc/shadow` — they're hashed, not encrypted reversibly. If lost, the only option is to reset it, not recover it.

### Groups

```bash
usermod -aG developers devuser
# Adds devuser to the "developers" group (without removing existing groups)
```

> Groups exist specifically so permissions can be managed once, for many users at a time, instead of configuring access user-by-user.

### Remote Access via SSH

```
Linux servers run the sshd process/daemon by default,
listening for incoming SSH connections.
```

```bash
ssh devuser@<ip-address>
```

---

## 📁 Part 9 — File Management Basics

| Command | Purpose |
|---|---|
| **`ls`** | List files and directories |
| **`cd`** | Change directory (`cd ..` moves up one level) |
| **`pwd`** | Print current working directory |
| **`mkdir`** | Create a new directory |
| **`touch`** | Create a new empty file |
| **`rmdir`** | Delete an empty directory |
| **`rm -rf`** | Forcefully and recursively delete a directory (and its contents) |
| **`cp`** | Copy files |
| **`mv`** | Move or rename files |

---

## ✍️ Part 10 — Vi Editor Shortcuts

### Modes

```
Normal Mode   → default; used for navigation, not typing
Insert Mode   → press "i" to start typing/editing
Command Mode  → press "Esc" then ":" to run editor commands
```

### Saving / Quitting

```
:wq!    → save changes and quit
:q!     → quit WITHOUT saving changes
```

### Reading/Viewing Files (Complementary Commands)

| Command | Purpose |
|---|---|
| **`less`** | View large files interactively (scrollable, searchable) |
| **`head`** | View the beginning of a file |
| **`tail`** | View the end of a file |
| **`echo`** | Print text; combine with `>>` to append output to a file |

---

## 🔐 Part 11 — File Permissions Management

### Why Permissions Exist

```
Linux is inherently multi-user. Without permission controls:
  → Any user could delete or corrupt critical system files
    (/etc, /sbin)
  → Any user could interfere with another user's files/work

Permissions are the protective layer that COMPLEMENTS
user management — user management establishes WHO you are,
permissions establish WHAT you can do.
```

### Reading a Permission String

```bash
ls -ltr
# -rwxr-xr--  1 ubuntu  developers  4096 Jul 4 10:00 script.sh
```

```
Breakdown of "-rwxr-xr--":

  Position 1:     File type       (- = file, d = directory)
  Positions 2-4:  User (owner)    permissions
  Positions 5-7:  Group           permissions
  Positions 8-10: Others          permissions

  r = read     (view contents)
  w = write    (modify/delete)
  x = execute  (run as a script/program)
```

### Modifying Permissions: `chmod`

**Symbolic mode:**
```bash
chmod u=rwx,g=rw,o= script.sh
# owner: full access | group: read+write | others: nothing
```

**Numeric (octal) mode:**
```
r = 4   w = 2   x = 1   (sum digits to combine permissions)

chmod 777 script.sh   → full access for everyone
chmod 400 script.sh   → read-only, owner only
```

### Changing Ownership: `chown`

```bash
chown newowner:newgroup script.sh
# Changes both the owning user and group of a file
# Typically requires root/sudo privileges
```

### The Bank & Locker Analogy (Directory Priority Rule)

```
Even with full read/write access to a FILE, if the user lacks
EXECUTE (x) permission on the containing DIRECTORY, they cannot
reach that file at all.

Analogy: having a key to your bank locker (the file) is useless
if you're not allowed through the bank's front door (the directory).
The path must be traversable FIRST.
```

> **Practice recommendation from the session:** deliberately experiment with different `chmod`/`chown` combinations on test files to build real muscle memory — permission bugs are common in production and intuition here saves significant debugging time.

---

## ⚙️ Part 12 — Process Management

```
A process = a running instance of a program.
```

### Viewing Processes

```bash
ps aux
# Includes memory and CPU usage per process

ps -ef
# Full-format listing, WITHOUT memory/CPU usage columns
```

### Controlling Processes

| Command | Effect |
|---|---|
| **`kill <PID>`** | Sends a standard termination signal |
| **`kill -9 <PID>`** | Forcefully kills a process when standard termination fails |
| **`kill -3 <PID>`** | Requests a thread dump (commonly used for Java applications) |
| **`kill -STOP <PID>`** | Temporarily pauses (suspends) a process |
| **`kill -CONT <PID>`** | Resumes a previously stopped process |

### Prioritization

```bash
renice -n <value> -p <PID>
# Value range: -20 (highest priority) to 19 (lowest priority)
```

### Services

```
Services = background processes that start automatically on boot
(e.g., web servers, databases, SSH daemon)
```

```bash
systemctl start <service_name>
systemctl stop <service_name>
systemctl status <service_name>
```

---

## 📊 Part 13 — Monitoring

```
Monitoring exists to keep the system healthy —
tracking hardware and resource utilization proactively,
before it becomes an outage.
```

| Command | Purpose |
|---|---|
| **`top`** / **`htop`** | Real-time CPU and memory usage monitoring |
| **`vmstat`** | Reports overall system performance metrics |
| **`free -h`** | Memory utilization in a human-readable format |
| **`nproc`** | Displays the number of available CPUs |
| **`df -h`** | Disk space utilization across file systems |
| **`du -sh <directory>`** | Identifies which specific folders consume the most disk space |

> **Pro-tip from the session:** for enterprise environments, don't rely on manually running these commands — integrate Linux servers with **Prometheus** and **Grafana** for automated, real-time alerting (directly connecting back to the monitoring stack covered a few sessions ago).

---

## 🌐 Part 14 — Networking (High-Level)

```
Networking fundamentals flagged as essential for DevOps/sysadmin work:
  → IP addressing
  → Subnets
  → The OSI model
```

> The instructor pointed to a dedicated **Networking Fundamentals** playlist (a full 3-hour deep dive) rather than covering this in depth here — flagged as a priority follow-up topic rather than something to skim.

---

## 💽 Part 15 — Disk Management

### Commands

| Command | Purpose |
|---|---|
| **`lsblk`** | Lists block devices attached to the instance |
| **`fdisk -l`** | Detailed partition information |
| **`mkfs -t <type> /dev/<device>`** | Formats a volume (e.g., `ext4`, `xfs`) |
| **`mount /dev/<device> /<mount_point>`** | Makes formatted storage accessible to the OS |

### The Full Workflow

```
1. Create a volume in the cloud console (e.g., AWS EBS)
2. Attach the volume to the instance
3. Format the file system → mkfs -t ext4 /dev/xvdf
4. Create a mount directory → mkdir /data
5. Mount the device → mount /dev/xvdf /data
```

> Without formatting, a raw attached volume has no file system the OS can read/write to — `mkfs` is the step that makes it actually usable, not just "attached."

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Kernel** | The core OS component managing processes, memory, and networking directly |
| **Distribution** | The Linux kernel packaged with a specific set of tools and defaults (Ubuntu, RHEL, etc.) |
| **WSL** | Windows Subsystem for Linux — runs a real Linux environment directly on Windows |
| **APT** | Debian/Ubuntu's package manager for installing and managing software |
| **`$PATH`** | Environment variable listing directories the shell searches to resolve commands |
| **`/etc/passwd` / `/etc/shadow`** | Files storing user account info and encrypted password data, respectively |
| **`usermod -aG`** | Adds a user to a group without removing existing group memberships |
| **`chmod`** | Changes file/directory permissions (symbolic or numeric/octal mode) |
| **`chown`** | Changes the owner and/or group of a file |
| **PID** | Process ID — the unique identifier for a running process |
| **`renice`** | Adjusts a running process's scheduling priority |
| **`systemctl`** | Manages system services (start/stop/enable/status) |
| **`df` / `du`** | Disk usage tools — `df` for filesystem-level space, `du` for directory-level space |
| **`mkfs`** | Formats a storage device with a specified filesystem type |

---

## 📂 Summary of Tasks
- ✅ Reviewed: OS fundamentals — why an OS exists and Linux's historical timeline.
- ✅ Understood: Linux's layered architecture (Hardware → Kernel → Utilities → Shell) and what a distribution actually is.
- ✅ Set up: A local Linux environment option via WSL and Docker.
- ✅ Practiced: Package management fundamentals with APT (`update`, `install`).
- ✅ Learned: Terminal prompt structure and root (`/`) vs. home (`~`).
- ✅ Mapped: Every major top-level Linux directory and its purpose.
- ✅ Understood: How `$PATH` resolves (or fails to resolve) commands.
- ✅ Practiced: User creation (`useradd` vs. `adduser`), password management, and group assignment.
- ✅ Connected: Via SSH to a remote Linux server.
- ✅ Practiced: Core file management commands (`ls`, `cd`, `mkdir`, `cp`, `mv`, `rm -rf`, etc.).
- ✅ Practiced: Vi editor modes, saving/quitting, and file viewing commands (`less`, `head`, `tail`).
- ✅ Understood: Permission strings, `chmod` (symbolic + numeric), `chown`, and the directory-execute priority rule.
- ✅ Practiced: Process management — viewing, killing, stopping/resuming, and prioritizing processes.
- ✅ Practiced: System monitoring with `top`, `htop`, `vmstat`, `free -h`, `df -h`, `du -sh`.
- ✅ Noted: Networking fundamentals as a flagged follow-up topic via a dedicated playlist.
- ✅ Practiced: The full disk management workflow — attach, format, mount.

---

## 💡 My Takeaway

Compressing all of this into one sprint made the connective tissue between sections much clearer than it would be spread across separate days. User management and file permissions are really two halves of the same security model — one establishes *who* you are, the other establishes *what you're allowed to do* — and the "bank and locker" analogy for directory execute permissions is one of those explanations that instantly resolves a confusion I didn't even realize I had (having file access doesn't matter if the directory path itself is blocked).

The monitoring section closing the loop back to Prometheus and Grafana from a few days ago was a good reminder that these Linux fundamentals aren't standalone trivia — `top`/`free -h`/`df -h` are exactly the manual, single-machine version of what a full observability stack automates and scales across a fleet of servers. Seeing that connection made the earlier Prometheus/Grafana session land with more weight in hindsight.

Networking being explicitly deferred to a dedicated 3-hour resource rather than rushed through here is also worth respecting — IP addressing, subnets, and the OSI model are foundational enough that skimming them in a few minutes would do more harm than good. That's a clear next priority rather than a gap to just note and move past.

---


## 🔗 Resources
* [Abhishek Veermalla Linux Repo ](https://github.com/iam-veeramalla/ultimate-linux-guide)
---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*