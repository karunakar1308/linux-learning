# Chapter 2: File System & Navigation

> **Difficulty**: Beginner | **Estimated Time**: 2-3 days

---

## 2.1 Linux Directory Structure

The Linux filesystem follows a hierarchical tree structure starting from the root directory `/`.

### Key Directories

| Directory | Purpose | Examples |
|-----------|---------|----------|
| `/` | Root directory | Top of the hierarchy |
| `/home` | User home directories | `/home/john`, `/home/alice` |
| `/root` | Root user's home | Admin user's home |
| `/etc` | System configuration files | `/etc/passwd`, `/etc/hostname` |
| `/var` | Variable data (logs, cache) | `/var/log`, `/var/cache` |
| `/tmp` | Temporary files | Cleared on reboot |
| `/usr` | User programs and data | `/usr/bin`, `/usr/local` |
| `/bin` | Essential binaries | `ls`, `cp`, `cat` |
| `/sbin` | System binaries | `fsck`, `reboot` |
| `/lib` | Shared libraries | System libraries |
| `/dev` | Device files | `/dev/sda`, `/dev/null` |
| `/proc` | Process information | Virtual filesystem |
| `/sys` | System information | Virtual filesystem |
| `/mnt` | Mount point for filesystems | Temporary mounts |
| `/media` | Removable media | USB drives, CDs |
| `/opt` | Optional software | Third-party apps |
| `/boot` | Boot loader files | Kernel, initrd |

### Directory Tree Example
```
/
├── bin/
├── boot/
├── dev/
├── etc/
│   ├── passwd
│   ├── hostname
│   └── network/
├── home/
│   ├── user1/
│   └── user2/
├── root/
├── tmp/
├── usr/
│   ├── bin/
│   ├── lib/
│   └── local/
└── var/
    ├── log/
    └── cache/
```

---

## 2.2 File and Directory Operations

### Creating Files and Directories

```bash
# Create an empty file
touch filename.txt

# Create multiple files
touch file1.txt file2.txt file3.txt

# Create a directory
mkdir mydir

# Create nested directories (parent directories too)
mkdir -p path/to/nested/directory

# Create multiple directories
mkdir dir1 dir2 dir3
```

### Listing Files

```bash
# List files
ls

# List with details (long format)
ls -l

# List all files including hidden
ls -a

# List all with details
ls -la

# List with human-readable sizes
ls -lh

# List sorted by modification time
ls -lt

# List sorted by size
ls -lS

# List recursively (show subdirectories)
ls -R

# List only directories
ls -d */
```

### Copying Files

```bash
# Copy file to destination
cp source.txt destination.txt

# Copy to directory
cp file.txt /path/to/directory/

# Copy multiple files to directory
cp file1.txt file2.txt /destination/

# Copy directory recursively
cp -r sourcedir/ destdir/

# Copy preserving attributes (permissions, timestamps)
cp -p file.txt backup.txt

# Copy with verbose output
cp -v file.txt copy.txt

# Interactive copy (prompt before overwrite)
cp -i file.txt existing.txt
```

### Moving/Renaming Files

```bash
# Rename a file
mv oldname.txt newname.txt

# Move file to directory
mv file.txt /path/to/directory/

# Move multiple files
mv file1.txt file2.txt /destination/

# Move directory
mv olddir/ newdir/

# Interactive move (prompt before overwrite)
mv -i file.txt /destination/
```

### Deleting Files

```bash
# Remove a file
rm file.txt

# Remove multiple files
rm file1.txt file2.txt file3.txt

# Interactive removal (prompt before each)
rm -i file.txt

# Remove directory and its contents
rm -r directory/

# Force removal without prompts
rm -f file.txt

# Remove empty directory
rmdir emptydir/

# Verbose output
rm -v file.txt
```

⚠️ **WARNING**: `rm -rf /` will delete everything! Be extremely careful with `rm -rf`.

---

## 2.3 Viewing File Contents

```bash
# Display entire file
cat filename.txt

# Display with line numbers
cat -n filename.txt

# View file page by page (forward only)
more filename.txt

# View file with navigation (forward & backward)
less filename.txt

# Display first 10 lines
head filename.txt

# Display first N lines
head -n 20 filename.txt

# Display last 10 lines
tail filename.txt

# Display last N lines
tail -n 50 filename.txt

# Follow file in real-time (useful for logs)
tail -f /var/log/syslog
```

---

## 2.4 File Searching

### Finding Files by Name

```bash
# Find file by name
find /path -name "filename.txt"

# Find case-insensitive
find /path -iname "FileName.TXT"

# Find all .txt files
find /path -name "*.txt"

# Find in current directory
find . -name "*.log"

# Find only directories
find /path -type d -name "dirname"

# Find only files
find /path -type f -name "*.conf"

# Find and execute command
find /path -name "*.tmp" -exec rm {} \;
```

### Using locate (faster but uses database)

```bash
# Update locate database
sudo updatedb

# Find file by name
locate filename.txt

# Case-insensitive search
locate -i FileName

# Limit results
locate -n 10 filename
```

