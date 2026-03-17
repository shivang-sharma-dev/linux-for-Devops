# Task 09 — Logs & Monitoring

> **Prerequisite:** Read `01-notes/10-logs-and-monitoring.md`
> **Environment:** Full Linux system (native, WSL2, or VM). You need `sudo` access.
> **Difficulty:** Intermediate

---

## Objective

By the end of this lab you will be able to:
- Find and read system logs
- Use `journalctl` effectively to filter and search logs
- Analyze logs with `grep`, `awk`, and `sort`
- Monitor logs in real-time
- Configure log rotation
- Build a simple log analysis script

---

## Part 1 — Where Are the Logs?

### Exercise 1.1: Explore /var/log

```bash
# List all log files
ls -la /var/log/

# Common log files
sudo cat /var/log/syslog | tail -20         # General system log (Debian/Ubuntu)
sudo cat /var/log/auth.log | tail -20       # Authentication events
sudo cat /var/log/kern.log | tail -20       # Kernel messages
sudo cat /var/log/dpkg.log | tail -20       # Package manager log
```

### Exercise 1.2: Log file sizes

```bash
# Which logs are taking the most space?
sudo du -sh /var/log/* 2>/dev/null | sort -rh | head -10
```

---

## Part 2 — journalctl

### Exercise 2.1: Basic usage

```bash
# All logs (paged)
sudo journalctl

# Last 50 lines
sudo journalctl -n 50

# Follow in real-time (like tail -f)
sudo journalctl -f
# (Ctrl+C to stop)

# Logs from current boot only
sudo journalctl -b

# Logs from previous boot
sudo journalctl -b -1

# List all boots
sudo journalctl --list-boots
```

### Exercise 2.2: Filter by unit/service

```bash
# Logs for a specific service
sudo journalctl -u ssh -n 30
sudo journalctl -u cron -n 30

# Multiple services
sudo journalctl -u ssh -u cron -n 30

# Kernel messages only
sudo journalctl -k
```

### Exercise 2.3: Filter by time

```bash
# Since a specific time
sudo journalctl --since "2024-01-01 00:00:00"

# Relative time
sudo journalctl --since "1 hour ago"
sudo journalctl --since "30 min ago"
sudo journalctl --since today

# Time range
sudo journalctl --since "1 hour ago" --until "30 min ago"
```

### Exercise 2.4: Filter by priority

```bash
# Priority levels:
# 0=emerg, 1=alert, 2=crit, 3=err, 4=warning, 5=notice, 6=info, 7=debug

# Only errors and above
sudo journalctl -p err

# Only critical
sudo journalctl -p crit

# Warnings and above, since last boot
sudo journalctl -p warning -b
```

### Exercise 2.5: Output formats

```bash
# JSON output (useful for parsing)
sudo journalctl -n 5 -o json-pretty

# Short output with timestamps
sudo journalctl -n 5 -o short-iso

# Verbose output (all fields)
sudo journalctl -n 1 -o verbose
```

---

## Part 3 — Searching and Analyzing Logs

### Exercise 3.1: Search with grep

```bash
# Find SSH login failures
sudo grep "Failed password" /var/log/auth.log | tail -10

# Count failed logins
sudo grep -c "Failed password" /var/log/auth.log

# Find errors in syslog
sudo grep -i "error" /var/log/syslog | tail -20

# Find with context (3 lines before and after)
sudo grep -B 3 -A 3 "error" /var/log/syslog | tail -30

# Search across multiple log files
sudo grep -r "error" /var/log/*.log 2>/dev/null | tail -10
```

### Exercise 3.2: Search with journalctl

```bash
# Search for a specific string
sudo journalctl -b --grep "error"

# Case-insensitive search
sudo journalctl -b --grep "failed" --case-sensitive=no

# Combine filters
sudo journalctl -u ssh -b --grep "Accepted"
```

### Exercise 3.3: Analyze with awk and sort

```bash
# Count log entries per hour (syslog format)
sudo awk '{print $1, $2, substr($3,1,2)":00"}' /var/log/syslog | sort | uniq -c | sort -rn | head -10

# Top processes generating logs
sudo journalctl -b --no-pager | awk '{print $5}' | sort | uniq -c | sort -rn | head -10

# Failed SSH attempts by IP (if any exist)
sudo grep "Failed password" /var/log/auth.log 2>/dev/null | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head -10
```

---

## Part 4 — Real-Time Monitoring

### Exercise 4.1: Follow logs live

```bash
# Follow system log
sudo tail -f /var/log/syslog

# Follow with journalctl (better)
sudo journalctl -f

# Follow a specific service
sudo journalctl -u ssh -f

# Follow and filter
sudo journalctl -f | grep --line-buffered "error"
```

### Exercise 4.2: Generate some log entries

Open a second terminal and generate activity while watching logs:

```bash
# Terminal 1: Watch auth logs
sudo journalctl -u ssh -f

# Terminal 2: Generate activity
sudo systemctl restart ssh
logger "Test message from lab-09"
logger -p auth.warning "Warning: test warning message"
```

### Exercise 4.3: Custom log messages with `logger`

