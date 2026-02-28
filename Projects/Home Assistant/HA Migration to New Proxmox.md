# HA Migration to New Proxmox Node

> Complete guide: backup HA + cloudflared on source Proxmox → restore on target → reconfigure network, NAT, and Cloudflare Tunnel.

---

## Overview

```
Source (Madeira):  Proxmox → HA VM + cloudflared LXC → madeira-tunnel
Target (Kyiv):     Proxmox → HA VM + cloudflared LXC → kyiv-tunnel
```

---

## 1. Backup Home Assistant (LXC) on Source Proxmox

### 1.1 Via Proxmox Web UI
1. Open Proxmox Web UI → select **HA LXC** container
2. Go to **Backup** tab → click **Backup Now**
3. Settings:
   - Storage: `local`
   - Mode: `Snapshot` (preferred) or `Stop`
   - Compression: `zstd`
4. Click **Backup** — wait for completion
5. Note the backup file path: `/var/lib/vz/dump/vzdump-lxc-<VMID>-*.tar.zst`

### 1.2 Via CLI
```bash
# On source Proxmox shell
vzdump <VMID> --storage local --mode snapshot --compress zstd
# Example:
vzdump 100 --storage local --mode snapshot --compress zstd
```

---

## 2. Backup Cloudflared LXC on Source Proxmox

```bash
# On source Proxmox shell
vzdump <CLOUDFLARED_VMID> --storage local --mode snapshot --compress zstd
# Example:
vzdump 101 --storage local --mode snapshot --compress zstd
```

Backup saved to: `/var/lib/vz/dump/`

---

## 3. Copy Backups to Target Proxmox

### Option A: SCP (recommended)
```bash
# Run from TARGET Proxmox or any machine with SSH access to both
scp root@<SOURCE_IP>:/var/lib/vz/dump/vzdump-lxc-100-*.tar.zst root@<TARGET_IP>:/var/lib/vz/dump/
scp root@<SOURCE_IP>:/var/lib/vz/dump/vzdump-lxc-101-*.tar.zst root@<TARGET_IP>:/var/lib/vz/dump/
```

### Option B: USB Drive
```bash
# On source
cp /var/lib/vz/dump/vzdump-lxc-*.tar.zst /mnt/usb/

# On target
cp /mnt/usb/vzdump-lxc-*.tar.zst /var/lib/vz/dump/
```

---

## 4. Restore LXCs on Target Proxmox

### 4.1 Via Proxmox Web UI
1. Go to **local storage** → **Backups** tab
2. Select the `.tar.zst` backup → click **Restore**
3. Assign new VMID if needed
4. **Uncheck** "Start after restore" — configure network first
5. Repeat for both HA and cloudflared LXCs

### 4.2 Via CLI
```bash
# Restore HA LXC as VMID 100
pct restore 100 /var/lib/vz/dump/vzdump-lxc-100-*.tar.zst --storage local-lvm

# Restore cloudflared LXC as VMID 101
pct restore 101 /var/lib/vz/dump/vzdump-lxc-101-*.tar.zst --storage local-lvm
```

---

## 5. Adjust Network on Target Proxmox

### 5.1 Proxmox Host Network (`/etc/network/interfaces`)
```bash
nano /etc/network/interfaces
```

Example config for target machine:
```
auto lo
iface lo inet loopback

auto enp0s3
iface enp0s3 inet static
    address 192.168.1.100/24   # target machine LAN IP
    gateway 192.168.1.1

auto vmbr0
iface vmbr0 inet static
    address 10.10.10.0/24      # internal bridge for LXCs
    bridge-ports none
    bridge-stp off
    bridge-fd 0
```

Apply:
```bash
systemctl restart networking
# or reboot
```

### 5.2 Update LXC Network Config
```bash
# Set static IP for HA LXC
nano /etc/pve/lxc/100.conf
```

Update line:
```
net0: name=eth0,bridge=vmbr0,ip=10.10.10.2/24,gw=10.10.10.1
```

Same for cloudflared LXC:
```
net0: name=eth0,bridge=vmbr0,ip=10.10.10.3/24,gw=10.10.10.1
```

---

## 6. Configure NAT on Target Proxmox

Enable IP forwarding and NAT so LXCs can reach the internet:

```bash
# Enable IP forwarding
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p

# Add NAT rule (replace enp0s3 with your WAN interface)
iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o enp0s3 -j MASQUERADE

# Make iptables persistent
apt install iptables-persistent -y
netfilter-persistent save
```

Verify:
```bash
iptables -t nat -L -n -v
```

---

## 7. Create New Cloudflare Tunnel

### 7.1 Login and Create Tunnel
```bash
# On cloudflared LXC (target machine)
cloudflared tunnel login   # opens browser auth — paste URL if headless
cloudflared tunnel create kyiv-tunnel
```

