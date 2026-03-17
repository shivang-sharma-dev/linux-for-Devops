# Task 11 — Cron & Automation

> **Prerequisite:** Read `01-notes/12-cron-and-automation.md`
> **Environment:** Full Linux system with cron (native, WSL2, or VM). You need `sudo` access.
> **Difficulty:** Intermediate

---

## Objective

By the end of this lab you will be able to:
- Understand cron syntax
- Create, list, edit, and remove cron jobs
- Verify that cron jobs actually run
- Debug cron jobs that aren't working
- Use `at` for one-time scheduled tasks
- Build practical automated scripts with cron

---

## Part 1 — Understanding Cron Syntax

### Exercise 1.1: Cron format

```
┌───────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌───────────── day of month (1-31)
│ │ │ ┌───────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0-7, 0 and 7 = Sunday)
│ │ │ │ │
* * * * *  command
```

### Exercise 1.2: Read these cron expressions

Try to understand what each one does before checking the answer:

| Expression | Schedule |
|-----------|----------|
| `* * * * *` | Every minute |
| `0 * * * *` | Every hour (at minute 0) |
| `0 9 * * *` | Every day at 9:00 AM |
| `0 9 * * 1` | Every Monday at 9:00 AM |
| `0 0 1 * *` | First day of every month at midnight |
| `*/5 * * * *` | Every 5 minutes |
| `0 9-17 * * 1-5` | Every hour 9 AM–5 PM, Monday–Friday |
| `0 0 * * 0` | Every Sunday at midnight |
| `30 2 * * *` | Every day at 2:30 AM |

### Exercise 1.3: Practice writing cron expressions

Write cron expressions for:

1. Every 15 minutes
2. Every day at 6:30 PM
3. Every weekday (Mon-Fri) at 8:00 AM
4. Twice a day at 6 AM and 6 PM
5. Every Sunday at 3:00 AM

<details>
<summary>Answers</summary>

```
1. */15 * * * *
2. 30 18 * * *
3. 0 8 * * 1-5
4. 0 6,18 * * *
5. 0 3 * * 0
```

</details>

---

## Part 2 — Managing Cron Jobs

### Exercise 2.1: Your crontab

```bash
# View your current crontab
crontab -l

# Edit your crontab
crontab -e
# This opens an editor — don't add anything yet, just look at the format
# Save and exit (:wq in vim, Ctrl+X in nano)

# View root's crontab
sudo crontab -l

# View another user's crontab
sudo crontab -u www-data -l
```

### Exercise 2.2: Create your first cron job

```bash
# Create a script to run
mkdir -p ~/lab-11

cat > ~/lab-11/heartbeat.sh << 'EOF'
#!/bin/bash
echo "$(date '+%Y-%m-%d %H:%M:%S') - Heartbeat: system is alive" >> ~/lab-11/heartbeat.log
EOF

chmod +x ~/lab-11/heartbeat.sh

# Test the script manually first
~/lab-11/heartbeat.sh
cat ~/lab-11/heartbeat.log

# Add a cron job that runs every minute
crontab -l 2>/dev/null > /tmp/mycron
echo "* * * * * $HOME/lab-11/heartbeat.sh" >> /tmp/mycron
crontab /tmp/mycron
rm /tmp/mycron

# Verify the cron job was added
crontab -l

# Wait 2-3 minutes, then check the log
sleep 120
cat ~/lab-11/heartbeat.log
```

### Exercise 2.3: Edit and remove cron jobs

```bash
# Edit crontab (opens editor)
crontab -e
# Change * * * * * to */5 * * * * (every 5 minutes)
# Save and exit

# Verify
crontab -l

# Remove a specific line — edit and delete the line
crontab -e

# Remove ALL cron jobs (be careful!)
# crontab -r
```

---

## Part 3 — System Cron

### Exercise 3.1: Explore system cron locations

```bash
# System-wide crontab
cat /etc/crontab

# Cron directories (scripts here run automatically)
ls /etc/cron.hourly/
ls /etc/cron.daily/
ls /etc/cron.weekly/
ls /etc/cron.monthly/

# Drop-in cron files
ls /etc/cron.d/
```

### Exercise 3.2: Cron directories vs crontab

| Method | When to Use |
|--------|-------------|
| `crontab -e` | Per-user jobs |
| `/etc/cron.d/` | System jobs with custom schedules |
| `/etc/cron.daily/` | Scripts that run once a day |
| `/etc/cron.hourly/` | Scripts that run every hour |

---

## Part 4 — Verify Cron is Working

### Exercise 4.1: Check cron service

```bash
# Is cron running?
sudo systemctl status cron

# Check cron logs
sudo grep CRON /var/log/syslog | tail -20
# or
sudo journalctl -u cron -n 20
```

### Exercise 4.2: Debug a cron job

Common reasons cron jobs fail:

```bash
# 1. PATH issue — cron has minimal PATH
# Add this to top of crontab:
# PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Check what PATH cron uses
cat > ~/lab-11/show-env.sh << 'EOF'
#!/bin/bash
env > ~/lab-11/cron-env.txt
EOF
chmod +x ~/lab-11/show-env.sh

# Add to crontab temporarily
crontab -l 2>/dev/null > /tmp/mycron
echo "* * * * * $HOME/lab-11/show-env.sh" >> /tmp/mycron
crontab /tmp/mycron

# Wait a minute, then check
sleep 70
cat ~/lab-11/cron-env.txt
# Notice: PATH is very limited compared to your interactive shell
```

### Exercise 4.3: Redirect cron output

