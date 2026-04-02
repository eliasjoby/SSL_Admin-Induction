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

### 6. Multi-Factor Authentication (Optional)
I configured Google Authenticator for SSH so that login requires a one-time code in addition to the SSH key.

Installed the PAM module:
- sudo apt install libpam-google-authenticator

Then ran:
- google-authenticator


This generated a QR code which I scanned using Google Authenticator on my phone.

SSH was configured to require both:

- SSH key authentication
- OTP verification code

Final login flow:
SSH key → verification code → login


---

### Verification
I verified the SSH configuration with:


- sudo sshd -T | grep authenticationmethods


Output showed:
- authenticationmethods publickey,keyboard-interactive

This confirms that SSH requires both the key and OTP for authentication.