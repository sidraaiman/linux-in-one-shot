# File Management in Linux

## Introduction

In Linux, **everything is a file** — regular documents, directories, devices, sockets, and even processes are represented as files. Mastering file and directory management is the most foundational Linux skill. Every other topic — permissions, networking, services — builds on the ability to navigate, read, and manipulate files from the command line.

---

## 1. Navigating the Filesystem

### `pwd` — Print Working Directory

Shows where you currently are in the filesystem:

```bash
pwd
# Output: /home/alice
```

### `cd` — Change Directory

```bash
cd /path/to/directory      # Absolute path (from root)
cd documents               # Relative path (from current location)
cd ~                       # Go to your home directory
cd -                       # Go back to previous directory
cd ..                      # Go up one level
cd ../..                   # Go up two levels
```

> `~` always expands to the current user's home directory (`/home/username` for regular users, `/root` for root).

### `ls` — List Files and Directories

```bash
ls                         # List files in current directory
ls /etc                    # List files in a specific directory
ls -l                      # Long format — permissions, size, owner, date
ls -a                      # Show hidden files (files starting with .)
ls -la                     # Long format + hidden files
ls -lh                     # Long format with human-readable sizes (KB, MB, GB)
ls -lt                     # Sort by modification time (newest first)
ls -lS                     # Sort by file size (largest first)
ls -R                      # Recursive — list all subdirectories too
ls -1                      # One file per line
```

Understanding `ls -l` output:
```
-rw-r--r-- 1 alice developers 4096 Jan 15 10:30 report.txt
│          │ │     │           │    │              │
│          │ │     │           │    │              └─ filename
│          │ │     │           │    └─ last modified date
│          │ │     │           └─ file size in bytes
│          │ │     └─ group owner
│          │ └─ user owner
│          └─ number of hard links
└─ file type + permissions (d=directory, -=file, l=symlink)
```

---

## 2. Creating Files and Directories

### `mkdir` — Make Directory

```bash
mkdir new_folder                        # Create a single directory
mkdir -p path/to/nested/directory       # Create nested directories (no error if exists)
mkdir dir1 dir2 dir3                    # Create multiple directories at once
mkdir -m 755 secure_dir                 # Create directory with specific permissions
```

### Creating Files

```bash
touch file.txt                          # Create empty file (or update timestamp if exists)
touch file1.txt file2.txt file3.txt     # Create multiple files at once
echo "Hello World" > file.txt           # Create file with content
> file.txt                              # Create empty file / clear existing file
```

---

## 3. Copying Files and Directories

### `cp` — Copy

```bash
# Copy a file
cp file.txt backup.txt

# Copy a file to a different directory
cp file.txt /tmp/

# Copy and preserve metadata (timestamps, permissions, ownership)
cp -p file.txt backup.txt

# Copy a directory recursively
cp -r source_dir/ destination_dir/

# Copy directory recursively and preserve all metadata
cp -rp source_dir/ destination_dir/

# Interactive — prompt before overwriting
cp -i file.txt existing_file.txt

# Verbose — show what's being copied
cp -rv source_dir/ destination_dir/

# Copy multiple files into a directory
cp file1.txt file2.txt file3.txt /destination/
```

> Always use `-r` (recursive) when copying directories. Without it, `cp` will error on directories.

---

## 4. Moving and Renaming

### `mv` — Move or Rename

`mv` does double duty: moving files to a new location and renaming them use the exact same command.

```bash
# Rename a file
mv old_name.txt new_name.txt

# Move a file to a directory
mv file.txt /tmp/

# Move and rename at the same time
mv file.txt /tmp/renamed.txt

# Move a directory
mv source_dir/ /new/location/

# Move multiple files into a directory
mv file1.txt file2.txt file3.txt /destination/

# Interactive — prompt before overwriting
mv -i file.txt existing.txt

# Verbose
mv -v file.txt /tmp/
```

