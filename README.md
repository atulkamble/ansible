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
