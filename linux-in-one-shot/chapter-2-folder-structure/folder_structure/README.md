# Folder Structure — Linux System Directories

## Overview

Linux follows the **Filesystem Hierarchy Standard (FHS)**, a specification that defines where different types of files live. Every Linux distribution respects this structure, so once you learn it, it applies everywhere — Ubuntu, Debian, Arch, containers, cloud VMs.

```
/
├── bin  -> /usr/bin          # Essential user binaries
├── sbin -> /usr/sbin         # System admin binaries
├── lib  -> /usr/lib          # Shared libraries
├── boot                      # Boot loader files
├── usr/                      # User-installed apps and libraries
├── var/                      # Variable data (logs, caches)
├── etc/                      # System configuration
├── home/                     # User home directories
├── root/                     # Root user's home
├── opt/                      # Optional third-party software
├── srv/                      # Service data
├── tmp/                      # Temporary files
├── run/                      # Runtime process data
├── proc/                     # Virtual FS — process info
├── sys/                      # Virtual FS — hardware/kernel info
├── dev/                      # Device files
├── mnt/                      # Temporary mount points
├── media/                    # Removable media mount points
└── data/                     # Your bind-mounted volume (Docker)
```

---

## 1. Symbolic Links (Legacy Compatibility)

Modern Linux distros have merged several old top-level directories into `/usr` and replaced the originals with symbolic links. This was done to simplify the filesystem and avoid duplication.

| Symlink | Points To | Purpose |
|---------|-----------|---------|
| `/bin -> /usr/bin` | `/usr/bin` | Essential user-facing binaries (`ls`, `cp`, `bash`) |
| `/sbin -> /usr/sbin` | `/usr/sbin` | System administration binaries (`fdisk`, `ip`, `iptables`) |
| `/lib -> /usr/lib` | `/usr/lib` | Shared libraries and kernel modules |
| `/lib64 -> /usr/lib64` | `/usr/lib64` | 64-bit shared libraries |

Verify this on your system:
```bash
ls -la / | grep "\->"
```

The symlinks exist purely for backward compatibility with old scripts and software that hardcoded paths like `/bin/sh` or `/sbin/ifconfig`.

---

## 2. Boot Directory

### `/boot`
Stores all files the bootloader needs to start the Linux kernel:

| File | Purpose |
|------|---------|
| `vmlinuz` | The compressed Linux kernel binary |
| `initrd.img` | Initial RAM disk — temporary filesystem used during boot |
| `grub/` | GRUB bootloader configuration |

```bash
ls /boot
```

> **In containers:** `/boot` is typically empty or absent. Containers don't boot — they share the host machine's kernel. The kernel is loaded once by the host OS; containers just use it.

---

## 3. `/usr` — User Programs and Libraries

`/usr` is the largest directory on most systems. It holds the bulk of installed software.

| Subdirectory | Contents |
|-------------|---------|
| `/usr/bin` | All user-facing binaries (`python3`, `git`, `curl`, `vim`) |
| `/usr/sbin` | Admin binaries not needed during early boot (`useradd`, `nginx`) |
| `/usr/lib` | Shared libraries used by programs in `/usr/bin` |
| `/usr/local/` | Software installed manually (not via package manager) |
| `/usr/share/` | Architecture-independent data (docs, icons, man pages) |
| `/usr/include/` | C/C++ header files for development |

```bash
# See what's in /usr/bin
ls /usr/bin | head -20

# Find which package owns a binary
dpkg -S /usr/bin/curl          # Debian/Ubuntu
rpm -qf /usr/bin/curl          # RHEL/Fedora
```

---

## 4. `/var` — Variable Data

Everything that changes frequently during system operation lives here.

| Subdirectory | Contents |
|-------------|---------|
| `/var/log/` | System and application logs (`syslog`, `auth.log`, `nginx/access.log`) |
| `/var/cache/` | Cached package data (`/var/cache/apt/archives/` stores downloaded `.deb` files) |
| `/var/lib/` | Persistent application state (databases, package manager state) |
| `/var/spool/` | Queued jobs — print jobs, mail queues, cron jobs |
| `/var/tmp/` | Temporary files that persist across reboots (unlike `/tmp`) |
| `/var/www/` | Default web root for Apache and Nginx |

```bash
# View system logs
tail -f /var/log/syslog

# See how much disk space logs are consuming
du -sh /var/log/*
```

---

## 5. `/etc` — Configuration Files

All system-wide configuration lives in `/etc`. No binaries here — only plain text config files.

| File/Directory | Purpose |
|---------------|---------|
| `/etc/hostname` | The machine's hostname |
| `/etc/hosts` | Static hostname-to-IP mappings |
| `/etc/passwd` | User account information |
| `/etc/shadow` | Encrypted user passwords |
| `/etc/group` | Group definitions |
| `/etc/fstab` | Filesystem mount table (what gets mounted at boot) |
| `/etc/apt/` | APT package manager config and repo list |
| `/etc/ssh/` | SSH server and client configuration |
| `/etc/nginx/` | Nginx web server configuration |
| `/etc/cron.d/` | Cron job definitions |
| `/etc/environment` | System-wide environment variables |
| `/etc/os-release` | OS name and version info |

```bash
# Check hostname
cat /etc/hostname

# Check OS version
cat /etc/os-release

# View all users on the system
cat /etc/passwd
```

