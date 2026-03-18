# Linux Learning Projects

Real-world projects to apply and solidify your Linux skills.

---

## Project 1: Personal File Server (Beginner)
**Time: 2-3 hours**

Build a file organization and sharing system.

### Objectives
- Practice directory navigation and creation
- Learn file permissions and user groups
- Write basic shell scripts for automation

### Steps
1. Create a directory structure: `shared/{public,private,projects}`
2. Set different permissions for each folder
3. Write a script to organize downloaded files by extension
4. Create a README documenting your structure

### Skills Practiced
- `mkdir -p`, `chmod`, `chown`
- Shell scripting basics
- Understanding directory hierarchies

---

## Project 2: Automated Backup System (Intermediate)
**Time: 3-4 hours**

Create an automated backup solution using shell scripts and cron.

### Objectives
- Master backup tools (tar, rsync)
- Learn cron for scheduling
- Practice log management

### Steps
1. Write a backup script that archives important directories
2. Compress backups with meaningful timestamps
3. Set up cron to run backups daily
4. Implement log rotation
5. Create a restore script to test backups

### Sample Script Structure
```bash
#!/bin/bash
SOURCE_DIR="/home/$USER/Documents"
BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)

tar -czf "$BACKUP_DIR/backup_$DATE.tar.gz" "$SOURCE_DIR"
```

### Skills Practiced
- `tar`, `rsync`, `gzip`
- `crontab`, `cron` jobs
- Shell scripting with variables
- Log file management

---

## Project 3: System Monitoring Dashboard (Intermediate)
**Time: 4-5 hours**

Build a script that collects and displays system metrics.

### Objectives
- Learn system monitoring commands
- Create informative output
- Practice data formatting

### Steps
1. Collect CPU usage (`top`, `uptime`)
2. Gather memory stats (`free`, `/proc/meminfo`)
3. Check disk space (`df`, `du`)
4. List running processes
5. Network statistics
6. Output everything in a formatted report

### Skills Practiced
- Process monitoring (`ps`, `top`)
- Memory and disk analysis
- Text formatting with `awk`/`printf`
- Reading `/proc` filesystem

---

## Project 4: Log Analysis Tool (Intermediate)
**Time: 3-4 hours**

Create a tool to analyze system and application logs.

### Objectives
- Practice log file parsing
- Use grep, awk, and sed effectively
- Generate security reports

### Steps
1. Parse `/var/log/auth.log` for failed logins
2. Identify the most active hours from logs
3. Find and report on error patterns
4. Create a summary report

### Skills Practiced
- `grep`, `awk`, `sed`, `sort`, `uniq`
- Log file analysis
- Pattern matching
- Report generation

---

## Project 5: Containerized Web Server (Advanced)
**Time: 5-6 hours**

Deploy and manage a web server using Docker.

### Objectives
- Learn container basics
- Practice networking concepts
- Understand deployment workflows

### Steps
1. Install Docker (if not installed)
2. Pull and run an nginx container
3. Configure custom web content
4. Set up port forwarding
5. Create a Dockerfile for a custom setup
6. Manage container lifecycle

### Skills Practiced
- Docker commands (`run`, `ps`, `logs`, `exec`)
- Port mapping and networking
- Volume mounting
- Container orchestration basics

---

## Project 6: User Management Automation (Advanced)
**Time: 4-5 hours**

Automate user account creation and management.

### Objectives
- Master user/group management
- Practice security best practices
- Create reusable automation scripts

### Steps
1. Write a script to create multiple users from a file
2. Assign users to appropriate groups
3. Set up home directories with templates
4. Configure sudo access
5. Implement password policies
6. Create a user audit report

### Skills Practiced
- `useradd`, `usermod`, `groupadd`
- `/etc/passwd`, `/etc/group`
- `sudoers` configuration
- Scripting with arrays and loops

---

## Project 7: DevOps Lab Environment (Advanced)
**Time: 8-10 hours**

Build a complete lab environment for DevOps learning.

### Objectives
- Combine multiple Linux skills
- Practice infrastructure setup
- Learn virtualization concepts

### Steps
1. Set up multiple VMs or containers
2. Configure networking between them
3. Install common DevOps tools (Git, Docker, Jenkins)
4. Create a CI/CD simulation
5. Implement monitoring and logging
6. Document the entire setup

### Skills Practiced
- Virtualization (VirtualBox, KVM)
- Networking configuration
- Tool installation and configuration
- System integration
- Documentation

---

## Project 8: Security Hardening Script (Advanced)
**Time: 5-6 hours**

Create a script to audit and harden system security.

### Objectives
- Learn Linux security fundamentals
- Practice audit techniques
- Implement security best practices

### Steps
1. Audit file permissions on sensitive directories
2. Check for SUID/SGID files
3. Review user accounts and sudo access
4. Check firewall rules
5. Scan for open ports
6. Generate a security report
7. Create remediation scripts

### Skills Practiced
- `find` with permission flags
- `ss`, `netstat` for network analysis
- Security auditing
- Hardening techniques

---

## Project Submission Template

For each project, create a directory with:

```
project-name/
  README.md        # Project description and instructions
  scripts/         # All shell scripts
  docs/            # Additional documentation
  logs/            # Sample logs (sanitized)
  solutions/       # Solution files (optional)
```

## Grading Criteria (Self-Assessment)

| Criteria | Points |
|----------|--------|
| Functionality - Does it work? | 30 |
| Code Quality - Clean, commented code | 20 |
| Documentation - Clear README | 20 |
| Best Practices - Proper permissions, logging | 15 |
| Creativity - Extra features | 15 |
| **Total** | **100** |

## Tips for Success

1. **Start small** - Get the basic functionality working first
2. **Test thoroughly** - Try edge cases and error conditions
3. **Document as you go** - Don't wait until the end
4. **Version control** - Use Git to track your progress
5. **Share your work** - Post on GitHub and get feedback
6. **Iterate** - Improve your projects over time
