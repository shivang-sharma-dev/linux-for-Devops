# 01 — Linux Fundamentals

> Before touching any command, understand what Linux actually is and how it is structured. This mental model will make every other topic click faster.

---

## What is Linux?

Linux is an **operating system kernel** — the core software that sits between your hardware and your applications. It manages CPU, memory, storage, and devices on behalf of every program running on the machine.

```
  Your Applications  (nginx, python, docker, your scripts)
        │
  ──────┼────────── System Call Interface ──────────────
        │
     Kernel          (memory mgmt, process scheduling,
        │              filesystem, networking, drivers)
        │
     Hardware        (CPU, RAM, Disk, Network Card)
```

When people say "Linux server", they usually mean a Linux distribution — the kernel plus a collection of tools, utilities, and a package manager bundled together.

---

## Linux Distributions — Why So Many?

The Linux kernel is open source. Anyone can take it, bundle it with tools, and release a distribution (distro). Different distros target different use cases.

```
              Linux Kernel
             /      |      \
           /        |        \
       Debian     Red Hat     Arch
         |           |          |
       Ubuntu     CentOS/     Manjaro
       Linux      RHEL/       (rolling
       Mint       Fedora       release)
                  Amazon
                  Linux
```

| Distro family | Package manager | Used in |
|---|---|---|
| Debian / Ubuntu | `apt` | Most cloud servers, AWS default, beginners |
| RHEL / CentOS / Fedora | `dnf` / `yum` | Enterprise, Red Hat environments |
| Amazon Linux | `dnf` / `yum` | AWS EC2 instances |
| Alpine | `apk` | Docker containers (tiny size) |
| Arch | `pacman` | Advanced users, desktop |

**For DevOps: learn Ubuntu first.** It is the most common distro on cloud platforms and has the largest community. Everything you learn transfers to other distros.

---

## The Filesystem Hierarchy Standard (FHS)

Linux organises everything under a single root `/`. There are no `C:\` or `D:\` drives — everything is a directory under `/`.

```
/
├── bin/        Essential user command binaries (ls, cp, mv)
├── sbin/       System binaries (only root should run: fdisk, iptables)
├── etc/        Configuration files (nginx.conf, passwd, fstab)
├── home/       User home directories (/home/alice, /home/bob)
├── root/       Home directory for the root user
├── var/        Variable data — logs, caches, mail (/var/log/)
├── tmp/        Temporary files (cleared on reboot)
├── usr/        User programs and their data
│   ├── bin/    Most user commands live here (python3, git, curl)
│   ├── lib/    Libraries for /usr/bin programs
│   └── local/  Locally compiled software (not from package manager)
├── lib/        Shared libraries needed by /bin and /sbin
├── dev/        Device files (disks, terminals, random)
├── proc/       Virtual filesystem — live kernel and process info
├── sys/        Virtual filesystem — hardware and kernel info
├── mnt/        Temporary mount point for filesystems
├── media/      Mount point for removable media (USB, CD)
├── opt/        Optional third-party software
└── boot/       Kernel, bootloader, initrd
```

**The four most important directories for DevOps:**

| Directory | Why you'll be there constantly |
|---|---|
| `/etc/` | Every app stores its config here. Nginx, SSH, systemd, cron — all in `/etc/` |
| `/var/log/` | All logs. First place you go when something breaks |
| `/home/` and `/root/` | Your files, scripts, SSH keys |
| `/proc/` | Real-time system info — CPU, memory, running processes, network stats |

---

## Everything is a File

This is the core Unix philosophy that Linux inherits. In Linux, almost everything is represented as a file:

| What it is | Where it lives as a file |
|---|---|
| Your hard disk | `/dev/sda` |
| A USB drive | `/dev/sdb` |
| Your terminal | `/dev/tty` |
| A running process (PID 1234) | `/proc/1234/` |
| Kernel parameters | `/proc/sys/` |
| Network interfaces | `/sys/class/net/` |
| Random number generator | `/dev/random` |
| Null (discard output) | `/dev/null` |

This is why Linux skills transfer so well — the same file-reading commands work everywhere. Want to check CPU info? `cat /proc/cpuinfo`. Want to tune a kernel parameter? Write to `/proc/sys/`.

---

## The Shell

The shell is the program that reads your commands and executes them. It is the primary interface for DevOps work.

```
You type:  ls -la /etc
              │
         Shell parses command
              │
         Kernel executes it
              │
         Output returned to shell
              │
         Shell prints to your terminal
