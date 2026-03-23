# Chapter 20: Advanced Security - SELinux and AppArmor

## Introduction

Security is a multi-layered discipline. While Chapter 13 covered basic security (firewalls, SSH hardening), this chapter dives deep into **Mandatory Access Control (MAC)** systems: **SELinux** (Security-Enhanced Linux) and **AppArmor**. These systems provide an additional security layer beyond traditional Discretionary Access Control (DAC - file permissions).

## Understanding MAC vs DAC

### Discretionary Access Control (DAC)

- **Who controls**: The file owner
- **Based on**: User ID and group ID
- **Limitations**: Root can override, no protection from compromised processes

```bash
# Traditional permissions (DAC)
ls -la /home/user/file
# -rw-r--r-- 1 user user 100 Jan 1 00:00 file
# Owner: read/write
# Group: read
# Others: read
```

### Mandatory Access Control (MAC)

- **Who controls**: System policy (not user)
- **Based on**: Security contexts/labels
- **Advantages**: Root cannot override policy, process confinement

## SELinux Fundamentals

### What is SELinux?

SELinux is a MAC system originally developed by the NSA and now maintained by Red Hat. It provides:
- Process confinement
- Resource access control
- Role-based access
- Multi-level security

### SELinux Modes

```bash
# Check SELinux status
sestatus

# Check current mode
getenforce

# Check SELinux mode (0=disabled, 1=permissive, 2=enforcing)
cat /sys/fs/selinux/enforce
```

| Mode | Description | Use Case |
|------|-------------|----------|
| **Enforcing** | Blocks and logs violations | Production |
| **Permissive** | Logs but does not block | Troubleshooting |
| **Disabled** | Completely off | Not recommended |

### Changing SELinux Mode

```bash
# Set mode temporarily (until reboot)
setenforce 0  # Permissive
setenforce 1  # Enforcing

# Set mode permanently
# Edit /etc/selinux/config
# SELINUX=permissive
# SELINUX=enforcing
# SELINUX=disabled

# Reboot required for permanent change if going from disabled
sudo reboot
```

## SELinux Contexts

### File Contexts

```bash
# View SELinux context of files
ls -Z /etc/passwd
# system_u:object_r:passwd_file_t:s0 /etc/passwd

# Format: user:role:type:level

# user - SELinux user (system_u)
# role - SELinux role (object_r)
# type - Security type (passwd_file_t) - MOST IMPORTANT
# level - Sensitivity level (s0)

# View all contexts
ls -laZ /var/www/html/

# Search for specific context
ls -Z / | grep http
```

### Process Contexts

```bash
# View process contexts
ps -eZ

# View specific process
ps -eZ | grep httpd

# View process with context
ps auxZ | grep nginx
```

### User Contexts

```bash
# View SELinux user mappings
semanage login -l

# Map Linux user to SELinux user
sudo semanage login -a -s user_u username

# View current user context
id -Z
```

## SELinux Booleans

### What are Booleans?

Booleans are toggle switches that enable/disable specific SELinux policies without modifying them.

```bash
# List all booleans
getsebool -a

# Search for specific boolean
getsebool -a | grep httpd

# Check specific boolean
getsebool httpd_can_network_connect

# Set boolean temporarily
setsebool httpd_can_network_connect on

# Set boolean permanently (-P makes it persistent across reboots)
setsebool -P httpd_can_network_connect on

# Common useful booleans
# httpd_can_network_connect    - Allow web server to make network connections
# httpd_read_user_content      - Allow web server to read user home dirs
# httpd_execmem               - Allow web server to execute memory
# ftpd_passive_mode           - Enable FTP passive mode
# ssh_sysadm_login           - Allow SSH sysadmin login
# named_tcp_bind_http_port   - Allow DNS on HTTP port
```

## SELinux File Context Management

### Restoring Contexts

```bash
# Restore default context to a file
restorecon -v /path/to/file

# Restore recursively
restorecon -Rv /var/www/html/

# Restore with verbose
restorecon -vR /home/user/data/

# Check what context would be set (without changing)
restorecon -n /path/to/file
```

### Changing File Contexts

```bash
# Set custom context temporarily
chcon -t httpd_sys_content_t /var/www/html/myfile.html

# Set custom context permanently (recommended method)
semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?"
restorecon -Rv /var/www/html/

# View custom file contexts
semanage fcontext -l

# Remove custom file context
semanage fcontext -d "/var/www/html(/.*)?"
restorecon -Rv /var/www/html/
```

### Context Types for Common Services

