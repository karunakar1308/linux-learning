# Chapter 1: Linux Fundamentals

> **Difficulty**: Beginner | **Estimated Time**: 2-3 days

---

## 1.1 What is Linux?

### Understanding the Kernel vs Distribution

- **Kernel**: The core of the operating system that manages hardware, memory, processes, and system resources.
- **Distribution (Distro)**: A complete OS package built around the Linux kernel, including package managers, desktop environments, and pre-installed software.

### Why Linux?

- Open source and free
- Highly customizable
- Excellent for servers and cloud
- Strong security model
- Huge community support

---

## 1.2 Popular Linux Distributions

### Debian-Based (APT package manager)
| Distribution | Use Case | Base |
|-------------|----------|------|
| **Ubuntu** | Desktop, Server, Cloud | Debian |
| **Linux Mint** | Desktop (user-friendly) | Ubuntu |
| **Kali Linux** | Security & Penetration Testing | Debian |
| **Pop!_OS** | Desktop, Gaming, Development | Ubuntu |

### Red Hat-Based (YUM/DNF package manager)
| Distribution | Use Case | Base |
|-------------|----------|------|
| **Fedora** | Cutting-edge Desktop | Independent |
| **RHEL** | Enterprise Server | Independent |
| **CentOS Stream** | Development/Testing | RHEL |
| **Rocky Linux** | Enterprise (RHEL clone) | RHEL |
| **AlmaLinux** | Enterprise (RHEL clone) | RHEL |

### Independent
| Distribution | Use Case |
|-------------|----------|
| **Arch Linux** | Advanced users, minimal install |
| **Gentoo** | Custom builds, optimization |
| **Alpine Linux** | Containers, lightweight |
| **openSUSE** | Enterprise, desktop |

---

## 1.3 Installing Linux

### Method 1: Virtual Machine (Recommended for Learning)

#### Using VirtualBox:
```bash
# Install VirtualBox on Ubuntu/Debian
sudo apt update
sudo apt install virtualbox virtualbox-ext-pack

# Download Ubuntu ISO:
# Visit: https://ubuntu.com/download/desktop

# Create a new VM:
# 1. Open VirtualBox
# 2. Click "New"
# 3. Name it, select Linux type, Ubuntu (64-bit)
# 4. Allocate 4GB RAM, 25GB disk
# 5. Select Ubuntu ISO as boot disk
# 6. Start and install
```

#### Using VMware:
```bash
# Download VMware Workstation Player (free for personal use)
# Follow similar VM creation steps as VirtualBox
```

### Method 2: WSL2 (Windows Subsystem for Linux)

```powershell
# Open PowerShell as Administrator
wsl --install

# List available distros
wsl --list --online

# Install specific distro
wsl --install -d Ubuntu-22.04

# Set default WSL version
wsl --set-default-version 2

# Access Linux from Windows
wsl
```

### Method 3: Dual Boot (Advanced)
- Create a bootable USB with Rufus or Etcher
- Backup your data
- Partition your drive
- Install Linux alongside Windows/macOS

---

## 1.4 The Shell and Terminal

### What is the Shell?
- A command-line interface (CLI) to interact with the OS
- Translates user commands into kernel actions
- Multiple shells available: bash, zsh, fish, etc.

### Default Shell: Bash
Bash (Bourne Again Shell) is the most common default shell.

```bash
# Check your current shell
echo $SHELL

# List available shells
cat /etc/shells

# Change your shell
chsh -s /bin/zsh
```

### Opening the Terminal
| Environment | How to Open |
|-------------|------------|
| Ubuntu Desktop | Ctrl+Alt+T |
| macOS | Cmd+Space, type "Terminal" |
| Windows WSL | Open "Ubuntu" app or `wsl` in cmd |
| Any Linux | Ctrl+Alt+T or Search "Terminal" |

---

## 1.5 Basic Shell Commands

### Essential First Commands

