
![Progress](https://img.shields.io/badge/Progress-11%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 11 — Ansible: Configuration Management & Hands-On Implementation

## 📝 Topic: Why Ansible Won, Installation, Ad-Hoc Commands, Playbooks & Roles
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** June 14, 2026

---

## 🎯 Learning Objectives
* Understand the historical problem of managing infrastructure at scale with shell scripts.
* Explain why Ansible became the industry standard over Puppet and Chef.
* Understand Agentless architecture and the Push model — and why they matter.
* Set up passwordless SSH authentication between control node and target servers.
* Run Ansible ad-hoc commands for one-off tasks.
* Use inventory files to group and target servers selectively.
* Write a complete playbook in YAML — including `become: yes` and verbose debugging.
* Understand Ansible Roles and the standard directory structure via `ansible-galaxy role init`.

---

## 🧱 Part 1 — The Problem: Managing Infrastructure at Scale

### 📜 The Old World: Custom Scripts

Before configuration management tools existed, system administrators managed servers with hand-written shell or PowerShell scripts.

```bash
# The old approach — repeated manually on every server
for server in server1 server2 server3 ... server200; do
    ssh $server "sudo apt update && sudo apt install nginx -y"
done
```

**Why this breaks down:**

| Problem | What happens |
|---|---|
| **No idempotency** | Running the script twice might break things — install conflicts, duplicate configs |
| **No state tracking** | No way to know which servers are configured correctly vs drifted |
| **Error handling** | One server fails halfway through → script continues blindly or stops entirely |
| **OS differences** | Ubuntu vs CentOS vs Windows — each needs different commands |
| **No audit trail** | No record of what changed, when, or by whom |

### ☁️ The 10x Problem: Cloud & Microservices

The shift to cloud computing and microservices didn't just add a few more servers — it multiplied server count by **10x or more**.

```
Pre-cloud era:
  1 application → 1-5 physical servers → manageable by hand

Cloud + microservices era:
  1 application → 50+ microservices → each needs 2-10 instances
                → 500+ servers to configure consistently
```

At this scale, manual scripts and SSH loops become **operationally impossible**. Configuration Management tools were built to solve exactly this problem.

---

## 🤖 Part 2 — Why Ansible Stands Out

### 🔌 Agentless Architecture

The single biggest differentiator from Puppet and Chef.

```
Puppet / Chef (Agent-based):
  Control Node
       ↓ (requires agent installed)
  Target Server 1 [Agent running]
  Target Server 2 [Agent running]
  Target Server 3 [Agent running]
  
  → Every server needs software installed and maintained
  → Agents need their own updates, version compatibility checks
  → More attack surface, more moving parts

Ansible (Agentless):
  Control Node
       ↓ (SSH or WinRM — already exists)
  Target Server 1 [no agent — just SSH]
  Target Server 2 [no agent — just SSH]
  Target Server 3 [no agent — just SSH]
  
  → Nothing to install on targets
  → Uses infrastructure that's already there (SSH/WinRM)
```

> **Why this matters:** Onboarding a new server into Ansible management requires zero setup on that server beyond SSH access — which it already has. With Puppet/Chef, every new server needs an agent installed, configured, and registered with the master before it can be managed.

### ⬅️➡️ Push vs Pull Model

```
PULL MODEL (Puppet, Chef default):
  Target servers periodically CHECK IN with a central master
  "Master, do you have new configuration for me?"
  → Configuration applied on the target's schedule (e.g. every 30 min)
  → Changes aren't instant — you wait for the next pull cycle

PUSH MODEL (Ansible):
  Control node CONNECTS OUT to targets and applies configuration immediately
  "I am pushing this configuration to you right now"
  → Changes are instant
  → Engineer controls exactly when changes happen
```

| | Pull (Puppet/Chef) | Push (Ansible) |
|---|---|---|
| **Trigger** | Target initiates on schedule | Control node initiates on demand |
| **Timing** | Eventual (next check-in) | Immediate |
| **Control** | Decentralized | Centralized |
| **Best for** | Continuous drift correction at massive scale | On-demand provisioning, deployments, ad-hoc fixes |

### 📝 Simplicity: YAML

```yaml
# Ansible playbook — readable by anyone, even non-programmers
- name: Install and start Nginx
  hosts: webservers
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
    - name: Start nginx service
      service:
        name: nginx
        state: started
```

Compare to Puppet's domain-specific language or Chef's Ruby-based DSL — YAML is **already known by most engineers**, dramatically lowering the learning curve.

### 🧩 Extensibility: Custom Modules & Ansible Galaxy

```
Ansible Module = a unit of work (install package, start service, manage file)
  → Built-in modules cover most needs (apt, yum, copy, template, service...)
  → Write custom modules in Python for anything not covered
  → Share/reuse modules via Ansible Galaxy (community hub)
```

```bash
# Browse community roles and modules
ansible-galaxy search nginx
ansible-galaxy install geerlingguy.nginx
```

---

## 🎤 Part 3 — Interview Questions Covered

### Q1: Ansible vs Puppet/Chef — What's the difference?

```
Architecture:   Ansible = agentless | Puppet/Chef = agent-based (master-slave)
Language:       Ansible = YAML | Puppet = custom DSL | Chef = Ruby DSL
Model:          Ansible = push | Puppet/Chef = pull (typically)
Setup:          Ansible = SSH only | Puppet/Chef = agent install on every node
Learning curve: Ansible = low (YAML) | Puppet/Chef = higher (DSL/Ruby)
```

### Q2: Push vs Pull mechanism?

> Already covered above — Ansible pushes configuration on-demand from a control node. Puppet/Chef agents pull configuration from a master on a schedule.

### Q3: Does Ansible support Windows?

```
Linux targets:   SSH (default, built-in)
Windows targets: WinRM (Windows Remote Management)
                 → requires WinRM enabled and configured on Windows hosts
                 → Ansible control node is still typically Linux/macOS
```

### Q4: How flexible is Ansible across cloud providers?

```
Ansible has dedicated modules for:
  - AWS  (ec2, s3, iam, etc.)
  - Azure (azure_rm_* modules)
  - GCP   (gcp_* modules)
  - VMware, OpenStack, and more

→ Same playbook structure, different cloud-specific modules
→ Genuinely multi-cloud, unlike some proprietary tools
```

---

## ⚠️ Part 4 — Honest Limitations of Ansible

The instructor flagged two real weaknesses — important for balanced understanding:

| Limitation | Detail |
|---|---|
| **Debugging** | Error messages can be cryptic; tracing failures across complex playbooks/roles takes practice |
| **Performance at very high scale** | SSH-based push to thousands of hosts simultaneously can become slow compared to agent-based pull models that distribute load over time |

> Knowing the trade-offs of your tools — not just the marketing pitch — is what separates junior from senior engineers in interviews.

---

## 🛠️ Part 5 — Installation & Prerequisites

### 📦 Installing Ansible

```bash
# On Ubuntu (control node)
sudo apt update
sudo apt install ansible -y

# Verify installation
ansible --version
```

### 🔑 Passwordless SSH — The Foundation

Ansible's agentless model depends entirely on SSH working without password prompts. Without this, every task would hang waiting for manual password entry.

```bash
# Step 1: Generate an SSH key pair on the control node
ssh-keygen -t rsa -b 4096
# Press Enter through all prompts for default location, no passphrase

# Step 2: Copy the public key to each target server
ssh-copy-id ubuntu@<target_server_ip>

# This appends your public key to:
# ~/.ssh/authorized_keys  on the target server

# Step 3: Verify passwordless SSH works
ssh ubuntu@<target_server_ip>
# Should connect WITHOUT a password prompt
```

```
Control Node (~/.ssh/id_rsa)
        │ public key copied to →
        ▼
Target Server (~/.ssh/authorized_keys)

Now: ssh ubuntu@target  → connects instantly, no password
```

---

## ⚡ Part 6 — Ansible Ad-Hoc Commands

For quick, one-off tasks — no playbook needed.

### 📋 The Inventory File

Before running any command, Ansible needs to know **which servers** to target.

```ini
# /etc/ansible/hosts  (or a custom inventory file)

[webservers]
192.168.1.10
192.168.1.11

[dbservers]
192.168.1.20

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**Grouping lets you target subsets of your infrastructure:**

```bash
# Run against ALL servers
ansible all -m ping -i hosts

# Run against ONLY web servers
ansible webservers -m ping -i hosts

# Run against ONLY db servers
ansible dbservers -m ping -i hosts
```

### 🧪 Common Ad-Hoc Commands

```bash
# Test connectivity to all hosts
ansible all -m ping -i hosts

# Check disk space across all web servers
ansible webservers -m shell -a "df -h" -i hosts

# Install a package (requires sudo)
ansible webservers -m apt -a "name=nginx state=present" --become -i hosts

# Copy a file to all targets
ansible all -m copy -a "src=./app.conf dest=/etc/app.conf" -i hosts

# Restart a service
ansible webservers -m service -a "name=nginx state=restarted" --become -i hosts
```

**Anatomy of an ad-hoc command:**
```
ansible  <target-group>  -m <module>  -a "<arguments>"  --become  -i <inventory>
            ↓                ↓              ↓               ↓          ↓
        which hosts      what to do     module args    use sudo   which inventory file
```

> **When to use ad-hoc vs playbooks:** Ad-hoc commands are for quick checks and emergency fixes — "is this service running on all 50 servers right now?" Playbooks are for repeatable, version-controlled configuration.

---

## 📜 Part 7 — Writing Playbooks

### 📋 Example: Install and Start Nginx

```yaml
# nginx-playbook.yml
---
- name: Install and configure Nginx web server
  hosts: webservers
  become: yes              # run tasks with sudo

  tasks:
    - name: Update apt package cache
      apt:
        update_cache: yes

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Start and enable nginx service
      service:
        name: nginx
        state: started
        enabled: yes        # start on boot too

    - name: Confirm nginx is running
      command: systemctl status nginx
      register: nginx_status

    - name: Print nginx status
      debug:
        msg: "{{ nginx_status.stdout }}"
```

### ▶️ Running the Playbook

```bash
ansible-playbook nginx-playbook.yml -i hosts

# With verbose output for debugging
ansible-playbook nginx-playbook.yml -i hosts -v
ansible-playbook nginx-playbook.yml -i hosts -vvv   # even more detail
```

### 🔑 Understanding `become: yes`

```yaml
become: yes
```

Equivalent to prefixing every task with `sudo`. Without it, tasks that need root privileges (installing packages, managing services, editing system files) will fail with permission errors.

```
become: yes  → "sudo apt install nginx"  (works)
(no become)  → "apt install nginx"        (Permission denied)
```

### 🔍 The `-v` Verbose Flag — Debugging Workflow

```bash
ansible-playbook nginx-playbook.yml -i hosts -v
```

Verbose mode shows:
* Exact module arguments sent to each host
* Raw JSON responses from targets
* Which tasks were skipped and why (`changed: false` vs `changed: true`)

```
TASK [Install nginx] ****************
changed: [192.168.1.10] => {"changed": true, "stdout": "Setting up nginx..."}
ok: [192.168.1.11] => {"changed": false}   ← already installed, no action taken
```

> **`changed: false` is good news** — it means Ansible checked the current state, found it already matches the desired state, and did nothing. This is **idempotency** — the core property that makes Ansible safe to run repeatedly.

---

## 📁 Part 8 — Ansible Roles

### ❓ Why Roles?

A single playbook works fine for small tasks. But for something like configuring a full Kubernetes cluster — dozens of tasks, variables, templates, handlers — one giant YAML file becomes unmanageable.

**Roles split a complex configuration into a standard, reusable structure.**

### 🏗️ Creating a Role

```bash
ansible-galaxy role init my-nginx-role
```

This generates the standard directory structure:

```
my-nginx-role/
├── tasks/
│   └── main.yml       ← the actual steps (like the playbook tasks)
├── handlers/
│   └── main.yml       ← triggered actions (e.g. "restart nginx" on config change)
├── templates/
│   └── nginx.conf.j2  ← Jinja2 templates for dynamic config files
├── files/
│   └── (static files to copy as-is)
├── vars/
│   └── main.yml       ← variables specific to this role
├── defaults/
│   └── main.yml       ← default variable values (lowest priority — easily overridden)
├── meta/
│   └── main.yml       ← role metadata, dependencies on other roles
└── README.md
```

### 📋 What Goes Where

| Directory | Purpose | Example |
|---|---|---|
| **tasks/** | The main list of actions to perform | "Install nginx, copy config, start service" |
| **handlers/** | Actions triggered by `notify` from tasks | "Restart nginx" (only runs if config changed) |
| **templates/** | Jinja2 templates — dynamic config files | `nginx.conf.j2` with `{{ server_name }}` variables |
| **vars/** | Fixed variables for this role | `nginx_port: 80` |
| **defaults/** | Overridable default variables | `nginx_worker_processes: auto` |
| **meta/** | Role dependencies and metadata | "This role requires the `common` role first" |

### 🔗 Using a Role in a Playbook

```yaml
# site.yml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  roles:
    - my-nginx-role
    - common
    - monitoring
```

### 🔔 Handlers Example

```yaml
# tasks/main.yml
- name: Copy nginx config from template
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart nginx    # only triggers handler if file actually changed

# handlers/main.yml
- name: Restart nginx
  service:
    name: nginx
    state: restarted
```

> **The point of handlers:** Restarting nginx is disruptive (drops connections). Only do it when the config actually changed — not on every playbook run. `notify` + handlers gives you this "only if changed" behavior automatically.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Configuration Management** | The practice of automating server setup and ensuring consistent state across infrastructure |
| **Idempotency** | Running the same operation multiple times produces the same result — no duplicate side effects |
| **Agentless** | No software needs to be installed on managed targets — Ansible uses existing SSH/WinRM |
| **Push Model** | The control node initiates configuration changes on demand |
| **Pull Model** | Target servers periodically check in with a master for new configuration |
| **Control Node** | The machine where Ansible is installed and from which commands are run |
| **Target/Managed Node** | A server being configured by Ansible — requires only SSH access |
| **Inventory** | A file listing target servers, optionally grouped (e.g. `[webservers]`, `[dbservers]`) |
| **Ad-Hoc Command** | A single Ansible command for a quick task — no playbook required |
| **Playbook** | A YAML file defining a set of tasks to run on target hosts |
| **Module** | A unit of work Ansible can perform — `apt`, `copy`, `service`, `template`, etc. |
| **`become: yes`** | Run tasks with elevated (sudo) privileges |
| **Task** | A single step in a playbook — calls one module with specific arguments |
| **`-v` / `-vvv`** | Verbose flags — show increasing levels of execution detail for debugging |
| **`changed: false`** | Ansible checked state, found it already correct, took no action — idempotency in action |
| **Role** | A reusable, structured package of tasks, handlers, templates, and variables |
| **`ansible-galaxy role init`** | Generates the standard role directory structure |
| **Handler** | A task that only runs when triggered via `notify` — typically for restarts |
| **Jinja2 Template** | A templating syntax (`.j2` files) for generating dynamic config files with variables |
| **Ansible Galaxy** | The community hub for sharing reusable roles and modules |
| **WinRM** | Windows Remote Management — the protocol Ansible uses for Windows targets |
| **`ssh-copy-id`** | Copies a local public SSH key to a remote server's `authorized_keys` |

---

## 📂 Summary of Tasks
- ✅ Understood: Why manual scripts break down at the scale cloud/microservices created.
- ✅ Understood: Agentless architecture — why no agent installation is a major advantage.
- ✅ Understood: Push vs Pull — Ansible's push model vs Puppet/Chef's pull model.
- ✅ Reviewed: 4 key interview questions on Ansible vs Puppet/Chef, push/pull, OS support, cloud flexibility.
- ✅ Acknowledged: Ansible's real limitations — debugging difficulty and high-scale performance.
- ✅ Installed: Ansible via `apt` and verified with `ansible --version`.
- ✅ Set up: Passwordless SSH using `ssh-keygen` and `ssh-copy-id`.
- ✅ Practiced: Ad-hoc commands with inventory files and host groups.
- ✅ Written: A complete playbook to install and start Nginx, with `become: yes`.
- ✅ Used: `-v` verbose flag to understand Ansible's execution flow and idempotency.
- ✅ Understood: Ansible Roles and the standard directory structure via `ansible-galaxy role init`.
- ✅ Understood: Handlers and `notify` — restart-on-change pattern.

---

## 💡 My Takeaway

The agentless + push model combination is the real reason Ansible became the default. Every other config management tool I've heard of requires some form of "install something on every server first" — which is exactly the chicken-and-egg problem you're trying to solve. Ansible's only requirement — SSH — is something every server already has. That's not a small convenience, it's the entire reason onboarding a new server takes minutes instead of a deployment cycle.

The idempotency concept also reframed how I read playbook output. `changed: false` isn't "nothing happened" in a bad way — it's confirmation that the system is already in the desired state. That's the property that makes it safe to run the same playbook against production every day without fear of duplicating installs or restarting services unnecessarily.

On Roles: the directory structure (`tasks`, `handlers`, `templates`, `vars`, `defaults`, `meta`) initially looked like over-engineering for a simple nginx install. But the handler pattern — "only restart nginx if the config actually changed" — is the kind of detail that matters enormously at scale. Restarting nginx on 200 servers every time you run a playbook, even when nothing changed, would cause real outages.

---


## 🔗 Resources
* [Abhishek Veermalla Date Playlist](https://youtube.com/playlist?list=PLdpzxOOAlwvIc1TjTwopNSjRJkzES2ZXk&si=kx2Ia6UsQ1Oou6Vg)
* [Ansible Documentation](https://docs.ansible.com/)
* [Ansible Galaxy — Community Roles](https://galaxy.ansible.com/)
* [Ansible Module Index](https://docs.ansible.com/ansible/latest/collections/index_module.html)
* [Jinja2 Template Designer Docs](https://jinja.palletsprojects.com/en/3.1.x/templates/)
* [Ansible Best Practices Guide](https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html)
* [Idempotency Explained — Ansible Docs](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*