> Unlike `cp`, `mv` does not need `-r` for directories — it moves the whole directory in one operation.

---

## 5. Deleting Files and Directories

### `rm` — Remove

```bash
# Delete a file
rm file.txt

# Delete multiple files
rm file1.txt file2.txt file3.txt

# Interactive — confirm before each deletion
rm -i file.txt

# Force deletion (no error if file doesn't exist, no prompts)
rm -f file.txt

# Delete a directory and all its contents recursively
rm -r folder/

# Force delete a directory (no prompts, ignore errors)
rm -rf folder/

# Verbose — show each file being deleted
rm -rv folder/
```

> `rm -rf` is irreversible. There is no Recycle Bin in Linux. Double-check your path before running it, especially as root.

### `rmdir` — Remove Empty Directory

```bash
rmdir empty_folder          # Only works if the directory is empty
rmdir -p a/b/c              # Remove nested empty directories
```

Use `rm -r` for non-empty directories.

---

## 6. Viewing File Contents

### `cat` — Concatenate and Display

```bash
cat file.txt                    # Display entire file
cat -n file.txt                 # Display with line numbers
cat file1.txt file2.txt         # Concatenate and display two files
cat file1.txt file2.txt > combined.txt  # Merge two files into one
```

### `tac` — Reverse of cat

Displays file content from last line to first:

```bash
tac file.txt
```

Useful for reading logs in reverse (most recent entry first).

### `less` — Scrollable File Viewer

`less` is the standard tool for reading large files. Unlike `cat`, it doesn't load the entire file into memory.

```bash
less file.txt
less /var/log/syslog
```

Navigation inside `less`:

| Key | Action |
|-----|--------|
| `Space` / `f` | Scroll down one page |
| `b` | Scroll up one page |
| `↑` / `↓` | Scroll one line |
| `g` | Go to beginning of file |
| `G` | Go to end of file |
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` | Next search match |
| `N` | Previous search match |
| `q` | Quit |

### `more` — Basic Pager

Older and simpler than `less` — moves forward only:

```bash
more file.txt
```

Prefer `less` in almost all situations. `more` is available on older or minimal systems where `less` may not be installed.

### `head` — View Beginning of a File

```bash
head file.txt                  # First 10 lines (default)
head -n 20 file.txt            # First 20 lines
head -n 1 file.txt             # First line only (useful for CSV headers)
head -c 100 file.txt           # First 100 bytes
```

### `tail` — View End of a File

```bash
tail file.txt                  # Last 10 lines (default)
tail -n 20 file.txt            # Last 20 lines
tail -n 1 file.txt             # Last line only
tail -c 100 file.txt           # Last 100 bytes

# Follow a file in real time (essential for log monitoring)
tail -f /var/log/syslog
tail -f /var/log/nginx/access.log

# Follow and show last 50 lines first
tail -n 50 -f /var/log/syslog
```

> `tail -f` is indispensable for watching logs as a service runs. Press `Ctrl+C` to stop.

---

## 7. Writing to Files — Redirection

### Overwrite with `>`

Writes output to a file, replacing all existing content:

```bash
echo "Hello World" > file.txt
ls -la > directory_listing.txt
cat /etc/os-release > os_info.txt
```

### Append with `>>`

Adds output to the end of a file without touching existing content:

```bash
echo "New line" >> file.txt
echo "$(date): backup complete" >> /var/log/backup.log
```

### Redirect Both stdout and stderr

```bash
command > output.txt 2>&1         # Redirect both stdout and stderr to file
command >> output.txt 2>&1        # Append both
command 2>/dev/null               # Discard errors, show normal output
command > /dev/null 2>&1          # Discard all output entirely
```

---

## 8. Text Editors

### `nano` — Beginner-Friendly Editor

Simple, modal-free editor. What you type goes directly into the file.

```bash
nano file.txt
nano /etc/hosts
```

Key shortcuts (shown at the bottom of the nano screen):

| Shortcut | Action |
|----------|--------|
| `Ctrl+O` | Save (Write Out) |
| `Ctrl+X` | Exit |
| `Ctrl+K` | Cut current line |
| `Ctrl+U` | Paste (Uncut) |
| `Ctrl+W` | Search |
| `Ctrl+G` | Help |

### `vi` / `vim` — Powerful Modal Editor

`vim` (Vi Improved) is available on virtually every Linux system and is the standard editor for production server work. It has a steeper learning curve but is far more powerful than nano.

```bash
vi file.txt
vim file.txt
```

Vim has three primary modes:

| Mode | How to Enter | Purpose |
|------|-------------|---------|
| **Normal** | `Esc` | Navigation, commands |
| **Insert** | `i`, `a`, `o` | Typing text |
| **Command** | `:` | Save, quit, search, replace |

Essential vim commands:

```
# Entering insert mode
i          Insert before cursor
a          Insert after cursor
o          Open new line below and insert
O          Open new line above and insert

