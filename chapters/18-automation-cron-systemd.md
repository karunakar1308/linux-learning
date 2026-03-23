# Chapter 18: Automation with Cron and Systemd Timers

## Introduction

Automation is essential for system administration and DevOps. This chapter covers two primary Linux automation tools: **cron** (the traditional scheduler) and **systemd timers** (the modern alternative). You'll learn when to use each, how to create scheduled tasks, and best practices for production automation.

## Cron Fundamentals

### What is Cron?

Cron is a time-based job scheduler that runs tasks at specified intervals. It's managed by the `cron` daemon (crond).

### Crontab Syntax

The crontab format consists of 6 fields:

```
* * * * * command
| | | | |
| | | | +---- Day of week (0-7, Sunday=0 or 7)
| | | +------ Month (1-12)
| | +-------- Day of month (1-31)
| +---------- Hour (0-23)
+------------ Minute (0-59)
```

### Special Characters

| Character | Meaning | Example |
|-----------|---------|--------|
| `*` | Any value | `*` = every minute |
| `,` | List of values | `1,3,5` = minutes 1, 3, 5 |
| `-` | Range | `1-5` = 1 through 5 |
| `/` | Step values | `*/10` = every 10 units |

### Common Cron Patterns

```
# Every minute
* * * * *

# Every 5 minutes
*/5 * * * *

# Every hour at minute 30
30 * * * *

# Every day at 3:00 AM
0 3 * * *

# Every Sunday at 2:30 AM
30 2 * * 0

# Every Monday through Friday at 9:00 AM
0 9 * * 1-5

# Every 1st of the month at midnight
0 0 1 * *

# Every weekday at 8:00 AM and 5:00 PM
0 8,17 * * 1-5
```

### Managing User Cron Jobs

```bash
# Edit your own crontab
crontab -e

# List your cron jobs
crontab -l

# Remove your crontab
crontab -r

# Edit another user's crontab (as root)
crontab -u username -e

# List another user's crontab
crontab -u username -l
```

### System-Wide Cron

```bash
# View system crontab
cat /etc/crontab

# System cron directories
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
ls -la /etc/cron.weekly/
ls -la /etc/cron.monthly/

# Add a system-wide cron job
echo "0 3 * * * root /usr/local/bin/backup.sh" | sudo tee /etc/cron.d/backup
```

### Cron Environment

```bash
# Default cron environment (limited!)
# PATH, HOME, SHELL, LOGNAME

# To see cron's PATH
crontab -l

# Set custom environment in crontab
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=admin@example.com
SHELL=/bin/bash
0 * * * * /path/to/script.sh
```

## Cron Best Practices

### Logging and Debugging

```bash
# Enable cron logging (Debian/Ubuntu)
sudo systemctl restart rsyslog
sudo grep CRON /var/log/syslog

# Enable cron logging (RHEL/CentOS)
sudo systemctl restart rsyslog
sudo grep CRON /var/log/cron

# Log output from cron jobs to a file
0 * * * * /path/to/script.sh >> /var/log/myjob.log 2>&1

# Send email on failure
0 * * * * /path/to/script.sh || echo "Job failed" | mail -s "Alert" admin@example.com
```

### Writing Cron-Friendly Scripts

```bash
#!/bin/bash
# Always use absolute paths
SCRIPT_DIR="/opt/scripts"
LOG_FILE="/var/log/myjob.log"

# Source environment if needed
source /etc/profile

# Use flock for preventing overlapping runs
(
    flock -x 200 || exit 1
    # Your script logic here
    echo "$(date): Job running"
) 200>/var/lock/myjob.lock

# Log with timestamp
echo "$(date): $(basename $0) completed" >> $LOG_FILE
```

### Cron Restrictions

```bash
# Check /etc/cron.allow (if exists, only listed users can use cron)
cat /etc/cron.allow

# Check /etc/cron.deny (users listed cannot use cron)
cat /etc/cron.deny

# Check crontab permissions
ls -la /var/spool/cron/
```

## Systemd Timers

### What are Systemd Timers?

Systemd timers are a modern alternative to cron, offering:
- Better logging via journalctl
- Precise timing (down to seconds)
- Dependency management
- Calendar events and monotonic timers
- On-demand activation

### Timer Units Structure

