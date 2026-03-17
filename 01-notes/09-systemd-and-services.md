# 09 — Systemd and Services

> Modern Linux runs on systemd. Every application you deploy will be managed as a systemd service. This is the most practically important topic for running apps in production.

---

## What is systemd?

systemd is PID 1 — the first process started by the kernel. It manages everything that starts up after the kernel:

```
Kernel boots
    ↓
systemd (PID 1) starts
    ↓
systemd reads unit files from /etc/systemd/system/ and /lib/systemd/system/
    ↓
Starts services in parallel (faster than old init scripts)
    ↓
System is ready
```

---

## systemctl — The Main Tool

```bash
# Service control
sudo systemctl start nginx          # start
sudo systemctl stop nginx           # stop
sudo systemctl restart nginx        # stop + start
sudo systemctl reload nginx         # reload config without restart
sudo systemctl status nginx         # detailed status

# Boot behaviour
sudo systemctl enable nginx         # start automatically on boot
sudo systemctl disable nginx        # don't start on boot
sudo systemctl enable --now nginx   # enable AND start immediately
sudo systemctl is-enabled nginx     # check if enabled
sudo systemctl is-active nginx      # check if running

# List services
systemctl list-units --type=service             # all active services
systemctl list-units --type=service --all       # including inactive
systemctl list-units --type=service --state=failed  # failed services

# System control
sudo systemctl reboot
sudo systemctl poweroff
sudo systemctl halt
```

---

## systemctl status — Reading the Output

```bash
sudo systemctl status nginx
# ● nginx.service - A high performance web server
#    Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
#    Active: active (running) since Mon 2024-01-15 10:30:00 UTC; 2h ago
#      Docs: man:nginx(8)
#  Main PID: 1234 (nginx)
#    CGroup: /system.slice/nginx.service
#            ├─1234 nginx: master process /usr/sbin/nginx
#            ├─1235 nginx: worker process
#            └─1236 nginx: worker process
#
# Jan 15 10:30:00 server nginx[1234]: nginx: configuration file ... ok
# Jan 15 10:30:00 server systemd[1]: Started nginx.service

# Status meanings:
# active (running)   = process is up
# active (exited)    = ran successfully and exited (one-shot services)
# inactive (dead)    = not running
# failed             = exited with error
```

---

## Unit Files — Defining a Service

Unit files tell systemd how to manage a process. Location: `/etc/systemd/system/`.

### Minimal service unit file

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Python Web Application
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

### Full production unit file with all common options

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
Documentation=https://github.com/me/myapp
After=network.target postgresql.service
Requires=postgresql.service         # hard dependency — fail if postgres fails
Wants=redis.service                 # soft dependency — try but don't fail

[Service]
Type=simple                         # simple | forking | oneshot | notify
User=appuser
Group=appgroup
WorkingDirectory=/opt/myapp

# Environment
Environment=NODE_ENV=production
Environment=PORT=8080
EnvironmentFile=/etc/myapp/env      # load vars from file

# Commands
ExecStartPre=/opt/myapp/pre-start.sh   # run before main process
ExecStart=/usr/bin/node /opt/myapp/server.js
ExecStartPost=/opt/myapp/post-start.sh # run after start
ExecStop=/opt/myapp/graceful-stop.sh
ExecReload=/bin/kill -HUP $MAINPID    # reload config

# Restart behaviour
Restart=on-failure                  # always | on-failure | on-abnormal | no
RestartSec=5s                       # wait 5s before restart
StartLimitIntervalSec=60            # within 60 seconds
StartLimitBurst=3                   # don't restart more than 3 times

# Resource limits
LimitNOFILE=65536                   # open file descriptors
MemoryMax=512M                      # memory limit (systemd slice)

# Security hardening
NoNewPrivileges=true
PrivateTmp=true                     # isolated /tmp

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

[Install]
WantedBy=multi-user.target
```

### After creating or editing a unit file

```bash
sudo systemctl daemon-reload        # REQUIRED after any unit file change
sudo systemctl enable --now myapp
```

---

## journalctl — Reading Logs

```bash
journalctl                          # all logs (oldest first)
journalctl -r                       # newest first
journalctl -f                       # follow (like tail -f)
journalctl -n 50                    # last 50 lines
journalctl -u nginx                 # logs for nginx service only
journalctl -u nginx -f              # follow nginx logs
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx --since "2024-01-15" --until "2024-01-16"
journalctl -b                       # logs from current boot
journalctl -b -1                    # logs from previous boot
journalctl -p err                   # only error level and above
journalctl -p err -u nginx          # nginx errors only
journalctl --disk-usage             # how much space logs are using
sudo journalctl --vacuum-size=500M  # reduce log size to 500MB
```

---

## Timers — Systemd Alternative to Cron

```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Run backup daily

[Timer]
OnCalendar=daily                    # daily at midnight
OnCalendar=*-*-* 02:00:00          # 2am every day
OnCalendar=Mon *-*-* 03:00:00      # every Monday at 3am
Persistent=true                     # run missed jobs after reboot

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/backup.service
[Unit]
Description=Backup service

[Service]
Type=oneshot
ExecStart=/opt/scripts/backup.sh
```

```bash
sudo systemctl enable --now backup.timer
systemctl list-timers               # see all timers and next run time
```

---

## Service Types Explained

| Type | Use when |
|---|---|
| `simple` | Process stays in foreground (most apps) |
| `forking` | App forks into background (old-style daemons) |
| `oneshot` | Run once, then exit (scripts, migrations) |
| `notify` | App signals systemd when it's ready (advanced) |
| `idle` | Start after all other jobs finish |

---

## Debugging Failed Services

```bash
# Step 1: See why it failed
sudo systemctl status myapp
journalctl -u myapp -n 50 --no-pager

# Step 2: Check unit file syntax
sudo systemd-analyze verify myapp.service

# Step 3: Run manually to see output directly
sudo -u appuser /usr/bin/python3 /opt/myapp/app.py

# Step 4: Check dependencies
systemctl list-dependencies myapp.service

# Step 5: Check environment
systemctl show myapp.service | grep Environment
```
