# Chapter 10: Advanced Shell Scripting

## Overview
This chapter covers advanced shell scripting techniques including functions, arrays, error handling, text processing, and optimization strategies for production scripts.

## Functions and Modularization

### Defining Functions
```bash
# Basic function
function greet() {
  echo "Hello, $1!"
}

# Alternative syntax
greet() {
  echo "Hello, $1!"
}

# Function with local variables
process_file() {
  local filename=$1
  local count=0
  echo "Processing: $filename"
}
```

### Function Parameters and Return Values
```bash
# Access all parameters
all_params() {
  echo "All args: $@"
  echo "Number of args: $#"
  echo "First arg: $1"
  echo "Script name: $0"
}

# Return values
get_status() {
  if command -v git &> /dev/null; then
    return 0  # Success
  else
    return 1  # Failure
  fi
}

if get_status; then
  echo "Git is installed"
fi
```

## Arrays and Complex Data Structures

### Indexed Arrays
```bash
# Declare array
declare -a my_array

# Initialize array
my_array=(one two three four)
echo "${my_array[@]}"      # All elements
echo "${my_array[2]}"      # Single element
echo "${#my_array[@]}"     # Array length

# Add elements
my_array+=(five six)

# Loop through array
for item in "${my_array[@]}"; do
  echo "$item"
done
```

### Associative Arrays
```bash
# Declare associative array
declare -A person
person[name]="John"
person[age]=30
person[city]="NYC"

# Access values
echo "${person[name]}"
echo "${person[age]}"

# Loop through array
for key in "${!person[@]}"; do
  echo "$key: ${person[$key]}"
done
```

## Advanced Control Structures

### Case Statements
```bash
case_example() {
  case "$1" in
    start)
      echo "Starting service..."
      ;;
    stop)
      echo "Stopping service..."
      ;;
    restart)
      echo "Restarting service..."
      ;;
    *)
      echo "Usage: $0 {start|stop|restart}"
      return 1
      ;;
  esac
}
```

### Select Menu
```bash
select_menu() {
  PS3="Choose an option: "
  select opt in "Option 1" "Option 2" "Option 3" "Quit"; do
    case $opt in
      "Option 1")
        echo "You selected Option 1"
        break
        ;;
      "Quit")
        echo "Exiting..."
        break
        ;;
      *)
        echo "Invalid option"
        ;;
    esac
  done
}
```

## Error Handling and Debugging

### Error Handling Techniques
```bash
# Set error handling flags
set -e      # Exit on error
set -u      # Exit on undefined variable
set -o pipefail  # Exit on pipe failure
set -x      # Print commands (debugging)

# Manual error checking
if ! command_that_might_fail; then
  echo "Command failed with status: $?"
  exit 1
fi

# Error trap
trap 'echo "Error on line $LINENO"' ERR
trap 'echo "Script interrupted"; exit' INT

# Cleanup trap
cleanup() {
  rm -f /tmp/tempfile
  echo "Cleanup complete"
}
trap cleanup EXIT
```

### Conditional Execution
```bash
# Command chaining
command1 && command2  # Execute command2 only if command1 succeeds
command1 || command2  # Execute command2 only if command1 fails

# Subshell grouping
(cd /tmp && ls)       # Run in subshell, doesn't affect current directory
{ cd /tmp && ls; }    # Run in current shell
```

## Text Processing and Parsing

### Regular Expressions
```bash
# Basic pattern matching
if [[ "$string" =~ ^[0-9]+$ ]]; then
  echo "String contains only digits"
fi

# Extract matches
string="Email: test@example.com"
if [[ $string =~ ([a-z]+)@([a-z.]+) ]]; then
  echo "User: ${BASH_REMATCH[1]}"
  echo "Domain: ${BASH_REMATCH[2]}"
fi
```

