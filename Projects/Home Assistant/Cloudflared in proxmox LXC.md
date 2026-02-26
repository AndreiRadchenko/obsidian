---
date_created: 2026-02-25
date_modified: 2026-02-25
document_type: note
project: Home Assistant
tags: home-assistant note
---
[[Projects/Home Assistant/Home|Home]] | [[Projects/Home Assistant/Meetings/All Meetings|Meetings]] | [[Projects/Home Assistant/Notes/All Notes|Notes]] | [[Projects/Home Assistant/References|References]]
# Cloudflared in proxmox LXC
**Overview**
Description:: 
Here is a complete guide to set up `cloudflared` in a Proxmox LXC container using the community scripts method (fastest) or manually.
## Note

## Method 1: Automated Script (Recommended)

Run this in the **Proxmox Shell** — it creates a dedicated LXC container and installs cloudflared automatically:[1][2]

```bash
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/cloudflared.sh)"
```

Accept defaults: ~512MB RAM, 2GB disk, Debian-based. The container is created and cloudflared is installed and running.[2]

## Method 2: Manual Install in Existing LXC

If you want to install cloudflared inside an **existing Debian/Ubuntu LXC**:

### Step 1: Create LXC Container (if not existing)

In Proxmox UI → **Create CT**:
- Template: Debian 12
- RAM: 512MB
- Disk: 2GB
- Network: DHCP

### Step 2: Install cloudflared

Open LXC console and run:[3]

```bash
# Add Cloudflare apt repo
mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | \
  tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null

echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] \
  https://pkg.cloudflare.com/cloudflared bookworm main' | \
  tee /etc/apt/sources.list.d/cloudflared.list

# Install
apt update && apt install -y cloudflared
```

### Step 3: Authenticate with Cloudflare Zero Trust

```bash
cloudflared tunnel login
```

A URL appears — open it in browser, select your domain, authorize.[3]

### Step 4: Create the Tunnel

```bash
# Create tunnel (gives you a tunnel ID)
cloudflared tunnel create my-tunnel

# Note the tunnel ID from output
cloudflared tunnel list
```

### Step 5: Create Config File

```bash
mkdir -p /etc/cloudflared
nano /etc/cloudflared/config.yml
```

Paste this config:[4][5]

```yaml
tunnel: <YOUR-TUNNEL-ID>
credentials-file: /root/.cloudflared/<YOUR-TUNNEL-ID>.json

ingress:
  # Home Assistant
  - hostname: ha.yourdomain.com
    service: http://192.168.1.x:8123

  # Proxmox Web UI
  - hostname: proxmox.yourdomain.com
    service: https://192.168.1.x:8006
    originRequest:
      noTLSVerify: true

  # Catch-all (required)
  - service: http_status:404
```

### Step 6: Create DNS Records

```bash
# Route DNS for each hostname
cloudflared tunnel route dns my-tunnel ha.yourdomain.com
cloudflared tunnel route dns my-tunnel proxmox.yourdomain.com
```

This auto-creates CNAME records in your Cloudflare DNS.[5][3]

### Step 7: Install as System Service

```bash
# Install and enable as systemd service
cloudflared --config /etc/cloudflared/config.yml service install

# Enable and start
systemctl enable cloudflared
systemctl start cloudflared

# Verify status
systemctl status cloudflared
```

### Step 8: Verify Tunnel

```bash
# Check tunnel is healthy
cloudflared tunnel info my-tunnel
```

In Cloudflare Dashboard → **Zero Trust → Networks → Tunnels** — status should show **Healthy**.[6][2]

## Add Access Policy (Optional but Recommended)

Protect your services with email-based login:[6]

- Cloudflare Zero Trust Dashboard → **Access → Applications → Add**
- Select hostname (`ha.yourdomain.com`)
- Policy: Allow emails matching `@yourdomain.com` or specific email list

## Multiple Services from One Tunnel

One cloudflared instance in one LXC can expose **many services** via `ingress` rules:[7][4]

```yaml
ingress:
  - hostname: ha.yourdomain.com
    service: http://192.168.1.10:8123
  - hostname: proxmox.yourdomain.com
    service: https://192.168.1.10:8006
    originRequest:
      noTLSVerify: true
  - hostname: nextcloud.yourdomain.com
    service: http://192.168.1.11:80
  - service: http_status:404
```

Restart after changes:
```bash
systemctl restart cloudflared
```

## Key Benefits

- **No port forwarding** needed on client router[6]
- **No static IP** required — works perfectly for your client deployment scenario
- **Free** on Cloudflare Zero Trust plan (up to 50 users)[2]
- One LXC container handles tunnels for all VMs/services on the same Proxmox host[4][7]

Sources
[1] Easiest Way to Set Up a Cloudflared Tunnel in Proxmox! https://www.youtube.com/watch?v=K7mjfnb22v4
[2] Cloudflare Tunnel LXC on Proxmox https://www.youtube.com/watch?v=mU2prjwerjk
[3] Create a locally-managed tunnel · Cloudflare One docs https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/do-more-with-tunnels/local-management/create-local-tunnel/
[4] Cloudflare Tunnel for multiple LXC containers in Proxmox https://www.answeroverflow.com/m/1229687725909737492
[5] GitHub - sfnemis/proxmox-traefikproxy-cloudflaretunnel: Proxmox LXC based Traefik Reverse Proxy with Cloudflare Tunnel https://github.com/sfnemis/proxmox-traefikproxy-cloudflaretunnel
[6] Secure Proxmox Access Anywhere Without VPN | Cloudflare Tunnel Setup Tutorial (2025 Guide) https://www.youtube.com/watch?v=3reG7IxoN90
[7] Cloudflared LXC + multiple VM's https://www.reddit.com/r/Proxmox/comments/1fd6072/cloudflared_lxc_multiple_vms/
[8] Installing Cloudflare Tunnels on Proxmox Via ... https://community.bigbeartechworld.com/t/installing-cloudflare-tunnels-on-proxmox-via-cloudflared-lxc-network-error/1177
[9] Secure Self-Hosting: Proxmox LXC with Traefik and Cloudflare Tunnel https://www.reddit.com/r/selfhosted/comments/1j66jca/secure_selfhosting_proxmox_lxc_with_traefik_and/
[10] Proxmox - How to install and configure cloudflare tunnels on proxmox https://www.youtube.com/watch?v=BEeJpQjUglc
[11] Run as a service on Linux https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/do-more-with-tunnels/local-management/as-a-service/linux/
[12] Securing and Isolating Proxmox with Cloudflare Tunnel https://www.vijay-narayanan.com/posts/proxmox/secure-proxmox-with-cloudflare-tunnel/
[13] GitHub - Cloudflared-for-LXC: This is the steps to use ... https://www.reddit.com/r/LXC/comments/1qy752e/github_cloudflaredforlxc_this_is_the_steps_to_use/
[14] How to Set Up Cloudflare DDNS in a Proxmox LXC Container https://www.youtube.com/watch?v=RxnUxFmJwXI
[15] Cloudron + Proxmox + Cloudflare tunnels https://forum.cloudron.io/topic/10395/cloudron-proxmox-cloudflare-tunnels


## References


---
[[Projects/Home Assistant/Home|Home]] | [[Projects/Home Assistant/Meetings/All Meetings|Meetings]] | [[Projects/Home Assistant/Notes/All Notes|Notes]] | [[Projects/Home Assistant/References|References]]
