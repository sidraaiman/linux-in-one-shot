# Process Management in Linux

## Introduction

A **process** is an instance of a running program. Every command you run, every service that starts, every background task — each becomes a process with its own isolated memory space, a unique **Process ID (PID)**, and a parent process that spawned it.

Linux gives you precise control over processes: you can inspect them, prioritize them, pause them, move them between foreground and background, and terminate them cleanly or forcefully.

---

## Process Lifecycle

```
fork()                exec()               exit()
Parent ──────────► Child created ──────► Program loads ──────► Process ends
                   (copy of parent)       (replaces child       (resources freed,
                                           with new program)     parent notified)
```

Every process on the system descends from **PID 1** (`systemd` on modern Linux). The full tree is:

```bash
pstree        # Show process tree
pstree -p     # Show with PIDs
```

---

## Process States

A process is always in one of these states:

| State | Symbol | Meaning |
|-------|--------|---------|
| Running | `R` | Actively using the CPU |
| Sleeping (interruptible) | `S` | Waiting for an event (I/O, timer) |
| Sleeping (uninterruptible) | `D` | Waiting for hardware I/O — cannot be killed |
| Stopped | `T` | Paused by a signal (Ctrl+Z) |
| Zombie | `Z` | Finished but parent hasn't read its exit code yet |

---

## 1. Viewing Processes

### `ps` — Process Snapshot

`ps` takes a snapshot of currently running processes at the moment you run it.

```bash
# Show all processes with full details (most common usage)
ps aux

# Show processes for a specific user
ps -u username

# Show a process by name
ps -C nginx

# Show process tree with hierarchy
ps auxf

# Show specific columns
ps -eo pid,ppid,user,stat,pcpu,pmem,comm
```

Understanding `ps aux` columns:

```
USER       PID  %CPU  %MEM    VSZ   RSS  TTY  STAT  START   TIME  COMMAND
alice     1234   0.5   1.2  54321  4096  pts/0  S   10:00  0:01  nginx
```

| Column | Meaning |
|--------|---------|
| `USER` | Owner of the process |
| `PID` | Process ID |
| `%CPU` | CPU usage percentage |
| `%MEM` | RAM usage percentage |
| `VSZ` | Virtual memory size (KB) |
| `RSS` | Resident Set Size — actual RAM used (KB) |
| `TTY` | Terminal attached (`?` = no terminal, i.e., daemon) |
| `STAT` | Process state (R, S, D, T, Z) |
| `START` | When process started |
| `TIME` | Cumulative CPU time consumed |
| `COMMAND` | Command that launched the process |

### `pgrep` — Find PID by Name

```bash
pgrep nginx              # Return PID(s) of nginx
pgrep -u alice           # PIDs of all processes owned by alice
pgrep -l nginx           # Show PID and process name
pgrep -a nginx           # Show PID and full command line
pgrep -c nginx           # Count how many instances are running
```

### `pidof` — Find PID of a Program

```bash
pidof nginx              # Returns PID(s) of nginx binary
pidof sshd
```

> `pgrep` searches by pattern; `pidof` searches for exact binary name. `pgrep nginx` matches `nginx`, `nginx: worker`, etc. `pidof nginx` matches only the exact binary `nginx`.

---

## 2. Real-Time Monitoring

### `top` — Interactive Process Viewer

```bash
top
```

`top` refreshes every 3 seconds and shows the most CPU-intensive processes at the top.

**Header section:**
```
top - 10:30:00 up 5 days, 3:22,  2 users,  load average: 0.15, 0.10, 0.08
Tasks: 120 total,   1 running, 119 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  0.5 sy,  0.0 ni, 97.0 id,  0.2 wa,  0.0 hi,  0.0 si
MiB Mem:   7982.3 total,   3421.1 free,   2105.2 used,   2456.0 buff/cache
MiB Swap:  2048.0 total,   2048.0 free,      0.0 used.   5500.0 avail Mem
```

| Header Field | Meaning |
|-------------|---------|
| `load average` | System load over 1, 5, 15 minutes. Above CPU count = overloaded |
| `us` | User-space CPU time |
| `sy` | Kernel/system CPU time |
| `id` | Idle percentage |
| `wa` | Time waiting for I/O (high = disk bottleneck) |
| `buff/cache` | Memory used as disk cache (can be reclaimed) |

**Interactive keys inside `top`:**

| Key | Action |
|-----|--------|
| `k` | Kill a process (prompts for PID) |
| `r` | Renice a process (change priority) |
| `u` | Filter by username |
| `M` | Sort by memory usage |
| `P` | Sort by CPU usage (default) |
| `T` | Sort by cumulative time |
| `1` | Show per-CPU stats |
| `f` | Add/remove columns |
| `d` | Change refresh interval |
| `q` | Quit |

