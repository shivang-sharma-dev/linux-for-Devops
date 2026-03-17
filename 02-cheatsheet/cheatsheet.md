# Linux for DevOps — Cheatsheet

---

## Navigation
```bash
pwd                        # current directory
ls -la                     # list all files with details
ls -lh                     # human readable sizes
cd /etc                    # absolute path
cd ..                      # up one level
cd ~                       # home directory
cd -                       # previous directory
tree -L 2                  # directory tree 2 levels deep
```

---

## File Operations
```bash
touch file.txt             # create empty file
mkdir -p a/b/c             # create nested dirs
cp -r src/ dest/           # copy directory
mv old.txt new.txt         # rename/move
rm -rf folder/             # force delete directory
ln -s /path/to/file link   # create symlink
stat file.txt              # file details
file report.pdf            # detect file type
```

---

## Viewing Files
```bash
cat file.txt               # print file
less file.txt              # scrollable (q=quit, /=search)
head -n 20 file.txt        # first 20 lines
tail -n 50 file.txt        # last 50 lines
tail -f /var/log/syslog    # follow live
wc -l file.txt             # count lines
```

---

## Finding Files
```bash
find / -name "nginx.conf"          # find by name
find / -type f -name "*.log"       # files only
find / -size +100M                 # larger than 100MB
find / -mtime -7                   # modified last 7 days
find / -user alice                 # owned by alice
which python3                      # where is command
locate nginx.conf                  # fast search (needs updatedb)
```

---

## Text Processing
```bash
grep "error" file.log              # search pattern
grep -i "error" file.log           # case insensitive
grep -r "TODO" /home/              # recursive
grep -n "failed" file.log          # show line numbers
grep -v "DEBUG" file.log           # invert match
grep -E "error|warn" file.log      # OR pattern

awk '{print $1}' file              # print 1st column
awk -F: '{print $1}' /etc/passwd   # custom delimiter
awk '{sum+=$1} END{print sum}'     # sum a column

sed 's/old/new/g' file             # replace all
sed -i 's/old/new/g' file          # in-place edit
sed '/^#/d' file                   # delete comment lines
sed '/^$/d' file                   # delete empty lines

sort file.txt                      # sort
sort -n file.txt                   # numeric sort
sort -rn file.txt                  # reverse numeric
uniq -c file.txt                   # count duplicates
sort file | uniq -c | sort -rn     # frequency ranking
cut -d: -f1 /etc/passwd            # cut field 1
```

---

## Pipes and Redirection
```bash
cmd1 | cmd2                # pipe output to next command
cmd > file.txt             # stdout to file (overwrite)
cmd >> file.txt            # stdout to file (append)
cmd 2> errors.txt          # stderr to file
cmd > out.txt 2>&1         # stdout + stderr to file
cmd > /dev/null 2>&1       # discard all output
cmd1 && cmd2               # run cmd2 only if cmd1 succeeds
cmd1 || cmd2               # run cmd2 only if cmd1 fails
```

---

## Permissions
```bash
chmod 755 file             # rwxr-xr-x
chmod 644 file             # rw-r--r--
chmod 600 file             # rw------- (private)
chmod 400 file             # r-------- (read only)
chmod -R 755 dir/          # recursive
chmod u+x file             # add execute for owner
chmod g-w file             # remove write for group
chmod o+r file             # add read for others

chown alice file           # change owner
chown alice:devops file    # change owner and group
chown -R www-data /var/www/

umask 022                  # default: files=644, dirs=755
umask 027                  # default: files=640, dirs=750
```

### Permission values
| Octal | Symbolic | Use |
|---|---|---|
| 755 | rwxr-xr-x | dirs, executables |
| 644 | rw-r--r-- | regular files |
| 600 | rw------- | private keys |
| 400 | r-------- | AWS .pem keys |

---

## Users and Groups
```bash
whoami                             # current user
id                                 # UID, GID, groups
id alice                           # info for alice

useradd -m -s /bin/bash alice      # create user
passwd alice                       # set password
usermod -aG docker alice           # add to group (ALWAYS use -a)
usermod -aG sudo alice             # give sudo
usermod -L alice                   # lock account
userdel -r alice                   # delete user + home

groupadd devops                    # create group
gpasswd -d alice devops            # remove from group

sudo -l                            # what can I sudo
sudo -u www-data command           # run as another user
su - alice                         # switch to alice
```

---

