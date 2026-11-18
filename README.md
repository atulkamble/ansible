# Ansible Configuration Management Training Guide

## Purpose and learning outcomes

This pointwise guide introduces configuration management, configuration as code, Ansible, ad hoc automation, YAML playbooks, reusable roles, variables, and cloud provisioning. The labs use an Ubuntu control node and Linux managed hosts. By the end, learners should be able to install Ansible, connect to hosts, run safe ad hoc tasks, write and execute playbooks, organize automation into roles, and understand the pattern for provisioning cloud resources.

## 1. Introduction to configuration management

1. **Configuration management** is the practice of defining, applying, and maintaining a system's desired state.
2. It covers operating-system settings, packages, users, services, files, application configuration, and deployment steps.
3. Without automation, engineers repeat manual steps on each server. This can cause configuration drift, inconsistent environments, and difficult recovery.
4. A configuration management tool applies repeatable instructions across one or many systems and reports whether changes were made.
5. **Idempotence** means that running the same automation repeatedly converges on the same desired state. A well-written task does not reinstall or rewrite something unnecessarily.
6. Configuration management complements image building and infrastructure provisioning. It usually configures hosts after they exist, though Ansible can also call cloud APIs to create infrastructure.

### Benefits to remember

- Consistent development, test, and production environments.
- Faster setup and recovery of servers.
- Reviewable change history when automation is stored in version control.
- Less repetitive manual work and fewer configuration differences.
- Easier auditing of the desired configuration.

## 2. Concepts of Configuration as Code

1. **Configuration as Code (CaC)** stores system configuration in readable, version-controlled files.
2. Code can be reviewed, tested, reused, and promoted through environments.
3. Keep secrets out of plain text source files. Use Ansible Vault or an approved secrets manager.
4. Use variables for environment-specific values instead of duplicating whole playbooks.
5. Prefer declarative tasks: describe the desired state, such as “nginx is installed and running,” rather than a long sequence of imperative shell commands.
6. Use source control branches and pull requests to review changes before applying them to important systems.

## 3. Ansible introduction and architecture

1. **Ansible** is an automation engine used for configuration management, application deployment, orchestration, and cloud provisioning.
2. It is agentless for common Linux administration: Ansible connects to managed hosts over SSH and runs modules remotely. Windows hosts commonly use WinRM.
3. The **control node** is where Ansible is installed and commands are run.
4. **Managed nodes** are the hosts Ansible configures. They need network access, a supported Python interpreter for most modules, and an SSH account with appropriate privileges.
5. An **inventory** names hosts and groups them.
6. A **module** performs a unit of work, such as managing a package, file, user, service, or cloud resource.
7. A **task** calls a module with parameters.
8. A **play** maps a set of tasks to selected hosts. A **playbook** contains one or more plays in YAML.
9. A **role** packages related tasks, handlers, templates, files, defaults, and variables for reuse.

### Execution flow

1. Ansible reads configuration and inventory.
2. It selects hosts that match the play or command pattern.
3. It connects to hosts, usually using SSH.
4. It runs modules with the requested parameters.
5. It reports `ok`, `changed`, `failed`, or `unreachable` results.

## 4. Lab setup

### Lab topology

- One Ubuntu 22.04 or 24.04 control node (a laptop or VM).
- One or two Ubuntu or Amazon Linux managed VMs reachable over SSH.
- A private key authorized for the managed VMs.
- Network rules that allow SSH from the control node to the managed nodes.

Use a lab account, not a personal production account. Restrict SSH access to the control node's address. Do not expose private key contents in class notes or repositories.

### Install Ansible on Ubuntu

```bash
sudo apt update
sudo apt install -y ansible-core openssh-client python3
ansible --version
```

On some Ubuntu releases, the `ansible` package provides the full community package, while `ansible-core` provides the engine. For the introductory labs, `ansible-core` is sufficient. Install collections separately when a lab requires them.

### Create a project directory

```bash
mkdir -p ~/ansible-lab/{group_vars,host_vars,playbooks,roles}
cd ~/ansible-lab
```

### Prepare SSH key access

Set the path to the private key file supplied for your lab. The following assumes it is stored locally at `~/.ssh/ansible-lab.pem`:

