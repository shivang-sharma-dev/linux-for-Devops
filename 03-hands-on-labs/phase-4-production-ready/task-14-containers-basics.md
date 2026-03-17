# Task 14 — Linux Container Fundamentals

> **Prerequisite:** Read `01-notes/15-linux-for-containers.md`
> **Environment:** Full Linux system (native or VM — NOT inside a container). You need `sudo` access and a kernel version 4.x+.
> **Difficulty:** Advanced

---

## Objective

By the end of this lab you will be able to:
- Understand what namespaces and cgroups are
- Create isolated environments manually using Linux primitives
- See how containers work under the hood
- Compare manual isolation with Docker containers
- Understand why "containers are just Linux processes"

---

## Part 1 — Namespaces

Namespaces are what make a process *think* it's alone on the system. Each namespace type isolates a different resource.

### Exercise 1.1: List your current namespaces

```bash
# See namespaces of the current process
ls -la /proc/$$/ns/

# Types of namespaces:
# cgroup  — cgroup root directory
# ipc     — inter-process communication
# mnt     — mount points
# net     — network stack
# pid     — process IDs
# user    — user/group IDs
# uts     — hostname and domain name
```

### Exercise 1.2: Create a new UTS namespace (hostname isolation)

```bash
# Current hostname
hostname

# Create a new UTS namespace and get a shell inside it
sudo unshare --uts bash

# Change the hostname inside the namespace
hostname container-test
hostname       # Shows "container-test"

# Open ANOTHER terminal and check
hostname       # Still shows your original hostname!

# Exit the namespace
exit
hostname       # Back to original
```

**What just happened:** The `unshare --uts` created a new UTS namespace. Changing the hostname inside didn't affect the host.

### Exercise 1.3: Create a new PID namespace

```bash
# Create a PID namespace (processes inside can't see outside processes)
sudo unshare --pid --mount-proc --fork bash

# Inside the namespace
ps aux
# You should see very few processes — maybe just bash and ps
# PID 1 is YOUR bash shell (not systemd)

echo "My PID is $$"    # Should be 1 or very low

exit
```

### Exercise 1.4: Create a new network namespace

```bash
# Create a named network namespace
sudo ip netns add testns

# List network namespaces
sudo ip netns list

# Run a command inside the namespace
sudo ip netns exec testns ip addr
# Only shows "lo" (loopback) — completely isolated network stack

# Compare with the host
ip addr
# Shows all your real interfaces

# Clean up
sudo ip netns del testns
```

### Exercise 1.5: Combine multiple namespaces

```bash
# Create an environment isolated in multiple ways
sudo unshare --uts --pid --mount-proc --net --fork bash

# Check isolation:
hostname          # Can be changed without affecting host
ps aux            # Only sees processes in this namespace
ip addr           # Only loopback, no real network

exit
```

---

## Part 2 — Cgroups (Control Groups)

Cgroups limit HOW MUCH of a resource a process can use.

### Exercise 2.1: Explore existing cgroups

```bash
# Check cgroup version
stat -fc %T /sys/fs/cgroup/
# "cgroup2fs" = cgroup v2
# "tmpfs" = cgroup v1

# List cgroup controllers available
cat /sys/fs/cgroup/cgroup.controllers

# See current resource usage
cat /proc/$$/cgroup
```

### Exercise 2.2: Create a cgroup and limit memory (cgroup v2)

```bash
# Create a cgroup
sudo mkdir -p /sys/fs/cgroup/lab-cgroup

# Set a memory limit (50MB)
echo "50M" | sudo tee /sys/fs/cgroup/lab-cgroup/memory.max

# Check the limit
cat /sys/fs/cgroup/lab-cgroup/memory.max

# Add the current shell to this cgroup
echo $$ | sudo tee /sys/fs/cgroup/lab-cgroup/cgroup.procs

# Check that we're in the cgroup
cat /proc/$$/cgroup

# Now any process started from this shell is limited to 50MB
# Try a memory-heavy command — it will be killed if it exceeds 50MB

# Check memory usage of the cgroup
cat /sys/fs/cgroup/lab-cgroup/memory.current
```

