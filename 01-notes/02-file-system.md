# 02 — File System

> The filesystem is the map of Linux. Master navigation and file operations and every other topic becomes easier.

---

## Navigating the Filesystem

```bash
pwd                     # where am I right now
ls                      # list files in current directory
ls -l                   # long format (permissions, size, date)
ls -la                  # include hidden files (starting with .)
ls -lh                  # human readable sizes (KB, MB, GB)
ls -lt                  # sort by modification time (newest first)
ls -R                   # recursive (show all subdirectories)

cd /etc                 # go to absolute path
cd nginx                # go to relative path (subfolder of current)
cd ..                   # go up one level
cd ~                    # go to your home directory
cd -                    # go back to previous directory
```

---

## Creating, Copying, Moving, Deleting

```bash
# Create
touch file.txt                  # create empty file (or update timestamp)
mkdir my-folder                 # create directory
mkdir -p a/b/c                  # create nested directories at once

# Copy
cp file.txt backup.txt          # copy file
cp -r folder/ backup-folder/    # copy directory recursively
cp -p file.txt dest/            # preserve permissions and timestamps

# Move / Rename
mv file.txt newname.txt         # rename
mv file.txt /tmp/               # move to another directory
mv folder/ /opt/myapp/          # move directory

# Delete
rm file.txt                     # delete file
rm -r folder/                   # delete directory and all contents
rm -rf folder/                  # force delete (no prompts) — be careful
rm -i file.txt                  # prompt before deleting (safer)

# Create and write to a file
echo "hello world" > file.txt   # write (overwrites)
echo "new line" >> file.txt     # append
cat > file.txt << EOF           # write multi-line (heredoc)
line one
line two
EOF
```

---

## Viewing File Contents

```bash
cat file.txt                    # print entire file
less file.txt                   # scrollable view (q to quit, / to search)
more file.txt                   # older pager (less is better)
head file.txt                   # first 10 lines
head -n 20 file.txt             # first 20 lines
tail file.txt                   # last 10 lines
tail -n 50 file.txt             # last 50 lines
tail -f /var/log/syslog         # follow file live (great for logs)
tail -F /var/log/nginx/access.log  # follow even if file is rotated

wc -l file.txt                  # count lines
wc -w file.txt                  # count words
wc -c file.txt                  # count bytes
```

---

## Finding Files

```bash
# find — searches in real time, most powerful
find /etc -name "nginx.conf"                    # find by name
find /home -name "*.log"                        # find by pattern
find /var -type f -name "*.log"                 # files only
find /var -type d                               # directories only
find / -size +100M                              # files larger than 100MB
find / -mtime -7                               # modified in last 7 days
find / -user alice                              # owned by user alice
find /tmp -empty                               # empty files/dirs
find . -name "*.py" -exec grep -l "TODO" {} \; # find py files containing TODO

# locate — uses a database (fast but may be outdated)
locate nginx.conf
sudo updatedb                   # update the locate database

# which — find where a command is installed
which python3                   # /usr/bin/python3
which nginx                     # /usr/sbin/nginx

# whereis — find binary, source, and man pages
whereis nginx

# type — tells you what a command is
type ls                         # ls is aliased to 'ls --color=auto'
type cd                         # cd is a shell builtin
```

---

## File Information

```bash
file report.pdf                 # detect file type (doesn't trust extension)
file script.sh                  # "Bourne-Again shell script, ASCII text executable"
stat file.txt                   # detailed info: size, inode, permissions, timestamps
du -sh folder/                  # disk usage of a folder (human readable)
du -sh *                        # disk usage of everything in current dir
du -sh /* 2>/dev/null           # disk usage of each top-level directory
df -h                           # disk usage of all mounted filesystems
```

---

## Links — Hard Links vs Symbolic Links

```
Hard link:
  file.txt ──────────────── inode 12345 ──── actual data on disk
  hardlink.txt ─────────────────┘

  Both point directly to the same inode.
  Deleting file.txt doesn't delete the data — hardlink.txt still works.
  Hard links can't span filesystems or link to directories.

Symbolic (soft) link:
  symlink.txt ──► "points to /path/to/file.txt" ──► inode 12345 ──── data
  
  symlink is just a pointer. If file.txt is deleted, symlink breaks.
  Can span filesystems. Can link to directories.
  This is what you use 99% of the time.
```

```bash
# Create a hard link
ln original.txt hardlink.txt

# Create a symbolic link
ln -s /etc/nginx/nginx.conf nginx.conf      # relative symlink
ln -s /usr/local/bin/python3 /usr/bin/python  # absolute symlink

# See where a symlink points
ls -la symlink.txt                          # shows: symlink.txt -> /path/to/target
readlink symlink.txt                        # prints the target path
readlink -f symlink.txt                     # resolves all symlinks, prints final path

# Real-world example: multiple Python versions
ls -la /usr/bin/python*
# python3 -> python3.11
# python -> python3
```

---

## Archiving and Compression

```bash
# tar — the go-to for bundling files
tar -czf archive.tar.gz folder/         # create compressed archive
tar -czf backup.tar.gz /etc/nginx/      # backup nginx config
tar -xzf archive.tar.gz                 # extract here
tar -xzf archive.tar.gz -C /opt/       # extract to specific directory
tar -tzf archive.tar.gz                 # list contents without extracting
tar -cvf archive.tar folder/            # create uncompressed (verbose)

# tar flags decoded:
# c = create, x = extract, t = list
# z = gzip compression (.gz), j = bzip2 (.bz2), J = xz (.xz)
# f = filename follows, v = verbose

# gzip / gunzip
gzip file.txt                           # compress → file.txt.gz (original deleted)
gzip -k file.txt                        # keep original
gunzip file.txt.gz                      # decompress

# zip (compatible with Windows)
zip archive.zip file1 file2
zip -r archive.zip folder/
unzip archive.zip
unzip archive.zip -d /tmp/destination/
```

