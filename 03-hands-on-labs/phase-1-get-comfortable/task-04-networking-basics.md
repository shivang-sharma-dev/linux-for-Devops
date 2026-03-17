# Task 04 — Networking Basics

> **Prerequisite:** Read `01-notes/05-networking.md`
> **Environment:** Any Linux terminal (native, WSL2, or VM). Some exercises need internet access.
> **Difficulty:** Beginner

---

## Objective

By the end of this lab you will be able to:
- Check your IP address and network interfaces
- Test connectivity with `ping` and `curl`
- Check which ports are open and what's listening on them
- Perform DNS lookups
- Download files from the command line
- Read basic network information and troubleshoot connectivity

---

## Part 1 — Your Network Interfaces

### Exercise 1.1: Check your IP address

```bash
# Modern way (recommended)
ip addr show
# or shorter:
ip a

# Old way (deprecated but still common)
ifconfig          # May need: sudo apt install net-tools
```

Identify from the output:
- `lo` — loopback interface (127.0.0.1, always present)
- `eth0` or `enp0s3` or `wlan0` — your actual network interface
- `inet` line — your IPv4 address
- `inet6` line — your IPv6 address

### Exercise 1.2: Check your gateway and routes

```bash
# Default gateway (your router)
ip route show
# or
ip r

# The "default via x.x.x.x" line is your gateway

# Check public IP (what the internet sees)
curl -s ifconfig.me
# or
curl -s icanhazip.com
```

### Exercise 1.3: DNS configuration

```bash
# Check your DNS servers
cat /etc/resolv.conf

# Check hostname
hostname
hostnamectl
```

---

## Part 2 — Testing Connectivity

### Exercise 2.1: Ping

```bash
# Ping a website (Ctrl+C to stop)
ping -c 4 google.com        # Send 4 packets

# Ping by IP address
ping -c 4 8.8.8.8           # Google's DNS

# Ping your gateway
ping -c 4 $(ip route | grep default | awk '{print $3}')

# Ping localhost
ping -c 2 127.0.0.1
```

**Reading ping output:**
```
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=12.3 ms
```
- `time=12.3 ms` — round-trip time (lower is better)
- `ttl=118` — time-to-live (how many hops before it was dropped)

### Exercise 2.2: Traceroute

```bash
# Install if needed
sudo apt install traceroute -y    # Debian/Ubuntu

# Trace the path to a server
traceroute google.com

# Shows each network hop between you and the destination
# Useful for finding WHERE a connection is failing
```

### Exercise 2.3: Test specific ports with `curl` and `nc`

```bash
# Test if a web server is responding
curl -I https://google.com        # Just headers (-I)
curl -s https://httpbin.org/ip    # Your public IP as seen by a server

# Test if a specific port is open (using netcat)
nc -zv google.com 443             # Test HTTPS port
nc -zv google.com 80              # Test HTTP port
# -z = scan mode, -v = verbose
```

---

## Part 3 — Ports and Listening Services

### Exercise 3.1: What's listening on your system?

```bash
# Show all listening ports (modern way)
sudo ss -tulnp

# Breakdown of flags:
# -t = TCP, -u = UDP, -l = listening, -n = numeric ports, -p = show process
```

**Reading the output:**

```
State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process
LISTEN  0       128     0.0.0.0:22          0.0.0.0:*          users:(("sshd",pid=821))
```

- Port 22 = SSH server
- `0.0.0.0` = listening on all interfaces
- `127.0.0.1` = only accepting local connections

### Exercise 3.2: Common ports to know

| Port | Service | Protocol |
|------|---------|----------|
| 22 | SSH | TCP |
| 80 | HTTP | TCP |
| 443 | HTTPS | TCP |
| 53 | DNS | TCP/UDP |
| 3306 | MySQL | TCP |
| 5432 | PostgreSQL | TCP |
| 6379 | Redis | TCP |
| 8080 | HTTP (alt) | TCP |
| 27017 | MongoDB | TCP |

```bash
# Check if a specific port is in use
sudo ss -tulnp | grep :80
sudo ss -tulnp | grep :22
```

