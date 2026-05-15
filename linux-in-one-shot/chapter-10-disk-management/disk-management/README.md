# Disk and Storage Management in Linux

## Introduction

Storage management is one of the most consequential tasks in Linux administration. Getting it wrong — wrong partition size, wrong format, wrong mount flags — can mean data loss or services that cannot write logs, databases, or temporary files. This chapter covers everything from inspecting raw disks to partitioning, formatting, mounting, LVM, and swap management.

---

## 1. Viewing Disk Information

### `lsblk` — Block Device Tree

The first command to run when you attach a new disk or want to understand the current storage layout:

```bash
lsblk                   # Show all block devices
lsblk -f                # Include filesystem type, UUID, and mount point
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT,UUID   # Custom columns
```

Sample output:
```
NAME    MAJ:MIN  RM   SIZE  RO  TYPE  MOUNTPOINT
sda       8:0     0   100G   0  disk
├─sda1    8:1     0    96G   0  part  /
└─sda2    8:2     0     4G   0  part  [SWAP]
sdb       8:16    0    20G   0  disk
nvme0n1 259:0     0   500G   0  disk
└─nvme0n1p1 259:1 0   500G  0  part  /data
```

| Column | Meaning |
|--------|---------|
| `NAME` | Device name (`sda` = SATA/SCSI, `nvme0n1` = NVMe SSD, `vda` = virtual disk) |
| `RM` | Removable (1 = USB/CD, 0 = internal) |
| `SIZE` | Total device size |
| `TYPE` | `disk` = raw device, `part` = partition, `lvm` = LVM volume |
| `MOUNTPOINT` | Where it is currently mounted (`[SWAP]` = swap space) |

### `fdisk -l` — Partition Table Details

```bash
sudo fdisk -l               # List all disks and their partition tables
sudo fdisk -l /dev/sdb      # List partitions on a specific disk
```

Output includes partition start/end sectors, size, and partition type (Linux, swap, EFI, etc.).

### `blkid` — Block Device UUIDs and Filesystem Types

```bash
sudo blkid                  # Show all devices with UUID, type, label
sudo blkid /dev/sda1        # Show info for specific partition
```

Sample output:
```
/dev/sda1: UUID="a1b2c3d4-e5f6-7890-abcd-ef1234567890" TYPE="ext4" PARTUUID="..."
/dev/sda2: UUID="deadbeef-..." TYPE="swap"
```

UUIDs are used in `/etc/fstab` for reliable persistent mounts — device names like `/dev/sda1` can change between reboots if disks are added/removed, but UUIDs never change.

### `df` — Filesystem Disk Space

```bash
df -h                       # All mounted filesystems, human-readable
df -hT                      # Include filesystem type
df -h /var                  # Check specific mount point
df -i                       # Inode usage (important for many-small-files workloads)
```

> Watch `Use%` — above 90% is a warning. Above 95% can cause services to crash (databases, web servers, cron jobs all need disk space to write).

### `du` — Directory Disk Usage

```bash
du -sh /var/log             # Total size of a directory
du -sh /var/log/*           # Size of each item inside
du -h --max-depth=1 /var    # One level deep
du -sh * | sort -rh | head  # Find 10 largest items in current directory
```

---

## 2. Understanding Device Naming

| Device Name | Type |
|-------------|------|
| `/dev/sda`, `/dev/sdb` | SATA or SCSI disk |
| `/dev/nvme0n1`, `/dev/nvme1n1` | NVMe SSD |
| `/dev/vda`, `/dev/vdb` | Virtual disk (KVM, cloud VMs) |
| `/dev/sda1`, `/dev/sda2` | Partitions on `sda` |
| `/dev/nvme0n1p1` | First partition on NVMe disk |
| `/dev/mapper/vg-lv` | LVM logical volume |

Partitions are always numbered — `/dev/sda1` is the first partition of `/dev/sda`. The parent disk (`/dev/sda`) is never directly formatted; partitions on it are.

---

## 3. Partition Management

### When to Use `fdisk` vs `parted`

| Tool | Use When |
|------|----------|
| `fdisk` | MBR partition tables, disks under 2TB, interactive use |
| `parted` | GPT partition tables, disks over 2TB, scripting |
| `gdisk` | GPT partition tables (fdisk-style interface) |

