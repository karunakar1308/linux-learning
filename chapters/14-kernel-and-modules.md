# Chapter 14: Kernel and Modules
> **Difficulty**: Advanced | **Estimated Time**: 3-4 days
---

## 14.1 The Linux Kernel

### What is the Kernel?
The kernel is the core of the Linux operating system. It manages hardware resources, memory, processes, and provides an abstraction layer between hardware and software.

### Kernel Responsibilities
- **Process Management**: Create, schedule, and terminate processes
- **Memory Management**: Allocate, deallocate, and manage RAM
- **Device Management**: Communicate with hardware via drivers
- **File System Management**: Handle reading/writing to storage
- **Network Management**: Manage network interfaces and protocols
- **Security**: Enforce permissions and access control

### Checking Kernel Information
```bash
# View kernel version
uname -r

# Full system info
uname -a

# View kernel release info
cat /proc/version

# Check if 64-bit
uname -m

# View loaded kernel modules
cat /proc/modules | head -20

# View kernel parameters
sysctl -a | head -50
```

---

## 14.2 Kernel Versions

### Understanding Version Numbers
```bash
# Example: 5.15.0-76-generic
# 5       = Major version
# 15      = Minor version
# 0       = Patch level
# 76      = Distribution-specific build number
# generic = Kernel variant
```

### Kernel Versioning Scheme
- **Stable**: Odd minor numbers (5.15, 5.17) are stable for production
- **Development**: Even minor numbers (5.16, 5.18) are development kernels
- **LTS (Long Term Support)**: Maintained for 2-6 years (e.g., 5.10, 5.15)

### Check Available Kernels
```bash
# List installed kernels (Debian/Ubuntu)
dpkg --list | grep linux-image

# List installed kernels (RHEL/CentOS)
rpm -qa | grep kernel

# View current running kernel
uname -r

# List all kernels in /boot
ls /boot/vmlinuz*
```

---

## 14.3 Kernel Modules

### What are Kernel Modules?
Kernel modules are pieces of code that can be loaded and unloaded into the kernel on demand. They extend kernel functionality without rebooting.

### Common Module Types
- **Device drivers**: Network cards, USB, graphics
- **Filesystem support**: NFS, SMB, ext4
- **Network protocols**: IPv6, bridge, bonding
- **Security modules**: AppArmor, SELinux
- **Hardware support**: WiFi, Bluetooth, GPU

### Module Commands
```bash
# List loaded modules
lsmod

# List with details
lsmod | column -t

# Search for a specific module
lsmod | grep ext4

# Load a module
sudo modprobe module_name

# Load a module with parameters
sudo modprobe module_name parameter=value

# Unload a module
sudo modprobe -r module_name

# Force unload (dangerous)
sudo modprobe -r -f module_name
```

### Detailed Module Info
```bash
# Show module info
modinfo module_name

# Show module dependencies
modinfo -d module_name

# Show module license
modinfo -l module_name

# Show all parameters
modinfo -p module_name
```

### Module Files and Directories
```bash
# Module location
ls /lib/modules/$(uname -r)/

# Kernel modules directory
ls /lib/modules/$(uname -r)/kernel/

# Module dependencies file
cat /lib/modules/$(uname -r)/modules.dep

# Loaded modules
ls /proc/modules
```

---

## 14.4 Blacklisting Modules

### Why Blacklist?
- Prevent loading of unnecessary modules
- Security: Disable unused hardware features
- Performance: Reduce memory footprint
- Conflict resolution: Prevent driver conflicts

### Blacklisting a Module
```bash
# Create blacklist file
sudo nano /etc/modprobe.d/blacklist.conf

# Add line to blacklist module
blacklist module_name

# Blacklist with reason
blacklist module_name    # Conflicts with xyz

# Example: Blacklist nouveau for NVIDIA
blacklist nouveau
options nouveau modeset=0

# Update initramfs
sudo update-initramfs -u

# Reboot for changes
echo "Reboot required"
```

---

## 14.5 /proc and /sys Filesystems

### The /proc Filesystem
Virtual filesystem providing runtime system information.

