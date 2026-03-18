# Chapter 06: File Permissions & Ownership

## 6.1 Understanding File Permissions

### Permission Basics
Every file and directory has three types of permissions:
- **Read (r)**: View file contents or list directory
- **Write (w)**: Modify file or create/delete files in directory
- **Execute (x)**: Run file as program or enter directory

### Permission Structure
```bash
-rw-r--r-- 1 user group 1024 Jan 01 12:00 file.txt
│ │││ │││ │ ││  └─────┘ ││
│ │││ │││ │ ││    │     └─ Timestamp
│ │││ │││ │ ││    └─────── File size
│ │││ │││ │ └────────────── Group name
│ │││ │││ │ ─────────────── Owner name
│ │││ │││ └──────────────── Links
│ │││ │││ ────────────────── Other permissions
│ │││ ┗━━━ ────────────────── Group permissions
│ ┗━━━ ──────────────────── Owner permissions
└─────────────────────────── File type
```

### Permission Values
```bash
r (read)    = 4
w (write)   = 2
x (execute) = 1

Owner:   rwx = 7
Group:   r-x = 5
Others:  r-- = 4

chmod 754 file.txt
```

## 6.2 Viewing Permissions

### ls Command
```bash
# List with permissions
ls -l

# Long format with human-readable sizes
ls -lh

# Include hidden files
ls -la

# Recursive listing
ls -lR

# Sort by modified time
ls -lt

# Reverse order
ls -ltr
```

### stat Command
```bash
# Detailed file information
stat file.txt

# Show octal permissions
stat -c '%a %n' file.txt

# Show only permissions
stat -c '%a' file.txt
```

## 6.3 Changing Permissions (chmod)

### Numeric Method
```bash
chmod 644 file.txt      # rw-r--r--
chmod 755 script.sh     # rwxr-xr-x
chmod 700 secret.txt    # rwx------
chmod 777 file.txt      # rwxrwxrwx
chmod 600 file.txt      # rw-------
```

### Symbolic Method
```bash
# Add permissions
chmod u+x script.sh     # Owner: add execute
chmod g+r file.txt      # Group: add read
chmod o-r file.txt      # Others: remove read
chmod a+r file.txt      # All: add read

# Remove permissions
chmod u-w file.txt      # Owner: remove write
chmod go-x file.txt     # Group and others: remove execute

# Set exactly
chmod u=rwx,g=rx,o= file.txt
```

### Recursive Changes
```bash
# Change all files in directory
chmod -R 755 directory/

# Change only files
find directory -type f -exec chmod 644 {} \;

# Change only directories
find directory -type d -exec chmod 755 {} \;
```

## 6.4 Changing Ownership (chown)

### Basic chown Usage
```bash
# Change owner
chown newuser file.txt

# Change owner and group
chown newuser:newgroup file.txt

# Change only group
chgrp newgroup file.txt

# Using numeric IDs
chown 1000:1000 file.txt
```

### Recursive Changes
```bash
# Change all files and directories
chown -R newuser:newgroup directory/

# Change owner only
chown -R newuser directory/

# Change group only
chown -R :newgroup directory/
```

### Examples
```bash
# Transfer file to another user
sudo chown john file.txt

# Make root the owner
sudo chown root:root file.txt

# Change entire home directory
sudo chown -R user:user /home/user
```

## 6.5 Special Permissions

### Set-User-ID (SUID) - 4000
```bash
# SUID bit allows file to run as owner
chmod u+s program.sh    # or chmod 4755 program.sh

# Examples
ls -l /usr/bin/sudo     # rws--S--S

# Dangerous! Use carefully
# File runs with owner's permissions regardless of who executes
```

### Set-Group-ID (SGID) - 2000
```bash
# SGID on file: runs as group
chmod g+s program.sh    # or chmod 2755 program.sh

# SGID on directory: files inherit group ownership
chmod g+s directory/

# Example with directory
mkdir shared_dir
chmod g+s shared_dir
```

### Sticky Bit - 1000
```bash
# Sticky bit on directory: only owner can delete files
chmod +t directory/     # or chmod 1777 directory/

# Common use: /tmp
ls -ld /tmp             # drwxrwxrwt

# Without sticky bit, anyone can delete anyone's files
# With sticky bit, only owner, group, or root can delete
```

### Combined Examples
```bash
# SUID + RWX for all
chmod 4777 program.sh

# SGID + RWX for all
chmod 2777 directory/

# Sticky bit + RWX for all
chmod 1777 shared_dir/

# SUID + SGID + Sticky
chmod 7777 file.txt
```

## 6.6 umask

### Understanding umask
```bash
# Default umask (shows what permissions are REMOVED)
umask

# Typical: 0022 (remove write for group and others)
# File default: 666 - 022 = 644 (rw-r--r--)
# Directory default: 777 - 022 = 755 (rwxr-xr-x)
```