### `fdisk` — Create and Manage Partitions (MBR/GPT)

```bash
sudo fdisk /dev/sdb         # Open interactive partition editor for sdb
```

Inside `fdisk`, key commands:

| Command | Action |
|---------|--------|
| `m` | Show help menu |
| `p` | Print current partition table |
| `n` | New partition |
| `d` | Delete a partition |
| `t` | Change partition type |
| `l` | List partition types |
| `w` | Write changes and exit (this commits changes to disk) |
| `q` | Quit without saving |

**Complete example — partition a new disk:**

```bash
# 1. Open fdisk for the new disk
sudo fdisk /dev/sdb

# Inside fdisk:
# n          → new partition
# p          → primary partition
# 1          → partition number 1
# Enter      → accept default start sector
# +10G       → size of 10GB (or Enter for entire disk)
# w          → write and exit

# 2. Verify the new partition appeared
lsblk
sudo fdisk -l /dev/sdb
```

### `parted` — GPT Partitioning (Disks > 2TB)

```bash
sudo parted /dev/sdb                    # Interactive mode
sudo parted /dev/sdb print              # Show partition table

# Non-interactive (scriptable)
sudo parted /dev/sdb mklabel gpt        # Initialize GPT partition table
sudo parted /dev/sdb mkpart primary ext4 0% 100%   # Use entire disk
sudo parted /dev/sdb mkpart primary ext4 0% 10GB   # 10GB partition
```

---

## 4. Formatting Partitions

After creating a partition, format it with a filesystem before use.

### Common Filesystem Types

| Filesystem | Command | Best For |
|------------|---------|---------|
| ext4 | `mkfs.ext4` | General purpose — default on most Linux systems |
| xfs | `mkfs.xfs` | Large files, high performance, used by RHEL by default |
| btrfs | `mkfs.btrfs` | Snapshots, compression, modern features |
| fat32 | `mkfs.fat -F 32` | USB drives, cross-platform compatibility |
| ntfs | `mkfs.ntfs` | Windows compatibility |

```bash
# Format as ext4
sudo mkfs.ext4 /dev/sdb1

# Format as ext4 with a label
sudo mkfs.ext4 -L "mydata" /dev/sdb1

# Format as XFS
sudo mkfs.xfs /dev/sdb1

# Format as XFS with a label
sudo mkfs.xfs -L "mydata" /dev/sdb1

# Format as btrfs
sudo mkfs.btrfs /dev/sdb1

# Format USB drive as FAT32
sudo mkfs.fat -F 32 /dev/sdb1
```

> Formatting **destroys all data** on the partition. Double-check the device name with `lsblk` before running `mkfs`.

---

## 5. Mounting and Unmounting

### `mount` — Attach a Filesystem

```bash
# Create a mount point (directory where the filesystem will appear)
sudo mkdir /mnt/mydisk

# Mount a partition
sudo mount /dev/sdb1 /mnt/mydisk

# Mount with specific filesystem type
sudo mount -t ext4 /dev/sdb1 /mnt/mydisk

# Mount as read-only
sudo mount -o ro /dev/sdb1 /mnt/mydisk

# Remount as read-write (change options without unmounting)
sudo mount -o remount,rw /mnt/mydisk

# Mount by UUID (more reliable than device name)
sudo mount UUID="a1b2c3d4-..." /mnt/mydisk

# Show all currently mounted filesystems
mount | column -t
mount | grep /dev/sd
```

### `umount` — Detach a Filesystem

```bash
sudo umount /mnt/mydisk             # Unmount by mount point
sudo umount /dev/sdb1               # Unmount by device

# Force unmount (use when regular umount fails)
sudo umount -f /mnt/mydisk

# Lazy unmount (detach immediately, cleanup when unused)
sudo umount -l /mnt/mydisk
```

> If `umount` fails with "device is busy," find what's using it:
```bash
lsof /mnt/mydisk         # Show processes with open files on the mount
fuser -m /mnt/mydisk     # Show PIDs using the mount
```

---

## 6. Persistent Mounts with `/etc/fstab`

Mounts made with `mount` do not survive a reboot. To make a mount permanent, add an entry to `/etc/fstab`.

```bash
sudo nano /etc/fstab
```