```bash
chmod 600 ~/.ssh/ansible-lab.pem
ssh -i ~/.ssh/ansible-lab.pem ubuntu@192.0.2.10
```

Replace `192.0.2.10` with a managed host's reachable IP address. For Amazon Linux, the default login user is commonly `ec2-user`; for Ubuntu cloud images, it is commonly `ubuntu`. Confirm the image documentation if connection fails.

## 5. Understanding the configuration file

Ansible loads settings from `ansible.cfg`. A project-local file is convenient for a lab; it applies when commands are run from that project directory.

Create `ansible.cfg`:

```ini
[defaults]
inventory = ./inventory.ini
remote_user = ubuntu
private_key_file = ~/.ssh/ansible-lab.pem
host_key_checking = True
retry_files_enabled = False
interpreter_python = auto_silent

[privilege_escalation]
become = False
```

### Configuration points to remember

- Ansible searches configuration in this order: `ANSIBLE_CONFIG`, current directory `ansible.cfg`, user configuration, then system configuration. The first applicable file wins.
- Check the selected file with `ansible --version`.
- Keep host key checking enabled in real environments. For a disposable lab, resolve host-key changes by reviewing and updating `known_hosts`; do not blindly trust a changed host key.
- `remote_user` is the SSH login account. `become` controls privilege escalation, commonly through `sudo`.
- Avoid setting passwords or secrets directly in `ansible.cfg`.

## 6. Inventory file basics

Create `inventory.ini`:

```ini
[web]
web1 ansible_host=192.0.2.10
web2 ansible_host=192.0.2.11

[web:vars]
ansible_user=ubuntu

[linux:children]
web
```

Inventory facts:

- `web` is a host group; `web1` and `web2` are inventory aliases.
- `ansible_host` is the network address or DNS name.
- `ansible_user`, `ansible_port`, and `ansible_ssh_private_key_file` can be assigned per host or group.
- Groups can contain child groups.
- YAML inventory is also supported, but INI is quick for a small lab.

Inspect and test inventory:

```bash
ansible-inventory --graph
ansible-inventory --list
ansible all -m ping
ansible web -m ping
```

The `ping` module checks Ansible connectivity and Python availability; it is not an ICMP network ping.

### Troubleshooting SSH and inventory

```bash
ansible-inventory --host web1
ssh -i ~/.ssh/ansible-lab.pem ubuntu@192.0.2.10
ansible web1 -m ping -vvv
```

Check that the key path exists, permissions are restricted, the login user matches the VM image, the VM is running, and inbound SSH permits the control node. A path such as `/etc/ansible/home/ec2-user/key.pem` is likely incorrect if the key is actually in the user's home directory; use the real absolute path, for example `/home/ec2-user/ansible/key.pem`.

## 7. Ad hoc tasks and common quick commands

Ad hoc commands run one task without creating a playbook. General form:

```bash
ansible <host-pattern> -m <module> -a '<module arguments>'
```

Examples:

```bash
# Verify Ansible connectivity
ansible all -m ping

# Gather operating system facts
ansible web -m setup -a 'filter=ansible_distribution*'

# Run a read-only command
ansible web -m command -a 'uptime'

# Install nginx on Debian or Ubuntu hosts
ansible web -b -m ansible.builtin.apt -a 'name=nginx state=present update_cache=true'

# Ensure nginx is enabled and running
ansible web -b -m ansible.builtin.service -a 'name=nginx state=started enabled=true'

# Create a directory
ansible web -b -m ansible.builtin.file -a 'path=/opt/ansible-demo state=directory owner=root group=root mode=0755'

# Check disk space (read-only shell command)
ansible all -m ansible.builtin.command -a 'df -h'

# Limit a command to one host
ansible web1 -m ping
```

Quick command reminders:

- `-i inventory.ini` selects an inventory explicitly.
- `-m` selects a module; `-a` passes module arguments.
- `-b` enables privilege escalation (`become`).
- `-K` prompts for the become password when needed.
- `-u` overrides the remote user.
- `--limit web1` narrows a playbook run to a host or group.
- `-C` or `--check` previews many changes without applying them; support depends on each module.
- `-D` or `--diff` shows file changes where supported. Avoid exposing secrets in diff output.
- Use `ansible-doc ansible.builtin.apt` or `ansible-doc -l` to inspect modules.