> `/etc` is the first place to look when troubleshooting misconfigured services. Most applications read their config from here on startup.

---

## 6. User and Application Directories

### `/home`
Each non-root user gets a subdirectory here:
```
/home/
├── alice/
├── bob/
└── deploy/
```

The home directory stores personal files, shell config (`.bashrc`, `.zshrc`), SSH keys (`~/.ssh/`), and application settings.

```bash
# Shortcut to your home directory
cd ~
echo $HOME
```

### `/root`
The home directory for the **root user**. Kept separate from `/home` so root's files remain accessible even if `/home` is on a separate partition that fails to mount.

```bash
# Root's home
ls /root
```

### `/opt`
Used for self-contained third-party applications that don't integrate with the system package manager — software installed in its own directory with all dependencies bundled.

Common examples:
- `/opt/google/chrome/`
- `/opt/atlassian/jira/`
- `/opt/confluent/`

```bash
ls /opt
```

### `/srv`
Intended for data served by the system — web server document roots, FTP data, etc. Rarely used in practice (most web servers default to `/var/www/`).

---

## 7. Temporary and Volatile Directories

### `/tmp`
Temporary scratch space. Cleared on every reboot. World-writable — any user or process can create files here.

```bash
# Create a temp file
touch /tmp/testfile

# Check how much temp space is used
df -h /tmp
```

> Never store important data in `/tmp`. Use `/var/tmp/` if files need to survive a reboot.

### `/run`
Runtime data for processes — PID files, sockets, lock files. Populated fresh at boot from a `tmpfs` (lives in RAM).

```bash
# See what's running
ls /run

# Example: nginx stores its PID here
cat /run/nginx.pid
```

---

## 8. Virtual Filesystems

These directories don't contain real files on disk. They are **virtual filesystems** generated by the kernel in real time, providing a window into kernel and process state.

### `/proc`
Every running process has a directory `/proc/<PID>/` containing:

| File | Contents |
|------|---------|
| `/proc/<PID>/status` | Process state, memory usage, UID |
| `/proc/<PID>/cmdline` | Full command used to start the process |
| `/proc/<PID>/fd/` | Open file descriptors |
| `/proc/cpuinfo` | CPU details |
| `/proc/meminfo` | RAM usage breakdown |
| `/proc/uptime` | System uptime in seconds |
| `/proc/mounts` | Currently mounted filesystems |

```bash
# See CPU details
cat /proc/cpuinfo

# Check memory info
cat /proc/meminfo

# See open files for a process (replace 1 with actual PID)
ls -la /proc/1/fd
```

### `/sys`
Exposes the kernel's view of hardware — devices, drivers, power management:

```bash
# List all block devices
ls /sys/block

# Check CPU scaling governor
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```

### `/dev`
Device files — Linux represents hardware as files. Reading and writing these files communicates with hardware.

| Device File | Represents |
|------------|-----------|
| `/dev/sda` | First SATA/SCSI disk |
| `/dev/sda1` | First partition on that disk |
| `/dev/nvme0n1` | First NVMe SSD |
| `/dev/null` | Discards all input (write anything, get nothing back) |
| `/dev/zero` | Produces infinite stream of zero bytes |
| `/dev/random` | Cryptographically secure random bytes |
| `/dev/tty` | Current terminal |
| `/dev/stdin` | Standard input |

```bash
# Discard output by redirecting to /dev/null
command 2>/dev/null

# Create a 1GB file filled with zeros (useful for testing)
dd if=/dev/zero of=/tmp/testfile bs=1M count=1024
```

---

## 9. Mount Points

### `/mnt`
A conventional location for temporarily mounting external filesystems — USB drives, network shares, additional disks.

```bash
# Mount a USB drive
mount /dev/sdb1 /mnt

# Unmount it when done
umount /mnt
```

### `/media`
Where the system automatically mounts removable media (USB sticks, CD-ROMs) when inserted — managed by `udev` and desktop environments.

```bash
ls /media
```

### `/data`
Not a standard FHS directory — but in the Docker setup from Chapter 1, this is where your bind-mounted host folder appears inside the container:

```
--mount type=bind,source="C:/Users/<username>/Downloads/ubuntu-container",target=/data
```

Files written to `/data` inside the container are immediately visible in your Windows/macOS Downloads folder, and they persist even if the container is deleted.

```bash
# Test the bind mount
echo "hello from linux" > /data/test.txt
# Now check C:\Users\<username>\Downloads\ubuntu-container\test.txt on Windows
```

---

## Quick Reference

| Directory | Remember It As |
|-----------|---------------|
| `/etc` | **E**dit **C**onfig here |
| `/var` | **Var**iable — logs, caches, runtime data |
| `/usr` | **U**ser-installed programs |
| `/bin`, `/sbin` | **Bin**aries (symlinked into `/usr`) |
| `/home` | User personal files |
| `/tmp` | **T**hrow away — cleared on reboot |
| `/proc` | **P**rocess window into the kernel |
| `/dev` | **Dev**ice files |
| `/mnt` | **M**oun**t** external storage here |

---

**← Back to [Chapter 1: Getting Started](../../chapter-1-getting-started/README.md)**  
**Next → Chapter 2: Understanding File Permissions**
