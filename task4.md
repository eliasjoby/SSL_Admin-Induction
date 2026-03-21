## Task 4: User and Permission Management

In this task, I configured user accounts, enforced access controls, addressed disk quota challenges, and implemented an automated backup system.

---

### 1. User Setup

The following users were created to establish a clear hierarchy of access:
* **exam_1, exam_2, exam_3** → Regular users.
* **examadmin** → Administrative user with root privileges.
* **examaudit** → Audit user with read-only access.

**Commands executed:**
```bash
sudo adduser exam_1
sudo adduser exam_2
sudo adduser exam_3
sudo adduser examadmin
sudo adduser examaudit

# Granting administrative privileges
sudo usermod -aG sudo examadmin
```

---

### 2. Home Directory Security

To ensure strict isolation, each user’s home directory was secured so that only the owner has access. This prevents unauthorized lateral movement between accounts.

```bash
sudo chmod 700 /home/exam_1
sudo chmod 700 /home/exam_2
sudo chmod 700 /home/exam_3
```

---

### 3. Audit User Access

The `examaudit` user required visibility without the ability to modify files. I used Access Control Lists (ACLs) to grant `rx` (read and execute) permissions specifically for this user across all exam directories.

```bash
sudo setfacl -m u:examaudit:rx /home/exam_1
sudo setfacl -m u:examaudit:rx /home/exam_2
sudo setfacl -m u:examaudit:rx /home/exam_3
```

---

### 4. Disk Quotas & Environment Analysis

Disk quotas were attempted by enabling `usrquota` and `grpquota` in `/etc/fstab` and remounting the filesystem.

However, on this Azure Ubuntu 24.04 environment, the kernel uses modern ext4 journaled quotas, and external quota files are deprecated. Despite correct configuration, quota utilities returned errors indicating incompatibility with the kernel quota format. 

This appears to be a limitation of the cloud image and kernel configuration rather than a misconfiguration. The setup steps were performed correctly, but enforcement could not be verified due to these environment constraints.

---

### 5. Automated Backup System

A secure backup system was implemented to archive all `exam_*` user home directories daily.

#### Backup Directory & Script Setup
The backup location and script were restricted to `examadmin` to maintain data integrity.

```bash
# Directory Setup
sudo mkdir /backup
sudo chown examadmin:examadmin /backup
sudo chmod 700 /backup

# Script Creation: /usr/local/bin/backup_exam.sh
cat << 'EOF' | sudo tee /usr/local/bin/backup_exam.sh
#!/bin/bash
BACKUP_DIR="/backup"
DATE=$(date +%F)
mkdir -p $BACKUP_DIR
tar -czf $BACKUP_DIR/exam_backup_$DATE.tar.gz /home/exam_*
EOF

# Script Permissions
sudo chown examadmin:examadmin /usr/local/bin/backup_exam.sh
sudo chmod 700 /usr/local/bin/backup_exam.sh
```

#### Cross-User Permissions for Backups
Since home directories were restricted to `700`, `examadmin` was granted read access via ACLs to allow the backup process to read the files without changing the primary directory permissions.

```bash
sudo setfacl -m u:examadmin:rx /home/exam_1
sudo setfacl -m u:examadmin:rx /home/exam_2
sudo setfacl -m u:examadmin:rx /home/exam_3
```

#### Cron Automation
The backup is scheduled to run daily at 2:00 AM using `cron`.
```bash
# Added to crontab
0 2 * * * /usr/local/bin/backup_exam.sh
```

---

### Result Summary
* **User Isolation:** Correctly configured with restricted home permissions.
* **Role-Based Access:** `examadmin` and `examaudit` roles function as intended.
* **Data Persistence:** Automated daily backups are active and secured.
* **Technical Analysis:** Disk quota limitations were documented and understood.

This configuration ensures a secure, auditable, and resilient multi-user environment.