---

## Text Processing — The Power Tools

These are used constantly in DevOps for parsing logs, config files, and command output.

### `grep` — Search for patterns

```bash
grep "error" /var/log/syslog            # find lines containing "error"
grep -i "error" /var/log/syslog         # case insensitive
grep -r "TODO" /home/alice/             # recursive search
grep -n "failed" auth.log               # show line numbers
grep -v "DEBUG" app.log                 # show lines NOT containing DEBUG
grep -c "404" access.log                # count matching lines
grep -A 3 "ERROR" app.log              # show 3 lines after each match
grep -B 2 "ERROR" app.log              # show 2 lines before each match
grep -E "error|warning|critical" app.log  # extended regex (OR)
grep "^2024" access.log                 # lines starting with 2024
grep "\.conf$" /etc/                    # lines ending with .conf
```

### `awk` — Field-based processing

```bash
# awk thinks of text as rows and columns
# $1 = first field, $2 = second, $NF = last field

awk '{print $1}' access.log             # print first column
awk '{print $1, $4}' access.log         # print first and fourth columns
awk -F: '{print $1}' /etc/passwd        # use : as delimiter, print usernames
awk -F: '{print $1, $3}' /etc/passwd    # username and UID
awk '$3 > 1000' /etc/passwd             # lines where 3rd field > 1000
awk '/ERROR/ {print $0}' app.log        # print lines containing ERROR
awk 'NR==5,NR==10' file.txt             # print lines 5 to 10
awk '{sum += $5} END {print sum}' file  # sum the 5th column
df -h | awk 'NR>1 {print $5, $6}'      # skip header, print usage and mount
```

### `sed` — Stream editor (find and replace)

```bash
sed 's/old/new/' file.txt               # replace first occurrence per line
sed 's/old/new/g' file.txt              # replace all occurrences
sed -i 's/old/new/g' file.txt           # edit file in-place
sed -i.bak 's/old/new/g' file.txt       # in-place with backup
sed -n '5,10p' file.txt                 # print only lines 5–10
sed '/^#/d' nginx.conf                  # delete comment lines
sed '/^$/d' file.txt                    # delete empty lines
sed 's/[[:space:]]*$//' file.txt        # remove trailing whitespace

# Real world: change a port in a config file
sed -i 's/port 8080/port 9090/g' app.conf
```

### `sort` and `uniq`

```bash
sort file.txt                           # alphabetical sort
sort -n file.txt                        # numeric sort
sort -r file.txt                        # reverse
sort -k2 file.txt                       # sort by second field
sort -t: -k3 -n /etc/passwd             # sort passwd by UID

uniq file.txt                           # remove consecutive duplicates
uniq -c file.txt                        # count occurrences
sort file.txt | uniq -c | sort -rn      # count + rank by frequency
```

### `cut` — Extract columns

```bash
cut -d: -f1 /etc/passwd                 # extract first field, : delimiter
cut -d: -f1,3 /etc/passwd               # extract fields 1 and 3
cut -c1-10 file.txt                     # extract first 10 characters per line
```

---

## Pipes and Redirection

```bash
# Pipe: send output of one command to another
ls -la | grep ".conf"
cat /var/log/syslog | grep ERROR | tail -20
ps aux | grep nginx | grep -v grep

# Redirect output
command > file.txt          # stdout to file (overwrite)
command >> file.txt         # stdout to file (append)
command 2> errors.txt       # stderr to file
command 2>&1                # redirect stderr to stdout
command > output.txt 2>&1   # both stdout and stderr to file
command &> output.txt       # shorthand for both

# Discard output
command > /dev/null          # discard stdout
command > /dev/null 2>&1     # discard everything

# Read from file
command < input.txt

# Real examples
# Find all failed SSH attempts and count by IP
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Find biggest directories
du -sh /* 2>/dev/null | sort -rh | head -10
```

---

## File Permissions (Preview — Full Detail in Note 03)

```bash
ls -la
# -rw-r--r-- 1 alice devops 1234 Jan 10 12:00 file.txt
#  ││││││││└─ world: r--
#  │││││└──── group: r--
#  ││└─────── owner: rw-
#  │└──────── type: - (file), d (dir), l (symlink)
```

---

## Common File Paths DevOps Engineers Visit Daily

```bash
/etc/hostname                   # machine's hostname
/etc/hosts                      # local DNS overrides
/etc/resolv.conf                # DNS server config
/etc/fstab                      # filesystems to mount at boot
/etc/crontab                    # system cron jobs
/etc/passwd                     # user accounts
/etc/group                      # group definitions
/etc/sudoers                    # sudo permissions
/var/log/syslog                 # general system logs
/var/log/auth.log               # authentication logs (Ubuntu)
/var/log/secure                 # authentication logs (RHEL)
/var/log/nginx/                 # nginx logs
/var/log/journal/               # systemd journal
/tmp/                           # temporary files (cleared on reboot)
/proc/cpuinfo                   # CPU information
/proc/meminfo                   # memory information
/proc/net/                      # network statistics
```
