# Task 02 — Users, Groups & File Permissions

> **Prerequisite:** Read `01-notes/03-users-and-permissions.md`
> **Environment:** Full Linux system (native, WSL2, or VM). You need `sudo` access.
> **Difficulty:** Beginner

---

## Objective

By the end of this lab you will be able to:
- Create and manage users and groups
- Understand the permission model (`rwx` for owner, group, others)
- Change file permissions with `chmod` (symbolic and numeric)
- Change file ownership with `chown` and `chgrp`
- Use `sudo` and understand why it matters
- Read and interpret `ls -l` output

---

## Part 1 — Understanding `ls -l` Output

### Exercise 1.1: Read permission strings

```bash
mkdir -p ~/lab-02 && cd ~/lab-02
touch myfile.txt
mkdir mydir
ls -la
```

You'll see something like:

```
-rw-rw-r-- 1 user user    0 Mar 17 10:00 myfile.txt
drwxrwxr-x 2 user user 4096 Mar 17 10:00 mydir
```

Break down the permission string `-rw-rw-r--`:

```
-    rw-    rw-    r--
│    │      │      │
│    │      │      └── Others: read only
│    │      └── Group:  read + write
│    └── Owner:  read + write
└── File type: - = file, d = directory, l = symlink
```

### Exercise 1.2: Check your own identity

```bash
whoami              # Your username
id                  # Your UID, GID, and all groups
groups              # Groups you belong to
cat /etc/passwd | grep $(whoami)    # Your entry in passwd file
```

---

## Part 2 — Managing Users

### Exercise 2.1: Create users

```bash
# Create a new user
sudo useradd -m -s /bin/bash testuser1

# Create another user with a comment
sudo useradd -m -s /bin/bash -c "Test User Two" testuser2

# Set passwords
sudo passwd testuser1
sudo passwd testuser2

# Verify users were created
cat /etc/passwd | tail -5
ls /home/
```

**Flags explained:**
- `-m` → create home directory
- `-s /bin/bash` → set default shell
- `-c "comment"` → add a description

### Exercise 2.2: Switch between users

```bash
# Switch to testuser1
su - testuser1
whoami
pwd                 # Should be /home/testuser1
exit                # Go back to your user

# Run a single command as another user
sudo -u testuser1 whoami
```

### Exercise 2.3: Delete a user

```bash
# Delete user but keep home directory
sudo userdel testuser2

# Delete user AND their home directory
sudo userdel -r testuser2    # Only if testuser2 still exists

# Verify
cat /etc/passwd | grep testuser2    # Should return nothing
```

---

## Part 3 — Managing Groups

### Exercise 3.1: Create and manage groups

```bash
# Create a group
sudo groupadd devteam

# Add testuser1 to the group
sudo usermod -aG devteam testuser1

# Verify
groups testuser1
cat /etc/group | grep devteam
```

**Important:** `-aG` means **append** to groups. Without `-a`, it replaces all secondary groups.

### Exercise 3.2: Create a shared project directory

```bash
# Create a shared directory
sudo mkdir /opt/project

# Change group ownership
sudo chgrp devteam /opt/project

# Allow group members to write
sudo chmod 775 /opt/project

# Verify
ls -ld /opt/project
```

---

## Part 4 — File Permissions with `chmod`

### Exercise 4.1: Symbolic mode

```bash
cd ~/lab-02

# Create test files
touch script.sh data.csv secret.key

# Add execute permission for owner
chmod u+x script.sh

# Remove write permission for others
chmod o-w data.csv

# Set read-only for everyone except owner
chmod u+rw,g+r,g-wx,o+r,o-wx data.csv

# Remove all permissions for group and others
chmod go-rwx secret.key

# Verify
ls -l
```

**Symbolic mode reference:**
- `u` = owner, `g` = group, `o` = others, `a` = all
- `+` = add, `-` = remove, `=` = set exactly
- `r` = read, `w` = write, `x` = execute

### Exercise 4.2: Numeric (octal) mode

Each permission has a number: `r=4, w=2, x=1`

```bash
# rwxr-xr-x = 755
chmod 755 script.sh

# rw-r--r-- = 644
chmod 644 data.csv

# rw------- = 600
chmod 600 secret.key

# rwxrwx--- = 770
chmod 770 mydir

# Verify
ls -l
```

**Common permission combos:**

