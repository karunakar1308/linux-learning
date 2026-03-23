# Chapter 19: Backup and Recovery

## Introduction

Backup and recovery are critical components of any IT infrastructure. This chapter covers backup strategies, tools, and best practices for protecting Linux systems and data. You'll learn about traditional tools like tar and rsync, as well as modern solutions for incremental and encrypted backups.

## Backup Fundamentals

### The 3-2-1 Backup Rule

- **3** copies of your data (1 primary + 2 backups)
- **2** different storage media types
- **1** offsite copy (cloud, remote server)

### Backup Types

| Type | Description | Pros | Cons |
|------|-------------|------|------|
| Full | Complete copy of all data | Fastest restore | Slowest backup, most storage |
| Incremental | Only changed data since last backup | Fast, least storage | Slowest restore, chain dependent |
| Differential | Changed data since last full backup | Faster restore than incremental | More storage than incremental |
| Snapshot | Point-in-time copy | Instant creation | Requires special storage |

### RPO and RTO

- **RPO (Recovery Point Objective)**: Maximum acceptable data loss
  - RPO = 0: No data loss acceptable
  - RPO = 24h: Up to 24 hours of data can be lost
- **RTO (Recovery Time Objective)**: Maximum acceptable downtime
  - RTO = 1 hour: System must be restored within 1 hour
  - RTO = 4 hours: System must be restored within 4 hours

## Using tar for Backups

### Creating Backups with tar

```bash
# Create a compressed backup
tar -czvf backup.tar.gz /path/to/data

# Create backup with date in filename
DATE=$(date +%Y%m%d)
tar -czvf backup_${DATE}.tar.gz /path/to/data

# Create backup excluding certain directories
tar -czvf backup.tar.gz --exclude='/path/to/exclude' \
    --exclude='/var/tmp/*' /path/to/data

# Create a backup to stdout and redirect
sudo tar czf - /etc /home > backup.tar.gz

# Preserve permissions and ownership
sudo tar -czpvf backup.tar.gz /path/to/data
```

### Restoring with tar

```bash
# Restore all files
tar -xzvf backup.tar.gz

# Restore to specific directory
tar -xzvf backup.tar.gz -C /restore/path

# List contents without extracting
tar -tzvf backup.tar.gz

# Extract specific files
tar -xzvf backup.tar.gz path/to/specific/file

# Extract with original permissions
tar -xzvpf backup.tar.gz
```

### Advanced tar Options

```bash
# Use gzip compression (-z)
# Use bzip2 compression (-j) - better compression, slower
# Use xz compression (-J) - best compression, slowest
# Use zstd compression (--zstd) - fast and good compression

tar --zstd -cvf backup.tar.zst /path/to/data

# Split large backups
tar czf - /path/to/data | split -b 1G - backup.tar.gz.

# Create differential backup
rsync -av --delete /source/ /backup/
```

## Using rsync for Backups

### Basic rsync Backup

```bash
# Sync files to backup location
rsync -av /source/ /backup/

# Sync with compression over network
rsync -avz /source/ user@backup-server:/backup/

# Sync with progress display
rsync -av --progress /source/ /backup/

# Sync and delete files that don't exist in source
rsync -av --delete /source/ /backup/

# Exclude specific files/directories
rsync -av --exclude='*.tmp' --exclude='cache/' /source/ /backup/
```

### rsync for Incremental Backups

```bash
# Create backup with hard links for unchanged files
rsync -av --delete --link-dest=/backup/last/ \
    /source/ /backup/current/

# Rotate backups
mv /backup/current /backup/last
rsync -av --delete --link-dest=/backup/last/ \
    /source/ /backup/current/
```

### rsync over SSH

```bash
# Backup to remote server via SSH
rsync -avz -e ssh /local/path/ user@remote:/remote/path/

# Use specific SSH port
rsync -avz -e 'ssh -p 2222' /local/path/ user@remote:/remote/path/

# Backup with bandwidth limit (500KB/s)
rsync -avz --bwlimit=500 /local/path/ user@remote:/remote/path/
```

### rsync Exclude File

```bash
# Create exclude file
cat > /tmp/exclude.txt << 'EOF'
*.tmp
*.log
*.cache
/var/tmp/*
/proc/*
/sys/*
/dev/*
EOF

# Use exclude file
rsync -av --exclude-from=/tmp/exclude.txt /source/ /backup/
```

## Using dd for Disk Imaging

### Creating Disk Images

```bash
# Create exact copy of a partition
sudo dd if=/dev/sda1 of=/backup/partition.img bs=4M status=progress

# Create compressed disk image
sudo dd if=/dev/sda1 bs=4M status=progress | gzip > /backup/partition.img.gz

# Create compressed image with progress
sudo dd if=/dev/sda bs=4M status=progress | gzip > /backup/disk.img.gz

# Clone disk to disk (both must be same or larger size)
sudo dd if=/dev/sda of=/dev/sdb bs=4M status=progress
```

