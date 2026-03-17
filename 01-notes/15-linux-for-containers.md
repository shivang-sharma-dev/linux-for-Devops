# 15 — Linux for Containers

> Containers are not magic. They are Linux processes using three kernel features: namespaces, cgroups, and union filesystems. Understanding this makes you a better Docker and Kubernetes engineer.

---

## The Big Picture

```
What most people think containers are:
  A container = a tiny virtual machine

What containers actually are:
  A container = a regular Linux process with:
    1. Namespaces  → isolated view of the system
    2. cgroups     → limited resources
    3. Union FS    → layered filesystem (images)

There is NO hypervisor. No separate kernel.
The container shares YOUR kernel.
```

This is why containers start in milliseconds (no OS boot) and why a container exploiting a kernel vulnerability can affect the host.

---

## Namespaces — The Isolation Layer

A namespace wraps a global resource so that processes inside see their own isolated instance.

```
Without namespaces:                With namespaces:
  Host                              Host kernel
  ├── process 1 (PID 1, systemd)    ├── container A
  ├── process 2 (PID 200, nginx)    │   ├── PID 1 (nginx) ← thinks it's PID 1
  ├── process 3 (PID 201, python)   │   └── /etc/hosts ← its own hosts file
  └── /etc/hosts (shared)           └── container B
                                        ├── PID 1 (python)
                                        └── eth0 ← its own network interface
```

### The 7 namespace types Linux provides

| Namespace | Isolates | What the container sees |
|---|---|---|
| `pid` | Process IDs | Its own PID 1, can't see host processes |
| `net` | Network | Its own interfaces, routing, firewall rules |
| `mnt` | Filesystem mounts | Its own root filesystem |
| `uts` | Hostname | Its own hostname |
| `ipc` | IPC (shared memory, semaphores) | Isolated IPC resources |
| `user` | User/Group IDs | UID 0 inside maps to unprivileged UID on host |
| `cgroup` | cgroup root | Its own cgroup hierarchy view |

```bash
# See namespaces of a running process
ls -la /proc/1234/ns/

# See all namespaces on the system
lsns

# Run a command in a new namespace (like Docker does)
sudo unshare --pid --fork --mount-proc /bin/bash
# Now you're in an isolated PID namespace — ps aux only shows your processes
ps aux   # only sees processes in this namespace
exit

# See which namespace a process is in
sudo lsns -p 1234
```

---

## cgroups — The Resource Limiter

Control Groups (cgroups) limit, account for, and isolate resource usage of processes.

```
Without cgroups:                  With cgroups:
  Any process can use all CPU       container A gets max 2 CPUs
  Any process can eat all RAM       container B gets max 512MB RAM
  → one bad process = whole         → noisy neighbour is contained
    server impacted
```

### cgroups v2 (modern Linux)

```bash
# See the cgroup hierarchy
ls /sys/fs/cgroup/

# See which cgroup a process is in
cat /proc/1234/cgroup

# Docker creates cgroups under:
ls /sys/fs/cgroup/system.slice/docker-*/

# See resource limits for a container
cat /sys/fs/cgroup/system.slice/docker-abc123.scope/memory.max
cat /sys/fs/cgroup/system.slice/docker-abc123.scope/cpu.max

# Manually create a cgroup and limit resources
sudo mkdir /sys/fs/cgroup/mygroup
echo "512M" | sudo tee /sys/fs/cgroup/mygroup/memory.max
echo "200000 1000000" | sudo tee /sys/fs/cgroup/mygroup/cpu.max
# (200000/1000000 = 20% of one CPU)

# Put a process into the cgroup
echo 1234 | sudo tee /sys/fs/cgroup/mygroup/cgroup.procs
```

---

## Union Filesystems — How Images Work

Container images are built in layers. Each Dockerfile instruction creates a new layer.

```
Image layers (read-only):
  Layer 3: COPY app.py /app/        ← your application code
  Layer 2: RUN pip install flask    ← dependencies
  Layer 1: FROM python:3.11-slim    ← base OS

Container layer (read-write):
  Layer 4: writable layer           ← changes made while container runs
  (deleted when container is removed)

Union mount: layers are merged → container sees one unified filesystem
```

```bash
# See Docker image layers
docker history nginx

# Docker stores layers here on the host
sudo ls /var/lib/docker/overlay2/

# Each layer is a directory
sudo ls /var/lib/docker/overlay2/abc123/
# diff/    ← the actual layer contents
# merged/  ← the union mount (what container sees)
# work/    ← overlay2 working directory

# overlay2 mount explained
# mount -t overlay overlay -o \
#   lowerdir=layer1:layer2:layer3,  ← base layers (read-only)
#   upperdir=writable,              ← container writes go here
#   workdir=work                    ← temp work dir
#   /merged                         ← what the container mounts as /
```

---

## Exploring Linux Container Primitives Manually

