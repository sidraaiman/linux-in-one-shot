# Linux System Monitoring

## Introduction

System monitoring is the practice of continuously observing CPU, memory, disk, network, and log activity to ensure a Linux machine is healthy and performing well. On a server, you are often the first to know something is wrong — not because an alert fires, but because you check these metrics routinely.

This chapter covers the essential tools for every layer of monitoring: resource usage, disk health, network activity, and log analysis.

---

## 1. CPU and Memory Monitoring

### `top` — Real-Time Resource Monitor

```bash
top
```

Key header fields to watch:

| Field | What to Look For |
|-------|-----------------|
| `load average` | 1, 5, 15 min averages. If any exceed your CPU count, system is overloaded |
| `%us` | User-space CPU — high = application load |
| `%sy` | Kernel CPU — high = excessive syscalls or driver issues |
| `%wa` | I/O wait — high = disk or network bottleneck |
| `%id` | Idle — should be above 0 on a healthy system |
| `buff/cache` | Memory used for disk cache — this is normal and reclaimed when needed |

Interactive shortcuts:

| Key | Action |
|-----|--------|
| `M` | Sort by memory |
| `P` | Sort by CPU |
| `1` | Show per-core CPU breakdown |
| `u` | Filter by username |
| `k` | Kill a process |
| `d` | Change refresh interval |
| `q` | Quit |

### `htop` — Enhanced Process Viewer

```bash
# Install
apt install htop           # Debian/Ubuntu
dnf install htop           # RHEL/Fedora

htop
```

Advantages over `top`:
- Visual CPU and memory bars per core
- Mouse support
- Color-coded process states
- `F4` to filter, `F5` for tree view, `F9` for signal menu

### `vmstat` — System Performance Statistics

`vmstat` shows CPU, memory, swap, I/O, and system activity in one view:

```bash
vmstat              # Single snapshot
vmstat 1 5          # Update every 1 second, show 5 updates
vmstat -s           # Summary statistics
vmstat -d           # Disk I/O statistics
```

Output columns:

```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 3421000 204800 2456000   0    0     5    12   80  200  2  1 97  0  0
```

| Column | Meaning |
|--------|---------|
| `r` | Processes waiting to run (runqueue) — above CPU count = bottleneck |
| `b` | Processes in uninterruptible sleep (blocked on I/O) |
| `si / so` | Swap in / swap out — non-zero means RAM is exhausted |
| `bi / bo` | Blocks read / written per second (disk I/O) |
| `in` | Interrupts per second |
| `cs` | Context switches per second — very high = too many processes |
| `us / sy / id / wa` | CPU: user / system / idle / I/O wait |

### `free` — Memory Usage

```bash
free -m             # Show in megabytes
free -h             # Show in human-readable (GB, MB)
free -s 2           # Refresh every 2 seconds
```

Output:
```
              total        used        free      shared  buff/cache   available
Mem:           7982        2105        3421         215        2456        5500
Swap:          2048           0        2048
```

| Column | Meaning |
|--------|---------|
| `total` | Total installed RAM |
| `used` | RAM currently in use by processes |
| `free` | Completely unused RAM |
| `buff/cache` | RAM used as disk cache — Linux reclaims this when needed |
| `available` | RAM available for new processes (free + reclaimable cache) |

> `available` is the real number to watch — not `free`. The OS intentionally keeps `free` low by using RAM for disk caching, which speeds up file access.

### `uptime` — Quick Load Check

```bash
uptime
# 10:30:00 up 5 days, 3:22,  2 users,  load average: 0.15, 0.10, 0.08
```

---

## 2. Disk Monitoring

### `df` — Disk Space Usage (Filesystem Level)

```bash
df -h               # Human-readable (GB, MB)
df -hT              # Include filesystem type
df -h /var          # Check specific mount point
df -i               # Show inode usage instead of blocks
```

