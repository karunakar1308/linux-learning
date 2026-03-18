# Chapter 9: System Administration

## Overview
System administration involves managing servers, users, system services, and maintaining system health and performance. This chapter covers essential Linux system administration tasks.

## User and Group Management

### Understanding Users
- **Root User**: System administrator with UID 0, unrestricted access
- **Regular Users**: UID >= 1000, restricted permissions
- **System Users**: UID < 1000, for running services

### Creating Users
```bash
# Create a new user
sudo useradd -m -s /bin/bash username

# Create user with specific UID
sudo useradd -m -u 1100 -s /bin/bash username

# Create user with specific home directory
sudo useradd -m -d /home/custom/path username
```

### Modifying Users
```bash
# Change user password
sudo passwd username

# Change user's login shell
sudo usermod -s /bin/zsh username

# Add user to group
sudo usermod -a -G groupname username

# Change user's home directory
sudo usermod -d /new/home username
```

### Deleting Users
```bash
# Remove user (keep home directory)
sudo userdel username

# Remove user and home directory
sudo userdel -r username

# Remove user and associated files
sudo userdel -r -f username
```

### Group Management
```bash
# Create a group
sudo groupadd groupname

# Add user to group
sudo usermod -a -G groupname username

# Create group with specific GID
sudo groupadd -g 1500 groupname

# Delete a group
sudo groupdel groupname
```

### Viewing User and Group Information
```bash
# View all users
cat /etc/passwd

# View user details
id username

# View all groups
cat /etc/group

# View groups for specific user
groups username

# View shadow file (password hashes)
sudo cat /etc/shadow
```

## User Permissions and Ownership

### File Permissions
```bash
# Change file permissions (symbolic)
chmod u+rwx,g+rx,o-rwx filename

# Change file permissions (octal)
chmod 750 filename

# Apply permissions recursively
chmod -R 755 directory/
```

### File Ownership
```bash
# Change owner
sudo chown username filename

# Change owner and group
sudo chown username:groupname filename

# Change recursively
sudo chown -R username:groupname directory/
```

### Special Permissions
```bash
# Set SUID (Run as owner)
chmod u+s filename

# Set SGID (Run as group)
chmod g+s filename

# Set sticky bit (Only owner can delete)
chmod o+t directory/
```

## System Services Management

### Systemd Services
```bash
# Start a service
sudo systemctl start servicename

# Stop a service
sudo systemctl stop servicename

# Restart a service
sudo systemctl restart servicename

# Reload service configuration
sudo systemctl reload servicename

# Enable service at boot
sudo systemctl enable servicename

# Disable service at boot
sudo systemctl disable servicename

# Check service status
sudo systemctl status servicename

# List all running services
sudo systemctl list-units --type=service --state=running
```

### Viewing Service Information
```bash
# Show service details
sudo systemctl show servicename

# View service configuration
sudo systemctl cat servicename

# Check service dependencies
sudo systemctl list-dependencies servicename
```

## System Monitoring

### CPU and Memory Usage
```bash
# Display system resources
top

# Interactive process viewer
htop

# Show memory information
free -h

# Display CPU information
lscpu

# Show system uptime
uptime
```

### Disk Usage
```bash
# Show disk space usage
df -h

# Show directory size
du -sh directory/

# List top 10 largest files
du -ah | sort -rh | head -10

# Show inode usage
df -i
```

### System Logs
```bash
# View system logs
sudo journalctl

# View recent logs
sudo journalctl -n 50

# View logs for specific service
sudo journalctl -u servicename

# View logs in real-time
sudo journalctl -f

# View logs since specific time
sudo journalctl --since "2024-01-01 00:00:00"
```

## Scheduling Tasks

### Cron Jobs
```bash
# Edit crontab
crontab -e

# View crontab
crontab -l

# Remove crontab
crontab -r

# View system crontab
sudo cat /etc/crontab
```

### Cron Syntax
```
# Format: minute hour day month weekday command
# Example:
0 2 * * 0 /path/to/backup.sh    # Run Sunday at 2 AM
30 * * * * /path/to/script.sh   # Run every hour at 30 min
0 0 1 * * /path/to/monthly.sh   # Run first day of month
*/5 * * * * /path/to/check.sh   # Run every 5 minutes
```

