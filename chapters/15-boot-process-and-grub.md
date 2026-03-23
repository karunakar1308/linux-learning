# Chapter 15: Boot Process and GRUB
> **Difficulty**: Advanced | **Estimated Time**: 3-4 days
---

## 15.1 The Linux Boot Process Overview

The Linux boot process consists of multiple stages, each with specific responsibilities.

### Boot Process Stages
```
1. BIOS/UEFI (Firmware)
2. Bootloader (GRUB2)
3. Kernel Loading
4. Init System (systemd)
5. Services and Targets
6. Login Prompt / Desktop
```

### Understanding Each Stage

**Stage 1: BIOS/UEFI**
- Firmware initializes hardware
- Runs Power-On Self-Test (POST)
- Locates and loads bootloader from boot device
- UEFI uses EFI System Partition (ESP)

**Stage 2: Bootloader (GRUB2)**
- Loads kernel image and initramfs into memory
- Passes kernel parameters
- Displays boot menu

**Stage 3: Kernel Initialization**
- Decompresses and initializes kernel
- Detects hardware
- Mounts root filesystem
- Executes /sbin/init (systemd)

**Stage 4: Init System (systemd)**
- Starts services and targets
- Manages dependencies
- Provides user sessions

---

## 15.2 GRUB Bootloader

### What is GRUB?
GRUB (Grand Unified Bootloader) is the default bootloader for most Linux distributions.

### GRUB Configuration Files
```bash
# Main configuration file (generated, do not edit directly)
/boot/grub/grub.cfg

# GRUB default settings (edit this)
/etc/default/grub

# Custom scripts (add here)
/etc/grub.d/
```

### Key GRUB Settings (/etc/default/grub)
```bash
# Default boot entry (0 = first entry)
GRUB_DEFAULT=0

# Timeout in seconds
GRUB_TIMEOUT=5

# Hide menu unless key pressed
GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT_STYLE=menu

# Kernel command line parameters
GRUB_CMDLINE_LINUX="quiet splash"
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"

# Enable serial console
GRUB_TERMINAL=console

# Enable GRUB menu password
GRUB_PASSWORD=grub.pbkdf2.sha512.hash...
```

### GRUB Commands
```bash
# Update GRUB configuration (Debian/Ubuntu)
sudo update-grub

# Generate GRUB config
sudo grub-mkconfig -o /boot/grub/grub.cfg

# Generate for RHEL/CentOS
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Install GRUB to disk
sudo grub-install /dev/sda

# Reinstall GRUB
sudo grub-install --recheck /dev/sda
```

---

## 15.3 GRUB Menu and Recovery

### GRUB Boot Menu
```
GRUB Boot Menu:
- Ubuntu (default)
- Ubuntu (recovery mode)
- Memory test (memtest86+)
- Windows Boot Manager
```

### Accessing GRUB
- **Shift key**: Hold during boot to show menu
- **Esc key**: On UEFI systems
- **e**: Edit boot parameters
- **c**: GRUB command line

### Editing Boot Parameters Temporarily
```
1. Select boot entry in GRUB menu
2. Press 'e' to edit
3. Find the line starting with 'linux'
4. Add parameters at the end of that line
5. Press Ctrl+X or F10 to boot
```

### Common Boot Parameters
```bash
# Boot into single-user mode
single
1

# Boot into emergency mode
systemd.unit=emergency.target

# Disable splash and show all messages
quiet splash  (remove these)

# Set specific runlevel (deprecated)
3

# Disable a specific module
module_name.blacklist=1

# Enable kernel debugging
debug

# Set specific console
console=ttyS0,115200n8
```

---

## 15.4 Init System (systemd)

### Understanding systemd
systemd is the init system and service manager for modern Linux.

### systemd Hierarchy
```
systemd (PID 1)
  \
   |-- target: multi-user.target (runlevel 3)
   |     |-- sshd.service
   |     |-- networking.service
   |     \-- ... many services
   |
   \-- target: graphical.target (runlevel 5)
         |-- display-manager.service
         \-- gnome-shell.service
```

