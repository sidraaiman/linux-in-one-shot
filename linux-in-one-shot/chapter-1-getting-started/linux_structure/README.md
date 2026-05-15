# Linux Structure — Core Components of a Linux Machine

Understanding how Linux is structured helps you reason about *where* things happen when you run a command, install software, or troubleshoot a failure. Linux follows a layered architecture where each layer communicates only with the layer directly above or below it.

---

## Architecture Overview

```
+----------------------------------------------------+
| User Applications (Vim, Docker, Apache, etc.)     |
+----------------------------------------------------+
| Shell (Bash, Zsh, Fish, etc.)                     |  <-- Part of the OS
+----------------------------------------------------+
| System Libraries (glibc, libc, OpenSSL, etc.)     |  <-- Part of the OS
+----------------------------------------------------+
| System Utilities (ls, grep, systemctl, etc.)      |  <-- Part of the OS
+----------------------------------------------------+
| Linux Kernel (Process, Memory, FS, Network)       |  <-- Core of the OS
+----------------------------------------------------+
| Hardware (CPU, RAM, Disk, Network, Peripherals)   |
+----------------------------------------------------+
```

Each layer has a specific responsibility. A request from a user application travels *down* through the layers until it reaches hardware, and the result travels back *up*.

---

## (a) Hardware Layer

The foundation of every Linux machine is its physical components:

| Component | Role |
|-----------|------|
| **CPU** | Executes instructions from processes |
| **RAM** | Temporarily stores running process data |
| **Disk (HDD/SSD)** | Persistently stores files and the OS itself |
| **Network Interface (NIC)** | Sends and receives network packets |
| **Peripherals** | Keyboards, displays, USB devices, GPUs |

The OS never talks to hardware directly in application code. Instead, it uses **device drivers** — small kernel modules that know how to speak the specific protocol of each hardware device.

> **Example:** When you plug in a USB drive, the kernel loads the appropriate USB driver, which translates generic "read file" requests into the exact electrical signals the drive understands.

---

## (b) Linux Kernel — The Core

The kernel is the heart of Linux. It runs in **privileged mode** (also called kernel space), meaning it has unrestricted access to all hardware. Everything else runs in **user space** with restricted access and must ask the kernel for resources via **system calls**.

The kernel has five primary responsibilities:

### Process Management
- Creates, schedules, and terminates processes
- Handles **multitasking** by rapidly switching the CPU between processes (context switching)
- Manages **process priorities** — determines which process runs next
- Maintains a **process table** tracking every running process (PID, state, memory, owner)

```
Running → Waiting → Ready → Running  (simplified process lifecycle)
```

### Memory Management
- Allocates RAM to processes when requested, frees it when released
- Implements **virtual memory** — each process thinks it has its own isolated address space
- Uses **paging** to move data between RAM and swap space when memory is low
- Prevents processes from reading or writing each other's memory (isolation)

### Device Drivers
- Acts as a translator between generic OS requests and hardware-specific instructions
- Drivers are either **built into the kernel** or loaded as **kernel modules** (`.ko` files)
- Check loaded modules with: `lsmod`
- Load a module manually with: `modprobe <module_name>`

### File System Management
- Manages how data is organized, stored, and retrieved on disk
- Supports dozens of file systems: **ext4**, **xfs**, **btrfs**, **tmpfs**, **ntfs**, **fat32**
- Provides a unified interface — you read/write files the same way regardless of the underlying file system
- Maintains **inodes** — metadata records that describe each file (permissions, size, timestamps, disk location)

### Network Management
- Manages the full networking stack: Ethernet, Wi-Fi, TCP/IP, UDP, DNS
- Handles packet routing, firewall rules (via **netfilter/iptables**), and socket connections
- Every network connection your applications make goes through the kernel's network subsystem

---

## (c) System Utilities

System utilities are small command-line programs that expose kernel functionality in a human-usable form. They are part of the OS but run in user space.

| Utility | What it Does |
|---------|-------------|
| `ls` | Lists files (asks the kernel for directory contents) |
| `grep` | Searches text (reads files via kernel file system calls) |
| `ps` | Shows running processes (reads from `/proc` kernel interface) |
| `systemctl` | Controls system services (communicates with systemd) |
| `ip` / `ifconfig` | Configures network interfaces |
| `mount` | Attaches file systems to the directory tree |
| `chmod` | Changes file permissions via kernel calls |
| `kill` | Sends signals to processes |

Most utilities are part of the **GNU Core Utilities** package (`coreutils`) and ship with every Linux distribution.

---

## (d) System Libraries

System libraries sit between user applications and the kernel. Instead of every application making raw system calls (which are low-level and complex), they call library functions that handle the complexity.

| Library | Purpose |
|---------|---------|
| **glibc** (GNU C Library) | The standard C library — wraps almost every system call |
| **libpthread** | POSIX threads for concurrent programming |
| **OpenSSL / libssl** | Cryptography and TLS/SSL connections |
| **libm** | Mathematical functions |
| **libdl** | Dynamic loading of shared libraries at runtime |

> **How it works:** When your program calls `printf()`, it's calling glibc, which internally calls the `write()` system call, which asks the kernel to send bytes to the terminal.

Libraries are stored in `/lib/`, `/lib64/`, and `/usr/lib/`. You can inspect which libraries a binary depends on:
```bash
ldd /bin/ls
```

---

## (e) Shell — The Command Interpreter

The shell is the interface between you and the kernel. When you type a command, the shell:
1. Parses your input
2. Locates the program (via `$PATH`)
3. Forks a child process
4. Executes the program (which makes system calls to the kernel)
5. Returns the output to your terminal

### Common Shells

| Shell | Description |
|-------|-------------|
| **Bash** (Bourne Again Shell) | Default on most Linux distros, highly scriptable |
| **Zsh** | Extended Bash with better autocomplete and theming (default on macOS) |
| **Fish** | User-friendly, syntax highlighting out of the box |
| **Dash** | Minimal, POSIX-compliant, used for system boot scripts |
| **Ksh** (KornShell) | Common in enterprise Unix environments |

Check your current shell:
```bash
echo $SHELL
```

Switch shell temporarily:
```bash
bash    # switch to bash
zsh     # switch to zsh
```

---

## (f) User Applications

User applications are the software you install and run: text editors, web servers, databases, container engines, etc.

- Run entirely in **user space** — no direct hardware access
- Interact with the OS through the shell, system libraries, or direct system calls
- Examples: `vim`, `docker`, `apache2`, `nginx`, `python3`, `mysql`

```
Application
    ↓  calls
System Library (glibc)
    ↓  wraps
System Call (read, write, open, fork...)
    ↓  handled by
Linux Kernel
    ↓  drives
Hardware
```

---

## Key Concepts to Remember

| Concept | Meaning |
|---------|---------|
| **Kernel space** | Privileged area where the kernel runs with full hardware access |
| **User space** | Restricted area where applications and shells run |
| **System call** | The controlled gateway from user space into the kernel |
| **Device driver** | Kernel module that translates OS requests into hardware instructions |
| **Process** | A running instance of a program, managed by the kernel |
| **Inode** | Kernel data structure that stores file metadata |

---

**← Back to [Chapter 1: Getting Started](../README.md)**  
**Next → Chapter 2: The Linux Filesystem**
