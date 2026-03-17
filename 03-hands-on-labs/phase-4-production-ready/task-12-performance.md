# Task 12 — Performance Tuning & Diagnostics

> **Prerequisite:** Read `01-notes/13-performance-tuning.md`
> **Environment:** Full Linux system (VM recommended for stress testing). You need `sudo` access.
> **Difficulty:** Advanced

---

## Objective

By the end of this lab you will be able to:
- Diagnose high CPU, memory, and disk I/O issues
- Use performance monitoring tools (`top`, `htop`, `vmstat`, `iostat`, `sar`)
- Stress test your system to simulate load
- Identify bottlenecks in a systematic way
- Tune basic kernel parameters with `sysctl`

---

## Part 1 — The Diagnostic Framework

When a server is "slow," the problem is always one of these:

```
1. CPU      → Is the processor overloaded?
2. Memory   → Is the system running out of RAM (swapping)?
3. Disk I/O → Is the disk too slow or overloaded?
4. Network  → Is the network the bottleneck?
```

### Exercise 1.1: Quick system overview

```bash
# One-command overview
uptime          # Load average
free -h         # Memory
df -h /         # Disk space

# All in one
echo "=== LOAD ===" && uptime
echo "=== MEMORY ===" && free -h
echo "=== DISK ===" && df -h /
echo "=== TOP CPU PROCESSES ===" && ps aux --sort=-%cpu | head -6
echo "=== TOP MEM PROCESSES ===" && ps aux --sort=-%mem | head -6
```

---

## Part 2 — CPU Diagnostics

### Exercise 2.1: Monitor CPU usage

```bash
# Load average
uptime
# Three numbers: 1-min, 5-min, 15-min averages
# Rule: if load > number of CPU cores → overloaded

# How many cores do you have?
nproc

# Real-time CPU monitoring
top
# Press 1 to see per-core usage
# Press P to sort by CPU
# Press q to quit

# Snapshot CPU stats
mpstat 1 5      # 1-second interval, 5 iterations
# Install if needed: sudo apt install -y sysstat
```

### Exercise 2.2: Stress test CPU

```bash
# Install stress tool
sudo apt install -y stress

# Use 100% of 2 CPU cores for 30 seconds
stress --cpu 2 --timeout 30 &

# In another terminal, watch the impact
top
# or
watch -n 1 uptime    # Watch load average rise
```

### Exercise 2.3: Find CPU-hungry processes

```bash
# Sort by CPU usage
ps aux --sort=-%cpu | head -10

# Continuous monitoring
pidstat 1 5     # Per-process CPU stats every 1 second

# What system calls is a process making?
strace -c -p <PID>    # Run for a few seconds, then Ctrl+C
# Shows which syscalls are taking the most time
```

---

## Part 3 — Memory Diagnostics

### Exercise 3.1: Understand memory usage

```bash
# Memory overview
free -h

# Detailed breakdown
cat /proc/meminfo | head -20

# Understanding free output:
#               total    used    free    shared  buff/cache   available
# Mem:          7.7G     1.2G    5.0G    100M    1.5G         6.2G
#
# "available" is what matters — it's free + reclaimable cache
# Linux uses free RAM for cache — this is GOOD, not a problem
```

### Exercise 3.2: Stress test memory

```bash
# Allocate 512MB of memory for 30 seconds
stress --vm 1 --vm-bytes 512M --timeout 30 &

# Watch memory usage
watch -n 1 free -h

# Check if system is swapping
vmstat 1 10
# Look at "si" (swap in) and "so" (swap out) columns
# If these are non-zero, the system is swapping — that's bad for performance
```

### Exercise 3.3: Find memory-hungry processes

```bash
# Sort by memory usage
ps aux --sort=-%mem | head -10

# Memory map of a specific process
pmap <PID> | tail -1

# Per-process memory stats
pidstat -r 1 5    # -r for memory
```

### Exercise 3.4: Swap analysis

