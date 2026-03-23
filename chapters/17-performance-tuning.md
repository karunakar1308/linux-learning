# Chapter 17: Performance Tuning and Optimization

## Introduction

Performance tuning is a critical skill for system administrators and DevOps engineers. This chapter covers tools and techniques for identifying bottlenecks, analyzing system performance, and optimizing Linux systems for various workloads.

## Performance Monitoring Fundamentals

### The USE Method

Before diving into tools, understand the USE method for analyzing performance:
- **Utilization**: Is the resource busy?
- **Saturation**: Is there a queue for the resource?
- **Errors**: Are there errors occurring?

### The 60-Second Strategy

A quick 60-second system health check:
1. `uptime` - Load average check
2. `dmesg | tail` - Check for errors
3. `vmstat 1` - CPU, memory, I/O overview
4. `mpstat -P ALL 1` - Per-CPU statistics
5. `pidstat 1` - Per-process stats
6. `iostat -xz 1` - Disk I/O statistics
7. `free -m` - Memory usage
8. `sar -n DEV 1` - Network statistics
9. `sar -n TCP,ETCP 1` - TCP connection stats
10. `top` - Process overview

## CPU Performance Analysis

### Monitoring CPU Usage

```bash
# Quick overview
top -b -n 1 | head -20

# CPU stats at interval
vmstat 1 5

# Detailed CPU per core
mpstat -P ALL 1 5

# Process CPU usage
pidstat -u 1 5

# CPU and memory per process
pidstat -urd 1 5

# Top processes by CPU
ps aux --sort=-%cpu | head -10
```

### Understanding CPU Metrics

| Metric | Description | Normal Range |
|--------|-------------|-------------|
| %user | CPU time in user mode | < 70% |
| %system | CPU time in kernel mode | < 30% |
| %iowait | Waiting for I/O | < 5% |
| %idle | Idle CPU | > 10% |
| %soft | Software interrupt time | < 10% |
| %steal | Stolen by hypervisor | < 1% |

### Identifying CPU Bottlenecks

```bash
# Check load average vs CPU cores
uptime
cat /proc/cpuinfo | grep processor | wc -l

# Find processes causing high load
ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu | head

# Check for CPU-intensive system calls
perf top -g -p $(pidof app)

# Profile specific process
perf record -g -p <PID> sleep 10
perf report
```

## Memory Performance Analysis

### Monitoring Memory Usage

```bash
# Memory overview
free -h

# Detailed memory info
cat /proc/meminfo

# Memory per process
ps aux --sort=-%mem | head -10

# Real-time memory monitoring
watch -n 1 'free -m'

# Detailed process memory
pidstat -r 1 5

# Smem - proportional set size
smem -r -t
```

### Understanding Memory Metrics

```bash
# Check memory details
cat /proc/meminfo | grep -E "MemTotal|MemFree|MemAvailable|Buffers|Cached"

# Check swap usage
swapon --show
free -h | grep Swap

# Check for memory pressure
dmesg | grep -i "oom\|killed"
```

### Memory Optimization

```bash
# Clear page cache (development only)
sudo sync && sudo sh -c 'echo 3 > /proc/sys/vm/drop_caches'

# Set swappiness
sudo sysctl vm.swappiness=10

# Monitor memory pressure
watch 'grep -E "Mem|Swap" /proc/meminfo'

# Check Transparent Huge Pages
cat /sys/kernel/mm/transparent_hugepage/enabled
# To disable: echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

## Disk I/O Performance Analysis

### Monitoring Disk I/O

```bash
# I/O statistics
iostat -xz 1 5

# Real-time I/O per process
iotop -o

# Check I/O wait
vmstat 1 5

# Detailed I/O per device
iostat -x 1 -d

# Check disk latency
iostat -x 1 | awk 'NR>6{print $1, $14, $18}'
```

### Understanding iostat Metrics

| Metric | Description | Good Value |
|--------|-------------|-----------|
| %util | Device utilization | < 80% |
| await | Avg wait time (ms) | < 10ms |
| svctm | Avg service time (ms) | < await |
| r/s | Reads per second | workload dependent |
| w/s | Writes per second | workload dependent |
| rkB/s | Read KB/s | workload dependent |
| wkB/s | Write KB/s | workload dependent |

### Identifying I/O Bottlenecks

```bash
# Check for slow disk operations
pidstat -d 1 5