`/etc/fstab` format:
```
<device>   <mountpoint>   <fstype>   <options>   <dump>   <pass>
```

Example entries:
```
# Mount by UUID (recommended)
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890  /data  ext4  defaults  0  2

# Mount by device name
/dev/sdb1  /mnt/backup  ext4  defaults,noatime  0  2

# NFS network mount
192.168.1.100:/exports/share  /mnt/nfs  nfs  defaults  0  0

# Swap
UUID=deadbeef-...  none  swap  sw  0  0
```

| Field | Meaning |
|-------|---------|
| `defaults` | rw, suid, dev, exec, auto, nouser, async |
| `noatime` | Don't update access timestamps — improves SSD performance |
| `ro` | Read-only |
| `dump` | `0` = don't backup with dump utility |
| `pass` | fsck order at boot: `0` = skip, `1` = root fs, `2` = others |

After editing `/etc/fstab`, test without rebooting:
```bash
sudo mount -a          # Mount all entries in fstab
sudo systemctl daemon-reload
```

---

## 7. Full New Disk Setup Workflow

```bash
# Step 1 — Identify the new disk
lsblk
# Look for a disk with no partitions (TYPE=disk, no children)

# Step 2 — Partition it
sudo fdisk /dev/sdb
# Inside fdisk: n → p → 1 → Enter → Enter → w

# Step 3 — Verify partition was created
lsblk
# Should now show sdb└─sdb1

# Step 4 — Format the partition
sudo mkfs.ext4 /dev/sdb1

# Step 5 — Create mount point
sudo mkdir /data

# Step 6 — Mount it
sudo mount /dev/sdb1 /data

# Step 7 — Verify
df -h /data

# Step 8 — Make it permanent
sudo blkid /dev/sdb1    # Copy the UUID
sudo nano /etc/fstab
# Add: UUID="<uuid>" /data ext4 defaults 0 2

# Step 9 — Test fstab without rebooting
sudo umount /data
sudo mount -a
df -h /data
```

---

## 8. Logical Volume Management (LVM)

LVM adds a layer of abstraction over physical disks, allowing you to resize volumes, add disks on the fly, and take snapshots without downtime.

```
Physical Disks (PV)  ──►  Volume Group (VG)  ──►  Logical Volumes (LV)
  /dev/sdb, /dev/sdc        vg_data                 lv_app, lv_logs
```

### LVM Concepts

| Layer | Name | Description |
|-------|------|-------------|
| Physical Volume (PV) | `/dev/sdb` | Raw disk or partition registered with LVM |
| Volume Group (VG) | `vg_data` | Pool of storage combining one or more PVs |
| Logical Volume (LV) | `lv_app` | Virtual partition carved out of a VG |

### Creating LVM Step by Step

```bash
# 1. Initialize disks as Physical Volumes
sudo pvcreate /dev/sdb
sudo pvcreate /dev/sdc

# Verify PVs
sudo pvs
sudo pvdisplay

# 2. Create a Volume Group from PVs
sudo vgcreate vg_data /dev/sdb /dev/sdc

# Verify VG
sudo vgs
sudo vgdisplay vg_data

# 3. Create Logical Volumes from the VG
sudo lvcreate -L 10G -n lv_app vg_data       # Fixed 10GB volume
sudo lvcreate -L 5G  -n lv_logs vg_data      # Fixed 5GB volume
sudo lvcreate -l 100%FREE -n lv_extra vg_data # Use all remaining space

# Verify LVs
sudo lvs
sudo lvdisplay

# 4. Format the Logical Volumes
sudo mkfs.ext4 /dev/vg_data/lv_app
sudo mkfs.ext4 /dev/vg_data/lv_logs

# 5. Mount them
sudo mkdir /app /var/log/app
sudo mount /dev/vg_data/lv_app /app
sudo mount /dev/vg_data/lv_logs /var/log/app

# 6. Add to fstab for persistence
echo "/dev/vg_data/lv_app  /app  ext4  defaults  0  2" | sudo tee -a /etc/fstab
```

### Extending a Logical Volume (Online Resize)

One of LVM's biggest advantages — grow a volume while it's mounted and in use:

