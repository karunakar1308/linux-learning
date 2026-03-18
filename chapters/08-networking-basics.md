# Chapter 08: Networking Basics

## Introduction

Networking is a fundamental aspect of Linux administration. Understanding how networks work, how to configure network interfaces, and how to troubleshoot network issues is essential for any Linux administrator.

## OSI Model and TCP/IP Model

### OSI Model Layers
1. **Physical Layer**: Hardware, cables, signals
2. **Data Link Layer**: MAC addresses, switching
3. **Network Layer**: IP addresses, routing
4. **Transport Layer**: TCP/UDP, ports
5. **Session Layer**: Connection establishment
6. **Presentation Layer**: Data formatting
7. **Application Layer**: HTTP, SSH, FTP, DNS

### TCP/IP Model (Simplified)
- **Link Layer**: Physical and Data Link combined
- **Internet Layer**: IP routing
- **Transport Layer**: TCP/UDP
- **Application Layer**: Services and protocols

## Network Interface Configuration

### Viewing Network Interfaces

```bash
# View all network interfaces and IP addresses
ip addr show
ip a

# Display only active interfaces
ip link show

# Display detailed information about network interfaces
ifconfig     # Older tool, less commonly used on modern systems

# View IPv6 information
ip -6 addr show

# Show interface statistics
ip -s link show
```

### Network Interface Files (Traditional)

Debian/Ubuntu: `/etc/network/interfaces`
RHEL/CentOS: `/etc/sysconfig/network-scripts/`
Arch: `/etc/netctl/`

```bash
# View network configuration
cat /etc/network/interfaces

# View network scripts
ls -la /etc/sysconfig/network-scripts/
```

### Modern Network Management: Netplan

Netplan is used by modern Ubuntu and Debian systems:

```bash
# Netplan configuration
cat /etc/netplan/01-netcfg.yaml

# Example static IP configuration
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4

# Apply netplan configuration
sudo netplan apply
```

### IP Address Configuration (Temporary)

```bash
# Assign static IP
sudo ip addr add 192.168.1.100/24 dev eth0

# Remove IP address
sudo ip addr del 192.168.1.100/24 dev eth0

# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down

# Set default gateway
sudo ip route add default via 192.168.1.1

# Add specific route
sudo ip route add 192.168.2.0/24 via 192.168.1.254
```

## DNS Configuration

### DNS Files

```bash
# Main DNS configuration file
cat /etc/resolv.conf

# Systemd-resolved (modern systems)
cat /etc/systemd/resolved.conf

# View current DNS servers
systemd-resolve --status
resolvectl status
```

### Setting DNS Servers

```bash
# Edit resolve.conf (temporary, may be overwritten)
sudo nano /etc/resolv.conf

# Add nameservers
nameserver 8.8.8.8
nameserver 8.8.4.4

# For permanent configuration with systemd-resolved
sudo nano /etc/systemd/resolved.conf
# Add DNS=8.8.8.8 8.8.4.4

# Apply changes
sudo systemctl restart systemd-resolved
```

## Routing

### View Routing Table

```bash
# Modern method (ip command)
ip route show
ip r

# Older method (route command)
route -n

# Show detailed routing information
ip route show all

# View default route
ip route show default
```

### Add and Remove Routes

```bash
# Add default gateway
sudo ip route add default via 192.168.1.1

# Add specific route
sudo ip route add 192.168.2.0/24 via 192.168.1.254 dev eth0

# Remove route
sudo ip route del 192.168.2.0/24 via 192.168.1.254

# Remove default gateway
sudo ip route del default via 192.168.1.1
```

## Network Testing and Troubleshooting

### Ping

```bash
# Test connectivity
ping 8.8.8.8

# Ping specific number of times
ping -c 4 google.com

# Set packet size
ping -s 1500 8.8.8.8

# Continuous ping
ping 8.8.8.8   # Press Ctrl+C to stop
```

### Traceroute

```bash
# Trace route to destination
traceroute google.com

# Show hops to reach destination
traceroute -m 30 8.8.8.8
```

### Netstat and ss

```bash
# View listening ports (netstat)
netstat -tlnp

# View all connections (netstat)
netstat -tunap

# Modern alternative (ss)
ss -tlnp          # Listening TCP ports
ss -tunap         # All TCP/UDP connections
ss -s             # Summary statistics
ss -i             # Show TCP information
```

### ifconfig (Legacy)

```bash
# Display all network interfaces
ifconfig

# Display specific interface
ifconfig eth0

# Set IP address (temporary)
sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0

# Bring interface up/down
sudo ifconfig eth0 up
sudo ifconfig eth0 down
```

### nslookup and dig

```bash
# DNS lookup (simple)
nslookup google.com

# DNS lookup (advanced)
dig google.com

# Show all record types
dig google.com ANY

# Reverse DNS lookup
dig -x 8.8.8.8

# Query specific nameserver
dig @8.8.8.8 google.com
```

### Network Performance Testing

```bash
# Check bandwidth usage
iftop

# Monitor network in real-time
nethogs

# Show network statistics
netstat -i

# TCP connection tracking
conntrack -L

# Test network throughput
iperf -s                    # Server
iperf -c 192.168.1.100      # Client

# Check port connectivity
telnet 192.168.1.1 22
nc -zv 192.168.1.1 22       # Using netcat
```

## Firewall Basics (Iptables)

### View Firewall Rules

```bash
# List all rules
sudo iptables -L

# List with line numbers
sudo iptables -L -n --line-numbers

# List specific chain
sudo iptables -L INPUT

# Show rules in commands format
sudo iptables -S
```

### Add Firewall Rules