| Context Type | Purpose |
|-------------|--------|
| `httpd_sys_content_t` | Static web content |
| `httpd_sys_content_exec_t` | Executable web scripts |
| `httpd_log_t` | Web server logs |
| `httpd_t` | Web server process |
| `mysqld_t` | MySQL process |
| `mysqld_db_t` | MySQL database files |
| `ssh_home_t` | SSH authorized_keys |
| `nfs_t` | NFS shared files |

## SELinux Troubleshooting

### Finding Denials

```bash
# Check audit log for denials
grep denials /var/log/audit/audit.log

# Use ausearch for denials
ausearch -m avc -ts recent

# View recent SELinux denials
ausearch -m avc -ts today

# Count denials
ausearch -m avc -ts today | wc -l
```

### Using audit2why

```bash
# Analyze denials and explain why they occurred
ausearch -m avc -ts today | audit2why

# Get policy recommendation
ausearch -m avc -ts today | audit2allow

# Create custom policy module
ausearch -m avc -ts today | audit2allow -M mypolicy
semodule -i mypolicy.pp
```

### Using sealert

```bash
# Install setroubleshoot-server if not installed
sudo yum install setroubleshoot-server
sudo systemctl start setroubleshoot

# Generate human-readable report
sealert -a /var/log/audit/audit.log

# Check for new alerts
sealert -b

# View specific report
sealert -l REPORT_ID
```

## AppArmor Fundamentals

### What is AppArmor?

AppArmor is a MAC system used primarily by Ubuntu and SUSE. Unlike SELinux which uses contexts, AppArmor uses path-based profiles.

### Checking AppArmor Status

```bash
# Check if AppArmor is enabled
sudo apparmor_status

# View loaded profiles
sudo aa-status

# Check specific profile
sudo aa-status | grep nginx
```

| Mode | Description |
|------|-------------|
| **Enforce** | Policy is active and enforced |
| **Complain** | Policy logs violations but does not block |
| **Unconfined** | No profile applied |

### AppArmor Profile Location

```bash
# Profile directory
ls /etc/apparmor.d/

# Profile content
cat /etc/apparmor.d/usr.sbin.nginx

# Profile status
sudo apparmor_status
```

## AppArmor Profile Management

### Enabling Profiles

```bash
# Put profile in enforce mode
sudo aa-enforce /path/to/program

# Put profile in complain mode (for testing)
sudo aa-complain /path/to/program

# Put all profiles in enforce mode
sudo aa-enforce /etc/apparmor.d/*

# Toggle specific profile
sudo aa-toggle /path/to/program
```

### Creating Custom Profiles

```bash
# Create profile in complain mode first
sudo aa-genprof /path/to/program
# Follow interactive prompts

# Or create manually
cat > /etc/apparmor.d/myapp << 'EOF'
#include <tunables/global>

/path/to/myapp {
  #include <abstractions/base>
  
  /usr/bin/myapp mr,
  /var/log/myapp.log rw,
  /etc/myapp.conf r,
  /home/myapp/data/ rw,
  network inet stream,
}
EOF

# Load the profile
sudo apparmor_parser -r /etc/apparmor.d/myapp

# Enable it
sudo aa-enforce /path/to/myapp
```

### AppArmor Commands

```bash
# Reload all profiles
sudo apparmor_parser -r /etc/apparmor.d/*

# Reload specific profile
sudo apparmor_parser -r /etc/apparmor.d/nginx

# Remove profile
sudo apparmor_parser -R /etc/apparmor.d/myapp

# View logs
dmesg | grep apparmor
journalctl -k | grep apparmor
```

## SELinux vs AppArmor Comparison

| Feature | SELinux | AppArmor |
|---------|---------|----------|
| Origin | NSA / Red Hat | Novell / Ubuntu |
| Control Type | Label-based | Path-based |
| Complexity | High | Medium |
| Flexibility | Very high | High |
| Default Distro | RHEL, CentOS, Fedora | Ubuntu, SUSE |
| Learning Curve | Steep | Moderate |
| Granularity | Fine-grained | Moderate |
| Community | Large | Good |

### Which One to Use?

**SELinux when:**
- Using RHEL/CentOS/Fedora
- Need fine-grained security controls
- Multi-tenant environments
- Compliance requirements (DoD, PCI-DSS)

**AppArmor when:**
- Using Ubuntu/SUSE
- Need simpler configuration
- Application-specific confinement
- Easier to learn and maintain

## Container Security

### SELinux with Containers