Use `command` or `shell` only when a purpose-built module does not fit. The `command` module does not interpret pipes, redirects, or shell expansion. `shell` does, but increases quoting and security risks.

## 8. First playbook using YAML

Create `playbooks/hello.yml`:

```yaml
---
- name: Verify managed hosts
  hosts: all
  gather_facts: true
  tasks:
    - name: Show the managed host name
      ansible.builtin.debug:
        msg: "Ansible connected to {{ inventory_hostname }} running {{ ansible_facts['distribution'] }}"
```

Run it:

```bash
ansible-playbook playbooks/hello.yml
ansible-playbook playbooks/hello.yml --syntax-check
ansible-playbook playbooks/hello.yml --list-hosts
ansible-playbook playbooks/hello.yml --list-tasks
```

YAML reminders:

- Indentation defines structure; use spaces, not tabs.
- `---` marks the beginning of a YAML document.
- A playbook is a YAML list of plays. Tasks are a list under `tasks:`.
- Quote strings containing template expressions or special YAML characters when appropriate.
- Ansible expressions use `{{ variable }}`. Conditions use expressions without braces, such as `when: ansible_facts['os_family'] == 'Debian'`.

## 9. Playbook for software installation and configuration

This example installs nginx, deploys a small page from a template, and starts the service. Create `playbooks/web.yml`:

```yaml
---
- name: Configure a basic web server
  hosts: web
  become: true
  gather_facts: true

  vars:
    web_package: nginx
    page_title: Ansible managed web server
    page_message: This page was deployed by an Ansible playbook.

  tasks:
    - name: Install nginx on Debian-family systems
      ansible.builtin.apt:
        name: "{{ web_package }}"
        state: present
        update_cache: true
        cache_valid_time: 3600
      when: ansible_facts['os_family'] == 'Debian'

    - name: Install nginx on RedHat-family systems
      ansible.builtin.dnf:
        name: nginx
        state: present
      when: ansible_facts['os_family'] == 'RedHat'

    - name: Deploy the index page
      ansible.builtin.template:
        src: ../templates/index.html.j2
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: '0644'
      notify: Restart nginx

    - name: Ensure nginx is enabled and running
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

Create the template at `templates/index.html.j2` (first create the directory with `mkdir -p templates`):

```html
<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>{{ page_title }}</title></head>
  <body>
    <h1>{{ page_title }}</h1>
    <p>{{ page_message }}</p>
    <p>Host: {{ inventory_hostname }}</p>
  </body>
</html>
```

Run and verify:

```bash
mkdir -p templates
ansible-playbook playbooks/web.yml --syntax-check
ansible-playbook playbooks/web.yml --check --diff
ansible-playbook playbooks/web.yml
curl -I http://192.0.2.10/
```

Replace the example address with a reachable web host address. Use `--check --diff` first where the tasks support check mode. The playbook can be run repeatedly; after the first successful run, unchanged tasks should report `ok` rather than `changed`.

### Modules and task patterns

- `apt` / `dnf`: manage operating-system packages.
- `service` or `systemd`: manage services.
- `file`: manage directories, permissions, links, and file state.
- `copy`: transfer a static file from the control node.
- `template`: render a Jinja2 template with variables.
- `user`: manage local accounts.
- `debug`: display values during development.
- `command`: run a command without shell parsing.
- `uri`: call HTTP APIs.
- `assert`: stop early if a required condition is not met.
- `notify` and `handlers`: run a handler only when a notifying task reports a change; handlers normally run once at the end of a play.

Prefer fully qualified collection names such as `ansible.builtin.apt` in maintained playbooks.

## 10. Variables for dynamic configuration

Variables let the same automation work across hosts and environments.

### Common variable locations

1. **Role defaults**: `roles/<role>/defaults/main.yml`; lowest precedence and intended for easy overrides.
2. **Inventory group variables**: `group_vars/<group>.yml`.
3. **Inventory host variables**: `host_vars/<host>.yml`.
4. **Play variables**: `vars:` inside a play.
5. **Extra variables**: supplied with `-e` at runtime; high precedence.

Example `group_vars/web.yml`:

```yaml
---
web_package: nginx
web_service: nginx
web_root: /var/www/html
web_listen_port: 80
```

Use a variable in a task:

```yaml
- name: Ensure the web package is installed
  ansible.builtin.package:
    name: "{{ web_package }}"
    state: present
