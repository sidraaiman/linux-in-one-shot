# File Permissions Management in Linux

## Introduction

Linux file permissions are a core security mechanism. Every file and directory on the system has a set of permissions that control exactly who can read it, write to it, or execute it. Getting permissions wrong is one of the most common causes of both security vulnerabilities (too permissive) and broken services (too restrictive).

---

## Permission Model Overview

Every file and directory has three permission scopes:

| Scope | Symbol | Meaning |
|-------|--------|---------|
| **Owner (User)** | `u` | The user who created or owns the file |
| **Group** | `g` | Users belonging to the file's assigned group |
| **Others** | `o` | Every other user on the system |

Within each scope, three permission types exist:

| Permission | Symbol | Numeric Value | On Files | On Directories |
|-----------|--------|--------------|---------|----------------|
| **Read** | `r` | `4` | View file contents | List directory contents (`ls`) |
| **Write** | `w` | `2` | Modify file contents | Create, delete, rename files inside |
| **Execute** | `x` | `1` | Run as program/script | Enter the directory (`cd`) |

---

## Reading Permission Output

```bash
ls -l filename
```

Example output:
```
-rwxr--r-- 1 alice developers 1234 Mar 28 10:00 myfile.sh
```

Breaking it down:

```
- rwx r-- r-- 1 alice developers 1234 Mar 28 10:00 myfile.sh
│ │   │   │   │ │     │          │    │              │
│ │   │   │   │ │     │          │    │              └─ filename
│ │   │   │   │ │     │          │    └─ last modified
│ │   │   │   │ │     │          └─ file size (bytes)
│ │   │   │   │ │     └─ group owner
│ │   │   │   │ └─ user (owner)
│ │   │   │   └─ hard link count
│ │   │   └─ others permissions (r--)
│ │   └─ group permissions (r--)
│ └─ owner permissions (rwx)
└─ file type: - = regular file, d = directory, l = symlink, c = char device, b = block device
```

### File Type Characters

| Character | Type |
|-----------|------|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device (e.g., `/dev/tty`) |
| `b` | Block device (e.g., `/dev/sda`) |
| `s` | Socket |
| `p` | Named pipe (FIFO) |

---

## Checking Permissions

```bash
ls -l filename               # Permissions of a specific file
ls -la                       # All files including hidden
ls -ld directory/            # Permissions of the directory itself (not its contents)
stat filename                # Detailed metadata including octal permissions
```

`stat` output:
```
File: myfile.sh
Size: 1234      Blocks: 8    IO Block: 4096  regular file
Access: (0755/-rwxr-xr-x)  Uid: (1001/alice)  Gid: (1005/developers)
```

---

## Changing Permissions with `chmod`

### Symbolic Mode

Uses letter symbols to add, remove, or set permissions.

**Syntax:** `chmod [who][operator][permissions] filename`

| Who | Meaning |
|-----|---------|
| `u` | User (owner) |
| `g` | Group |
| `o` | Others |
| `a` | All (user + group + others) |

| Operator | Meaning |
|----------|---------|
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set exactly (replaces existing) |

```bash
# Add execute for owner
chmod u+x filename

# Remove write from group
chmod g-w filename

# Set others to read-only (removes write and execute)
chmod o=r filename

# Add read+write for everyone
chmod a+rw filename

# Remove execute from group and others
chmod go-x filename

# Set precise permissions: owner=rwx, group=rx, others=nothing
chmod u=rwx,g=rx,o= filename

# Apply recursively to a directory
chmod -R 755 directory/
```

### Numeric (Octal) Mode

Each permission has a numeric value. Add the values together for each scope:

```
r = 4
w = 2
x = 1
- = 0
```

Calculate each group:
```
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
-wx = 0+2+1 = 3
-w- = 0+2+0 = 2
--x = 0+0+1 = 1
--- = 0+0+0 = 0
```

**Syntax:** `chmod [owner][group][others] filename`