```

**Common shells:**

| Shell | Default on | Notes |
|---|---|---|
| `bash` | Ubuntu, most Linux | The standard, learn this first |
| `sh` | Minimal systems, scripts | POSIX compliant, least features |
| `zsh` | macOS (since 2019) | More features, compatible with bash |
| `fish` | Some distros | Beginner friendly, not POSIX |

Check your current shell:
```bash
echo $SHELL
# /bin/bash

# See all installed shells
cat /etc/shells
```

---

## Basic Orientation Commands

Run these on any new Linux system you access — they tell you where you are and what you're working with:

```bash
# Who am I and where am I
whoami                  # your username
id                      # your UID, GID, and groups
pwd                     # current directory (Print Working Directory)
hostname                # machine's hostname
hostname -I             # machine's IP address(es)

# What system is this
uname -a                # kernel version + architecture
cat /etc/os-release     # distro name and version
lsb_release -a          # distro info (Ubuntu/Debian)

# What resources does this machine have
nproc                   # number of CPU cores
free -h                 # RAM usage (human readable)
df -h                   # disk usage
uptime                  # how long system has been running + load average

# What is running
ps aux                  # all running processes
systemctl list-units --type=service --state=running   # running services
```

---

## The Linux Boot Process (Simplified)

Understanding this helps when a server won't start:

```
1. BIOS/UEFI       Hardware self-test, finds boot device
        ↓
2. Bootloader      GRUB loads the kernel into memory
(GRUB)
        ↓
3. Kernel          Initialises hardware, mounts root filesystem
        ↓
4. Init system     systemd (PID 1) starts — the parent of all processes
(systemd)
        ↓
5. Services        systemd starts services defined in unit files
        ↓
6. Login prompt    getty starts, you log in
```

```bash
# See boot messages
journalctl -b          # all logs from current boot
journalctl -b -1       # all logs from previous boot (useful after a crash)
dmesg                  # kernel ring buffer (hardware detection messages)
```

---

## Users: Root vs Regular Users

Linux has a strict separation between regular users and the superuser (root).

```
root (UID 0)
  │
  ├── Can do anything: read any file, kill any process,
  │   modify system config, install software
  │
  └── Dangerous: a typo as root can destroy the system

Regular user (UID 1000+)
  │
  ├── Can only touch their own files
  │
  └── Can run privileged commands via sudo
       (if they're in the sudo/wheel group)
```

```bash
# Run a command as root
sudo apt update

# Switch to root shell (avoid for long sessions)
sudo -i        # or: sudo su

# Run a command as another user
sudo -u www-data cat /var/log/nginx/error.log

# See who has sudo access
cat /etc/sudoers
sudo visudo    # safe way to edit sudoers
```

**Best practice:** Never log in as root directly. Use a regular user with `sudo`. This creates an audit trail and prevents accidents.

---

## Getting Help

```bash
# Built-in manual (comprehensive)
man ls
man chmod
man 5 passwd      # section 5 = config files

# Quick summary of a command
tldr ls           # install: npm install -g tldr
# or
curl cheat.sh/ls

# Built-in help (shorter than man)
ls --help
chmod --help

# What does this command do?
whatis ls         # one-line description
which python3     # where is this command installed
type cd           # is this a builtin or external command
```

---

## Key Concepts to Remember

| Concept | What it means |
|---|---|
| Everything is a file | Devices, processes, config — all accessed as files |
| Case sensitive | `File.txt` and `file.txt` are different files |
| Hidden files start with `.` | `.bashrc`, `.ssh/`, `.gitignore` |
| No file extensions required | Linux doesn't rely on `.exe`, `.bat` etc. |
| Spaces in filenames need quotes | `cd "my folder"` or `cd my\ folder` |
| `/` is both root dir and path separator | `cd /etc/nginx` |
| `~` means your home directory | `cd ~` = `cd /home/yourname` |
| `.` is current dir, `..` is parent | `cd ..` goes up one level |