```bash
# Process information
cat /proc/cpuinfo          # CPU details
cat /proc/meminfo          # Memory details
cat /proc/version          # Kernel version
cat /proc/uptime           # System uptime
cat /proc/loadavg          # Load averages
cat /proc/interrupts       # IRQ info
cat /proc/mounts           # Mounted filesystems
cat /proc/swaps            # Swap usage
cat /proc/net/dev          # Network stats
cat /proc/dma              # DMA channels
cat /proc/ioports          # I/O port ranges

# Process-specific info
cat /proc/PID/cmdline      # Command line
cat /proc/PID/environ      # Environment variables
cat /proc/PID/status       # Process status
cat /proc/PID/fd/          # File descriptors
```

### The /sys Filesystem
Virtual filesystem exposing kernel objects and their attributes.

```bash
# View block devices
ls /sys/block/

# View network interfaces
ls /sys/class/net/

# View CPU info
ls /sys/devices/system/cpu/

# View GPU info
ls /sys/class/drm/

# Check PCI devices
ls /sys/bus/pci/devices/

# View USB devices
ls /sys/bus/usb/devices/

# Modify kernel behavior
echo 1 > /sys/module/module_name/parameters/param
```

---

## 14.6 Kernel Parameters

### Runtime Parameter Modification
```bash
# View all kernel parameters
sysctl -a

# View specific parameter
sysctl net.ipv4.ip_forward

# Set parameter temporarily
sudo sysctl -w net.ipv4.ip_forward=1

# Set parameter with direct file
echo 1 > /proc/sys/net/ipv4/ip_forward
```

### Common Kernel Parameters
```bash
# Memory
vm.swappiness              # How aggressive swap is (default: 60)
vm.dirty_ratio             # % of RAM for dirty pages
vm.overcommit_memory       # Memory allocation policy

# Network
net.ipv4.ip_forward        # Enable IP forwarding
net.ipv4.tcp_syncookies    # SYN flood protection
net.ipv4.conf.all.accept_redirects  # ICMP redirects
net.core.somaxconn         # Max socket connections
net.core.rmem_max          # Max receive buffer

# Process
kernel.pid_max             # Maximum PID value
kernel.panic               # Reboot on panic
kernel.sysrq               # Enable magic SysRq key
```

### Persistent Configuration
```bash
# Edit sysctl configuration
sudo nano /etc/sysctl.conf

# Add custom settings
net.ipv4.ip_forward = 1
vm.swappiness = 10
kernel.panic = 10

# Apply all settings
sudo sysctl -p

# Apply specific file
sudo sysctl -p /etc/sysctl.d/99-custom.conf

# Create new config file
sudo nano /etc/sysctl.d/99-custom.conf
```

---

## 14.7 Kernel Logs

### View Kernel Messages
```bash
# View kernel ring buffer
dmesg

# View kernel messages in real-time
dmesg -w

# View messages with timestamps
dmesg -T

# Filter by level
dmesg -l err,warn

# View messages since boot
dmesg --since boot

# View messages by facility
dmesg -f kern

# Clear kernel ring buffer
dmesg -C

# View kernel logs via journalctl
journalctl -k

# Follow kernel logs
journalctl -k -f
```

### Understanding dmesg Output
```bash
# Hardware detection messages
dmesg | grep -i usb

# Storage device messages
dmesg | grep -i sd

# Network interface messages
dmesg | grep -i eth

# Error messages only
dmesg | grep -i error

# Warning messages only
dmesg | grep -i warn
```

---

## 14.8 Compiling a Kernel (Advanced)

### Prerequisites
```bash
# Install build dependencies (Debian/Ubuntu)
sudo apt install build-essential libncurses-dev bison flex libssl-dev libelf-dev

# Install dependencies (RHEL/CentOS)
sudo dnf groupinstall "Development Tools"
sudo dnf install ncurses-devel bison flex openssl-devel elfutils-libelf-devel
```

### Build Process Overview
```bash
# Download source
wget https://kernel.org/pub/linux/kernel/v5.x/linux-5.15.0.tar.xz
tar -xf linux-5.15.0.tar.xz
cd linux-5.15.0

# Clean build directory
make mrproper

# Configure kernel
make menuconfig          # TUI config (recommended)
make xconfig             # GUI config (X11)
make nconfig             # Ncurses config

# Start from existing config
cp /boot/config-$(uname -r) .config
make olddefconfig

# Compile kernel (adjust J for CPU cores)
make -j$(nproc)

# Compile modules
make -j$(nproc) modules

# Install modules
sudo make modules_install

# Install kernel
sudo make install

# Update bootloader
sudo update-grub         # Debian/Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # RHEL/CentOS

# Reboot
sudo reboot
```

