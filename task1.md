## Task 1: Initial Setup

### Creating an Ubuntu VM on Azure

For this task, I created a virtual machine using Microsoft Azure. I activated the **Azure for Students** subscription using my institute email ID.

Steps I followed:

1. Logged in to the Azure Portal.
2. Navigated to **Virtual Machines → Create → Azure Virtual Machine**.
3. Created a new resource group called `labadmin-rg`.
4. Named the VM `labadmin-vm`.
5. Selected **Ubuntu Server 24.04 LTS** as the operating system.
6. Chose a small VM size (`B2ats_v2`) since the task only required a basic machine.
7. Configured authentication using **SSH public key**.
8. Allowed inbound traffic only on **SSH (port 22)**.
9. Created a **static public IP** for the VM.

After deployment, the VM was successfully created and was accessible through SSH.

Public IP of my VM:
104.208.66.85


### Connecting to the VM

To connect to the server, I used the SSH key that was generated during VM creation.

Command: 
ssh -i labadmin-vm_key.pem elias@104.208.66.85


After logging in, I verified the system by running basic commands like `whoami` and `uname -a`.

---

### System Updates

Once inside the VM, I updated the package lists and upgraded installed packages:

- sudo apt update
- sudo apt upgrade -y


Then I installed and enabled **unattended-upgrades** so that the system automatically installs important security updates.

- sudo apt install unattended-upgrades
- sudo dpkg-reconfigure unattended-upgrades


This ensures the system stays updated with security patches.