### Restoring Disk Images

```bash
# Restore from uncompressed image
sudo dd if=/backup/partition.img of=/dev/sda1 bs=4M status=progress

# Restore from compressed image
zcat /backup/partition.img.gz | sudo dd of=/dev/sda1 bs=4M status=progress
```

### Important dd Safety Notes

```bash
# ALWAYS verify if= and of= before running dd
# Double-check device names!
# Wrong target device can destroy data instantly

# Get a count of blocks to verify
sudo dd if=/dev/sda1 of=/dev/null bs=4M

# Or use ddrescue for damaged disks
sudo ddrescue /dev/sda /backup/disk.img /backup/disk.log
```

## Using tar with Encryption

### Encrypting Backups with GPG

```bash
# Create encrypted backup
tar czf - /path/to/data | gpg -c > backup.tar.gz.gpg
# Or use symmetric encryption with passphrase

# Use public key encryption
tar czf - /path/to/data | gpg -e -r recipient@email.com > backup.tar.gz.gpg

# Decrypt and extract
gpg -d backup.tar.gz.gpg | tar xzf -

# With passphrase
gpg --passphrase-file /secure/key -d backup.tar.gz.gpg | tar xzf -
```

### Encrypting with OpenSSL

```bash
# Encrypt backup
tar czf - /path/to/data | openssl enc -aes-256-cbc -salt -pbkdf2 \
    -pass pass:YOUR_PASSWORD > backup.tar.gz.enc

# Decrypt backup
openssl enc -d -aes-256-cbc -pbkdf2 -pass pass:YOUR_PASSWORD \
    -in backup.tar.gz.enc | tar xzf -
```

## Backup Strategies

### Full System Backup Script

```bash
#!/bin/bash
# Full system backup script

BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="full_${DATE}.tar.gz"
EXCLUDE_FILE="/tmp/backup_excludes.txt"

# Create exclude list
cat > $EXCLUDE_FILE << 'EOF'
/proc
/sys
/dev
/run
/tmp
/backup
*.log
*.tmp
EOF

# Create backup
echo "Starting backup at $(date)"
sudo tar --exclude-from=$EXCLUDE_FILE \
    -czvf ${BACKUP_DIR}/${BACKUP_FILE} /
echo "Backup completed at $(date)"

# Verify backup
tar tzf ${BACKUP_DIR}/${BACKUP_FILE} > /dev/null && \
    echo "Backup verified successfully" || \
    echo "Backup verification FAILED"

# Cleanup old backups (keep 7 days)
find ${BACKUP_DIR} -name "full_*.tar.gz" -mtime +7 -delete

echo "Cleanup completed at $(date)"
```

### Incremental Backup with rsync

```bash
#!/bin/bash
# Incremental backup script

SOURCE="/data"
TARGET="/backup/current"
LAST="/backup/last"

# Ensure last backup exists
if [ ! -d "$LAST" ]; then
    mkdir -p "$LAST"
fi

# Run incremental backup
rsync -av --delete --link-dest="$LAST" \
    "$SOURCE/" "$TARGET/"

# Rotate backups
rm -rf "$LAST"
mv "$TARGET" "$LAST"
mkdir -p "$TARGET"
```

### Database Backup

```bash
# MySQL/MariaDB backup
mysqldump --all-databases --single-transaction --quick \
    --lock-tables=false > all-databases.sql
gzip all-databases.sql

# MySQL with encryption
mysqldump --all-databases | gpg -c > all-databases.sql.gpg

# PostgreSQL backup
pg_dumpall | gzip > postgres_backup.sql.gz

# PostgreSQL with specific database
pg_dump -U postgres -Fc mydb > mydb.dump

# MongoDB backup
mongodump --out=/backup/mongodb
```

## Recovery Procedures

### Restoring from tar Backup

```bash
# Full restore
tar -xzvf backup.tar.gz -C /

# Restore specific directory
mkdir -p /restore/home
tar -xzvf backup.tar.gz -C /restore home/

# Restore with original permissions
sudo tar -xzvpf backup.tar.gz -C /
```

### Restoring from rsync Backup

```bash
# Reverse rsync (backup to source)
rsync -av /backup/current/ /source/

# Restore with delete option
rsync -av --delete /backup/current/ /source/
```

### Restoring from Disk Image

```bash
# Restore disk image
dd if=/backup/partition.img of=/dev/sda1 bs=4M status=progress

# Restore compressed image
zcat /backup/partition.img.gz | dd of=/dev/sda1 bs=4M status=progress
```

### Recovery Mode

```bash
# Boot into recovery mode (GRUB menu)
# Select "Advanced options" > "Recovery mode"

# Mount filesystem read-only
mount -o remount,ro /

# Filesystem check
fsck -y /dev/sda1

# Reset root password
passwd root

# Repair GRUB
grub-install /dev/sda
update-grub
```

