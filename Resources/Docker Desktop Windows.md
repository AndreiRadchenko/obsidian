---
date_created: 2025-09-28
date_modified: 2025-09-28
document_type: resource
tags:
  - domain/ai
  - domain/software-dev
  - type/resource
---
[[Resources Dashboard|Resources]] / **[[Docker Desktop Windows|Docker Desktop Windows]]**
# Docker Desktop Windows Installation Guide
**Overview**
Description:: Complete guide to install and configure Docker Desktop on Windows

## System Requirements

### Minimum Requirements
- **Windows 10 64-bit**: Version 2004 (Build 19041) or higher
- **Windows 11**: Any version
- **4GB RAM** minimum (8GB recommended)
- **CPU**: Must support virtualization (Intel VT-x or AMD-V)
- **WSL 2**: Windows Subsystem for Linux 2 must be enabled

### Recommended Requirements
- **16GB RAM** for development work
- **SSD** for better performance
- **Windows 10/11 Pro** for Hyper-V features (Home edition uses WSL 2 backend)

## Installation Steps

### Step 1: Enable WSL 2 (Windows Subsystem for Linux 2)

Open PowerShell as Administrator and run:

```powershell
# Enable WSL feature
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# Enable Virtual Machine Platform
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# Restart your computer when prompted
```

### Step 2: Set WSL 2 as Default Version

After restart, open PowerShell and run:

```powershell
wsl --set-default-version 2
```

### Step 3: Install a Linux Distribution (Optional but Recommended)

Install Ubuntu from Microsoft Store or use:

```powershell
wsl --install -d Ubuntu
```

### Step 4: Download Docker Desktop

1. Go to [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)
2. Download the Windows installer
3. Run the installer (`Docker Desktop Installer.exe`)

### Step 5: Installation Process

1. **Run the installer** and follow the setup wizard
2. **Enable WSL 2 backend** during installation (recommended)
3. **Add shortcut to desktop** if desired
4. **Complete the installation** and restart when prompted

## Post-Installation Configuration

### Verify Installation

Open PowerShell or Command Prompt:

```powershell
# Check Docker version
docker --version

# Check Docker Compose version
docker-compose --version

# Run test container
docker run hello-world
```

### Configure Docker Settings

1. **Right-click Docker tray icon** → **Settings**
2. **General tab**:
   - Start Docker Desktop when you log in
   - Use WSL 2 based engine (recommended)
   - Enable Docker Compose V2
3. **Resources tab**:
   - Adjust CPU, Memory, and Disk image size
   - Configure WSL integration
4. **Docker Engine tab**:
   - Configure daemon.json settings

### WSL 2 Integration Configuration

In Docker Desktop Settings → Resources → WSL Integration:

- Enable integration with your default WSL distro
- Enable integration with additional distros if needed

## Common Issues and Solutions

### Issue: "Docker Desktop requires a newer WSL kernel version"

**Solution**: Update WSL kernel:

```powershell
wsl --update
```

### Issue: Virtualization not enabled in BIOS

**Solution**:
1. Restart computer and enter BIOS/UEFI settings
2. Enable Intel VT-x or AMD-V virtualization
3. Enable Hyper-V in BIOS if available

### Issue: WSL 2 not working properly

**Solution**:

```powershell
# Check WSL status
wsl --list --verbose

# Set default version to WSL 2
wsl --set-default-version 2

# Update WSL
wsl --update
```

### Issue: Docker daemon not starting

**Solution**:
1. Check Windows Services: Ensure Docker Desktop Service is running
2. Reset Docker Desktop: Right-click tray icon → Troubleshoot → Reset to factory defaults

## Performance Optimization

### WSL 2 Memory Management

Create or edit `%UserProfile%\.wslconfig`:

```ini
[wsl2]
memory=8GB      # Limit memory usage
processors=4    # Limit CPU cores
swap=2GB        # Swap space size
localhostForwarding=true
```

### Docker Disk Image Location

To move docker-desktop-data to a different drive:

```powershell
# Export WSL distribution
wsl --export docker-desktop-data "D:\docker\docker-desktop-data.tar"

# Unregister current distribution
wsl --unregister docker-desktop-data

# Import to new location
wsl --import docker-desktop-data "D:\docker\data" "D:\docker\docker-desktop-data.tar" --version 2
```

## Useful Docker Desktop Features

### Kubernetes Integration

Docker Desktop includes a single-node Kubernetes cluster:

1. Enable Kubernetes in Settings → Kubernetes
2. Check status: `kubectl cluster-info`
3. Deploy sample: `kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4`

### Dev Environments

Create isolated development environments:

```bash
docker dev init my-project
cd my-project
docker dev up
```

### Volume Management

Manage volumes through Docker Desktop GUI or CLI:

```bash
# List volumes
docker volume ls

# Create volume
docker volume create my-volume

# Inspect volume
docker volume inspect my-volume
```

## Troubleshooting Commands

### Docker System Information

```bash
docker info
docker system info
docker version
```

### WSL Diagnostics

```powershell
wsl --status
wsl --list --verbose
wsl --shutdown
```

### Reset Docker Desktop

```powershell
# Through Docker Desktop GUI: Troubleshoot → Reset
# Or manually:
wsl --unregister docker-desktop-data
wsl --unregister docker-desktop
# Then restart Docker Desktop
```

## Recommended WSL Distros for Docker

1. **Ubuntu** - Most popular, well-supported
2. **Debian** - Stable, lightweight
3. **Alpine WSL** - Minimal, fast

Install additional distros:

```powershell
wsl --install -d Ubuntu-22.04
wsl --install -d Debian
```

## Security Considerations

1. **Keep WSL updated**: Regular `wsl --update`
2. **Use trusted images**: Only pull from official repositories
3. **Manage port exposure**: Be cautious with published ports
4. **Regular backups**: Backup important WSL distributions

## Additional Resources

- [Docker Desktop Documentation](https://docs.docker.com/desktop/)
- [WSL 2 Documentation](https://docs.microsoft.com/windows/wsl/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Community Forums](https://forums.docker.com/)

## Next Steps After Installation

1. **Test with a simple container**: `docker run -it ubuntu bash`
2. **Try Docker Compose**: Create a `docker-compose.yml` file
3. **Explore sample projects**: Docker samples repository
4. **Set up development environment**: Configure your IDE with Docker

---
[[Resources Dashboard|Resources]] / **[[Docker Desktop Windows|Docker Desktop Windows]]**