# Chapter 03: Text Processing & Editors

## 3.1 Text Editors

### Nano Editor (Beginner-Friendly)
```bash
# Open or create file
nano filename.txt

# Common shortcuts:
# Ctrl+O : Save file
# Ctrl+X : Exit
# Ctrl+K : Cut line
# Ctrl+U : Paste
# Ctrl+W : Search
# Ctrl+\ : Find and replace
```

### Vi/Vim Editor (Advanced)
```bash
# Open file
vim filename.txt

# Vim has 3 modes:
# 1. Command mode (default)
# 2. Insert mode (press 'i')
# 3. Visual mode (press 'v')
```

#### Vim Command Mode Operations
```bash
# Navigation:
h, j, k, l    # Left, down, up, right
w             # Next word
b             # Previous word
0             # Beginning of line
$             # End of line
gg            # First line
G             # Last line
:n            # Go to line n

# Editing:
i             # Insert before cursor
a             # Insert after cursor
o             # New line below
O             # New line above
dd            # Delete line
yy            # Copy line
p             # Paste after
P             # Paste before
u             # Undo
Ctrl+r        # Redo

# Search and Replace:
/pattern      # Search forward
?pattern      # Search backward
n             # Next occurrence
N             # Previous occurrence
:%s/old/new/g # Replace all occurrences

# Save and Exit:
:w            # Save
:q            # Quit
:wq or :x     # Save and quit
:q!           # Quit without saving
```

## 3.2 Text Processing Commands

### cat - Concatenate and Display
```bash
# Display file
cat file.txt

# Display with line numbers
cat -n file.txt

# Concatenate multiple files
cat file1.txt file2.txt > combined.txt

# Create file
cat > newfile.txt
# Type content, press Ctrl+D to save
```

### head and tail
```bash
# First 10 lines
head file.txt

# First 20 lines
head -n 20 file.txt

# Last 10 lines
tail file.txt

# Last 20 lines
tail -n 20 file.txt

# Follow file (real-time monitoring)
tail -f /var/log/syslog
```

### more and less
```bash
# View file page by page
more file.txt

# Better pager with more features
less file.txt

# less commands:
# Space : Next page
# b     : Previous page
# /     : Search forward
# ?     : Search backward
# q     : Quit
```

## 3.3 grep - Pattern Matching

### Basic grep Usage
```bash
# Search for pattern
grep "pattern" file.txt

# Case-insensitive search
grep -i "pattern" file.txt

# Show line numbers
grep -n "pattern" file.txt

# Search recursively
grep -r "pattern" /path/to/directory

# Invert match (show non-matching lines)
grep -v "pattern" file.txt

# Count matches
grep -c "pattern" file.txt

# Show only matching part
grep -o "pattern" file.txt
```

### Advanced grep
```bash
# Multiple patterns
grep -e "pattern1" -e "pattern2" file.txt

# Extended regex
grep -E "pattern1|pattern2" file.txt

# Show context (2 lines before and after)
grep -C 2 "pattern" file.txt

# Only show filenames
grep -l "pattern" *.txt

# Exclude files
grep --exclude="*.log" "pattern" *
```

## 3.4 sed - Stream Editor

### Basic sed Operations
```bash
# Replace first occurrence in each line
sed 's/old/new/' file.txt

# Replace all occurrences
sed 's/old/new/g' file.txt

# Edit file in place
sed -i 's/old/new/g' file.txt

# Delete lines
sed '3d' file.txt              # Delete line 3
sed '2,5d' file.txt            # Delete lines 2-5
sed '/pattern/d' file.txt      # Delete lines matching pattern

# Print specific lines
sed -n '5p' file.txt           # Print line 5
sed -n '10,20p' file.txt       # Print lines 10-20
sed -n '/pattern/p' file.txt   # Print matching lines
```

### Advanced sed
```bash
# Multiple operations
sed -e 's/old/new/g' -e 's/foo/bar/g' file.txt

# Insert line before match
sed '/pattern/i New line' file.txt

# Append line after match
sed '/pattern/a New line' file.txt

# Replace on specific lines
sed '5s/old/new/' file.txt     # Only line 5
```

## 3.5 awk - Text Processing Language

### Basic awk
```bash
# Print specific columns
awk '{print $1}' file.txt      # First column
awk '{print $1, $3}' file.txt  # First and third columns
awk '{print $NF}' file.txt     # Last column

# With custom delimiter
awk -F: '{print $1}' /etc/passwd

# Print with condition
awk '$3 > 100' file.txt        # Print if 3rd field > 100
```

