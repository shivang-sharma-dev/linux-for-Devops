# Task 05 — Shell Scripting

> **Prerequisite:** Read `01-notes/06-shell-scripting.md`
> **Environment:** Any Linux terminal with Bash
> **Difficulty:** Beginner → Intermediate

---

## Objective

By the end of this lab you will be able to:
- Write and run Bash scripts
- Use variables, arguments, and user input
- Write conditionals (`if/else`) and loops (`for`, `while`)
- Work with functions
- Handle exit codes and basic error handling
- Build 3 practical scripts from scratch

---

## Part 1 — Your First Script

### Exercise 1.1: Create and run a script

```bash
mkdir -p ~/lab-05 && cd ~/lab-05

# Create the script
cat > hello.sh << 'EOF'
#!/bin/bash
echo "Hello, World!"
echo "Running as: $(whoami)"
echo "Current directory: $(pwd)"
echo "Today is: $(date)"
EOF

# Make it executable
chmod +x hello.sh

# Run it
./hello.sh
```

**Key points:**
- `#!/bin/bash` — the shebang line, tells the system to use Bash
- `chmod +x` — required to make it executable
- `./` — run from current directory

---

## Part 2 — Variables

### Exercise 2.1: Variable basics

```bash
cat > variables.sh << 'EOF'
#!/bin/bash

# Assigning variables (NO spaces around =)
name="Linux"
version=22
today=$(date +%Y-%m-%d)

# Using variables
echo "Learning $name"
echo "Version: $version"
echo "Date: $today"

# String with variables (double quotes allow expansion)
echo "Welcome to $name version $version"

# Single quotes — NO variable expansion
echo 'This prints $name literally'

# Curly braces for clarity
file="report"
echo "${file}_2024.txt"
EOF

chmod +x variables.sh && ./variables.sh
```

### Exercise 2.2: Script arguments

```bash
cat > args.sh << 'EOF'
#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "Number of arguments: $#"
EOF

chmod +x args.sh
./args.sh hello world
./args.sh one two three
```

### Exercise 2.3: Read user input

```bash
cat > greeting.sh << 'EOF'
#!/bin/bash

read -p "What is your name? " username
read -p "What is your role? " role

echo "Hello, $username! You are a $role."
EOF

chmod +x greeting.sh && ./greeting.sh
```

---

## Part 3 — Conditionals

### Exercise 3.1: if/else

```bash
cat > check-file.sh << 'EOF'
#!/bin/bash

filename=$1

if [ -z "$filename" ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

if [ -f "$filename" ]; then
    echo "$filename is a regular file"
    echo "Size: $(wc -c < "$filename") bytes"
elif [ -d "$filename" ]; then
    echo "$filename is a directory"
    echo "Contents: $(ls "$filename" | wc -l) items"
else
    echo "$filename does not exist"
fi
EOF

chmod +x check-file.sh
./check-file.sh /etc/hostname
./check-file.sh /etc
./check-file.sh nonexistent
./check-file.sh
```

### Exercise 3.2: Test operators

```bash
cat > comparisons.sh << 'EOF'
#!/bin/bash

# Numeric comparisons
a=10
b=20

if [ $a -lt $b ]; then
    echo "$a is less than $b"
fi

if [ $a -eq 10 ]; then
    echo "a equals 10"
fi

# String comparisons
str1="hello"
str2="world"

if [ "$str1" != "$str2" ]; then
    echo "'$str1' is not equal to '$str2'"
fi

# Check if string is empty
empty_var=""
if [ -z "$empty_var" ]; then
    echo "Variable is empty"
fi

# File tests
if [ -f /etc/passwd ]; then
    echo "/etc/passwd exists and is a file"
fi

if [ -w /tmp ]; then
    echo "/tmp is writable"
fi
EOF

chmod +x comparisons.sh && ./comparisons.sh
```

**Common test operators:**

| Numeric | Meaning | String | Meaning | File | Meaning |
|---------|---------|--------|---------|------|---------|
| `-eq` | equal | `=` | equal | `-f` | is file |
| `-ne` | not equal | `!=` | not equal | `-d` | is directory |
| `-lt` | less than | `-z` | is empty | `-e` | exists |
| `-gt` | greater than | `-n` | not empty | `-r` | readable |
| `-le` | less or equal | | | `-w` | writable |
| `-ge` | greater or equal | | | `-x` | executable |

---

## Part 4 — Loops

### Exercise 4.1: for loop