Note the new tunnel UUID shown — e.g. `4ed2ce83-d16f-4268-8e7b-2382f14ef0b1`

### 7.2 Verify Credentials File Created
```bash
ls -la /root/.cloudflared/
# Should contain:
# cert.pem
# 4ed2ce83-d16f-4268-8e7b-2382f14ef0b1.json
```

---

## 8. Configure Cloudflared on Target

### 8.1 Write config.yml
```bash
nano /etc/cloudflared/config.yml
```

```yaml
tunnel: 4ed2ce83-d16f-4268-8e7b-2382f14ef0b1
credentials-file: /root/.cloudflared/4ed2ce83-d16f-4268-8e7b-2382f14ef0b1.json

ingress:
  - hostname: kyiv.grykos.pp.ua
    service: http://10.10.10.2:8123
    originRequest:
      disableChunkedEncoding: true

  - hostname: kyiv-proxmox.grykos.pp.ua
    service: https://10.10.10.1:8006
    originRequest:
      noTLSVerify: true

  - service: http_status:404
```

> ⚠️ Always use the tunnel **UUID** in `tunnel:` field, not the name — avoids routing to wrong tunnel.

### 8.2 Add DNS Routes
```bash
# Use UUID directly to avoid ambiguity
cloudflared tunnel route dns 4ed2ce83-d16f-4268-8e7b-2382f14ef0b1 kyiv.grykos.pp.ua
cloudflared tunnel route dns 4ed2ce83-d16f-4268-8e7b-2382f14ef0b1 kyiv-proxmox.grykos.pp.ua
```

### 8.3 Enable and Start Service
```bash
cloudflared service install
systemctl enable cloudflared
systemctl start cloudflared
systemctl status cloudflared
```

---

## 9. Verify Everything Works

```bash
# Check tunnel connections
cloudflared tunnel info 4ed2ce83-d16f-4268-8e7b-2382f14ef0b1

# Check both tunnels are separate and healthy
cloudflared tunnel list
```

Expected output:
```
4ed2ce83  kyiv-tunnel    2026-02-28  2xXXX (kyiv connectors)
b30e9298  madeira-tunnel 2026-02-25  4xXXX (madeira connectors only)
```

Test in browser:
- https://kyiv.grykos.pp.ua
- https://kyiv-proxmox.grykos.pp.ua

---

## ⚠️ Pitfalls & Solutions

### Pitfall 1: DNS Route Assigned to Wrong Tunnel
**Symptom:**
```
cloudflared tunnel route dns kyiv-tunnel kyiv.grykos.pp.ua
→ INF Added CNAME... tunnelID=b30e9298  ← wrong tunnel!
```
**Cause:** `cert.pem` still references the old tunnel as default when using tunnel name.

**Fix:** Always use the **UUID** directly instead of name:
```bash
cloudflared tunnel route dns 4ed2ce83-d16f-4268-8e7b-2382f14ef0b1 kyiv.grykos.pp.ua
```
Or use `--overwrite-dns` flag to force correct assignment:
```bash
cloudflared tunnel route dns --overwrite-dns 4ed2ce83-d16f-4268-8e7b-2382f14ef0b1 kyiv.grykos.pp.ua
```

---

### Pitfall 2: Cloudflared Fails to Start — Missing Credentials File
**Symptom:**
```
Tunnel credentials file '/root/.cloudflared/b30e9298-...json' does not exist
```
**Cause:** `config.yml` still has the old tunnel UUID after restoring from backup.

**Fix:** Update `config.yml` with the new tunnel UUID and credentials path:
```bash
nano /etc/cloudflared/config.yml
# Change both tunnel: and credentials-file: to new UUID
```
Verify the `.json` file exists:
```bash
ls -la /root/.cloudflared/
```
If missing, regenerate:
```bash
cloudflared tunnel token --cred-file /root/.cloudflared/4ed2ce83-....json 4ed2ce83-...
```

---

### Pitfall 3: Shared Tunnel vs Separate Tunnel
**Symptom:** Both machines connect under same tunnel — hard to debug which location is down.

**Rule:** Always create **one tunnel per location/machine**.
- Easier to identify which connector is offline
- Isolated credentials — compromise of one doesn't affect other
- Clean dashboard view per site

---

### Pitfall 4: LXC Has No Internet After Restore
**Cause:** NAT / IP forwarding not configured on new Proxmox host.

**Fix:**
```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf && sysctl -p
iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o <WAN_IFACE> -j MASQUERADE
netfilter-persistent save
```

---

### Pitfall 5: HA LXC Starts with Wrong IP
**Cause:** LXC config retains source machine IP after restore.

**Fix:** Edit LXC config before starting:
```bash
nano /etc/pve/lxc/100.conf
# Update net0 line with correct IP for target network
```

---

*Last updated: 2026-02-28 | Kyiv deployment*