### Advanced awk
```bash
# Pattern matching
awk '/pattern/ {print $0}' file.txt

# BEGIN and END blocks
awk 'BEGIN {print "Start"} {print $1} END {print "Done"}' file.txt

# Calculate sum
awk '{sum+=$1} END {print sum}' numbers.txt

# Count lines
awk 'END {print NR}' file.txt

# Multiple conditions
awk '$1 > 10 && $2 < 50 {print $0}' file.txt

# Custom output
awk '{printf "Name: %s, Age: %d\n", $1, $2}' file.txt
```

## 3.6 cut - Extract Sections

```bash
# Extract characters
cut -c 1-5 file.txt            # Characters 1-5

# Extract fields (tab-delimited by default)
cut -f 1,3 file.txt            # Fields 1 and 3

# Custom delimiter
cut -d: -f1 /etc/passwd        # First field with : delimiter

# Extract range
cut -d: -f1-3 /etc/passwd      # Fields 1 through 3
```

## 3.7 sort and uniq

### sort Command
```bash
# Alphabetical sort
sort file.txt

# Reverse sort
sort -r file.txt

# Numeric sort
sort -n numbers.txt

# Sort by specific column
sort -k 2 file.txt             # Sort by 2nd column

# Unique sort
sort -u file.txt
```

### uniq Command
```bash
# Remove duplicate consecutive lines
uniq file.txt

# Count occurrences
uniq -c file.txt

# Show only duplicates
uniq -d file.txt

# Show only unique lines
uniq -u file.txt

# Common usage (must sort first)
sort file.txt | uniq
```

## 3.8 wc - Word Count

```bash
# Count lines, words, characters
wc file.txt

# Only lines
wc -l file.txt

# Only words
wc -w file.txt

# Only characters
wc -c file.txt

# Multiple files
wc *.txt
```

## 3.9 tr - Translate Characters

```bash
# Convert lowercase to uppercase
tr 'a-z' 'A-Z' < file.txt

# Delete characters
tr -d '0-9' < file.txt         # Remove digits

# Squeeze repeats
tr -s ' ' < file.txt           # Remove duplicate spaces

# Replace characters
echo "hello" | tr 'el' 'ip'    # hippo
```

## 3.10 Practical Examples

### Log Analysis
```bash
# Find error messages
grep -i "error" /var/log/syslog

# Count errors by type
grep "ERROR" app.log | awk '{print $5}' | sort | uniq -c

# Find top 10 IP addresses in access log
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10

# Extract failed login attempts
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c
```

### Data Processing
```bash
# Extract email addresses
grep -oE '\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b' file.txt

# Convert CSV to tab-delimited
sed 's/,/\t/g' file.csv

# Remove empty lines
sed '/^$/d' file.txt

# Add line numbers
awk '{print NR ": " $0}' file.txt
```

### System Administration
```bash
# List users
cut -d: -f1 /etc/passwd | sort

# Find large files
find /home -type f -size +100M | sort -h

# Monitor log file for pattern
tail -f /var/log/syslog | grep --color "ERROR"
```

## Practice Exercises

### Exercise 1: Vim Mastery
1. Create a file with vim: `vim practice.txt`
2. Enter insert mode and type 5 lines of text
3. Copy line 2 and paste it at the end
4. Delete line 3
5. Search for a word and replace all occurrences
6. Save and exit

### Exercise 2: Text Processing Pipeline
Create a file `users.txt` with this content:
```
John:Sales:50000
Mary:IT:75000
Bob:HR:45000
Alice:IT:80000
Tom:Sales:55000
```

Tasks:
1. Extract only names: `cut -d: -f1 users.txt`
2. List IT department: `grep "IT" users.txt`
3. Find highest salary: `sort -t: -k3 -nr users.txt | head -1`
4. Count employees per department: `cut -d: -f2 users.txt | sort | uniq -c`

### Exercise 3: Log Analysis
Create a sample log file and:
1. Count total lines
2. Find all ERROR entries
3. Extract unique error types
4. Show last 20 entries in real-time

### Exercise 4: Advanced sed
1. Replace all occurrences of "error" with "ERROR"
2. Delete all blank lines
3. Number all lines
4. Insert a header line at the beginning

### Exercise 5: awk Programming
Process a file with columns: name, age, salary
1. Calculate average salary
2. Find people older than 30
3. Print formatted output
4. Calculate total and average

## Key Takeaways

- **Nano**: Best for beginners, straightforward interface
- **Vim**: Powerful but steep learning curve, worth mastering
- **grep**: Essential for searching text and patterns
- **sed**: Stream editor for text transformation
- **awk**: Full programming language for text processing
- **Pipes**: Combine commands for powerful text processing
- **Regular Expressions**: Learn regex for advanced pattern matching

## Next Steps

- Practice vim daily to build muscle memory
- Learn regular expressions thoroughly
- Combine commands with pipes for complex tasks
- Study real-world log files for practical experience
- Explore advanced awk programming
