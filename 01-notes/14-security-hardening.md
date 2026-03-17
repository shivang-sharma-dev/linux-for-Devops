# 14 — Security Hardening

> A freshly provisioned Linux server is not secure by default. These are the steps every DevOps engineer should apply to any server before it goes into production.

---

## The Hardening Mindset

```
Security is not a feature you add — it is a process you follow.

Principle of Least Privilege:
  Every user, process, and service should have the minimum
  permissions needed to do its job. Nothing more.

Attack Surface Reduction:
  Every open port, running service, and installed package
  is a potential entry point. Remove what you don't need.

Defence in Depth:
  No single control is enough. Layer multiple controls so
  that if one fails, others still protect you.
```

---

## 1. Keep the System Updated

```bash
# Ubuntu / Debian
sudo apt update && sudo apt upgrade -y

# Enable automatic security updates
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
cat /etc/apt/apt.conf.d/50unattended-upgrades  # review config

# RHEL / CentOS
sudo dnf update -y
sudo dnf install dnf-automatic
sudo systemctl enable --now dnf-automatic.timer
```

---

## 2. Secure SSH (Most Critical)

```bash
sudo nano /etc/ssh/sshd_config
```

```
# /etc/ssh/sshd_config — recommended settings

Port 2222                       # change from 22 (reduces automated scans)
Protocol 2                      # only SSHv2

# Authentication
PermitRootLogin no              # NEVER allow direct root login
PasswordAuthentication no       # keys only — no passwords
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PermitEmptyPasswords no

# Restrictions
MaxAuthTries 3                  # 3 failed attempts = disconnect
LoginGraceTime 20               # 20 seconds to authenticate
MaxSessions 10
ClientAliveInterval 300         # disconnect idle after 5 minutes
ClientAliveCountMax 2

# Disable unused features
X11Forwarding no
AllowTcpForwarding no           # disable unless you need SSH tunneling
AllowAgentForwarding no

# Whitelist specific users (most restrictive)
AllowUsers alice bob deploy

# Or whitelist a group
AllowGroups sshusers
```

```bash
# Test config before restarting
sudo sshd -t && echo "Config OK"

# Apply
sudo systemctl reload sshd

# Add users to sshusers group
sudo groupadd sshusers
sudo usermod -aG sshusers alice
```

---

## 3. Configure the Firewall

```bash
# UFW (Ubuntu)
sudo ufw default deny incoming      # block everything in by default
sudo ufw default allow outgoing     # allow everything out
sudo ufw allow 2222/tcp             # your new SSH port
sudo ufw allow 80/tcp               # HTTP
sudo ufw allow 443/tcp              # HTTPS
sudo ufw enable
sudo ufw status verbose

# Only allow SSH from specific IPs (best practice)
sudo ufw allow from 203.0.113.0/24 to any port 2222

# firewalld (RHEL)
sudo firewall-cmd --set-default-zone=drop
sudo firewall-cmd --add-service=ssh --permanent
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --add-service=https --permanent
sudo firewall-cmd --reload
```

---

## 4. Install and Configure fail2ban

fail2ban watches log files and temporarily bans IPs that show signs of brute-force.

```bash
sudo apt install fail2ban

# Create local config (never edit the .conf files directly)
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

```ini
# /etc/fail2ban/jail.local
[DEFAULT]
bantime  = 1h           # ban for 1 hour
findtime = 10m          # look at last 10 minutes
maxretry = 5            # 5 failures = ban
banaction = ufw         # use ufw to ban

[sshd]
enabled = true
port    = 2222          # your SSH port
logpath = %(sshd_log)s
maxretry = 3            # stricter for SSH
bantime = 24h
```

```bash
sudo systemctl enable --now fail2ban
sudo fail2ban-client status             # overall status
sudo fail2ban-client status sshd        # SSH jail status
sudo fail2ban-client set sshd unbanip 1.2.3.4   # unban an IP
```

---

## 5. User Account Hardening

```bash
# Lock unused system accounts
sudo passwd -l nobody
sudo usermod -s /sbin/nologin www-data  # no shell for service accounts

# Set password policies
sudo apt install libpam-pwquality
sudo nano /etc/security/pwquality.conf
```

```
# /etc/security/pwquality.conf
minlen = 12             # minimum password length
dcredit = -1            # require at least 1 digit
ucredit = -1            # require at least 1 uppercase
lcredit = -1            # require at least 1 lowercase
ocredit = -1            # require at least 1 special char
maxrepeat = 3           # no more than 3 repeated chars
```

```bash
# Password aging (force regular changes)
sudo chage -M 90 alice          # max 90 days
sudo chage -W 14 alice          # warn 14 days before expiry
sudo chage -l alice             # view policy for alice