### Kernel Configuration Tips
```bash
# Save current config
make localmodconfig      # Only modules in use
make localyesconfig      # Convert modules to built-in

# Load custom config
scripts/config --enable CONFIG_FEATURE
scripts/config --disable CONFIG_FEATURE

# Diff current config
make olddefconfig && scripts/diffconfig
```

---

## 14.9 Kernel Tuning

### Swap Configuration
```bash
# Check current swappiness
cat /proc/sys/vm/swappiness

# Reduce swappiness (keep RAM usage longer)
sudo sysctl vm.swappiness=10

# Make permanent
echo "vm.swappiness = 10" | sudo tee -a /etc/sysctl.conf
```

### File Descriptor Limits
```bash
# View current limits
ulimit -n

# View soft and hard limits
ulimit -Sn
ulimit -Hn

# Set session limit
ulimit -n 65536

# Set system-wide (per-user)
sudo nano /etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536
```

### I/O Scheduler
```bash
# View available schedulers for disk
cat /sys/block/sda/queue/scheduler

# Change scheduler
echo "deadline" | sudo tee /sys/block/sda/queue/scheduler

# Available schedulers
# - none (NVMe default)
# - mq-deadline
# - bfq
# - kyber
```

---

## 14.10 Debugging Tools

### Performance Monitoring
```bash
# CPU profiling
sudo perf top

# Record performance data
sudo perf record -g -p $(pidof application)

# Analyze recorded data
sudo perf report

# Trace system calls
sudo strace -p $(pidof application)

# Trace with details
sudo strace -f -e trace=open,read,write -p $(pidof application)
```

### Kernel Debugging
```bash
# Enable debug messages
echo 8 > /proc/sys/kernel/printk

# List loaded modules with info
sudo lsmod | sort

# Check module dependencies
lsmod | column -t
modinfo module_name

# View module usage
sudo lsof | grep module_name
```

---

## 14.11 Magic SysRq Key

### What is SysRq?
A debugging feature that allows sending commands to the kernel regardless of system state.

```bash
# Check if SysRq is enabled
cat /proc/sys/kernel/sysrq

# Enable SysRq
sudo sysctl -w kernel.sysrq=1

# Permanently enable
echo "kernel.sysrq = 1" | sudo tee -a /etc/sysctl.conf

# Common SysRq commands (Alt+SysRq+KEY):
# h  - Show help
# s  - Sync disks
# e  - Terminate all processes (SIGTERM)
# i  - Kill all processes (SIGKILL)
# u  - Remount all filesystems read-only
# b  - Reboot immediately
# o  - Shut off system
# t  - Show process list
# w  - Show blocked processes
# p  - Show CPU registers
# l  - Show stack traces
# m  - Show memory info
# 1-9 - Set console log level
```

---

## Practice Exercises

### Exercise 1: Kernel Information
1. Find your kernel version
2. List all loaded modules
3. Check CPU and memory info from /proc
4. View kernel boot messages

### Exercise 2: Module Management
1. List all loaded modules and count them
2. Find information about the ext4 module
3. Load the ppdev module
4. Unload the ppdev module
5. Check module dependencies

### Exercise 3: Kernel Parameters
1. View all kernel parameters related to network
2. Change swappiness to 10 temporarily
3. Make it permanent in /etc/sysctl.conf
4. Create a custom sysctl.d config file
5. Verify changes persist after reboot

### Exercise 4: Blacklisting
1. Check currently blacklisted modules
2. Create a blacklist entry for a module
3. Update initramfs
4. Verify module is not loaded after reboot

### Exercise 5: Debugging
1. Use dmesg to find USB device messages
2. Use strace to monitor a simple command
3. View live kernel logs with journalctl
4. Check available I/O schedulers

## Key Takeaways
- **Kernel**: Core OS component managing hardware and processes
- **Modules**: Loadable kernel extensions, managed with modprobe/lsmod
- **/proc and /sys**: Virtual filesystems for system information
- **sysctl**: Runtime kernel parameter configuration
- **dmesg/journalctl**: Kernel message and debugging logs
- **Blacklisting**: Prevent modules from loading
- **Compilation**: Custom kernel builds for specific needs
- **Magic SysRq**: Emergency kernel commands

## Next Steps
- Study the boot process and init system
- Learn about system initialization (systemd)
- Explore kernel debugging and crash analysis
- Practice kernel module programming
- Study kernel namespaces and cgroups
