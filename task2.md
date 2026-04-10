## Task 2: Enhanced SSH Security

In this task I configured SSH on the VM to make it more secure.

### 1. Disable Root Login
I edited the SSH configuration file and disabled direct login as root.

- sudo vim /etc/ssh/sshd_config


Changed the setting:

- PermitRootLogin no


This prevents anyone from logging in directly as root over SSH.

---

### 2. Disable Password Authentication
Since SSH keys are more secure than passwords, I disabled password login.

- PasswordAuthentication no


Now login is only possible using SSH keys.

---

### 3. Enable Public Key Authentication
Public key authentication was enabled in the SSH configuration.

- PubkeyAuthentication yes


---

### 4. Restrict SSH Access
SSH access was restricted to my user.

- AllowUsers elias


This ensures only this user can connect through SSH.

---

### 5. Fail2Ban Setup
Fail2Ban was installed to protect the server from brute-force login attempts.

Installation:

- sudo apt install fail2ban


I verified it was running with:
- sudo fail2ban-client status sshd


Fail2Ban monitors SSH logs and temporarily blocks IP addresses that repeatedly fail to log in.

---
