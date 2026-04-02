## Task 7: VPN Setup using WireGuard

In this task, I configured a secure VPN using WireGuard to enable encrypted communication between my local machine and the VM. I also extended the setup to support multiple clients.

---

### 1. Install WireGuard

WireGuard was installed on the VM:

```bash
sudo apt update
sudo apt install wireguard
```

---

### 2. Generate Server Keys

Server keys were generated using:

```bash
wg genkey | tee server_private.key | wg pubkey > server_public.key
```

---

### 3. Configure WireGuard Server

The WireGuard configuration file was created:

```bash
sudo nano /etc/wireguard/wg0.conf
```

Configuration:

```ini
[Interface]
PrivateKey = <server_private_key>
Address = 10.0.0.1/24
ListenPort = 51820
```

---

### 4. Enable IP Forwarding

IP forwarding was enabled to allow packet routing:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

To make it persistent, edit `/etc/sysctl.conf`:

```bash
sudo nano /etc/sysctl.conf
```

Added:

```ini
net.ipv4.ip_forward=1
```

Then applied:

```bash
sudo sysctl -p
```

---

### 5. Configure Firewall

#### Allow WireGuard Port

```bash
sudo ufw allow 51820/udp
```

#### Allow Forwarding

```bash
sudo ufw route allow in on wg0 out on eth0
```

#### Allow SSH from VPN subnet

```bash
sudo ufw allow from 10.0.0.0/24 to any port 22
```

---

### 6. Start WireGuard Server

```bash
sudo wg-quick up wg0
```

Verified using:

```bash
sudo wg
```

---

### 7. Configure Client 1

#### Generate Keys

```bash
wg genkey | tee client1_private.key | wg pubkey > client1_public.key
```

#### Add Client to Server

Update `/etc/wireguard/wg0.conf` on the server:

```ini
[Peer]
PublicKey = <client1_public_key>
AllowedIPs = 10.0.0.2/32
```

#### Create Client Config

Create `client1.conf` on the client machine:

```ini
[Interface]
PrivateKey = <client1_private_key>
Address = 10.0.0.2/24
DNS = 8.8.8.8
MTU = 1280

[Peer]
PublicKey = <server_public_key>
Endpoint = <VM_PUBLIC_IP>:51820
AllowedIPs = 10.0.0.0/24
PersistentKeepalive = 25
```

---

### 8. Connect Client 1

On the laptop:

```bash
sudo wg-quick up client1.conf
```

---

### 9. Debugging and Fixes

During setup, several issues were encountered and resolved:

#### Firewall Issue

- Azure NSG allowed UDP 51820, but UFW initially blocked it.
- **Fix:** Allowed port 51820/udp in UFW.

#### Routing Issue

- IP forwarding was not enabled initially.
- **Fix:** Enabled using `sysctl`.

#### SSH Access Issue

- SSH was blocked for the VPN subnet.
- **Fix:** Allowed SSH from `10.0.0.0/24` in UFW.

#### MTU Issue

- SSH connection hung despite successful ping, caused by packet fragmentation.
- **Fix:** Set `MTU = 1280` in the client configuration.

#### Authentication Issue

- Password authentication was disabled.
- **Fix:** Required explicit use of SSH key or SSH config.

---

### 10. Verification

- Ping test:
  - `ping 10.0.0.1` → successful
- SSH over VPN:
  - `ssh elias@10.0.0.1` → successful
- WireGuard status:
  - `sudo wg` showed active handshake and data transfer

---

### 11. Configure Client 2

#### Generate Keys

```bash
wg genkey | tee client2_private.key | wg pubkey > client2_public.key
```

#### Add Client to Server

Update `/etc/wireguard/wg0.conf` on the server:

```ini
[Peer]
PublicKey = <client2_public_key>
AllowedIPs = 10.0.0.3/32
```

#### Create Client Config

Create `client2.conf` on the client machine:

```ini
[Interface]
PrivateKey = <client2_private_key>
Address = 10.0.0.3/24
DNS = 8.8.8.8
MTU = 1280

[Peer]
PublicKey = <server_public_key>
Endpoint = <VM_PUBLIC_IP>:51820
AllowedIPs = 10.0.0.0/24
PersistentKeepalive = 25
```

---

### 12. Connect Client 2

```bash
sudo wg-quick up client2.conf
```

---

### 13. Final Verification

- Both clients successfully connected.
- Each client was assigned a unique IP:
  - Client 1 → `10.0.0.2`
  - Client 2 → `10.0.0.3`
- Server verified both peers using:
  ```bash
  sudo wg
  ```
- Secure communication established over VPN.

---

### Result

A secure WireGuard VPN was successfully deployed with:

- Encrypted communication between client and server.
- Multi-client support.
- Secure SSH access over the private network.
- Proper firewall and routing configuration.

This setup ensures secure remote access to the VM without exposing services directly to the public internet.