```bash
# Allow incoming SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow incoming HTTP
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Allow incoming HTTPS
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Block specific IP
sudo iptables -A INPUT -s 192.168.1.50 -j DROP

# Set default policy
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD DROP
```

### Save Iptables Rules

```bash
# Debian/Ubuntu
sudo apt install iptables-persistent
sudo iptables-save > /etc/iptables/rules.v4

# RHEL/CentOS
sudo service iptables save
```

## UFW (Uncomplicated Firewall)

```bash
# Enable firewall
sudo ufw enable

# Disable firewall
sudo ufw disable

# Allow port
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Deny port
sudo ufw deny 23/tcp

# Allow from specific IP
sudo ufw allow from 192.168.1.100 to any port 22

# Delete rule
sudo ufw delete allow 22/tcp

# Show firewall status
sudo ufw status
sudo ufw status verbose

# Reset firewall
sudo ufw reset
```

## SSH and Remote Access

### SSH Connection

```bash
# Basic SSH connection
ssh username@hostname

# SSH with specific port
ssh -p 2222 username@hostname

# SSH with key file
ssh -i ~/.ssh/id_rsa username@hostname

# Run command over SSH
ssh username@hostname 'command'

# Copy files with SCP
scp /local/file username@hostname:/remote/path
scp -r /local/dir username@hostname:/remote/path

# Secure copy using SSH key
scp -i ~/.ssh/id_rsa /local/file username@hostname:/remote/path
```

### SSH Key Generation

```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096

# Generate ED25519 key (recommended)
ssh-keygen -t ed25519

# Add key to SSH agent
ssh-add ~/.ssh/id_rsa

# Copy public key to remote server
ssh-copy-id username@hostname

# Manual setup (if ssh-copy-id not available)
cat ~/.ssh/id_rsa.pub | ssh username@hostname 'cat >> ~/.ssh/authorized_keys'
```

## Network Bonding and Teaming

### Network Bonding

```bash
# Create bond interface
sudo ip link add bond0 type bond mode active-backup

# Add interfaces to bond
sudo ip link set eth0 master bond0
sudo ip link set eth1 master bond0

# Bring up bond
sudo ip link set bond0 up
```

### Network Team

```bash
# Create team interface
sudo ip link add team0 type team

# Configure team interface
sudo ip link set eth0 master team0
sudo ip link set eth1 master team0
```

## Proxy Configuration

### System-wide Proxy

```bash
# Set environment variables in /etc/environment
export http_proxy=http://proxy.example.com:8080
export https_proxy=https://proxy.example.com:8080
export ftp_proxy=http://proxy.example.com:8080
export no_proxy="localhost,127.0.0.1,internal.example.com"

# For user-specific configuration
nano ~/.bashrc
export http_proxy=http://proxy.example.com:8080
```

### Application-specific Proxy

```bash
# APT package manager
sudo nano /etc/apt/apt.conf.d/proxy.conf
Acquire::http::Proxy "http://proxy.example.com:8080";
Acquire::https::Proxy "https://proxy.example.com:8080";
```

## Network Monitoring Tools

- **ifconfig/ip**: Display interface configuration
- **ping**: Test connectivity
- **traceroute/mtr**: Trace network path
- **netstat/ss**: Connection statistics
- **arp**: View ARP table
- **route**: View routing table
- **nslookup/dig**: DNS queries
- **iftop**: Bandwidth monitoring
- **nethogs**: Process-level network monitoring
- **wireshark**: Packet capture and analysis
- **tcpdump**: Command-line packet capture

## Practice Exercises

### Basic Exercises

1. View all network interfaces on your system
2. Identify your default gateway and DNS servers
3. Test connectivity to google.com using ping
4. Use traceroute to see the path to a remote server
5. Check open ports on your system using netstat or ss
6. Perform a DNS lookup using nslookup and dig
7. Use ifconfig to view detailed network information

### Intermediate Exercises

1. Configure a static IP address using netplan or /etc/network/interfaces
2. Add a static route to your routing table
3. Set up SSH key-based authentication
4. Use SCP to transfer a file between systems
5. Monitor bandwidth usage with iftop or nethogs
6. Check DNS resolution for a domain
7. Create a firewall rule using iptables or UFW

### Advanced Exercises

1. Set up network bonding with active-backup mode
2. Configure multiple IP addresses on a single interface
3. Create a bridge interface for virtualization
4. Set up a packet filter with detailed iptables rules
5. Monitor and analyze traffic with tcpdump
6. Configure network team for load balancing
7. Set up IPv6 addressing and configuration

## Common Issues and Solutions

### No Network Connectivity
- Check if interface is up: `ip link show`
- Check IP configuration: `ip addr show`
- Check default gateway: `ip route show default`
- Verify DNS: `cat /etc/resolv.conf`

### Wrong IP Address
- Check configured IP: `ip addr show`
- Configure correct IP: `ip addr add`
- Restart networking: `sudo systemctl restart networking`

### Cannot Reach Remote Host
- Check routing: `ip route show`
- Test with ping: `ping <host>`
- Use traceroute: `traceroute <host>`
- Check firewall: `sudo iptables -L INPUT`

### DNS Not Working
- Check resolv.conf: `cat /etc/resolv.conf`
- Test nslookup: `nslookup google.com`
- Restart resolver: `sudo systemctl restart systemd-resolved`

## Resources

- [Linux Foundation Networking](https://linux.com)
- [Man pages for ip, route, netstat, ss](https://linux.die.net/)
- [Netplan Documentation](https://netplan.io/)
- [iptables Documentation](https://linux.die.net/man/8/iptables)
- [SSH Security Best Practices](https://man.openbsd.org/ssh)
