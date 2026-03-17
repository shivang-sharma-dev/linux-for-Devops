# 06 — Shell Scripting

> Shell scripting is the single most valuable skill in this repo. Every repetitive DevOps task — backups, deployments, health checks, cleanup jobs — gets automated with a shell script.

---

## The Shebang Line

Every script starts with a shebang — tells the system which interpreter to use:

```bash
#!/bin/bash         # use bash (most common)
#!/bin/sh           # use POSIX sh (more portable, fewer features)
#!/usr/bin/env bash # find bash wherever it is (better for portability)
```

```bash
# Make a script executable
chmod +x script.sh
./script.sh         # run it
bash script.sh      # run without execute permission
```

---

## Variables

```bash
#!/bin/bash

# Assign (no spaces around =)
name="alice"
age=30
path="/var/log/nginx"

# Use ($ prefix)
echo "Hello, $name"
echo "Age: $age"
echo "Path: $path"

# Curly braces (safer, needed for concatenation)
echo "Hello, ${name}!"
echo "${path}/access.log"

# Command substitution (store command output in variable)
current_date=$(date +%Y-%m-%d)
disk_usage=$(df -h / | awk 'NR==2{print $5}')
hostname=$(hostname)

echo "Date: $current_date"
echo "Disk: $disk_usage"
```

### Special variables

```bash
$0          # script name
$1, $2...   # positional arguments
$@          # all arguments as separate words
$*          # all arguments as a single string
$#          # number of arguments
$?          # exit code of last command (0 = success)
$$          # current script's PID
$!          # PID of last background command
$HOME       # home directory
$USER       # current username
$HOSTNAME   # machine hostname
$PATH       # executable search path
```

---

## User Input

```bash
# Read from user
read name
echo "Hello, $name"

# Read with prompt
read -p "Enter username: " username
read -sp "Enter password: " password   # -s = silent (no echo)
echo ""    # newline after silent input

# Read with timeout
read -t 10 -p "Answer in 10 seconds: " answer

# Read into array
read -a fruits <<< "apple banana cherry"
echo "${fruits[0]}"    # apple
```

---

## Conditionals

```bash
# if / elif / else
if [ condition ]; then
    commands
elif [ other_condition ]; then
    commands
else
    commands
fi

# Examples
if [ "$USER" = "root" ]; then
    echo "Running as root"
fi

if [ -f "/etc/nginx/nginx.conf" ]; then
    echo "Nginx config exists"
else
    echo "Nginx not configured"
fi

age=25
if [ $age -ge 18 ]; then
    echo "Adult"
fi
```

### Test conditions — single bracket `[ ]`

```bash
# String tests
[ "$a" = "$b" ]     # equal
[ "$a" != "$b" ]    # not equal
[ -z "$a" ]         # empty string
[ -n "$a" ]         # non-empty string

# Numeric tests
[ $a -eq $b ]       # equal
[ $a -ne $b ]       # not equal
[ $a -lt $b ]       # less than
[ $a -le $b ]       # less than or equal
[ $a -gt $b ]       # greater than
[ $a -ge $b ]       # greater than or equal

# File tests
[ -f file ]         # is a regular file
[ -d dir ]          # is a directory
[ -e path ]         # exists (file or dir)
[ -r file ]         # readable
[ -w file ]         # writable
[ -x file ]         # executable
[ -s file ]         # non-empty file
[ -L file ]         # is a symlink

# Combine conditions
[ cond1 ] && [ cond2 ]     # AND
[ cond1 ] || [ cond2 ]     # OR
[ ! cond1 ]                # NOT
```

### Double bracket `[[ ]]` — bash-only, more powerful

```bash
[[ $a == $b ]]          # string comparison (= also works)
[[ $a =~ ^[0-9]+$ ]]    # regex match
[[ -f file && -r file ]] # AND in one bracket
[[ $str == *"substr"* ]] # wildcard matching
```

---

## Loops

### for loop

```bash
# Over a list
for fruit in apple banana cherry; do
    echo "Fruit: $fruit"
done

# Over a range
for i in {1..10}; do
    echo "Number: $i"
done

# C-style
for ((i=0; i<5; i++)); do
    echo "i = $i"
done

# Over files
for file in /var/log/*.log; do
    echo "Processing: $file"
done

# Over command output
for user in $(cut -d: -f1 /etc/passwd); do
    echo "User: $user"
done

# Over array
servers=("web01" "web02" "db01")
for server in "${servers[@]}"; do
    echo "Checking $server..."
    ssh $server 'uptime'
done
```

### while loop

```bash
# Basic
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/passwd

# Infinite loop with break
while true; do
    if some_condition; then
        break
    fi
    sleep 1
done

# Until loop (opposite of while)
until [ $count -gt 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

---

## Functions

```bash
# Define
greet() {
    local name=$1       # local = scoped to this function
    echo "Hello, $name!"
}

# Call
greet "Alice"
greet "Bob"

# Return values
# Bash functions return exit codes (0-255), not values
# Use echo + command substitution for actual values

get_disk_usage() {
    df -h / | awk 'NR==2{print $5}'
}

usage=$(get_disk_usage)
echo "Disk usage: $usage"

# Check return code
backup_files() {
    tar -czf backup.tar.gz /etc/ 2>/dev/null
    return $?    # return exit code of tar
}

if backup_files; then
    echo "Backup succeeded"
else
    echo "Backup failed"
fi
```

---

## Exit Codes

```bash
# Every command returns 0 (success) or non-zero (failure)
ls /tmp
echo $?       # 0

ls /nonexistent
echo $?       # 2

# Exit your script
exit 0        # success
exit 1        # generic failure
exit 2        # misuse of command

