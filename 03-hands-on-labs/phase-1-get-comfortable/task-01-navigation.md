# Task 01 — Filesystem Navigation & File Operations

> **Prerequisite:** Read `01-notes/01-linux-fundamentals.md` and `01-notes/02-file-system.md`
> **Environment:** Any Linux terminal (native, WSL2, VM, or container)
> **Difficulty:** Beginner

---

## Objective

By the end of this lab you will be able to:
- Navigate the Linux filesystem confidently
- Create, move, copy, rename, and delete files and directories
- Use absolute and relative paths without confusion
- Read file contents and search for text
- Understand the purpose of key directories (`/etc`, `/var`, `/home`, `/tmp`, `/proc`)

---

## Part 1 — Explore the System

### Exercise 1.1: Identify your system

Run each command and understand the output:

```bash
whoami
hostname
uname -a
cat /etc/os-release
uptime
```

**Questions to answer:**
- What user are you logged in as?
- What Linux distribution and version are you running?
- How long has the system been running?

### Exercise 1.2: Explore the root filesystem

```bash
ls /
```

Now explore each major directory. For each one, run `ls` inside it and try to understand what's there:

```bash
ls /etc        # Configuration files
ls /var        # Variable data (logs, caches)
ls /home       # User home directories
ls /tmp        # Temporary files
ls /usr/bin    # User programs
ls /sbin       # System programs
ls /proc       # Live kernel/process data (virtual filesystem)
ls /dev        # Device files
```

**Questions to answer:**
- Where would you find the system's hostname configuration?
- Where are log files stored?
- What happens when you run `cat /proc/cpuinfo`?

---

## Part 2 — Navigation

### Exercise 2.1: Moving around

```bash
pwd                     # Where am I?
cd /var/log             # Go to log directory
pwd                     # Confirm location
ls                      # What's here?
cd ..                   # Go up one level
pwd                     # Should be /var
cd ~                    # Go to home directory
pwd                     # Should be /home/<username>
cd -                    # Go back to previous directory
pwd                     # Should be /var again
```

### Exercise 2.2: Absolute vs relative paths

```bash
# Absolute path (starts from /)
cd /var/log
pwd

# Relative path (from current location)
cd ../../tmp
pwd                     # Should be /tmp

# Go home — all three do the same thing
cd ~
cd $HOME
cd
```

---

## Part 3 — Creating Files and Directories

### Exercise 3.1: Build a project structure

Create this structure from scratch:

```
~/lab-01/
├── project/
│   ├── src/
│   │   ├── main.sh
│   │   └── utils.sh
│   ├── docs/
│   │   └── readme.txt
│   └── config/
│       └── settings.conf
└── backup/
```

Commands:

```bash
mkdir -p ~/lab-01/project/{src,docs,config}
mkdir -p ~/lab-01/backup

touch ~/lab-01/project/src/main.sh
touch ~/lab-01/project/src/utils.sh
touch ~/lab-01/project/docs/readme.txt
touch ~/lab-01/project/config/settings.conf
```

Verify the structure:

```bash
tree ~/lab-01           # If tree is installed
# OR
find ~/lab-01 -type f   # List all files
find ~/lab-01 -type d   # List all directories
```

### Exercise 3.2: Add content to files

```bash
echo "#!/bin/bash" > ~/lab-01/project/src/main.sh
echo "echo 'Hello from main'" >> ~/lab-01/project/src/main.sh

echo "App version: 1.0" > ~/lab-01/project/config/settings.conf
echo "Debug mode: false" >> ~/lab-01/project/config/settings.conf
```

**Understand the difference:**
- `>` overwrites the file
- `>>` appends to the file

Verify:

```bash
cat ~/lab-01/project/src/main.sh
cat ~/lab-01/project/config/settings.conf
```

---

## Part 4 — Copy, Move, Rename, Delete

### Exercise 4.1: Copy files

```bash
# Copy a single file
cp ~/lab-01/project/config/settings.conf ~/lab-01/backup/

# Copy a directory and all its contents
cp -r ~/lab-01/project/src ~/lab-01/backup/

# Verify
ls ~/lab-01/backup/
```