### systemd Targets (vs. Runlevels)
```
Runlevel    systemd Target          Description
0           poweroff.target         Shutdown
1           rescue.target           Single user
2           multi-user.target       Multi-user, no GUI
3           multi-user.target       Multi-user, no GUI
4           multi-user.target       Custom
5           graphical.target        Multi-user with GUI
6           reboot.target           Reboot
```

### Target Commands
```bash
# View current target
systemctl get-default

# Set default target
sudo systemctl set-default multi-user.target
sudo systemctl set-default graphical.target

# Change target immediately
sudo systemctl isolate multi-user.target
sudo systemctl isolate graphical.target

# List all targets
systemctl list-units --type=target

# Show dependencies of a target
systemctl list-dependencies default.target
```

---

## 15.5 Managing Services with systemd

### Service Lifecycle
```bash
# Start a service
sudo systemctl start service

# Stop a service
sudo systemctl stop service

# Restart a service
sudo systemctl restart service

# Reload configuration (without restart)
sudo systemctl reload service

# Check status
systemctl status service

# Enable at boot
sudo systemctl enable service

# Disable at boot
sudo systemctl disable service

# Mask a service (cannot start)
sudo systemctl mask service

# Unmask a service
sudo systemctl unmask service
```

### Listing Services
```bash
# All services
systemctl list-units --type=service

# Running services
systemctl list-units --type=service --state=running

# Failed services
systemctl --failed

# All units
systemctl list-units --all

# Show tree view
systemctl list-dependencies

# Show unit properties
systemctl show service
```

### Service Analysis
```bash
# Boot time analysis
systemd-analyze

# Show boot time per service
systemd-analyze blame

# Show boot critical chain
systemd-analyze critical-chain

# View timeline
systemd-analyze plot > boot.svg
```

---

## 15.6 Boot Problems and Recovery

### Common Boot Issues

**1. GRUB not loading**
```bash
# Boot from live USB
# Mount the installation
sudo mount /dev/sda1 /mnt

# Chroot into system
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt

# Reinstall GRUB
grub-install /dev/sda
update-grub
exit
sudo reboot
```

**2. Kernel panic**
```bash
# Boot into recovery mode from GRUB
# Select 'recovery mode' in GRUB menu
# Choose 'root' shell
# Check filesystem
fsck /dev/sda1
# Check logs
journalctl -xb
```

**3. Cannot find root filesystem**
```bash
# At GRUB: press 'e' to edit
# Check the root= parameter in linux line
# Verify correct partition: (hd0,0) vs /dev/sda1
# Check UUID: blkid
```

**4. Emergency mode boot**
```bash
# Check why: journalctl -xb
# Common causes:
# - Filesystem errors
# - Missing mount entries in /etc/fstab
# - Missing /etc/crypttab entries

# Fix: remount root as read-write
mount -o remount,rw /

# Check fstab
cat /etc/fstab
nano /etc/fstab
```

---

## 15.7 initramfs / initrd

### What is initramfs?
Initial RAM Filesystem - temporary root filesystem loaded into memory during boot.

### Contents of initramfs
- Kernel modules needed to mount root filesystem
- Scripts for LVM, RAID, encryption
- Tools for root filesystem detection

### Managing initramfs
```bash
# Regenerate initramfs (Debian/Ubuntu)
sudo update-initramfs -u
sudo update-initramfs -c -k $(uname -r)

# Regenerate initramfs (RHEL/CentOS)
sudo dracut -f
sudo dracut --force $(uname -r)

# List initramfs contents
lsinitcpio /boot/initramfs.img  # Arch
unmkinitramfs /boot/initrd.img /tmp/initrd

# Add custom module to initramfs
echo "module_name" | sudo tee -a /etc/initramfs-tools/modules
sudo update-initramfs -u
```

### Debug initramfs
```bash
# Add debug option to GRUB
GRUB_CMDLINE_LINUX="break=top"
sudo update-grub

# Boot will stop at initramfs prompt
# Use commands like:
ls /dev/           # List devices
ls /dev/sd*        # List storage
blkid              # Show partitions
cat /proc/cmdline  # Boot parameters
exit               # Continue boot
```