Output:
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   22G   26G  46% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
/dev/sda2       200G   80G  110G  40% /data
```

> Watch the `Use%` column. Above 90% is a warning; above 95% can cause services to fail (databases, web servers, logs all need disk space to write).

### `du` — Disk Usage (Directory Level)

```bash
du -sh /var/log             # Total size of a directory
du -sh /*                   # Size of each top-level directory
du -sh /var/log/*           # Size of each item inside a directory
du -h --max-depth=1 /var    # One level deep, human-readable
du -sh * | sort -rh | head  # Find the 10 largest items in current directory
```

### `iostat` — CPU and Disk I/O Statistics

```bash
# Install
apt install sysstat

iostat                      # Single snapshot
iostat -x 1 5               # Extended stats, every 1 sec, 5 updates
iostat -x -h 1              # Human-readable, continuous
```

Key columns in extended mode (`-x`):

| Column | Meaning |
|--------|---------|
| `r/s` | Reads per second |
| `w/s` | Writes per second |
| `rkB/s` | KB read per second |
| `wkB/s` | KB written per second |
| `await` | Average wait time for I/O (ms) — high = slow disk |
| `%util` | Disk utilization — near 100% means disk is saturated |

### `lsblk` — Block Device Layout

```bash
lsblk               # Show disk and partition layout
lsblk -f            # Include filesystem type and UUID
```

---

## 3. Network Monitoring

### `ip` — Network Interface Info (Modern)

```bash
ip a                        # Show all interfaces and IP addresses
ip a show eth0              # Show specific interface
ip -br a                    # Brief summary
ip r                        # Show routing table
ip link show                # Show interface status (UP/DOWN)
```

> `ifconfig` is deprecated and not installed by default on modern distros. Use `ip` instead.

### `netstat` — Network Connections and Ports

```bash
# Install (part of net-tools)
apt install net-tools

netstat -tulnp              # TCP/UDP listening ports with PID
netstat -anp                # All connections with PID
netstat -s                  # Network statistics summary
```

### `ss` — Socket Statistics (Modern Replacement for netstat)

```bash
ss -tulnp                   # Listening TCP/UDP ports with process info
ss -anp                     # All sockets
ss -tnp                     # TCP connections with process names
ss -s                       # Summary statistics
ss -tulnp | grep :80        # Find what's using port 80
```

`ss` output columns:

```
Netid  State   Recv-Q  Send-Q  Local Address:Port    Peer Address:Port  Process
tcp    LISTEN  0       128     0.0.0.0:80             0.0.0.0:*          users:(("nginx",pid=1235,fd=6))
```

| Column | Meaning |
|--------|---------|
| `Netid` | Protocol (tcp, udp, unix) |
| `State` | LISTEN, ESTABLISHED, TIME-WAIT, CLOSE-WAIT |
| `Local Address:Port` | Interface and port the service is bound to |
| `Peer Address:Port` | Remote side (`*` = any for listeners) |
| `Process` | Program name and PID |

### `ping` — Test Connectivity

```bash
ping google.com             # Continuous ping (Ctrl+C to stop)
ping -c 4 google.com        # Send exactly 4 packets
ping -i 0.5 google.com      # Ping every 0.5 seconds
ping -s 1400 google.com     # Test with large packet size (MTU check)
```

Output to watch:
- **Packet loss** — any loss indicates network problems
- **Round-trip time (RTT)** — spikes indicate congestion or routing issues

### `traceroute` — Trace Network Path

```bash
# Install
apt install traceroute

traceroute google.com       # Show each hop to the destination
traceroute -n google.com    # Skip DNS resolution (faster)
```

Each line is one router (hop) along the path. `* * *` means a hop didn't respond — not always a problem.

### `nslookup` / `dig` — DNS Resolution

```bash
nslookup example.com        # Basic DNS lookup
nslookup example.com 8.8.8.8  # Query specific DNS server

# dig is more powerful (install with: apt install dnsutils)
dig example.com             # Full DNS query output
dig example.com A           # Get A record (IPv4)
dig example.com MX          # Get mail records
dig +short example.com      # Just the IP address
dig @8.8.8.8 example.com    # Query against Google DNS
```

### `curl` — Test HTTP Endpoints

```bash
curl -I http://example.com          # HTTP headers only (HEAD request)
curl -o /dev/null -w "%{http_code}" http://example.com   # Just the status code
curl -v http://example.com          # Verbose — full request and response
```

---

## 4. Log Monitoring

Logs are the primary source of truth for diagnosing system and application issues. Linux stores logs in two places:

| Location | Used By |
|----------|---------|
| `/var/log/` | Traditional text log files |
| `journald` | systemd's binary journal (queried with `journalctl`) |

### Key Log Files

| File | Contents |
|------|---------|
| `/var/log/syslog` | General system activity (Debian/Ubuntu) |
| `/var/log/messages` | General system activity (RHEL/Fedora) |
| `/var/log/auth.log` | Authentication events — logins, sudo, SSH (Debian/Ubuntu) |
| `/var/log/secure` | Authentication events (RHEL/Fedora) |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/dmesg` | Hardware and boot messages |
| `/var/log/nginx/access.log` | Nginx HTTP access log |
| `/var/log/nginx/error.log` | Nginx errors |
| `/var/log/apt/history.log` | Package install/remove history |

### `tail` — Live Log Monitoring

```bash
# Follow a log file in real time
tail -f /var/log/syslog
tail -f /var/log/nginx/access.log

# Show last 50 lines, then follow
tail -n 50 -f /var/log/syslog

# Watch multiple files simultaneously
tail -f /var/log/syslog /var/log/auth.log
```

### `journalctl` — systemd Journal

```bash
journalctl -f                        # Follow live journal output
journalctl -n 50                     # Show last 50 entries
journalctl -u nginx                  # Logs for a specific service
journalctl -u nginx -f               # Follow logs for nginx
journalctl --since "1 hour ago"      # Logs from last hour
journalctl --since "2024-01-15 10:00" --until "2024-01-15 11:00"
journalctl -p err                    # Only error-level and above
journalctl -p err -u nginx           # Errors from nginx only
journalctl -b                        # Logs since last boot
journalctl -b -1                     # Logs from previous boot
journalctl --disk-usage              # How much space journal is using
```

Priority levels for `-p`:

| Level | Name | Meaning |
|-------|------|---------|
| 0 | `emerg` | System is unusable |
| 1 | `alert` | Immediate action required |
| 2 | `crit` | Critical conditions |
| 3 | `err` | Error conditions |
| 4 | `warning` | Warning conditions |
| 5 | `notice` | Normal but significant |
| 6 | `info` | Informational |
| 7 | `debug` | Debug messages |

### `dmesg` — Kernel Ring Buffer

```bash
dmesg                       # All kernel messages since boot
dmesg | tail                # Most recent kernel messages
dmesg -T                    # With human-readable timestamps
dmesg -T | grep -i error    # Filter for errors
dmesg -T | grep -i usb      # USB device events
dmesg -w                    # Follow in real time (like tail -f)
```

Common uses for `dmesg`:
- Diagnosing hardware issues (disk errors, memory errors, USB problems)
- Checking kernel module loading
- Seeing what happened at boot

---

## 5. Composite Monitoring Commands

```bash
# One-liner system health snapshot
echo "=== CPU ===" && uptime && echo "=== Memory ===" && free -h && echo "=== Disk ===" && df -h && echo "=== Top Processes ===" && ps aux --sort=-%cpu | head -6

# Find what's eating your disk
du -sh /* 2>/dev/null | sort -rh | head -10

# Find what's listening on all ports
ss -tulnp

# Check if a service is consuming too much CPU
ps aux --sort=-%cpu | head -10

# Watch memory usage over time
watch -n 2 free -h

# Watch disk I/O in real time
iostat -x 1
```

### `watch` — Repeat Any Command Periodically

```bash
watch -n 2 df -h            # Refresh df every 2 seconds
watch -n 1 ss -tulnp        # Watch open ports every second
watch -n 5 'ps aux --sort=-%mem | head -10'
```

---

## 6. Quick Diagnostics Workflow

When a system is slow or unresponsive, check in this order:

```
1. uptime          → Is load average too high?
2. free -h         → Is the system out of memory / swapping?
3. top             → Which process is consuming CPU or RAM?
4. df -h           → Is any filesystem full?
5. iostat -x 1     → Is the disk I/O saturated (%util near 100)?
6. ss -tulnp       → Are expected ports listening?
7. journalctl -p err -n 50  → Any recent errors in logs?
8. dmesg -T | tail → Any kernel-level hardware errors?
```

---

## Summary

| Task | Command |
|------|---------|
| Real-time CPU/mem | `top` / `htop` |
| Memory summary | `free -h` |
| Performance stats | `vmstat 1 5` |
| Disk space | `df -h` |
| Directory size | `du -sh /path` |
| Disk I/O | `iostat -x 1` |
| Network interfaces | `ip a` |
| Listening ports | `ss -tulnp` |
| Test connectivity | `ping google.com` |
| Trace route | `traceroute google.com` |
| DNS lookup | `dig example.com` |
| Follow logs | `tail -f /var/log/syslog` |
| Service logs | `journalctl -u nginx -f` |
| Error logs only | `journalctl -p err` |
| Kernel messages | `dmesg -T \| tail` |
| Repeat a command | `watch -n 2 df -h` |

---

**← Back to [Chapter 7: Process Management](../../chapter-7-process-management/process-management/README.md)**  
**Next → Chapter 9: Networking**
