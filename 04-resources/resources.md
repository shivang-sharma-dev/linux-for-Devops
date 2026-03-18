# Linux for DevOps — Resources

> Curated resources organised by type. Every link here is free or has a free tier unless marked 💰.

---

## Official Documentation

| Resource | Link | What it covers |
|---|---|---|
| Linux man pages online | https://man7.org/linux/man-pages/ | Every command, system call, config file |
| GNU Coreutils docs | https://www.gnu.org/software/coreutils/manual/ | ls, cp, chmod, chown and all core tools |
| systemd docs | https://systemd.io | Unit files, journalctl, timers |
| Bash manual | https://www.gnu.org/software/bash/manual/ | Complete bash reference |
| Ubuntu Server docs | https://ubuntu.com/server/docs | Ubuntu-specific guides |
| RHEL docs | https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/ | Red Hat specific |
| Linux kernel docs | https://www.kernel.org/doc/html/latest/ | Kernel internals, /proc, cgroups |

---

## Interactive Practice (Browser-Based — No Setup Needed)

| Platform | Link | Best for | Difficulty |
|---|---|---|---|
| KillerCoda | https://killercoda.com/learn | Linux scenarios, systemd, SSH | Beginner → Advanced |
| Linux Survival | https://linuxsurvival.com | Basic navigation and commands | Beginner |
| OverTheWire: Bandit | https://overthewire.org/wargames/bandit/ | Gamified Linux challenges | Beginner → Intermediate |
| TryHackMe — Linux Fundamentals | https://tryhackme.com/module/linux-fundamentals | Structured Linux rooms | Beginner |
| Play With Docker | https://labs.play-with-docker.com | Free Linux terminal in browser | Any |
| Webminal | https://www.webminal.org | Online Linux terminal | Beginner |
| JSLinux | https://bellard.org/jslinux/ | Linux in the browser (no signup) | Beginner |

---

## Free Courses

| Course | Link | Platform | What it covers |
|---|---|---|---|
| The Linux Command Line | https://linuxcommand.org/tlcl.php | Free book/site | Complete CLI guide |
| Linux Journey | https://linuxjourney.com | linuxjourney.com | Interactive lessons by topic |
| NDG Linux Essentials | https://www.netacad.com/courses/os-it/ndg-linux-essentials | Cisco NetAcad | Full Linux course (free signup) |
| Linux Foundation: Intro to Linux | https://training.linuxfoundation.org/training/introduction-to-linux/ | edX | Free audit available |
| Bash Scripting Tutorial | https://ryanstutorials.net/bash-scripting-tutorial/ | ryanstutorials.net | Shell scripting from scratch |
| ShellCheck | https://www.shellcheck.net | Tool | Lint and fix your shell scripts |
| Regex101 | https://regex101.com | Tool | Test grep/sed/awk patterns live |

---

## YouTube Channels

| Channel | Link | Why watch it |
|---|---|---|
| NetworkChuck | https://www.youtube.com/@NetworkChuck | Engaging Linux + DevOps content for beginners |
| TechWorld with Nana | https://www.youtube.com/@TechWorldwithNana | Best DevOps content, Linux included |
| Christian Lempa | https://www.youtube.com/@christianlempa | Homelab, Linux server setup |
| LearnLinuxTV | https://www.youtube.com/@LearnLinuxTV | Dedicated Linux channel, very detailed |
| DorianDotSlash | https://www.youtube.com/@DorianDotSlash | Linux on desktop + server |
| LiveOverflow | https://www.youtube.com/@LiveOverflow | Linux internals, security |
| Low Level Learning | https://www.youtube.com/@LowLevelLearning | How Linux works under the hood |

### Must-Watch Videos

| Video | Link | Topic |
|---|---|---|
| Linux Directories Explained | https://www.youtube.com/watch?v=42iQKuQodW4 | FHS — why each folder exists |
| You Need to Learn Linux RIGHT NOW | https://www.youtube.com/watch?v=a8CwZhhZYrY | NetworkChuck Linux intro |
| Bash Scripting Full Course | https://www.youtube.com/watch?v=e7BufAVwDiM | 3 hour scripting tutorial |
| Linux Permissions Explained | https://www.youtube.com/watch?v=LnKoncbQBsM | chmod, chown, umask |
| systemd in 100 seconds | https://www.youtube.com/watch?v=Kzpm-rGAXos | Fireship — quick visual |
| SSH Crash Course | https://www.youtube.com/watch?v=hQWRp-FdTpc | Traversy Media |

---

## Books

| Book | Author | Level | Notes |
|---|---|---|---|
| The Linux Command Line | William Shotts | Beginner | Free at https://linuxcommand.org/tlcl.php — the best starter book |
| How Linux Works | Brian Ward | Intermediate | Explains internals in plain language |
| Linux Bible | Christopher Negus | Intermediate | Comprehensive reference |
| UNIX and Linux System Administration Handbook | Nemeth et al | Advanced | The sysadmin bible 💰 |
| The Linux Programming Interface | Michael Kerrisk | Advanced | Deep internals — system calls, /proc 💰 |
| Linux Pocket Guide | Daniel J. Barrett | Any | Quick reference, great for the desk 💰 |

---

## Cheat Sheet Sites