## Processes
```bash
ps aux                             # all processes
ps aux --sort=-%cpu | head -10     # top CPU
ps aux --sort=-%mem | head -10     # top memory
pgrep -l nginx                     # find PID by name
pidof nginx                        # same

top                                # live monitor
htop                               # better live monitor

kill -15 PID                       # graceful stop (SIGTERM)
kill -9 PID                        # force kill (SIGKILL)
killall nginx                      # kill all by name
pkill -f "python app"              # kill by pattern

command &                          # run in background
Ctrl+Z then bg                     # send to background
fg                                 # bring to foreground
jobs -l                            # list background jobs
nohup command &                    # persist after logout

nice -n 10 command                 # lower priority
renice -n 10 -p PID                # change running priority

uptime                             # load average
# load > number of cores = overloaded
```

### Key signals
| Signal | Number | Use |
|---|---|---|
| SIGHUP | 1 | Reload config |
| SIGINT | 2 | Ctrl+C |
| SIGKILL | 9 | Force kill |
| SIGTERM | 15 | Graceful stop |

---

## Disk and Storage
```bash
df -h                              # filesystem usage
du -sh folder/                     # folder size
du -sh * | sort -rh | head -10     # biggest items
ncdu /                             # interactive disk usage

lsblk                              # list block devices
lsblk -f                           # with filesystem types
blkid                              # UUIDs and types

mount /dev/sdb1 /mnt/data          # mount
umount /mnt/data                   # unmount
mount -a                           # mount all in fstab

# Extend disk after resizing in cloud
sudo growpart /dev/xvda 1          # extend partition
sudo resize2fs /dev/xvda1          # extend ext4
sudo xfs_growfs /                  # extend XFS

# Swap
swapon --show                      # check swap
fallocate -l 2G /swapfile          # create swap file
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

---

## Networking
```bash
ip a                               # show interfaces + IPs
ip route                           # routing table
ip link show                       # interface state

ss -tulnp                          # listening ports + process
ss -tn state established           # active connections
ss -s                              # summary

ping -c 4 google.com               # test connectivity
traceroute google.com              # trace route
mtr google.com                     # live traceroute

nslookup google.com                # DNS lookup
dig google.com                     # detailed DNS
dig +short google.com              # just the IP
cat /etc/hosts                     # local overrides
cat /etc/resolv.conf               # DNS servers

curl -I https://example.com        # headers only
curl -s -o /dev/null -w "%{http_code}" https://example.com  # status code only
wget -q -O- https://example.com    # fetch to stdout

# UFW firewall
ufw status verbose
ufw allow 22/tcp
ufw allow from 10.0.0.0/8 to any port 22
ufw deny 8080
ufw enable
```

---

## Package Management
```bash
# apt (Ubuntu/Debian)
apt update                         # refresh index
apt upgrade -y                     # upgrade all
apt install nginx                  # install
apt remove nginx                   # remove
apt purge nginx                    # remove + config
apt autoremove                     # remove orphans
apt search nginx                   # search
apt show nginx                     # package info
dpkg -l | grep nginx               # is it installed?
dpkg -L nginx                      # list installed files

# dnf (RHEL/CentOS)
dnf update -y
dnf install nginx
dnf remove nginx
rpm -qa | grep nginx
```

---

## Systemd and Services
```bash
systemctl start nginx              # start
systemctl stop nginx               # stop
systemctl restart nginx            # restart
systemctl reload nginx             # reload config
systemctl status nginx             # status + recent logs
systemctl enable nginx             # start on boot
systemctl disable nginx            # don't start on boot
systemctl enable --now nginx       # enable + start now
systemctl is-active nginx          # running?
systemctl is-enabled nginx         # enabled?

systemctl list-units --type=service --state=running
systemctl list-units --type=service --state=failed
systemctl daemon-reload            # after editing unit files

journalctl -u nginx -f             # follow service logs
journalctl -u nginx -n 50          # last 50 lines
journalctl -u nginx -p err         # errors only
journalctl -b                      # current boot logs
journalctl -b -1                   # previous boot logs
journalctl --since "1 hour ago"
```

### Minimal unit file
```ini
[Unit]
Description=My App
After=network.target

[Service]
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

---

