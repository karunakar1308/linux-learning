# Chapter 05: Shell Scripting

## 5.1 Introduction to Shell Scripting

### What is a Shell Script?
A shell script is a file containing a series of Linux commands, executed sequentially by the shell interpreter. It's a way to automate repetitive tasks.

### Advantages of Shell Scripting
- Automation: Run complex tasks with a single command
- Scheduling: Use cron to run scripts automatically
- Portability: Run on any Unix-like system with a shell
- Simplicity: Easy to learn and understand
- Integration: Combine multiple commands and tools

### Which Shell?
```bash
# Bash (Most common)
#!/bin/bash

# Sh (POSIX compatible)
#!/bin/sh

# Zsh
#!/bin/zsh

# Ksh
#!/bin/ksh
```

## 5.2 Script Basics

### Creating Your First Script
```bash
#!/bin/bash
# This is a comment
echo "Hello, World!"
```

### Making Script Executable
```bash
# Add execute permission
chmod +x script.sh

# Run the script
./script.sh

# Or use bash directly
bash script.sh
```

### Shebang (#!)
```bash
#!/bin/bash
# Must be first line (no spaces before #!)
# Tells system which interpreter to use
# Can check with: head -1 script.sh
```

## 5.3 Variables

### Declaring Variables
```bash
# Simple assignment (no spaces around =)
NAME="John"
AGE=25
FILE=/etc/passwd

# From command output
DATE=$(date)
USERS=$(cat /etc/passwd | wc -l)
```

### Using Variables
```bash
#!/bin/bash
NAME="Alice"
echo $NAME
echo ${NAME}  # Preferred way
echo "Hello, $NAME"
echo "Hello, ${NAME}!"
```

### Variable Scope
```bash
#!/bin/bash
# Local variable (default)
LOCAL_VAR="local"

# Export to subshells
export EXPORTED_VAR="exported"

# Display environment variables
env | grep VAR
```

### Special Variables
```bash
$0        # Script name
$1, $2    # Arguments (positional parameters)
$@        # All arguments
$#        # Number of arguments
$?        # Exit status of last command
$$        # Process ID (PID)
$!        # PID of last background process
```

## 5.4 User Input

### read Command
```bash
#!/bin/bash
# Simple read
read VAR
echo "You entered: $VAR"

# With prompt
read -p "Enter your name: " NAME
echo "Hello, $NAME"

# Silent input (password)
read -sp "Enter password: " PASSWORD
echo ""

# Read multiple values
read FIRST LAST
echo "$FIRST $LAST"

# Read array
read -a ARRAY
echo "${ARRAY[0]}"
```

### Command-line Arguments
```bash
#!/bin/bash
# Script: ./script.sh arg1 arg2 arg3

echo "Script name: $0"
echo "First arg: $1"
echo "Second arg: $2"
echo "All args: $@"
echo "Number of args: $#"

# Process all arguments
for arg in "$@"; do
    echo "Argument: $arg"
done
```

## 5.5 Conditional Statements

### if Statement
```bash
#!/bin/bash
if [ condition ]; then
    # Do something
elif [ condition ]; then
    # Do something else
else
    # Default action
fi
```

### Test Conditions
```bash
# File tests
[ -f file ]       # File exists
[ -d directory ]  # Directory exists
[ -s file ]       # File exists and not empty
[ -x file ]       # File is executable
[ -r file ]       # File is readable
[ -w file ]       # File is writable

# String tests
[ -z string ]     # String is empty
[ -n string ]     # String is not empty
[ s1 = s2 ]       # Strings are equal
[ s1 != s2 ]      # Strings are not equal

# Numeric tests
[ num1 -eq num2 ] # Equal
[ num1 -ne num2 ] # Not equal
[ num1 -lt num2 ] # Less than
[ num1 -le num2 ] # Less than or equal
[ num1 -gt num2 ] # Greater than
[ num1 -ge num2 ] # Greater than or equal
```

### Example
```bash
#!/bin/bash
if [ $# -eq 0 ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

if [ -f "$1" ]; then
    echo "File exists"
    if [ -s "$1" ]; then
        echo "File is not empty"
    else
        echo "File is empty"
    fi
else
    echo "File does not exist"
fi
```

## 5.6 Loops

### for Loop
```bash
# Loop through list
for item in one two three; do
    echo "Item: $item"
done

# Loop through files
for file in *.txt; do
    echo "File: $file"
done

# C-style for loop
for ((i=0; i<5; i++)); do
    echo "Number: $i"
done

# Loop through array
ARRAY=("apple" "banana" "cherry")
for item in "${ARRAY[@]}"; do
    echo "Fruit: $item"
done
```

### while Loop
```bash
#!/bin/bash
# Basic while
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))
done

# Read file line by line
while read line; do
    echo "Line: $line"
done < /etc/hostname
```

### until Loop
```bash
#!/bin/bash
# Repeat until condition is true
COUNT=1
until [ $COUNT -gt 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))
done
```

### Loop Control
```bash
# Break out of loop
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        break
    fi
    echo $i
done

# Skip to next iteration
for i in {1..5}; do
    if [ $i -eq 3 ]; then
        continue
    fi
    echo $i
done
```

## 5.7 Functions

### Function Definition
```bash
#!/bin/bash
# Method 1
function greet() {
    echo "Hello, $1"
}

# Method 2
greet() {
    echo "Hello, $1"
}

# Call function
greet "Alice"
greet "Bob"
```