```bash
# Check swap usage
swapon --show
free -h | grep Swap

# Which processes are using swap?
for pid in /proc/[0-9]*; do
    swap=$(awk '/VmSwap/ {print $2}' "$pid/status" 2>/dev/null)
    if [ -n "$swap" ] && [ "$swap" -gt 0 ]; then
        name=$(awk '/Name/ {print $2}' "$pid/status")
        echo "${swap} KB - $name (PID: $(basename $pid))"
    fi
done 2>/dev/null | sort -rn | head -10
```

---

## Part 4 — Disk I/O Diagnostics

### Exercise 4.1: Monitor disk I/O

```bash
# Install sysstat if not present
sudo apt install -y sysstat

# Disk I/O stats
iostat -xz 1 5
# Key columns:
# %util  → How busy the disk is (100% = saturated)
# await  → Average time for I/O requests (ms)
# r/s, w/s → Reads/writes per second

# Per-process I/O
sudo iotop -o     # -o shows only processes doing I/O
# Install: sudo apt install -y iotop
```

### Exercise 4.2: Stress test disk I/O

```bash
# Write stress test
stress --io 4 --timeout 30 &

# Or use dd to write a large file
dd if=/dev/zero of=/tmp/disk-test bs=1M count=500 oflag=direct 2>&1

# Watch I/O during the test
iostat -xz 1

# Clean up
rm -f /tmp/disk-test
```

### Exercise 4.3: File system cache

```bash
# Check how much RAM is used for disk cache
free -h | grep "buff/cache"

# Drop caches (for testing only — never in production)
sudo sync
sudo sh -c 'echo 3 > /proc/sys/vm/drop_caches'
free -h    # Cache reduced, but it will rebuild naturally
```

---

## Part 5 — vmstat Deep Dive

### Exercise 5.1: Read vmstat output

```bash
vmstat 1 10    # 1-second interval, 10 samples
```

```
procs ---------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs  us sy id wa
 1  0      0 5242880  65536 1048576   0    0     0     0  100  200  2  1 97  0
```

| Column | Meaning | Warning Sign |
|--------|---------|-------------|
| `r` | Processes waiting for CPU | > nproc |
| `b` | Processes blocked (waiting for I/O) | > 0 consistently |
| `swpd` | Swap used (KB) | Increasing over time |
| `si/so` | Swap in/out | Any non-zero values |
| `bi/bo` | Block device reads/writes | Very high = disk bottleneck |
| `us` | User CPU % | High = application busy |
| `sy` | System CPU % | High = kernel busy |
| `id` | Idle CPU % | Low = overloaded |
| `wa` | CPU waiting for I/O | High = disk bottleneck |

---

## Part 6 — Network Performance

### Exercise 6.1: Network monitoring

```bash
# Bandwidth per interface
sar -n DEV 1 5

# Active connections
ss -s     # Summary stats

# Bandwidth per process
sudo nethogs    # Install: sudo apt install -y nethogs

# Check for packet drops
ip -s link show
# Look for "dropped" or "errors" that are non-zero
```

---

## Part 7 — Kernel Tuning with sysctl

### Exercise 7.1: View current settings

```bash
# List all kernel parameters
sudo sysctl -a | head -30

# Check specific parameter
sysctl vm.swappiness
sysctl net.ipv4.ip_forward

# What is swappiness?
# 0 = avoid swapping as much as possible
# 100 = aggressively swap
# Default is usually 60
```

### Exercise 7.2: Tune a parameter (temporary)

```bash
# Reduce swappiness (keeps more in RAM)
sudo sysctl vm.swappiness=10

# Verify
sysctl vm.swappiness

# This change is temporary — lost after reboot
```

### Exercise 7.3: Make it permanent

```bash
# Add to sysctl config
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.d/99-custom.conf

# Apply without reboot
sudo sysctl --system

# Verify
sysctl vm.swappiness
```

---

## Part 8 — Build a Performance Dashboard Script

