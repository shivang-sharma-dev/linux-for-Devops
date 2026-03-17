# Linux for DevOps

> A structured, beginner-to-advanced Linux learning repo built for DevOps engineers.
> Notes · Labs · Projects · Interview Prep · Resources — all in one place.

---

## Why Linux for DevOps?

Every server you will ever deploy to, every container you will ever run, every CI/CD pipeline you will ever build — runs on Linux. It is not optional knowledge. It is the foundation everything else sits on.

> "You don't need to become a Linux sysadmin. But you need to feel at home on a Linux server the way you feel at home in your own room."

---

## What This Repo Covers

```
The Linux Mental Model for DevOps
══════════════════════════════════════════════════════════════

  Everything is a file          The shell is your superpower
  ─────────────────────         ────────────────────────────
  /etc  → config files          Commands → pipes → output
  /var  → runtime data          Scripts  → automation
  /proc → live kernel data      SSH      → remote control
  /dev  → devices as files


  The 4 things you do every day on a Linux server
  ────────────────────────────────────────────────
  1. Navigate & find files      (filesystem)
  2. Control who can do what    (users + permissions)
  3. Manage what is running     (processes + services)
  4. Automate repetitive work   (shell scripting + cron)
```

---

## Repo Structure

```
linux-for-devops/
│
├── 01-notes/               ← 15 topic files, concept-first approach
├── 02-cheatsheet/          ← one-page quick reference
├── 03-hands-on-labs/
│   ├── online-platforms.md ← practice in the browser (no setup needed)
│   └── local-tasks/        ← 7 structured exercises on your own machine
├── 04-projects/
│   ├── beginner/           ← system health checker, user management script
│   └── intermediate/       ← log analyser, backup automation
├── 05-interview-prep/      ← Q&A organised by topic
├── 06-resources/           ← curated books, courses, YouTube, docs
└── 07-troubleshooting/     ← real scenarios with step-by-step fixes
```

---

## How to Use This Repo

**If you are a complete beginner:**
Start at `01-notes/01-linux-fundamentals.md` and go in order.
Do each lab task after finishing its corresponding note file.

**If you know the basics:**
Use the [ROADMAP](./ROADMAP.md) to identify your gaps.
Jump directly to the topic files you need.

**If you are preparing for interviews:**
Go straight to `05-interview-prep/` — it maps to the note files so you can quickly review anything you're unsure about.

**If you are debugging something right now:**
Check `07-troubleshooting/` — it has step-by-step guides for the most common Linux problems in production.

---

## Topics Covered

| # | Topic | Why It Matters for DevOps |
|---|---|---|
| 01 | Linux Fundamentals | Understand what you're working with |
| 02 | File System | Everything in Linux is a file |
| 03 | Users & Permissions | Misconfigs cause security incidents |
| 04 | Processes | Know what's running on your servers |
| 05 | Networking | Diagnose connectivity issues fast |
| 06 | Shell Scripting | Automate everything |
| 07 | Package Management | Install and manage software |
| 08 | Storage & Disks | Manage volumes in cloud environments |
| 09 | Systemd & Services | Run and manage applications as services |
| 10 | Logs & Monitoring | Debugging starts with reading logs |
| 11 | SSH & Remote Access | 90% of DevOps work is remote |
| 12 | Cron & Automation | Schedule tasks reliably |
| 13 | Performance Tuning | Diagnose slow servers |
| 14 | Security Hardening | Secure your Linux servers |
| 15 | Linux for Containers | How Docker uses Linux under the hood |

---

## Prerequisites

- A computer with Linux installed (Ubuntu 22.04 recommended), OR
- WSL2 on Windows, OR
- A free cloud VM (AWS Free Tier, GCP Free Tier, Oracle Always Free)
- Basic comfort with typing commands (you will learn the rest here)

---

## Progress Tracker

Copy this into your own fork and check off as you go:

- [ ] 01 Linux Fundamentals
- [ ] 02 File System
- [ ] 03 Users & Permissions
- [ ] 04 Processes
- [ ] 05 Networking
- [ ] 06 Shell Scripting
- [ ] 07 Package Management
- [ ] 08 Storage & Disks
- [ ] 09 Systemd & Services
- [ ] 10 Logs & Monitoring
- [ ] 11 SSH & Remote Access
- [ ] 12 Cron & Automation
- [ ] 13 Performance Tuning
- [ ] 14 Security Hardening
- [ ] 15 Linux for Containers
- [ ] All local lab tasks completed
- [ ] At least one project built
- [ ] Interview prep reviewed

---

## Contributing

Found a mistake? Want to add a lab task or resource?
See [CONTRIBUTING.md](./CONTRIBUTING.md) — contributions from fellow students are welcome.

---

*Part of the [DevOps Mastery](https://github.com/yourusername) learning series.*