```bash
# Add a new disk to the volume group first (if needed)
sudo pvcreate /dev/sdd
sudo vgextend vg_data /dev/sdd

# Extend the logical volume by 5GB
sudo lvextend -L +5G /dev/vg_data/lv_app

# Resize the filesystem to use the new space (ext4)
sudo resize2fs /dev/vg_data/lv_app

# For XFS filesystem (XFS can only grow, not shrink)
sudo xfs_growfs /app

# One-command extend + resize filesystem
sudo lvextend -L +5G -r /dev/vg_data/lv_app   # -r resizes filesystem automatically
```

### LVM Snapshots

```bash
# Create a snapshot of lv_app (requires free space in the VG)
sudo lvcreate -L 2G -s -n lv_app_snap /dev/vg_data/lv_app

# Mount the snapshot (read-only view of lv_app at this point in time)
sudo mount -o ro /dev/vg_data/lv_app_snap /mnt/snapshot

# Remove snapshot when done
sudo umount /mnt/snapshot
sudo lvremove /dev/vg_data/lv_app_snap
```

---

## 9. Swap Management

Swap is disk space used as overflow when RAM is full. It prevents out-of-memory crashes but is much slower than RAM.

### Swap Partition

```bash
# Create swap on a partition
sudo mkswap /dev/sdb2

# Enable the swap partition
sudo swapon /dev/sdb2

# Verify swap is active
free -h
swapon --show

# Disable swap
sudo swapoff /dev/sdb2
```

### Swap File (More Flexible — No Partition Needed)

```bash
# Create a 2GB swap file
sudo fallocate -l 2G /swapfile
# Or use dd if fallocate is unavailable:
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048

# Set correct permissions (must be root-readable only)
sudo chmod 600 /swapfile

# Initialize as swap
sudo mkswap /swapfile

# Enable it
sudo swapon /swapfile

# Verify
free -h
swapon --show

# Make it permanent
echo "/swapfile  none  swap  sw  0  0" | sudo tee -a /etc/fstab
```

### Swappiness — How Aggressively Linux Uses Swap

```bash
# Check current swappiness (default is 60)
cat /proc/sys/vm/swappiness

# Set lower value for servers (less aggressive swapping)
sudo sysctl vm.swappiness=10

# Make permanent
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
```

| Swappiness Value | Behavior |
|-----------------|---------|
| `0` | Avoid swap unless absolutely necessary |
| `10` | Recommended for database/server workloads |
| `60` | Default — balanced |
| `100` | Aggressively swap to free RAM for cache |

---

## 10. Disk Health Monitoring

```bash
# Install smartmontools
apt install smartmontools

# Check disk health (overall PASSED/FAILED)
sudo smartctl -H /dev/sda

# Full disk information
sudo smartctl -a /dev/sda

# Run a short self-test
sudo smartctl -t short /dev/sda

# Check for bad sectors
sudo badblocks -v /dev/sdb
```

---

## Summary

| Task | Command(s) |
|------|-----------|
| View all disks | `lsblk` |
| View partition details | `sudo fdisk -l` |
| Show UUIDs | `sudo blkid` |
| Check disk space | `df -h` |
| Check directory size | `du -sh /path` |
| Partition a disk | `sudo fdisk /dev/sdb` |
| Format as ext4 | `sudo mkfs.ext4 /dev/sdb1` |
| Format as XFS | `sudo mkfs.xfs /dev/sdb1` |
| Mount partition | `sudo mount /dev/sdb1 /mnt/point` |
| Unmount partition | `sudo umount /mnt/point` |
| Persistent mount | Edit `/etc/fstab` |
| Test fstab | `sudo mount -a` |
| Create PV (LVM) | `sudo pvcreate /dev/sdb` |
| Create VG (LVM) | `sudo vgcreate vg_name /dev/sdb` |
| Create LV (LVM) | `sudo lvcreate -L 10G -n lv_name vg_name` |
| Extend LV online | `sudo lvextend -L +5G -r /dev/vg/lv` |
| Create swap file | `fallocate -l 2G /swapfile && mkswap /swapfile && swapon /swapfile` |
| Check swap | `free -h` / `swapon --show` |

---

**← Back to [Chapter 9: Networking](../../chapter-9-networking/networking-commands/README.md)**  
**Next → Chapter 11: Shell Scripting**
