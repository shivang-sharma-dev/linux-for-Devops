# Task 03 — Process Management & Signals

> **Prerequisite:** Read `01-notes/04-processes.md`
> **Environment:** Any Linux terminal (native, WSL2, VM, or container)
> **Difficulty:** Beginner

---

## Objective

By the end of this lab you will be able to:
- List running processes and understand the output
- Run processes in the foreground and background
- Send signals to processes (kill, stop, continue)
- Monitor system resources with `top` and `htop`
- Identify and kill a process using excessive resources
- Understand parent-child process relationships

---

## Part 1 — Viewing Processes

### Exercise 1.1: Basic process listing

```bash
# Show your processes
ps

# Show all processes (full format)
ps aux

# Show all processes (tree view — parent/child)
ps auxf

# Show processes for a specific user
ps -u $(whoami)

# Count total running processes
ps aux | wc -l
```

### Exercise 1.2: Understanding `ps aux` output

```bash
ps aux | head -5
```

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1 169580 13256 ?        Ss   10:00   0:01 /sbin/init
root         2  0.0  0.0      0     0 ?        S    10:00   0:00 [kthreadd]
```

| Column | Meaning |
|--------|---------|
| USER | Process owner |
| PID | Process ID (unique number) |
| %CPU | CPU usage percentage |
| %MEM | Memory usage percentage |
| VSZ | Virtual memory size |
| RSS | Resident (actual) memory used |
| STAT | Process state (S=sleeping, R=running, Z=zombie) |
| COMMAND | The command that started the process |

### Exercise 1.3: Find specific processes

```bash
# Find processes by name
ps aux | grep bash

# Better way — avoids showing the grep itself
pgrep -a bash

# Find a process by PID
ps -p 1          # PID 1 is init/systemd

# Find what's using a specific port
sudo ss -tulnp | grep :22     # What's on port 22?
```

---

## Part 2 — Foreground & Background Processes

### Exercise 2.1: Run commands in the background

```bash
# Start a long-running process in the foreground
sleep 300         # This blocks your terminal — press Ctrl+C to stop it

# Start it in the background instead
sleep 300 &
# Output: [1] 12345  → job number and PID

# List background jobs
jobs

# Start another background job
sleep 600 &

# List all jobs
jobs -l           # Shows PIDs too
```

### Exercise 2.2: Move processes between foreground and background

```bash
# Start a process in the foreground
sleep 300

# Press Ctrl+Z to STOP (pause) it
# Output: [1]+  Stopped     sleep 300

# Check jobs
jobs              # Shows "Stopped"

# Resume it in the BACKGROUND
bg %1

# Check again
jobs              # Shows "Running"

# Bring it back to the FOREGROUND
fg %1

# Now press Ctrl+C to kill it
```

**Summary:**
- `Ctrl+C` → Kill the foreground process
- `Ctrl+Z` → Pause (stop) the foreground process
- `bg %N` → Resume stopped job N in background
- `fg %N` → Bring job N to foreground
- `command &` → Start in background directly

---

## Part 3 — Signals and Killing Processes

### Exercise 3.1: Understanding signals

```bash
# List all available signals
kill -l
```

| Signal | Number | Meaning | Keyboard |
|--------|--------|---------|----------|
| SIGHUP | 1 | Hangup — reload config | — |
| SIGINT | 2 | Interrupt | Ctrl+C |
| SIGKILL | 9 | Force kill (cannot be caught) | — |
| SIGTERM | 15 | Graceful termination (default) | — |
| SIGSTOP | 19 | Pause process | Ctrl+Z |
| SIGCONT | 18 | Resume paused process | — |

### Exercise 3.2: Kill processes

```bash
# Start some background processes
sleep 1000 &
sleep 2000 &
sleep 3000 &

# Check them
jobs -l

# Kill by PID (sends SIGTERM — graceful)
kill <PID_of_sleep_1000>

# Kill by job number
kill %2

# Force kill (SIGKILL — when process won't die)
kill -9 <PID_of_sleep_3000>

# Verify they're gone
jobs
```

### Exercise 3.3: Kill by name

```bash
# Start multiple sleep processes
sleep 500 &
sleep 500 &
sleep 500 &