### `htop` — Enhanced Interactive Viewer

```bash
# Install
apt install htop       # Debian/Ubuntu
dnf install htop       # Fedora/RHEL

htop
```

`htop` improvements over `top`:
- Visual CPU/memory bars
- Mouse support — click to select, scroll to navigate
- Color-coded process states
- Tree view with `F5`
- Easier kill with `F9` (shows signal menu)
- Filter processes with `F4`
- No need to remember keyboard shortcuts — function key menu at bottom

---

## 3. Killing and Signalling Processes

Linux communicates with processes using **signals**. Killing is just one type of signal.

### Common Signals

| Signal | Number | Name | Meaning |
|--------|--------|------|---------|
| `SIGTERM` | 15 | Terminate | Politely ask process to stop (can be caught/ignored) |
| `SIGKILL` | 9 | Kill | Force-terminate immediately — cannot be caught or ignored |
| `SIGHUP` | 1 | Hangup | Reload config without restarting (used by daemons) |
| `SIGSTOP` | 19 | Stop | Pause process — cannot be caught or ignored |
| `SIGCONT` | 18 | Continue | Resume a stopped process |
| `SIGINT` | 2 | Interrupt | Same as pressing Ctrl+C |
| `SIGQUIT` | 3 | Quit | Terminate with core dump |

### `kill` — Send Signal by PID

```bash
# Default: sends SIGTERM (graceful shutdown)
kill PID

# Force kill (SIGKILL — use only when SIGTERM fails)
kill -9 PID
kill -SIGKILL PID

# Reload configuration (SIGHUP)
kill -1 PID
kill -HUP PID

# Pause a process
kill -STOP PID
kill -19 PID

# Resume a paused process
kill -CONT PID
kill -18 PID

# List all available signals
kill -l
```

> Always try `kill PID` (SIGTERM) first. Give the process a few seconds to shut down cleanly. Only escalate to `kill -9` if it doesn't respond — SIGKILL gives the process no chance to clean up (close files, flush buffers, release locks).

### `pkill` — Send Signal by Name

```bash
# Terminate all nginx processes
pkill nginx

# Force kill all instances
pkill -9 nginx

# Send SIGHUP to reload config
pkill -HUP nginx

# Kill processes by user
pkill -u alice

# Kill only exact process name match (not substrings)
pkill -x nginx
```

### `killall` — Kill by Exact Name

```bash
killall nginx            # Kill all processes named exactly "nginx"
killall -9 nginx         # Force kill
killall -u alice nginx   # Kill nginx processes owned by alice
```

> `pkill` matches by pattern; `killall` matches by exact name. Use `killall` when you want to avoid accidentally killing processes whose name contains your target as a substring.

---

## 4. Process Priority (nice / renice)

Linux uses a **nice value** (NI) to determine CPU scheduling priority:

- Range: **-20 to +19**
- **Lower value = higher priority** (more CPU time)
- **Higher value = lower priority** (yields to other processes)
- Default nice value: **0**
- Only root can set negative nice values

```
-20 ◄─── Highest Priority ────── 0 ────── Lowest Priority ───► +19
(root only)                   (default)
```

### `nice` — Start a Process with a Given Priority

```bash
# Run a command with lower priority (nicer to other processes)
nice -n 10 command
nice -n 10 tar -czf backup.tar.gz /home/

# Run with higher priority (root only)
sudo nice -n -5 command

# Default (nice value 0)
nice command
```

### `renice` — Change Priority of a Running Process

```bash
# Lower priority of a running process
renice -n 10 -p PID

# Raise priority (requires root)
sudo renice -n -5 -p PID

# Change priority of all processes owned by a user
sudo renice -n 5 -u alice

# Change priority of all processes in a group
sudo renice -n 5 -g groupname
```

View nice values in `top` or `ps`:
```bash
# The NI column in top shows the nice value
top

# Show nice value with ps
ps -eo pid,ni,comm | grep nginx
```

---

## 5. Background and Foreground Jobs

When you run a command in the terminal, it runs in the **foreground** — your shell waits for it to complete. You can move processes between foreground and background during a session.

### Running in Background

```bash
# Start a command directly in the background
command &
sleep 300 &
tar -czf archive.tar.gz /data &

# Output still appears in terminal — redirect to suppress it
command > /dev/null 2>&1 &
```

When a job starts in the background:
```
[1] 4523        # [job number] PID
```

### `jobs` — List Background Jobs

```bash
jobs            # List all background/suspended jobs in current shell
jobs -l         # Include PIDs
```

