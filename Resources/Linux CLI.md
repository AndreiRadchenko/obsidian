---
date_created: 2025-09-27
date_modified: 2025-09-27
document_type: resource
tags:
  - domain/software-dev
  - type/documentattion
---
[[Resources Dashboard|Resources]] / **[[Linux CLI|Linux CLI]]**
# Most Usable 100 Ubuntu Server Commands
**Overview**
Description: Top 100 linux commands

## Table of Contents

1. [[#Basic File and Folder Actions]]
2. [[#Permissions for Files and Users]]
3. [[#User and Group Management]]
4. [[#Network Diagnostics and Adjustments]]
5. [[#System Monitoring and Process Management]]
6. [[#Package Management]]
7. [[#System Administration]]
8. [[#File Compression and Archiving]]
9. [[#Disk Management]]
10. [[#Security and Firewall]]
11. [[#Backup and Restore]]

[[#Most Usable 100 Ubuntu Server Commands|Back to Top]]

## ssh connection

[[#Most Usable 100 Ubuntu Server Commands|Back to Top]] Generate a rsa keypair: (no file name needed)

`ssh-keygen`

then copy it on the server with one simple command:

`ssh-copy-id hassio@192.168.0.2`

you can now log in without password:

`ssh root@192.168.0.2`

connect to remote server:

`ssh andrii@176.126.62.89 -p 2822`

### You can use SSHFS instead of NFS.

1. Install SSH Server addon

2. Configure it to make a successful ssh root@hassio.local
3. `mkdir ~/hassio-sshfs; cd ~/hassio-sshfs`
4. `sshfs root@192.168.0.2:/config ~/hassio-sshfs`

`sshfs root@176.126.62.89:/ ~/hassio-sshfs`

`#password: andrei`

`fusermount -u ~/hassio-sshfs` -> unmount

5. ls # it works!

`/home/username/.ssh` - if authentification problem occur delete known-host file

## Basic File and Folder Actions

If your Ubuntu Server only shows 64 GB available on a 128 GB SSD, here are the **most likely reasons** and how to check/fix them:

---

### ✅ 1. **Only part of the SSD was partitioned during installation**

The Ubuntu installer may have created a partition using just 64 GB, leaving the rest unallocated.

#### 🔍 Check:

Run this command to inspect disk usage and partitions:

```bash
lsblk
```

You should see something like:

```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 119.2G  0 disk
├─sda1   8:1    0    64G  0 part /
└─sda2   8:2    0   ...   0 part [swap or other...]
```

If only part of `sda` is used, the rest is unpartitioned.

---

### ✅ 3. **If you're using LVM**, space might be in **Volume Group but not Logical Volume**

Check with:

```bash
sudo vgdisplay
sudo lvdisplay
```

If there's free space in the Volume Group:

```bash
sudo lvextend -l +100%FREE /dev/ubuntu-vg/root
sudo resize2fs /dev/ubuntu-vg/root
```
---

[[#Most Usable 100 Ubuntu Server Commands|Back to Top]]

- `ls` - List directory contents.
- `ls -l` - List with detailed information.
- `ls -a` - List hidden files.
- `cd /path/to/dir` - Change directory.
- `pwd` - Show current directory path.
- `mkdir dir_name` - Create a new directory.
- `mkdir -p /path/to/dir` - Create nested directories.
- `touch file_name` - Create an empty file.
- `cp file1 file2` - Copy a file.
- `cp -r dir1 dir2` - Copy a directory recursively.
- `mv file1 file2` - Move or rename a file.
- `rm file_name` - Remove a file.
- `rm -r dir_name` - Remove a directory and its contents.
- `cat file_name` - Display file content.
- `less file_name` - View file content page by page.
- `head -n 10 file_name` - Display the first 10 lines of a file.
- `tail -n 10 file_name` - Display the last 10 lines of a file.
- `find /path -name file_name` - Search for a file by name.
- `du -h file_or_dir` - Show disk usage of a file or directory.
- `df -h` - Display available disk space.

## Permissions for Files and Users

[[#Most Usable 100 Ubuntu Server Commands|Back to Top]]

- `chmod 644 file_name` - Set file permissions (read/write for owner, read-only
  for others).
- `chmod -R 755 dir_name` - Set directory permissions recursively.
- `chown user file_name` - Change file owner.
- `chown user:group file_name` - Change file owner and group.
- `chown -R user:group dir_name` - Change ownership recursively.
- `ls -l` - View permissions and ownership.
- `umask 022` - Set default file permissions.

## User and Group Management

[[#Most Usable 100 Ubuntu Server Commands|Back to Top]]

- `adduser username` - Add a new user.
- `passwd username` - Change a user's password.
- `deluser username` - Delete a user.
- `usermod -aG group username` - Add a user to a group.
- `groupadd group_name` - Create a new group.
- `groupdel group_name` - Delete a group.
- `groups username` - Show groups for a user.
- `id username` - Display user ID and group ID.
- `who` - Show logged-in users.
- `w` - Display active users and their activities.
- `last` - Show login history.
- `sudo visudo` - Edit sudoers file.

## Network Diagnostics and Adjustments

[[#Most Usable 100 Ubuntu Server Commands|Back to Top]]

### Setting static ip

Ethernet ports setup for one port static ip and second port - use dhcp

Check network interfaces:

```bash
ip addr show
```

Bring up the interface
```bash
sudo ip link set eno1 up
```

Setup static ip

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

```yaml                                                                         
network:
    version: 2
    ethernets:
        eno1:
            dhcp4: no
            addresses:
              - 192.168.29.100/22
            nameservers:
                addresses:
                  - 1.1.1.1
                  - 8.8.8.8
        eno2:
            dhcp4: true
```

```bash
sudo netplan apply
sudo systemctl restart systemd-networkd
networkctl status eno1
```

- `networkctl status eno2`
- `ip a show eno1`

- `ifconfig` - Display network interfaces.
- `ip addr` - Show IP addresses.
- `ping host` - Test network connectivity to a host.
- `traceroute host` - Show route to a host.
- `netstat -tuln` - Display active connections.
- `ss -tuln` - Show listening ports and services.
- `curl http://example.com` - Fetch a URL.
- `wget http://example.com` - Download a file from a URL.
- `scp file user@host:/path` - Securely copy a file to a remote host.
- `scp user@host:/path/file .` - Securely copy a file from a remote host.
- `rsync -av source/ dest/` - Synchronize files between locations.
- `nmcli dev status` - Display network status.
- `nmcli con show` - Show active network connections.
- `systemctl restart networking` - Restart networking service.

## System Monitoring and Process Management

[[#Most Usable 100 Ubuntu Server Commands|Back to Top]]

- `top` - Display running processes.
- `htop` - Interactive process viewer (install with `sudo apt install htop`).
- `ps aux` - Show detailed process list.
- `kill PID` - Terminate a process by PID.
- `killall process_name` - Terminate all processes with a specific name.
- `uptime` - Show system uptime.
- `free -h` - Display memory usage.
- `vmstat` - View system performance.
- `iostat` - Show CPU and I/O stats.
- `df -h` - Show disk usage.
- `du -sh /path` - Show directory size.
- `journalctl -xe` - View detailed system logs.
- `dmesg` - Display kernel logs.
- `sysctl -a` - Show system configuration.

## Package Management

[[#Most Usable 100 Ubuntu Server Commands|Back to Top]]

- `sudo apt update` - Update package list.
- `sudo apt upgrade` - Upgrade installed packages.
- `sudo apt install package_name` - Install a package.
- `sudo apt remove package_name` - Remove a package.
- `sudo apt autoremove` - Remove unnecessary packages.
- `dpkg -l` - List installed packages.
- `dpkg -s package_name` - Show package details.
- `apt-cache search package_name` - Search for a package.

## System Administration

[[#Most Usable 100 Ubuntu Server Commands|Back to Top]]

- `sudo systemctl start service_name` - Start a service.
- `sudo systemctl stop service_name` - Stop a service.
- `sudo systemctl restart service_name` - Restart a service.
- `sudo systemctl enable service_name` - Enable a service to start on boot.
- `sudo systemctl disable service_name` - Disable a service from starting on
  boot.
- `sudo systemctl status service_name` - Check the status of a service.
- `reboot` - Restart the system.
- `shutdown now` - Shut down immediately.
- `shutdown -r +5` - Restart the system in 5 minutes.
- `hostnamectl` - Manage the hostname.
- `timedatectl` - Show or set system time and date.

## File Compression and Archiving

[[#Most Usable 100 Ubuntu Server Commands|Back to Top]]

- `tar -cvf archive.tar /path` - Create a tar archive.
- `tar -xvf archive.tar` - Extract a tar archive.
- `tar -czvf archive.tar.gz /path` - Create a compressed tarball.
- `tar -xzvf archive.tar.gz` - Extract a compressed tarball.
- `gzip file_name` - Compress a file.
- `gunzip file_name.gz` - Decompress a file.
- `zip archive.zip file1 file2` - Create a zip archive.
- `unzip archive.zip` - Extract a zip archive.

## Disk Management

[[#Most Usable 100 Ubuntu Server Commands|Back to Top]]

- `lsblk` - List block devices.
- `fdisk -l` - Display disk partitions.
- `parted /dev/sdX` - Partition a disk interactively.
- `mkfs.ext4 /dev/sdX1` - Format a partition as ext4.
- `mount /dev/sdX1 /mnt` - Mount a partition.
- `umount /mnt` - Unmount a partition.
- `blkid` - Show UUIDs of devices.
- `df -h` - Check disk usage.
- `du -sh /path` - Show size of a directory.
- `fsck /dev/sdX1` - Check and repair filesystem.

## Security and Firewall

[[#Most Usable 100 Ubuntu Server Commands|Back to Top]]

- `ufw enable` - Enable the firewall.
- `ufw disable` - Disable the firewall.
- `ufw status` - Check firewall status.
- `ufw allow 22` - Allow SSH (port 22).
- `ufw deny 80` - Deny HTTP (port 80).
- `sudo fail2ban-client status` - Check Fail2Ban status.
- `sudo fail2ban-client restart` - Restart Fail2Ban.

## Backup and Restore

[[#Most Usable 100 Ubuntu Server Commands|Back to Top]]

- `rsync -av /source /destination` - Sync files for backup.
- `tar -czvf backup.tar.gz /path` - Archive and compress files.
- `scp user@host:/path/to/backup.tar.gz .` - Retrieve a backup.
- `dd if=/dev/sdX of=/path/backup.img` - Create a disk image.
- `dd if=/path/backup.img of=/dev/sdX` - Restore a disk image.


---
[[Resources Dashboard|Resources]] / **[[Linux CLI|Linux CLI]]**