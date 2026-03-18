# Chapter 04: Process Management

## 4.1 Understanding Processes

### What is a Process?
A process is an instance of a running program. Every command or program you execute creates one or more processes.

### Process Attributes
```bash
PID    # Process ID (unique identifier)
PPID   # Parent Process ID
UID    # User ID of owner
GID    # Group ID
Priority # Process priority
State  # Running, Sleeping, Stopped, Zombie
```

### Process States
- **Running (R)**: Currently executing
- **Sleeping (S)**: Waiting for an event
- **Stopped (T)**: Stopped by signal
- **Zombie (Z)**: Terminated but not reaped by parent
- **Defunct**: Dead process waiting for parent

## 4.2 Viewing Processes

### ps Command
```bash
# Show current user processes
ps

# Show all processes
ps aux

# Show all processes with hierarchy
ps auxf

# Show processes for specific user
ps -u username

# Custom format
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem

# Show process tree
ps -ejH
```

### Common ps Options
```bash
a     # Show processes for all users
u     # Display user-oriented format
x     # Show processes without controlling terminal
f     # ASCII art process tree
-e    # Select all processes
-f    # Full format listing
-l    # Long format
```

### top Command (Interactive)
```bash
# Launch top
top

# Common top commands while running:
# k : Kill a process
# r : Renice a process
# P : Sort by CPU usage
# M : Sort by memory usage
# 1 : Show individual CPUs
# h : Help
# q : Quit
```

### htop (Enhanced Interactive Viewer)
```bash
# Install htop
sudo apt install htop    # Debian/Ubuntu
sudo yum install htop    # RHEL/CentOS

# Launch htop
htop

# Features:
# - Colored output
# - Mouse support
# - Horizontal and vertical scrolling
# - Tree view
# - Easy process killing
```

## 4.3 Process Control

### Starting Processes

#### Foreground Processes
```bash
# Run in foreground (blocks terminal)
command

# Example
sleep 100
```

#### Background Processes
```bash
# Run in background (frees terminal)
command &

# Example
sleep 100 &

# View background jobs
jobs

# Bring job to foreground
fg %1

# Send job to background
bg %1
```

### Stopping Processes

#### Ctrl+C
```bash
# Send SIGINT (interrupt signal)
# Run a command and press Ctrl+C to stop
ping google.com
# Press Ctrl+C
```

#### Ctrl+Z
```bash
# Send SIGTSTP (suspend signal)
# Run a command and press Ctrl+Z to suspend
ping google.com
# Press Ctrl+Z
# Resume with 'fg' or 'bg'
```

### kill Command
```bash
# Send SIGTERM (graceful termination)
kill PID

# Send specific signal
kill -SIGNAL PID

# Force kill (SIGKILL)
kill -9 PID
kill -SIGKILL PID

# Kill all processes by name
killall process_name

# Kill all processes matching pattern
pkill pattern

# Examples
kill 1234
kill -9 1234
killall firefox
pkill -u username
```

### Common Signals
```bash
Signal  Number  Description
SIGHUP    1     Hangup (reload configuration)
SIGINT    2     Interrupt (Ctrl+C)
SIGQUIT   3     Quit
SIGKILL   9     Kill (cannot be caught)
SIGTERM  15     Terminate (graceful shutdown)
SIGSTOP  19     Stop process
SIGCONT  18     Continue process
```

## 4.4 Process Priority

### nice Command
```bash
# Nice values: -20 (highest priority) to 19 (lowest)
# Default is 0

# Start process with nice value
nice -n 10 command

# Examples
nice -n 10 tar -czf backup.tar.gz /home
nice -n -5 important_task  # Requires root for negative values
```

### renice Command
```bash
# Change priority of running process
renice -n 5 -p PID

# Renice all processes for user
renice -n 10 -u username

# Examples
renice -n 5 -p 1234
renice -n 10 -u john
```

## 4.5 Background Job Management

### nohup Command
```bash
# Run process immune to hangup signal
nohup command &

# Output goes to nohup.out by default
nohup ./long_script.sh &

# Redirect output
nohup ./script.sh > output.log 2>&1 &
```

### disown Command
```bash
# Remove job from shell's job table
command &
disown %1

# Disown all jobs
disown -a
```

### Screen (Terminal Multiplexer)
```bash
# Install screen
sudo apt install screen

# Start new screen session
screen

# Create named session
screen -S mysession

# Detach: Ctrl+A, then D

# List sessions
screen -ls

# Reattach to session
screen -r
screen -r mysession

# Kill session
screen -X -S mysession quit
```

### tmux (Modern Terminal Multiplexer)
```bash
# Install tmux
sudo apt install tmux

# Start new session
tmux

# Create named session
tmux new -s mysession

# Detach: Ctrl+B, then D

# List sessions
tmux ls

# Attach to session
tmux attach
tmux attach -t mysession

# Kill session
tmux kill-session -t mysession

# Common tmux commands:
# Ctrl+B % : Split vertically
# Ctrl+B " : Split horizontally
# Ctrl+B arrow : Switch panes
# Ctrl+B c : Create new window
# Ctrl+B n : Next window
# Ctrl+B p : Previous window
```

## 4.6 System Monitoring

### uptime
```bash
# Show system uptime and load average
uptime

# Output example:
# 10:30:45 up 5 days, 2:15, 3 users, load average: 0.45, 0.52, 0.48
```

### free
```bash
# Display memory usage
free

# Human-readable format
free -h

# Continuous monitoring (every 2 seconds)
free -h -s 2
```

