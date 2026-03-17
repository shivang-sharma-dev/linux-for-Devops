# 12 — Cron and Automation

> Scheduled tasks keep systems healthy automatically — backups, log cleanup, health checks, report generation. Never do manually what a machine can do on a schedule.

---

## Cron Syntax

```
*  *  *  *  *  command
│  │  │  │  └── Day of week (0-7, 0 and 7 = Sunday)
│  │  │  └───── Month (1-12)
│  │  └──────── Day of month (1-31)
│  └─────────── Hour (0-23)
└────────────── Minute (0-59)
```

### Examples

```
0 * * * *          Every hour at minute 0
0 2 * * *          Every day at 2:00 AM
0 2 * * 0          Every Sunday at 2:00 AM
0 2 1 * *          First day of every month at 2:00 AM
*/5 * * * *        Every 5 minutes
0,30 * * * *       Every 30 minutes (at :00 and :30)
0 9-17 * * 1-5     Every hour 9am-5pm, weekdays only
@reboot            Once at system startup
@daily             Once a day (same as 0 0 * * *)
@weekly            Once a week
@monthly           Once a month
```

---

## Managing Crontabs

```bash
crontab -e                          # edit your crontab
crontab -l                          # list your crontab
crontab -r                          # remove your crontab (careful!)
sudo crontab -e -u alice            # edit alice's crontab
sudo crontab -l -u alice            # list alice's crontab

# System crontab (root only)
sudo nano /etc/crontab              # system crontab (has user column)
ls /etc/cron.d/                     # per-app cron files
ls /etc/cron.daily/                 # scripts run daily (no cron syntax needed)
ls /etc/cron.hourly/
ls /etc/cron.weekly/
ls /etc/cron.monthly/
```

### `/etc/crontab` format (has extra user field)

```
0 2 * * *  root  /opt/scripts/backup.sh
*/5 * * * * www-data  /opt/scripts/healthcheck.sh
```

### `/etc/cron.d/` format (same as system crontab)

```bash
# /etc/cron.d/myapp
0 3 * * *  appuser  /opt/myapp/cleanup.sh >> /var/log/myapp-cleanup.log 2>&1
```

---

## Best Practices for Cron Jobs

```bash
# 1. Always use full paths
0 2 * * * /usr/bin/python3 /opt/scripts/backup.py

# 2. Redirect output to log file
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# 3. Use flock to prevent overlapping runs
0 * * * * /usr/bin/flock -n /tmp/myscript.lock /opt/scripts/job.sh

# 4. Use a wrapper with logging
0 2 * * * /opt/scripts/run-with-logging.sh backup >> /var/log/cron-jobs.log 2>&1

# 5. Test your cron syntax
# Use crontab.guru (website) to verify complex expressions
```

---

## Viewing Cron Logs

```bash
grep CRON /var/log/syslog           # Ubuntu cron logs
grep CRON /var/log/cron             # RHEL cron logs
journalctl -u cron                  # systemd journal for cron
journalctl -u cron --since "1 day ago"
```

---

## at — Run Once at a Specific Time

```bash
# Schedule a one-off job
at 2:00 AM
at> /opt/scripts/maintenance.sh
at> Ctrl+D                          # save

at 10:00 AM tomorrow
at now + 2 hours
at 9:00 AM July 20

atq                                 # list scheduled jobs
at -l                               # same as atq
atrm 3                              # remove job number 3
```

---

## systemd Timers (Modern Alternative)

See Note 09 for full detail. Timers are more reliable than cron for production use:
- Logs go to journalctl (not just syslog)
- Can handle missed runs (Persistent=true)
- Dependency management

```bash
systemctl list-timers               # all timers and next run time
systemctl list-timers --all         # including inactive
```

---

## Common Automation Tasks

```bash
# Daily backup at 2am
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# Clean /tmp files older than 7 days, every Sunday midnight
0 0 * * 0 find /tmp -mtime +7 -delete

# Rotate application logs daily at midnight
0 0 * * * /usr/sbin/logrotate /etc/logrotate.d/myapp

# Health check every 5 minutes
*/5 * * * * /opt/scripts/healthcheck.sh

# Renew SSL certificates monthly
0 3 1 * * certbot renew --quiet

# Database backup every night
0 1 * * * pg_dump mydb | gzip > /backups/db-$(date +\%Y\%m\%d).sql.gz
# Note: escape % as \% in crontab
```