```bash
cat > loops.sh << 'EOF'
#!/bin/bash

# Loop over a list
echo "=== Fruits ==="
for fruit in apple banana cherry; do
    echo "I like $fruit"
done

# Loop over files
echo -e "\n=== Config files in /etc (first 5) ==="
for file in $(ls /etc/*.conf 2>/dev/null | head -5); do
    echo "Found: $file"
done

# Loop with a range
echo -e "\n=== Counting ==="
for i in {1..5}; do
    echo "Number: $i"
done

# C-style for loop
echo -e "\n=== Squares ==="
for ((i=1; i<=5; i++)); do
    echo "$i squared = $((i*i))"
done
EOF

chmod +x loops.sh && ./loops.sh
```

### Exercise 4.2: while loop

```bash
cat > countdown.sh << 'EOF'
#!/bin/bash

count=${1:-10}    # Default to 10 if no argument

echo "Countdown from $count:"
while [ $count -gt 0 ]; do
    echo "$count..."
    count=$((count - 1))
    sleep 1
done
echo "DONE!"
EOF

chmod +x countdown.sh && ./countdown.sh 5
```

### Exercise 4.3: Read a file line by line

```bash
# Create a test file
echo -e "alice:developer\nbob:devops\ncharlie:sysadmin" > ~/lab-05/team.txt

cat > read-file.sh << 'EOF'
#!/bin/bash

input_file=$1

if [ ! -f "$input_file" ]; then
    echo "File not found: $input_file"
    exit 1
fi

echo "Team Members:"
echo "-------------"
while IFS=':' read -r name role; do
    echo "$name → $role"
done < "$input_file"
EOF

chmod +x read-file.sh && ./read-file.sh team.txt
```

---

## Part 5 — Functions

### Exercise 5.1: Basic functions

```bash
cat > functions.sh << 'EOF'
#!/bin/bash

# Define a function
greet() {
    local name=$1    # local variable
    echo "Hello, $name!"
}

# Function with return value
is_root() {
    if [ "$(id -u)" -eq 0 ]; then
        return 0    # true
    else
        return 1    # false
    fi
}

# Function that outputs a value
get_disk_usage() {
    df -h / | awk 'NR==2 {print $5}'
}

# Use the functions
greet "DevOps Engineer"
greet "Linux Admin"

if is_root; then
    echo "Running as root"
else
    echo "NOT running as root"
fi

usage=$(get_disk_usage)
echo "Disk usage: $usage"
EOF

chmod +x functions.sh && ./functions.sh
```

---

## Part 6 — Exit Codes & Error Handling

### Exercise 6.1: Exit codes

```bash
cat > exit-codes.sh << 'EOF'
#!/bin/bash

# Every command returns an exit code
# 0 = success, non-zero = failure

ls /etc/passwd > /dev/null
echo "ls exit code: $?"       # Should be 0

ls /nonexistent 2>/dev/null
echo "ls exit code: $?"       # Should be non-zero

# Using exit codes in scripts
check_command() {
    if command -v "$1" > /dev/null 2>&1; then
        echo "$1 is installed"
    else
        echo "$1 is NOT installed"
    fi
}

check_command git
check_command docker
check_command nonexistent_tool
EOF

chmod +x exit-codes.sh && ./exit-codes.sh
```

### Exercise 6.2: set -e and error handling

```bash
cat > safe-script.sh << 'EOF'
#!/bin/bash
set -euo pipefail
# -e : exit on error
# -u : error on undefined variables
# -o pipefail : catch errors in pipes

echo "This script stops on any error"

# This will succeed
echo "Step 1: OK"

# Simulate success
true
echo "Step 2: OK"

echo "All steps completed successfully"
EOF

chmod +x safe-script.sh && ./safe-script.sh
```

---

## Part 7 — Build 3 Real Scripts

### Script 1: System Info Reporter

```bash
cat > sysinfo.sh << 'EOF'
#!/bin/bash
# System Information Reporter

echo "================================"
echo "     SYSTEM INFORMATION"
echo "================================"
echo ""
echo "Hostname     : $(hostname)"
echo "OS           : $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)"
echo "Kernel       : $(uname -r)"
echo "Uptime       : $(uptime -p)"
echo "User         : $(whoami)"
echo ""
echo "--- CPU ---"
echo "Model        : $(lscpu | grep 'Model name' | sed 's/.*: *//')"
echo "Cores        : $(nproc)"
echo "Load Average : $(cat /proc/loadavg | awk '{print $1, $2, $3}')"
echo ""
echo "--- Memory ---"
free -h | awk 'NR==2{printf "Total: %s | Used: %s | Free: %s\n", $2, $3, $4}'
echo ""
echo "--- Disk ---"
df -h / | awk 'NR==2{printf "Total: %s | Used: %s (%s) | Free: %s\n", $2, $3, $5, $4}'
echo ""
echo "--- Network ---"
echo "IP Address   : $(hostname -I | awk '{print $1}')"
echo "Gateway      : $(ip route | grep default | awk '{print $3}')"
echo ""
echo "================================"
EOF

chmod +x sysinfo.sh && ./sysinfo.sh
```

