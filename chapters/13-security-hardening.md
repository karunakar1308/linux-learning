# Chapter 13: Security Hardening
> **Difficulty**: Advanced | **Estimated Time**: 4-5 days
---

## 13.1 Security Fundamentals

### The Security Triad (CIA)
- **Confidentiality**: Only authorized users can access data
- **Integrity**: Data remains accurate and unaltered
- **Availability**: Systems and data are accessible when needed

### Common Security Threats
- Brute force attacks
- Privilege escalation
- Malware and rootkits
- Denial of Service (DoS)
- Man-in-the-middle attacks
- Social engineering

---

## 13.2 Securing SSH

### Disable Root Login
```bash
# Edit SSH configuration
sudo nano /etc/ssh/sshd_config

# Change these settings:
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes

# Restart SSH service
sudo systemctl restart sshd
```

### Change SSH Port
```bash
# Change default port from 22
Port 2222

# Update firewall
sudo ufw allow 2222/tcp
```

### SSH Key Best Practices
```bash
# Generate strong key (Ed25519 recommended)
ssh-keygen -t ed25519 -a 100

# Set proper permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

### SSH Hardening Checklist
```bash
# /etc/ssh/sshd_config
Protocol 2
X11Forwarding no
AllowTcpForwarding no
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers admin developer
```n
---

## 13.3 Firewall Configuration

### UFW - Uncomplicated Firewall
```bash
# Enable firewall
sudo ufw enable

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow specific ports
sudo ufw allow 22/tcp      # SSH
sudo ufw allow 80/tcp      # HTTP
sudo ufw allow 443/tcp     # HTTPS
sudo ufw allow from 192.168.1.0/24

# View status
sudo ufw status verbose
sudo ufw status numbered

# Delete a rule
sudo ufw delete allow 22/tcp

# Enable logging
sudo ufw logging on
```

### IPTables - Advanced Firewall
```bash
# List all rules
sudo iptables -L -n -v --line-numbers

# Drop all incoming, allow established
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow SSH, HTTP, HTTPS
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Rate limit SSH (prevent brute force)
sudo iptables -A INPUT -p tcp --dport 22 -m recent --set
sudo iptables -A INPUT -p tcp --dport 22 -m recent --update --seconds 60 --hitcount 4 -j DROP

# Save rules
sudo iptables-save > /etc/iptables/rules.v4
sudo iptables-save > /etc/iptables/rules.v6
```

### Fail2Ban - Intrusion Prevention
```bash
# Install fail2ban
sudo apt install fail2ban

# Create local configuration
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit jail.local to protect SSH
[jail.ssh]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600

# Start fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd
```n
---

## 13.4 SELinux (Security-Enhanced Linux)

### Understanding SELinux
```bash
# Check SELinux status
getenforce
sestatus

# SELinux modes:
# Enforcing - SELinux blocks and logs violations
# Permissive - SELinux only logs, does not block
# Disabled - SELinux is turned off

# Set mode temporarily
setenforce 1  # Enforcing
setenforce 0  # Permissive

# Set mode permanently
# Edit /etc/selinux/config
# SELINUX=enforcing
```

### Common SELinux Commands
```bash
# Check context of a file
ls -Z /path/to/file

# Change file context
chcon -t httpd_exec_t /path/to/file

# Restore default context
restorecon -vR /var/www/html

# Check for SELinux denials
ausearch -m avc -ts recent

# Set boolean (persistent)
setsebool -P httpd_can_network_connect 1

# List all booleans
getsebool -a
```n
---

## 13.5 AppArmor

### AppArmor Basics
```bash
# Check status
sudo aa-status

# AppArmor modes:
# Complain - Log violations but do not block
# Enforce - Block and log violations

# Change profile mode
sudo aa-complain /usr/sbin/nginx
sudo aa-enforce /usr/sbin/nginx

# Create new profile
sudo aa-genprof /path/to/application

