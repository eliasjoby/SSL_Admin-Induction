## Task 8: Docker Fundamentals and Portfolio Website Deployment

In this task, I set up Docker on the VM and deployed a simple portfolio website using a containerized Nginx server. The website was exposed using a reverse proxy configured on the host system.

---

### 1. Docker Installation and Setup

Docker was installed on the VM:

```bash
sudo apt update
sudo apt install docker.io -y
```

Docker service was started and enabled on boot:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

The user was added to the docker group for non-root usage:

```bash
sudo usermod -aG docker elias
newgrp docker
```

Docker installation was verified using:

```bash
docker run hello-world
```

---

### 2. Creating the Portfolio Website

A basic static website was created:

```bash
mkdir ~/portfolio
cd ~/portfolio
nano index.html
```

The HTML file contained a simple portfolio page.

---

### 3. Containerizing the Website

The website was deployed using an Nginx Docker container:

```bash
docker run -d \
  -p 8080:80 \
  --name portfolio-container \
  -v ~/portfolio:/usr/share/nginx/html \
  nginx
```

Key aspects:

* The container runs in detached mode
* Port 8080 on the VM maps to port 80 inside the container
* The website directory is mounted as a Docker volume for persistence

---

### 4. Ensuring Persistence and Auto-Start

The container was configured to restart automatically:

```bash
docker update --restart unless-stopped portfolio-container
```

This ensures the container runs automatically after system reboot.

---

### 5. Nginx Reverse Proxy Configuration (Host)

A reverse proxy was configured on the host VM to route traffic from port 80 to the Docker container.

Configuration file:

```bash
sudo nano /etc/nginx/sites-available/portfolio
```

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

The configuration was enabled:

```bash
sudo ln -s /etc/nginx/sites-available/portfolio /etc/nginx/sites-enabled/
```

The default site was removed to avoid conflicts:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Nginx was tested and restarted:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

### 6. Firewall Configuration

HTTP traffic was allowed through the firewall:

```bash
sudo ufw allow 80/tcp
```

---

### 7. Verification

The website was successfully accessed through a browser using the VM’s public IP:

```
http://<VM_PUBLIC_IP>
```

This confirmed that:

* Nginx reverse proxy is working
* Docker container is serving the website
* Public access is properly configured

---

### Result

A containerized portfolio website was successfully deployed with:

* Docker-based hosting using Nginx
* Persistent storage using Docker volumes
* Automatic container restart on boot
* Reverse proxy routing via host Nginx
* Public accessibility through the VM’s IP

This setup reflects a production-style deployment using containerization and reverse proxy architecture.
