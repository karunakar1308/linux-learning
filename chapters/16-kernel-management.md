# Chapter 16: Kernel Management

## Introduction

The Linux kernel is the core of the operating system, responsible for managing hardware resources, process scheduling, memory management, and providing system calls for user-space applications. Understanding kernel management is essential for system administrators and DevOps engineers.

## What is the Linux Kernel?

The kernel is the lowest-level software layer that interacts directly with hardware. It provides:
- **Process Management**: Creating, scheduling, and terminating processes
- **Memory Management**: Allocating and freeing memory, virtual memory
- **Device Management**: Driver interfaces for hardware
- **System Calls**: Interface between user space and kernel space
- **File System Management**: Handling file operations and storage

## Checking Kernel Information

### Current Kernel Version

```bash
# Check kernel version
uname -r

# Check all kernel info
uname -a

# More detailed kernel info
cat /proc/version

# Check kernel architecture
uname -m
```

### Loaded Kernel Modules

```bash
# List all loaded kernel modules
lsmod

# List modules with details
lsmod | column -t

# Search for a specific module
lsmod | grep ext4

# Count loaded modules
lsmod | wc -l
```

### Kernel Parameters

```bash
# List all kernel parameters
sysctl -a

# Search for specific parameters
sysctl -a | grep -i mem

# View a specific parameter
sysctl net.ipv4.tcp_max_syn_backlog

# Check parameters from proc filesystem
cat /proc/sys/kernel/randomize_va_space
```

## Managing Kernel Modules

### Loading Modules

```bash
# Load a module
sudo modprobe <module_name>

# Load module with options
sudo modprobe -v bonding mode=802.3ad

# Load module from specific file
sudo insmod /lib/modules/$(uname -r)/kernel/drivers/<path>.ko

# Check if module loaded successfully
lsmod | grep <module_name>
```

### Unloading Modules

```bash
# Unload a module
sudo modprobe -r <module_name>

# Force unload (use carefully)
sudo rmmod <module_name>

# Check if module is in use
lsof | grep <module_name>
```

### Module Information

```bash
# Get module info
modinfo <module_name>

# Get module dependencies
modinfo -F depends <module_name>

# Check module signing
modinfo <module_name> | grep -i sign

# View module parameters
modinfo -p <module_name>
```

### Blacklisting Modules

```bash
# Blacklist a module temporarily
sudo modprobe -r <module_name>

# Permanently blacklist a module
echo "blacklist <module_name>" | sudo tee /etc/modprobe.d/blacklist.conf

# Update initramfs after blacklisting
sudo update-initramfs -u  # Debian/Ubuntu
sudo dracut --force       # RHEL/CentOS
```

## Kernel Parameters (sysctl)

### Reading Parameters

```bash
# Read all parameters
cat /proc/sys/*/*

# Read specific category
cat /proc/sys/net/ipv4/ip_forward

# Using sysctl
sysctl kernel.hostname
sysctl vm.swappiness
```

### Modifying Parameters

```bash
# Temporary change (lost on reboot)
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl net.ipv4.ip_forward=1

# Using proc filesystem
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Permanent change
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf

# Apply all changes from sysctl.conf
sudo sysctl -p

# Apply specific file
sudo sysctl -p /etc/sysctl.d/99-custom.conf
```

### Common Important Parameters

```bash
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Increase maximum connections
sudo sysctl -w net.core.somaxconn=65535

# Tune TCP stack
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65536
sudo sysctl -w net.ipv4.tcp_fin_timeout=30
sudo sysctl -w net.ipv4.tcp_tw_reuse=1

# Memory settings
sudo sysctl -w vm.swappiness=10
sudo sysctl -w vm.dirty_ratio=20
sudo sysctl -w vm.dirty_background_ratio=10

# File descriptors
sudo sysctl -w fs.file-max=2097152
sudo sysctl -w fs.nr_open=1048576
```

## Kernel Tuning for Performance

### Network Tuning

```bash
# Optimize TCP buffer sizes
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 67108864"
sudo sysctl -w net.ipv4.tcp_wmem="4096 65536 67108864"

# Increase connection queue
sudo sysctl -w net.core.netdev_max_backlog=50000
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=8192

# Enable TCP BBR congestion control
sudo sysctl -w net.core.default_qdisc=fq
sudo sysctl -w net.ipv4.tcp_congestion_control=bbr
```

### Memory Tuning

```bash
# Adjust swappiness (lower = less swap usage)
sudo sysctl -w vm.swappiness=1

# Tune dirty page writeback
sudo sysctl -w vm.dirty_ratio=15
sudo sysctl -w vm.dirty_background_ratio=5
sudo sysctl -w vm.dirty_expire_centisecs=3000
sudo sysctl -w vm.dirty_writeback_centisecs=500

# Adjust overcommit behavior
sudo sysctl -w vm.overcommit_memory=1
sudo sysctl -w vm.overcommit_ratio=90
```