### Script 2: Backup Script

```bash
cat > backup.sh << 'EOF'
#!/bin/bash
set -euo pipefail

# Configuration
SOURCE=${1:?"Usage: $0 <source_dir> [backup_dir]"}
BACKUP_DIR=${2:-"$HOME/backups"}
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_NAME="backup_$(basename "$SOURCE")_$TIMESTAMP.tar.gz"

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Check source exists
if [ ! -d "$SOURCE" ]; then
    echo "ERROR: Source directory '$SOURCE' does not exist"
    exit 1
fi

# Create the backup
echo "Backing up: $SOURCE"
echo "Destination: $BACKUP_DIR/$BACKUP_NAME"

tar -czf "$BACKUP_DIR/$BACKUP_NAME" -C "$(dirname "$SOURCE")" "$(basename "$SOURCE")"

# Verify
if [ -f "$BACKUP_DIR/$BACKUP_NAME" ]; then
    size=$(du -h "$BACKUP_DIR/$BACKUP_NAME" | awk '{print $1}')
    echo "Backup created successfully ($size)"
else
    echo "ERROR: Backup failed"
    exit 1
fi

# Show recent backups
echo ""
echo "Recent backups:"
ls -lh "$BACKUP_DIR" | tail -5
EOF

chmod +x backup.sh

# Test it
mkdir -p ~/lab-05/testdata
echo "important data" > ~/lab-05/testdata/file1.txt
echo "more data" > ~/lab-05/testdata/file2.txt
./backup.sh ~/lab-05/testdata
```

### Script 3: Service Health Checker

```bash
cat > health-check.sh << 'EOF'
#!/bin/bash

# List of services to check
SERVICES=("sshd" "cron" "systemd-resolved")

# List of URLs to check
URLS=("https://google.com" "https://github.com")

echo "========================================="
echo "       SERVICE HEALTH CHECK"
echo "  $(date)"
echo "========================================="

# Check system services
echo ""
echo "--- System Services ---"
for service in "${SERVICES[@]}"; do
    if systemctl is-active "$service" > /dev/null 2>&1; then
        echo "[OK]   $service is running"
    else
        echo "[FAIL] $service is NOT running"
    fi
done

# Check URLs
echo ""
echo "--- URL Checks ---"
for url in "${URLS[@]}"; do
    status=$(curl -s -o /dev/null -w "%{http_code}" --max-time 5 "$url" 2>/dev/null)
    if [ "$status" = "200" ] || [ "$status" = "301" ] || [ "$status" = "302" ]; then
        echo "[OK]   $url (HTTP $status)"
    else
        echo "[FAIL] $url (HTTP $status)"
    fi
done

# Check disk space
echo ""
echo "--- Disk Space ---"
disk_usage=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')
if [ "$disk_usage" -lt 80 ]; then
    echo "[OK]   Disk usage: ${disk_usage}%"
elif [ "$disk_usage" -lt 90 ]; then
    echo "[WARN] Disk usage: ${disk_usage}% — getting high"
else
    echo "[CRIT] Disk usage: ${disk_usage}% — critically high!"
fi

# Check memory
echo ""
echo "--- Memory ---"
mem_usage=$(free | awk 'NR==2 {printf "%.0f", $3/$2 * 100}')
echo "[INFO] Memory usage: ${mem_usage}%"

echo ""
echo "========================================="
EOF

chmod +x health-check.sh && ./health-check.sh
```

---

## Challenges

1. **Write a script that accepts a directory as an argument and reports:**
   - Total number of files
   - Total number of directories
   - Total size
   - Largest file (name and size)

2. **Write a script that monitors a log file in real-time** and prints only lines containing "ERROR" or "WARN"

3. **Write a calculator script** that takes two numbers and an operator (`+`, `-`, `*`, `/`) as arguments

4. **Write a script that renames all `.txt` files in a directory** to `.md` files, with a confirmation prompt before each rename

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| `name = "value"` | Spaces around `=` cause errors | `name="value"` — no spaces |
| `if [ $var = "test" ]` | Breaks if `$var` is empty | Always quote: `if [ "$var" = "test" ]` |
| Missing `#!/bin/bash` | Script may run with wrong shell | Always include the shebang |
| Forgetting `chmod +x` | Script won't execute | `chmod +x script.sh` |
| Using `echo` for complex output | Breaks with special characters | Use `printf` for formatted output |

---

## Cleanup

```bash
rm -rf ~/lab-05
```

---

## What's Next?

Proceed to `task-06-package-management.md` → Installing and managing software packages.