You can create a basic container by hand — no Docker needed. This shows what Docker does under the hood:

```bash
# Step 1: Create a minimal root filesystem
mkdir -p /tmp/mycontainer
sudo debootstrap --variant=minbase focal /tmp/mycontainer

# Step 2: Enter with isolated namespaces (like Docker run)
sudo unshare \
  --pid \
  --fork \
  --net \
  --mount \
  --uts \
  --ipc \
  --mount-proc=/tmp/mycontainer/proc \
  chroot /tmp/mycontainer \
  /bin/bash

# Inside this "container":
ps aux          # only sees this shell (PID namespace isolation)
hostname        # different from host
ip a            # different network interfaces (net namespace)
ls /            # different root filesystem (mount namespace)
exit
```

---

## /proc — The Window Into the Kernel

`/proc` is a virtual filesystem that exposes kernel and process data as files.

```bash
cat /proc/cpuinfo                   # CPU details
cat /proc/meminfo                   # memory details
cat /proc/uptime                    # system uptime in seconds
cat /proc/loadavg                   # load average + process counts
cat /proc/net/tcp                   # active TCP connections (hex)
cat /proc/sys/kernel/hostname       # system hostname
cat /proc/sys/net/ipv4/ip_forward   # IP forwarding enabled?

# Per-process
cat /proc/$$/status                 # current shell's status
cat /proc/$$/cmdline | tr '\0' ' '  # current shell's command
ls /proc/$$/fd                      # current shell's open files
cat /proc/$$/maps                   # current shell's memory maps
```

---

## How Docker Uses These Primitives

```
docker run -d --memory=512m --cpus=1.5 nginx

What Docker actually does:
  1. Pulls image layers → stores in /var/lib/docker/overlay2/
  2. Creates overlay2 mount (union of layers + writable layer)
  3. Creates namespaces:
       unshare --pid --net --mnt --uts --ipc
  4. Creates cgroup:
       /sys/fs/cgroup/.../docker-abc123/
       memory.max = 536870912 (512MB)
       cpu.max = 150000 1000000  (1.5 CPUs)
  5. Sets up veth pair (virtual ethernet) for networking
  6. chroot into the container filesystem
  7. Runs nginx as PID 1 of the new PID namespace
```

---

## Capabilities — Fine-Grained Root Powers

Traditional Linux: root can do everything, non-root can't.
Capabilities: split root's powers into ~40 distinct capabilities.

```bash
# Common capabilities
CAP_NET_BIND_SERVICE   # bind to ports < 1024 (without being root)
CAP_SYS_ADMIN          # mount filesystems, change hostname
CAP_NET_ADMIN          # configure network interfaces
CAP_CHOWN              # change file ownership
CAP_KILL               # send signals to any process
CAP_SYS_PTRACE         # use strace/gdb on other processes

# See capabilities of a process
cat /proc/1234/status | grep Cap
capsh --decode=00000000a80425fb    # decode hex capability set

# Docker drops many capabilities by default
# Run with specific capability added:
docker run --cap-add NET_BIND_SERVICE nginx
# Run with all capabilities (avoid):
docker run --privileged nginx
```

---

## seccomp — System Call Filtering

Docker applies a seccomp (secure computing) profile by default — blocks ~44 dangerous system calls.

```bash
# See Docker's default seccomp profile
cat /etc/docker/seccomp-default.json | python3 -m json.tool | head -50

# Run container without seccomp (for debugging only)
docker run --security-opt seccomp=unconfined nginx

# Apply custom seccomp profile
docker run --security-opt seccomp=/path/to/profile.json nginx
```

---

## Key Takeaways for DevOps

```
1. Containers share the host kernel
   → A kernel exploit in a container can affect the host
   → Always keep the host kernel updated

2. "Root in a container" is still dangerous
   → If the container is privileged, root inside = root outside
   → Never run containers as root unless absolutely necessary

3. Resource limits are cgroups
   → OOMKilled in Kubernetes = cgroup memory limit hit
   → CPU throttling = cgroup cpu.max limit hit

4. Container images are just tarballs of filesystem layers
   → docker save, docker load, docker export work with this
   → Layers are cached — this is why rebuilds are fast

5. Network between containers = virtual ethernet + bridge
   → docker0 bridge on host connects containers
   → Each container has a veth pair (one end in container, one on host)
```

---

## Useful Commands for Container Debugging

```bash
# See all namespaces on the system
lsns

# Enter a running container's namespace
sudo nsenter -t $(docker inspect -f '{{.State.Pid}}' container_name) --net --pid bash

# See a container's cgroup limits
docker inspect container_name | grep -A5 Memory

# Check what system calls a container is making
sudo strace -p $(docker inspect -f '{{.State.Pid}}' container_name)

# See container's overlay filesystem
docker inspect container_name | grep MergedDir
sudo ls $(docker inspect -f '{{.GraphDriver.Data.MergedDir}}' container_name)
```