```bash
# Cron output goes nowhere by default — capture it!

# Redirect stdout and stderr to a log file
# * * * * * /path/to/script.sh >> /var/log/myjob.log 2>&1

# Redirect errors to a separate file
# * * * * * /path/to/script.sh >> /var/log/myjob.log 2>> /var/log/myjob.err

# Discard all output (silent cron)
# * * * * * /path/to/script.sh > /dev/null 2>&1
```

---

## Part 5 — The `at` Command (One-Time Scheduling)

### Exercise 5.1: Schedule a one-time task

```bash
# Install at if needed
sudo apt install -y at
sudo systemctl enable --now atd

# Schedule a command to run in 1 minute
echo "echo 'at job ran at $(date)' >> ~/lab-11/at-test.log" | at now + 1 minute

# Schedule at a specific time
echo "echo 'scheduled task' >> ~/lab-11/at-test.log" | at 14:30
# (use a time that's a few minutes from now)

# List pending jobs
atq

# View a specific job's commands
at -c <job_number>

# Remove a pending job
atrm <job_number>

# Check if it ran
sleep 70
cat ~/lab-11/at-test.log
```

### Exercise 5.2: at time formats

```bash
# Various time formats at accepts:
# at now + 5 minutes
# at now + 2 hours
# at 3:00 PM
# at midnight
# at noon tomorrow
# at 2:00 PM next Friday
```

---

## Part 6 — Build Practical Automation

### Exercise 6.1: Automated disk space checker

```bash
cat > ~/lab-11/disk-check.sh << 'EOF'
#!/bin/bash
THRESHOLD=80
LOGFILE="$HOME/lab-11/disk-alerts.log"

usage=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')

if [ "$usage" -ge "$THRESHOLD" ]; then
    echo "$(date): WARNING - Disk usage is at ${usage}% (threshold: ${THRESHOLD}%)" >> "$LOGFILE"
else
    echo "$(date): OK - Disk usage is at ${usage}%" >> "$LOGFILE"
fi
EOF

chmod +x ~/lab-11/disk-check.sh

# Test manually
~/lab-11/disk-check.sh
cat ~/lab-11/disk-alerts.log

# Schedule: every hour
# 0 * * * * ~/lab-11/disk-check.sh
```

### Exercise 6.2: Automated backup with cron

```bash
cat > ~/lab-11/backup-cron.sh << 'EOF'
#!/bin/bash
set -euo pipefail

SOURCE="$HOME/lab-11"
BACKUP_DIR="$HOME/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M)
BACKUP_FILE="${BACKUP_DIR}/lab-backup_${TIMESTAMP}.tar.gz"
KEEP_DAYS=7

mkdir -p "$BACKUP_DIR"

# Create backup
tar -czf "$BACKUP_FILE" -C "$(dirname "$SOURCE")" "$(basename "$SOURCE")" 2>/dev/null

# Clean up old backups
find "$BACKUP_DIR" -name "lab-backup_*.tar.gz" -mtime +${KEEP_DAYS} -delete

# Log
echo "$(date): Backup created: $BACKUP_FILE ($(du -h "$BACKUP_FILE" | awk '{print $1}'))" >> "$BACKUP_DIR/backup.log"
EOF

chmod +x ~/lab-11/backup-cron.sh

# Test
~/lab-11/backup-cron.sh
ls -la ~/backups/
cat ~/backups/backup.log

# Schedule: daily at 2 AM
# 0 2 * * * ~/lab-11/backup-cron.sh
```

### Exercise 6.3: Log cleanup automation

```bash
cat > ~/lab-11/log-cleanup.sh << 'EOF'
#!/bin/bash
LOG_DIR="$HOME/lab-11"
MAX_SIZE_KB=1024    # 1MB

for logfile in "$LOG_DIR"/*.log; do
    if [ -f "$logfile" ]; then
        size=$(du -k "$logfile" | awk '{print $1}')
        if [ "$size" -ge "$MAX_SIZE_KB" ]; then
            # Keep last 100 lines, rotate the rest
            tail -100 "$logfile" > "${logfile}.tmp"
            mv "${logfile}.tmp" "$logfile"
            echo "$(date): Rotated $logfile (was ${size}KB)" >> "$LOG_DIR/cleanup.log"
        fi
    fi
done
EOF

chmod +x ~/lab-11/log-cleanup.sh

# Schedule: daily at 3 AM
# 0 3 * * * ~/lab-11/log-cleanup.sh
```

---

## Challenges

1. **Create a cron job that:**
   - Runs every 5 minutes
   - Checks if a specific service (e.g., `ssh`) is running
   - If it's not running, restarts it and logs the event

2. **Create a cron job that runs only on weekdays** and writes the top 5 memory-consuming processes to a log file

3. **Create a "cron tester"** — a script that takes a cron expression and a command as arguments, shows when it would next run, and asks for confirmation before installing it

4. **Investigate:** What cron jobs are already running on your system? Check all users and all system cron directories

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Forgetting full paths in cron | Cron has minimal `$PATH` | Use `/usr/bin/python3` not `python3` |
| Not making scripts executable | Cron can't run them | `chmod +x script.sh` |
| Not redirecting output | Silent failures, or email floods | Always add `>> logfile 2>&1` |
| Using `~` or `$HOME` in crontab | May not expand in all cron implementations | Use full absolute paths |
| `crontab -r` accidentally | Deletes ALL your cron jobs | Use `crontab -e` to remove specific lines |
| Not testing manually first | Cron runs it and it fails silently | Always run the script manually before scheduling |

---

## Cleanup

```bash
# Remove all cron jobs we added
crontab -r 2>/dev/null

# Remove lab files
rm -rf ~/lab-11
rm -rf ~/backups
```

---

## What's Next?

Phase 3 complete! Move on to **Phase 4** → `task-12-performance.md` — Performance tuning and diagnostics.