```bash
# Print current working directory
pwd

# List files in current directory
ls

# List files with details (permissions, owner, size, date)
ls -l

# List ALL files including hidden ones (those starting with .)
ls -la

# List files in human-readable format (KB, MB, GB)
ls -lh

# Change directory
cd /path/to/directory

# Go to home directory
cd ~

# Go to parent directory
cd ..

# Go back to previous directory
cd -

# Clear the terminal screen
clear
# or press Ctrl+L

# Print current date and time
date

# Display system information (kernel, hostname)
uname -a

# Show currently logged-in user
whoami

# Show who is logged in
who
```

### Getting Help

```bash
# Manual pages (man pages) - press 'q' to quit
man ls
man -k keyword  # search for keyword

# Built-in help for shell commands
help cd

# Command info (more detailed than man)
info ls

# What is this command?
which ls
whereis ls
type ls
```

---

## 1.6 Understanding Command Structure

```
command [options] [arguments]
  |        |           |
  |        |           └─ Files, directories, or values
  |        └─ Flags that modify behavior (-l, -h, etc.)
  └─ The program or utility to run
```

### Examples:
```bash
ls -la /home         # ls command, options -la, argument /home
cp -r source dest    # cp command, option -r, arguments source dest
```

---

## 1.7 File Paths

### Absolute vs Relative Paths

| Type | Example | Description |
|------|---------|-------------|
| **Absolute** | `/home/user/documents` | Starts from root `/` |
| **Relative** | `documents/file.txt` | Relative to current directory |
| **Home shortcut** | `~/documents` | Relative to your home directory |

### Special Path Notations
```bash
.    # Current directory
..   # Parent directory
~    # Home directory
/    # Root directory
```n
## 1.8 History and Navigation Shortcuts

```bash
# View command history
history

# Run command #123 from history
!123

# Re-run the last command with sudo
sudo !!

# Re-run last command starting with 'apt'
!apt

# Search history (Ctrl+R in terminal)
# Press Ctrl+R, then type to search, Enter to run
```

### Keyboard Shortcuts
| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Cancel/kill current command |
| `Ctrl+Z` | Suspend current command |
| `Ctrl+D` | Exit shell/terminal |
| `Ctrl+L` | Clear screen |
| `Ctrl+A` | Go to beginning of line |
| `Ctrl+E` | Go to end of line |
| `Ctrl+U` | Delete from cursor to start |
| `Ctrl+K` | Delete from cursor to end |
| `Tab` | Auto-complete commands/filenames |

---

## 1.9 Practice Exercises

Complete ALL of these exercises in your terminal:

```bash
# Exercise 1: Explore your system
pwd          # Where am I?
ls -la       # What files are here?
whoami       # Who am I logged in as?
date         # What time is it?
uname -a     # What OS/kernel am I running?

# Exercise 2: Navigation practice
cd /         # Go to root
cd home      # Go to home directory
cd ..        # Go back one level
cd /etc      # Go to /etc using absolute path
pwd          # Verify where you are

# Exercise 3: Help system
man ls       # Read ls manual, press q to exit
help cd      # Get help for cd command
which python # Where is python installed?

# Exercise 4: History exploration
history | head -20
!1           # Re-run your first command

# Exercise 5: Clear and restart
clear        # Clear screen
Ctrl+L       # Clear screen shortcut
```

---

## 1.10 Quiz Yourself

1. What's the difference between `ls` and `ls -la`?
2. How do you go to the root directory from anywhere?
3. What does `~` represent in a path?
4. How do you view the manual for a command?
5. What does `pwd` stand for and what does it do?
6. How do you cancel a running command?
7. What's the purpose of the `Tab` key in the shell?
8. How do you check your current shell?
9. What's the difference between an absolute and relative path?
10. Which command shows your system's kernel version?

---

## 1.11 Next Steps

After completing this chapter, you should be comfortable with:
- [ ] Navigating the file system
- [ ] Running basic commands
- [ ] Using the man pages
- [ ] Understanding command structure
- [ ] Using keyboard shortcuts

**Next**: Move to [Chapter 2: File System & Navigation](02-file-system-navigation.md) for detailed file operations and directory management.