# Normal mode navigation
h j k l    Move left / down / up / right
gg         Go to first line
G          Go to last line
:n         Go to line number n (e.g., :42)
w          Move forward one word
b          Move back one word

# Editing in normal mode
dd         Delete current line
yy         Copy (yank) current line
p          Paste below
u          Undo
Ctrl+r     Redo
x          Delete character under cursor
dw         Delete word

# Saving and quitting (in command mode — press : first)
:w         Save
:q         Quit
:wq        Save and quit
:q!        Quit without saving
:wq!       Force save and quit (read-only file override)

# Search and replace
/pattern        Search forward
?pattern        Search backward
n               Next match
:%s/old/new/g   Replace all occurrences in file
```

> When you SSH into a server and need to edit a config file, `vi` is almost always available even on minimal installs. Learning the basics is non-negotiable for server work.

---

## 9. Finding Files

```bash
# Find files by name
find /path -name "filename.txt"

# Case-insensitive search
find /path -iname "readme*"

# Find files modified in the last 24 hours
find /var/log -mtime -1

# Find files larger than 100MB
find / -size +100M

# Find and delete files (use carefully)
find /tmp -name "*.tmp" -delete

# Find files owned by a user
find / -user alice 2>/dev/null
```

---

## 10. Useful File Operations

### Check File Type

```bash
file file.txt          # Determines type: ASCII text, binary, symlink, etc.
file /bin/ls           # ELF 64-bit LSB shared object (binary executable)
```

### Check Disk Usage of Files and Directories

```bash
du -sh folder/              # Total size of a directory (human-readable)
du -sh *                    # Size of everything in current directory
du -sh /* 2>/dev/null       # Size of each top-level directory
```

### Count Lines, Words, Characters

```bash
wc file.txt                 # Lines, words, bytes
wc -l file.txt              # Line count only
wc -w file.txt              # Word count only
```

### Create Symbolic Links

```bash
ln -s /original/path /link/path        # Soft link (symlink)
ln /original/file /hard/link/file      # Hard link
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Where am I? | `pwd` |
| List files (detailed) | `ls -lh` |
| Create directory (nested) | `mkdir -p path/to/dir` |
| Copy directory | `cp -r src/ dest/` |
| Move/rename | `mv old new` |
| Delete directory | `rm -rf folder/` |
| View large file | `less file.txt` |
| Watch log live | `tail -f logfile` |
| Write to file | `echo "text" > file.txt` |
| Append to file | `echo "text" >> file.txt` |
| Edit (simple) | `nano file.txt` |
| Edit (powerful) | `vim file.txt` |
| Find files | `find / -name "file.txt"` |
| File size | `du -sh folder/` |
| File type | `file filename` |

---

**← Back to [Chapter 3: User Management](../../chapter-3-user-management/user-management/README.md)**  
**Next → Chapter 5: File Permissions**
