---
date_created: 2026-03-20
date_modified: 2026-03-20
document_type: note
project: Proxmox
tags:
  - proxmox
  - "#vm"
  - immich
  - "#debian"
---

## Goal
Create a Debian VM in Proxmox for an app server that will host bot, n8n, file services, and STT/TTS. **Key**: one LAN interface for internet, one internal interface for Cloudflared private traffic [Proxmox-HA-Cloudflare-Tunnel-Setup](obsidian://open?vault=Vault&file=Projects%2FHome%20Assistant%2FProxmox%20HA%20Cloudflare%20Tunnel%20Setup).

## VM Specs
- **CPU**: 4 cores (host)
- **RAM**: 8 GB
- **Disk**: 200 GB (VirtIO SCSI)
- **OS**: Debian 12 Bookworm netinst
- **Network**: 2 NICs (`vmbr0` LAN DHCP + `vmbr1` internal static)

## Step 1: ISO Upload
```
Datacenter → Storage → local → ISO Images → Upload URL
https://debian.netforce.hosting/debian-cd/current/amd64/iso-cd/debian-13.4.0-amd64-netinst.iso
```

## Step 2: Create VM
**General**: `app-server`
**OS**: Debian 13 netinst, UEFI
**System**: q35, VirtIO SCSI
**Disk**: 200 GB, SSD emulation, IO thread
**CPU**: 4 cores host
**Memory**: 8192 MiB
**Network 1**: `vmbr0` VirtIO
**Network 2**: `vmbr1` VirtIO

## Step 3: Debian Install
1. `Install` (text)
2. English/US keyboard
3. DHCP network
4. Root password
5. User account
6. **Disk**: Guided, all in one partition
7. **Software**: SSH server + standard utilities **NO desktop**

## Step 4: First Boot (root)
```bash
apt update && apt upgrade -y
apt install -y sudo qemu-guest-agent ifupdown
systemctl enable --now qemu-guest-agent
usermod -aG sudo andrii
```

## Step 5: Network Setup (/etc/network/interfaces)
```bash
auto lo
iface lo inet loopback

auto ens18  # LAN vmbr0
iface ens18 inet dhcp

auto ens19  # Internal vmbr1
iface ens19 inet static
  address 10.10.10.2
  netmask 255.255.255.0
```
```bash
ifup -a
ip a  # ens18: 192.168.x.x, ens19: 10.10.10.4
```

## Step 6: Docker Install
```bash
apt install -y ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian bookworm stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
apt update && apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
usermod -aG docker $USER
newgrp docker
docker run hello-world
```

## Step 7: Proxmox Side
```
VM → Options → QEMU Guest Agent → Enable
VM → Reboot
```
**Summary**: IP показує обидві адреси

## Step 8: Docker Layout
```
mkdir -p ~/bot ~/n8n ~/files ~/stt-tts
```

**n8n example** (`~/n8n/docker-compose.yml`):
```yaml
version: '3.8'
services:
  n8n:
    image: n8nio/n8n:latest
    ports: ["5678:5678"]
    volumes: [n8n_/home/node/.n8n]
    environment:
      - N8N_HOST=0.0.0.0
    restart: unless-stopped
volumes:
  n8n_
```
```bash
cd ~/n8n && docker compose up -d
```

## Verification
- `ping 10.10.10.1` (Proxmox)
- `ping 8.8.8.8` (internet)
- Proxmox UI: VM IP = `192.168.0.181, 10.10.10.2`
- `docker run hello-world`

---

## Pitfalls & Fixes

| Problem | Fix |
|---------|-----|
| `sudo: command not found` | `apt install sudo` (root), `newgrp sudo` |
| `netplan: command not found` | Debian uses `/etc/network/interfaces`, not netplan |
| `ifup: command not found` | `apt install ifupdown` or use `ip` commands |
| QEMU Agent dependency failed | Proxmox: `Options → QEMU Guest Agent → Enable`, reboot VM |
| `ens19 DOWN` | `ip link set ens19 up` + static IP |
| Cloudflared no vztmpl | Storage → local → Edit → Content: `Container template` |
| Two default routes | Static NIC **NO gateway** |
| Cloudflared unreachable | Use internal IP `10.10.10.4:port` in config [[Proxmox-HA-Cloudflare-Tunnel-Setup]]

---

**Related**: [Proxmox-HA-Cloudflare-Tunnel-Setup](obsidian://open?vault=Vault&file=Projects%2FHome%20Assistant%2FProxmox%20HA%20Cloudflare%20Tunnel%20Setup) [Install Immich in vm](obsidian://open?vault=Vault&file=Projects%2FProxmox%2FInstall%20Immich%20in%20vm)