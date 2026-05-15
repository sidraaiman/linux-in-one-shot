# Package Managers in Linux

## What is a Package Manager?

A **package manager** is a tool that automates the process of installing, updating, configuring, and removing software on a Linux system. It handles dependency resolution automatically — meaning if software A requires software B to work, the package manager installs both without you having to track that down manually.

Without a package manager, you would have to:
- Manually find and download software
- Resolve all dependencies yourself
- Track installed versions
- Manually remove leftover files when uninstalling

Package managers eliminate all of this.

---

## How a Package Manager Works

### 1. Repositories (Repos)
A **repository** is a remote server that stores a curated collection of software packages. Your package manager knows which repos to check based on a local configuration file.

- Ubuntu: `/etc/apt/sources.list` or `/etc/apt/sources.list.d/`
- Fedora/RHEL: `/etc/yum.repos.d/`
- Arch: `/etc/pacman.conf`

### 2. Installing Software
When you run `apt install nginx`, the package manager:
1. Checks the local package index (cached list of available packages)
2. Resolves all dependencies nginx needs
3. Downloads the package(s) from the repository
4. Installs and configures everything automatically

### 3. Updating Software
A single command refreshes the package index and upgrades all installed packages to their latest versions — no manual tracking needed.

### 4. Removing Software
Packages are removed cleanly. With `autoremove`, orphaned dependencies that are no longer needed are also purged.

---

## Popular Package Managers by Distro

| Distro | Package Manager | Package Format |
|--------|----------------|----------------|
| Ubuntu, Debian | `apt` / `dpkg` | `.deb` |
| Fedora, RHEL, CentOS | `dnf` (older: `yum`) | `.rpm` |
| Arch Linux | `pacman` | `.pkg.tar.zst` |
| OpenSUSE | `zypper` | `.rpm` |
| Alpine | `apk` | `.apk` |

---

## Repository Configuration

### Ubuntu Repository Entry (`/etc/apt/sources.list.d/ubuntu.sources`)

```
Types: deb
URIs: http://ports.ubuntu.com/ubuntu-ports/
Suites: noble noble-updates noble-backports noble-security
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

### What Each Field Means

| Field | Meaning |
|-------|---------|
| `Types: deb` | Binary package format (use `deb-src` for source code) |
| `URIs` | The repository server URL |
| `Suites` | Release codename + update channels (`noble` = Ubuntu 24.04) |
| `Components` | Package categories — see table below |
| `Signed-By` | GPG key used to verify package authenticity |

### Ubuntu Components Explained

| Component | Contains |
|-----------|---------|
| `main` | Official, fully supported free software |
| `universe` | Community-maintained free software |
| `restricted` | Proprietary drivers (e.g., NVIDIA) |
| `multiverse` | Software with licensing restrictions |

### Update Channels

| Suite Suffix | Purpose |
|-------------|---------|
| `noble` | Base release packages |
| `noble-updates` | Stable bug-fix and security updates |
| `noble-backports` | Newer versions backported from future releases |
| `noble-security` | Critical security patches only |

---

## Why Run `apt update` After a Fresh Install?

The Ubuntu ISO image is built at a point in time. By the time you install it, weeks or months of updates may have accumulated in the repository. Running `apt update` does **not** install anything — it refreshes the local index so your package manager knows what's currently available.

```bash
# Step 1 — Install sudo if using a minimal/container install
apt install sudo

# Step 2 — Refresh the package index
sudo apt update

# Step 3 — Upgrade all installed packages to latest versions
sudo apt upgrade -y
```

Always run `apt update` before installing any new software. Installing without updating can result in outdated packages or dependency conflicts.

---

## Essential Commands

### APT — Debian, Ubuntu

```bash
sudo apt update                  # Refresh package index from repos
sudo apt upgrade -y              # Upgrade all installed packages
sudo apt install nginx           # Install a package
sudo apt remove nginx            # Remove a package (keep config files)
sudo apt purge nginx             # Remove package + all its config files
sudo apt autoremove              # Remove unused dependency packages
sudo apt search nginx            # Search for a package by name/keyword
sudo apt show nginx              # Show package details and dependencies
sudo apt list --installed        # List all installed packages
sudo apt-cache policy nginx      # Show available and installed versions
```

### DNF — Fedora, RHEL, CentOS

```bash
sudo dnf check-update            # Check what updates are available
sudo dnf update -y               # Update all packages
sudo dnf install nginx           # Install a package
sudo dnf remove nginx            # Remove a package
sudo dnf search nginx            # Search for a package
sudo dnf info nginx              # Show package details
sudo dnf autoremove              # Remove unused dependencies
sudo dnf history                 # View transaction history
sudo dnf history undo <id>       # Roll back a specific transaction
```

> `yum` is the older command used on CentOS 7 and RHEL 7. `dnf` replaced it and is fully backward-compatible — `yum` commands still work on newer systems as aliases.

### Pacman — Arch Linux

```bash
sudo pacman -Syu                 # Sync repos and upgrade all packages
sudo pacman -S nginx             # Install a package
sudo pacman -R nginx             # Remove a package
sudo pacman -Rs nginx            # Remove package + unused dependencies
sudo pacman -Ss nginx            # Search for a package
sudo pacman -Si nginx            # Show package info
sudo pacman -Q                   # List all installed packages
sudo pacman -Qdt                 # List orphaned (unused) packages
```

### Zypper — OpenSUSE

```bash
sudo zypper refresh              # Refresh package list
sudo zypper update               # Update all packages
sudo zypper install nginx        # Install a package
sudo zypper remove nginx         # Remove a package
sudo zypper search nginx         # Search for a package
sudo zypper info nginx           # Show package details
```

### APK — Alpine Linux

```bash
apk update                       # Refresh package index
apk upgrade                      # Upgrade all packages
apk add nginx                    # Install a package
apk del nginx                    # Remove a package
apk search nginx                 # Search for a package
apk info nginx                   # Show package details
apk list --installed             # List installed packages
```

---

## Verifying Packages

Package managers verify packages using **GPG signatures** before installation. This ensures the package came from a trusted source and hasn't been tampered with.

If you add a third-party repository (e.g., Docker, Nginx official repo), you must also add its GPG key:

```bash
# Example: Adding Docker's official GPG key on Ubuntu
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

---

## Enabling Automatic Security Updates (Ubuntu)

For servers, you can configure Ubuntu to automatically apply security patches:

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

This installs updates from the `noble-security` suite automatically, without upgrading everything else.

---

## Best Practices

Always update the package index before installing:
```bash
sudo apt update && sudo apt install <package>
```

Clean up orphaned dependencies regularly:
```bash
sudo apt autoremove
```

Use `purge` instead of `remove` when you want a clean uninstall (removes config files too):
```bash
sudo apt purge <package>
```

Check what a package installs before committing:
```bash
apt show <package>
```

Pin a specific version if you need consistency across environments:
```bash
sudo apt install nginx=1.24.0-1ubuntu1
```

---

**← Back to [Setup Linux Environment](../setup-linux-environment/README.md)**  
**← Back to [Chapter 1: Getting Started](../README.md)**  
**Next → Chapter 2: The Linux Filesystem**