## SSH
```bash
ssh user@host                      # connect
ssh -p 2222 user@host              # custom port
ssh -i ~/.ssh/key.pem user@host    # specify key
ssh user@host "df -h"              # run remote command

ssh-keygen -t ed25519 -C "comment" # generate key pair
ssh-copy-id user@host              # copy key to server
ssh -T git@github.com              # test GitHub auth

scp file.txt user@host:/tmp/       # copy to remote
scp user@host:/tmp/file.txt ./     # copy from remote
scp -r folder/ user@host:/opt/     # copy directory

rsync -avz folder/ user@host:/dest/        # sync to remote
rsync -avz --delete src/ user@host:/dest/  # mirror

# tmux
tmux new -s mysession              # new named session
tmux attach -t mysession           # reattach
tmux ls                            # list sessions
Ctrl+B d                           # detach
Ctrl+B c                           # new window
Ctrl+B %                           # split vertical
Ctrl+B "                           # split horizontal
```

---

## Cron
```bash
crontab -e                         # edit crontab
crontab -l                         # list crontab

# Syntax: MIN HOUR DOM MON DOW command
0 2 * * *      # daily at 2am
*/5 * * * *    # every 5 minutes
0 9-17 * * 1-5 # every hour 9-5, weekdays
@reboot        # on startup
@daily         # once a day

# Always use full paths and redirect output
0 2 * * * /usr/bin/python3 /opt/scripts/backup.py >> /var/log/backup.log 2>&1
```

---

## Logs
```bash
tail -f /var/log/syslog            # follow system log
tail -f /var/log/auth.log          # follow auth log
grep "Failed password" /var/log/auth.log   # SSH failures
grep -i error /var/log/nginx/error.log

# Top IPs hitting nginx
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head

# HTTP status code counts
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Failed SSH login IPs
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn
```

---

## Performance
```bash
uptime                             # load average
top / htop                         # CPU + memory live
mpstat 1 5                         # per-CPU stats
free -h                            # memory
vmstat 1 5                         # memory + I/O + CPU
iostat -x 1 5                      # disk I/O
iotop                              # I/O per process
iftop                              # network by connection
nethogs                            # network by process

lsof -p PID                        # files open by process
lsof -i :80                        # who uses port 80
lsof | grep deleted                # deleted files still open

strace -p PID                      # trace system calls
strace -c command                  # system call summary
```

---

## Security
```bash
# fail2ban
fail2ban-client status             # overall status
fail2ban-client status sshd        # SSH jail
fail2ban-client set sshd unbanip 1.2.3.4

# Find SUID files (review these)
find / -perm /4000 -type f 2>/dev/null

# World-writable files (should be rare)
find / -perm -o+w -type f 2>/dev/null | grep -v /proc

# Active listening services
ss -tulnp

# Logged in users
who
w
last | head -20
```

---

## Shell Scripting
```bash
#!/bin/bash
set -euo pipefail                  # exit on error, unset vars, pipe fails

VAR="value"                        # assign
echo "$VAR"                        # use
CMD=$(command)                     # capture output

# Conditionals
if [ -f file ]; then echo "exists"; fi
if [ "$a" = "$b" ]; then ...; fi
if [ $n -gt 10 ]; then ...; fi

# Loops
for i in {1..5}; do echo $i; done
for f in *.log; do echo $f; done
while [ $n -lt 10 ]; do ((n++)); done
while IFS= read -r line; do echo "$line"; done < file

# Functions
myfunc() { local arg=$1; echo "$arg"; }
myfunc "hello"

# Exit codes
command && echo "ok" || echo "failed"
$?    # exit code of last command

# Common file tests
[ -f file ]   # is file
[ -d dir ]    # is directory
[ -e path ]   # exists
[ -z "$var" ] # empty string
[ -n "$var" ] # non-empty string
```

---

## Archiving
```bash
tar -czf archive.tar.gz folder/    # create compressed
tar -xzf archive.tar.gz            # extract
tar -xzf archive.tar.gz -C /dest/  # extract to dir
tar -tzf archive.tar.gz            # list contents
zip -r archive.zip folder/         # zip
unzip archive.zip -d /dest/        # unzip
```

---

## Key File Locations
```
/etc/hosts          local DNS overrides
/etc/resolv.conf    DNS servers
/etc/fstab          filesystems to mount at boot
/etc/passwd         user accounts
/etc/shadow         password hashes
/etc/group          group definitions
/etc/sudoers        sudo permissions
/etc/ssh/sshd_config  SSH server config
/etc/crontab        system cron jobs
/etc/systemd/system/  custom unit files
/var/log/syslog     general system log
/var/log/auth.log   authentication log
/var/log/nginx/     nginx logs
/proc/cpuinfo       CPU info
/proc/meminfo       memory info
/proc/PID/          per-process info
```
