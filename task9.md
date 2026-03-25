# 🛠️ Task 9 – Ansible Lab Implementation

## 📌 Overview

This lab demonstrates the setup and use of Ansible for configuration management using Docker containers. The implementation progresses from basic setup to advanced role-based automation, following industry best practices.

---

## 🔹 Part 1: Ansible Setup and Basics

### 🧩 Docker Network

A custom Docker network was created to enable communication between containers:

```bash
docker network create ansible-net
```

---

### 🖥️ Container Setup

Three containers were deployed:

* **Control Container** → Installed with Ansible
* **Target Containers** → Running OpenSSH server

```bash
docker run -dit --name control --network ansible-net ubuntu
docker run -dit --name target1 --network ansible-net ubuntu
docker run -dit --name target2 --network ansible-net ubuntu
```

---

### 🔐 SSH Configuration

SSH service was manually enabled inside target containers:

```bash
mkdir -p /var/run/sshd
/usr/sbin/sshd
```

---

### 🔑 SSH Key-Based Authentication

```bash
ssh-keygen -t rsa
ssh-copy-id root@target1
ssh-copy-id root@target2
```

This enabled passwordless authentication from the control node.

---

### 📄 Inventory File

```ini
[target]
target1
target2
```

---

### ⚙️ Basic Playbook

```yaml
- hosts: target
  tasks:
    - name: Ping targets
      ping:

    - name: Check disk space
      command: df -h

    - name: Show system uptime
      command: uptime
```

---

## 🔹 Part 2: Lab Configuration Management

### 🎯 Objectives Achieved

* Automated package installation
* Configured shell and editor environments
* Implemented security policies
* Used variables, templates, and conditionals
* Documented playbook with structured comments

---

### 📦 Package Installation

Due to Docker limitations, the `command` module was used instead of `apt` to avoid interactive blocking:

```yaml
- name: Update package list
  command: apt update

- name: Install required software packages
  command: apt install -y python3 git vim htop curl wget tmux
```

---

## 🧾 Bash Configuration (Template)

```bash
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias update='apt update && apt upgrade'
```

---

## 🧾 Vim Configuration (Template)

```vim
set number
set tabstop=4
set shiftwidth=4
set expandtab
syntax on
```

---

## 🔐 Security Policies

### SSH Hardening

```yaml
- name: Disable root SSH login
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^PermitRootLogin'
    line: 'PermitRootLogin no'

- name: Disable password authentication
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^PasswordAuthentication'
    line: 'PasswordAuthentication no'
```

---

### UFW Conditional Configuration

```yaml
- name: Check if UFW exists
  command: which ufw
  register: ufw_check
  ignore_errors: yes

- name: Allow SSH via UFW
  command: ufw allow 22
  when: ufw_check.rc == 0
```

---

### File Permissions

```yaml
- name: Set permissions on /etc/passwd
  file:
    path: /etc/passwd
    mode: '0644'
```

---

## ⚙️ Advanced Features Used

* **Variables** → package list
* **Templates** → bashrc and vimrc
* **Conditionals** → UFW detection
* **Serial Execution** → prevents system overload

```yaml
serial: 1
```

---

# 🔹 Part 3: Ansible Roles for Lab Management

## 📁 Role Structure

```text
ansible_lab/
  ├── site.yml
  ├── roles/
  │     ├── lab-base/
  │     │     └── tasks/main.yml
  │     ├── student-workstation/
  │     │     └── tasks/main.yml
```

---

## 🔧 lab-base Role

Responsibilities:

* System update
* Timezone configuration
* Monitoring tools installation

```yaml
- name: Update system
  command: apt update

- name: Set timezone
  copy:
    dest: /etc/timezone
    content: "UTC"

- name: Install monitoring tools
  command: apt install -y net-tools htop curl
```

---

## 💻 student-workstation Role

Responsibilities:

* Create student user
* Install development tools
* Setup workspace

```yaml
- name: Create student user
  user:
    name: student
    state: present

- name: Install development tools
  command: apt install -y git vim python3 tmux

- name: Create workspace directory
  file:
    path: /home/student/workspace
    state: directory
    owner: student
```

---

## 🚀 Role-Based Playbook

```yaml
- hosts: target
  serial: 1
  roles:
    - lab-base
    - student-workstation
```

---

# 🔹 Key Concepts Demonstrated

* Infrastructure automation using Ansible
* SSH-based remote configuration
* Idempotent system configuration
* Modular role-based architecture
* Use of templates, variables, and conditionals

---

# ⚠️ Challenges Faced

| Issue                 | Solution                        |
| --------------------- | ------------------------------- |
| SSH not persistent    | Manually started sshd           |
| apt module hanging    | Replaced with command module    |
| systemd not available | Avoided timedatectl/hostnamectl |
| resource overload     | Used `serial: 1`                |

---

# 🚀 Conclusion

This lab successfully demonstrates a complete Ansible automation workflow, evolving from basic configuration to modular, role-based infrastructure management. The implementation reflects real-world DevOps practices, ensuring scalability, maintainability, and robustness.