### Finding by Content

```bash
# Search for text in file
grep "searchtext" filename.txt

# Case-insensitive search
grep -i "SearchText" filename.txt

# Search recursively in directory
grep -r "searchtext" /path/

# Search with line numbers
grep -n "searchtext" filename.txt

# Search and show only filenames
grep -l "searchtext" *.txt

# Invert match (lines NOT containing text)
grep -v "excludetext" filename.txt
```

---

## 2.5 File Permissions

### Understanding Permissions

Linux files have three permission types for three categories:

**Permission Types:**
- `r` (read) = 4
- `w` (write) = 2
- `x` (execute) = 1

**Categories:**
- Owner (u)
- Group (g)
- Others (o)

### Reading Permissions

```bash
ls -l filename.txt
# Output: -rw-r--r-- 1 user group 1234 Mar 18 14:00 filename.txt
#         |  |  |
#         |  |  └─ Others: read only
#         |  └──── Group: read only  
#         └─────── Owner: read + write
```

### Changing Permissions

```bash
# Add execute permission for owner
chmod u+x script.sh

# Remove write permission for group
chmod g-w file.txt

# Add read permission for others
chmod o+r file.txt

# Set exact permissions (owner=rwx, group=rx, others=r)
chmod 754 script.sh

# Make file executable for everyone
chmod +x script.sh

# Recursive permission change
chmod -R 755 directory/
```

### Changing Ownership

```bash
# Change owner
sudo chown newowner file.txt

# Change owner and group
sudo chown newowner:newgroup file.txt

# Change only group
sudo chgrp newgroup file.txt

# Recursive ownership change
sudo chown -R user:group directory/
```

---

## 2.6 Links (Symbolic and Hard)

### Hard Links
- Multiple names for the same file
- Share the same inode
- Cannot link directories
- Cannot cross filesystems

```bash
# Create hard link
ln original.txt hardlink.txt

# Check inodes (should be same)
ls -i original.txt hardlink.txt
```

### Symbolic (Soft) Links
- Pointer to another file
- Different inode
- Can link directories
- Can cross filesystems
- Breaks if original is deleted

```bash
# Create symbolic link
ln -s /path/to/original.txt symlink.txt

# Create symbolic link for directory
ln -s /path/to/directory symlinkdir

# List with link target shown
ls -l symlink.txt
```

---

## 2.7 Disk Usage

```bash
# Check disk space
df -h

# Check directory size
du -sh /path/to/directory

# Check sizes of all items in directory
du -h --max-depth=1 /path/

# Check disk usage for current directory
du -sh *

# Find largest files
du -ah /path | sort -rh | head -20
```

---

## 2.8 File Compression and Archives

### tar (Tape Archive)

```bash
# Create tar archive
tar -cvf archive.tar /path/to/directory

# Extract tar archive
tar -xvf archive.tar

# Create compressed tar.gz
tar -czvf archive.tar.gz /path/to/directory

# Extract tar.gz
tar -xzvf archive.tar.gz

# Create compressed tar.bz2
tar -cjvf archive.tar.bz2 /path/to/directory

# Extract tar.bz2
tar -xjvf archive.tar.bz2

# List contents without extracting
tar -tvf archive.tar
```

### gzip & gunzip

```bash
# Compress file
gzip file.txt  # Creates file.txt.gz

# Decompress
gunzip file.txt.gz

# Keep original while compressing
gzip -k file.txt
```

### zip & unzip

```bash
# Create zip archive
zip -r archive.zip directory/

# Extract zip
unzip archive.zip

# Extract to specific directory
unzip archive.zip -d /path/to/destination
```

---

## 2.9 Practice Exercises

```bash
# Exercise 1: Create directory structure
mkdir -p ~/practice/project/{src,docs,tests}
cd ~/practice/project
touch README.md
touch src/main.sh
touch docs/guide.md

# Exercise 2: File operations
cp README.md README.backup
mv README.backup backups/
ls -lR

# Exercise 3: Permissions
chmod 755 src/main.sh
ls -l src/main.sh

# Exercise 4: Find files
find ~/practice -name "*.md"
find ~/practice -type d

# Exercise 5: Archive
tar -czvf project-backup.tar.gz ~/practice/project
ls -lh project-backup.tar.gz
```

---

## 2.10 Quiz

1. What is the root directory in Linux?
2. Where are user home directories typically located?
3. What does `ls -la` show?
4. How do you recursively copy a directory?
5. What's the difference between `rm` and `rmdir`?
6. What does permission `755` mean?
7. What's the difference between hard and symbolic links?
8. How do you compress a directory using tar and gzip?
9. What command shows disk space usage?
10. How do you search for files by name?

---

## 2.11 Next Steps

After completing this chapter, you should be able to:
- [ ] Navigate the Linux directory structure
- [ ] Create, copy, move, and delete files and directories
- [ ] View and search file contents
- [ ] Understand and modify file permissions
- [ ] Create links between files
- [ ] Compress and archive files

**Next**: Move to [Chapter 3: Text Processing & Editors](03-text-processing-editors.md)
