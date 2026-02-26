---
date_created: 2026-02-26
date_modified: 2026-02-26
document_type: resource
tags:
  - resource
  - domain/devops
  - proxmox
  - homeassistant
  - cloudflare
---
[[Resources/Resources Dashboard|Resources]] / **[[Proxmox HA Cloudflare Tunnel Setup|Proxmox HA Cloudflare Tunnel Setup]]**
# Proxmox + HA + Cloudflare Tunnel — Internal Network Setup

**Overview**
Description:: Configure Proxmox with an isolated internal bridge (`vmbr1`) so that cloudflared LXC reaches Home Assistant VM via internal network only. Designed for portable client deployments where the client LAN is unknown in advance.

---

## Architecture

```
[Internet]
     ↕ Cloudflare Tunnel (outbound only, no open ports)
[cloudflared LXC]   10.10.10.3  — vmbr1 only
[Proxmox Host]      10.10.10.1  — vmbr1 (static) + vmbr0 (DHCP from client router)
[HA VM NIC2]        10.10.10.2  — vmbr1 (static, no gateway)
[HA VM NIC1]        192.168.x.x — vmbr0 (DHCP, default gateway)
                          ↕
                   Client LAN devices (smart home)
```

**Key design rules:**
- Only NIC1 on HA VM has a default gateway (client router)
- NIC2 on HA VM has no gateway — prevents asymmetric routing
- cloudflared LXC attached to `vmbr1` only — isolated from client LAN
- NAT on Proxmox host routes `10.10.10.0/24` → internet via `vmbr0`
- `RELATED,ESTABLISHED` iptables rule must come first in FORWARD chain

---

## Step 1 — Proxmox Host: Configure Network Bridges

Edit network interfaces:

```bash
nano /etc/network/interfaces
```

```bash
auto lo
iface lo inet loopback

iface eno1 inet manual

# Client LAN — DHCP from client router
auto vmbr0
iface vmbr0 inet dhcp
    bridge-ports eno1
    bridge-stp off
    bridge-fd 0

# Internal infra network — static, isolated
auto vmbr1
iface vmbr1 inet static
    address 10.10.10.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0
```

> ⚠️ Replace `eno1` with your actual NIC name. Check with: `ip link show` or `ip addr`

Apply without reboot:

```bash
ifreload -a
```

---

## Step 2 — Proxmox Host: Enable NAT and IP Forwarding

```bash
# Enable IP forwarding permanently
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p

# Allow return traffic first (MUST be first rule)
iptables -I FORWARD 1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# NAT: masquerade internal net through client LAN interface
iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o vmbr0 -j MASQUERADE

# Forward: allow internal → external
iptables -I FORWARD -i vmbr1 -o vmbr0 -j ACCEPT

# Block: prevent client LAN from reaching internal net
iptables -I FORWARD -i vmbr0 -o vmbr1 -j DROP

# Persist rules across reboots
apt install -y iptables-persistent
netfilter-persistent save
```

Verify rules:

```bash
iptables -t nat -L POSTROUTING -n -v
iptables -L FORWARD -n -v
cat /proc/sys/net/ipv4/ip_forward   # Must return: 1
```

Expected FORWARD chain order:

```
1. ACCEPT  RELATED,ESTABLISHED   ← return traffic
2. DROP    vmbr0 → vmbr1         ← block client LAN → infra
3. ACCEPT  vmbr1 → vmbr0         ← allow infra → internet
```

---

## Step 3 — HA VM: Add Second NIC in Proxmox UI

1. Select HA VM → **Hardware** tab
2. Click **Add → Network Device**
3. Configure:
   - Bridge: `vmbr1`
   - Model: `VirtIO`
   - Firewall: unchecked
4. Click **Add**
5. **Reboot the HA VM**

After reboot, HA VM has:
- `net0` (enp6s18) → `vmbr0` — client LAN, DHCP, default gateway
- `net1` (enp6s19) → `vmbr1` — internal net, to be configured static

---

## Step 4 — Home Assistant: Configure NIC1 (Client LAN)

In HA web UI:

**Settings → System → Network**

