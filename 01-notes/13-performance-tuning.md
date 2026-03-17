# 13 — Performance Tuning

> When a server is slow, you need a systematic approach. Guess randomly and you'll waste hours. Follow this process and you'll find the bottleneck in minutes.

---

## The Diagnostic Process

```
Server is slow
      │
      ▼
Is it CPU-bound?  →  top, mpstat, ps aux --sort=-%cpu
      │
      ▼
Is it Memory-bound?  →  free -h, vmstat, /proc/meminfo
      │
      ▼
Is it Disk I/O bound?  →  iostat, iotop, df -h
      │
      ▼
Is it Network bound?  →  iftop, nethogs, ss -s
      │
      ▼
Is it a specific process?  →  strace, lsof, /proc/PID/
```

---

## CPU Analysis

```bash
top                                 # overall CPU, sortable
htop                                # interactive
mpstat 1 5                          # per-core CPU stats, 5 samples
mpstat -P ALL 1                     # all cores every second
sar -u 1 5                          # historical + live CPU
pidstat 1                           # per-process CPU

# Find top CPU consumers
ps aux --sort=-%cpu | head -10
top -bn1 | head -20

# CPU info
lscpu                               # cores, threads, architecture
cat /proc/cpuinfo | grep "cpu MHz" # current clock speed
```

---

## Memory Analysis

```bash
free -h                             # quick overview
cat /proc/meminfo                   # full details

vmstat 1 5
# procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
# r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
# 1  0      0 512000  45000 900000    0    0     1     5   100  200  5  2 92  1  0
# r = runnable processes, b = blocked on I/O
# si/so = swap in/out (non-zero = memory pressure)
# us/sy/id/wa = user/system/idle/iowait CPU %

# Memory per process
ps aux --sort=-%mem | head -10
smem -rs rss | head -10             # accurate RSS (install smem)

# Check for OOM kills
dmesg | grep -i "oom\|out of memory"
journalctl -k | grep -i "oom"
grep -i "killed process" /var/log/syslog
```

---

## Disk I/O Analysis

```bash
iostat -x 1 5                       # extended disk stats
# %util — how busy the disk is (100% = saturated)
# await — average I/O wait time (ms)
# r/s w/s — reads/writes per second

iotop                               # I/O per process (like top)
iotop -o                            # only show processes doing I/O
iotop -a                            # accumulated I/O

# Find what's writing to disk
sudo iotop -o -P

# Disk latency test
sudo hdparm -tT /dev/sda            # raw read speed

# Find large files
find / -size +100M -type f 2>/dev/null | sort
du -sh /* 2>/dev/null | sort -rh | head -10
```

---

## Network Analysis

```bash
iftop                               # bandwidth by connection (install separately)
nethogs                             # bandwidth by process
sar -n DEV 1 5                      # bytes in/out per interface

ss -s                               # socket statistics summary
ss -tn state established | wc -l   # count established connections

# Packet drops (may indicate network congestion)
ip -s link show eth0
netstat -s | grep -i "failed\|error\|drop"
```

---

## System-Wide Snapshot

```bash
sar -A 1 3                          # everything: CPU, memory, I/O, network
dstat                               # colourful all-in-one (install dstat)
glances                             # modern all-in-one (install glances)
nmon                                # interactive monitor popular on RHEL

# One-liner health snapshot
echo "=== CPU ===" && mpstat 1 1 | tail -2
echo "=== Memory ===" && free -h
echo "=== Disk ===" && df -h
echo "=== Load ===" && uptime
echo "=== Top Processes ===" && ps aux --sort=-%cpu | head -5
```

---

## strace — Trace System Calls

```bash
strace -p 1234                      # attach to running process
strace command                      # trace a command from start
strace -c command                   # summary (counts + times)
strace -e trace=open,read command   # only file operations
strace -f -p 1234                   # follow child processes too

# Find what files a process opens
strace -e trace=open,openat -p 1234 2>&1 | grep "\.conf"
```

---

## lsof — List Open Files

```bash
lsof                                # all open files (huge output)
lsof -p 1234                        # files open by PID 1234
lsof -u alice                       # files open by alice
lsof /var/log/app.log               # who has this file open
lsof -i :80                         # who is using port 80
lsof -i TCP:22                      # TCP connections on port 22
lsof -i                             # all network connections
lsof +D /var/log/                   # all files open in directory

# Find deleted files still held open (eating disk space)
lsof | grep deleted
```

---

## Kernel Tuning — sysctl

```bash
sysctl -a                           # all kernel parameters
sysctl net.ipv4.ip_forward          # check a value
sudo sysctl -w net.ipv4.ip_forward=1    # set temporarily

# Make permanent
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p                      # apply

# Common tuning parameters
net.core.somaxconn = 65535          # max connection backlog
net.ipv4.tcp_max_syn_backlog = 65535
vm.swappiness = 10                  # prefer RAM over swap (10 = mostly RAM)
fs.file-max = 2097152               # max open file handles system-wide
```
