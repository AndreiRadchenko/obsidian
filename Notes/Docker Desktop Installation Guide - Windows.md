---
tags:
  - domain/ai
  - domain/software-dev
  - type/resource
created: 2025-09-29
---

# Docker Desktop Installation Guide - Windows

## Prerequisites

Before installing Docker Desktop on Windows, ensure your system meets these requirements:

- **Windows 10 64-bit**: Pro, Enterprise, or Education (Build 19041 or higher)
- **Windows 11 64-bit**: Pro, Enterprise, or Education
- **Hardware requirements**:
  - 64-bit processor with Second Level Address Translation (SLAT)
  - 4GB system RAM minimum
  - BIOS-level hardware virtualization support must be enabled
- **WSL 2** (Windows Subsystem for Linux 2) - recommended backend

## Step 1: Enable WSL 2

WSL 2 is the recommended backend for Docker Desktop on Windows as it provides better performance.

1. **Open PowerShell as Administrator** and run:
```powershell
wsl --install
```

2. This command will:
   - Enable the WSL feature
   - Enable the Virtual Machine Platform feature
   - Download and install the latest Linux kernel
   - Set WSL 2 as the default version
   - Install Ubuntu as the default Linux distribution

3. **Restart your computer** when prompted.

4. After restart, verify WSL 2 installation:
```powershell
wsl --list --verbose
```

## Step 2: Enable Virtualization in BIOS

Docker requires hardware virtualization to be enabled:

1. Restart your computer and enter BIOS/UEFI settings (usually by pressing F2, F10, Del, or Esc during startup)
2. Look for virtualization settings (may be called "Intel VT-x", "AMD-V", "Virtualization Technology", or "SVM Mode")
3. Enable the virtualization option
4. Save changes and exit BIOS

## Step 3: Download Docker Desktop

1. Go to the official Docker website: https://www.docker.com/products/docker-desktop
2. Click "Download for Windows"
3. The installer file will be named something like `Docker Desktop Installer.exe`

## Step 4: Install Docker Desktop

1. **Run the installer** by double-clicking the downloaded file
2. When prompted, ensure the **"Use WSL 2 instead of Hyper-V"** option is selected
3. Follow the installation wizard
4. Click "Close" when installation is complete
5. **Restart your computer** if prompted

## Step 5: Start Docker Desktop

1. Launch Docker Desktop from the Start menu
2. Accept the Docker Subscription Service Agreement
3. Optionally sign in with your Docker Hub account (or skip)
4. Wait for Docker to start (you'll see the Docker icon in the system tray)

## Step 6: Verify Installation

Open PowerShell or Command Prompt and run these commands:

```powershell
# Check Docker version
docker --version

# Check Docker Compose version
docker compose version

# Run a test container
docker run hello-world
```

If the hello-world container runs successfully, Docker is working correctly!

## Common Issues and Solutions

### Issue: WSL 2 installation fails

**Solution**: Manually enable required Windows features:
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
Then restart and install WSL 2 kernel update from Microsoft's website.

### Issue: "Hardware assisted virtualization and data execution protection must be enabled in the BIOS"

**Solution**: Enter BIOS and enable virtualization as described in Step 2.

### Issue: Docker Desktop won't start

**Solution**: 
- Check that Windows features are enabled: Open "Turn Windows features on or off" and ensure "Virtual Machine Platform" and "Windows Subsystem for Linux" are checked
- Restart Docker Desktop from the system tray icon
- Run Docker Desktop as Administrator

### Issue: Slow performance

**Solution**:
- Ensure WSL 2 is being used (not Hyper-V)
- Allocate more resources in Docker Desktop Settings → Resources
- Store project files in the WSL 2 filesystem for better I/O performance

## Configuration Tips

### Optimize Docker Desktop Resources

1. Open Docker Desktop
2. Go to Settings → Resources
3. Adjust:
   - **CPUs**: Allocate at least 2 CPUs
   - **Memory**: Allocate at least 4GB RAM
   - **Disk image size**: Increase if needed

### Configure Docker to Start on Boot

1. Open Docker Desktop Settings
2. Go to General
3. Check "Start Docker Desktop when you log in"

### WSL Integration

1. Go to Settings → Resources → WSL Integration
2. Enable integration with your WSL distributions
3. This allows you to run Docker commands directly from WSL terminals

## Basic Docker Commands

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# List images
docker images

# Pull an image
docker pull <image-name>

# Run a container
docker run <image-name>

# Stop a container
docker stop <container-id>

# Remove a container
docker rm <container-id>

# Remove an image
docker rmi <image-id>

# View logs
docker logs <container-id>

# Execute command in running container
docker exec -it <container-id> bash
```

## Next Steps

- Learn about Docker Compose for multi-container applications
- Explore Docker Hub for pre-built images
- Practice creating your own Dockerfiles
- Set up development environments using Docker containers

## Resources

- Official Docker Documentation: https://docs.docker.com/
- Docker Hub: https://hub.docker.com/
- WSL 2 Documentation: https://docs.microsoft.com/en-us/windows/wsl/
- Docker Desktop for Windows docs: https://docs.docker.com/desktop/windows/

