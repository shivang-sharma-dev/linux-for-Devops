# Task 08 — Systemd & Services

> **Prerequisite:** Read `01-notes/09-systemd-and-services.md`
> **Environment:** Full Linux system with systemd (native Linux or VM — NOT a Docker container, as most containers don't run systemd). You need `sudo` access.
> **Difficulty:** Intermediate

---

## Objective

By the end of this lab you will be able to:
- Manage services with `systemctl`
- Check service status, logs, and dependencies
- Write a custom systemd service unit file
- Enable services to start at boot
- Understand service types and restart policies

---

## Part 1 — Managing Existing Services

### Exercise 1.1: Check service status

```bash
# Check status of SSH service
sudo systemctl status sshd
# or on Ubuntu:
sudo systemctl status ssh

# Check cron
sudo systemctl status cron

# Quick checks
systemctl is-active sshd
systemctl is-enabled sshd
```

### Exercise 1.2: Start, stop, restart services

```bash
# Stop a service
sudo systemctl stop cron
systemctl is-active cron    # Should show "inactive"

# Start it again
sudo systemctl start cron
systemctl is-active cron    # Should show "active"

# Restart (stop then start)
sudo systemctl restart cron

# Reload config without restarting (not all services support this)
sudo systemctl reload cron
# If unsure, use reload-or-restart:
sudo systemctl reload-or-restart cron
```

### Exercise 1.3: Enable and disable services

```bash
# Disable a service from starting at boot
sudo systemctl disable cron
systemctl is-enabled cron    # Should show "disabled"

# Enable it again
sudo systemctl enable cron
systemctl is-enabled cron    # Should show "enabled"

# Enable AND start in one command
sudo systemctl enable --now cron
```

---

## Part 2 — List and Explore Services

### Exercise 2.1: List all services

```bash
# List all loaded services
systemctl list-units --type=service

# List only running services
systemctl list-units --type=service --state=running

# List failed services
systemctl list-units --type=service --state=failed

# List all installed service files
systemctl list-unit-files --type=service
```

### Exercise 2.2: Service dependencies

```bash
# What does this service depend on?
systemctl list-dependencies sshd

# What depends on this service?
systemctl list-dependencies --reverse sshd

# Show the service file location
systemctl show sshd -p FragmentPath
```

### Exercise 2.3: Read a service file

```bash
# View the unit file
systemctl cat sshd

# Or find and read it directly
cat /lib/systemd/system/ssh.service
```

---

## Part 3 — Service Logs with journalctl

### Exercise 3.1: View logs

```bash
# All logs for a service
sudo journalctl -u sshd

# Last 20 lines
sudo journalctl -u sshd -n 20

# Follow logs in real-time
sudo journalctl -u sshd -f
# (Ctrl+C to stop)

# Logs since last boot
sudo journalctl -u sshd -b

# Logs from a specific time
sudo journalctl -u sshd --since "1 hour ago"
sudo journalctl -u sshd --since "2024-01-01" --until "2024-01-02"
```

### Exercise 3.2: System-wide logs

```bash
# All logs since last boot
sudo journalctl -b

# Kernel messages only
sudo journalctl -k

# Show only errors and above
sudo journalctl -p err

# Disk usage of journal
sudo journalctl --disk-usage

# Clean old logs (keep last 7 days)
sudo journalctl --vacuum-time=7d
```

---

## Part 4 — Write a Custom Service

### Exercise 4.1: Create a simple application

First, create a script that our service will run:

```bash
sudo mkdir -p /opt/myapp

sudo tee /opt/myapp/server.sh << 'EOF'
#!/bin/bash
LOG_FILE="/var/log/myapp.log"

echo "$(date): MyApp starting..." >> "$LOG_FILE"

while true; do
    echo "$(date): MyApp is running. PID=$$" >> "$LOG_FILE"
    sleep 10
done
EOF

sudo chmod +x /opt/myapp/server.sh
```

### Exercise 4.2: Write the unit file

```bash
sudo tee /etc/systemd/system/myapp.service << 'EOF'
[Unit]
Description=My Custom Application
After=network.target

[Service]
Type=simple
ExecStart=/opt/myapp/server.sh
ExecStop=/bin/kill -SIGTERM $MAINPID
Restart=on-failure
RestartSec=5
User=nobody
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF
```

**Section breakdown:**

| Section | Field | Meaning |
|---------|-------|---------|
| `[Unit]` | Description | Human-readable name |
| | After | Start after these units are ready |
| `[Service]` | Type | `simple` (default), `forking`, `oneshot`, `notify` |
| | ExecStart | Command to start the service |
| | Restart | When to auto-restart: `no`, `on-failure`, `always` |
| | RestartSec | Wait time before restarting |
| | User | Run as this user |
| `[Install]` | WantedBy | Which target enables this service |

### Exercise 4.3: Manage your custom service

```bash
# Reload systemd to pick up the new file
sudo systemctl daemon-reload

# Start the service
sudo systemctl start myapp

# Check status
sudo systemctl status myapp

# Check logs
sudo journalctl -u myapp -n 10

# Check the log file
sudo cat /var/log/myapp.log

# Enable at boot
sudo systemctl enable myapp

# Test restart behavior — kill the process
sudo kill $(pgrep -f server.sh)

# Wait 5 seconds, then check — it should auto-restart
sleep 6
sudo systemctl status myapp
```

### Exercise 4.4: Modify and reload

```bash
# Edit the service file
sudo systemctl edit myapp --full

# Or edit directly and reload
sudo nano /etc/systemd/system/myapp.service

# After editing, always reload
sudo systemctl daemon-reload
sudo systemctl restart myapp
```

---

## Part 5 — Service Types

### Exercise 5.1: Create a one-shot service

```bash
sudo tee /opt/myapp/cleanup.sh << 'EOF'
#!/bin/bash
echo "$(date): Running cleanup..." >> /var/log/myapp-cleanup.log
find /tmp -type f -mtime +7 -delete 2>/dev/null
echo "$(date): Cleanup complete" >> /var/log/myapp-cleanup.log
EOF

sudo chmod +x /opt/myapp/cleanup.sh

sudo tee /etc/systemd/system/myapp-cleanup.service << 'EOF'
[Unit]
Description=MyApp Temp File Cleanup

[Service]
Type=oneshot
ExecStart=/opt/myapp/cleanup.sh

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl start myapp-cleanup
sudo systemctl status myapp-cleanup    # Will show "inactive (dead)" — that's normal for oneshot
sudo cat /var/log/myapp-cleanup.log
```

### Exercise 5.2: Create a timer (systemd alternative to cron)

```bash
sudo tee /etc/systemd/system/myapp-cleanup.timer << 'EOF'
[Unit]
Description=Run MyApp Cleanup Daily

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now myapp-cleanup.timer

# List all active timers
systemctl list-timers --all
```

---

## Challenges

1. **Create a web-like service:**
   - Write a Python or Bash script that listens on a port (use `python3 -m http.server 8080`)
   - Create a systemd service for it
   - Make it restart automatically on failure
   - Make it start at boot
   - Test by killing the process and verifying it restarts

2. **Investigate your system:**
   - How many services are currently running?
   - Which services failed since last boot?
   - What is the boot order? (`systemd-analyze blame`)

3. **Create a service that depends on another:**
   - Service A writes to a file every 5 seconds
   - Service B depends on Service A (using `Requires=` and `After=`)
   - Test that stopping A also stops B

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Forgetting `daemon-reload` | systemd doesn't see your changes | Always run `sudo systemctl daemon-reload` after editing unit files |
| Wrong `ExecStart` path | Service fails to start | Use absolute paths, test the command manually first |
| Running service as root unnecessarily | Security risk | Set `User=` to a non-root user when possible |
| Using `Type=simple` for forking daemons | systemd thinks the service died immediately | Use `Type=forking` for daemons that fork |
| Not checking logs when service fails | Guessing instead of reading the error | `journalctl -u servicename -n 50` |

---

## Cleanup

```bash
sudo systemctl stop myapp
sudo systemctl disable myapp
sudo systemctl stop myapp-cleanup.timer
sudo systemctl disable myapp-cleanup.timer
sudo rm /etc/systemd/system/myapp.service
sudo rm /etc/systemd/system/myapp-cleanup.service
sudo rm /etc/systemd/system/myapp-cleanup.timer
sudo systemctl daemon-reload
sudo rm -rf /opt/myapp
sudo rm -f /var/log/myapp.log /var/log/myapp-cleanup.log
```

---

## What's Next?

Proceed to `task-09-logs.md` → Reading, searching, and analyzing logs.