### At Command (One-time scheduling)
```bash
# Schedule task for specific time
at 14:30

# Schedule task for tomorrow
at 14:30 tomorrow

# Schedule task for specific date
at 14:30 2024-12-25

# List scheduled tasks
atq

# Remove scheduled task
atrm job_id
```

## Backup and Recovery

### Basic Backup Methods
```bash
# Backup using tar
tar -czf backup.tar.gz /path/to/backup/

# Backup specific files
tar -czf backup.tar.gz /home /etc

# List archive contents
tar -tzf backup.tar.gz

# Extract archive
tar -xzf backup.tar.gz

# Extract to specific directory
tar -xzf backup.tar.gz -C /restore/path/
```

### Using rsync
```bash
# Sync directories
rsync -av source/ destination/

# Sync with deletion
rsync -av --delete source/ destination/

# Remote sync (pull)
rsync -av user@server:/remote/path /local/path

# Remote sync (push)
rsync -av /local/path user@server:/remote/path/
```

## System Updates and Patches

### Update Management
```bash
# Update package lists
sudo apt update

# Upgrade packages
sudo apt upgrade

# Full upgrade (Ubuntu/Debian)
sudo apt full-upgrade

# Upgrade packages (RHEL/CentOS)
sudo yum update

# Check available updates
sudo apt list --upgradable
```

### Security Updates
```bash
# Install security updates only
sudo unattended-upgrade

# Enable automatic security updates
sudo apt install unattended-upgrades

# Check update history
sudo grep "upgrade" /var/log/apt/history.log
```

## Firewall Management

### UFW (Uncomplicated Firewall)
```bash
# Enable firewall
sudo ufw enable

# Disable firewall
sudo ufw disable

# Allow port
sudo ufw allow 22/tcp

# Allow service
sudo ufw allow ssh

# Deny port
sudo ufw deny 8080/tcp

# Delete rule
sudo ufw delete allow 22/tcp

# View rules
sudo ufw show added

# Check status
sudo ufw status
```

### Firewalld (RHEL/CentOS)
```bash
# Start firewalld
sudo systemctl start firewalld

# Enable at boot
sudo systemctl enable firewalld

# Add port
sudo firewall-cmd --permanent --add-port=8080/tcp

# Remove port
sudo firewall-cmd --permanent --remove-port=8080/tcp

# Reload firewall
sudo firewall-cmd --reload
```

## System Logs and Auditing

### Log Locations
```bash
# System logs
/var/log/syslog (Debian/Ubuntu)
/var/log/messages (RHEL/CentOS)

# Authentication logs
/var/log/auth.log (Debian/Ubuntu)
/var/log/secure (RHEL/CentOS)

# Application logs
/var/log/apache2/
/var/log/nginx/
```

### Log Rotation
```bash
# View logrotate configuration
sudo cat /etc/logrotate.conf

# Manually rotate logs
sudo logrotate -f /etc/logrotate.conf

# Check logrotate status
sudo logrotate -d /etc/logrotate.conf
```

## System Information

### Checking System Info
```bash
# Full system information
uname -a

# Kernel version
uname -r

# Operating system
uname -s

# Hostname
hostname

# System uptime
uptime

# CPU cores
nproc

# Total installed memory
free -h
```

## Practice Exercises

1. **User Management**:
   - Create a new user "developer" with home directory
   - Add developer to "sudo" group
   - Create a new group "backend" and add developer to it
   - Set developer's default shell to zsh
   - Delete the developer user and verify

2. **Service Management**:
   - Start and enable SSH service
   - Check status of various running services
   - Stop a non-essential service
   - View systemd service logs

3. **Monitoring**:
   - Use top to identify processes consuming most CPU
   - Use df -h to check disk usage
   - Monitor system logs in real-time with journalctl
   - Use free -h to check memory usage

4. **Backup and Scheduling**:
   - Create a tar backup of /home directory
   - Set up a cron job to run daily backup at 2 AM
   - Test rsync for directory synchronization

5. **Security**:
   - Enable UFW firewall
   - Allow SSH (port 22) through firewall
   - Block a specific port
   - View firewall rules

## Key Takeaways
- User and group management is fundamental to Linux administration
- Services can be easily managed using systemctl
- Regular monitoring helps maintain system health
- Automated backups and scheduling prevent data loss
- Firewalls provide essential security protection
- Understanding logs is crucial for troubleshooting