### Exercise 2.3: Limit CPU usage

```bash
# Create a CPU-limited cgroup
sudo mkdir -p /sys/fs/cgroup/lab-cpu

# Limit to 50% of one CPU core
# Format: MAX PERIOD (microseconds)
echo "50000 100000" | sudo tee /sys/fs/cgroup/lab-cpu/cpu.max
# 50000/100000 = 50% of one core

# Verify
cat /sys/fs/cgroup/lab-cpu/cpu.max

# Start a CPU-heavy process in this cgroup
sudo bash -c "echo $$ > /sys/fs/cgroup/lab-cpu/cgroup.procs && stress --cpu 1 --timeout 15" &

# In another terminal, watch it — should stay around 50% of one core
top
```

### Exercise 2.4: Clean up cgroups

```bash
# Kill any processes in the cgroup first
# Then remove the cgroup directory
sudo rmdir /sys/fs/cgroup/lab-cgroup 2>/dev/null
sudo rmdir /sys/fs/cgroup/lab-cpu 2>/dev/null
```

---

## Part 3 — chroot (Change Root)

`chroot` changes the root filesystem for a process — an early form of filesystem isolation.

### Exercise 3.1: Create a minimal chroot environment

```bash
mkdir -p ~/lab-14/myroot/{bin,lib,lib64,proc,sys,etc}

# Copy bash and its dependencies into the chroot
cp /bin/bash ~/lab-14/myroot/bin/
cp /bin/ls ~/lab-14/myroot/bin/
cp /bin/cat ~/lab-14/myroot/bin/
cp /bin/echo ~/lab-14/myroot/bin/

# Copy required libraries (find them with ldd)
ldd /bin/bash | grep -o '/lib[^ ]*' | while read lib; do
    dir=$(dirname "$lib")
    mkdir -p ~/lab-14/myroot"$dir"
    cp "$lib" ~/lab-14/myroot"$lib"
done

ldd /bin/ls | grep -o '/lib[^ ]*' | while read lib; do
    dir=$(dirname "$lib")
    mkdir -p ~/lab-14/myroot"$dir"
    cp "$lib" ~/lab-14/myroot"$lib"
done

# Enter the chroot
sudo chroot ~/lab-14/myroot /bin/bash

# Inside the chroot:
ls /           # Only sees the minimal filesystem we created
echo "I'm in a chroot!"
ls /bin        # Only bash, ls, cat, echo

# Try to access host files
ls /home       # Empty — host /home is not visible
cat /etc/passwd  # Won't work — no passwd file in chroot

exit
```

**Key insight:** `chroot` changes the filesystem root but does NOT provide true isolation. The process still shares the host kernel, network, PIDs, etc. Containers go much further.

---

## Part 4 — Putting It Together: Building a "Container" Manually

### Exercise 4.1: A manual container (namespace + cgroup + chroot)

```bash
# This combines everything:
# 1. New namespaces (PID, UTS, network)
# 2. New root filesystem (chroot)
# 3. Resource limits (cgroup)

# Create isolated environment
sudo unshare --uts --pid --fork --mount-proc bash -c '
    hostname my-container
    echo "=== Manual Container ==="
    echo "Hostname: $(hostname)"
    echo "PID: $$"
    echo "Processes:"
    ps aux
    echo ""
    echo "This is what Docker does under the hood."
    echo "(plus a LOT more — image layers, networking, volumes, etc.)"
'
```

---

## Part 5 — Docker Comparison (If Docker Is Installed)

### Exercise 5.1: See Docker using namespaces and cgroups

```bash
# Skip this section if Docker is not installed

# Run a container
docker run -d --name test-container --memory=64m --cpus=0.5 alpine sleep 1000

# Find the container's PID on the host
PID=$(docker inspect --format '{{.State.Pid}}' test-container)
echo "Container PID on host: $PID"

# See its namespaces
sudo ls -la /proc/$PID/ns/

# See its cgroup
cat /proc/$PID/cgroup

# See it in the process list (it's just a process!)
ps aux | grep "sleep 1000"

# See its resource limits
# docker stats test-container --no-stream

# Clean up
docker rm -f test-container
```