| Site | Link | What it has |
|---|---|---|
| cheat.sh | https://cheat.sh | `curl cheat.sh/chmod` — terminal cheatsheets |
| tldr.sh | https://tldr.sh | Simplified man pages with examples |
| explainshell.com | https://explainshell.com | Paste any command — it explains every part |
| devhints.io | https://devhints.io/bash | Bash scripting cheatsheet |
| ss64.com | https://ss64.com/bash/ | Full command reference A-Z |
| crontab.guru | https://crontab.guru | Test cron expressions visually |
| chmod calculator | https://chmod-calculator.com | Visual chmod permission calculator |

---

## Tools Worth Installing

| Tool | Install | What it does |
|---|---|---|
| `htop` | `apt install htop` | Better top — interactive process monitor |
| `ncdu` | `apt install ncdu` | Interactive disk usage explorer |
| `tldr` | `apt install tldr` | Simplified man pages |
| `bat` | `apt install bat` | Better cat — syntax highlighted output |
| `fd` | `apt install fd-find` | Better find — faster, friendlier syntax |
| `ripgrep` | `apt install ripgrep` | Better grep — very fast |
| `fzf` | `apt install fzf` | Fuzzy finder — search history, files |
| `tmux` | `apt install tmux` | Terminal multiplexer — persistent sessions |
| `mtr` | `apt install mtr` | Better traceroute — live network path |
| `nethogs` | `apt install nethogs` | Network usage per process |
| `iotop` | `apt install iotop` | Disk I/O per process |
| `glances` | `apt install glances` | All-in-one system monitor |
| `dstat` | `apt install dstat` | Live system stats — CPU, mem, disk, net |
| `jq` | `apt install jq` | Parse and format JSON in terminal |
| `watch` | built-in | Re-run command every N seconds |
| `lnav` | `apt install lnav` | Log file navigator — coloured, searchable |

---

## Linux Certifications (if you want to go further)

| Certification | Issuer | Level | Cost |
|---|---|---|---|
| Linux Essentials (LPI 010) | LPI | Beginner | ~$120 |
| LPIC-1 | LPI | Intermediate | ~$200 per exam |
| CompTIA Linux+ | CompTIA | Intermediate | ~$370 💰 |
| RHCSA | Red Hat | Intermediate | ~$400 💰 |
| RHCE | Red Hat | Advanced | ~$400 💰 |
| LFCS | Linux Foundation | Intermediate | ~$395 💰 |

> For DevOps roles, certifications are optional — a strong GitHub repo with real projects matters more. But RHCSA/LFCS look great on a resume.

---

## Communities and Forums

| Community | Link | Best for |
|---|---|---|
| r/linux | https://reddit.com/r/linux | News, discussions |
| r/linuxquestions | https://reddit.com/r/linuxquestions | Getting help |
| r/devops | https://reddit.com/r/devops | DevOps-specific Linux questions |
| Unix & Linux Stack Exchange | https://unix.stackexchange.com | Deep technical Q&A |
| Ask Ubuntu | https://askubuntu.com | Ubuntu-specific |
| Linux Questions forums | https://www.linuxquestions.org | Active community |

---

## Newsletters and Blogs

| Name | Link | What it covers |
|---|---|---|
| Julia Evans (b0rk) | https://jvns.ca | Linux internals explained beautifully |
| Brendan Gregg | https://brendangregg.com | Performance tuning — the best in the world |
| The Geek Stuff | https://thegeekstuff.com | Practical Linux how-tos |
| Linux Handbook | https://linuxhandbook.com | Beginner friendly tutorials |
| It's FOSS | https://itsfoss.com | Linux news and tutorials |

---

## Practice Platforms with Linux Tracks

| Platform | Link | Free tier |
|---|---|---|
| TryHackMe | https://tryhackme.com | Yes — Linux Fundamentals path |
| HackTheBox | https://hackthebox.com | Yes — Linux machines |
| OverTheWire | https://overthewire.org | Fully free |
| PentesterLab | https://pentesterlab.com | Some free content |
| VulnHub | https://vulnhub.com | Free — download VMs locally |

---

## Map: Which Resource for Which Note

| Note file | Best resource to pair with |
|---|---|
| 01-linux-fundamentals | Linux Journey → Grasshopper section |
| 02-file-system | explainshell.com + tldr.sh |
| 03-users-and-permissions | chmod-calculator.com + TryHackMe Linux Fundamentals 2 |
| 04-processes | Brendan Gregg's site for deep dives |
| 05-networking | KillerCoda networking scenarios |
| 06-shell-scripting | ShellCheck + devhints.io/bash |
| 07-package-management | Ubuntu Server docs |
| 08-storage-and-disks | How Linux Works (book) chapter on filesystems |
| 09-systemd-and-services | systemd.io official docs |
| 10-logs-and-monitoring | Brendan Gregg — USE Method |
| 11-ssh-and-remote | SSH Crash Course (YouTube) + man ssh_config |
| 12-cron-and-automation | crontab.guru |
| 13-performance-tuning | Brendan Gregg's perf tools |
| 14-security-hardening | Lynis docs + CIS Benchmarks |
| 15-linux-for-containers | Linux kernel docs on namespaces + cgroups |
