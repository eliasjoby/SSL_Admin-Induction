## Task 3: Firewall and Network Security

In this task, I configured a firewall using UFW (Uncomplicated Firewall) to secure the VM by controlling incoming and outgoing network traffic.

---

### 1. Install and Enable UFW

UFW was already available on the system. I configured it and enabled it to start on boot.

- sudo ufw enable  

This activates the firewall and ensures it runs automatically.

---

### 2. Default Firewall Policies

I set secure default rules to control traffic:

- sudo ufw default deny incoming  
- sudo ufw default allow outgoing  

This ensures:
- All incoming connections are blocked by default  
- Outgoing connections are allowed  

---

### 3. Allow Required Services

Only essential ports were opened to allow necessary services.

#### SSH Access (Port 22)

- sudo ufw allow 22/tcp  
- sudo ufw allow OpenSSH  

This allows secure remote login to the VM.

> Note: As per the task instructions, SSH was initially intended to be configured on a non-default port (e.g., 2222). However, due to network restrictions in the NIT environment, connections over port 2222 were not successful. Therefore, SSH was retained on the default port 22 to ensure reliable access.

---

#### HTTP (Port 80)

- sudo ufw allow 80/tcp  

This allows web traffic over HTTP.

---

#### HTTPS (Port 443)

- sudo ufw allow 443/tcp  

This allows secure web traffic over HTTPS.

---

### 4. Enable Logging

Logging was enabled to monitor firewall activity.

- sudo ufw logging on  

This helps in auditing and tracking network access attempts.

---

### 5. Verification

I verified the firewall status and rules using:

- sudo ufw status verbose  

Output confirmed:
- Firewall is active  
- Only ports 22, 80, and 443 are allowed  
- All other incoming traffic is blocked  

---

### Result

The system is now secured with a firewall that:
- Blocks all unnecessary incoming connections  
- Allows only essential services (SSH, HTTP, HTTPS)  
- Logs activity for monitoring and auditing  

Despite the inability to use a non-default SSH port due to network constraints, the firewall configuration ensures a secure and controlled access environment.