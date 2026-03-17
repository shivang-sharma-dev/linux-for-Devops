# 03 — Users and Permissions

> Permissions are the most interview-tested Linux topic. More importantly — misconfigured permissions are the #1 cause of security incidents on Linux servers.

---

## The Permission Model

Linux assigns every file and directory three permission sets:

```
-  rw-  r--  r--   1   alice   devops   4096   Jan 10   file.txt
│  │    │    │     │   │       │        │
│  │    │    │     │   │       │        └─ file size
│  │    │    │     │   │       └─ group owner
│  │    │    │     │   └─ user owner
│  │    │    │     └─ number of hard links
│  │    │    └─ other (everyone else): r-- = read only
│  │    └─ group: r-- = read only
│  └─ user/owner: rw- = read + write
└─ file type: - = file, d = directory, l = symlink, c = char device, b = block device
```

### Permission bits

| Symbol | Octal | Meaning on FILE | Meaning on DIRECTORY |
|---|---|---|---|
| `r` | 4 | Read file contents | List directory contents (`ls`) |
| `w` | 2 | Write / modify file | Create/delete files inside dir |
| `x` | 1 | Execute file | Enter directory (`cd`) |
| `-` | 0 | Permission not set | Permission not set |

**On directories: `x` (execute) means enter.** Without `x` on a directory, you cannot `cd` into it even if you have `r`.

---

## chmod — Change Permissions

### Symbolic mode (easier to read)

```bash
chmod u+x script.sh         # add execute for user/owner
chmod g-w file.txt          # remove write for group
chmod o+r file.txt          # add read for others
chmod a+x script.sh         # add execute for all (a = ugo)
chmod u=rwx,g=rx,o= dir/    # set exact permissions (o= = no permissions)
chmod +x script.sh          # add execute for everyone (shorthand)
```

### Numeric/Octal mode (faster once you know it)

```bash
# Calculate: owner | group | other
# r=4, w=2, x=1 — add them up for each set

chmod 755 script.sh     # rwxr-xr-x  (owner: 7=rwx, group: 5=rx, other: 5=rx)
chmod 644 file.txt      # rw-r--r--  (owner: 6=rw, group: 4=r, other: 4=r)
chmod 600 private-key   # rw-------  (owner: 6=rw, only owner can read/write)
chmod 777 folder/       # rwxrwxrwx  (everyone full access — avoid in production)
chmod 700 ~/.ssh/       # rwx------  (only owner can access)
chmod 400 key.pem       # r--------  (AWS PEM key — read only by owner)

# Recursive: apply to directory and all contents
chmod -R 755 /var/www/html/
```

### Common permission values to memorise

| Octal | Symbolic | Typical use |
|---|---|---|
| `755` | `rwxr-xr-x` | Directories, executables, web server files |
| `644` | `rw-r--r--` | Regular files, config files |
| `600` | `rw-------` | Private keys, secret files |
| `700` | `rwx------` | Private directories |
| `400` | `r--------` | AWS `.pem` keys |
| `777` | `rwxrwxrwx` | Avoid — everyone has full access |

---

## chown — Change Ownership

```bash
chown alice file.txt              # change owner to alice
chown alice:devops file.txt       # change owner to alice, group to devops
chown :devops file.txt            # change group only
chown -R www-data /var/www/html/  # recursive — change all files in dir
```

```bash
# Real world: web server files owned by nginx user
sudo chown -R www-data:www-data /var/www/mysite/
sudo chmod -R 755 /var/www/mysite/
```

---

## chgrp — Change Group

```bash
chgrp devops file.txt             # change group to devops
chgrp -R developers /opt/project/ # recursive
```

---

## User Management

### Understanding `/etc/passwd`

```bash
cat /etc/passwd
# alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash
# │     │ │    │    │           │            └─ login shell
# │     │ │    │    │           └─ home directory
# │     │ │    │    └─ comment/full name (GECOS)
# │     │ │    └─ primary GID
# │     │ └─ UID
# │     └─ x = password is in /etc/shadow
# └─ username
```

### Understanding `/etc/shadow`

Stores hashed passwords — only root can read it:
```bash
sudo cat /etc/shadow
# alice:$6$rounds=...:19000:0:99999:7:::
# username : hashed_password : last_change : min_age : max_age : warn ...
```

### User commands

```bash
# Create users
sudo useradd alice                          # create user (minimal)
sudo useradd -m -s /bin/bash alice          # create with home dir and bash shell
sudo useradd -m -s /bin/bash -G sudo alice  # create and add to sudo group
sudo adduser alice                          # interactive (Ubuntu) — friendlier

# Set / change password
sudo passwd alice

# Modify existing user
sudo usermod -aG docker alice        # add alice to docker group (-a = append, important!)
sudo usermod -aG sudo alice          # give alice sudo access
sudo usermod -s /bin/zsh alice       # change shell
sudo usermod -l newname alice        # rename user
sudo usermod -L alice                # lock account (disable login)
sudo usermod -U alice                # unlock account

# Delete user
sudo userdel alice                   # delete user (keep home dir)
sudo userdel -r alice                # delete user AND home directory

# See user info
id alice                             # UID, GID, groups
id                                   # your own info
groups alice                         # which groups alice belongs to
who                                  # who is currently logged in
w                                    # who is logged in and what they're doing
last                                 # login history
```

---

## Group Management