```

Override an ordinary variable for a run:

```bash
ansible-playbook playbooks/web.yml -e 'page_title=Staging web server'
```

Variable points to remember:

- Use meaningful names and keep related values together.
- Quote values that may be interpreted as booleans, numbers, or special YAML values when they should be strings.
- Prefer defaults for reusable role settings and group variables for environment-level configuration.
- Avoid defining the same variable in many places; precedence can be surprising.
- Use Ansible Vault to encrypt secrets, for example `ansible-vault encrypt group_vars/prod/vault.yml`.
- Do not print secret variables with `debug`; mark secret-handling tasks with `no_log: true` where appropriate.

## 11. Ansible roles and reusable automation

Roles provide a standard structure for modular automation. Create a role skeleton:

```bash
ansible-galaxy role init roles/webserver
```

Typical role structure:

```text
roles/webserver/
├── defaults/main.yml
├── files/
├── handlers/main.yml
├── tasks/main.yml
├── templates/
├── vars/main.yml
└── meta/main.yml
```

Role directories:

- `tasks/`: ordered actions the role performs.
- `handlers/`: actions triggered by task notifications, such as restarting a service.
- `templates/`: Jinja2 templates copied after variable rendering.
- `files/`: static files copied without rendering.
- `defaults/`: default variables designed to be overridden.
- `vars/`: role variables with higher precedence; use sparingly for values that should not be casually overridden.
- `meta/`: role metadata and dependencies.

Example `roles/webserver/defaults/main.yml`:

```yaml
---
webserver_package: nginx
webserver_service: nginx
webserver_document_root: /var/www/html
webserver_page_title: Managed by Ansible role
```

Example `roles/webserver/tasks/main.yml`:

```yaml
---
- name: Install the web server package
  ansible.builtin.package:
    name: "{{ webserver_package }}"
    state: present

- name: Deploy the home page
  ansible.builtin.template:
    src: index.html.j2
    dest: "{{ webserver_document_root }}/index.html"
    owner: root
    group: root
    mode: '0644'
  notify: Restart web server

- name: Enable and start the web service
  ansible.builtin.service:
    name: "{{ webserver_service }}"
    state: started
    enabled: true
```

Example `roles/webserver/handlers/main.yml`:

```yaml
---
- name: Restart web server
  ansible.builtin.service:
    name: "{{ webserver_service }}"
    state: restarted
```

Example `roles/webserver/templates/index.html.j2`:

```html
<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>{{ webserver_page_title }}</title></head>
  <body><h1>{{ webserver_page_title }}</h1><p>Configured on {{ inventory_hostname }}</p></body>
</html>
```

Use the role in `playbooks/site.yml`:

```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  roles:
    - role: webserver
      vars:
        webserver_page_title: Web fleet managed by a reusable role
```

Run it from the project root:

```bash
ansible-playbook playbooks/site.yml --syntax-check
ansible-playbook playbooks/site.yml --check --diff
ansible-playbook playbooks/site.yml
```

Role design points:

- Make roles focused and reusable; expose appropriate settings through defaults.
- Keep templates and static files in their correct directories.
- Notify handlers only when a configuration change requires a restart or reload.
- Use tags when operators need to select stable subsets of tasks, but keep normal full-play execution reliable.
- Pin third-party role or collection versions for repeatable builds.

## 12. Cloud provisioning and infrastructure setup

Ansible can provision cloud resources through provider APIs. For AWS, install the collection and its Python SDK on the control node:

```bash
ansible-galaxy collection install amazon.aws
python3 -m pip install --user boto3 botocore
```

Provide AWS credentials using an approved method such as an IAM role, AWS profile, or environment credentials. Do not hard-code access keys in playbooks or commit credentials. Use least-privilege IAM permissions and confirm region, network, cost, and cleanup requirements before creating resources.

Example: create an EC2 security group. This is a cloud API example, not a full VM deployment. Save as `playbooks/aws-security-group.yml`:

```yaml
---
- name: Create a restricted lab security group
  hosts: localhost
  connection: local
  gather_facts: false

  vars:
    aws_region: us-east-1
    lab_security_group: ansible-lab-web
    # Replace with the public IP/CIDR of the lab operator, e.g. 203.0.113.10/32.
    operator_ssh_cidr: 203.0.113.10/32

  tasks:
    - name: Create security group allowing SSH from the operator only
      amazon.aws.ec2_security_group:
        name: "{{ lab_security_group }}"
        description: Restricted SSH access for an Ansible training lab
        region: "{{ aws_region }}"
        rules:
          - proto: tcp
            ports: [22]
            cidr_ip: "{{ operator_ssh_cidr }}"
        state: present
      register: lab_sg

    - name: Display the security group ID
      ansible.builtin.debug:
        var: lab_sg.group_id