# Use in conditionals
if ! grep -q "error" logfile.txt; then
    echo "No errors found"
fi

# Short-circuit operators
mkdir /tmp/mydir && echo "Created"     # run second only if first succeeds
rm file.txt || echo "Could not delete" # run second only if first fails
```

---

## Error Handling

```bash
#!/bin/bash
set -e          # exit immediately if any command fails
set -u          # treat unset variables as errors
set -o pipefail # catch errors in pipes (not just last command)
set -x          # print each command before executing (debug mode)

# Combine all (common in production scripts)
set -euo pipefail

# Trap — run cleanup on exit or error
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/lockfile
}
trap cleanup EXIT         # run on any exit
trap cleanup ERR          # run on error
trap cleanup INT TERM     # run on Ctrl+C or termination

# Custom error handling
error_exit() {
    echo "ERROR: $1" >&2
    exit 1
}

[ -f "$1" ] || error_exit "File not found: $1"
```

---

## Arrays

```bash
# Declare
fruits=("apple" "banana" "cherry")
declare -a servers=("web01" "web02" "db01")

# Access
echo "${fruits[0]}"          # apple (0-indexed)
echo "${fruits[-1]}"         # cherry (last element)
echo "${fruits[@]}"          # all elements
echo "${#fruits[@]}"         # number of elements

# Modify
fruits+=("date")             # append
fruits[1]="blueberry"        # change element

# Iterate
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# Iterate with index
for i in "${!fruits[@]}"; do
    echo "$i: ${fruits[$i]}"
done

# Slice
echo "${fruits[@]:1:2}"      # elements 1 and 2 (banana cherry)
```

---

## String Operations

```bash
str="Hello, World!"

echo ${#str}                  # length: 13
echo ${str:7}                 # substring from index 7: World!
echo ${str:7:5}               # substring, 5 chars: World
echo ${str,,}                 # lowercase: hello, world!
echo ${str^^}                 # uppercase: HELLO, WORLD!
echo ${str/World/Linux}       # replace first: Hello, Linux!
echo ${str//l/L}              # replace all: HeLLo, WorLd!

file="/var/log/nginx/access.log"
echo ${file##*/}              # basename: access.log
echo ${file%/*}               # dirname: /var/log/nginx
echo ${file%.log}             # remove suffix: /var/log/nginx/access
echo ${file##*.}              # extension: log

# Default values
echo ${var:-"default"}        # use default if var is unset/empty
echo ${var:="default"}        # assign default if var is unset/empty
```

---

## Real-World Script Examples

### System health check

```bash
#!/bin/bash
set -euo pipefail

THRESHOLD_CPU=80
THRESHOLD_MEM=90
THRESHOLD_DISK=85

echo "=== System Health Check - $(date) ==="

# CPU
cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d. -f1)
echo "CPU Usage: ${cpu_usage}%"
[ "$cpu_usage" -gt "$THRESHOLD_CPU" ] && echo "WARNING: High CPU usage!"

# Memory
mem_usage=$(free | awk '/Mem/{printf "%.0f", $3/$2*100}')
echo "Memory Usage: ${mem_usage}%"
[ "$mem_usage" -gt "$THRESHOLD_MEM" ] && echo "WARNING: High memory usage!"

# Disk
disk_usage=$(df / | awk 'NR==2{print $5}' | tr -d '%')
echo "Disk Usage: ${disk_usage}%"
[ "$disk_usage" -gt "$THRESHOLD_DISK" ] && echo "WARNING: High disk usage!"

echo "=== Check Complete ==="
```

### Backup script with logging

```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/backups"
SOURCE_DIR="/etc"
LOG_FILE="/var/log/backup.log"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/etc_backup_${DATE}.tar.gz"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

log "Starting backup of $SOURCE_DIR"

mkdir -p "$BACKUP_DIR"

if tar -czf "$BACKUP_FILE" "$SOURCE_DIR" 2>/dev/null; then
    SIZE=$(du -sh "$BACKUP_FILE" | cut -f1)
    log "Backup completed: $BACKUP_FILE ($SIZE)"
else
    log "ERROR: Backup failed"
    exit 1
fi

# Keep only last 7 backups
find "$BACKUP_DIR" -name "etc_backup_*.tar.gz" -mtime +7 -delete
log "Old backups cleaned up"
```

### User creation script with argument validation

```bash
#!/bin/bash
set -euo pipefail

usage() {
    echo "Usage: $0 <username> <group>"
    exit 1
}

[ $# -ne 2 ] && usage

USERNAME=$1
GROUP=$2

# Check if user already exists
if id "$USERNAME" &>/dev/null; then
    echo "User $USERNAME already exists"
    exit 1
fi

# Create group if it doesn't exist
if ! getent group "$GROUP" &>/dev/null; then
    sudo groupadd "$GROUP"
    echo "Created group: $GROUP"
fi

# Create user
sudo useradd -m -s /bin/bash -G "$GROUP" "$USERNAME"
echo "Created user: $USERNAME in group: $GROUP"

# Set a random temporary password
TEMP_PASS=$(openssl rand -base64 12)
echo "$USERNAME:$TEMP_PASS" | sudo chpasswd
echo "Temporary password: $TEMP_PASS"
echo "User must change password on first login"
sudo passwd --expire "$USERNAME"
```

---

## Debugging Scripts

```bash
bash -x script.sh           # print each command as it runs
bash -n script.sh           # syntax check only (don't run)
bash -v script.sh           # verbose (print each line before running)

set -x                      # enable debug inside script
set +x                      # disable debug

# Add debug output
echo "DEBUG: variable=$variable" >&2    # print to stderr
```