```bash
chmod 755 filename     # rwxr-xr-x  — owner full, group/others read+execute
chmod 644 filename     # rw-r--r--  — owner read+write, group/others read only
chmod 700 filename     # rwx------  — owner full, no access for anyone else
chmod 600 filename     # rw-------  — owner read+write only (private files)
chmod 777 filename     # rwxrwxrwx  — full access for everyone (avoid this)
chmod 400 filename     # r--------  — read-only for owner (e.g., SSH private keys)
chmod 664 filename     # rw-rw-r--  — owner and group can write, others read only
chmod 750 filename     # rwxr-x---  — owner full, group read+execute, others none
```

### Common Permission Scenarios

| Scenario | Permission | Octal |
|---------|-----------|-------|
| Web server files (public HTML) | `-rw-r--r--` | `644` |
| Web server directories | `drwxr-xr-x` | `755` |
| Shell scripts | `-rwxr-xr-x` | `755` |
| Private config files | `-rw-------` | `600` |
| SSH private key | `-r--------` | `400` |
| Shared group directory | `drwxrwxr-x` | `775` |
| Sensitive data (owner only) | `-rw-------` | `600` |

---

## Changing Ownership with `chown`

```bash
# Change file owner
chown newuser filename

# Change owner and group
chown newuser:newgroup filename

# Change only the group (note the colon prefix)
chown :newgroup filename

# Recursive — change owner+group on directory and all contents
chown -R newuser:newgroup directory/

# Copy ownership from another file
chown --reference=referencefile targetfile
```

Real-world examples:
```bash
# Fix web server file ownership
chown -R www-data:www-data /var/www/html/

# Transfer file ownership when a user is deleted
chown -R alice:alice /home/alice/

# Set ownership for an application
chown -R appuser:appgroup /opt/myapp/
```

---

## Changing Group Ownership with `chgrp`

```bash
# Change group of a file
chgrp newgroup filename

# Change group recursively on a directory
chgrp -R newgroup directory/

# Verify the change
ls -l filename
```

> `chown :newgroup` and `chgrp newgroup` do the same thing. `chown` is more commonly used because it can do both owner and group in one command.

---

## Special Permissions

Special permissions extend the standard `rwx` model for advanced use cases.

### SetUID — `s` on Owner Execute Bit

When set on an **executable file**, it runs with the **file owner's privileges** instead of the user who launched it.

```bash
# Set SetUID
chmod u+s filename
chmod 4755 filename      # 4 = SetUID, 755 = standard permissions

# View SetUID files (lowercase s on owner execute)
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root ... /usr/bin/passwd
```

Real-world example: `/usr/bin/passwd` is owned by root with SetUID. When you run `passwd`, it temporarily runs as root so it can write your new password hash to `/etc/shadow` — but only for that specific operation.

```bash
# Find all SetUID files on the system (security audit)
find / -perm -4000 -type f 2>/dev/null
```

> SetUID on shell scripts is ignored by the kernel for security reasons. It only applies to binary executables.

### SetGID — `s` on Group Execute Bit

**On files:** The file runs with the **group's privileges**.  
**On directories:** Files created inside **inherit the directory's group** instead of the creator's primary group.

```bash
# Set SetGID on a file
chmod g+s filename
chmod 2755 filename      # 2 = SetGID

# Set SetGID on a directory (shared team directories)
chmod g+s directory/
chmod 2775 directory/
```

Real-world example:
```bash
# Create a shared project directory where all files belong to the team group
mkdir /srv/project
chown root:developers /srv/project
chmod 2775 /srv/project
# Now any file created inside inherits the "developers" group automatically
```

```bash
# Find all SetGID files
find / -perm -2000 -type f 2>/dev/null
```

### Sticky Bit — `t` on Others Execute Bit

Applied to **directories**. Even if users have write permission on the directory, they can only delete or rename **their own files** — not files owned by other users.

```bash
# Set sticky bit
chmod +t directory/
chmod 1777 directory/    # 1 = sticky bit

# View (lowercase t on others execute)
ls -ld /tmp
# drwxrwxrwt 1 root root ... /tmp
```