- Interface connected to `vmbr0`:
  - IPv4: **DHCP** (default)
  - DNS: auto or `1.1.1.1`
  - ✅ This sets the **default gateway** automatically

---

## Step 5 — Home Assistant: Configure NIC2 (Internal, Static, No Gateway)

> ⚠️ Must be done via **Advanced SSH & Web Terminal** add-on, not the HA web UI.
> The web UI adds a gateway automatically — CLI avoids this.

Open **Advanced SSH & Web Terminal** in HA and run:

```bash
ha network update enp6s19 \
  --ipv4-method static \
  --ipv4-address 10.10.10.2/24
```

> `enp6s19` is the interface name for NIC2 — verify with `ha network info` if different.

**Do NOT set `--ipv4-gateway`** — this is intentional. Only one default gateway must exist.

Verify routing table from SSH terminal:

```bash
ip route
```

Expected output:

```
default via 192.168.x.1 dev enp6s18     ← single default route, client LAN only
10.10.10.0/24 dev enp6s19 scope link    ← internal net, no gateway
192.168.x.0/24 dev enp6s18 scope link
```

❌ If you see two `default` lines — the gateway was set incorrectly on NIC2. Fix:

```bash
ha network update enp6s19 \
  --ipv4-method static \
  --ipv4-address 10.10.10.2/24
# Do not include --ipv4-gateway
```

---

## Step 6 — Home Assistant: Add Trusted Proxies

Edit `configuration.yaml` (via File Editor or Studio Code Server):

```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 10.10.10.3       # cloudflared LXC IP
    - 10.10.10.0/24    # entire internal subnet
```

Restart HA: **Settings → System → Restart**

---

## Step 7 — Create Cloudflared LXC Container

### Option A: Automated Script (Recommended)

Run in **Proxmox Shell** — creates LXC container and installs cloudflared automatically:

```bash
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/cloudflared.sh)"
```

Accept defaults when prompted (Debian 12, 512MB RAM, 2GB disk).

> ⚠️ After script completes, the LXC will be on `vmbr0` (DHCP) by default.
> You must change it to `vmbr1` with static IP.

**Reconfigure network after script:**

1. **Shut down** the new cloudflared LXC
2. Proxmox UI → LXC → **Network** tab → `eth0` → **Edit**:
   - Bridge: `vmbr1`
   - IPv4: Static
   - IP: `10.10.10.3/24`
   - Gateway: `10.10.10.1`
3. **Start** the LXC

Or via Proxmox host shell (replace `101` with your LXC ID):

```bash
# Find LXC ID
pct list

# Stop LXC
pct stop 101

# Edit config
nano /etc/pve/lxc/101.conf
# Change: net0: name=eth0,bridge=vmbr0,...,ip=dhcp
# To:     net0: name=eth0,bridge=vmbr1,...,ip=10.10.10.3/24,gw=10.10.10.1

# Start LXC
pct start 101
```

Verify inside LXC:
```bash
ip addr show eth0       # Should show 10.10.10.3/24
ping -c 3 8.8.8.8       # Internet via NAT
```

---

### Option B: Manual LXC Creation

In Proxmox UI → **Create CT**:

| Setting | Value |
|---------|-------|
| Template | Debian 12 |
| RAM | 512 MB |
| Disk | 2 GB |
| Network Bridge | `vmbr1` only |
| IP | `10.10.10.3/24` |
| Gateway | `10.10.10.1` |
| DNS | `1.1.1.1` |

> ⚠️ Do NOT attach `vmbr0` to this container.

---

## Step 8 — Install Cloudflared in LXC

> Skip if you used Option A (automated script) — cloudflared is already installed.
> Proceed from Step 3 (Authenticate) below.

### Step 1: Install Package

Inside the cloudflared LXC console:

```bash
# Add Cloudflare apt repo
mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | \
  tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null

echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] \
  https://pkg.cloudflare.com/cloudflared bookworm main' | \
  tee /etc/apt/sources.list.d/cloudflared.list

apt update && apt install -y cloudflared
```

### Step 2: Verify Installation

```bash
cloudflared --version
```

