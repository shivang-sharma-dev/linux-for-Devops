# Linux for DevOps — Learning Roadmap

> Follow this roadmap in order. Each phase builds on the last.
> Estimated total time: 4–6 weeks at 1–2 hours per day.

---

## The Big Picture

```
PHASE 1 — Get Comfortable         PHASE 2 — Go Deeper
─────────────────────────         ──────────────────────────────
  Linux Fundamentals                  Shell Scripting
       ↓                                   ↓
  File System                        Package Management
       ↓                                   ↓
  Users & Permissions                Storage & Disks
       ↓                                   ↓
  Processes                          Systemd & Services
       ↓                                   ↓
  Networking basics                  Logs & Monitoring


PHASE 3 — Work Like a Pro         PHASE 4 — Production Ready
─────────────────────────         ──────────────────────────────
  SSH & Remote Access                 Performance Tuning
       ↓                                   ↓
  Cron & Automation                  Security Hardening
                                          ↓
                                    Linux for Containers
```

---

## Phase 1 — Get Comfortable (Week 1)

**Goal:** Feel at home navigating and managing a Linux system.

| Step | File | Time | Lab |
|---|---|---|---|
| 1 | `01-notes/01-linux-fundamentals.md` | 1 hr | Explore your system — `uname`, `whoami`, `ls /` |
| 2 | `01-notes/02-file-system.md` | 1.5 hr | `task-01-navigation.md` |
| 3 | `01-notes/03-users-and-permissions.md` | 2 hr | `task-02-permissions.md` |
| 4 | `01-notes/04-processes.md` | 1.5 hr | `task-03-processes.md` |
| 5 | `01-notes/05-networking.md` | 1 hr | Run `ip a`, `ss -tulnp`, `curl` on your system |

**Phase 1 checkpoint — you should be able to:**
- Navigate the filesystem without looking up commands
- Create users, set file permissions, use `sudo` confidently
- Find and kill a process by name
- Check which ports are listening on your machine

---

## Phase 2 — Go Deeper (Week 2–3)

**Goal:** Automate tasks and understand how a Linux server actually runs.

| Step | File | Time | Lab |
|---|---|---|---|
| 6 | `01-notes/06-shell-scripting.md` | 3 hr | `task-04-scripting.md` (write 3 scripts) |
| 7 | `01-notes/07-package-management.md` | 1 hr | Install, remove, pin package versions |
| 8 | `01-notes/08-storage-and-disks.md` | 1.5 hr | Check disk usage, mount a filesystem |
| 9 | `01-notes/09-systemd-and-services.md` | 2 hr | `task-05-services.md` (write a unit file) |
| 10 | `01-notes/10-logs-and-monitoring.md` | 1.5 hr | Read logs, grep for errors, follow a log live |

**Phase 2 checkpoint — you should be able to:**
- Write a shell script with variables, loops, and conditionals
- Install packages from official and third-party repos
- Write and enable a custom systemd service
- Find errors in `/var/log/` and `journalctl`

---

## Phase 3 — Work Like a Pro (Week 3–4)

**Goal:** Work on remote servers confidently. Automate scheduled tasks.

| Step | File | Time | Lab |
|---|---|---|---|
| 11 | `01-notes/11-ssh-and-remote.md` | 2 hr | `task-06-ssh-setup.md` |
| 12 | `01-notes/12-cron-and-automation.md` | 1 hr | Schedule a disk-check script with cron |

**Phase 3 checkpoint — you should be able to:**
- SSH into a server using key-based auth (no password)
- Configure `~/.ssh/config` for multiple servers
- Set up a cron job and verify it runs correctly
- Use `tmux` to keep sessions alive on remote servers

---

## Phase 4 — Production Ready (Week 4–6)

**Goal:** Harden, tune, and prepare Linux for real DevOps work.

| Step | File | Time | Lab |
|---|---|---|---|
| 13 | `01-notes/13-performance-tuning.md` | 2 hr | Stress-test your system and diagnose it |
| 14 | `01-notes/14-security-hardening.md` | 2 hr | Harden a fresh Ubuntu server |
| 15 | `01-notes/15-linux-for-containers.md` | 1.5 hr | Explore namespaces and cgroups manually |

**Phase 4 checkpoint — you should be able to:**
- Identify what's causing high CPU/memory/disk I/O on a server
- Apply basic security hardening to a fresh server
- Explain how containers use Linux namespaces and cgroups

---

## After Completing All Phases

1. Build at least one project from `04-projects/`
2. Do the OverTheWire Bandit challenges (see `06-resources/`)
3. Review `05-interview-prep/` before any DevOps interview
4. Move on to the **Docker** repo — Linux is the foundation Docker builds on

---

## Topic Dependency Map

```
linux-fundamentals
      │
      ├──► file-system ──────────────────► shell-scripting
      │         │                                │
      │         └──► users-and-permissions       └──► cron-and-automation
      │                     │
      ├──► processes ────────┤
      │                     │
      └──► networking        └──► security-hardening
                │
                └──► ssh-and-remote


package-management ──► storage-and-disks
                              │
                    systemd-and-services ──► logs-and-monitoring
                                                     │
                                           performance-tuning


                    linux-for-containers
                    (requires: processes, networking, storage)
```

---

## Time Estimates

| Your situation | Time to complete |
|---|---|
| Absolute beginner | 6–8 weeks |
| Know basic commands | 3–4 weeks |
| Intermediate, filling gaps | 1–2 weeks |
| Pre-interview review | 3–5 days |
