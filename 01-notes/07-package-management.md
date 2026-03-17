# 07 — Package Management

> Installing, updating, and managing software is the first thing you do on any new server.

---

## apt — Debian / Ubuntu

```bash
# Update package index (always do this before installing)
sudo apt update

# Upgrade all installed packages
sudo apt upgrade -y
sudo apt full-upgrade -y        # also handles dependency changes

# Install
sudo apt install nginx
sudo apt install nginx curl git  # multiple packages
sudo apt install -y nginx        # non-interactive

# Remove
sudo apt remove nginx            # remove package, keep config files
sudo apt purge nginx             # remove package AND config files
sudo apt autoremove              # remove unused dependencies

# Search
apt search nginx
apt-cache search nginx

# Info
apt show nginx                   # package details
dpkg -l | grep nginx             # is it installed?
dpkg -L nginx                    # list files installed by package

# History and logs
cat /var/log/apt/history.log
```

---

## dnf / yum — RHEL / CentOS / Amazon Linux

```bash
sudo dnf update -y
sudo dnf install nginx
sudo dnf remove nginx
sudo dnf search nginx
dnf info nginx
rpm -qa | grep nginx             # is it installed?
rpm -ql nginx                    # list installed files
```

---

## Adding Third-Party Repositories

```bash
# Example: Docker on Ubuntu
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
echo "deb [signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt install docker-ce

# Example: Node.js via NodeSource
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt install nodejs
```

---

## snap — Universal Packages

```bash
sudo snap install code --classic
sudo snap install kubectl --classic
snap list
snap remove code
snap refresh                      # update all snaps
```

---

## Installing from Source (when no package exists)

```bash
# Generic pattern
wget https://example.com/app-1.0.tar.gz
tar -xzf app-1.0.tar.gz
cd app-1.0/
./configure --prefix=/usr/local
make
sudo make install
```

---

## Pinning Package Versions

```bash
# Hold a package at current version (prevent auto-upgrade)
sudo apt-mark hold nginx
sudo apt-mark unhold nginx
apt-mark showhold                 # see held packages

# Install a specific version
sudo apt install nginx=1.18.0-0ubuntu1
```
