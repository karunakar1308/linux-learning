# Practice Exercises

## Beginner Level

### Exercise 1: File Navigation
Create a directory structure and practice navigation:

```bash
# Create a nested directory structure
mkdir -p ~/linux-practice/{documents,scripts,logs,backup}

# Navigate into the directory
cd ~/linux-practice

# List contents
cd documents
touch report1.txt report2.txt notes.md
ls -lh

# Go back to parent
cd ..

# Go to home using tilde
cd ~
pwd

# Clean up
rm -rf ~/linux-practice
```

### Exercise 2: File Operations
Practice creating, copying, moving, and deleting files:

```bash
# Create some files
cd ~
mkdir file-ops-practice
cd file-ops-practice
touch file1.txt file2.txt file3.txt

# Copy a file
cp file1.txt file1_backup.txt

# Move/rename a file
mv file2.txt renamed_file2.txt

# Delete a file
rm file3.txt

# View file contents
echo "Hello Linux!" > file1.txt
cat file1.txt

# Copy entire directory
cp -r ~/file-ops-practice ~/backup-copy

# Clean up
rm -rf ~/file-ops-practice ~/backup-copy
```

### Exercise 3: File Searching
Practice finding files and content:

```bash
# Create test files
mkdir ~/search-practice && cd ~/search-practice
echo "important data" > important.txt
echo "notes for review" > notes.txt
echo "log entry 1" > app.log
echo "log entry 2" > app2.log
mkdir subfolder
echo "hidden file" > subfolder/secret.txt

# Find files by name
find . -name "*.txt"
find . -name "*log*"

# Search file content
grep -r "important" .
grep -l "log" .

# Find recently modified files
find . -mmin -10

# Clean up
rm -rf ~/search-practice
```

## Intermediate Level

### Exercise 4: Text Processing
Practice grep, sed, awk, and pipes:

```bash
# Create sample data
mkdir ~/text-processing && cd ~/text-processing
cat > data.csv << 'EOF'
name,age,city,salary
Alice,30,New York,75000
Bob,25,Los Angeles,65000
Charlie,35,Chicago,85000
Diana,28,Houston,70000
EOF

# Extract specific columns
cut -d',' -f1,3 data.csv

# Filter rows
grep "New York" data.csv
grep -v "Bob" data.csv

# Count lines/words
wc -l data.csv
wc -w data.csv

# Replace text with sed
sed 's/New York/NYC/g' data.csv

# Sort data
sort -t',' -k2 -n data.csv

# Clean up
rm -rf ~/text-processing
```

### Exercise 5: Process Management
Practice managing processes:

```bash
# List all processes
ps aux

# Find a specific process
ps aux | grep bash
pgrep bash

# Check system resources
top -b -n 1
free -h
df -h

# Run a background process
sleep 300 &
jobs
fg %1
Ctrl+Z
bg %1

# Kill a process (don't actually kill system processes)
sleep 300 &
PID=$!
echo "Process ID: $PID"
kill $PID
```

### Exercise 6: Permissions Practice
Practice file permissions:

```bash
mkdir ~/perm-practice && cd ~/perm-practice
touch file1.txt file2.txt file3.txt

# View current permissions
ls -l

# Change permissions (numeric)
chmod 644 file1.txt
ls -l file1.txt

# Change permissions (symbolic)
chmod u+x file2.txt
chmod g-w file3.txt

# Change ownership (needs sudo)
sudo chown $USER:$USER file1.txt

# Set special permissions
chmod +s file1.txt  # SUID bit (for learning only)
chmod 1777 /tmp    # Sticky bit

# Clean up
rm -rf ~/perm-practice
```

### Exercise 7: Shell Scripting Basics
Create your first scripts:

```bash
# Create a simple script
mkdir ~/scripts && cd ~/scripts
cat > hello.sh << 'EOF'
#!/bin/bash
echo "Hello, $(whoami)!"
echo "Current date: $(date)"
echo "Hostname: $(hostname)"
EOF

# Make it executable and run
chmod +x hello.sh
./hello.sh

# Create a script with variables
cat > greet.sh << 'EOF'
#!/bin/bash
NAME="Linux Learner"
echo "Welcome, $NAME!"
echo "Home directory: $HOME"
EOF

chmod +x greet.sh
./greet.sh

# Script with conditionals
cat > check-file.sh << 'EOF'
#!/bin/bash
if [ -f "/etc/hostname" ]; then
    echo "Host file exists"
else
    echo "Host file not found"
fi
EOF

chmod +x check-file.sh
./check-file.sh

# Clean up (keep scripts for later use)
```