# Kill ALL processes named "sleep"
killall sleep

# Verify
jobs

# Alternative: pkill (supports pattern matching)
sleep 500 &
pkill -f "sleep 500"
```

---

## Part 4 — Monitoring with `top` and `htop`

### Exercise 4.1: Using `top`

```bash
top
```

While in `top`:
- Press `P` → Sort by CPU usage
- Press `M` → Sort by memory usage
- Press `k` → Kill a process (enter PID)
- Press `1` → Show individual CPU cores
- Press `h` → Help
- Press `q` → Quit

### Exercise 4.2: Using `htop` (if installed)

```bash
# Install if not available
sudo apt install htop -y    # Debian/Ubuntu
# sudo dnf install htop -y  # Fedora/RHEL

htop
```

`htop` is more user-friendly:
- Use arrow keys to navigate
- Press `F6` to sort
- Press `F9` to send signals
- Press `F5` for tree view
- Press `F10` or `q` to quit

### Exercise 4.3: Snapshot monitoring

```bash
# Check memory usage
free -h

# Check CPU info
lscpu
nproc                   # Number of CPU cores

# Check disk I/O
iostat                  # May need: sudo apt install sysstat

# Check who is logged in
w

# Check load average
uptime
```

**Load average explained:**
```
load average: 0.50, 0.75, 1.00
                │      │      │
                │      │      └── 15-min average
                │      └── 5-min average
                └── 1-min average
```

If load average > number of CPU cores → system is overloaded.

---

## Part 5 — Process Priority (nice)

### Exercise 5.1: Set process priority

```bash
# Start a process with low priority (nice)
nice -n 19 sleep 1000 &

# Start with high priority (needs sudo)
sudo nice -n -20 sleep 1000 &

# Check priority
ps -eo pid,ni,comm | grep sleep

# Change priority of a running process
renice 10 -p <PID>
```

**Nice values:**
- Range: -20 (highest priority) to 19 (lowest priority)
- Default: 0
- Only root can set negative (high priority) values

---

## Part 6 — Zombie and Orphan Processes

### Exercise 6.1: Understand process states

```bash
# Check for zombie processes
ps aux | grep Z

# A zombie shows as:
# user  PID  0.0  0.0  0  0 ?  Z  TIME defunct
```

A **zombie** is a process that has finished but its parent hasn't acknowledged it yet. Usually harmless unless there are thousands.

```bash
# Find parent of a zombie
ps -eo pid,ppid,stat,comm | grep Z

# Then check the parent
ps -p <PPID>
```

---

## Challenges

1. **Resource hog detection:**
   - Open `top` or `htop`
   - In another terminal, run: `yes > /dev/null &` (this will eat 100% of one CPU core)
   - Find the process in `top` and note its PID
   - Kill it from within `top` (press `k`)

2. **Process tree:**
   - Run `pstree -p` and identify:
     - What is PID 1?
     - What process started your terminal?
     - How deep is your shell's process tree?

3. **Background job management:**
   - Start 5 different `sleep` commands with different durations in the background
   - List them all with `jobs -l`
   - Kill the 3rd one by job number
   - Bring the 2nd one to the foreground
   - Stop it with Ctrl+Z, then resume in background

4. **Find the top 5 memory-consuming processes:**
   ```bash
   ps aux --sort=-%mem | head -6
   ```

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Using `kill -9` as first option | Doesn't let the process clean up gracefully | Try `kill` (SIGTERM) first, `kill -9` only if it's stuck |
| `Ctrl+Z` thinking it kills a process | It only pauses it — still using memory | Use `Ctrl+C` to kill, or `bg`/`fg` after `Ctrl+Z` |
| Ignoring zombie processes | Many zombies indicate a buggy parent process | Find and fix/restart the parent process |
| Not checking `jobs` before logging out | Background jobs die when you log out | Use `nohup command &` or `tmux`/`screen` to persist |

---

## Cleanup

```bash
# Kill any remaining sleep processes
killall sleep 2>/dev/null
```

---

## What's Next?

Proceed to `task-04-networking-basics.md` → Network interfaces, ports, and DNS.