---

## 15.8 Bootloader Security

### GRUB Password
```bash
# Generate password hash
grub-mkpasswd-pbkdf2

# Copy the hash output
# Add to /etc/grub.d/40_custom:
set superusers="admin"
password_pbkdf2 admin YOUR_HASH_HERE

# Update GRUB
sudo update-grub
```

### Secure Boot
```bash
# Check if Secure Boot is enabled
mokutil --sb-state

# Enroll MOK keys
sudo mokutil --import /path/to/key

# Disable Secure Boot (in BIOS/UEFI)
# Boot into firmware settings
# Find Secure Boot option
# Disable it
```

---

## 15.9 Kernel Command Line

### Common Parameters
```bash
# Set root filesystem
root=/dev/sda2
root=UUID=xxxx-xxxx-xxxx

# Read-only root (for repairs)
root=/dev/sda2 ro

# Set hostname at boot
hostname=myserver

# Disable IPv6
ipv6.disable=1

# Set CPU frequency governor
intel_pstate=disable

# Disable specific hardware
nouveau.modeset=0
i915.modeset=0

# Systemd targets
systemd.unit=multi-user.target
systemd.unit=rescue.target
```

### Viewing Current Parameters
```bash
# Show kernel command line
cat /proc/cmdline

# Show bootloader entry
bootctl status
```

---

## 15.10 System Boot Performance

### Measuring Boot Time
```bash
# Total boot time
systemd-analyze

# Per-service boot time
systemd-analyze blame

# Critical path
systemd-analyze critical-chain

# Generate SVG timeline
systemd-analyze plot > boot.svg
```

### Optimizing Boot
```bash
# Disable unnecessary services
sudo systemctl disable cups
sudo systemctl disable bluetooth
sudo systemctl disable network-manager

# Use parallel startup
# systemd does this by default

# Use compressed kernel
zcat /boot/vmlinuz* | wc -c

# Minimize initramfs size
ls -lh /boot/initrd*
```

---

## 15.11 Dual Boot Configuration

### Installing GRUB for Dual Boot
```bash
# Install OS/2 boot manager entry
sudo os-prober
sudo update-grub

# Chain load Windows bootloader
# GRUB auto-detects Windows
# If not, add to /etc/grub.d/40_custom:
menuentry "Windows 10" {
    set root=(hd0,1)
    chainloader +1
}
```

---

## Practice Exercises

### Exercise 1: GRUB Configuration
1. View current GRUB configuration
2. Change timeout from default to 10 seconds
3. Set default boot entry to recovery mode
4. Regenerate GRUB config
5. Verify changes on reboot

### Exercise 2: systemd Targets
1. Check current default target
2. Switch to multi-user target temporarily
3. Set multi-user as default
4. List all active services
5. Switch back to graphical target

### Exercise 3: Boot Analysis
1. Check total boot time
2. Identify slowest services with systemd-analyze blame
3. Generate boot timeline SVG
4. Identify services to disable for faster boot

### Exercise 4: Recovery Mode
1. Boot into recovery mode from GRUB
2. Access root shell
3. Check and repair filesystem
4. View system logs
5. Reboot normally

### Exercise 5: initramfs
1. Regenerate initramfs with current kernel
2. Check initramfs size
3. Add a custom kernel module
4. Verify module is included
5. Boot and confirm

## Key Takeaways
- **Boot stages**: BIOS/UEFI -> GRUB -> Kernel -> Init (systemd) -> Services
- **GRUB**: Bootloader that loads kernel with parameters
- **systemd**: Modern init system managing services and targets
- **Targets**: Replace traditional runlevels (3=multi-user, 5=graphical)
- **initramfs**: Temporary root filesystem for boot process
- **Recovery**: GRUB edit and recovery mode for troubleshooting
- **Security**: GRUB passwords and Secure Boot

## Next Steps
- Learn advanced systemd unit file creation
- Study network boot (PXE)
- Explore initramfs customization
- Practice disk encryption during boot
- Study container boot processes