### Step 3: Authenticate with Cloudflare Zero Trust

```bash
cloudflared tunnel login
```

A URL appears — open it in browser, select your domain, authorize.
Credentials file saved to `/root/.cloudflared/cert.pem`.

### Step 4: Create the Tunnel

```bash
# Create tunnel (gives you a tunnel ID)
cloudflared tunnel create madeira-tunnel

# Note the tunnel ID from output
cloudflared tunnel list
```

---

## Step 9 — Cloudflared Config File

```bash
mkdir -p /etc/cloudflared
nano /etc/cloudflared/config.yml
```

```yaml
tunnel: <YOUR-TUNNEL-ID>
credentials-file: /root/.cloudflared/<YOUR-TUNNEL-ID>.json

ingress:
  # Home Assistant
  - hostname: madeira.grykos.pp.ua
    service: http://10.10.10.2:8123
    originRequest:
      disableChunkedEncoding: true

  # Proxmox Web UI
  - hostname: madeira-proxmox.grykos.pp.ua
    service: https://10.10.10.1:8006
    originRequest:
      noTLSVerify: true

  # Catch-all (required)
  - service: http_status:404
```

Create DNS routes and enable service:

```bash
cloudflared tunnel route dns madeira-tunnel madeira.grykos.pp.ua
cloudflared tunnel route dns madeira-tunnel madeira-proxmox.grykos.pp.ua

cloudflared --config /etc/cloudflared/config.yml service install
systemctl enable cloudflared
systemctl start cloudflared
systemctl status cloudflared
```

---

## Step 10 — Sanity Checks

### From Proxmox host shell:
```bash
ping -c 3 10.10.10.2    # HA internal NIC
ping -c 3 10.10.10.3    # cloudflared LXC
```

### From cloudflared LXC:
```bash
ping -c 3 8.8.8.8                         # internet via NAT
ping -c 3 10.10.10.1                      # Proxmox host
curl -o /dev/null -s -w "%{http_code}\n" http://10.10.10.2:8123   # Must return 200
curl -k -o /dev/null -s -w "%{http_code}\n" https://10.10.10.1:8006  # Must return 200
```

### From HA SSH terminal:
```bash
ping -c 3 8.8.8.8         # internet via client LAN NIC1
ping -c 3 10.10.10.3      # cloudflared LXC via NIC2
ping -c 3 10.10.10.1      # Proxmox host
ip route                   # verify single default route
```

### From external browser:
```
https://madeira.grykos.pp.ua           # HA login page
https://madeira-proxmox.grykos.pp.ua   # Proxmox UI
```

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| LXC can't ping 8.8.8.8 | Missing `RELATED,ESTABLISHED` FORWARD rule | `iptables -I FORWARD 1 -m state --state RELATED,ESTABLISHED -j ACCEPT` |
| HA unreachable via tunnel | Missing trusted_proxies in HA | Add `10.10.10.0/24` to `http.trusted_proxies` |
| Two default routes in HA | Gateway set on NIC2 | Re-run `ha network update enp6s19` without `--ipv4-gateway` |
| Proxmox vmbr0 has no IP | DHCP failed on client network | Check physical cable, `journalctl -u networking` |
| Cloudflared tunnel unhealthy | LXC has no internet | Fix NAT/FORWARD iptables rules |
| 502 Bad Gateway on HA URL | cloudflared can't reach 10.10.10.2 | Check NIC2 is up in HA, check vmbr1 bridge |

---

## If Deploying to New Client

Checklist before shipping mini PC:

- [ ] Test full setup in your lab with simulated unknown LAN
- [ ] Reboot full system — verify everything auto-recovers
- [ ] Confirm `https://madeira.grykos.pp.ua` loads from mobile data (not local WiFi)
- [ ] Confirm `https://madeira-proxmox.grykos.pp.ua` loads remotely
- [ ] Rotate Cloudflare Access policy to your email only
- [ ] Remove any test credentials from HA

---

[[Resources/Resources Dashboard|Resources]] / **[[Proxmox HA Cloudflare Tunnel Setup|Proxmox HA Cloudflare Tunnel Setup]]**
