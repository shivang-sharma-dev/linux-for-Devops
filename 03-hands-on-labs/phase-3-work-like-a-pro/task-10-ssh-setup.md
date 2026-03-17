# Task 10 — SSH & Remote Access

> **Prerequisite:** Read `01-notes/11-ssh-and-remote.md`
> **Environment:** Full Linux system. Ideally two machines (or a VM/cloud instance) to practice remote connections. A single machine works for key generation and local SSH.
> **Difficulty:** Intermediate

---

## Objective

By the end of this lab you will be able to:
- Generate SSH key pairs
- Configure SSH for passwordless authentication
- Set up `~/.ssh/config` for named connections
- Transfer files with SCP and rsync
- Use SSH tunnels
- Apply basic SSH security hardening

---

## Part 1 — SSH Key Generation

### Exercise 1.1: Generate a key pair

```bash
# Generate an Ed25519 key (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# When prompted:
# - File location: press Enter for default (~/.ssh/id_ed25519)
# - Passphrase: enter a passphrase (recommended) or press Enter for none

# Alternative: RSA 4096-bit (wider compatibility)
# ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### Exercise 1.2: Examine the keys

```bash
# List your SSH keys
ls -la ~/.ssh/

# View public key (this is what you share)
cat ~/.ssh/id_ed25519.pub

# View key fingerprint
ssh-keygen -lf ~/.ssh/id_ed25519.pub

# NEVER share your private key
# cat ~/.ssh/id_ed25519    # This stays on YOUR machine only
```

**Key files:**
| File | What It Is | Share? |
|------|-----------|--------|
| `id_ed25519` | Private key | NEVER |
| `id_ed25519.pub` | Public key | Yes — give to servers/GitHub |
| `authorized_keys` | Public keys allowed to log in | On the server |
| `known_hosts` | Fingerprints of servers you've connected to | Auto-managed |
| `config` | SSH client configuration | Your machine |

### Exercise 1.3: Set correct permissions

```bash
# SSH is strict about permissions — wrong permissions = connection refused
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519           # Private key: owner read/write only
chmod 644 ~/.ssh/id_ed25519.pub       # Public key: readable
chmod 600 ~/.ssh/authorized_keys 2>/dev/null  # If it exists
chmod 644 ~/.ssh/config 2>/dev/null           # If it exists

# Verify
ls -la ~/.ssh/
```

---

## Part 2 — SSH to Localhost (Practice Without a Remote Server)

### Exercise 2.1: SSH into your own machine

```bash
# Make sure SSH server is running
sudo systemctl status ssh
# If not installed:
# sudo apt install openssh-server -y
# sudo systemctl enable --now ssh

# SSH into localhost
ssh localhost
# Type "yes" to accept the fingerprint, then enter your password
# You're now in a new SSH session on your own machine

whoami
hostname
exit        # Exit the SSH session
```

### Exercise 2.2: Set up passwordless login to localhost

```bash
# Copy your public key to authorized_keys
ssh-copy-id localhost
# Enter your password one last time

# Now SSH without a password
ssh localhost
whoami
exit

# Verify — it should NOT ask for a password
ssh localhost echo "Passwordless SSH works!"
```

### Exercise 2.3: Manually set up authorized_keys (understanding the process)

```bash
# This is what ssh-copy-id does under the hood:
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

For a remote server, you would do:
```bash
# ssh-copy-id user@remote-server
# OR manually:
# cat ~/.ssh/id_ed25519.pub | ssh user@remote-server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

---

## Part 3 — SSH Config File

### Exercise 3.1: Create an SSH config

```bash
cat > ~/.ssh/config << 'EOF'
# Default settings for all hosts
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    AddKeysToAgent yes

# Localhost alias
Host local
    HostName 127.0.0.1
    User $USER
    IdentityFile ~/.ssh/id_ed25519

# Example: Cloud VM
# Host myserver
#     HostName 203.0.113.50
#     User ubuntu
#     Port 22
#     IdentityFile ~/.ssh/id_ed25519

# Example: GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
EOF

# Fix: replace $USER with actual username
sed -i "s/\$USER/$(whoami)/" ~/.ssh/config
chmod 644 ~/.ssh/config

# Now you can use the alias
ssh local echo "Connected via alias!"
```

**Benefits of SSH config:**
- `ssh myserver` instead of `ssh -i ~/.ssh/key -p 2222 user@203.0.113.50`
- Sets per-host options (key, port, user)
- Keeps alive connections

---

## Part 4 — File Transfer

### Exercise 4.1: SCP (Secure Copy)

```bash
mkdir -p ~/lab-10

# Create test files
echo "Transfer test" > ~/lab-10/test.txt
mkdir -p ~/lab-10/data
echo "Data file 1" > ~/lab-10/data/file1.txt
echo "Data file 2" > ~/lab-10/data/file2.txt

# Copy a file to remote (using localhost as practice)
scp ~/lab-10/test.txt localhost:/tmp/test-received.txt

# Verify
ssh localhost cat /tmp/test-received.txt

# Copy a directory recursively
scp -r ~/lab-10/data localhost:/tmp/data-received

