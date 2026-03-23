# Chapter 12: Disk Management
> **Difficulty**: Intermediate-Hard | **Estimated Time**: 3-4 days
---
## 12.1 Disk Partitioning Concepts
### What is Partitioning?
Partitioning divides a physical disk into separate logical sections called partitions. Each partition acts as an independent drive.
### Why Partition?
- Separate OS from data
- Improved security isolation
- Multiple filesystem types on one disk
- Easier backups and recovery
- Multi-boot systems
### Partition Types
| Type | Description | Limit |
|------|-------------|-------|
| **Primary** | Bootable partitions | Max 4 per disk |
| **Extended** | Container for logical partitions | Only 1 per disk |
| **Logical** | Created inside extended | Unlimited |
### Partitioning Schemes
| Scheme | Max Size | Max Partitions | Use Case |
|--------|----------|----------------|----------|
| **MBR** | 2TB | 4 primary | Legacy systems |
| **GPT** | 9.4ZB | 128 (typical) | Modern systems |
## 12.2 Partitioning Tools
### fdisk (MBR)
```bash
# Start fdisk
sudo fdisk /dev/sda

# Common fdisk commands:
# n : Create new partition
# d : Delete partition
# p : Print partition table
# w : Write changes to disk
# q : Quit without saving
# m : Show help
# l : List partition types
# t : Change partition type
```
### GParted (GUI)
```bash
# Install GParted
sudo apt install gparted
sudo gparted
```
### parted (Advanced)
```bash
# View all disks
sudo parted -l

# Enter parted interactive mode
sudo parted /dev/sdb

# parted commands:
# print - show partition table
# mklabel gpt - create GPT table
# mkpart primary 0% 50% - create partition
# quit - exit
```
## 12.3 Creating and Formatting Partitions
### Creating a Partition with fdisk
```bash
# 1. Start fdisk
sudo fdisk /dev/sdb

# 2. Create new partition (press n)
# 3. Choose primary or logical (press p or l)
# 4. Partition number (default 1)
# 5. First sector (press Enter for default)
# 6. Last sector or size (e.g., +10G for 10GB)
# 7. Write changes (press w)
```
### Formatting Partitions
```bash
# Format as ext4
sudo mkfs.ext4 /dev/sdb1

# Format as xfs
sudo mkfs.xfs /dev/sdb1

# Format as FAT32
sudo mkfs.vfat /dev/sdb1

# Format as NTFS
sudo mkfs.ntfs /dev/sdb1

# Format with label
sudo mkfs.ext4 -L mydata /dev/sdb1

# Format with specific block size
sudo mkfs.ext4 -b 4096 /dev/sdb1
```
### Verify Filesystem
```bash
# Check filesystem
sudo fsck /dev/sdb1

# Check XFS
sudo xfs_repair /dev/sdb1
```
## 12.4 Mounting Filesystems
### Mounting Manually
```bash
# Create mount point
sudo mkdir /mnt/data

# Mount the partition
sudo mount /dev/sdb1 /mnt/data

# Verify mount
mount | grep sdb1
df -h /mnt/data

# Unmount
sudo umount /mnt/data
```
### Mount by UUID (Recommended)
```bash
# Find UUID
sudo blkid /dev/sdb1
# Output: /dev/sdb1: UUID="abc-123" TYPE="ext4"

# Mount by UUID
sudo mount UUID="abc-123" /mnt/data
```
### Mount by Label
```bash
sudo mount LABEL=mydata /mnt/data
```
### Mount Options
```bash
# Common mount options
mount -o defaults /dev/sdb1 /mnt/data
mount -o ro /dev/sdb1 /mnt/data      # Read-only
mount -o noatime /dev/sdb1 /mnt/data # Skip access time
mount -o sync /dev/sdb1 /mnt/data    # Synchronous writes
mount -o uid=1000,gid=1000 /dev/sdb1 /mnt/data
```
### Auto-Mount at Boot (/etc/fstab)
```bash
# Get UUID
sudo blkid

# Edit fstab
sudo nano /etc/fstab

# Add line:
# UUID=abc-123  /mnt/data  ext4  defaults  0  2

# Fields:
# 1. Device (UUID/label/path)
# 2. Mount point
# 3. Filesystem type
# 4. Options
# 5. Dump (0=no, 1=yes)
# 6. Pass (0=skip, 1=root, 2=others)

# Test fstab
sudo mount -a
```
### View All Mounts
```bash
# Show mounted filesystems
mount
df -h
lsblk -f
findmnt
```
## 12.5 Logical Volume Management (LVM)
### Why LVM?
- Resize partitions on-the-fly
- Combine multiple disks into one volume
- Snapshots for backups
- Striping and mirroring
### LVM Terminology
| Term | Description |
|------|-------------|
| **Physical Volume (PV)** | Physical disk or partition |
| **Volume Group (VG)** | Pool of storage from PVs |
| **Logical Volume (LV)** | Virtual partition from VG |
### LVM Setup
```bash
# 1. Initialize Physical Volume
sudo pvcreate /dev/sdb1
sudo pvcreate /dev/sdc1

# 2. Create Volume Group
sudo vgcreate vg_data /dev/sdb1 /dev/sdc1

# 3. Create Logical Volume
sudo lvcreate -L 10G -n lv_app vg_data
sudo lvcreate -L 5G -n lv_db vg_data

# 4. Create filesystem
sudo mkfs.ext4 /dev/vg_data/lv_app

# 5. Mount
sudo mkdir /mnt/app
sudo mount /dev/vg_data/lv_app /mnt/app
```
### LVM Management
```bash
# View PV information
sudo pvs
sudo pvdisplay

# View VG information
sudo vgs
sudo vgdisplay

# View LV information
sudo lvs
sudo lvdisplay

# Extend LV
sudo lvextend -L +5G /dev/vg_data/lv_app

# Extend filesystem (ext4)
sudo resize2fs /dev/vg_data/lv_app

# Extend filesystem (xfs)
sudo xfs_growfs /mnt/app

# Create snapshot
sudo lvcreate -L 1G -s -n lv_app_snap /dev/vg_data/lv_app

# Remove LV
sudo lvremove /dev/vg_data/lv_app
```
## 12.6 Software RAID
### RAID Levels
| Level | Min Disks | Description | Pros | Cons |
|-------|-----------|-------------|------|------|
| **RAID 0** | 2 | Striping | Speed | No redundancy |
| **RAID 1** | 2 | Mirroring | Redundancy | 50% capacity |
| **RAID 5** | 3 | Striping + parity | Redundancy + space | Write speed |
| **RAID 6** | 4 | Dual parity | 2-disk tolerance | Slower writes |
| **RAID 10** | 4 | Stripe of mirrors | Speed + redundancy | 50% capacity |
### Create RAID 1 (Mirroring)
```bash
# Install mdadm
sudo apt install mdadm

# Create RAID 1 array
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1

# Check status
cat /proc/mdstat
sudo mdadm --detail /dev/md0

# Create filesystem
sudo mkfs.ext4 /dev/md0

# Mount
sudo mkdir /mnt/raid1
sudo mount /dev/md0 /mnt/raid1
```
### Create RAID 5 (Striping with Parity)
```bash
sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb1 /dev/sdc1 /dev/sdd1
```
### RAID Management
```bash
# Stop array
sudo mdadm --stop /dev/md0

# Add disk to array
sudo mdadm --add /dev/md0 /dev/sde1

# Remove disk
sudo mdadm /dev/md0 --fail /dev/sdb1
sudo mdadm /dev/md0 --remove /dev/sdb1

# Save configuration
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```
## 12.7 Swap Space Management
### Check Swap
```bash
free -h
swapon --show
cat /proc/swaps
```
### Create Swap File
```bash
# 1. Allocate file (4GB example)
sudo fallocate -l 4G /swapfile
# If fallocate not supported:
sudo dd if=/dev/zero of=/swapfile bs=1M count=4096

# 2. Set permissions
sudo chmod 600 /swapfile

# 3. Format as swap
sudo mkswap /swapfile

# 4. Enable swap
sudo swapon /swapfile

# 5. Verify
free -h

# 6. Make permanent (add to /etc/fstab)
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```
### Swap Configuration
```bash
# View swappiness (0-100)
cat /proc/sys/vm/swappiness

# Set swappiness temporarily
echo 10 | sudo tee /proc/sys/vm/swappiness

# Make permanent
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf

# Remove swap
sudo swapoff /swapfile
sudo rm /swapfile
# Remove from /etc/fstab
```
## 12.8 Disk Monitoring and Health
### Check Disk Health
```bash
# Install smartmontools
sudo apt install smartmontools

# Check disk health
sudo smartctl -H /dev/sda

# Full SMART report
sudo smartctl -a /dev/sda

# Run self-test
sudo smartctl -t short /dev/sda

# Check bad blocks
sudo badblocks -sv /dev/sda
```
### Check Disk I/O
```bash
# Install sysstat
sudo apt install sysstat

# I/O statistics
iostat -x 2

# I/O monitoring
iotop

# Watch disk I/O
watch -n 1 'iostat -x'
```
### Check Disk Usage
```bash
# Overall disk space
df -h

# Find largest files
du -ah / | sort -rh | head -20

# Find large directories
du -sh /* | sort -rh | head -10

# Find files larger than 1GB
find / -type f -size +1G -exec ls -lh {} \;
```
## 12.9 Advanced Filesystems
### XFS Filesystem
```bash
# Create XFS
sudo mkfs.xfs /dev/sdb1

# Check XFS
sudo xfs_info /dev/sdb1

# Repair XFS
sudo xfs_repair /dev/sdb1

# Shrink (NOT supported - grow only)
```
### Btrfs Filesystem
```bash
# Create Btrfs
sudo mkfs.btrfs /dev/sdb1

# Mount Btrfs
sudo mount /dev/sdb1 /mnt/btrfs

# Create subvolume
sudo btrfs subvolume create /mnt/btrfs/data

# Create snapshot
sudo btrfs subvolume snapshot /mnt/btrfs/data /mnt/btrfs/snapshot

# Balance filesystem
sudo btrfs balance /mnt/btrfs

# Check filesystem
sudo btrfs filesystem show /mnt/btrfs
sudo btrfs filesystem df /mnt/btrfs
```
### ext4 Filesystem
```bash
# Check disk usage per directory
du -sh /mnt/data/*

# Check filesystem
sudo e2fsck -f /dev/sdb1

# Tune ext4
sudo tune2fs -c 50 /dev/sdb1  # Check every 50 mounts
sudo tune2fs -i 7d /dev/sdb1  # Check weekly

# View ext4 info
sudo tune2fs -l /dev/sdb1
```
## 12.10 Practice Exercises
### Exercise 1: Basic Partitioning
1. Create a 5GB partition on a spare disk using fdisk
2. Format it as ext4
3. Mount it to /mnt/test
4. Add to /etc/fstab for auto-mount
5. Verify with `df -h`
### Exercise 2: LVM Setup
1. Create two 2GB PVs from spare disk partitions
2. Create a VG named "vg_lab"
3. Create a 2GB LV named "lv_lab"
4. Format and mount it
5. Extend the LV by 1GB
6. Resize the filesystem
### Exercise 3: RAID 1
1. Prepare two spare partitions
2. Create a RAID 1 array using mdadm
3. Monitor the sync process
4. Create filesystem and mount
5. Fail one disk and verify redundancy
6. Add disk back and resync
### Exercise 4: Swap Management
1. Create a 2GB swap file
2. Set swappiness to 20
3. Make it permanent
4. Verify with `free -h`
5. Remove swap file properly
### Exercise 5: Disk Health Check
1. Check SMART health of your disks
2. Find files larger than 100MB in /var
3. Identify the largest directory in /home
4. Monitor I/O while running a stress test
## Key Takeaways
- **fdisk/parted**: Partition disk drives
- **mkfs**: Format partitions with filesystems
- **mount/fstab**: Mount and auto-mount filesystems
- **LVM**: Flexible logical volume management
- **RAID**: Software redundancy and striping
- **swap**: Virtual memory extension
- **smartmontools**: Monitor disk health
- **UUID**: Preferred way to reference disks in fstab
## Next Steps
- Study kernel module management
- Learn security hardening techniques
- Master systemd service management
- Explore performance tuning
- Practice disaster recovery procedures
## Next**: Move to [Chapter 13: Linux Kernel Management](13-kernel-management.md)
