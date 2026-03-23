## Task 6: Database Security

In this task, I installed and secured a MariaDB database server, created a least-privilege user, restricted access, and implemented automated backups.

---

### 1. MariaDB Installation

MariaDB was installed on the system:

```bash
sudo apt update  
sudo apt install mariadb-server -y  
```

The service was started and enabled:

```bash
sudo systemctl start mariadb  
sudo systemctl enable mariadb  
```

---

### 2. Securing the Database Server

The database was secured using:

```bash
sudo mysql_secure_installation  
```

The following security measures were applied:

- Removed anonymous users  
- Disallowed remote root login  
- Removed test database  
- Reloaded privilege tables  

---

### 3. Database and User Creation

Logged into MariaDB:

```bash
sudo mysql  
```

Created a database:

```sql
CREATE DATABASE secure_onboarding;  
```

Created a user with restricted access:

```sql
CREATE USER 'secure_user'@'localhost' IDENTIFIED BY 'StrongPass123!';  
```

Granted only necessary privileges:

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON secure_onboarding.* TO 'secure_user'@'localhost';  
```

Applied changes:

```sql
FLUSH PRIVILEGES;  
```

---

### 4. Authentication Fix

During testing, login for the created user initially failed due to a password mismatch.  
This was resolved by resetting the password:

```sql
ALTER USER 'secure_user'@'localhost' IDENTIFIED BY 'Test@123';  
FLUSH PRIVILEGES;  
```

After verification, the user was able to successfully log in.

---

### 5. Restricting Database Access

To prevent external access, the database was bound to localhost.

Edited configuration file:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf  
```

Ensured:

`bind-address = 127.0.0.1`

Restarted MariaDB:

```bash
sudo systemctl restart mariadb  
```

Verification:

```bash
ss -tulpn | grep 3306  
```

Output confirmed:

`127.0.0.1:3306`

This ensures that the database is only accessible locally.

---

### 6. Testing User Access

Tested login using:

```bash
mysql -u secure_user -p  
```

Successfully accessed the database and verified functionality.

---

### 7. Automated Database Backup

A backup mechanism was implemented to regularly store database snapshots.

---

#### Backup Directory

```bash
sudo mkdir /db_backup  
sudo chmod 700 /db_backup  
```

This ensures only privileged users can access backups.

---

#### Backup Script

Created at:

`/usr/local/bin/db_backup.sh`

Script contents:

```bash
#!/bin/bash

DATE=$(date +%F)
BACKUP_DIR="/db_backup"

mysqldump -u root secure_onboarding > $BACKUP_DIR/db_backup_$DATE.sql
```

---

#### Script Permissions

```bash
sudo chmod +x /usr/local/bin/db_backup.sh
```

---

#### Manual Testing

```bash
sudo /usr/local/bin/db_backup.sh
```

Verified using:

```bash
ls /db_backup
```

A `.sql` backup file was successfully created.

---

#### Automation using Cron

```bash
sudo crontab -e
```

Added:

```bash
0 3 * * * /usr/local/bin/db_backup.sh
```

This schedules the backup to run daily at 3 AM.

---

### Result

* MariaDB securely installed and configured
* Root access restricted and secured
* Database created with least-privilege user
* Remote access disabled via localhost binding
* Automated backup system implemented

This ensures secure database operation, controlled access, and reliable data backup.