```

Run:

```bash
ansible-playbook playbooks/aws-security-group.yml --syntax-check
ansible-playbook playbooks/aws-security-group.yml
```

Cloud provisioning reminders:

- Provider collections may have version-specific module options; check `ansible-doc amazon.aws.ec2_security_group` for the installed version.
- Cloud API tasks run from the control node and use its credentials and network access.
- Use variables for region, image IDs, instance sizes, subnet IDs, and names.
- Add explicit tags and ownership metadata to resources where supported.
- Plan cleanup. For this example, set `state: absent` in a separate cleanup play after confirming the security group is no longer in use.
- Creating a VM requires additional decisions and identifiers: image, instance type, VPC/subnet, key pair, security groups, storage, and IAM role. Use reviewed, environment-specific values rather than guessed defaults.
- Ansible can also configure the provisioned host after its address is added to or discovered in inventory.

## 13. Safety, troubleshooting, and operational habits

1. Start with `--syntax-check`, `--list-hosts`, and a small target group.
2. Use `--check --diff` for a preview when modules support it; check mode is not a perfect simulation for every task.
3. Keep production inventory separate from lab inventory and explicitly review `--limit` values.
4. Use `become: true` only for tasks that require elevated privileges.
5. Protect SSH keys and use dedicated, least-privilege accounts.
6. Store source in Git; do not commit secrets, private keys, or unencrypted vault passwords.
7. Read task output carefully: `changed` indicates a modification, while `failed` and `unreachable` require investigation.
8. Use `-vvv` temporarily for connection diagnosis; verbose output can reveal paths or sensitive context, so review before sharing logs.
9. Avoid broad destructive tasks such as recursive deletion unless the target path and scope have been verified.
10. Use `ansible-lint` in a maintained project when available, and validate changes in a disposable environment before production.

## 14. Quick revision sheet

- Control node: runs Ansible.
- Managed node: target configured by Ansible.
- Inventory: host and group definitions.
- Configuration: `ansible.cfg` settings.
- Module: unit of work, such as `apt`, `file`, or `service`.
- Task: module invocation with parameters.
- Play: tasks mapped to selected hosts.
- Playbook: YAML file containing plays.
- Role: reusable automation structure.
- Handler: task triggered by a notification, commonly for a restart or reload.
- Variable: value used to make automation adaptable.
- Idempotence: repeated runs converge on the same state without unnecessary changes.
- `ansible all -m ping`: test Ansible connectivity.
- `ansible-playbook site.yml --syntax-check`: check YAML/playbook structure.
- `ansible-playbook site.yml --check --diff`: preview supported changes.
- `ansible-doc <module>`: read module documentation.

## 15. Suggested classroom practice sequence

1. Install Ansible and confirm the version.
2. Configure SSH access to one managed Linux VM.
3. Create inventory and run `ansible all -m ping`.
4. Run read-only ad hoc commands and inspect facts.
5. Install nginx with an ad hoc module command.
6. Write and run the hello playbook.
7. Build the web configuration playbook with a template and handler.
8. Move the web configuration into a role and override a default variable.
9. Review the AWS security group example without applying it, then run it only in an authorized training account with a restricted CIDR.
10. Discuss Git review, secret handling, idempotence, check mode, and cleanup.


## 🔁 Ansible vs Other DevOps Tools – Full Comparison Chart

| Tool                     | Category                              | Primary Purpose                             | Use Case                                               | Significance                                  | Why to Use?                                                           |
| ------------------------ | ------------------------------------- | ------------------------------------------- | ------------------------------------------------------ | --------------------------------------------- | --------------------------------------------------------------------- |
| **Ansible**              | Configuration Management & Automation | Automate server configuration & deployments | Configure VMs, install software, deploy apps, patching | Agentless, simple, YAML-based, fast execution | When you need **easy, readable, agentless automation** across servers |
| **Chef**                 | Configuration Management              | Manage complex system states                | Large-scale infra with strict state enforcement        | Strong state management using Ruby DSL        | When infra is **very large & complex** and Ruby skills exist          |
| **Puppet**               | Configuration Management              | Enforce desired system state                | Long-running server compliance                         | Mature, pull-based model                      | When **continuous enforcement** is more important than speed          |
| **Terraform**            | Infrastructure as Code (IaC)          | Provision cloud resources                   | Create VMs, VPCs, AKS/EKS, Load Balancers              | Declarative, state-based infra management     | When you need **cloud infra provisioning**, not OS config             |
| **Jenkins**              | CI/CD Tool                            | Automate build & deployment pipelines       | Build, test, deploy apps                               | Strong pipeline & plugin ecosystem            | When you need **end-to-end CI/CD orchestration**                      |
| **Azure DevOps**         | DevOps Platform                       | CI/CD + Boards + Repos                      | Azure-based enterprise DevOps                          | Deep Azure integration                        | When working in **Microsoft/Azure ecosystem**                         |
| **GitHub Actions**       | CI/CD Tool                            | Automate workflows                          | Build/test on GitHub events                            | Native GitHub automation                      | When you want **lightweight CI/CD** close to code                     |
| **Kubernetes**           | Container Orchestration               | Manage containerized apps                   | Microservices, scaling containers                      | Auto-scaling, self-healing                    | When running **container-based production workloads**                 |
| **Helm**                 | Kubernetes Package Manager            | Deploy K8s apps                             | Install complex apps on Kubernetes                     | Reusable charts                               | When managing **complex Kubernetes deployments**                      |
| **Docker**               | Containerization                      | Package applications                        | Build portable app images                              | Consistency across environments               | When you need **environment-independent apps**                        |
| **Shell Script**         | Scripting                             | Task automation                             | Small system tasks                                     | Lightweight, native                           | When automation is **simple & local only**                            |
| **CloudFormation / ARM** | IaC (Cloud-specific)                  | Provision cloud infra                       | AWS / Azure native infra                               | Vendor-supported                              | When **cloud-native templates** are required                          |

---

## 🔑 Key Takeaway (Interview Line)

> **Terraform builds infrastructure, Ansible configures it, Jenkins/Azure DevOps orchestrates it, Docker packages apps, and Kubernetes runs them at scale.**

---

## 🧠 One-Line Memory Trick

```
Infra → Terraform
Config → Ansible
CI/CD → Jenkins / Azure DevOps
Containers → Docker
Orchestration → Kubernetes
```

---


# 🚀 Ansible – Complete Guide

**Theory • Basics • Hands-On • Use Cases • Azure & AWS Comparison**

![Image](https://www.interviewbit.com/blog/wp-content/uploads/2022/06/Ansible-Architecture-1024x560.png)

![Image](https://codingpackets.com/img/blog/ansible-overview/ansible-architecture.svg)

![Image](https://withdevo.net/wp-content/uploads/2021/12/aap21-ansible-development-workflow-1.jpeg)

![Image](https://www.sokube.io/wp-content/uploads/014-a-ansible.png)

---

## 📌 Introduction

**Ansible** is an **open-source, agentless automation and configuration management tool** used to configure servers, install software, deploy applications, and automate repetitive IT tasks.

It works across:

* 🖥️ On-premise servers
* ☁️ Azure Virtual Machines
* ☁️ AWS EC2
* 🌍 Multi-cloud & hybrid environments

Ansible uses **YAML-based playbooks** and connects to systems using **SSH**, making it simple, powerful, and easy to adopt.

---

## 🎯 Objectives of This Document

* Understand Ansible fundamentals
* Learn Ansible architecture & components
* Perform hands-on practice on Azure VMs
* Understand real-world use cases
* Compare Ansible with **Azure** and **AWS** native services
* Prepare for **DevOps interviews & real projects**

---

## 🧠 Why Ansible?

* ✅ Agentless (no agent on target machines)
* ✅ Easy YAML syntax
* ✅ Idempotent execution
* ✅ Multi-cloud support
* ✅ Large module ecosystem
* ✅ Faster automation & consistency

---

## 🧩 Requirements

### 🔹 Control Node Requirements

| Component | Requirement          |
| --------- | -------------------- |
| OS        | Ubuntu 20.04 / 22.04 |
| RAM       | Minimum 2 GB         |
| CPU       | 2 vCPU (recommended) |
| Python    | Python 3.x           |
| Network   | Internet access      |
| Access    | SSH enabled          |

---

### 🔹 Managed Node Requirements

| Item  | Requirement                        |
| ----- | ---------------------------------- |
| OS    | Linux (Ubuntu, Amazon Linux, RHEL) |
| SSH   | Port 22 open                       |
| User  | Sudo access                        |
| Agent | ❌ Not required                     |

---

### 🔹 Cloud Prerequisites

#### Azure

* Azure Subscription
* Linux Virtual Machines
* Network Security Group allowing SSH (22)
* Same VNet (recommended)

#### AWS

* AWS Account
* EC2 Instances
* Security Group allowing SSH
* Optional: IAM Role (if using AWS SSM)

---

## 🏗️ Ansible Architecture

```text
          Control Node
        (Ansible Installed)
                 |
                 | SSH
                 |
     --------------------------------
     |              |              |
 Managed Node   Managed Node   Managed Node
 (Azure VM)     (AWS EC2)      (On-Prem)
