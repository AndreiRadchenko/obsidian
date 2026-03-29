---
title: Proxmox as Long-Term Linux Environment with MacBook as Thin Client
created: 2026-03-29
updated: 2026-03-29
tags:
  - proxmox
  - macbook
  - vscode
  - remote-ssh
  - thin-client
  - workflow
  - project
---

# Proxmox as Long-Term Linux Environment with MacBook as Thin Client

## Goal
Use Proxmox as the persistent Linux workstation/server environment and use the MacBook only as a thin client through VS Code Remote SSH. The main idea is that all code, services, dependencies, and long-running tasks live on the Proxmox VM or LXC, while the MacBook stays lightweight and mostly acts as a terminal, editor, and browser.

## Architecture
- Proxmox host runs the durable Linux environment.
- One Debian app VM hosts development tools, bot code, n8n, and optional file or AI services.
- Cloudflared or other internal services remain on the internal `vmbr1` network.
- MacBook connects over SSH using VS Code Remote SSH.
- Local Mac storage is only for documents, downloads, and temporary files.

## Step 1: Prepare the Proxmox side
1. Make sure the app VM is installed and reachable over the LAN network.
2. Confirm the VM has a stable LAN IP on `vmbr0` and, if needed, a second internal IP on `vmbr1`.
3. Enable `qemu-guest-agent` in Proxmox for reliable IP visibility and clean shutdowns.
4. Install Docker and the base dev tools inside the VM.

## Step 2: Create a dedicated Linux user
1. Use a non-root user inside the VM, for example `andrii`.
2. Add the user to the `sudo` group.
3. Add the user to the `docker` group if Docker will be used.
4. Keep root for recovery only.

Example:
```bash
adduser andrii
usermod -aG sudo,docker andrii
```

## Step 3: Install and verify SSH
1. Install the SSH server inside the VM.
2. Confirm SSH works from another machine on the network.
3. Use a stable hostname or static IP so VS Code can reconnect without extra steps.
4. Optionally add SSH keys for passwordless login.
		
		ssh-keygen -t ed25519 -C "macbook-air-m3"
		ssh-copy-id andrii@192.168.1.50

	Якщо  ssh-copy-id  не встановлений на macOS, можна вручну:
	
		cat ~/.ssh/id_ed25519.pub
		
	Потім встав цей ключ у контейнері в:

		/home/dev/.ssh/authorized_keys

	 і вистав права:
```
mkdir -p /home/dev/.ssh
chmod 700 /home/dev/.ssh
chmod 600 /home/dev/.ssh/authorized_keys
chown -R dev:dev /home/dev/.ssh
```


Example:
```bash
apt install -y openssh-server
systemctl enable --now ssh
```

### SSH hardening

Тепер налаштуй SSH-сервер. Відкрий:
```
sudo nano /etc/ssh/sshd_config
```
Зміни або додай:
```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
KbdInteractiveAuthentication no
X11Forwarding no
AllowUsers dev

```
Типові кроки hardening для Debian включають вимкнення root login і паролів, залишаючи тільки ключі.
Після цього перезапусти SSH:
```
sudo systemctl restart ssh
```
## Step 4: Set up VS Code on the MacBook
1. Install Visual Studio Code on macOS.
2. Install the `Remote - SSH` extension.
3. Add an SSH host entry in `~/.ssh/config`.
4. Connect to the Proxmox VM from VS Code.

Example SSH config:
```ssh
Host proxmox-dev
  HostName 192.168.0.181
  User andrii
  IdentityFile ~/.ssh/id_ed25519
```

## Step 5: Open the remote workspace
1. In VS Code, choose `Remote-SSH: Connect to Host...`.
2. Select the Proxmox VM host entry.
3. Open the project folder inside the VM, not on the MacBook.
4. Keep the code repository and runtime files on the VM filesystem.

## Step 6: Use the VM as the real workstation
1. Edit code in VS Code through the remote session.
2. Run containers, scripts, and tests directly on the VM.
3. Keep long-running processes inside `tmux`, `screen`, or Docker Compose.
4. Use the MacBook only for the UI, terminal, and browser.

Recommended tools inside the VM:
- Docker and Docker Compose.
- Git.
- Python and virtual environments.
- `tmux` for persistent terminal sessions.
- `htop`, `tree`, `curl`, `jq` for debugging.

## Step 7: Keep the system resilient
1. Put app services in Docker Compose files.
2. Use systemd or Docker restart policies for auto-start.
3. Keep backups of VM disk snapshots and important data directories.
4. Use Cloudflared or internal routing for any services that must stay private.

## Step 8: Use the internal network correctly
1. `vmbr0` is for normal LAN access and internet.
2. `vmbr1` is for private service traffic.
3. Do not assign a gateway to the internal NIC.
4. Use internal IPs like `10.10.10.x` for service-to-service access.

This is especially important if Cloudflared is expected to reach services such as Home Assistant, Immich, or future internal apps.

## Step 9: Suggested daily workflow
1. Open the MacBook.
2. Connect to the Proxmox VM with VS Code Remote SSH.
3. Edit code, compose files, and docs remotely.
4. Run tests or services on the VM.
5. Close the laptop without stopping the environment.

## Benefits
- The working environment survives MacBook shutdowns and battery loss.
- All dependencies stay in one place.
- The VM becomes the single source of truth for code and services.
- The MacBook stays fast, quiet, and low-maintenance.

## Pitfalls and solutions

| Pitfall | Solution |
|---|---|
| VS Code Remote SSH cannot connect | Verify SSH service is running and the VM IP or hostname is correct. |
| SSH session drops when laptop sleeps | Use `tmux` or keep services in Docker so they continue independently. |
| Docker containers stop after reboot | Add restart policies or use `docker compose up -d` from a startup script or systemd service. |
| Internal services become reachable from LAN by mistake | Keep the internal NIC on `vmbr1` without a gateway and use firewall/NAT rules carefully. |
| Working files are split between Mac and VM | Keep all project folders inside the VM and use the MacBook only as a client. |
| QEMU Guest Agent does not show the IP in Proxmox | Enable the agent in Proxmox VM options and reboot the VM. |
| Cloudflared cannot reach internal apps | Point Cloudflared to the internal IP on `vmbr1`, not the LAN IP. |

## Related notes
- [[Proxmox Debian VM Setup]]
- [[Proxmox-HA-Cloudflare-Tunnel-Setup]]
