# 11 — SSH and Remote Access

> 90% of DevOps work happens on remote servers via SSH. Master this and you'll feel at home anywhere.

---

## SSH Basics

```bash
ssh user@hostname                   # connect
ssh user@192.168.1.10               # connect by IP
ssh -p 2222 user@hostname           # custom port
ssh -i ~/.ssh/mykey.pem user@host   # specify key file
ssh -v user@hostname                # verbose (debug connection issues)
ssh -vvv user@hostname              # very verbose

# Run a command remotely without interactive session
ssh user@host "df -h"
ssh user@host "sudo systemctl restart nginx"
ssh user@host "cat /var/log/nginx/error.log | tail -50"
```

---

## SSH Key Authentication Setup

```bash
# Step 1: Generate key pair on YOUR machine
ssh-keygen -t ed25519 -C "your-comment"
# Keys saved: ~/.ssh/id_ed25519 (private) and ~/.ssh/id_ed25519.pub (public)

# Step 2: Copy public key to server
ssh-copy-id user@hostname                           # easiest way
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@hostname  # specify key

# Manual way (if ssh-copy-id isn't available)
cat ~/.ssh/id_ed25519.pub | ssh user@host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# Step 3: Test
ssh user@hostname                   # should log in without password prompt
```

---

## `~/.ssh/config` — Connection Shortcuts

```
# ~/.ssh/config
Host webserver
    HostName 192.168.1.10
    User ubuntu
    IdentityFile ~/.ssh/web_key.pem
    Port 22

Host bastion
    HostName bastion.example.com
    User ec2-user
    IdentityFile ~/.ssh/aws_key.pem

# Jump through bastion to reach internal server
Host internal-db
    HostName 10.0.1.50
    User ubuntu
    ProxyJump bastion

# Reuse connections (faster for multiple ssh commands)
Host *
    ControlMaster auto
    ControlPath ~/.ssh/cm-%r@%h:%p
    ControlPersist 10m
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

```bash
# After setting config
ssh webserver                       # uses all settings from config
ssh internal-db                     # automatically jumps through bastion
```

---

## Hardening SSH Server

```bash
sudo nano /etc/ssh/sshd_config
```

Key settings to change:
```
Port 2222                           # change from default 22
PermitRootLogin no                  # never allow root SSH login
PasswordAuthentication no           # key-only authentication
PubkeyAuthentication yes
MaxAuthTries 3                      # limit login attempts
ClientAliveInterval 300             # disconnect idle clients after 5 mins
ClientAliveCountMax 2
AllowUsers alice bob                # whitelist specific users
X11Forwarding no
```

```bash
# Validate config before restarting
sudo sshd -t

# Apply changes
sudo systemctl reload sshd
```

---

## Copying Files

```bash
# scp — simple file copy over SSH
scp file.txt user@host:/remote/path/
scp user@host:/remote/file.txt ./local/
scp -r folder/ user@host:/remote/path/     # recursive
scp -P 2222 file.txt user@host:/tmp/       # custom port

# rsync — smarter sync (only transfers changes)
rsync -avz folder/ user@host:/remote/      # sync local to remote
rsync -avz user@host:/remote/ folder/      # sync remote to local
rsync -avz --delete folder/ user@host:/remote/  # mirror (delete extra files)
rsync -avz -e "ssh -p 2222" folder/ user@host:/remote/  # custom port
rsync -n folder/ user@host:/remote/        # dry run (what would change)

# rsync flags: -a=archive, -v=verbose, -z=compress, --progress=show progress
```

---

## tmux — Terminal Multiplexer

tmux lets you run sessions that persist after you disconnect — critical for long-running operations.

```bash
# Sessions
tmux                                # start new session
tmux new -s deploy                  # named session
tmux ls                             # list sessions
tmux attach -t deploy               # reattach
tmux kill-session -t deploy         # kill session

# Inside tmux — prefix key is Ctrl+B
Ctrl+B d            # detach (session keeps running)
Ctrl+B c            # new window
Ctrl+B n            # next window
Ctrl+B p            # previous window
Ctrl+B ,            # rename window
Ctrl+B %            # split pane vertically
Ctrl+B "            # split pane horizontally
Ctrl+B arrow        # move between panes
Ctrl+B z            # zoom current pane (toggle fullscreen)
Ctrl+B [            # scroll mode (arrow keys to scroll, q to quit)
```

---

## SSH Tunneling

```bash
# Local port forwarding — access remote service locally
# Access remote MySQL (port 3306) via localhost:3307
ssh -L 3307:localhost:3306 user@remote-server -N

# Dynamic forwarding — SOCKS proxy
ssh -D 8080 user@remote-server -N
# Now configure browser to use SOCKS5 proxy at localhost:8080

# Remote port forwarding — expose local service to remote server
ssh -R 8080:localhost:3000 user@remote-server -N
```

---

## SCP vs rsync vs SFTP

| Tool | Best for |
|---|---|
| `scp` | Quick one-off file transfer |
| `rsync` | Syncing directories, incremental backups, large files |
| `sftp` | Interactive file browsing on remote server |

```bash
# sftp interactive
sftp user@hostname
sftp> ls
sftp> get remote-file.txt
sftp> put local-file.txt
sftp> quit
```