# Account lockout after failed logins
sudo nano /etc/pam.d/common-auth
# Add: auth required pam_tally2.so onerr=fail deny=5 unlock_time=900
```

---

## 6. Minimise Attack Surface

```bash
# List all listening ports — close anything unnecessary
sudo ss -tulnp

# Disable unused services
sudo systemctl disable bluetooth
sudo systemctl disable avahi-daemon
sudo systemctl disable cups        # printing — rarely needed on servers
sudo systemctl stop avahi-daemon

# Remove unnecessary packages
sudo apt autoremove
sudo apt purge telnet rsh-client

# List installed packages (review periodically)
dpkg --list | grep "^ii" | awk '{print $2}'
```

---

## 7. File System Security

```bash
# Find SUID/SGID files (potential privilege escalation paths)
find / -perm /4000 -type f 2>/dev/null    # SUID files
find / -perm /2000 -type f 2>/dev/null    # SGID files

# Find world-writable files (should be rare)
find / -perm -o+w -type f 2>/dev/null | grep -v /proc

# Secure /tmp — mount with noexec,nosuid
# Add to /etc/fstab:
# tmpfs /tmp tmpfs defaults,noexec,nosuid,nodev 0 0

# Immutable files (can't be modified even by root until removed)
sudo chattr +i /etc/passwd          # make immutable
sudo chattr -i /etc/passwd          # remove immutable
lsattr /etc/passwd                  # check attribute
```

---

## 8. Audit and Logging

```bash
# auditd — comprehensive system auditing
sudo apt install auditd

# Watch a sensitive file
sudo auditctl -w /etc/passwd -p wa -k passwd-changes
sudo auditctl -w /etc/sudoers -p wa -k sudoers-changes

# Watch for privilege escalation
sudo auditctl -a always,exit -F arch=b64 -S execve -F euid=0 -k root-commands

# View audit logs
sudo ausearch -k passwd-changes
sudo aureport --summary
sudo ausearch -m USER_AUTH --start today

# Make rules persistent
sudo nano /etc/audit/rules.d/hardening.rules
```

```bash
# Log all sudo usage (already happens via auth.log)
grep sudo /var/log/auth.log

# Enable process accounting
sudo apt install acct
sudo accton on
sudo lastcomm root           # commands run by root
sudo sa                      # summary of commands
```

---

## 9. Kernel Hardening — sysctl

```bash
sudo nano /etc/sysctl.d/99-hardening.conf
```

```
# Network hardening
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.tcp_syncookies = 1             # SYN flood protection
net.ipv4.conf.all.rp_filter = 1        # IP spoofing protection
net.ipv6.conf.all.accept_redirects = 0

# Restrict dmesg to root
kernel.dmesg_restrict = 1

# Hide kernel pointers
kernel.kptr_restrict = 2

# Prevent core dumps from SUID programs
fs.suid_dumpable = 0
```

```bash
sudo sysctl -p /etc/sysctl.d/99-hardening.conf
```

---

## 10. Security Scanning Tools

```bash
# Lynis — comprehensive security audit
sudo apt install lynis
sudo lynis audit system             # full audit report
sudo lynis audit system --quick     # faster scan

# chkrootkit — check for rootkits
sudo apt install chkrootkit
sudo chkrootkit

# rkhunter — rootkit hunter
sudo apt install rkhunter
sudo rkhunter --update
sudo rkhunter --check

# ClamAV — antivirus (useful for shared hosting)
sudo apt install clamav
sudo freshclam               # update definitions
sudo clamscan -r /home/      # scan directory
```

---

## Hardening Checklist (Quick Reference)

```
[ ] System updated and auto-updates enabled
[ ] SSH: root login disabled
[ ] SSH: password auth disabled, keys only
[ ] SSH: non-default port
[ ] SSH: AllowUsers or AllowGroups set
[ ] Firewall enabled, default deny incoming
[ ] fail2ban installed and configured
[ ] Unused services disabled
[ ] Unused packages removed
[ ] SUID/SGID files reviewed
[ ] Password policy enforced
[ ] auditd installed and logging sensitive files
[ ] Kernel hardening sysctl applied
[ ] Run lynis and address high-priority findings
```