### Exercise 4.2: Move and rename

```bash
# Move a file
mv ~/lab-01/project/docs/readme.txt ~/lab-01/project/docs/README.md

# Rename a directory
mv ~/lab-01/backup ~/lab-01/archive

# Verify
ls ~/lab-01/project/docs/
ls ~/lab-01/
```

### Exercise 4.3: Delete files and directories

```bash
# Delete a file
rm ~/lab-01/archive/settings.conf

# Try deleting a non-empty directory (this will fail)
rm ~/lab-01/archive

# Delete a directory and its contents
rm -r ~/lab-01/archive

# Verify
ls ~/lab-01/
```

**Warning:** `rm -r` is permanent. There is no recycle bin. Always double-check before running it.

---

## Part 5 — Reading and Searching Files

### Exercise 5.1: Different ways to read files

```bash
# Create a longer file for practice
seq 1 100 > ~/lab-01/numbers.txt

# Read entire file
cat ~/lab-01/numbers.txt

# Read first 10 lines
head ~/lab-01/numbers.txt

# Read last 10 lines
tail ~/lab-01/numbers.txt

# Read first 5 lines
head -n 5 ~/lab-01/numbers.txt

# Read with page-by-page scrolling
less ~/lab-01/numbers.txt
# (press q to quit, / to search, n for next match)
```

### Exercise 5.2: Search for text

```bash
# Create a sample log file
echo -e "INFO: App started\nERROR: Connection failed\nINFO: Retrying\nERROR: Timeout\nINFO: Recovered" > ~/lab-01/app.log

# Search for lines containing "ERROR"
grep "ERROR" ~/lab-01/app.log

# Search case-insensitive
grep -i "error" ~/lab-01/app.log

# Show line numbers
grep -n "ERROR" ~/lab-01/app.log

# Count matching lines
grep -c "ERROR" ~/lab-01/app.log

# Search recursively in a directory
grep -r "echo" ~/lab-01/project/
```

### Exercise 5.3: Find files

```bash
# Find all .sh files
find ~/lab-01 -name "*.sh"

# Find all directories
find ~/lab-01 -type d

# Find files modified in the last 10 minutes
find ~/lab-01 -mmin -10

# Find files larger than 1K
find ~/lab-01 -size +1k
```

---

## Part 6 — Wildcards and Shortcuts

### Exercise 6.1: Glob patterns

```bash
# Create some test files
touch ~/lab-01/{file1.txt,file2.txt,file3.log,file4.log,notes.md}

# List all .txt files
ls ~/lab-01/*.txt

# List all files starting with "file"
ls ~/lab-01/file*

# List files with single character before extension
ls ~/lab-01/file?.txt

# List .txt and .log files
ls ~/lab-01/*.{txt,log}
```

---

## Challenges

Try these without looking at the answers:

1. **Create a nested directory structure 5 levels deep in a single command**
2. **Find all `.conf` files anywhere under `/etc` (you'll need sudo)**
3. **Count how many files exist in `/usr/bin`**
4. **Copy the contents of `/etc/hostname` into a file in your home directory**
5. **Create 100 files named `test-001.txt` through `test-100.txt` in one command**

<details>
<summary>Hint for challenge 5</summary>

Look into brace expansion: `touch test-{001..100}.txt`

</details>

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| `rm -rf /` | Deletes entire filesystem | Never run this. Ever. |
| `cd somedir` when you don't know where you are | You might end up somewhere unexpected | Always `pwd` first or use absolute paths |
| Using spaces in filenames | Breaks commands if not quoted | Use quotes: `"my file.txt"` or avoid spaces |
| Forgetting `-r` when copying directories | `cp` without `-r` skips directories | `cp -r source/ dest/` |
| Using `>` when you meant `>>` | Overwrites the file instead of appending | Double-check before redirecting |

---

## Cleanup

```bash
rm -r ~/lab-01
```

---

## What's Next?

Proceed to `task-02-permissions.md` → Users, groups, and file permissions.