```bash
# Docker with SELinux labels
docker run -d --security-opt label=disable nginx
# WARNING: Disabling labels reduces security

docker run -d --security-opt label=type:svirt_sandbox nginx

# Check container SELinux context
ps auxZ | grep docker

# Set specific label
docker run -d --security-opt label=user:system_u \
    --security-opt label=role:system_r nginx
```

### AppArmor with Containers

```bash
# Docker with AppArmor profile
docker run -d --security-opt apparmor=profile-name nginx

# Check AppArmor profiles for containers
sudo apparmor_status | grep docker

# Create custom Docker profile
cat > /etc/apparmor.d/docker-custom << 'EOF'
#include <tunables/global>

profile docker-custom flags=(attach_disconnected,mediate_deleted) {
  #include <abstractions/base>
  network inet tcp,
  network inet udp,
  network inet icmp,
  deny network raw,
  deny network packet,
  file,
  capability,
}
EOF

# Use it
docker run -d --security-opt apparmor=docker-custom nginx
```

### Kubernetes Security Context

```yaml
# Pod security context with SELinux
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    seLinuxOptions:
      level: "s0:c123,c456"
      role: "system_r"
      type: "svirt_sandbox_t"
      user: "system_u"
  containers:
  - name: app
    image: myapp
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

## Exercises

### Exercise 1: SELinux Basics

1. Check SELinux status on your system
2. If in enforcing mode, switch to permissive
3. List file contexts in /var/www
4. Switch back to enforcing

### Exercise 2: SELinux Troubleshooting

1. Install Apache/nginx
2. Create a custom web directory
3. Configure SELinux context for it
4. Use setsebool to allow network connections
5. Check audit logs for any denials

### Exercise 3: AppArmor Profiles

1. Check AppArmor status
2. List loaded profiles
3. Put a profile in complain mode
4. Generate activity and check logs
5. Put it back in enforce mode

### Exercise 4: Custom SELinux Policy

1. Create a situation that generates SELinux denials
2. Use audit2allow to analyze them
3. Create a custom policy module
4. Load the policy and verify

### Exercise 5: Container Security

1. Run a Docker container with security options
2. Check the container's SELinux/AppArmor context
3. Apply a custom security profile
4. Verify restrictions are working

## Key Takeaways

- **MAC vs DAC**: MAC provides additional security layer beyond traditional permissions
- **SELinux**: Label-based, more complex, RHEL/CentOS default
- **AppArmor**: Path-based, simpler, Ubuntu/SUSE default
- **SELinux Modes**: Enforcing (block+log), Permissive (log only), Disabled
- **File Contexts**: Use `restorecon` to fix, `semanage` to persist
- **Booleans**: Toggle policy options without editing rules
- **Troubleshooting**: Use `ausearch`, `audit2why`, `sealert` (SELinux) or `aa-status`, `apparmor_parser` (AppArmor)
- **Containers**: Both SELinux and AppArmor integrate with Docker/Kubernetes
- **Never disable**: Avoid disabling MAC entirely; troubleshoot instead

## Quick Reference

```bash
# SELinux commands
sestatus                    # Check status
getenforce                  # Get current mode
setenforce 0|1              # Set mode
getsebool -a                # List booleans
setsebool -P bool on        # Set boolean
restorecon -Rv /path        # Restore contexts
semanage fcontext           # Manage file contexts
ausearch -m avc             # Find denials
audit2allow -M policy       # Create policy

# AppArmor commands
sudo aa-status              # Check status
sudo aa-enforce profile     # Enforce profile
sudo aa-complain profile    # Complain mode
sudo apparmor_parser -r     # Reload profile
```

## Troubleshooting

```bash
# SELinux blocking something
# 1. Switch to permissive temporarily
setenforce 0

# 2. Check audit log
ausearch -m avc -ts recent

# 3. Analyze and fix
ausearch -m avc -ts recent | audit2why

# 4. Either fix context or use boolean
setsebool -P httpd_can_network_connect on
restorecon -Rv /var/www/html/

# AppArmor blocking something
# 1. Check status
sudo aa-status

# 2. Check dmesg for AppArmor denials
dmesg | grep apparmor

# 3. Switch to complain mode for debugging
sudo aa-complain /path/to/program

# 4. After fixing, switch back to enforce
sudo aa-enforce /path/to/program

# Common issues
# "Permission denied" with SELinux - check context with ls -Z
# Service won't start - check boolean with getsebool
# AppArmor "denied" in logs - check profile with aa-status
```

## Next Steps

- Study Linux audit framework (auditd) in depth
- Learn about security benchmarking (CIS, STIG)
- Explore runtime security tools (Falco, Sysdig)
- Study container security best practices
- Learn about kernel security modules (LSM)
