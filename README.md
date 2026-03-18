# Linux Learning Path - From Beginner to Advanced

> A comprehensive, hands-on guide to mastering Linux - from absolute basics to advanced system administration.

---

## Table of Contents

| Level | Chapter | Topics | Difficulty |
|-------|---------|--------|------------|
| **Beginner** | [01-linux-fundamentals](chapters/01-linux-fundamentals.md) | What is Linux, Distributions, Installation, Shell Basics | Easy |
| **Beginner** | [02-file-system-navigation](chapters/02-file-system-navigation.md) | Directory Structure, File Operations, Navigation | Easy |
| **Beginner** | [03-text-processing-editors](chapters/03-text-processing-editors.md) | Vim, Nano, Cat, Grep, Sed, Awk Basics | Easy-Medium |
| **Beginner** | [04-user-group-management](chapters/04-user-group-management.md) | Users, Groups, Permissions, sudo | Easy-Medium |
| **Intermediate** | [05-process-management](chapters/05-process-management.md) | Processes, Jobs, Signals, Monitoring | Medium |
| **Intermediate** | [06-networking-linux](chapters/06-networking-linux.md) | TCP/IP, Network Commands, Firewall | Medium |
| **Intermediate** | [07-shell-scripting](chapters/07-shell-scripting.md) | Variables, Loops, Functions, Debugging | Medium |
| **Intermediate** | [08-package-management](chapters/08-package-management.md) | apt, yum, dnf, pacman, Snap, Flatpak | Medium |
| **Intermediate** | [09-disk-management](chapters/09-disk-management.md) | Partitions, LVM, Filesystems, RAID | Medium-Hard |
| **Advanced** | [10-security-permissions](chapters/10-security-permissions.md) | ACL, SELinux, AppArmor, Firewalls | Hard |
| **Advanced** | [11-system-administration](chapters/11-system-administration.md) | Boot Process, Services, Logs, Backup | Hard |
| **Advanced** | [12-containers-virtualization](chapters/12-containers-virtualization.md) | Docker, KVM, LXC, Namespaces | Hard |
| **Practice** | [labs/practice-exercises.md](labs/practice-exercises.md) | Hands-on Labs & Exercises for All Topics | All Levels |
| **Practice** | [labs/projects.md](labs/projects.md) | Real-world Projects & Scenarios | All Levels |
| **Reference** | [reference/cheat-sheets.md](reference/cheat-sheets.md) | Quick Command Reference | All Levels |
| **Reference** | [reference/glossary.md](reference/glossary.md) | Linux Terminology & Concepts | All Levels |

---

## Course Overview

This repository is designed as a self-paced learning path for anyone who wants to master Linux. Whether you're a developer, system administrator, DevOps engineer, or just starting out, this guide covers everything you need.

### How to Use This Repo

1. **Start from Chapter 1** if you're new to Linux
2. **Use the table of contents** to jump to specific topics
3. **Complete all hands-on labs** after each chapter
4. **Build the projects** to reinforce learning
5. **Reference cheat sheets** when you need quick reminders

### Prerequisites

- Basic computer literacy
- A Linux environment (VM, WSL, or physical machine)
- Terminal/command line access
- Curiosity and patience!

---

## Recommended Distributions for Learning

| Distribution | Best For | Why |
|-------------|----------|------|
| **Ubuntu** | Beginners | Large community, great documentation |
| **Debian** | Stability | Rock-solid, great for learning fundamentals |
| **Fedora** | Cutting-edge | Latest features, good for developers |
| **CentOS/Rocky** | Enterprise | Production-like environment |
| **Arch** | Advanced users | Learn by doing, minimal base |
| **Kali** | Security | Penetration testing & security learning |

---

## Setup Your Learning Environment

### Option 1: Virtual Machine (Recommended for Beginners)
```bash
# Install VirtualBox
sudo apt install virtualbox
# Download Ubuntu ISO from ubuntu.com
# Create a new VM and install Ubuntu
```

### Option 2: WSL2 (Windows Users)
```powershell
# In PowerShell as Admin
wsl --install
wsl --list --online
wsl --install -d Ubuntu
```

### Option 3: Docker Container (Quick Start)
```bash
docker run -it ubuntu:latest bash
```

---

## Learning Path Recommendations

### For Beginners (2-3 months)
- Complete Chapters 1-4
- Do all practice exercises
- Build the "Personal File Server" project

### For Intermediate Users (1-2 months)
- Complete Chapters 5-9
- Do all shell scripting exercises
- Build the "Automated Backup System" project

### For Advanced Users (1-2 months)
- Complete Chapters 10-12
- Build the "Containerized Web Server" project
- Build the "System Monitoring Dashboard" project

---

## Quick Start Commands

| Task | Command |
|------|--------|
| List files | `ls -la` |
| Create directory | `mkdir dirname` |
| Create file | `touch filename` |
| View file content | `cat filename` |
| Edit file | `nano filename` or `vim filename` |
| Copy file | `cp source dest` |
| Move file | `mv source dest` |
| Delete file | `rm filename` |
| Find file | `find /path -name pattern` |
| Check disk space | `df -h` |
| Check memory | `free -h` |
| List processes | `ps aux` |
| Network info | `ip addr` or `ifconfig` |
| Update packages | `sudo apt update && sudo apt upgrade` |

---

## Contributing

Feel free to:
- Report issues or suggest improvements
- Add new chapters or exercises
- Fix typos or improve explanations
- Share your project solutions

## Resources

- [Linux Documentation Project](https://tldp.org/)
- [GNU Manuals](https://www.gnu.org/manual/manual.html)
- [Explain Shell](https://explainshell.com/) - Understand commands
- [OverTheWire Bandit](https://overthewire.org/wargames/bandit/) - Learn Linux through a game
- [Linux Journey](https://linuxjourney.com/) - Interactive tutorials