A systemd timer setup requires two files:
1. `.timer` unit - defines when to run
2. `.service` unit - defines what to run

### Creating a Simple Timer

```bash
# Create the service unit
sudo tee /etc/systemd/system/backup.service << 'EOF'
[Unit]
Description=Daily Backup Service
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
User=root
Group=root
EOF

# Create the timer unit
sudo tee /etc/systemd/system/backup.timer << 'EOF'
[Unit]
Description=Run backup daily at 3 AM

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
EOF
```

### Systemd Calendar Syntax

```
# Format: DayOfWeek Year-Month-Day Hour:Minute:Second

# Every minute
*-*-* *:*:*

# Every hour at minute 30
*-*-* *:30:00

# Every day at 3:00 AM
*-*-* 03:00:00

# Every Sunday at 2:30 AM
Sun *-*-* 02:30:00

# Every Monday through Friday at 9:00 AM
Mon..Fri *-*-* 09:00:00

# Every 1st of the month at midnight
*-*-01 00:00:00

# Every weekday at 8:00 AM and 5:00 PM
Mon..Fri *-*-* 08:00:00,17:00:00

# Every 15 minutes
*-*-* *:0/15:00

# Every Monday at 9 AM
Mon *-*-* 09:00:00
```

### Managing Timer Units

```bash
# Reload systemd to pick up changes
sudo systemctl daemon-reload

# List all timers
systemctl list-timers

# List timers with all details
systemctl list-timers --all

# Check specific timer status
systemctl status backup.timer

# Start a timer immediately
sudo systemctl start backup.timer

# Stop a timer
sudo systemctl stop backup.timer

# Enable timer to start at boot
sudo systemctl enable backup.timer

# Disable timer
sudo systemctl disable backup.timer

# Check last/next run
systemctl list-timers --all | grep backup
```

### Timer Unit Options

```ini
[Timer]
# Calendar event
OnCalendar=*-*-* 03:00:00

# Monotonic timers (relative to boot)
OnBootSec=15min        # Run 15 minutes after boot
OnUnitActiveSec=1h     # Run 1 hour after last activation
OnUnitInactiveSec=30min # Run 30 minutes after unit stopped

# Accuracy
AccuracySec=1min       # Allow up to 1 minute delay for batching

# Persistent
Persistent=true        # Run missed jobs after system restart

# Randomized delay
RandomizedDelaySec=300 # Add 0-5 min random delay

# Unit to activate
Unit=backup.service    # Default: same name as timer
```

### Systemd Timer Logging

```bash
# Check timer logs
journalctl -u backup.timer
journalctl -u backup.service

# Follow logs in real-time
journalctl -f -u backup.service

# Check timer triggers
journalctl --field=MESSAGE | grep backup

# View detailed timer info
systemctl cat backup.timer
systemctl cat backup.service
systemctl show backup.timer
```

## Cron vs Systemd Timers Comparison

| Feature | Cron | Systemd Timer |
|---------|------|---------------|
| Granularity | Minutes | Seconds |
| Logging | mail/syslog | journalctl |
| Dependencies | None | Full systemd deps |
| Timezones | System timezone | Supports timezone |
| Precision | Minute-level | Second-level |
| Resource limits | None | cgroups support |
| Boot behavior | Runs as daemon | Can delay until ready |
| Missed runs | Skipped | Persistent option |
| Complex schedules | Limited | Full calendar syntax |
| On-demand | Manual | Socket activation |

### When to Use Each

**Use Cron when:**
- Simple, minute-precision schedules
- Legacy system compatibility
- User-level automation
- Quick ad-hoc scheduling

**Use Systemd Timers when:**
- Second-level precision needed
- Need proper logging/monitoring
- Service dependencies matter
- Need resource limits (cgroups)
- Modern systemd-based systems
- Need to handle missed executions

## Advanced Automation Patterns

### Chained Jobs

```bash
# Cron: Sequential with && or dependency
0 1 * * * /path/to/backup.sh && /path/to/verify.sh

# Systemd: Dependency chain
# backup.service
[Unit]
After=network.target

# verify.timer
[Timer]
OnCalendar=*-*-* 02:00:00

# verify.service
[Unit]
After=backup.service
Requires=backup.service
```

### Conditional Execution

