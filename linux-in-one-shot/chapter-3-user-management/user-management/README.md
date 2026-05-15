# User Management in Linux

## Introduction

Linux is a **multi-user operating system** — multiple users can log in and operate simultaneously, each with their own isolated environment, permissions, and files. Proper user management is fundamental to system security: it controls who can access what, prevents privilege escalation, and maintains accountability through audit trails.

Every action on a Linux system is associated with a user. Even background services run under dedicated service accounts (e.g., `www-data` for nginx, `mysql` for MySQL) to limit the damage if they are compromised.

---

## Key System Files

All user and group information is stored in plain text files under `/etc`. Understanding these files is essential for troubleshooting and auditing.

### `/etc/passwd` — User Account Details

Each line represents one user account:

```
username:x:UID:GID:comment:home_directory:shell
```

Example:
```
root:x:0:0:root:/root:/bin/bash
alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

| Field | Meaning |
|-------|---------|
| `username` | Login name |
| `x` | Password placeholder (`x` means password is in `/etc/shadow`) |
| `UID` | User ID — root is always 0, system users are 1–999, regular users start at 1000 |
| `GID` | Primary group ID |
| `comment` | Full name or description (GECOS field) |
| `home_directory` | User's home directory path |
| `shell` | Default login shell (`/usr/sbin/nologin` means the account cannot log in interactively) |

```bash
cat /etc/passwd
```

### `/etc/shadow` — Encrypted Passwords

Only readable by root. Stores hashed passwords and password policy settings:

```
username:$6$hash...:last_changed:min:max:warn:inactive:expire:
```

| Field | Meaning |
|-------|---------|
| `$6$hash` | Hashed password (`$6$` = SHA-512, `$1$` = MD5, `!` or `*` = account locked) |
| `last_changed` | Days since epoch when password was last changed |
| `min` | Minimum days before password can be changed |
| `max` | Maximum days before password must be changed |
| `warn` | Days before expiry to warn the user |
| `inactive` | Days after expiry before account is disabled |
| `expire` | Absolute date when account expires |

```bash
sudo cat /etc/shadow
```

### `/etc/group` — Group Information

```
groupname:x:GID:member1,member2,...
```

Example:
```
sudo:x:27:alice,bob
docker:x:999:alice
developers:x:1005:alice,charlie,dave
```

```bash
cat /etc/group
```

### `/etc/gshadow` — Secure Group Details

Stores group passwords and administrators (rarely used in practice):

```bash
sudo cat /etc/gshadow
```

---

## Creating Users

### `useradd` — Low-Level User Creation (All Distros)

`useradd` is the standard, scriptable command available on all Linux distributions. It creates a user but does **not** set a password or create a home directory by default.

```bash
# Basic user creation (no home directory)
useradd username

# Create user with a home directory
useradd -m username

# Create user with home directory and specific shell
useradd -m -s /bin/bash username

# Full example — specify everything
useradd -m -s /bin/bash -c "Alice Smith" -u 1500 -g developers alice
```

| Flag | Meaning |
|------|---------|
| `-m` | Create home directory at `/home/username` |
| `-s /bin/bash` | Set default login shell |
| `-c "comment"` | Set the GECOS/comment field (usually full name) |
| `-u 1500` | Assign a specific UID |
| `-g groupname` | Set primary group |
| `-G group1,group2` | Add to supplementary groups |
| `-d /custom/path` | Set a custom home directory path |
| `-r` | Create a system account (UID < 1000, no home directory) |
| `-e 2025-12-31` | Set account expiry date |

After creating a user, always set a password:
```bash
passwd username
```

### `adduser` — Interactive User Creation (Debian/Ubuntu)

`adduser` is a friendlier Perl wrapper around `useradd`. It prompts for a password and additional details interactively:

```bash
adduser username
```

Output:
```
Adding user 'alice' ...
Adding new group 'alice' (1001) ...
Adding new user 'alice' (1001) with group 'alice' ...
Creating home directory '/home/alice' ...
New password:
Retype new password:
Full Name []: Alice Smith
Room Number []:
...
```

Use `useradd` in scripts; use `adduser` when setting up users interactively.

---

## Managing Passwords

### Setting and Changing Passwords

```bash
# Set/change password for a user (prompts interactively)
passwd username

# Change your own password
passwd

# Set password non-interactively (useful in scripts)
echo "username:newpassword" | chpasswd
```

### Password Policy with `chage`

`chage` manages the password aging policy stored in `/etc/shadow`:

```bash
# View current password aging info
chage -l username

# Set maximum password age to 90 days
chage -M 90 username

# Set minimum days between password changes to 7
chage -m 7 username

# Warn user 14 days before expiry
chage -W 14 username

# Force password change on next login
chage -d 0 username

# Set account expiry date
chage -E 2025-12-31 username
```

### Locking and Unlocking Accounts

```bash
# Lock a user account (prepends ! to password hash in /etc/shadow)
passwd -l username

# Unlock a user account
passwd -u username

# Check if account is locked
passwd -S username
```

Output of `passwd -S`:
```
alice P 2024-01-15 0 90 14 -1
#      ^ P=Password set, L=Locked, NP=No Password
```

---

## Modifying Users

Use `usermod` to change properties of an existing user:

```bash
# Rename a user (does not rename home directory automatically)
usermod -l new_username old_username

