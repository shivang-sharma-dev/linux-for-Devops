# Online Practice Platforms

> Practice Linux in your browser — no local setup needed. Great for beginners or when you're away from your main machine.

---

## Wargames & Challenges

These teach Linux through progressive challenges. You start easy and each level gets harder.

### OverTheWire — Bandit (Highly Recommended)

- **Link:** https://overthewire.org/wargames/bandit/
- **What it is:** 34 levels that teach you Linux commands through a capture-the-flag style game
- **What you learn:** File navigation, permissions, SSH, redirection, pipes, find, grep, base64, compression, networking basics
- **How it works:** SSH into their server, find the password for the next level using only the command line
- **Difficulty:** Beginner → Intermediate
- **Cost:** Free

**Start here if you've never done a Linux challenge before.** Bandit is the gold standard.

After Bandit, continue with:
- **Leviathan** — https://overthewire.org/wargames/leviathan/ (file permissions, setuid)
- **Natas** — https://overthewire.org/wargames/natas/ (web security basics)
- **Narnia** — https://overthewire.org/wargames/narnia/ (binary exploitation intro)

---

### Sadservers

- **Link:** https://sadservers.com
- **What it is:** Real Linux troubleshooting scenarios on actual VMs in your browser
- **What you learn:** Debugging real-world problems — disk full, service won't start, network issues, permissions broken
- **Difficulty:** Beginner → Advanced
- **Cost:** Free

**Best for:** Practicing the kind of problems you'll face in real DevOps jobs.

---

### Linux Survival

- **Link:** https://linuxsurvival.com
- **What it is:** Interactive tutorial with a built-in terminal simulator
- **What you learn:** Basic commands — ls, cd, mv, cp, rm, permissions, find
- **Difficulty:** Absolute beginner
- **Cost:** Free

**Best for:** Your very first time using Linux commands.

---

### Killercoda (formerly Katacoda)

- **Link:** https://killercoda.com
- **What it is:** Browser-based interactive labs with real Ubuntu environments
- **What you learn:** Linux basics, Docker, Kubernetes, networking — structured scenarios
- **Difficulty:** Beginner → Advanced
- **Cost:** Free

**Best for:** Guided, step-by-step labs when you want structure.

---

### Linux Journey

- **Link:** https://linuxjourney.com
- **What it is:** Learn Linux in bite-sized lessons with quizzes
- **What you learn:** Grasshopper (basics) → Journeyman (intermediate) → Networking Nomad (networking)
- **Difficulty:** Beginner → Intermediate
- **Cost:** Free

---

### HackTheBox

- **Link:** https://www.hackthebox.com
- **What it is:** Cybersecurity CTF platform with vulnerable machines to exploit
- **What you learn:** Advanced Linux, privilege escalation, networking, security
- **Difficulty:** Intermediate → Advanced
- **Cost:** Free tier available (limited machines)

**Best for:** After you're comfortable with Linux basics and want to go deep into security.

---

### TryHackMe

- **Link:** https://tryhackme.com
- **What it is:** Beginner-friendly cybersecurity platform with guided rooms
- **What you learn:** Linux fundamentals, networking, bash scripting, security, CTF challenges
- **Difficulty:** Beginner → Advanced
- **Cost:** Free tier available

**Best for:** Beginners who want guided paths rather than being dropped into a challenge.

Recommended rooms:
- Linux Fundamentals Part 1, 2, 3
- Bash Scripting
- Linux Privilege Escalation

---

## Online Linux Terminals

Need a quick Linux terminal without installing anything? Use these:

| Platform | Description | Link |
|----------|-------------|------|
| **JSLinux** | Full Linux running in your browser (Bellard's project) | https://bellard.org/jslinux/ |
| **Webminal** | Free online Linux terminal for practice | https://www.webminal.org |
| **copy.sh/v86** | x86 PC emulator running Linux in browser | https://copy.sh/v86/ |
| **Google Cloud Shell** | Full Debian VM in browser (free, needs Google account) | https://shell.cloud.google.com |
| **GitHub Codespaces** | Full VS Code + terminal in browser (free tier: 60 hrs/month) | https://github.com/codespaces |

---

## Recommended Practice Path

```
Absolute Beginner:
  1. Linux Survival          → Learn basic commands
  2. Linux Journey           → Structured lessons with quizzes
  3. OverTheWire Bandit      → Apply what you learned

Intermediate:
  4. Killercoda              → Guided real-VM labs
  5. Sadservers              → Debug real problems
  6. TryHackMe               → Security-focused Linux practice

Advanced:
  7. HackTheBox              → Full CTF challenges
  8. OverTheWire (Leviathan+)→ Binary & privilege challenges
```

---

> **Start with OverTheWire Bandit.** It's free, requires no setup (just SSH from your terminal), and teaches more practical Linux than most courses. Complete all 34 levels and you'll be more comfortable on a Linux terminal than most junior DevOps engineers.