### String Manipulation
```bash
# Remove prefix/suffix
string="prefix_value_suffix"
echo "${string#prefix_}"      # Remove prefix
echo "${string%_suffix}"      # Remove suffix
echo "${string//prefix/new}"  # Replace all

# Case conversion
uppercase="${string^^}"
lowercase="${string,,}"

# String length and slicing
echo "${#string}"             # Length
echo "${string:0:5}"          # First 5 characters
echo "${string: -3}"          # Last 3 characters
```

### AWK and SED
```bash
# AWK - Field processing
awk '{print $1, $3}' file.txt

# AWK - Pattern matching
awk '/pattern/ {print $0}' file.txt

# SED - Substitution
sed 's/old/new/g' file.txt

# SED - Delete lines
sed '/pattern/d' file.txt

# SED - In-place editing
sed -i.bak 's/old/new/g' file.txt
```

## Process and Signal Handling

### Background Processes
```bash
# Run in background
long_running_command &

# Get process IDs
echo $!                  # Last background PID
jobs                     # List jobs
jobs -p                  # List PIDs

# Wait for background process
wait $pid
echo "Process completed with status: $?"

# Run processes in parallel
for i in {1..5}; do
  process_task $i &
done
wait
```

### Signal Handling
```bash
# Handle signals
handler() {
  echo "Received signal: $1"
}

trap 'handler TERM' TERM
trap 'handler INT' INT

# Send signals
kill -TERM $pid
kill -9 $pid  # Force kill
```

## Advanced I/O Operations

### Input/Output Redirection
```bash
# Redirect to file
command > file.txt       # Stdout to file
command >> file.txt      # Append to file
command 2> errors.txt    # Stderr to file
command &> output.txt    # Both stdout and stderr

# Redirect from file
command < input.txt

# Here documents
cat << EOF
This is a multi-line
string with variables: $HOME
EOF

# Here strings
command <<< "single line string"
```

### File Descriptors
```bash
# Open file for reading
exec 3< file.txt
read -u 3 line
exec 3<&-    # Close file descriptor

# Duplicate file descriptors
exec 3>&1    # Duplicate stdout to FD 3
echo "normal"
exec 1> /tmp/output.txt
echo "redirected"
exec 1>&3    # Restore stdout
```

## Performance Optimization

### Efficient Scripting Patterns
```bash
# Avoid subshells
# Bad:
for file in *.txt; do
  size=$(stat -f%z "$file")  # Spawns process for each file
done

# Good:
for file in *.txt; do
  [[ -f "$file" ]] && size=${#file}
done

# Use built-in commands
# Bad:
name=$(basename "$file")
ext=$(echo "$name" | awk -F. '{print $NF}')

# Good:
name="${file##*/}"
ext="${name##*.}"
```

### Profiling Scripts
```bash
# Time script execution
time ./script.sh

# Profile with bash debug
bash -x ./script.sh

# Detailed timing
PS4='+ $(date "+%s.%N"): '
bash -x ./script.sh
```

## Practice Exercises

1. **Functions and Arrays**:
   - Create a function that processes an array of numbers
   - Return sum, average, and count of numbers
   - Use local variables properly

2. **Error Handling**:
   - Write a script with comprehensive error handling
   - Implement trap handlers for EXIT and ERR
   - Test various failure scenarios

3. **Text Processing**:
   - Parse a CSV file using awk/sed
   - Extract specific fields and transform data
   - Handle edge cases

4. **Process Management**:
   - Create a script that runs multiple background tasks
   - Implement proper cleanup on script termination
   - Monitor and report task completion

5. **Advanced I/O**:
   - Implement a log rotation script
   - Use file descriptors for complex I/O operations
   - Handle large file processing efficiently

## Best Practices

- Always quote variables: "$var" not $var
- Use local variables in functions
- Enable error checking: set -e, set -u
- Use meaningful variable names
- Add comments and documentation
- Test error conditions
- Avoid unnecessary subshells
- Use built-in commands when possible

## Key Takeaways
- Functions enable code reuse and modularity
- Arrays provide powerful data structure capabilities
- Proper error handling is crucial for production scripts
- Text processing commands (awk, sed) are powerful tools
- Process management allows sophisticated automation
- Performance considerations matter in scripts