### Changing umask
```bash
# Set for current session
umask 0077      # Remove all permissions except owner

# Verify
umask           # Shows 0077

# Make permanent (add to ~/.bashrc)
echo "umask 0077" >> ~/.bashrc
```

### Common umask Values
```bash
0022 : Files 644, Directories 755 (default)
0077 : Files 600, Directories 700 (secure)
0002 : Files 664, Directories 775 (collaborative)
0007 : Files 660, Directories 770 (group only)
```

## 6.7 Default Ownership and Groups

### User and Group Info
```bash
# Current user
whoami

# All users
cat /etc/passwd | cut -d: -f1

# All groups
cat /etc/group | cut -d: -f1

# User's groups
groups

# User's groups in numeric
id
```

### Create Users and Groups
```bash
# Create user
sudo useradd username

# Create user with home directory
sudo useradd -m username

# Create user with specific UID
sudo useradd -u 1500 username

# Create group
sudo groupadd groupname

# Add user to group
sudo usermod -aG groupname username

# Remove user from group
sudo gpasswd -d username groupname
```

## 6.8 Access Control Lists (ACL)

### ACL Basics
```bash
# Install acl (if not present)
sudo apt install acl

# View ACL
getfacl file.txt

# Set ACL for user
setfacl -m u:username:rwx file.txt

# Set ACL for group
setfacl -m g:groupname:rx file.txt

# Remove specific ACL
setfacl -x u:username file.txt

# Remove all ACLs
setfacl -b file.txt

# Recursive ACL
setfacl -R -m u:username:rwx directory/
```

## 6.9 Practical Examples

### Web Server Files
```bash
# Website root
chmod 755 /var/www/html
chmod 644 /var/www/html/*.html
chmod 755 /var/www/html/*.cgi

# Logs
chmod 750 /var/log/apache2
chmod 640 /var/log/apache2/*.log
```

### Configuration Files
```bash
# System configs (root only)
chmod 600 /etc/shadow
chmod 644 /etc/passwd

# SSH keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 600 ~/.ssh/authorized_keys
```

### Development Environment
```bash
# Source code
chmod 755 source/
chmod 644 source/*.c
chmod 644 source/*.h

# Executables
chmod 755 bin/program
chmod 755 scripts/*.sh
```

### Shared Directory
```bash
# Create shared directory
mkdir /home/shared
chmod 770 /home/shared
chown :developers /home/shared

# Enable SGID
chmod g+s /home/shared
```

## 6.10 Security Best Practices

### Principle of Least Privilege
```bash
# Give only necessary permissions
# Not: chmod 777 everything

# Files: 644 (rw-r--r--)
# Directories: 755 (rwxr-xr-x)
# Executables: 755 (rwxr-xr-x)
# Sensitive files: 600 (rw-------)
```

### Owner Restrictions
```bash
# Prevent unauthorized access
chmod 700 /home/user

# Limit script modification
chmod 755 /usr/local/bin/script.sh

# Protect database files
chmod 600 /var/db/database.db
```

### SUID/SGID Dangers
```bash
# Find SUID files
find / -perm -4000

# Find SGID files
find / -perm -2000

# Find world-writable files
find / -perm -002

# Remove unnecessary SUID
sudo chmod u-s /usr/bin/example
```

## Practice Exercises

### Exercise 1: Permission Management
1. Create file `test.txt` with content
2. Change permissions to 600
3. Try to read as different user (fails)
4. Change to 644 (succeeds)
5. Make it executable: 755

### Exercise 2: Ownership
1. Create directory `shared`
2. Create group `team`
3. Add users to `team`
4. Change ownership with SGID
5. Verify inheritance

### Exercise 3: Security Audit
1. Find all world-writable files
2. Find all SUID files
3. Check home directory permissions
4. Review /etc permissions

### Exercise 4: ACL Usage
1. Create file with standard permissions
2. Add read permission for specific user via ACL
3. Verify with `getfacl`
4. Remove ACL entry

### Exercise 5: Web Server Setup
1. Create /var/www/app directory structure
2. Set appropriate permissions (755 dirs, 644 files)
3. Set correct ownership (www-data)
4. Create logs directory with 750
5. Verify security

## Key Takeaways

- **chmod**: Change file permissions
- **chown**: Change file owner and group
- **Special permissions**: SUID (4000), SGID (2000), Sticky (1000)
- **umask**: Default permission mask
- **ACL**: Fine-grained access control
- **Security**: Principle of least privilege
- **Best practices**: 644 files, 755 directories

## Next Steps

- Learn SELinux for advanced access control
- Study audit logging and monitoring
- Explore file capabilities (getcap, setcap)
- Practice security hardening
- Learn about AppArmor profiles
