# Chapter 1: Linux Basics

## 1.1 What is Linux?

Linux is a free, open-source operating system based on the Unix kernel. It powers most of the world's servers, cloud infrastructure, and embedded devices.

### Key Characteristics:
- **Open Source**: Source code is freely available and modifiable
- **Multi-user**: Multiple users can access the system simultaneously
- **Multi-tasking**: Multiple processes can run concurrently
- **Portable**: Runs on various hardware architectures
- **Secure**: Strong permission and user management system

## 1.2 Linux Architecture

```
+-------------------------+
|     Applications        |
+-------------------------+
|     Shell (Bash, Zsh)   |
+-------------------------+
|     System Libraries    |
+-------------------------+
|     Linux Kernel        |
+-------------------------+
|     Hardware            |
+-------------------------+
```

### Components:
1. **Kernel**: Core of the OS, manages hardware, memory, and processes
2. **Shell**: Command-line interface (CLI) for user interaction
3. **System Libraries**: Functions used by applications
4. **System Utilities**: Basic tools for system management

## 1.3 Common Linux Distributions

| Distribution | Use Case |
|-------------|----------|
| Ubuntu | Desktop, servers, cloud |
| CentOS/RHEL | Enterprise servers |
| Debian | Stable servers |
| Fedora | Cutting-edge features |
| Arch Linux | Advanced users, customization |
| Alpine Linux | Containers, minimal systems |

## 1.4 First Commands

### Basic Navigation
```bash
pwd          # Print Working Directory
cd /path     # Change Directory
cd ..        # Go to parent directory
cd ~         # Go to home directory
ls           # List files
ls -l        # Detailed listing
ls -a        # Include hidden files
clear        # Clear terminal screen
```

### Basic Information
```bash
whoami       # Current username
uname -a     # System information
hostname     # System hostname
date         # Current date and time
cal          # Display calendar
```

### Manual Pages
```bash
man command      # Manual for a command
man -k keyword   # Search manual pages
whatis command   # Brief description
```

## 1.5 Help Commands

```bash
command --help   # Quick help for a command
info command     # Detailed info pages
apropos keyword  # Search for commands by keyword
```

## Practice Exercises

### Exercise 1.1: Explore Your System
1. Open a terminal
2. Run `pwd` to see your current location
3. Run `ls -la` to list all files including hidden
4. Run `uname -a` to see kernel info

### Exercise 1.2: Navigation
1. Navigate to `/home`
2. Navigate to `/var/log`
3. Return to your home directory
4. List contents with `ls -lh` (human-readable sizes)

### Exercise 1.3: Using Man Pages
1. Read the manual for `ls`: `man ls`
2. Search for a command with `apropos date`
3. Check the brief description: `whatis pwd`

## Quick Reference: Essential Commands

| Command | Description |
|---------|-------------|
| `pwd` | Show current directory |
| `ls` | List directory contents |
| `cd` | Change directory |
| `mkdir` | Create directory |
| `rmdir` | Remove empty directory |
| `touch` | Create empty file |
| `cat` | Display file contents |
| `man` | Manual pages |
| `clear` | Clear screen |
| `exit` | Exit shell/session |