| Numeric | Symbolic | Meaning | Use Case |
|---------|----------|---------|----------|
| `644` | `rw-r--r--` | Owner reads/writes, others read | Config files |
| `755` | `rwxr-xr-x` | Owner full, others read/execute | Scripts, directories |
| `600` | `rw-------` | Owner only | SSH keys, secrets |
| `700` | `rwx------` | Owner only (directory) | Private directories |
| `775` | `rwxrwxr-x` | Owner+group full, others read | Shared project dirs |

### Exercise 4.3: Directory permissions

Permissions mean different things for directories:

| Permission | On Files | On Directories |
|-----------|----------|----------------|
| `r` (read) | Read file contents | List directory contents (`ls`) |
| `w` (write) | Modify file contents | Create/delete files in dir |
| `x` (execute) | Run as program | Enter directory (`cd`) |

```bash
# Demonstrate: directory without x
mkdir ~/lab-02/testdir
touch ~/lab-02/testdir/file.txt
chmod 644 ~/lab-02/testdir    # Remove execute from directory

ls ~/lab-02/testdir           # Can list (r is set)
cd ~/lab-02/testdir           # FAILS — can't enter without x
cat ~/lab-02/testdir/file.txt # FAILS — can't traverse without x

# Fix it
chmod 755 ~/lab-02/testdir
```

---

## Part 5 — Ownership with `chown` and `chgrp`

### Exercise 5.1: Change ownership

```bash
cd ~/lab-02

# Change owner of a file
sudo chown testuser1 data.csv
ls -l data.csv

# Change owner AND group
sudo chown testuser1:devteam script.sh
ls -l script.sh

# Change group only
sudo chgrp devteam secret.key
ls -l secret.key

# Change ownership recursively (entire directory)
sudo chown -R testuser1:devteam mydir
ls -l
```

---

## Part 6 — sudo

### Exercise 6.1: Understand sudo

```bash
# Run a command as root
sudo whoami               # Output: root

# Edit a system file (requires root)
sudo cat /etc/shadow      # Works with sudo
cat /etc/shadow            # FAILS without sudo

# Check who can use sudo
sudo cat /etc/sudoers      # Don't edit this directly

# Check sudo log
sudo cat /var/log/auth.log | grep sudo | tail -10
```

### Exercise 6.2: Give a user sudo access

```bash
# Add testuser1 to the sudo group
sudo usermod -aG sudo testuser1

# Verify
groups testuser1     # Should include "sudo"

# Test it
su - testuser1
sudo whoami          # Should output "root"
exit
```

---

## Part 7 — Special Permissions

### Exercise 7.1: Setuid, Setgid, Sticky bit

```bash
# Find setuid programs on your system
find /usr/bin -perm -4000 2>/dev/null | head -10
# passwd, sudo, ping — these run as root even when called by a normal user

# Check the sticky bit on /tmp
ls -ld /tmp
# drwxrwxrwt — the "t" at the end is the sticky bit
# Anyone can write to /tmp, but can only delete their OWN files

# Set sticky bit on a directory
chmod +t ~/lab-02/mydir
ls -ld ~/lab-02/mydir
```

---

## Challenges

1. **Create a "project" scenario:**
   - Create users: `alice`, `bob`
   - Create group: `webteam`
   - Add both users to `webteam`
   - Create `/var/www/project` owned by root, group `webteam`, permissions `775`
   - Verify both `alice` and `bob` can create files in it
   - Verify a user NOT in `webteam` cannot write to it

2. **Lock down a secrets directory:**
   - Create `~/secrets/` with permissions `700`
   - Create `~/secrets/api-key.txt` with permissions `600`
   - Verify no other user can read, list, or enter the directory

3. **Find all world-writable files on your system:**
   ```bash
   find / -perm -o+w -type f 2>/dev/null | head -20
   ```

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| `chmod 777` everywhere | Makes files writable by everyone — security risk | Use the minimum permissions needed |
| `usermod -G` without `-a` | Replaces all groups instead of appending | Always use `usermod -aG` |
| Forgetting `chmod +x` on scripts | Script won't run without execute permission | `chmod +x script.sh` |
| Running everything as `root` | One wrong command can destroy the system | Use a normal user + `sudo` for specific commands |
| Setting `600` on a directory | Can't enter it (needs `x`) | Use `700` for private directories |

---

## Cleanup

```bash
sudo userdel -r testuser1 2>/dev/null
sudo groupdel devteam 2>/dev/null
sudo rm -rf /opt/project
rm -rf ~/lab-02
```

---

## What's Next?

Proceed to `task-03-processes.md` → Process management and signals.