### vmstat
```bash
# Virtual memory statistics
vmstat

# Update every 2 seconds
vmstat 2

# Update 5 times every 2 seconds
vmstat 2 5
```

### iostat
```bash
# Install sysstat package
sudo apt install sysstat

# CPU and I/O statistics
iostat

# Extended statistics
iostat -x

# Update every 2 seconds
iostat 2
```

## 4.7 Process Information

### /proc Filesystem
```bash
# View process information
cat /proc/PID/status
cat /proc/PID/cmdline
cat /proc/PID/environ

# Examples
cat /proc/1/status        # Init process info
cat /proc/self/status     # Current shell info
```

### lsof (List Open Files)
```bash
# Install lsof
sudo apt install lsof

# List all open files
lsof

# Files opened by specific process
lsof -p PID

# Files opened by user
lsof -u username

# Network connections
lsof -i

# Specific port
lsof -i :80
lsof -i :22

# Files in directory
lsof +D /var/log
```

### pgrep and pidof
```bash
# Find process ID by name
pgrep process_name

# Show full command line
pgrep -a process_name

# Find PID by exact name
pidof process_name

# Examples
pgrep firefox
pgrep -a python
pidof sshd
```

## 4.8 System Resource Limits

### ulimit
```bash
# Show all limits
ulimit -a

# Show specific limit
ulimit -n    # Open files
ulimit -u    # Max processes

# Set soft limit
ulimit -n 4096

# Set hard limit (requires root)
ulimit -H -n 65536

# Common limits:
# -c : Core file size
# -d : Process data segment
# -f : File size
# -n : Open files
# -t : CPU time
# -u : User processes
# -v : Virtual memory
```

### systemd-cgtop
```bash
# Show resource usage by control group
systemd-cgtop

# Sort by CPU
systemd-cgtop --order=cpu

# Sort by memory
systemd-cgtop --order=memory
```

## 4.9 Practical Examples

### Find and Kill Resource-Heavy Processes
```bash
# Find top 10 CPU consumers
ps aux --sort=-%cpu | head -10

# Find top 10 memory consumers
ps aux --sort=-%mem | head -10

# Kill all chrome processes
pkill -9 chrome

# Kill processes using more than 50% CPU
ps aux | awk '$3 > 50 {print $2}' | xargs kill
```

### Monitor Specific Process
```bash
# Continuous monitoring
watch -n 1 'ps -p PID -o pid,ppid,%cpu,%mem,cmd'

# Monitor process tree
watch -n 1 'pstree -p PID'

# Check if process is running
pgrep -x process_name || echo "Not running"
```

### Background Task Management
```bash
# Start long-running task
nohup python train_model.py > training.log 2>&1 &

# Check progress
tail -f training.log

# Find PID
pgrep -f train_model

# Check resource usage
ps -p $(pgrep -f train_model) -o pid,%cpu,%mem,cmd
```

### Cleanup Zombie Processes
```bash
# Find zombie processes
ps aux | grep 'Z'

# Kill parent process (zombies will be adopted by init)
kill -9 PPID

# Or restart the parent process gracefully
```

## 4.10 Advanced Process Management

### systemd Process Management
```bash
# View service status
systemctl status service_name

# Start service
sudo systemctl start service_name

# Stop service
sudo systemctl stop service_name

# Restart service
sudo systemctl restart service_name

# Enable service at boot
sudo systemctl enable service_name

# View all running services
systemctl list-units --type=service --state=running
```

### cgroups (Control Groups)
```bash
# View cgroup hierarchy
systemd-cgls

# Check cgroup of process
cat /proc/PID/cgroup

# Resource limits with systemd
sudo systemctl set-property service_name CPUQuota=50%
sudo systemctl set-property service_name MemoryLimit=1G
```

## Practice Exercises

### Exercise 1: Process Monitoring
1. Open three terminals
2. In terminal 1: Run `top`
3. In terminal 2: Run `stress --cpu 2 --timeout 60`
4. In terminal 3: Monitor with `ps aux | grep stress`
5. Observe CPU usage changes

### Exercise 2: Job Control
1. Start a long process: `sleep 300`
2. Suspend it with Ctrl+Z
3. Put it in background: `bg`
4. List jobs: `jobs`
5. Bring to foreground: `fg`
6. Kill it: Ctrl+C

### Exercise 3: Process Priority
1. Start two processes with different nice values:
   ```bash
   nice -n 10 sha256sum /dev/zero &
   nice -n 5 sha256sum /dev/zero &
   ```
2. Compare CPU usage with `top`
3. Change priority with `renice`

### Exercise 4: Signal Handling
Write a bash script that traps signals:
```bash
#!/bin/bash
trap "echo 'Caught SIGTERM'; exit" SIGTERM
trap "echo 'Caught SIGINT'; exit" SIGINT

while true; do
    echo "Running..."
    sleep 2
done
```

Test different signals: `kill -TERM PID`, `kill -INT PID`

### Exercise 5: Resource Monitoring
1. Find the top 5 memory-consuming processes
2. Identify processes listening on port 80
3. List all files opened by your user
4. Check current system load average

## Key Takeaways

- **ps**: Static snapshot of processes
- **top/htop**: Real-time process monitoring
- **kill**: Send signals to processes
- **nice/renice**: Adjust process priority
- **nohup/screen/tmux**: Run persistent background tasks
- **jobs/fg/bg**: Manage shell jobs
- **Signals**: Different ways to communicate with processes
- **Resource limits**: Control process resource usage

## Next Steps

- Learn systemd service management
- Explore cgroups and resource limits
- Master tmux for terminal multiplexing
- Study process scheduling algorithms
- Practice shell scripting with job control
