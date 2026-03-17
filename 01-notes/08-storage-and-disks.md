# 08 — Storage and Disks

> Disk management is critical in DevOps — full disks crash applications silently. Understanding storage helps you provision, expand, and maintain server volumes.

---

## Checking Disk Usage

```bash
df -h                           # disk usage of all filesystems (human readable)
df -h /                         # just root filesystem
df -i                           # inode usage (can be "full" even if space is free)
du -sh /var/log/                # size of a directory
du -sh /*  2>/dev/null          # size of all top-level directories
du -sh * | sort -rh | head -10  # top 10 largest items in current dir
ncdu /                          # interactive disk usage (install: apt install ncdu)
```

---

## Block Devices

```bash
lsblk                           # list block devices (disks, partitions) as a tree
lsblk -f                        # show filesystem types and mount points
fdisk -l                        # list all disks and partitions (root)
blkid                           # show UUIDs and filesystem types
```

---

## Mounting Filesystems

```bash
# Mount manually
sudo mount /dev/sdb1 /mnt/data
sudo mount -t ext4 /dev/sdb1 /mnt/data    # specify type

# Unmount
sudo umount /mnt/data
sudo umount /dev/sdb1

# See currently mounted filesystems
mount | column -t
cat /proc/mounts
```

### `/etc/fstab` — Persistent mounts

```
# /etc/fstab
# device        mountpoint    type    options         dump  pass
/dev/sda1       /             ext4    defaults        0     1
/dev/sdb1       /data         ext4    defaults,nofail 0     2
UUID=abc-123    /backup       xfs     defaults        0     2

# Mount all entries in fstab
sudo mount -a

# Get UUID of a device
blkid /dev/sdb1
```

---

## Creating and Formatting Partitions

```bash
# Create partition (interactive)
sudo fdisk /dev/sdb
# n = new partition, p = primary, w = write

# Format
sudo mkfs.ext4 /dev/sdb1        # format as ext4
sudo mkfs.xfs /dev/sdb1         # format as XFS
sudo mkfs.vfat /dev/sdb1        # format as FAT32

# Check and repair filesystem
sudo fsck /dev/sdb1             # check (unmount first)
sudo e2fsck -f /dev/sdb1        # ext4 check
```

---

## Extending a Disk (Cloud — Common Scenario)

```bash
# After resizing disk in AWS/GCP console:
lsblk                           # confirm new size visible
sudo growpart /dev/xvda 1       # extend partition to fill disk
sudo resize2fs /dev/xvda1       # extend ext4 filesystem
# or for XFS:
sudo xfs_growfs /               # extend XFS filesystem
```

---

## LVM — Logical Volume Manager

LVM adds a flexible abstraction layer over physical disks — resize volumes without downtime.

```
Physical Disks (/dev/sdb, /dev/sdc)
       ↓  (pvcreate)
Physical Volumes (PV)
       ↓  (vgcreate)
Volume Group (VG)  ← pool of all disk space
       ↓  (lvcreate)
Logical Volumes (LV)  ← what you format and mount
       ↓  (mkfs)
Filesystem
```

```bash
# Create PV
sudo pvcreate /dev/sdb

# Create VG
sudo vgcreate data-vg /dev/sdb

# Create LV (50GB)
sudo lvcreate -L 50G -n data-lv data-vg

# Format and mount
sudo mkfs.ext4 /dev/data-vg/data-lv
sudo mount /dev/data-vg/data-lv /data

# Extend LV (add 20GB)
sudo lvextend -L +20G /dev/data-vg/data-lv
sudo resize2fs /dev/data-vg/data-lv

# Show info
sudo pvs         # physical volumes
sudo vgs         # volume groups
sudo lvs         # logical volumes
```

---

## Swap Space

```bash
# Check swap
swapon --show
free -h

# Create swap file (2GB)
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```
