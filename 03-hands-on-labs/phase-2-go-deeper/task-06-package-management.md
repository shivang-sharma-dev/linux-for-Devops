# Task 06 — Package Management

> **Prerequisite:** Read `01-notes/07-package-management.md`
> **Environment:** Full Linux system (native, WSL2, or VM). You need `sudo` access. Exercises use `apt` (Debian/Ubuntu). RHEL/Fedora equivalents noted where applicable.
> **Difficulty:** Beginner

---

## Objective

By the end of this lab you will be able to:
- Update your system packages
- Search, install, and remove packages
- Understand package dependencies
- Pin package versions
- Add third-party repositories
- Clean up unused packages

---

## Part 1 — Update Your System

### Exercise 1.1: Update package lists and upgrade

```bash
# Update the package index (fetches latest info from repos)
sudo apt update

# See what can be upgraded
apt list --upgradable

# Upgrade all packages
sudo apt upgrade -y

# Full upgrade (handles dependency changes)
sudo apt full-upgrade -y
```

**Fedora/RHEL equivalent:**
```bash
sudo dnf check-update
sudo dnf upgrade -y
```

---

## Part 2 — Search and Install Packages

### Exercise 2.1: Search for packages

```bash
# Search by name
apt search htop

# Search with description
apt search "text editor"

# Show detailed info about a package
apt show htop

# Check if a package is installed
dpkg -l | grep htop
# or
apt list --installed | grep htop
```

### Exercise 2.2: Install packages

```bash
# Install a single package
sudo apt install -y tree

# Install multiple packages at once
sudo apt install -y htop ncdu tldr

# Verify installation
which tree
tree --version
htop --version
```

### Exercise 2.3: What files does a package provide?

```bash
# List files installed by a package
dpkg -L tree

# Find which package owns a file
dpkg -S /usr/bin/tree
# or
apt-file search /usr/bin/tree   # needs: sudo apt install apt-file && sudo apt-file update
```

---

## Part 3 — Remove Packages

### Exercise 3.1: Different ways to remove

```bash
# Remove a package (keeps config files)
sudo apt remove tree

# Remove package AND its config files
sudo apt purge tree

# Remove unused dependencies
sudo apt autoremove -y

# Verify it's gone
which tree    # Should return nothing
```

---

## Part 4 — Understanding Dependencies

### Exercise 4.1: Check dependencies

```bash
# Show what a package depends on
apt depends htop

# Show what depends on a package (reverse dependencies)
apt rdepends curl

# Simulate an install to see what would be installed
apt install --simulate nginx
```

### Exercise 4.2: Fix broken dependencies

```bash
# If you ever get broken package errors
sudo apt --fix-broken install

# Reconfigure packages
sudo dpkg --configure -a
```

---

## Part 5 — Pin Package Versions

### Exercise 5.1: Hold a package version

```bash
# Check current version
apt show curl | grep Version

# Prevent a package from being upgraded
sudo apt-mark hold curl

# List held packages
apt-mark showhold

# Remove the hold
sudo apt-mark unhold curl
```

### Exercise 5.2: Install a specific version

```bash
# List available versions
apt list -a curl

# Install a specific version (if available)
# sudo apt install curl=7.81.0-1ubuntu1.x
# (version number depends on your system)
```

---

## Part 6 — Add Third-Party Repositories (PPA)

### Exercise 6.1: Add a PPA (Ubuntu)

```bash
# Common example: adding a PPA
# (This is a demonstration — only add PPAs you trust)
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update

# List configured repositories
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/

# Remove a PPA
sudo add-apt-repository --remove ppa:deadsnakes/ppa -y
sudo apt update
```

### Exercise 6.2: Add a repo manually (example: Docker)

This is how real-world third-party repos are added:

```bash
# 1. Add the GPG key
# 2. Add the repository
# 3. Update and install

# Example structure (don't run unless you want Docker):
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
# echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
# sudo apt update
# sudo apt install docker-ce
```

---

## Part 7 — Cleanup and Maintenance

### Exercise 7.1: Clean up disk space

```bash
# Remove cached .deb files
sudo apt clean

# Remove only outdated cached files
sudo apt autoclean

# Remove unused dependencies
sudo apt autoremove -y

# Check how much space apt cache uses
du -sh /var/cache/apt/archives/
```

### Exercise 7.2: List and audit packages

```bash
# Total installed packages
dpkg -l | wc -l

# List all manually installed packages (not auto-dependencies)
apt-mark showmanual | head -20

# List packages sorted by size
dpkg-query -W --showformat='${Installed-Size}\t${Package}\n' | sort -rn | head -20
```

---

## Challenges

1. **Install `nginx`, verify it's running, then completely remove it** (including config files and dependencies)

2. **Find out which package provides the `dig` command** (hint: it's not called `dig`)

3. **Write a script that:**
   - Checks if a list of packages is installed
   - Reports which are missing
   - Optionally installs the missing ones

4. **Check your system for:**
   - Number of installed packages
   - Top 10 largest packages by size
   - Any packages that can be upgraded

---

## Quick Reference

| Task | apt (Debian/Ubuntu) | dnf (Fedora/RHEL) |
|------|--------------------|--------------------|
| Update index | `apt update` | `dnf check-update` |
| Upgrade all | `apt upgrade` | `dnf upgrade` |
| Install | `apt install pkg` | `dnf install pkg` |
| Remove | `apt remove pkg` | `dnf remove pkg` |
| Search | `apt search term` | `dnf search term` |
| Info | `apt show pkg` | `dnf info pkg` |
| List installed | `apt list --installed` | `dnf list installed` |
| Clean cache | `apt clean` | `dnf clean all` |
| Fix broken | `apt --fix-broken install` | `dnf distro-sync` |

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Not running `apt update` first | Installing from stale package lists | Always `sudo apt update` before installing |
| Using `remove` when you want `purge` | Leaves config files behind | Use `purge` for clean removal |
| Adding random PPAs | Security risk, can break your system | Only add PPAs from trusted sources |
| Never running `autoremove` | Orphaned dependencies waste disk space | Run `sudo apt autoremove` periodically |
| Installing from random `.deb` files | No automatic updates, potential conflicts | Prefer official repos when possible |

---

## What's Next?

Proceed to `task-07-storage-and-disks.md` → Disk usage, mounting, and partitions.
