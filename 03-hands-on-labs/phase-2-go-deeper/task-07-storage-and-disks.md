# Task 07 — Storage & Disks

> **Prerequisite:** Read `01-notes/08-storage-and-disks.md`
> **Environment:** Full Linux system (VM recommended — safer to experiment with disks). You need `sudo` access.
> **Difficulty:** Intermediate

---

## Objective

By the end of this lab you will be able to:
- Check disk usage and find what's consuming space
- Understand block devices and partitions
- Mount and unmount filesystems
- Work with `/etc/fstab` for persistent mounts
- Create and manage swap space
- Use LVM basics (if available)

---

## Part 1 — Check Disk Usage

### Exercise 1.1: Overview of disk space

```bash
# Disk usage for all mounted filesystems
df -h

# Same but only for real filesystems (skip tmpfs, devtmpfs)
df -h -x tmpfs -x devtmpfs

# Disk usage for a specific path
df -h /home
```

### Exercise 1.2: Find what's eating space

```bash
# Size of a specific directory
du -sh /var/log
du -sh /home

# Top-level directories sorted by size
sudo du -sh /* 2>/dev/null | sort -rh | head -10

# Interactive disk usage viewer (install ncdu)
sudo apt install -y ncdu
sudo ncdu /        # Navigate with arrow keys, q to quit
```

### Exercise 1.3: Find large files

```bash
# Find files larger than 100MB
sudo find / -type f -size +100M 2>/dev/null | head -10

# Find with human-readable sizes
sudo find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | awk '{print $5, $9}' | head -10

# Find the 10 largest files
sudo find / -type f -exec du -h {} + 2>/dev/null | sort -rh | head -10
```

---

## Part 2 — Block Devices

### Exercise 2.1: List block devices

```bash
# List all block devices
lsblk

# With more detail
lsblk -f        # Shows filesystem type and UUID

# Old way
sudo fdisk -l
```

**Reading `lsblk` output:**
```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0    50G  0 disk
├─sda1   8:1    0     1G  0 part /boot
└─sda2   8:2    0    49G  0 part /
```

- `sda` = first disk
- `sda1`, `sda2` = partitions on that disk
- TYPE: `disk`, `part` (partition), `lvm` (logical volume)

### Exercise 2.2: Check filesystem type

```bash
# Check filesystem type
df -T

# Check a specific device
sudo blkid
```

---

## Part 3 — Mount and Unmount

### Exercise 3.1: Create a practice filesystem (using a file as a disk)

This is safe — we create a virtual disk from a file instead of touching real hardware:

```bash
mkdir -p ~/lab-07

# Create a 100MB file to use as a virtual disk
dd if=/dev/zero of=~/lab-07/virtual-disk.img bs=1M count=100

# Format it with ext4 filesystem
mkfs.ext4 ~/lab-07/virtual-disk.img

# Create a mount point
sudo mkdir -p /mnt/lab-disk

# Mount it
sudo mount ~/lab-07/virtual-disk.img /mnt/lab-disk

# Verify
df -h /mnt/lab-disk
mount | grep lab-disk
```

### Exercise 3.2: Use the mounted filesystem

```bash
# Create files on the mounted filesystem
sudo bash -c 'echo "Hello from mounted disk" > /mnt/lab-disk/test.txt'
sudo mkdir /mnt/lab-disk/data
sudo bash -c 'echo "data file" > /mnt/lab-disk/data/info.txt'

# List contents
ls -la /mnt/lab-disk/

# Check usage
df -h /mnt/lab-disk
```

### Exercise 3.3: Unmount

```bash
# Unmount the filesystem
sudo umount /mnt/lab-disk

# Verify it's unmounted
df -h | grep lab-disk     # Should return nothing
ls /mnt/lab-disk/         # Should be empty (mount point exists, but no content)

# Remount to verify data persists
sudo mount ~/lab-07/virtual-disk.img /mnt/lab-disk
cat /mnt/lab-disk/test.txt    # Data is still there
```

**Important:** If unmount fails with "device is busy":
```bash
# Check what's using the mount
sudo lsof /mnt/lab-disk

# Force unmount (use with caution)
sudo umount -l /mnt/lab-disk    # Lazy unmount
```