## Kernel Upgrades

### Check Available Kernels

```bash
# Debian/Ubuntu - check available kernels
apt-cache search linux-image

# Check installed kernels
dpkg --list | grep linux-image

# RHEL/CentOS/Fedora
yum list kernel
rpm -qa | grep kernel
```

### Installing a New Kernel

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install linux-image-generic  # Latest stable
sudo apt install linux-image-5.15.0-generic  # Specific version

# RHEL/CentOS
sudo yum install kernel

# Fedora
sudo dnf install kernel
```

### GRUB Configuration for Multiple Kernels

```bash
# View GRUB entries
cat /boot/grub/grub.cfg | grep menuentry

# Set default kernel in GRUB
sudo nano /etc/default/grub
# Change: GRUB_DEFAULT=0 (0 = first entry)

# Update GRUB configuration
sudo update-grub    # Debian/Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # RHEL/CentOS

# List boot entries (GRUB2)
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /boot/grub2/grub.cfg
```

### Kernel Build from Source (Advanced)

```bash
# Install build dependencies
sudo apt install build-essential libssl-dev bc kmod cpio flex bison

# Download kernel source
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.15.tar.xz
tar -xf linux-5.15.tar.xz
cd linux-5.15

# Configure kernel
cp /boot/config-$(uname -r) .config
make menuconfig  # Interactive configuration

# Build kernel
make -j$(nproc)
sudo make modules_install
sudo make install
```

## Kernel Security

### Kernel Security Modules

```bash
# Check available security modules
cat /sys/kernel/security/lsm

# Check if SELinux is enabled
getenforce
sestatus

# Check if AppArmor is enabled
sudo apparmor_status

# Enable SELinux
sudo setenforce 1

# Enable AppArmor for a profile
sudo aa-enforce /usr/sbin/app-profile
```

### Kernel Hardening

```bash
# Disable Ctrl+Alt+Del
sudo systemctl mask ctrl-alt-del.target

# Disable core dumps
echo "* soft core 0" | sudo tee -a /etc/security/limits.conf

# Restrict dmesg access
sudo sysctl -w kernel.dmesg_restrict=1

# Restrict kernel pointers
sudo sysctl -w kernel.kptr_restrict=2

# Restrict unprivileged user namespaces
sudo sysctl -w user.max_user_namespaces=0
```

## Exercises

### Exercise 1: Kernel Information Gathering

1. Check your current kernel version
2. Count the number of loaded kernel modules
3. Check your kernel architecture
4. Find out the hostname from kernel parameters

### Exercise 2: Module Management

1. List all loaded modules
2. Find the module for your network interface
3. Check the dependencies of the ext4 module
4. Try loading and unloading a harmless module (e.g., loop)

### Exercise 3: Sysctl Configuration

1. Check your current swappiness value
2. Temporarily change swappiness to 10
3. Verify the change
4. Create a custom sysctl config file with network optimizations
5. Apply the configuration

### Exercise 4: Kernel Tuning

1. Check current TCP settings
2. Apply TCP optimization parameters
3. Verify the changes
4. Create a persistent sysctl.d configuration

### Exercise 5: Kernel Module Blacklisting

1. Identify a module you want to blacklist (e.g., bluetooth if not needed)
2. Add it to the blacklist configuration
3. Verify it's blacklisted
4. Check lsmod output

## Key Takeaways

- **Kernel Version**: Always check with `uname -r` before making changes
- **Modules**: Use `modprobe` for loading/unloading, `lsmod` for listing
- **Sysctl**: Temporary changes with `-w`, permanent in `/etc/sysctl.conf` or `/etc/sysctl.d/`
- **Parameters**: Check `/proc/sys/` for current values
- **Performance**: Tune TCP stack, memory, and file descriptors for server workloads
- **Security**: Enable LSM (SELinux/AppArmor), restrict kernel pointers, disable unused modules
- **Upgrades**: Always keep a working kernel available when upgrading
- **Testing**: Test kernel changes in a VM before production

## Troubleshooting

```bash
# Module load failed - check for errors
dmesg | tail -20
journalctl -k --since "1 hour ago"

# Parameter not persisting
sudo sysctl -p --help

# Kernel panic on boot
# Boot into recovery mode from GRUB menu
# Select previous working kernel

# Module dependency issues
depmod -a
```

## Next Steps

- Learn about kernel debugging with kdump and crash utility
- Explore eBPF for kernel-level observability
- Study container runtime and kernel namespaces
- Deep dive into kernel security modules (LSM framework)
