# Chapter 07: Package Management

## Introduction

Package management is one of the most important aspects of Linux system administration. A package manager is a collection of software tools that automates the process of installing, upgrading, configuring, and removing software packages in a consistent manner.

## Why Package Management?

- **Dependency Resolution**: Automatically installs required dependencies
- **Centralized Repository**: Single source for all software
- **Version Control**: Manage and track software versions
- **Easy Updates**: Keep software up to date with single commands
- **Clean Uninstall**: Remove software without leaving residue

## Debian/Ubuntu Package Management (APT/dpkg)

### Update Package Lists

```bash
# Update package index (always run first)
sudo apt update

# Upgrade installed packages
sudo apt upgrade

# Full upgrade (handles dependencies better)
sudo apt full-upgrade
```

### Installing Packages

```bash
# Install a single package
sudo apt install <package-name>

# Install multiple packages
sudo apt install package1 package2 package3

# Install a specific version
sudo apt install package=1.2.3

# Search for a package
apt search <keyword>

# Show package details
apt show <package-name>
```

### Removing Packages

```bash
# Remove a package (keep config files)
sudo apt remove <package-name>

# Remove package with config files
sudo apt purge <package-name>

# Remove unused dependencies
sudo apt autoremove

# Clean up downloaded packages
sudo apt autoclean
sudo apt clean
```

### Listing Packages

```bash
# List installed packages
apt list --installed

# List available packages
apt list

# List upgradable packages
apt list --upgradable
```

### Adding Repositories (PPA)

```bash
# Add a PPA repository
sudo add-apt-repository ppa:username/ppa-name

# Update after adding PPA
sudo apt update

# Remove a PPA
sudo add-apt-repository --remove ppa:username/ppa-name
```

### dpkg - Low-Level Package Manager

```bash
# Install a .deb file
dpkg -i package.deb

# Remove a package
dpkg -r package-name

# List installed packages
dpkg -l

# Check package status
dpkg -s package-name

# Fix broken packages
dpkg --configure -a
```

## RHEL/CentOS/Fedora Package Management (DNF/YUM/rpm)

### DNF (Fedora, RHEL 8+, CentOS 8+)

```bash
# Update package cache
dnf check-update

# Update all packages
sudo dnf update

# Install a package
sudo dnf install <package-name>

# Search for packages
dnf search <keyword>

# Get package info
dnf info <package-name>

# Remove a package
sudo dnf remove <package-name>

# List installed packages
dnf list installed

# List available packages
dnf list available

# Show package dependencies
dnf repoquery --whatrequires package-name
```

### YUM (RHEL/CentOS 7)

```bash
# Update package cache
sudo yum check-update

# Update packages
sudo yum update

# Install a package
sudo yum install <package-name>

# Search for packages
yum search <keyword>

# Remove a package
sudo yum remove <package-name>

# List installed packages
yum list installed

# Show package info
yum info <package-name>
```

### rpm - Low-Level Package Manager

```bash
# Install an .rpm file
rpm -ivh package.rpm

# Upgrade an .rpm file
rpm -Uvh package.rpm

# Remove a package
rpm -e package-name

# List all installed packages
rpm -qa

# Check if a package is installed
rpm -q package-name

# Verify installed package
rpm -V package-name

# Check package dependencies
rpm -qpR package.rpm
```

## Arch Linux Package Management (Pacman)

### Installing Packages

```bash
# Synchronize package databases and update
sudo pacman -Syu

# Install a package
sudo pacman -S <package-name>

# Install multiple packages
sudo pacman -S package1 package2

# Search for packages
pacman -Ss <keyword>

# Show package details
pacman -Si <package-name>

# List installed packages
pacman -Q

# Remove a package
sudo pacman -R <package-name>

# Remove with dependencies
sudo pacman -Rs <package-name>

# Remove and orphan dependencies
sudo pacman -Rns <package-name>
```

### Querying Packages

```bash
# Search installed packages
pacman -Qs <keyword>

# List orphaned packages
pacman -Qdt

# Remove orphans
sudo pacman -Rns $(pacman -Qtdq)

# Clean package cache
sudo pacman -Sc
```

## Snap and Flatpak (Universal Packages)

### Snap Packages

```bash
# Install a snap package
sudo snap install <package-name>

# List installed snaps
snap list

# Update snaps
sudo snap refresh

# Remove a snap
sudo snap remove <package-name>

# Search for snaps
snap find <keyword>
```