# Reload profiles
sudo systemctl reload apparmor
sudo aa-enforce /etc/apparmor.d/*
```

---

## 13.6 File Integrity Monitoring

### AIDE (Advanced Intrusion Detection Environment)
```bash
# Install AIDE
sudo apt install aide

# Initialize database
sudo aide --init

# Move database
sudo mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz

# Check for changes
sudo aide --check

# Update database
sudo aide --update
```

### Tripwire
```bash
# Install tripwire
sudo apt install tripwire

# Configure and initialize
sudo twintool --init

# Check integrity
sudo tripwire --check
```

---

## 13.7 User Account Security

### Password Policies
```bash
# Set password aging
sudo chage -M 90 -m 7 -W 14 username
# -M: Max days between password changes
# -m: Min days between password changes  
# -W: Days before expiry to warn

# Force password change on next login
sudo chage -d 0 username
```

### PAM Configuration
```bash
# Password complexity
# /etc/pam.d/common-password
password requisite pam_pwquality.so retry=3 minlen=12

# Lock account after failed attempts
# /etc/pam.d/common-auth
auth required pam_faillock.so preauth deny=5 unlock_time=900

# Disable empty passwords
# /etc/login.defs
PASSWD_MIN_LEN 12
```

### Account Locking
```bash
# Lock an account
sudo passwd -l username

# Unlock an account
sudo passwd -u username

# Check lock status
sudo passwd -S username
```

---

## 13.8 Securing Running Services

### Disable Unnecessary Services
```bash
# List all services
systemctl list-units --type=service --all

# Stop and disable unnecessary services
sudo systemctl stop cups
sudo systemctl disable cups
sudo systemctl stop bluetooth
sudo systemctl disable bluetooth

# Mask a service (prevent starting even by other services)
sudo systemctl mask cups
```

### Check Open Ports
```bash
# List all listening ports
ss -tlnp
netstat -tlnp

# Check what's listening on specific port
ss -tlnp | grep :22
lsof -i :22
```

---

## 13.9 Kernel Hardening

### Sysctl Hardening
```bash
# View current settings
sysctl -a

# Common hardening settings
# /etc/sysctl.conf

# Disable ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Disable IP forwarding
net.ipv4.ip_forward = 0

# Disable source routing
net.ipv4.conf.all.accept_source_route = 0

# Enable SYN flood protection
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048

# Ignore ICMP broadcasts
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Ignore bogus ICMP errors
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Limit core dumps
fs.suid_dumpable = 0

# Apply settings
sudo sysctl -p
```

---

## 13.10 File and Directory Security

### Secure Sensitive Directories
```bash
# Protect /etc
chmod 755 /etc
chmod 600 /etc/shadow
chmod 644 /etc/passwd

# Protect /root
chmod 700 /root

# Protect /boot
chmod 700 /boot

# Secure shared directories
chmod 1777 /tmp
chmod 1777 /var/tmp

# Remove SUID/SGID from unnecessary files
chmod u-s /usr/bin/example
```

### Find Security Issues
```bash
# Find world-writable files
find / -type f -perm -002 2>/dev/null

# Find SUID binaries
find / -type f -perm -4000 2>/dev/null

# Find SGID binaries
find / -type f -perm -2000 2>/dev/null

# Find files with no owner
find / -nouser -o -nogroup 2>/dev/null

# Find empty passwords
sudo awk -F: '($2 == "")' /etc/shadow
```

---

## 13.11 Network Security Tools

### Nmap - Network Scanner
```bash
# Scan a host
nmap 192.168.1.1

# Detailed scan
nmap -sV -sC 192.168.1.1

# Scan for vulnerabilities
nmap --script vuln 192.168.1.1

# Scan specific ports
nmap -p 22,80,443 192.168.1.1

# OS detection
nmap -O 192.168.1.1
```

### tcpdump - Packet Capture
```bash
# Capture all traffic
tcpdump

# Capture on specific interface
tcpdump -i eth0

# Capture specific port
tcpdump -i eth0 port 22

# Capture to file
tcpdump -i eth0 -w capture.pcap

# Read captured file
tcpdump -r capture.pcap

# Filter by IP
tcpdump host 192.168.1.100
```

---

## 13.12 Security Audit Checklist

### Automated Security Audits
```bash
# Install lynis
sudo apt install lynis

# Run security audit
sudo lynis audit system

# View report
sudo less /var/log/lynis.log

# Install rkhunter (rootkit hunter)
sudo apt install rkhunter

# Update signatures
sudo rkhunter --update

# Check for rootkits
sudo rkhunter --check
```

### Manual Security Checklist
- [ ] Change default passwords
- [ ] Update all packages
- [ ] Configure firewall (UFW/iptables)
- [ ] Harden SSH configuration
- [ ] Disable unnecessary services
- [ ] Set up fail2ban
- [ ] Configure log rotation
- [ ] Set up intrusion detection (AIDE/Tripwire)
- [ ] Apply kernel hardening (sysctl)
- [ ] Review file permissions
- [ ] Remove SUID/SGID from unnecessary binaries
- [ ] Configure SELinux/AppArmor
- [ ] Set up regular security updates
- [ ] Configure proper umask (0027 or stricter)
- [ ] Review user accounts and sudo access
- [ ] Enable audit logging

---

## 13.13 Best Practices

### Principle of Least Privilege
- Grant minimum permissions needed
- Use sudo for specific commands, not full root
- Create separate users for services
- Restrict network access by default

### Regular Maintenance
- Apply security patches weekly
- Review logs daily/weekly
- Rotate credentials quarterly
- Audit permissions monthly
- Update intrusion detection signatures

### Defense in Depth
- Layer multiple security controls
- Don't rely on a single mechanism
- Combine firewall, SELinux, and application security

---

## Practice Exercises

### Exercise 1: SSH Hardening
1. Generate Ed25519 SSH key
2. Configure SSH to disable password authentication
3. Disable root login
4. Change SSH port
5. Test connection with new settings

### Exercise 2: Firewall Setup
1. Enable UFW with default deny incoming
2. Allow only SSH, HTTP, HTTPS
3. Add rate limiting for SSH
4. Install and configure fail2ban
5. Test firewall rules

### Exercise 3: Security Audit
1. Install lynis and run full audit
2. Fix all high-priority warnings
3. Install and run rkhunter
4. Check for SUID/SGID files
5. Review world-writable files

### Exercise 4: Account Security
1. Create a new user with strong password
2. Set password aging policy
3. Add user to sudo with limited commands
4. Lock an account and test
5. Review /etc/passwd and /etc/shadow

### Exercise 5: Kernel Hardening
1. Edit /etc/sysctl.conf with hardening parameters
2. Apply settings with sysctl -p
3. Verify settings are active
4. Make settings persistent across reboots

## Key Takeaways
- **SSH**: Disable root, use keys, change port, install fail2ban
- **Firewall**: Default deny, allow only needed ports
- **SELinux/AppArmor**: Mandatory access control layers
- **PAM**: Enforce strong password policies
- **File integrity**: AIDE/Tripwire detect unauthorized changes
- **Least privilege**: Never run as root unnecessarily
- **Defense in depth**: Multiple layers of security
- **Regular audits**: Automated and manual security checks

## Next Steps
- Study incident response procedures
- Learn about penetration testing
- Explore security automation and compliance frameworks
- Practice CTF (Capture The Flag) challenges
- Read the CIS Benchmarks for your distribution