## Modern Backup Tools

### Restic

```bash
# Install restic
# Debian/Ubuntu
sudo apt install restic
# RHEL/CentOS
sudo yum install restic

# Initialize repository
restic init --repo /backup/restic
# Or remote
restic init --repo sftp:user@host:/backup/restic
# Or S3
restic init --repo s3:s3.amazonaws.com/bucket/restic

# Create backup
restic -r /backup/restic backup /data

# List snapshots
restic -r /backup/restic snapshots

# Restore
restic -r /backup/restic restore latest --target /restore

# Prune old snapshots
restic -r /backup/restic forget --keep-last 7 --prune
```

### BorgBackup

```bash
# Install Borg
sudo apt install borgbackup

# Initialize repository
borg init --encryption=repokey /backup/borg-repo

# Create backup
borg create /backup/borg-repo::'{hostname}-{now}' /data

# List archives
borg list /backup/borg-repo

# Extract
borg extract /backup/borg-repo::archive-name

# Prune
borg prune --keep-daily 7 --keep-weekly 4 --keep-monthly 6 \
    /backup/borg-repo
```

## Backup Verification

### Automated Backup Testing

```bash
#!/bin/bash
# Backup verification script

BACKUP_FILE="/backup/latest.tar.gz"
TEST_DIR=$(mktemp -d)

# Test 1: Check file exists and has content
if [ ! -s "$BACKUP_FILE" ]; then
    echo "ERROR: Backup file empty or missing"
    exit 1
fi

# Test 2: Verify tar integrity
if ! tar tzf "$BACKUP_FILE" > /dev/null 2>&1; then
    echo "ERROR: Backup archive corrupted"
    exit 1
fi

# Test 3: Extract and verify
if ! tar xzf "$BACKUP_FILE" -C "$TEST_DIR"; then
    echo "ERROR: Backup extraction failed"
    exit 1
fi

# Test 4: Check key files exist
if [ ! -d "$TEST_DIR/etc" ]; then
    echo "ERROR: Critical directory missing"
    exit 1
fi

rm -rf "$TEST_DIR"
echo "Backup verification passed"
```

## Exercises

### Exercise 1: Basic tar Backup

1. Create a tar.gz backup of your home directory
2. Verify the backup integrity
3. Extract to a test directory and compare

### Exercise 2: rsync Incremental Backup

1. Create a source directory with files
2. Run rsync to create a backup
3. Modify some files and run rsync again
4. Use --link-dest to create space-efficient backups

### Exercise 3: Full System Backup Script

1. Write a script that backs up /etc, /home, /var/www
2. Exclude temporary and log files
3. Add timestamp to backup filename
4. Schedule with cron

### Exercise 4: Database Backup

1. Install MySQL or PostgreSQL
2. Create a sample database with tables
3. Perform a logical backup
4. Restore to a new database and verify

### Exercise 5: Encrypted Backup

1. Create a tar backup
2. Encrypt it with GPG
3. Decrypt and restore
4. Set up key-based encryption

## Key Takeaways

- **3-2-1 Rule**: Always have 3 copies, 2 media types, 1 offsite
- **RPO/RTO**: Define acceptable data loss and downtime
- **tar**: Universal tool for file-level backups
- **rsync**: Efficient for incremental and remote backups
- **dd**: For disk/partition imaging (use carefully!)
- **Encryption**: Always encrypt backups containing sensitive data
- **Verification**: Always test your backups regularly
- **Documentation**: Document your recovery procedures
- **Automation**: Automate backups, not recovery

## Quick Reference

```bash
# tar backup
tar -czvf backup.tar.gz /path
tar -xzvf backup.tar.gz

# rsync backup
rsync -av --delete /source/ /backup/
rsync -avz /source/ user@host:/backup/

# dd image
dd if=/dev/sda1 of=image.img bs=4M status=progress

# Encrypted backup
tar czf - /data | gpg -e -r email > backup.gpg

# Verify backup
tar tzf backup.tar.gz > /dev/null && echo "OK"

# Restic
restic -r /backup backup /data
restic -r /backup restore latest --target /restore
```

## Troubleshooting

```bash
# tar: Cannot open: Permission denied
# Use sudo or check permissions

# rsync: connection unexpectedly closed
# Check SSH connectivity and firewall rules
ssh -o ServerAliveInterval=60 user@host

# dd: No space left on device
# Check free space before creating image
df -h

# Backup too slow
# Use local storage first, then transfer
# Use compression: gzip, bzip2, or xz
# Consider incremental backups

# Restore fails on different hardware
# May need to adjust fstab, kernel modules
# Consider using system imaging tools for full recovery
```

## Next Steps

- Explore cloud backup solutions (AWS S3, Azure Blob Storage)
- Study backup orchestration tools (Bacula, Amanda)
- Learn about disaster recovery planning
- Implement automated backup monitoring and alerting