```bash
# Send a message to syslog
logger "Application deployment started"

# Send with a specific priority
logger -p local0.info "Deployment: version 2.0 deployed"
logger -p user.err "Deployment: migration failed"

# Tag your messages
logger -t myapp "Server health check passed"

# Verify
sudo journalctl -t myapp -n 5
```

---

## Part 5 — Log Rotation

### Exercise 5.1: Understand logrotate

```bash
# View logrotate configuration
cat /etc/logrotate.conf

# View service-specific configs
ls /etc/logrotate.d/
cat /etc/logrotate.d/apt

# Force a logrotate run (dry run)
sudo logrotate -d /etc/logrotate.conf 2>&1 | head -30

# Force actual rotation
sudo logrotate -f /etc/logrotate.conf
```

### Exercise 5.2: Create a custom logrotate config

```bash
# Create a log file to rotate
sudo mkdir -p /var/log/myapp
sudo bash -c 'for i in $(seq 1 1000); do echo "Log line $i" >> /var/log/myapp/app.log; done'

# Check size
ls -lh /var/log/myapp/app.log

# Create logrotate config
sudo tee /etc/logrotate.d/myapp << 'EOF'
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 644 root root
}
EOF

# Test it (dry run)
sudo logrotate -d /etc/logrotate.d/myapp

# Force rotation
sudo logrotate -f /etc/logrotate.d/myapp

# Check — you should see rotated files
ls -la /var/log/myapp/
```

---

## Part 6 — Journal Maintenance

### Exercise 6.1: Manage journal size

```bash
# Check journal disk usage
sudo journalctl --disk-usage

# Clean old entries — keep last 7 days
sudo journalctl --vacuum-time=7d

# Or limit by size — keep max 500MB
sudo journalctl --vacuum-size=500M

# Check persistent journal config
cat /etc/systemd/journald.conf
```

---

## Part 7 — Build a Log Analysis Script

```bash
mkdir -p ~/lab-09

cat > ~/lab-09/log-analyzer.sh << 'SCRIPT'
#!/bin/bash
set -euo pipefail

echo "============================================"
echo "         LOG ANALYSIS REPORT"
echo "  Generated: $(date)"
echo "============================================"

# 1. Recent errors
echo ""
echo "--- Recent Errors (last 1 hour) ---"
error_count=$(sudo journalctl --since "1 hour ago" -p err --no-pager 2>/dev/null | wc -l)
echo "Total error entries: $error_count"
if [ "$error_count" -gt 0 ]; then
    echo "Latest errors:"
    sudo journalctl --since "1 hour ago" -p err --no-pager -n 5 2>/dev/null
fi

# 2. Failed services
echo ""
echo "--- Failed Services ---"
failed=$(systemctl list-units --type=service --state=failed --no-pager --no-legend)
if [ -z "$failed" ]; then
    echo "No failed services"
else
    echo "$failed"
fi

# 3. Auth failures
echo ""
echo "--- Authentication Failures (today) ---"
if [ -f /var/log/auth.log ]; then
    fail_count=$(sudo grep -c "Failed password" /var/log/auth.log 2>/dev/null || echo 0)
    echo "Failed password attempts: $fail_count"
else
    echo "No auth.log found"
fi

# 4. Disk space check
echo ""
echo "--- Disk Space ---"
df -h / | awk 'NR==2{printf "Usage: %s of %s (%s used)\n", $3, $2, $5}'

# 5. Top log-producing services
echo ""
echo "--- Top 5 Log-Producing Services (this boot) ---"
sudo journalctl -b --no-pager 2>/dev/null | awk '{print $5}' | sed 's/\[.*//;s/://' | sort | uniq -c | sort -rn | head -5

echo ""
echo "============================================"
echo "         END OF REPORT"
echo "============================================"
SCRIPT

chmod +x ~/lab-09/log-analyzer.sh
~/lab-09/log-analyzer.sh
```

---

## Challenges

1. **Find the top 5 IP addresses** that attempted failed SSH logins on your system

2. **Create a monitoring script** that checks logs every 60 seconds and sends an alert (prints to console) if it finds any new "error" or "critical" messages

3. **Calculate how many log entries** were generated per service in the last 24 hours, sorted by count

4. **Set up journal persistence** across reboots:
   ```bash
   sudo mkdir -p /var/log/journal
   sudo systemctl restart systemd-journald
   ```

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Only checking `/var/log/syslog` | Many services log to journal, not files | Use `journalctl -u service` first |
| Not using `-b` flag | Gets logs from previous boots too | Add `-b` for current boot only |
| Grepping without context | Hard to understand what caused the error | Use `grep -B 3 -A 3` for context |
| Letting logs fill the disk | `/var/log` can consume all space | Set up logrotate and journal size limits |
| Not following logs during debugging | Missing real-time events | Use `journalctl -f -u service` while testing |

---

## Cleanup

```bash
sudo rm -rf /var/log/myapp
sudo rm -f /etc/logrotate.d/myapp
rm -rf ~/lab-09
```

---

## What's Next?

Phase 2 complete! Move on to **Phase 3** → `task-10-ssh-setup.md` — SSH keys, config, and remote access.
