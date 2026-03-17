# Task 13 — Security Hardening

> **Prerequisite:** Read `01-notes/14-security-hardening.md`
> **Environment:** Full Linux system (VM strongly recommended — you'll be changing security configurations). You need `sudo` access.
> **Difficulty:** Advanced

---

## Objective

By the end of this lab you will be able to:
- Audit a fresh Linux server for security issues
- Harden SSH configuration
- Set up a firewall with `ufw` or `iptables`
- Configure `fail2ban` for brute-force protection
- Manage users and sudo access securely
- Apply basic kernel-level hardening

---

## Part 1 — Security Audit

### Exercise 1.1: Assess your current security posture

```bash
# What OS and kernel version?
cat /etc/os-release
uname -r

# Check for available security updates
sudo apt update
apt list --upgradable 2>/dev/null | head -20

# What users exist?
cat /etc/passwd | grep -v nologin | grep -v false

# Who has sudo access?
grep -E '^sudo:|^wheel:' /etc/group

# What's listening on the network?
sudo ss -tulnp

# Any failed login attempts?
sudo grep "Failed password" /var/log/auth.log 2>/dev/null | wc -l

# Check for world-writable files in sensitive areas
sudo find /etc -perm -o+w -type f 2>/dev/null

# Check for SUID binaries (programs that run as root)
sudo find / -perm -4000 -type f 2>/dev/null | head -20
```

---

## Part 2 — SSH Hardening

### Exercise 2.1: Audit current SSH config

```bash
# Check current SSH settings
sudo sshd -T | grep -E "permitrootlogin|passwordauthentication|port |maxauthtries|logingracetime|x11forwarding"
```

### Exercise 2.2: Harden SSH configuration

**Warning:** Make sure you have key-based SSH working before disabling password auth, or you'll be locked out.

```bash
# Backup the original config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Create a hardening drop-in file
sudo tee /etc/ssh/sshd_config.d/hardening.conf << 'EOF'
# Disable root login
PermitRootLogin no

# Disable password auth (key-only) — ONLY if you have keys set up
# PasswordAuthentication no

# Limit authentication attempts
MaxAuthTries 3

# Reduce login timeout
LoginGraceTime 20

# Disable X11 forwarding (unless needed)
X11Forwarding no

# Disable empty passwords
PermitEmptyPasswords no

# Use only SSH protocol 2
Protocol 2

# Idle timeout (disconnect after 5 min idle)
ClientAliveInterval 300
ClientAliveCountMax 0
EOF

# Test config before applying
sudo sshd -t
echo "Config test passed: $?"

# Apply changes
sudo systemctl restart ssh

# Verify
sudo sshd -T | grep -E "permitrootlogin|maxauthtries|logingracetime"
```

### Exercise 2.3: Restrict SSH to specific users (optional)

```bash
# Only allow specific users to SSH in
# Add to /etc/ssh/sshd_config.d/hardening.conf:
# AllowUsers yourusername admin

# Or restrict by group
# AllowGroups sshusers
```

---

## Part 3 — Firewall with UFW

### Exercise 3.1: Set up UFW (Uncomplicated Firewall)

```bash
# Install UFW
sudo apt install -y ufw

# Check current status
sudo ufw status verbose

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (CRITICAL — do this before enabling!)
sudo ufw allow ssh
# or by port:
sudo ufw allow 22/tcp

# Enable the firewall
sudo ufw enable

# Verify
sudo ufw status verbose
```

### Exercise 3.2: Manage firewall rules

```bash
# Allow specific services
sudo ufw allow http        # Port 80
sudo ufw allow https       # Port 443

# Allow a specific port
sudo ufw allow 8080/tcp

# Allow from a specific IP
sudo ufw allow from 192.168.1.100

# Allow from a subnet to a specific port
sudo ufw allow from 192.168.1.0/24 to any port 22

# Deny a specific IP
sudo ufw deny from 10.0.0.5

# List rules with numbers
sudo ufw status numbered

# Delete a rule by number
sudo ufw delete <rule_number>

# Delete a rule by specification
sudo ufw delete allow 8080/tcp
```

### Exercise 3.3: UFW logging

```bash
# Enable logging
sudo ufw logging on

# Check firewall logs
sudo grep UFW /var/log/syslog | tail -10
```

---

## Part 4 — Fail2Ban (Brute-Force Protection)

### Exercise 4.1: Install and configure fail2ban

```bash
# Install
sudo apt install -y fail2ban

# Check status
sudo systemctl status fail2ban

# View default configuration
cat /etc/fail2ban/jail.conf | head -50
```

### Exercise 4.2: Create a custom jail for SSH

```bash
# Never edit jail.conf directly — create a local override
sudo tee /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
# Ban for 1 hour
bantime = 3600

# Time window for counting failures
findtime = 600

# Number of failures before ban
maxretry = 5

# Send alert emails (uncomment and configure if you have mail set up)
# destemail = your@email.com
# sendername = Fail2Ban
# action = %(action_mwl)s

[sshd]
enabled = true
port = ssh
logpath = %(sshd_log)s
maxretry = 3
bantime = 7200
EOF

# Restart fail2ban
sudo systemctl restart fail2ban

# Check jail status
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

### Exercise 4.3: Manage bans

```bash
# View banned IPs
sudo fail2ban-client status sshd

# Manually ban an IP
sudo fail2ban-client set sshd banip 10.0.0.99

# Unban an IP
sudo fail2ban-client set sshd unbanip 10.0.0.99

# Check fail2ban log
sudo tail -20 /var/log/fail2ban.log
```

---

## Part 5 — User Security

### Exercise 5.1: Password policies

```bash
# Check password aging for a user
sudo chage -l $(whoami)

# Set password aging policies
# sudo chage -M 90 -m 7 -W 14 username
# -M 90  = password expires after 90 days
# -m 7   = minimum 7 days between changes
# -W 14  = warn 14 days before expiry

# Check for users with no password
sudo awk -F: '($2 == "" || $2 == "!") {print $1}' /etc/shadow
```

### Exercise 5.2: Sudo security

```bash
# Check current sudo configuration
sudo cat /etc/sudoers

# Best practice: use sudoers.d directory
# sudo visudo -f /etc/sudoers.d/custom

# Limit a user to specific commands only
# username ALL=(ALL) /usr/bin/systemctl restart nginx, /usr/bin/journalctl

# Log all sudo commands
sudo grep -r "Defaults.*log" /etc/sudoers /etc/sudoers.d/ 2>/dev/null
```

### Exercise 5.3: Find and disable unnecessary accounts

```bash
# List all users with login shells
grep -v -E "nologin|false" /etc/passwd

# Lock a user account
# sudo usermod -L username

# Expire an account
# sudo usermod --expiredate 1970-01-01 username

# Check for users with UID 0 (root-level)
awk -F: '($3 == 0) {print $1}' /etc/passwd
# Only 'root' should appear
```

---

## Part 6 — Kernel Hardening

### Exercise 6.1: Apply sysctl hardening

```bash
# Create a security-focused sysctl config
sudo tee /etc/sysctl.d/99-security.conf << 'EOF'
# Disable IP forwarding (unless this is a router)
net.ipv4.ip_forward = 0

# Disable ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Disable source routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0

# Enable SYN flood protection
net.ipv4.tcp_syncookies = 1

# Log suspicious packets
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# Ignore ICMP broadcast requests
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Protect against SUID core dumps
fs.suid_dumpable = 0

# Randomize memory addresses (ASLR)
kernel.randomize_va_space = 2
EOF

# Apply
sudo sysctl --system

# Verify a few
sysctl net.ipv4.tcp_syncookies
sysctl kernel.randomize_va_space
```

---

## Part 7 — File Integrity

### Exercise 7.1: Monitor critical files

```bash
# Record checksums of critical files
mkdir -p ~/lab-13

sha256sum /etc/passwd /etc/shadow /etc/group /etc/sudoers /etc/ssh/sshd_config > ~/lab-13/baseline.sha256

cat ~/lab-13/baseline.sha256

# Later, verify nothing has changed
sha256sum -c ~/lab-13/baseline.sha256
```

### Exercise 7.2: Find recently modified system files

```bash
# Files in /etc modified in the last 24 hours
sudo find /etc -type f -mtime -1 | head -20

# Files in /usr modified in the last 24 hours (should be rare)
sudo find /usr -type f -mtime -1 | head -20
```

---

## Part 8 — Security Audit Script

```bash
cat > ~/lab-13/security-audit.sh << 'SCRIPT'
#!/bin/bash

echo "============================================"
echo "        SECURITY AUDIT REPORT"
echo "  $(hostname) — $(date)"
echo "============================================"

echo ""
echo "--- System Updates ---"
updates=$(apt list --upgradable 2>/dev/null | grep -c upgradable)
if [ "$updates" -gt 0 ]; then
    echo "[WARN] $updates packages can be upgraded"
else
    echo "[OK]   System is up to date"
fi

echo ""
echo "--- SSH Configuration ---"
root_login=$(sudo sshd -T 2>/dev/null | grep "permitrootlogin " | awk '{print $2}')
echo "Root login: $root_login"
[ "$root_login" = "no" ] && echo "[OK]   Root login disabled" || echo "[WARN] Root login is enabled"

echo ""
echo "--- Firewall ---"
if sudo ufw status | grep -q "Status: active"; then
    echo "[OK]   UFW is active"
    sudo ufw status | grep -E "ALLOW|DENY" | head -10
else
    echo "[WARN] Firewall is NOT active"
fi

echo ""
echo "--- Fail2Ban ---"
if systemctl is-active fail2ban > /dev/null 2>&1; then
    echo "[OK]   Fail2Ban is running"
    banned=$(sudo fail2ban-client status sshd 2>/dev/null | grep "Currently banned" | awk '{print $NF}')
    echo "  Currently banned IPs: ${banned:-0}"
else
    echo "[WARN] Fail2Ban is NOT running"
fi

echo ""
echo "--- Open Ports ---"
sudo ss -tulnp | grep LISTEN | awk '{print "  " $1, $5}' | sort -u

echo ""
echo "--- Users with Login Shells ---"
grep -v -E "nologin|false|sync|shutdown|halt" /etc/passwd | awk -F: '{print "  " $1 " (UID:" $3 ")"}'

echo ""
echo "--- Failed Login Attempts (last 24h) ---"
if [ -f /var/log/auth.log ]; then
    fails=$(sudo grep "Failed password" /var/log/auth.log 2>/dev/null | grep "$(date +%b\ %d)" | wc -l)
    echo "  Failed attempts today: $fails"
else
    echo "  No auth.log found"
fi

echo ""
echo "--- Root-Level Users (UID 0) ---"
roots=$(awk -F: '($3 == 0) {print $1}' /etc/passwd)
echo "  $roots"
[ "$(echo "$roots" | wc -w)" -eq 1 ] && echo "[OK]   Only root has UID 0" || echo "[WARN] Multiple users with UID 0!"

echo ""
echo "============================================"
echo "        END OF AUDIT"
echo "============================================"
SCRIPT

chmod +x ~/lab-13/security-audit.sh
sudo ~/lab-13/security-audit.sh
```

---

## Challenges

1. **Full server hardening exercise:**
   - Start with a fresh VM
   - Apply every hardening step from this lab
   - Run the security audit script
   - Document what you changed and why

2. **Set up fail2ban for another service** (e.g., nginx or Apache) with custom filter rules

3. **Create a monitoring script** that checks if any new SUID binaries appeared since the last check

4. **Implement port knocking** — where SSH port is closed by default and opens only after knocking on a sequence of ports

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Enabling firewall before allowing SSH | Locks yourself out | Always `ufw allow ssh` before `ufw enable` |
| Disabling password auth without testing keys | Locked out permanently | Test key auth in a separate session first |
| Setting `MaxAuthTries 1` | Legitimate users get locked out on typos | Use 3-5 as a reasonable limit |
| Hardening a production server without testing | Could break services | Test on a staging VM first |
| Not keeping the system updated | Known vulnerabilities remain exploitable | `sudo apt update && sudo apt upgrade` regularly |

---

## Cleanup

```bash
# Revert SSH changes
sudo rm -f /etc/ssh/sshd_config.d/hardening.conf
sudo systemctl restart ssh

# Remove sysctl hardening (if you want to revert)
sudo rm -f /etc/sysctl.d/99-security.conf
sudo sysctl --system

# Remove fail2ban custom config
sudo rm -f /etc/fail2ban/jail.local
sudo systemctl restart fail2ban

rm -rf ~/lab-13
```

---

## What's Next?

Proceed to `task-14-containers-basics.md` → Linux namespaces, cgroups, and container fundamentals.
