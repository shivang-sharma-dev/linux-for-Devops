# 10 — Logs and Monitoring

> When something breaks in production, logs are your only witness. Knowing how to find, read, and parse logs quickly is the difference between a 2-minute fix and a 2-hour outage.

---

## Where Logs Live

```
/var/log/
├── syslog              General system log (Ubuntu)
├── messages            General system log (RHEL)
├── auth.log            Authentication: SSH logins, sudo usage (Ubuntu)
├── secure              Authentication log (RHEL)
├── kern.log            Kernel messages
├── dmesg               Hardware detection, boot messages
├── apt/                apt install/upgrade history
├── nginx/
│   ├── access.log      Every HTTP request
│   └── error.log       Nginx errors
├── mysql/              MySQL logs
├── postgresql/         PostgreSQL logs
└── journal/            systemd journal (binary)
```

---

## Reading Logs

```bash
tail -f /var/log/syslog                     # live follow
tail -n 100 /var/log/auth.log               # last 100 lines
grep "Failed password" /var/log/auth.log    # SSH failures
grep -i "error" /var/log/nginx/error.log    # case insensitive
grep "Jan 15" /var/log/syslog               # filter by date

# Multiple files
tail -f /var/log/nginx/access.log /var/log/nginx/error.log

# Compressed rotated logs
zcat /var/log/syslog.2.gz | grep "error"
zgrep "error" /var/log/syslog.*.gz
```

---

## journalctl (systemd journal)

```bash
journalctl -f                           # live follow all
journalctl -u nginx -f                  # follow nginx only
journalctl -p err..emerg                # errors and worse
journalctl --since "10 minutes ago"     # recent logs
journalctl -b 0                         # current boot
journalctl -b -1                        # previous boot (after crash)
journalctl -o json-pretty -u nginx -n 5 # JSON output
```

---

## Log Analysis Patterns

```bash
# Count HTTP status codes from nginx access log
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Top 10 IPs making requests
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Find all failed SSH login attempts
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Find slowest requests (if nginx has request time in log)
awk '{print $NF, $7}' /var/log/nginx/access.log | sort -rn | head -20

# Count errors by hour
grep "error" /var/log/nginx/error.log | awk '{print $2}' | cut -d: -f1 | sort | uniq -c
```

---

## Log Rotation — logrotate

```bash
cat /etc/logrotate.conf             # main config
ls /etc/logrotate.d/                # per-app configs

# Example /etc/logrotate.d/nginx
# /var/log/nginx/*.log {
#     daily                 rotate daily
#     missingok             don't error if file missing
#     rotate 14             keep 14 rotations
#     compress              gzip old logs
#     delaycompress         wait one rotation before compressing
#     notifempty            don't rotate if empty
#     sharedscripts
#     postrotate
#         nginx -s reopen   tell nginx to reopen log files
#     endscript
# }

# Test rotation config
sudo logrotate -d /etc/logrotate.d/nginx    # dry run
sudo logrotate -f /etc/logrotate.d/nginx    # force rotation now
```

---

## System Monitoring Commands

```bash
# CPU, memory, I/O — live
top / htop

# Memory breakdown
free -h
cat /proc/meminfo | grep -E "MemTotal|MemFree|MemAvailable|Cached"

# CPU info
lscpu
cat /proc/cpuinfo | grep "model name" | uniq
mpstat 1 5                          # CPU stats every 1s, 5 times

# I/O
iostat -x 1 5                       # I/O stats per disk
iotop                               # I/O per process (like top)

# Network
iftop                               # network by connection
nethogs                             # network by process
vnstat                              # historical bandwidth
sar -n DEV 1 5                      # network interface stats

# All-in-one
vmstat 1 5                          # virtual memory stats
sar -u 1 5                          # CPU utilization via sysstat
```

---

## Writing Your Own Monitor Script

```bash
#!/bin/bash
# Simple alert script — run via cron

ALERT_EMAIL="admin@example.com"
CPU_THRESHOLD=80
DISK_THRESHOLD=90
MEM_THRESHOLD=85

alert() {
    echo "$1" | mail -s "SERVER ALERT: $HOSTNAME" $ALERT_EMAIL
    logger -t monitor-alert "$1"    # also write to syslog
}

# CPU
cpu=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d. -f1)
[ "$cpu" -gt "$CPU_THRESHOLD" ] && alert "HIGH CPU: ${cpu}% on $HOSTNAME"

# Disk
df -h | awk 'NR>1' | while read fs size used avail pct mount; do
    usage=${pct/\%/}
    [ "$usage" -gt "$DISK_THRESHOLD" ] && alert "HIGH DISK: $pct on $mount ($HOSTNAME)"
done

# Memory
mem=$(free | awk '/Mem/{printf "%.0f", $3/$2*100}')
[ "$mem" -gt "$MEM_THRESHOLD" ] && alert "HIGH MEMORY: ${mem}% on $HOSTNAME"
```