```bash
#!/bin/bash
# Run only on weekdays
if [ $(date +%u) -le 5 ]; then
    /path/to/weekday-task.sh
fi

# Run only on specific day of month
if [ $(date +%d) -eq 1 ]; then
    /path/to/monthly-task.sh
fi
```

### Parallel Execution Control

```bash
#!/bin/bash
# Prevent overlapping executions
LOCKFILE=/var/lock/myjob.lock
exec 200>$LOCKFILE
flock -n 200 || { echo "Already running"; exit 1; }

# Your job
/path/to/task.sh

# Lock released automatically when script exits
```

### Alerting on Failure

```bash
#!/bin/bash
set -e

# Your task
/path/to/task.sh || {
    curl -X POST -H "Content-Type: application/json" \
        -d '{"text":"Task failed: '$(basename $0)'"}' \
        https://hooks.slack.com/services/XXX
    exit 1
}

# Success notification
curl -X POST -H "Content-Type: application/json" \
    -d '{"text":"Task succeeded: '$(basename $0)'"}' \
    https://hooks.slack.com/services/XXX
```

## Exercises

### Exercise 1: Basic Cron Job

1. Create a cron job that runs every 5 minutes and writes to a log file
2. Verify it's running correctly
3. Check the logs

### Exercise 2: Daily Backup Cron

1. Create a backup script that archives a directory
2. Schedule it to run daily at 2 AM
3. Add email notification on failure
4. Test the script manually first

### Exercise 3: Systemd Timer Setup

1. Create a simple service that writes to a file
2. Create a timer unit for it
3. Enable and start the timer
4. Verify it's running with `systemctl list-timers`

### Exercise 4: Migration from Cron to Systemd

1. Identify an existing cron job on your system
2. Convert it to a systemd timer
3. Disable the cron job
4. Verify the timer works correctly

### Exercise 5: Monitoring Automation

1. Create a script that monitors disk usage
2. Schedule it with systemd timer
3. Set up email/Slack alert on high disk usage
4. Add proper logging

## Key Takeaways

- **Cron Syntax**: 5 fields - minute, hour, day, month, weekday
- **Cron Locations**: User (`crontab -e`), System (`/etc/crontab`, `/etc/cron.d/`)
- **Cron Limitations**: Minute precision, limited logging, no dependencies
- **Systemd Timer**: Two-file setup (.timer + .service)
- **Systemd Advantages**: Second precision, journalctl logging, dependencies, cgroups
- **Persistent Timers**: `Persistent=true` runs missed jobs
- **Logging**: Always log output: `>> /var/log/job.log 2>&1`
- **Locking**: Use `flock` to prevent overlapping runs
- **Testing**: Always test scripts manually before scheduling

## Quick Reference

```bash
# Crontab quick commands
crontab -e    # Edit
crontab -l    # List
crontab -r    # Remove

# Systemd timer commands
systemctl list-timers           # List all timers
systemctl status backup.timer   # Check status
systemctl daemon-reload         # Reload after changes
systemctl enable --now timer    # Enable and start
journalctl -u service           # Check logs
```

## Troubleshooting

```bash
# Cron job not running
# 1. Check if cron is running
systemctl status cron      # Debian/Ubuntu
systemctl status crond     # RHEL/CentOS

# 2. Check cron logs
sudo grep CRON /var/log/syslog    # Debian/Ubuntu
sudo grep CRON /var/log/cron      # RHEL/CentOS

# 3. Check crontab syntax (use https://crontab.guru/)
# 4. Ensure script is executable: chmod +x script.sh
# 5. Use absolute paths in scripts
# 6. Check file permissions

# Systemd timer not running
# 1. Check timer status
systemctl status backup.timer

# 2. Check timer was triggered
journalctl -u backup.timer

# 3. Check service output
journalctl -u backup.service

# 4. Verify timer syntax
systemd-analyze verify /etc/systemd/system/backup.timer

# 5. Check if timer is enabled
systemctl is-enabled backup.timer

# Debug timer trigger time
systemd-analyze calendar "*-*-* 03:00:00"
```

## Next Steps

- Learn about CI/CD pipeline automation (GitHub Actions, Jenkins)
- Study infrastructure automation with Ansible
- Explore event-driven automation with tools like Webhooks
- Deep dive into systemd socket activation for on-demand services
