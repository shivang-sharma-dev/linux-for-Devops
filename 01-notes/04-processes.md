# 04 — Processes

> A process is a running program. Every DevOps task involves managing processes — starting apps, monitoring them, killing stuck ones, and understanding what's consuming your server's resources.

---

## What is a Process?

When you run a program, the kernel creates a process — an instance of that program running in memory. Every process gets:

```
Process
├── PID (Process ID)       — unique number assigned by kernel
├── PPID (Parent PID)      — the process that spawned this one
├── UID / GID              — who owns this process
├── Memory space           — its own isolated chunk of RAM
├── File descriptors       — open files: stdin(0), stdout(1), stderr(2)
├── Working directory      — where it was started from
└── Environment variables  — its env at the time of creation
```

Every process has a parent. The very first process (PID 1) is `systemd`. It is the parent of everything.

```
systemd (PID 1)
├── sshd (PID 412)
│   └── sshd: alice (PID 891)
│       └── bash (PID 892)
│           └── python3 app.py (PID 1204)
├── nginx (PID 567)
│   ├── nginx worker (PID 568)
│   └── nginx worker (PID 569)
└── ...
```

---

## Viewing Processes

### `ps` — Process snapshot

```bash
ps                          # your processes in current shell only
ps aux                      # ALL processes, full info
ps aux | grep nginx         # find nginx processes
ps -ef                      # full format (shows PPID)
ps -ef | grep python        # find python processes
ps --sort=-%cpu             # sort by CPU usage
ps --sort=-%mem             # sort by memory usage
ps -u alice                 # processes owned by alice
ps --ppid 1234              # children of process 1234

# ps aux columns explained:
# USER   PID   %CPU  %MEM   VSZ    RSS   TTY   STAT   START   TIME   COMMAND
# alice  1234  0.5   1.2   54321  12345   ?    Ss    10:30   0:01   python3 app.py
```

**STAT column meanings:**