# Copy FROM remote to local
scp localhost:/etc/hostname ~/lab-10/remote-hostname.txt
cat ~/lab-10/remote-hostname.txt
```

### Exercise 4.2: rsync (better than SCP for most cases)

```bash
# Install if not available
sudo apt install -y rsync

# Sync a directory
rsync -avz ~/lab-10/data/ localhost:/tmp/rsync-test/
# -a = archive (preserves permissions, timestamps)
# -v = verbose
# -z = compress during transfer

# Verify
ssh localhost ls -la /tmp/rsync-test/

# Sync again (only transfers changes — much faster)
echo "new file" > ~/lab-10/data/file3.txt
rsync -avz ~/lab-10/data/ localhost:/tmp/rsync-test/

# Dry run — see what WOULD be transferred without doing it
rsync -avzn ~/lab-10/data/ localhost:/tmp/rsync-test/
```

**rsync vs scp:**
| Feature | scp | rsync |
|---------|-----|-------|
| Transfers everything | Yes | Only changes (delta) |
| Resume interrupted transfer | No | Yes |
| Compression | No | Yes (`-z`) |
| Preserve permissions | Basic | Full (`-a`) |
| Exclude files | No | Yes (`--exclude`) |

---

## Part 5 — SSH Tunnels

### Exercise 5.1: Local port forwarding

"I want to access a service on the remote server through my local machine"

```bash
# Start a simple HTTP server on localhost:8000
python3 -m http.server 8000 &
SERVER_PID=$!

# Create a tunnel: local port 9000 → localhost:8000 via SSH
ssh -L 9000:localhost:8000 -N localhost &
TUNNEL_PID=$!

# Now port 9000 on your machine tunnels to port 8000
curl http://localhost:9000

# Clean up
kill $SERVER_PID $TUNNEL_PID 2>/dev/null
```

**Flag breakdown:**
- `-L 9000:localhost:8000` — Map local port 9000 to remote's localhost:8000
- `-N` — Don't open a shell, just the tunnel

### Exercise 5.2: Run commands remotely

```bash
# Execute a single command
ssh localhost "df -h / && free -h"

# Run a script remotely
ssh localhost 'bash -s' << 'REMOTE'
echo "Running on: $(hostname)"
echo "Uptime: $(uptime -p)"
echo "Disk: $(df -h / | awk 'NR==2{print $5}') used"
REMOTE
```

---

## Part 6 — SSH Security Hardening

### Exercise 6.1: Review SSH server config

```bash
# View current SSH server configuration
sudo cat /etc/ssh/sshd_config | grep -v "^#" | grep -v "^$"

# Key settings to know:
grep -i "PermitRootLogin" /etc/ssh/sshd_config
grep -i "PasswordAuthentication" /etc/ssh/sshd_config
grep -i "Port " /etc/ssh/sshd_config
```

### Exercise 6.2: Recommended hardening (understand, don't apply blindly)

These are changes you would make on a production server:

```bash
# 1. Disable root login
# PermitRootLogin no

# 2. Disable password auth (key-only)
# PasswordAuthentication no

# 3. Change default port (security through obscurity, minor benefit)
# Port 2222

# 4. Limit SSH to specific users
# AllowUsers deploy admin

# 5. Set login timeout
# LoginGraceTime 30

# After changes:
# sudo sshd -t              # Test config for errors
# sudo systemctl restart ssh # Apply changes
```

**Warning:** If you disable password auth before setting up key-based auth, you'll be locked out. Always test key auth works first.

---

## Part 7 — SSH Agent

### Exercise 7.1: Use ssh-agent

```bash
# Start the agent
eval "$(ssh-agent -s)"

# Add your key
ssh-add ~/.ssh/id_ed25519

# List loaded keys
ssh-add -l

# Now SSH won't ask for your passphrase again (this session)
ssh localhost echo "Agent works!"
```

---

## Challenges

1. **Set up key-based SSH to a cloud VM** (use AWS/GCP/Oracle free tier) and configure `~/.ssh/config` so you can connect with a simple alias

2. **Create a deployment script** that:
   - SSHes into a remote server (or localhost)
   - Copies files with rsync
   - Runs a command on the remote server
   - Reports success/failure

3. **Test SSH security:**
   - Check if root login is enabled
   - Check if password authentication is enabled
   - List all currently active SSH sessions: `who` or `ss | grep ssh`

4. **Set up SSH for GitHub:**
   ```bash
   # Test GitHub SSH connection
   ssh -T git@github.com
   ```

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Sharing your private key | Anyone with it can log in as you | Never share `id_ed25519` — only share `.pub` |
| Wrong permissions on `~/.ssh/` | SSH refuses to work with loose permissions | `chmod 700 ~/.ssh`, `chmod 600` for private keys |
| Disabling password auth before testing keys | Locked out of server | Test key auth first, then disable passwords |
| Not using a passphrase on keys | Stolen laptop = compromised servers | Always set a passphrase, use ssh-agent for convenience |
| Using `scp` for large/repeated transfers | Slow, transfers everything every time | Use `rsync` instead |

---

## Cleanup

```bash
rm -rf ~/lab-10
ssh localhost rm -rf /tmp/test-received.txt /tmp/data-received /tmp/rsync-test 2>/dev/null
```

---

## What's Next?

Proceed to `task-11-cron-automation.md` → Schedule and verify cron jobs.