# Find which processes are doing heavy I/O
for file in /proc/*/io; do cat $file 2>/dev/null; done | grep -B 2 "read_bytes\|write_bytes" | sort -t: -k 2 -n

# Check file system cache efficiency
vmstat 1 | awk 'NR>3 {print "bi="$4, "bo="$5}'

# Check open file descriptors
lsof | wc -l
```

## Network Performance Analysis

### Monitoring Network Statistics

```bash
# Network I/O stats
sar -n DEV 1 5

# Per-interface statistics
ip -s link

# Real-time network monitoring
iftop
nethogs

# Connection tracking
ss -s
netstat -s

# TCP statistics
ss -tan | awk 'NR>1 {a[$2]++} END {for(i in a)print i, a[i]}'

# Network errors
netstat -i
```

### Network Performance Commands

```bash
# Check bandwidth
ip -s link show eth0

# Check open ports
ss -tuln

# Count connections by state
ss -tan | awk 'NR>1 {print $1}' | sort | uniq -c

# Find slowest network connections
dnf install iperf3  # or apt install iperf3
iperf3 -c server_ip

# DNS resolution time
time dig google.com
```

### Network Optimization

```bash
# Check and enable TCP BBR
sudo sysctl -w net.ipv4.tcp_congestion_control=bbr

# Increase network buffer sizes
sudo sysctl -w net.core.rmem_max=134217728
sudo sysctl -w net.core.wmem_max=134217728
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 67108864"
sudo sysctl -w net.ipv4.tcp_wmem="4096 65536 67108864"

# Increase connection queue
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=8192

# Reduce TIME_WAIT sockets
sudo sysctl -w net.ipv4.tcp_tw_reuse=1
sudo sysctl -w net.ipv4.tcp_fin_timeout=30

# Increase file descriptor limits
sudo sysctl -w fs.file-max=2097152
```

## Process Performance Analysis

### Identifying Resource-Hungry Processes

```bash
# Top CPU consumers
ps aux --sort=-%cpu | head -10

# Top memory consumers
ps aux --sort=-%mem | head -10

# Top I/O consumers
pidstat -d 1 5

# Process tree with resource usage
pstree -p

# Check process state distribution
ps -e -o stat | sort | uniq -c | sort -rn
```

### Advanced Process Monitoring

```bash
# Monitor specific process
pidstat -p <PID> -urdh 1

# Check process open files
lsof -p <PID> | wc -l

# Check process network connections
ss -tanp | grep <PID>

# Process memory map
pmap -x <PID> | tail -1

# Check process limits
ulimit -a
cat /proc/<PID>/limits
```

## System Load Analysis

### Understanding Load Average

```bash
# Load average explanation
uptime

# Load per CPU
uptime
cat /proc/cpuinfo | grep processor | wc -l
# Load > CPU count indicates saturation

# Detailed load breakdown
w

# Historical load
cat /var/log/syslog | grep load
```

### Load Average Interpretation

- **1-minute average**: Very recent load
- **5-minute average**: Recent trend
- **15-minute average**: Longer-term trend
- **Load > CPU count**: System is overloaded
- **Load > 2x CPU count**: Critical, system unresponsive

## Performance Tools Deep Dive

### perf - Linux Performance Analyzer

```bash
# Install perf
sudo apt install linux-tools-$(uname -r)  # Debian/Ubuntu
sudo yum install perf                       # RHEL/CentOS

# Profile CPU usage
perf top

# Profile specific process
perf top -p <PID>

# Record performance data
perf record -g -p <PID> sleep 10
perf report

# Profile system calls
perf trace

# Profile I/O
perf record -e block:block_rq_issue -ag
```

### strace - System Call Tracer

```bash
# Trace system calls of running process
strace -p <PID>

# Trace with timing
strace -p <PID> -T

# Trace specific syscalls
strace -p <PID> -e trace=file
strace -p <PID> -e trace=network

# Trace child processes
strace -p <PID> -f

# Count syscalls
strace -c -p <PID>

# Trace new process
strace -f ./program
```

### htop - Interactive Process Viewer

```bash
# Install htop
sudo apt install htop  # Debian/Ubuntu
sudo yum install htop  # RHEL/CentOS

# Run htop
htop

# Key commands in htop:
# F9 - Kill process
# F6 - Sort by column
# M - Sort by memory
# P - Sort by CPU
# T - Sort by time
# F4 - Filter processes
# F5 - Tree view
# Space - Tag process
# F10 - Quit
```

### nmon - Nigel's Performance Monitor

```bash
# Install nmon
sudo apt install nmon  # Debian/Ubuntu
sudo yum install nmon  # RHEL/CentOS

# Run interactively
nmon

# Record data for analysis
nmon -f -t -s 10 -c 360  # Sample every 10s, 360 times

# Convert to CSV for analysis
nmon -f
```

## Performance Baselines and Benchmarking

### Creating Performance Baselines

```bash
# Collect baseline CPU data
mpstat -P ALL 1 60 > cpu_baseline.log

# Collect baseline memory data
free -m >> mem_baseline.log

# Collect baseline I/O data
iostat -xz 1 60 > io_baseline.log

# Collect baseline network data
sar -n DEV 1 60 > net_baseline.log
```

### Simple Benchmarking

```bash
# CPU benchmark
sysbench cpu --cpu-max-prime=20000 run

# Memory benchmark
sysbench memory --memory-block-size=1M --memory-total-size=100G run

# Disk benchmark
fio --name=randread --ioengine=libaio --iodepth=1 --rw=randread \
    --bs=4k --direct=1 --size=1G --numjobs=1 --runtime=60

# Network benchmark (between two hosts)
iperf3 -s  # Server
iperf3 -c server_ip  # Client
```

## Exercises

### Exercise 1: System Health Check

1. Run the 60-second strategy commands on your system
2. Document the current state of CPU, memory, and I/O
3. Identify any obvious issues or warnings

### Exercise 2: CPU Analysis

1. Use `top` and `mpstat` to find the CPU usage pattern
2. Use `pidstat` to identify the top 3 CPU-consuming processes
3. Profile a process with `perf`

### Exercise 3: Memory Analysis

1. Check total, used, and available memory
2. Identify processes using the most memory
3. Check swap usage and swappiness setting

### Exercise 4: I/O Analysis

1. Monitor disk I/O with `iostat`
2. Find the busiest disk
3. Use `iotop` to find I/O-heavy processes

### Exercise 5: Network Analysis

1. Check network interface statistics
2. Count connections by state
3. Identify the busiest network interface

## Key Takeaways

- **Baseline First**: Always establish a performance baseline before making changes
- **USE Method**: Check Utilization, Saturation, and Errors for each resource
- **CPU**: High %user = app issue; High %system = kernel issue; High %iowait = I/O bottleneck
- **Memory**: Low available memory + high swap = memory pressure
- **Disk**: High await or %util > 80% = I/O bottleneck
- **Network**: Connection errors, retransmissions, or dropped packets = network issue
- **Tools**: `top`, `vmstat`, `iostat`, `pidstat`, `perf`, `strace`, `htop`
- **Change One Thing**: When tuning, change one parameter at a time and measure the effect

## Quick Reference

```bash
# One-liner system health check
uptime && echo "---" && free -h && echo "---" && df -h

# Real-time dashboard
tmux new-session -d 'while true; do clear; echo "=== UPTIME ==="; uptime; echo "=== CPU ==="; top -bn1 | grep "Cpu(s)"; echo "=== MEM ==="; free -h; echo "=== DISK ==="; df -h; sleep 5; done'
```

## Troubleshooting

```bash
# System running slow - check everything
dmesg | tail -50
journalctl --since "1 hour ago" | grep -i error
vmstat 1 10
iostat -xz 1 10

# OOM killer triggered
journalctl -k | grep -i "out of memory"

# High load average with low CPU
vmstat 1 10  # Check %iowait
iostat -xz 1 10  # Check disk queue

# Memory leak suspected
ps aux --sort=-%mem | head -5
cat /proc/<PID>/status | grep -E "VmRSS|VmSize"
```

## Next Steps

- Learn about container resource limits and monitoring
- Explore distributed tracing with Jaeger and Zipkin
- Study application-level profiling (pprof, py-spy)
- Deep dive into performance tuning for databases (MySQL, PostgreSQL)