```bash
# Create group
sudo groupadd devops

# Add user to group
sudo usermod -aG devops alice        # -a is CRITICAL: without it, replaces all groups

# Remove user from group
sudo gpasswd -d alice devops

# Delete group
sudo groupdel devops

# See group membership
cat /etc/group | grep devops
getent group devops

# Switch to another group in current session
newgrp docker                        # switch primary group (affects new files)
```

**Why `-a` matters with `usermod -G`:**
```bash
# BAD: replaces ALL existing groups with just docker
sudo usermod -G docker alice        # alice loses sudo, loses everything else

# GOOD: appends docker to existing groups
sudo usermod -aG docker alice
```

---

## sudo — Controlled Privilege Escalation

```bash
sudo command                    # run as root
sudo -u www-data command        # run as specific user
sudo -i                         # root shell (interactive)
sudo -l                         # list what you can run with sudo
sudo !!                         # re-run last command with sudo

# Edit sudoers safely (validates syntax before saving)
sudo visudo
```

### `/etc/sudoers` format

```
# /etc/sudoers
# Format: WHO  WHERE=(AS_WHOM)  WHAT

# Full sudo access
alice   ALL=(ALL:ALL) ALL

# Run without password prompt
alice   ALL=(ALL) NOPASSWD: ALL

# Only allow specific commands
deploy  ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx, /bin/systemctl status nginx

# Group syntax (% = group)
%sudo   ALL=(ALL:ALL) ALL
%devops ALL=(ALL) NOPASSWD: /usr/bin/docker
```

### `/etc/sudoers.d/` — drop-in files

Better practice than editing the main sudoers file:
```bash
# Create a file for a specific user or team
sudo visudo -f /etc/sudoers.d/deploy-user
# Add: deploy ALL=(ALL) NOPASSWD: /bin/systemctl restart myapp
```

---

## umask — Default Permission Mask

`umask` controls what permissions new files and directories get by default.

```
New file  base permissions: 666 (rw-rw-rw-)
New dir   base permissions: 777 (rwxrwxrwx)

umask 022:
  File: 666 - 022 = 644 (rw-r--r--)
  Dir:  777 - 022 = 755 (rwxr-xr-x)

umask 027:
  File: 666 - 027 = 640 (rw-r-----)
  Dir:  777 - 027 = 750 (rwxr-x---)
```

```bash
umask                   # see current umask
umask 022               # set umask (this session only)
umask 027               # more restrictive (group gets r, others get nothing)

# Set permanently — add to ~/.bashrc or /etc/profile
echo "umask 022" >> ~/.bashrc
```

---

## Special Permissions

### SUID (Set User ID) — `4xxx`

When set on an executable, it runs as the file's **owner** regardless of who executes it.

```bash
ls -la /usr/bin/passwd
# -rwsr-xr-x  root  root  /usr/bin/passwd
#   ^  s = SUID bit

# The 's' in owner execute position = SUID
# This is why any user can change their own password —
# passwd runs as root (needed to write /etc/shadow)
```

```bash
chmod u+s script            # set SUID
chmod 4755 script           # set SUID with octal
```

### SGID (Set Group ID) — `2xxx`

On a file: runs as the file's group.
On a directory: new files inside inherit the directory's group.

```bash
chmod g+s shared-dir/       # set SGID on directory
chmod 2755 shared-dir/      # octal

# Useful for team shared directories
sudo mkdir /opt/team
sudo chgrp devops /opt/team
sudo chmod 2775 /opt/team   # SGID + rwxrwxr-x
# Now all files created in /opt/team belong to devops group
```

### Sticky Bit — `1xxx`

On a directory: users can only delete their own files, even if they have write permission on the directory.

```bash
ls -la /tmp
# drwxrwxrwt  root  root  /tmp
#          t = sticky bit

chmod +t /shared-dir/       # set sticky bit
chmod 1777 /shared-dir/     # octal

# Why /tmp has it: everyone can write there, but you can't delete others' files
```

---

## ACLs — Fine-Grained Permissions

Standard Linux permissions only have three buckets (owner, group, other). ACLs (Access Control Lists) allow per-user and per-group permissions.

```bash
# Check if ACL is set (+ at end of permissions in ls -la)
ls -la file.txt
# -rw-r--r--+  alice  alice  file.txt   ← + means ACL is set

# View ACL
getfacl file.txt

# Set ACL: give bob read+write access to alice's file
setfacl -m u:bob:rw file.txt

# Give devops group read access
setfacl -m g:devops:r file.txt

# Set default ACL on directory (inherited by new files)
setfacl -d -m g:devops:rw /shared/

# Remove ACL
setfacl -x u:bob file.txt

# Remove all ACLs
setfacl -b file.txt
```

---

## Permission Troubleshooting — Systematic Approach

When you get `Permission denied`:

```bash
# Step 1: Check the file's permissions
ls -la /path/to/file

# Step 2: Check who you are
whoami
id

# Step 3: Check directory permissions (need x to enter, r to list)
ls -la /path/to/

# Step 4: Check if there's an ACL
getfacl /path/to/file

# Step 5: Check SELinux/AppArmor (on RHEL/some Ubuntus)
ls -laZ /path/to/file          # SELinux context
getenforce                     # is SELinux enforcing?
aa-status                      # AppArmor status

# Common fixes
sudo chmod +r /path/to/file
sudo chown youruser /path/to/file
sudo usermod -aG groupname youruser   # then log out and back in
```

**Remember: group membership changes only take effect after logging out and back in** (or running `newgrp groupname`).