`/tmp` is the classic example: world-writable (anyone can create files) but sticky (nobody can delete someone else's files).

```bash
# Find sticky bit directories
find / -perm -1000 -type d 2>/dev/null
```

### Special Permission Numeric Prefixes

| Prefix | Special Bit |
|--------|------------|
| `4xxx` | SetUID |
| `2xxx` | SetGID |
| `1xxx` | Sticky Bit |
| `6xxx` | SetUID + SetGID |
| `7xxx` | SetUID + SetGID + Sticky |

---

## Default Permissions with `umask`

`umask` (user file-creation mask) defines what permissions are **removed** from newly created files and directories.

```bash
# Check current umask
umask
# Output: 0022

# Check with symbolic representation
umask -S
# Output: u=rwx,g=rx,o=rx
```

### How umask Works

The umask is subtracted from the maximum default:
- Maximum for **files**: `666` (rw-rw-rw-) — execute never set by default
- Maximum for **directories**: `777` (rwxrwxrwx)

```
umask 022:

Files:       666 - 022 = 644  (rw-r--r--)
Directories: 777 - 022 = 755  (rwxr-xr-x)
```

```
umask 027:

Files:       666 - 027 = 640  (rw-r-----)
Directories: 777 - 027 = 750  (rwxr-x---)
```

### Common umask Values

| umask | Files | Directories | Use Case |
|-------|-------|-------------|---------|
| `022` | `644` | `755` | Standard (default on most systems) |
| `027` | `640` | `750` | More restrictive — others get no access |
| `077` | `600` | `700` | Maximum privacy — owner only |
| `002` | `664` | `775` | Team/group collaboration |

### Setting umask

```bash
# Set for current session only
umask 027

# Set permanently for a user (add to ~/.bashrc or ~/.profile)
echo "umask 027" >> ~/.bashrc

# Set system-wide default (affects all users)
echo "umask 022" >> /etc/profile
```

---

## Access Control Lists (ACLs) — Fine-Grained Permissions

Standard permissions only allow one owner and one group. ACLs let you assign permissions to **specific users or groups** beyond the owner/group model.

```bash
# Install ACL tools (if not present)
apt install acl

# View ACL on a file
getfacl filename

# Grant read+write to a specific user
setfacl -m u:bob:rw filename

# Grant read to a specific group
setfacl -m g:contractors:r filename

# Remove a specific ACL entry
setfacl -x u:bob filename

# Remove all ACL entries
setfacl -b filename
```

Files with ACLs show a `+` at the end of the permission string:
```
-rw-rw-r--+ 1 alice developers 1234 Jan 15 10:00 report.txt
```

---

## Practical Security Checklist

```bash
# Find world-writable files (security risk)
find / -perm -o+w -type f 2>/dev/null

# Find world-writable directories
find / -perm -o+w -type d 2>/dev/null

# Find files with no owner (orphaned after user deletion)
find / -nouser 2>/dev/null

# Find SetUID binaries (privilege escalation vectors)
find / -perm -4000 -type f 2>/dev/null

# Fix common web server permission issues
chown -R www-data:www-data /var/www/html
find /var/www/html -type f -exec chmod 644 {} \;
find /var/www/html -type d -exec chmod 755 {} \;
```

---

## Summary

| Command | Purpose |
|---------|---------|
| `ls -l` | View permissions |
| `chmod 755 file` | Set permissions (octal) |
| `chmod u+x file` | Add execute for owner (symbolic) |
| `chown user:group file` | Change owner and group |
| `chown -R user:group dir/` | Recursive ownership change |
| `chgrp group file` | Change group only |
| `chmod u+s file` | Set SetUID |
| `chmod g+s dir/` | Set SetGID on directory |
| `chmod +t dir/` | Set sticky bit |
| `umask` | View default permission mask |
| `umask 027` | Set stricter default |
| `getfacl file` | View ACL entries |
| `setfacl -m u:bob:rw file` | Grant specific user access via ACL |

---

**← Back to [Chapter 5: VI Editor](../../chapter-5-vi-editor/vi-shortcuts/README.md)**  
**Next → Chapter 7: Process Management**