### Exercise 3.3: Find what process is using a port

```bash
# What's on port 22?
sudo lsof -i :22

# What ports is a specific process using?
sudo lsof -i -p $(pgrep sshd | head -1)
```

---

## Part 4 — DNS Lookups

### Exercise 4.1: Resolve domain names

```bash
# Basic DNS lookup
nslookup google.com

# More detailed DNS query
dig google.com

# Short answer only
dig +short google.com

# Look up a specific record type
dig google.com MX          # Mail servers
dig google.com NS          # Name servers
dig google.com TXT         # TXT records

# Reverse DNS lookup (IP → hostname)
dig -x 8.8.8.8

# Use a specific DNS server
dig @8.8.8.8 google.com    # Query Google's DNS directly
```

### Exercise 4.2: The hosts file

```bash
# View the hosts file
cat /etc/hosts

# Add a custom hostname (requires sudo)
# WARNING: This overrides DNS for this hostname
sudo bash -c 'echo "127.0.0.1 myapp.local" >> /etc/hosts'

# Test it
ping -c 2 myapp.local      # Should resolve to 127.0.0.1

# Remove it when done
sudo sed -i '/myapp.local/d' /etc/hosts
```

---

## Part 5 — Downloading Files

### Exercise 5.1: curl

```bash
# Download and display content
curl https://httpbin.org/get

# Save to file
curl -o test.html https://example.com

# Follow redirects
curl -L -o page.html https://github.com

# Download silently (no progress bar)
curl -sL https://httpbin.org/ip

# Send a POST request
curl -X POST -d "name=test" https://httpbin.org/post

# Set custom headers
curl -H "Content-Type: application/json" https://httpbin.org/headers
```

### Exercise 5.2: wget

```bash
# Download a file
wget https://example.com/index.html

# Download and save with a different name
wget -O mypage.html https://example.com

# Download in the background
wget -b https://example.com/large-file.zip

# Resume a failed download
wget -c https://example.com/large-file.zip
```

---

## Part 6 — Practical Scenarios

### Exercise 6.1: "Is the server reachable?"

```bash
# Step 1: Can I reach it at all?
ping -c 3 example.com

# Step 2: Can I reach the specific port?
nc -zv example.com 443

# Step 3: Does the service respond?
curl -I https://example.com

# Step 4: Is DNS working?
dig example.com
```

### Exercise 6.2: "What's using all the bandwidth?"

```bash
# Install nethogs (shows bandwidth per process)
sudo apt install nethogs -y
sudo nethogs

# Or use iftop (shows bandwidth per connection)
sudo apt install iftop -y
sudo iftop
```

---

## Challenges

1. **Find your machine's:**
   - Private IP address
   - Public IP address
   - Default gateway
   - DNS server
   — all in one script

2. **Port scan your own machine:**
   ```bash
   # Scan common ports on localhost
   for port in 22 80 443 3306 5432 8080; do
     nc -zv 127.0.0.1 $port 2>&1
   done
   ```

3. **DNS investigation:**
   - Find all the name servers for `github.com`
   - Find the mail servers (MX records) for `google.com`
   - Do a reverse DNS lookup on `1.1.1.1` — who owns it?

4. **Download and verify:**
   - Download a file using `curl`
   - Check its md5 or sha256 checksum
   - Compare with the expected checksum from the website

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Using `ifconfig` on modern systems | Deprecated, may not be installed | Use `ip addr` instead |
| Pinging without `-c` flag | Runs forever until Ctrl+C | Always use `ping -c 4` in scripts |
| Confusing `ss` flags | Wrong flags give wrong output | `-tulnp` for listening TCP/UDP with process names |
| Forgetting `sudo` with `ss -p` | Won't show process names without root | `sudo ss -tulnp` |
| Not checking DNS first | Assuming network is down when it's a DNS issue | `ping 8.8.8.8` works but `ping google.com` fails → DNS problem |

---

## Cleanup

```bash
rm -f test.html page.html mypage.html index.html
```

---

## What's Next?

Phase 1 complete! Move on to **Phase 2** → `task-05-scripting.md` — Shell scripting.