### Exercise 8: Disk and Memory
Practice disk and memory management:

```bash
# Check disk space
df -h
df -i  # inodes

# Check memory usage
free -h
cat /proc/meminfo

# Check CPU info
cat /proc/cpuinfo | grep "model name" | head -1
lscpu

# Find large files
find /home -size +100M 2>/dev/null
du -sh /* 2>/dev/null | sort -h | tail -10

# Monitor disk I/O
iostat -x 1 3  # if installed
```

## Advanced Level

### Exercise 9: Network Diagnostics
Practice network troubleshooting:

```bash
# Check network interfaces
ip addr show
ip route show

# Check DNS resolution
cat /etc/resolv.conf
nslookup google.com
dig google.com

# Test connectivity
ping -c 4 google.com

# Check open ports
ss -tulpn  # modern alternative to netstat
netstat -tulpn  # if netstat is installed

# Check active connections
ss -s

# Trace network path
tracepath google.com  # or traceroute
```

### Exercise 10: Automation with Cron
Practice scheduled tasks:

```bash
# View current crontab
crontab -l

# Create a simple cron job (will open editor)
crontab -e

# Example entries (add these for learning):
# * * * * * command    - every minute
# 0 * * * * command    - every hour
# 0 0 * * * command    - every day at midnight
# 0 0 * * 0 command    - every Sunday at midnight

# Example: backup script
echo "0 2 * * * tar -czf ~/backup-\$(date +\%Y\%m\%d).tar.gz ~/Documents" | crontab -
crontab -l

# Remove the job after learning
crontab -r
```

### Exercise 11: Log Analysis
Practice reading and analyzing logs:

```bash
# View system logs
sudo tail -f /var/log/syslog    # Ubuntu/Debian
sudo tail -f /var/log/messages  # RHEL/CentOS

# Search in logs
sudo grep -i "error" /var/log/syslog
sudo grep -i "failed" /var/log/auth.log

# View boot messages
dmesg | less
journalctl -b

# View specific service logs
sudo journalctl -u ssh -n 50

# Check for failed logins
sudo grep "Failed password" /var/log/auth.log | tail -20
```

### Exercise 12: Backup and Restore
Practice backup strategies:

```bash
# Create a test directory to backup
mkdir -p ~/data-project
echo "Important data" > ~/data-project/file1.txt
echo "More data" > ~/data-project/file2.txt
mkdir ~/data-project/subfolder
echo "Subfolder data" > ~/data-project/subfolder/file3.txt

# Create a tar backup
tar -czvf ~/data-backup.tar.gz ~/data-project

# Verify the backup
tar -tzvf ~/data-backup.tar.gz

# Create incremental backup
tar --listed-incremental=~/backup.snar -czvf ~/incremental-backup.tar.gz ~/data-project

# Restore from backup
mkdir ~/restore-test
tar -xzvf ~/data-backup.tar.gz -C ~/restore-test

# Verify restoration
ls -la ~/restore-test/data-project/

# Clean up
rm -rf ~/data-project ~/restore-test ~/data-backup.tar.gz ~/incremental-backup.tar.gz
```

## Self-Assessment Checklist

After completing these exercises, you should be able to:

- [ ] Navigate the filesystem confidently
- [ ] Create, copy, move, and delete files/directories
- [ ] Search for files and content within files
- [ ] Use text processing tools (grep, sed, awk, cut)
- [ ] Monitor and manage processes
- [ ] Set and modify file permissions
- [ ] Write basic shell scripts
- [ ] Check disk and memory usage
- [ ] Diagnose network connectivity
- [ ] Schedule tasks with cron
- [ ] Read and analyze system logs
- [ ] Create and restore backups

## Challenge: Build a Personal File Server

As a capstone project, combine what you've learned:

1. Create a directory structure for organizing files
2. Set appropriate permissions for different user groups
3. Write a shell script to automate file organization
4. Schedule a daily backup using cron
5. Write a monitoring script to check disk usage
6. Create a log of all file operations

This exercise combines navigation, permissions, scripting, cron, and logging into one practical project.