| Code | Meaning |
|---|---|
| `R` | Running |
| `S` | Sleeping (waiting for something) |
| `D` | Uninterruptible sleep (usually I/O) |
| `Z` | Zombie (finished but parent hasn't collected it) |
| `T` | Stopped |
| `s` | Session leader |
| `+` | Foreground process group |

### `top` — Live process monitor

```bash
top                         # live view, refreshes every 3s
top -u alice                # only show alice's processes
top -p 1234                 # monitor a specific PID

# Inside top:
# k → kill a process (enter PID)
# r → renice (change priority)
# q → quit
# M → sort by memory
# P → sort by CPU
# 1 → show individual CPU cores
```

### `htop` — Better top (install separately)

```bash
sudo apt install htop
htop                        # interactive, colourful, mouse support
htop -u alice               # filter by user
```

### `pgrep` and `pidof` — Find PIDs by name

```bash
pgrep nginx                 # print PIDs of processes named nginx
pgrep -l nginx              # print PID and name
pgrep -u alice python       # PIDs of python processes owned by alice
pidof nginx                 # similar to pgrep
```

---

## Signals — Talking to Processes

Signals are notifications sent to processes. The kernel and users can send them.

```bash
# Send a signal
kill -SIGNAL PID
kill -9 1234            # force kill PID 1234
kill -15 1234           # graceful termination (default)
kill -1 1234            # reload (SIGHUP)
kill -0 1234            # check if process exists (no actual signal)

# Kill by name instead of PID
killall nginx           # kill all processes named nginx
pkill nginx             # similar — uses pattern matching
pkill -u alice          # kill all processes owned by alice
pkill -f "python app"   # match against full command line
```

### Most important signals

| Signal | Number | Name | What happens |
|---|---|---|---|
| `SIGHUP` | 1 | Hangup | Reload config (most daemons respond to this) |
| `SIGINT` | 2 | Interrupt | Ctrl+C — graceful stop |
| `SIGKILL` | 9 | Kill | **Force kill — cannot be caught or ignored** |
| `SIGTERM` | 15 | Terminate | Graceful shutdown request (default for `kill`) |
| `SIGSTOP` | 19 | Stop | Pause process (cannot be caught) |
| `SIGCONT` | 18 | Continue | Resume a stopped process |

**Always try `kill -15` (SIGTERM) before `kill -9` (SIGKILL).** SIGTERM allows the process to clean up — flush data, close connections, remove lock files. SIGKILL gives it no chance and can cause data corruption.

```bash
# Graceful → force sequence
kill -15 1234           # ask nicely first
sleep 5
kill -9 1234            # force if still running
```

---

## Background and Foreground Jobs

```bash
# Run in background (& at end)
python3 server.py &
sleep 100 &

# See background jobs
jobs                    # list jobs in current shell
jobs -l                 # include PIDs

# Bring to foreground
fg                      # bring most recent background job
fg %2                   # bring job number 2

# Send foreground job to background
# Press Ctrl+Z first (pauses it), then:
bg                      # resume in background
bg %2                   # resume specific job

# Ctrl+Z = pause (SIGSTOP) then bg = continue in background
# Ctrl+C = terminate (SIGINT)
```

---

## `nohup` — Keep Running After Logout

When your shell closes, all child processes get SIGHUP and die. `nohup` makes a process ignore SIGHUP.

```bash
nohup python3 app.py &               # runs even after logout, output → nohup.out
nohup python3 app.py > app.log 2>&1 & # custom log file
nohup ./deploy.sh &
```

Use `tmux` or `screen` for interactive sessions instead (see Note 11). `nohup` is for fire-and-forget commands.

---

## Process Priority — nice and renice

Every process has a priority (niceness). Lower nice value = higher priority.

```
Nice value range: -20 (highest priority) to +19 (lowest priority)
Default: 0
Only root can set negative values
```

```bash
# Start a process with a specific nice value
nice -n 10 python3 heavy-job.py     # lower priority (be nice to other processes)
nice -n -5 critical-task.sh         # higher priority (root only)

# Change priority of running process
renice -n 10 -p 1234                # lower priority of PID 1234
renice -n -5 -p 1234                # higher priority (root only)
renice -n 10 -u alice               # lower priority of all alice's processes

# See current nice values
ps -eo pid,ni,comm --sort=ni | head -20
top                                  # NI column shows nice value
```

---

## `/proc` — The Process Filesystem

Every running process has a directory in `/proc/PID/`:

```bash
ls /proc/1234/
cat /proc/1234/status       # process status (name, PID, state, memory)
cat /proc/1234/cmdline      # exact command that started the process
cat /proc/1234/environ      # environment variables
ls -la /proc/1234/fd/       # open file descriptors
cat /proc/1234/maps         # memory mappings

# System-wide info
cat /proc/cpuinfo            # CPU details
cat /proc/meminfo            # memory details
cat /proc/uptime             # system uptime in seconds
cat /proc/loadavg            # load average
cat /proc/net/tcp            # active TCP connections
```

---

## Understanding Load Average

`uptime` and `top` show load average — three numbers like `0.52 0.38 0.24`.

```
Load average: 0.52  0.38  0.24
               │     │     └─ average over last 15 minutes
               │     └─ average over last 5 minutes
               └─ average over last 1 minute
```

Load average represents the average number of processes **waiting to run** (in the run queue + waiting for I/O).

**Interpreting load average:**
```
On a 4-core machine:
  Load 0.x   = idle, barely used
  Load 1.x   = one core fully busy
  Load 4.0   = all 4 cores 100% busy
  Load 6.0   = overloaded — processes waiting for CPU time
  
Rule: load average > number of cores = system is overloaded
```

```bash
nproc                       # how many cores do you have
uptime                      # see load average
top                         # shows load average at the top
```

---

## Process Management Quick Reference

```bash
# Find
ps aux | grep process_name
pgrep -l process_name
pidof process_name

# Monitor
top / htop
watch -n 2 'ps aux --sort=-%cpu | head -10'   # refresh every 2s

# Control
kill -15 PID               # graceful stop
kill -9 PID                # force kill
killall process_name       # kill all by name

# Background
command &                  # start in background
Ctrl+Z then bg             # send current to background
nohup command &            # persist after logout

# Priority
nice -n 10 command         # start with lower priority
renice -n 10 -p PID        # change priority of running process
```