### Function Parameters and Return
```bash
#!/bin/bash
add() {
    local A=$1
    local B=$2
    local SUM=$((A + B))
    return $SUM
}

add 5 3
echo "Sum: $?"  # Note: exit codes are 0-255

# Better approach with echo
add() {
    local A=$1
    local B=$2
    echo $((A + B))
}

RESULT=$(add 5 3)
echo "Sum: $RESULT"
```

## 5.8 Advanced Features

### Arrays
```bash
#!/bin/bash
# Declare array
FRUITS=("apple" "banana" "cherry")

# Access elements
echo ${FRUITS[0]}      # First element
echo ${FRUITS[@]}      # All elements
echo ${#FRUITS[@]}     # Array length

# Add element
FRUITS+=("orange")

# Loop through array
for fruit in "${FRUITS[@]}"; do
    echo "Fruit: $fruit"
done
```

### String Manipulation
```bash
#!/bin/bash
STRING="Hello World"

# Length
echo ${#STRING}

# Substring
echo ${STRING:0:5}     # "Hello"

# Replace
echo ${STRING//World/Linux}

# Default value
echo ${VAR:-default}
```

### Arithmetic
```bash
#!/bin/bash
# Method 1
NUM=$((5 + 3))

# Method 2
NUM=$[5 + 3]

# Method 3
NUM=$(expr 5 + 3)

# Increment
((NUM++))
```

### Case Statement
```bash
#!/bin/bash
read -p "Enter fruit: " FRUIT

case $FRUIT in
    apple)
        echo "Apple is red"
        ;;
    banana)
        echo "Banana is yellow"
        ;;
    *)
        echo "Unknown fruit"
        ;;
esac
```

## 5.9 Error Handling

### Exit Status
```bash
#!/bin/bash
# 0 = success, non-zero = failure

command
if [ $? -eq 0 ]; then
    echo "Success"
else
    echo "Failed"
fi

# Check exit status of pipes
command1 | command2
# Only checks command2's status
```

### Exit Command
```bash
#!/bin/bash
if [ $# -eq 0 ]; then
    echo "Error: No arguments"
    exit 1
fi

# Rest of script
exit 0
```

### Error Messages
```bash
#!/bin/bash
# Output to stderr
echo "Error message" >&2

# Conditional exit
[ -f "$1" ] || exit 1

# Trap errors
trap 'echo "Error on line $LINENO"' ERR
```

## 5.10 Practical Examples

### Backup Script
```bash
#!/bin/bash
SOURCE="/home/user"
DEST="/backup"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

if [ ! -d "$SOURCE" ]; then
    echo "Source directory not found"
    exit 1
fi

tar -czf "$DEST/backup_$TIMESTAMP.tar.gz" "$SOURCE"
echo "Backup completed: $DEST/backup_$TIMESTAMP.tar.gz"
```

### File Processor
```bash
#!/bin/bash
if [ $# -eq 0 ]; then
    echo "Usage: $0 <directory>"
    exit 1
fi

for file in "$1"/*.txt; do
    if [ -f "$file" ]; then
        echo "Processing: $file"
        # Do something with file
    fi
done
```

### System Monitor
```bash
#!/bin/bash
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}')
MEMORY=$(free -h | awk 'NR==2 {print $3}')

echo "CPU Usage: $CPU_USAGE"
echo "Memory Usage: $MEMORY"

if (( ${CPU_USAGE%\%} > 80 )); then
    echo "WARNING: High CPU usage"
fi
```

### Interactive Menu
```bash
#!/bin/bash
while true; do
    echo "===== Main Menu ====="
    echo "1. List files"
    echo "2. Show date"
    echo "3. Exit"
    read -p "Choose option: " CHOICE

    case $CHOICE in
        1) ls -la ;;
        2) date ;;
        3) echo "Goodbye"; exit 0 ;;
        *) echo "Invalid choice" ;;
    esac
done
```

## Practice Exercises

### Exercise 1: Greeting Script
Create `greet.sh` that:
1. Takes a name as argument
2. Greets the user
3. Shows current time

### Exercise 2: File Backup
Create `backup.sh` that:
1. Takes source and destination directories
2. Creates timestamped tar archive
3. Verifies success

### Exercise 3: System Report
Create `report.sh` that:
1. Shows CPU usage
2. Shows memory usage
3. Shows disk usage
4. Checks if any service is down

### Exercise 4: Password Validator
Create `validate_pwd.sh` that:
1. Reads password
2. Validates it (min 8 chars, must have numbers)
3. Confirms password matches

### Exercise 5: Log Analyzer
Create `analyze_log.sh` that:
1. Takes log file as argument
2. Shows line count
3. Shows unique IPs
4. Shows error count

## Key Takeaways

- **Shebang**: Must be first line of script
- **Variables**: No spaces around = sign
- **Test conditions**: Use [ ] with proper syntax
- **Loops**: for, while, until with break/continue
- **Functions**: Reusable code blocks
- **Error handling**: Check exit status and return codes
- **Debugging**: Use `bash -x` script.sh to trace execution
- **Comments**: Use # to document code

## Next Steps

- Practice writing scripts daily
- Learn debugging techniques (set -x, set -e)
- Explore sed and awk for advanced text processing
- Study cron for scheduling scripts
- Learn about package managers and installation scripts