### Exercise 5.2: Enter a running container's namespaces

```bash
# Skip if Docker is not installed

docker run -d --name demo alpine sleep 1000
PID=$(docker inspect --format '{{.State.Pid}}' demo)

# Enter the container's namespace using nsenter
sudo nsenter -t $PID -m -u -i -n -p -- /bin/sh

# You're now inside the container's namespaces!
hostname
ps aux
ip addr

exit
docker rm -f demo
```

---

## Part 6 — Key Concepts Summary

### Exercise 6.1: Container = Namespaces + Cgroups + Filesystem

```
┌──────────────────────────────────────────────────┐
│                   CONTAINER                       │
│                                                   │
│  ┌─────────────┐  ┌──────────────┐               │
│  │ Namespaces   │  │   Cgroups     │              │
│  │              │  │              │               │
│  │ PID:  own 1  │  │ CPU:   50%   │               │
│  │ NET:  own IP │  │ MEM:   256MB │               │
│  │ MNT:  own /  │  │ I/O:   limit │               │
│  │ UTS:  own    │  │              │               │
│  │   hostname   │  │              │               │
│  └─────────────┘  └──────────────┘               │
│                                                   │
│  ┌──────────────────────────────────┐             │
│  │     Root Filesystem (Image)      │             │
│  │   /bin  /etc  /usr  /var  /app   │             │
│  └──────────────────────────────────┘             │
│                                                   │
│  Running on the HOST KERNEL (shared)              │
└──────────────────────────────────────────────────┘
```

Test your understanding:

```bash
# 1. What provides process isolation?   → PID namespace
# 2. What provides network isolation?   → NET namespace
# 3. What limits CPU usage?             → cgroups (cpu controller)
# 4. What limits memory?                → cgroups (memory controller)
# 5. What provides filesystem isolation?→ MNT namespace + chroot/pivot_root
# 6. Do containers have their own kernel? → NO — they share the host kernel
```

---

## Challenges

1. **Create a fully isolated "container"** manually:
   - New PID, UTS, NET, MNT namespaces
   - Custom hostname
   - Memory limited to 100MB via cgroup
   - A minimal rootfs with bash and a few commands

2. **Compare namespace IDs** between two Docker containers and the host to prove they're isolated

3. **Write a script** that shows all namespaces for a given PID and what type each namespace is

4. **Explore `/proc`** — for any running process, examine:
   - `/proc/<PID>/ns/` — namespaces
   - `/proc/<PID>/cgroup` — cgroup membership
   - `/proc/<PID>/root` — root filesystem
   - `/proc/<PID>/status` — process details

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Thinking containers are VMs | Containers share the host kernel — much lighter | Understand: namespace + cgroup + rootfs = container |
| Forgetting `--fork` with `unshare --pid` | PID namespace doesn't work correctly without fork | Always use `--fork` with `--pid` |
| Not mounting `/proc` in PID namespace | `ps` won't work correctly | Use `--mount-proc` with `unshare` |
| Deleting cgroup dir with processes in it | Will fail | Move or kill processes first |
| Thinking `chroot` = security | chroot is trivially escapable by root | Use namespaces for real isolation |

---

## Cleanup

```bash
sudo ip netns del testns 2>/dev/null
sudo rmdir /sys/fs/cgroup/lab-cgroup 2>/dev/null
sudo rmdir /sys/fs/cgroup/lab-cpu 2>/dev/null
rm -rf ~/lab-14
killall stress 2>/dev/null
```

---

## What's Next?

You've completed all 14 hands-on labs! You now have practical experience with every major Linux subsystem.

**Recommended next steps:**
1. Build a project from `04-projects/`
2. Complete the OverTheWire Bandit challenges
3. Review `05-interview-prep/`
4. Move on to **Docker and Kubernetes** — you now understand the Linux foundations they build on
