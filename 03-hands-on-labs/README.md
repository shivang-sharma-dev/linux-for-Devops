# Hands-On Labs

> Learn by doing. Each lab maps to a notes file and gives you real commands to run on a real Linux system.

---

## How to Use These Labs

1. **Read the notes first** — each lab lists its prerequisite note file
2. **Use a real Linux environment** — not just reading, actually type every command
3. **Don't copy-paste** — type commands manually to build muscle memory
4. **Try the challenges** — they push you beyond the basics
5. **Break things** — that's how you learn. Use a VM or container so you can reset

---

## Environment Options

You need a Linux terminal to do these labs. Pick one:

| Option | Best For | Setup |
|--------|----------|-------|
| **Native Linux** | Already running Linux | None needed |
| **WSL2** | Windows users | `wsl --install` in PowerShell |
| **VirtualBox + Ubuntu** | Full VM experience | Install VirtualBox → create Ubuntu VM |
| **Cloud VM** | Real server experience | AWS/GCP/Oracle free tier |
| **Docker container** | Quick throwaway env | `docker run -it ubuntu:22.04 bash` |
| **Online platforms** | No setup at all | See `online-platforms.md` |

> For most labs, any of these options work. Labs that require `systemd` (task-08) or kernel features (task-14) need a full VM, not a container.

---

## Lab Overview

### Phase 1 — Get Comfortable

| Lab | Topic | Prerequisite Note |
|-----|-------|-------------------|
| `task-01-navigation.md` | Filesystem navigation & file ops | `01-linux-fundamentals.md` + `02-file-system.md` |
| `task-02-permissions.md` | Users, groups, chmod, chown | `03-users-and-permissions.md` |
| `task-03-processes.md` | Process management & signals | `04-processes.md` |
| `task-04-networking-basics.md` | Network interfaces, ports, DNS | `05-networking.md` |

### Phase 2 — Go Deeper

| Lab | Topic | Prerequisite Note |
|-----|-------|-------------------|
| `task-05-scripting.md` | Write 3 shell scripts from scratch | `06-shell-scripting.md` |
| `task-06-package-management.md` | Install, remove, manage packages | `07-package-management.md` |
| `task-07-storage-and-disks.md` | Disk usage, mounting, partitions | `08-storage-and-disks.md` |
| `task-08-services.md` | Write a custom systemd service | `09-systemd-and-services.md` |
| `task-09-logs.md` | Read, search, and analyze logs | `10-logs-and-monitoring.md` |

### Phase 3 — Work Like a Pro

| Lab | Topic | Prerequisite Note |
|-----|-------|-------------------|
| `task-10-ssh-setup.md` | SSH keys, config, tunnels | `11-ssh-and-remote.md` |
| `task-11-cron-automation.md` | Schedule and verify cron jobs | `12-cron-and-automation.md` |

### Phase 4 — Production Ready

| Lab | Topic | Prerequisite Note |
|-----|-------|-------------------|
| `task-12-performance.md` | Diagnose CPU, memory, disk issues | `13-performance-tuning.md` |
| `task-13-security-hardening.md` | Harden a fresh Linux server | `14-security-hardening.md` |
| `task-14-containers-basics.md` | Namespaces, cgroups, container fundamentals | `15-linux-for-containers.md` |

---

## Progress Tracker

Copy this and check off as you complete each lab:

- [ ] Task 01 — Navigation
- [ ] Task 02 — Permissions
- [ ] Task 03 — Processes
- [ ] Task 04 — Networking Basics
- [ ] Task 05 — Scripting
- [ ] Task 06 — Package Management
- [ ] Task 07 — Storage & Disks
- [ ] Task 08 — Services
- [ ] Task 09 — Logs
- [ ] Task 10 — SSH Setup
- [ ] Task 11 — Cron Automation
- [ ] Task 12 — Performance
- [ ] Task 13 — Security Hardening
- [ ] Task 14 — Containers Basics

---

## Rules

1. **Never run labs on a production server** — use a VM, container, or cloud sandbox
2. **Read the "Common Mistakes" section** in each lab before starting
3. **If you break something, fix it** — that's the most valuable part of the exercise
4. **Document what you learn** — add notes to your fork of this repo