```

---

## 🧱 Core Ansible Components

| Component    | Description                   |
| ------------ | ----------------------------- |
| Control Node | System running Ansible        |
| Managed Node | Target servers                |
| Inventory    | List of target hosts          |
| Playbook     | YAML automation file          |
| Module       | Task execution unit           |
| Role         | Reusable automation structure |
| Facts        | System information            |
| Vault        | Secret encryption             |

---

## 🔄 Ansible Execution Flow

```text
Run Playbook
   ↓
Read Inventory
   ↓
Connect via SSH
   ↓
Gather Facts
   ↓
Execute Tasks
   ↓
Generate Report
```

---

## 🛠️ Ansible Basics

### 🔹 Install Ansible

```bash
sudo apt update -y
sudo apt install ansible -y
ansible --version
```

---

### 🔹 Inventory File

```ini
[web]
10.0.0.4
10.0.0.5

[web:vars]
ansible_user=azureuser
```

---

### 🔹 Ad-Hoc Commands

```bash
ansible all -m ping
ansible web -m shell -a "uptime"
ansible web -m apt -a "name=nginx state=present" --become
```

---

## 📜 Ansible Playbook Example

### Install & Start Nginx

```yaml
- name: Install Nginx Web Server
  hosts: web
  become: true

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Start nginx service
      service:
        name: nginx
        state: started
        enabled: true