```bash
mkdir -p ~/lab-12

cat > ~/lab-12/perf-dashboard.sh << 'SCRIPT'
#!/bin/bash

clear
echo "============================================"
echo "      PERFORMANCE DASHBOARD"
echo "  $(hostname) — $(date)"
echo "============================================"

# CPU
echo ""
echo "--- CPU ---"
echo "Cores: $(nproc)"
uptime | awk -F'load average:' '{print "Load Average:" $2}'
echo "Top CPU consumers:"
ps aux --sort=-%cpu | awk 'NR>1 && NR<=4 {printf "  %-8s %5s%%  %s\n", $1, $3, $11}'

# Memory
echo ""
echo "--- Memory ---"
free -h | awk '
    NR==2 {printf "RAM:  %s total | %s used | %s available\n", $2, $3, $7}
    NR==3 {printf "Swap: %s total | %s used | %s free\n", $2, $3, $4}
'
echo "Top memory consumers:"
ps aux --sort=-%mem | awk 'NR>1 && NR<=4 {printf "  %-8s %5s%%  %s\n", $1, $4, $11}'

# Disk
echo ""
echo "--- Disk ---"
df -h / | awk 'NR==2 {printf "Root: %s total | %s used (%s) | %s free\n", $2, $3, $5, $4}'

# Disk I/O wait
wa=$(vmstat 1 2 | tail -1 | awk '{print $16}')
echo "I/O Wait: ${wa}%"

# Network
echo ""
echo "--- Network ---"
echo "Active connections: $(ss -t | grep ESTAB | wc -l)"
echo "Listening ports: $(ss -tln | grep LISTEN | wc -l)"

# Warnings
echo ""
echo "--- Alerts ---"
load=$(awk '{print $1}' /proc/loadavg)
cores=$(nproc)
if (( $(echo "$load > $cores" | bc -l) )); then
    echo "[WARN] Load ($load) exceeds core count ($cores)"
fi

mem_pct=$(free | awk 'NR==2 {printf "%.0f", ($2-$7)/$2 * 100}')
if [ "$mem_pct" -gt 85 ]; then
    echo "[WARN] Memory usage is at ${mem_pct}%"
fi

disk_pct=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')
if [ "$disk_pct" -gt 80 ]; then
    echo "[WARN] Disk usage is at ${disk_pct}%"
fi

swap_used=$(free | awk 'NR==3 {print $3}')
if [ "$swap_used" -gt 0 ]; then
    echo "[WARN] System is using swap"
fi

echo ""
echo "============================================"
SCRIPT

chmod +x ~/lab-12/perf-dashboard.sh
~/lab-12/perf-dashboard.sh
```

---

## Challenges

1. **Full stress test:**
   - Run `stress --cpu 2 --vm 1 --vm-bytes 256M --io 2 --timeout 60 &`
   - While it runs, use `vmstat 1`, `iostat 1`, and `top` to observe the impact
   - Write down which resource was the bottleneck

2. **Create a monitoring script** that runs every minute (via cron) and logs load average, memory usage, and disk I/O to a CSV file

3. **Find and fix a performance issue:**
   - Create a script that intentionally wastes CPU (infinite loop)
   - Use `top` or `htop` to find it
   - Kill it
   - Document the steps

4. **Compare swappiness values:** Set swappiness to 10, then 60, then 90. Run a memory stress test at each setting and observe when swap starts being used

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Thinking high buff/cache = problem | Linux uses spare RAM for cache — this is normal | Look at "available" memory, not "free" |
| Checking only load average | Load doesn't tell you WHICH resource is bottlenecked | Check CPU, memory, disk I/O, and network separately |
| Using `drop_caches` in production | Destroys cache, causes performance hit | Only use for testing |
| Changing sysctl without understanding | Can cause instability or security issues | Research each parameter before changing |
| Not establishing a baseline | Can't tell if metrics are abnormal | Monitor regularly and know your normal values |

---

## Cleanup

```bash
killall stress 2>/dev/null
rm -rf ~/lab-12
```

---

## What's Next?

Proceed to `task-13-security-hardening.md` → Harden a Linux server.