### Flatpak Packages

```bash
# Install a flatpak app
flatpak install <remote> <app-id>

# List installed flatpaks
flatpak list

# Update flatpaks
flatpak update

# Remove a flatpak
flatpak uninstall <app-id>

# Run a flatpak app
flatpak run <app-id>

# Search for flatpaks
flatpak search <keyword>
```

## Package Management Best Practices

1. **Always update first**: Run update before installing new packages
2. **Review changes**: Check what will be installed/removed before confirming
3. **Keep system updated**: Regularly apply security updates
4. **Use official repos**: Prefer official repositories over third-party
5. **Pin versions**: For production, pin critical package versions
6. **Backup before major upgrades**: Especially before distro upgrades
7. **Clean regularly**: Remove orphaned and cached packages
8. **Document custom repos**: Keep track of added third-party repositories

## Common Package Management Tasks

### Finding Which Package Owns a File

```bash
# Debian/Ubuntu
dpkg -S /path/to/file

# RHEL/CentOS
rpm -qf /path/to/file

# Arch
pacman -Qo /path/to/file
```

### Checking Package Files

```bash
# List files in an installed package
dpkg -L package-name        # Debian
rpm -ql package-name        # RHEL
pacman -Ql package-name     # Arch
```

### Reinstalling a Package

```bash
# Debian/Ubuntu
sudo apt install --reinstall <package-name>

# RHEL/CentOS
sudo dnf reinstall <package-name>

# Arch
sudo pacman -S <package-name>
```

## Practice Exercises

### Basic Exercises

1. Update your system's package lists and upgrade all packages
2. Install a text editor (nano or vim) using your distribution's package manager
3. Install the following utilities: curl, wget, htop, and tree
4. Search for packages related to "git" and install it
5. Check which version of a package is installed on your system
6. List all installed packages and count them
7. Remove a package you installed and then clean up unused dependencies

### Intermediate Exercises

1. Install a package and then find which files it installed
2. Install a package from a .deb or .rpm file (download one first)
3. Add a third-party repository (like NodeSource or Docker) and install a package from it
4. Use snap or flatpak to install an application not in your native repo
5. Find which package provides a specific binary (e.g., `which python3`)
6. Reinstall a package that you accidentally removed
7. Compare `apt` vs `apt-get` commands on Ubuntu

### Advanced Exercises

1. Download a package without installing it: `apt download` or `yumdownloader`
2. Extract files from a .deb/.rpm package without installing
3. Set up package version pinning to prevent a package from upgrading
4. Create a local repository from downloaded packages
5. Use `apt-mark` to hold a package at its current version
6. Write a script that checks for and lists all outdated packages
7. Compare the same software package across different distributions (sizes, dependencies, versions)

## Common Commands Quick Reference

| Task | Debian/Ubuntu | RHEL/CentOS | Arch |
|------|--------------|-------------|------|
| Update repos | apt update | dnf check-update | pacman -Sy |
| Install | apt install pkg | dnf install pkg | pacman -S pkg |
| Remove | apt remove pkg | dnf remove pkg | pacman -R pkg |
| Search | apt search kw | dnf search kw | pacman -Ss kw |
| Info | apt show pkg | dnf info pkg | pacman -Si pkg |
| List installed | dpkg -l | rpm -qa | pacman -Q |
| File owner | dpkg -S file | rpm -qf file | pacman -Qo file |
| Clean | apt autoremove | dnf autoremove | pacman -Rns |

## Troubleshooting

### Broken Packages (Debian/Ubuntu)

```bash
# Fix broken installations
sudo apt --fix-broken install

# Clean and reconfigure
sudo dpkg --configure -a
sudo apt-get update
sudo apt-get dist-upgrade
```

### Broken Dependencies (RHEL/CentOS)

```bash
# Rebuild RPM database
sudo rpm --rebuilddb

# Clean cache
sudo dnf clean all
sudo dnf makecache
```

### GPG Key Issues

```bash
# Update keys
sudo apt-key update
sudo dnf check-update
```

## Resources

- [Debian Package Management](https://www.debian.org/doc/debian-policy/ch-pkgrelationstoc)
- [Arch Wiki - Pacman](https://wiki.archlinux.org/title/Pacman)
- [Fedora DNF Guide](https://docs.fedoraproject.org/en-US/quick-docs/dnf/)
- [Ubuntu Package Management](https://help.ubuntu.com/community/AptGet/Howto)