```

Run:

```bash
ansible-playbook nginx.yml
```

---

## 🧪 Hands-On Practice: Ansible on Azure VM

![Image](https://opensource.microsoft.com/en-us/opensource/blog/2018/05/22/media_10b2f358da0d5795414b22c931c8106b941e21082.png?format=pjpg\&optimize=medium\&width=1200)

![Image](https://i0.wp.com/rajanieshkaushikk.com/wp-content/uploads/2021/07/ansible-cover.jpg?fit=1024%2C594\&ssl=1)

![Image](https://techcommunity.microsoft.com/t5/s/gxcuf89792/images/bS00MDQ3NjA0LTU0ODU5M2lEMzRBRUUzMzUxNEI0ODM0?revision=2)

### Step 1️⃣ Create Azure Linux VMs

* One **Control Node**
* One or more **Managed Nodes**
* Ubuntu 20.04 / 22.04
* SSH enabled

---

### Step 2️⃣ Setup SSH Access

```bash
ssh-copy-id azureuser@<TARGET_VM_IP>
```

---

### Step 3️⃣ Test Connectivity

```bash
ansible web -m ping
```

---

### Step 4️⃣ Verify Installation

```bash
curl http://<VM-IP>
```

---

## ✅ Real-World Use Cases

### 🔹 OS & Infrastructure

* User & group creation
* Disk mounting
* Timezone & NTP configuration
* OS patching
* Security hardening

### 🔹 Application Automation

* Web server installation
* Middleware setup
* App deployment
* Configuration templating
* Log rotation

### 🔹 DevOps & Containers

* Docker installation
* Kubernetes node preparation
* CI/CD agent setup
* Monitoring agents installation

### 🔹 Multi-Cloud Automation

* Same playbook for Azure & AWS
* Environment consistency
* Reduced vendor lock-in

---

## ☁️ Azure Services Parallel to Ansible

| Azure Service    | Purpose                      | Limitation      |
| ---------------- | ---------------------------- | --------------- |
| Azure Automation | Runbooks (PowerShell/Python) | Agent-based     |
| VM Extensions    | One-time scripts             | Not reusable    |
| Azure DevOps     | CI/CD pipelines              | Not config mgmt |

➡️ **Ansible is better for OS & app configuration**

---

## ☁️ AWS Services Parallel to Ansible

| AWS Service               | Purpose           | Limitation    |
| ------------------------- | ----------------- | ------------- |
| AWS Systems Manager (SSM) | EC2 management    | Agent-based   |
| OpsWorks                  | Chef/Puppet       | Complex       |
| CodeDeploy                | App deployment    | Limited scope |
| User Data                 | Bootstrap scripts | One-time      |

➡️ **AWS SSM is the closest AWS alternative**

---

## 🔄 Best-Practice Tool Mapping

```text
Terraform → Infrastructure Provisioning
Ansible   → Configuration Management
CI/CD     → Build & Deployment
```

---

## 📊 Comparison Summary

| Feature          | Ansible | Azure Automation | AWS SSM |
| ---------------- | ------- | ---------------- | ------- |
| Agentless        | ✅       | ❌                | ❌       |
| Multi-Cloud      | ✅       | ❌                | ❌       |
| YAML             | ✅       | ❌                | ❌       |
| OS Configuration | ✅       | ✅                | ✅       |
| Vendor Lock-in   | ❌       | ✅                | ✅       |

---

## 🎯 Interview & Exam Key Points

* Ansible is **agentless**
* Uses **SSH**
* YAML-based
* Idempotent
* Ideal for **multi-cloud**
* Azure Automation & AWS SSM are **cloud-native**
* Terraform + Ansible is **industry standard**

---

## 🧠 One-Line Summary

> **Ansible is a universal, agentless configuration management tool, while Azure Automation and AWS Systems Manager are cloud-native alternatives with vendor lock-in.**

---

## 🚀 Future Enhancements

* Ansible Roles
* Dynamic Inventory (Azure & AWS)
* Terraform + Ansible projects
* Ansible Vault
* CI/CD integration

---


<div align="center">
<h1>🚀 Ansible</h1>
<p><strong>Built with ❤️ by <a href="https://github.com/atulkamble">Atul Kamble</a></strong></p>

<p>
<a href="https://codespaces.new/atulkamble/template.git">
<img src="https://github.com/codespaces/badge.svg" alt="Open in GitHub Codespaces" />
</a>
<a href="https://vscode.dev/github/atulkamble/template">
<img src="https://img.shields.io/badge/Open%20with-VS%20Code-007ACC?logo=visualstudiocode&style=for-the-badge" alt="Open with VS Code" />
</a>
<a href="https://vscode.dev/redirect?url=vscode://ms-vscode-remote.remote-containers/cloneInVolume?url=https://github.com/atulkamble/template">
<img src="https://img.shields.io/badge/Dev%20Containers-Ready-blue?logo=docker&style=for-the-badge" />
</a>
<a href="https://desktop.github.com/">
<img src="https://img.shields.io/badge/GitHub-Desktop-6f42c1?logo=github&style=for-the-badge" />
</a>
</p>

<p>
<a href="https://github.com/atulkamble">
<img src="https://img.shields.io/badge/GitHub-atulkamble-181717?logo=github&style=flat-square" />
</a>
<a href="https://www.linkedin.com/in/atuljkamble/">
<img src="https://img.shields.io/badge/LinkedIn-atuljkamble-0A66C2?logo=linkedin&style=flat-square" />
</a>
<a href="https://x.com/atul_kamble">
<img src="https://img.shields.io/badge/X-@atul_kamble-000000?logo=x&style=flat-square" />
</a>
</p>

<strong>Version 1.0.0</strong> | <strong>Last Updated:</strong> January 2026
</div>

---