Output:
```
[1]+  Running    sleep 300 &
[2]-  Stopped    vim file.txt
```

| Status | Meaning |
|--------|---------|
| `Running` | Actively executing in background |
| `Stopped` | Suspended (paused) |
| `Done` | Completed |

### Moving Jobs Between Foreground and Background

```bash
# Suspend a foreground process (pause it)
Ctrl+Z

# Resume suspended job in background
bg %1           # Resume job 1 in background
bg              # Resume most recent job

# Bring background job to foreground
fg %1           # Bring job 1 to foreground
fg              # Bring most recent job

# Kill a specific background job
kill %1
```

### `nohup` — Run Process That Survives Shell Exit

```bash
# Process keeps running even after you close the terminal
nohup command &
nohup python3 server.py > server.log 2>&1 &
```

Output goes to `nohup.out` by default, or redirect explicitly.

### `disown` — Detach Job from Shell

```bash
# Start a job and detach it from the shell session
command &
disown %1       # Job continues even after shell exits
disown -a       # Disown all jobs
```

---

## 6. Daemon Process Management with systemctl

A **daemon** is a background process that runs without a terminal — started at boot and managed by `systemd` (PID 1 on modern Linux).

```bash
# List all active services
systemctl list-units --type=service

# List all services including inactive
systemctl list-units --type=service --all

# Check status of a service (shows logs, PID, state)
systemctl status nginx

# Start a service
systemctl start nginx

# Stop a service
systemctl stop nginx

# Restart a service (stop then start)
systemctl restart nginx

# Reload config without restarting (if supported)
systemctl reload nginx

# Enable service to start at boot
systemctl enable nginx

# Disable service from starting at boot
systemctl disable nginx

# Enable and start in one command
systemctl enable --now nginx

# Check if service is enabled at boot
systemctl is-enabled nginx

# Check if service is currently running
systemctl is-active nginx
```

### Reading `systemctl status` Output

```
● nginx.service - A high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled)
     Active: active (running) since Mon 2024-01-15 10:00:00 UTC; 5h ago
    Process: 1234 ExecStartPre=/usr/sbin/nginx -t
   Main PID: 1235 (nginx)
      Tasks: 3 (limit: 4915)
     Memory: 8.2M
        CPU: 250ms
     CGroup: /system.slice/nginx.service
             ├─1235 "nginx: master process"
             └─1236 "nginx: worker process"
```

---

## 7. Viewing Process Resource Usage

```bash
# Real-time per-process I/O usage
iotop                    # apt install iotop

# Real-time per-process network usage
nethogs                  # apt install nethogs

# Detailed process info from /proc
cat /proc/PID/status     # Process state, memory, UIDs
cat /proc/PID/cmdline    # Full command line (null-separated)
cat /proc/PID/environ    # Environment variables
ls -la /proc/PID/fd      # Open file descriptors

# Show open files by a process
lsof -p PID

# Show which process has a file open
lsof /path/to/file

# Show which process is listening on a port
lsof -i :80
ss -tlnp | grep :80
```

---

## 8. Process Inspection Cheatsheet

```bash
# Find the PID of nginx
pgrep nginx
pidof nginx

# What command started PID 1234?
ps -p 1234 -o comm,args

# Who owns PID 1234?
ps -p 1234 -o user

# What files does PID 1234 have open?
lsof -p 1234

# What's the parent of PID 1234?
ps -p 1234 -o ppid

# How much memory is PID 1234 using?
ps -p 1234 -o pid,rss,vsz,pmem,comm
```

---

## Summary

| Task | Command |
|------|---------|
| See all processes | `ps aux` |
| Real-time monitor | `top` / `htop` |
| Find PID by name | `pgrep nginx` |
| Graceful kill | `kill PID` |
| Force kill | `kill -9 PID` |
| Kill by name | `pkill nginx` |
| Reload daemon config | `kill -HUP PID` |
| Pause process | `kill -STOP PID` |
| Resume process | `kill -CONT PID` |
| Start with low priority | `nice -n 10 command` |
| Change running priority | `renice -n 10 -p PID` |
| Run in background | `command &` |
| List background jobs | `jobs` |
| Bring to foreground | `fg %1` |
| Keep running after logout | `nohup command &` |
| Start a service | `systemctl start nginx` |
| Enable at boot | `systemctl enable nginx` |
| Check service status | `systemctl status nginx` |
| Open files by process | `lsof -p PID` |

---

**← Back to [Chapter 6: File Permissions](../../chapter-6-file-permissions/file-permissions/README.md)**  
**Next → Chapter 8: Networking**
