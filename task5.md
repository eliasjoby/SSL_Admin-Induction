## Task 5: Web Server Deployment and Secure Configuration

In this task, I deployed a web server using Nginx, configured reverse proxy routing for multiple backend applications, secured access, enabled HTTPS, and implemented security headers.

---

### 1. Nginx Installation and Setup

Nginx was installed and configured as the primary web server.

```bash
sudo apt update  
sudo apt install nginx -y  
```

The service was started and enabled:

```bash
sudo systemctl start nginx  
sudo systemctl enable nginx  
```

Verification:

```bash
sudo systemctl status nginx  
```

The default Nginx page was accessible via the VM's public IP.

---

### 2. Non-Privileged User for Applications

As required, backend applications were run using a non-privileged user.

```bash
sudo adduser appuser  
```

All applications were executed under this user to follow the principle of least privilege.

---

### 3. Backend Application Setup

#### App1 (Port 8008)

Downloaded and executed:

```bash
wget https://do.edvinbasil.com/ssl/app  
chmod +x app  
./app  
```

The application runs on:

`http://localhost:8008`

---

#### App2 (Port 3000)

Cloned and set up:

```bash
git clone https://gitlab.com/tellmeY/issslopen.git  
cd issslopen  
npm install  
```

The application required the `bun` runtime, which was installed:

```bash
curl -fsSL https://bun.sh/install | bash  
```

The binary was made globally available:

```bash
sudo cp ~/.bun/bin/bun /usr/local/bin/  
```

Application started using:

```bash
npm start  
```

The app runs on:

`http://localhost:3000`

---

### 4. Reverse Proxy Configuration

Nginx was configured to route incoming requests to backend applications.

Configuration file:

`/etc/nginx/sites-available/default`

The following location blocks were added inside the server block:

```nginx
location /server1/ {
    proxy_pass http://localhost:8008/;
}

location /server2/ {
    proxy_pass http://localhost:3000/;
}

location /sslopen {
    proxy_pass http://localhost:3000;
}
```

Nginx was restarted:

```bash
sudo systemctl restart nginx
```

---

### 5. Restricting Direct Access to Backend Services

To ensure backend applications were not directly exposed to the internet, firewall rules were applied:

```bash
sudo ufw deny 8008
sudo ufw deny 3000
```

This allows access only through Nginx while blocking external direct access.

---

### 6. Domain Configuration

A domain was assigned:

`zonkaroo.sslnitc.site`

The Nginx configuration was updated:

```nginx
server_name zonkaroo.sslnitc.site;
```

DNS resolution was verified to ensure the domain points to the VM.

---

### 7. HTTPS Configuration (SSL)

Certbot was used to obtain and configure SSL certificates:

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx
```

The domain was configured and HTTP traffic was redirected to HTTPS.

The application is now accessible securely via:

* [https://zonkaroo.sslnitc.site](https://zonkaroo.sslnitc.site)

---

### 8. Content Security Policy (CSP)

To improve security and prevent XSS attacks, a Content Security Policy header was added:

```nginx
add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self';";
```

Nginx was restarted after applying changes.

---

### 9. Verification

The setup was tested using the following endpoints:

* [https://zonkaroo.sslnitc.site/server1/](https://zonkaroo.sslnitc.site/server1/)
* [https://zonkaroo.sslnitc.site/server2/](https://zonkaroo.sslnitc.site/server2/)
* [https://zonkaroo.sslnitc.site/sslopen](https://zonkaroo.sslnitc.site/sslopen)

Results:

* `/server1` correctly routed to App1
* `/server2` correctly routed to App2
* `/sslopen` successfully loaded the IsSSLOpen application
* HTTPS was functional
* Direct access to backend ports was blocked

---

### Result

* Nginx successfully configured as a reverse proxy
* Multiple backend services routed correctly
* Applications secured behind the proxy
* HTTPS enabled using valid SSL certificates
* CSP implemented for additional security

This setup ensures a secure, scalable, and production-like web server architecture.
```