# Change home directory and move existing files
usermod -d /new/home/path -m username

# Change default shell
usermod -s /bin/zsh username

# Add user to a supplementary group (without removing from existing groups)
usermod -aG docker username

# Change primary group
usermod -g new_primary_group username

# Lock/unlock via usermod
usermod -L username    # lock
usermod -U username    # unlock

# Set account expiry date
usermod -e 2025-12-31 username
```

> Always use `-aG` (append + groups) when adding to supplementary groups. Without `-a`, usermod **replaces** all existing group memberships.

---

## Deleting Users

```bash
# Remove user account only (home directory and files remain)
userdel username

# Remove user account AND home directory
userdel -r username

# Interactive removal on Debian/Ubuntu (also removes mail spool)
deluser username
deluser --remove-home username
```

Before deleting, check if the user owns files elsewhere on the system:
```bash
find / -user username 2>/dev/null
```

---

## Working with Groups

### Creating Groups

```bash
groupadd groupname

# Create group with a specific GID
groupadd -g 1500 developers
```

### Adding and Removing Users from Groups

```bash
# Add user to a group
usermod -aG groupname username

# Remove user from a group
gpasswd -d username groupname

# Add user to multiple groups at once
usermod -aG docker,sudo,developers alice
```

### Viewing Group Memberships

```bash
# See all groups a user belongs to
groups username

# Detailed view with GIDs
id username

# See all members of a specific group
getent group groupname
```

Example output of `id alice`:
```
uid=1001(alice) gid=1001(alice) groups=1001(alice),27(sudo),999(docker)
```

### Changing Primary Group

The **primary group** is used when a user creates files — the file's group ownership is set to the primary group.

```bash
# Change primary group
usermod -g developers alice

# Switch active group in current session without logging out
newgrp developers
```

### Deleting Groups

```bash
groupdel groupname
```

> You cannot delete a group that is still a user's primary group.

---

## Sudo Access and Privilege Escalation

### How `sudo` Works

`sudo` lets a permitted user run a command as root (or another user) without logging in as root. Every `sudo` usage is logged to `/var/log/auth.log` (Debian/Ubuntu) or `/var/log/secure` (RHEL/Fedora).

### Adding a User to the Sudo Group

```bash
# Debian/Ubuntu — sudo group
usermod -aG sudo username

# RHEL/Fedora/CentOS — wheel group
usermod -aG wheel username
```

After adding, the user must log out and back in for group changes to take effect, or use:
```bash
su - username
```

### Verifying Sudo Access

```bash
sudo -l -U username
```

Shows exactly what commands the user can run with sudo.

### The `/etc/sudoers` File

Never edit `/etc/sudoers` directly — always use `visudo`, which validates syntax before saving (a syntax error in sudoers can lock you out of sudo entirely):

```bash
visudo
```

### Sudoers Syntax

```
# Format:
# user  host=(run_as_user:run_as_group)  commands

# Allow alice to run everything as root with password
alice ALL=(ALL:ALL) ALL

# Allow alice to run everything without password
alice ALL=(ALL) NOPASSWD: ALL

# Allow alice to only restart nginx without password
alice ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx

# Allow the entire sudo group to run everything
%sudo ALL=(ALL:ALL) ALL

# Allow developers group to run specific commands
%developers ALL=(ALL) NOPASSWD: /usr/bin/docker, /bin/systemctl
```

### Drop-in Sudoers Files

Instead of editing the main sudoers file, add per-user or per-service rules in `/etc/sudoers.d/`:

```bash
echo "alice ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx" > /etc/sudoers.d/alice
chmod 0440 /etc/sudoers.d/alice
```

---

## Switching Users

```bash
# Switch to another user (requires their password)
su username

# Switch to another user and load their full environment
su - username

# Switch to root
su -

# Run a single command as another user
su -c "command" username
```

---

## Viewing and Auditing Users

```bash
# List all users on the system
cut -d: -f1 /etc/passwd

# List only human users (UID >= 1000)
awk -F: '$3 >= 1000 {print $1}' /etc/passwd

# See who is currently logged in
who
w

# See last login history
last username

# See failed login attempts
lastb

# Check sudo usage in logs
grep sudo /var/log/auth.log
```

---

## Summary: User Management Commands

| Task | Command |
|------|---------|
| Create user with home dir | `useradd -m -s /bin/bash username` |
| Set password | `passwd username` |
| View password policy | `chage -l username` |
| Lock account | `passwd -l username` |
| Add to group | `usermod -aG groupname username` |
| View user's groups | `id username` |
| Grant sudo (Debian) | `usermod -aG sudo username` |
| Grant sudo (RHEL) | `usermod -aG wheel username` |
| Edit sudoers safely | `visudo` |
| Delete user + home | `userdel -r username` |
| Find files owned by user | `find / -user username` |

---

**← Back to [Chapter 2: Folder Structure](../../chapter-2-folder-structure/folder_structure/README.md)**  
**Next → Chapter 4: File Permissions**