---

## Part 4 — Persistent Mounts with /etc/fstab

### Exercise 4.1: Understand /etc/fstab

```bash
# View current fstab
cat /etc/fstab
```

**Format:**
```
<device>    <mount-point>    <type>    <options>    <dump>    <pass>
/dev/sda1   /boot            ext4      defaults     0         2
```

| Field | Meaning |
|-------|---------|
| device | Device path or UUID |
| mount-point | Where to mount |
| type | Filesystem type (ext4, xfs, etc.) |
| options | Mount options (defaults, ro, noexec) |
| dump | Backup flag (0 = no) |
| pass | fsck check order (0 = skip) |

### Exercise 4.2: Add a persistent mount (practice only)

```bash
# Get the UUID of our virtual disk (not applicable for loop device, so use path)
# For real disks, use: sudo blkid /dev/sdXn

# Show what a fstab entry would look like (DO NOT add this to real fstab carelessly)
echo "# Example fstab entry for our lab disk:"
echo "~/lab-07/virtual-disk.img  /mnt/lab-disk  ext4  defaults,loop  0  0"

# Test fstab without rebooting
# sudo mount -a    # Mounts everything in fstab that isn't already mounted
```

**Warning:** A bad `/etc/fstab` entry can prevent your system from booting. Always test with `sudo mount -a` before rebooting.

---

## Part 5 — Swap Space

### Exercise 5.1: Check current swap

```bash
# Check swap status
free -h
swapon --show

# Detailed swap info
cat /proc/swaps
```

### Exercise 5.2: Create a swap file

```bash
# Create a 256MB swap file
sudo dd if=/dev/zero of=/swapfile-lab bs=1M count=256

# Set correct permissions
sudo chmod 600 /swapfile-lab

# Format as swap
sudo mkswap /swapfile-lab

# Enable it
sudo swapon /swapfile-lab

# Verify
free -h
swapon --show
```

### Exercise 5.3: Remove the swap file

```bash
# Disable the swap
sudo swapoff /swapfile-lab

# Remove the file
sudo rm /swapfile-lab

# Verify
free -h
```

---

## Part 6 — Useful Disk Commands

### Exercise 6.1: Monitor disk I/O

```bash
# Install iostat if needed
sudo apt install -y sysstat

# Check disk I/O statistics
iostat

# Watch I/O in real-time (every 2 seconds)
iostat -x 2 3    # 3 iterations, 2 seconds apart

# Check I/O per process
sudo iotop        # Install: sudo apt install -y iotop
```

### Exercise 6.2: Check filesystem health

```bash
# Check filesystem (must be unmounted first)
sudo umount /mnt/lab-disk
sudo fsck ~/lab-07/virtual-disk.img

# Remount after check
sudo mount ~/lab-07/virtual-disk.img /mnt/lab-disk
```

---

## Challenges

1. **Disk space audit:** Write a script that:
   - Reports total, used, and free space on `/`
   - Lists the top 5 largest directories under `/var`
   - Warns if disk usage exceeds 80%

2. **Create a second virtual disk** (200MB), format it as `xfs` (install `xfsprogs` if needed), mount it at `/mnt/lab-disk2`, create some files, unmount, and verify data persists

3. **Find all files modified in the last 24 hours** that are larger than 10MB

4. **Check your system's partition layout** with `lsblk -f` and draw a diagram of your disk structure

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Editing `/etc/fstab` without testing | Bad entry can prevent boot | Always run `sudo mount -a` to test first |
| Running `fsck` on a mounted filesystem | Can cause data corruption | Unmount first, then `fsck` |
| Ignoring "disk almost full" warnings | System can crash when disk hits 100% | Set up monitoring and alerts |
| Using `dd` without double-checking `of=` | Can overwrite your actual disk | Triple-check the output file path |
| Not using `sudo` for mount/umount | Regular users can't mount by default | Use `sudo` |

---

## Cleanup

```bash
sudo umount /mnt/lab-disk 2>/dev/null
sudo rmdir /mnt/lab-disk
rm -rf ~/lab-07
```

---

## What's Next?

Proceed to `task-08-services.md` → Write a custom systemd service.
