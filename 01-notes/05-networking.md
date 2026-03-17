# 05 — Networking (Linux Commands)

> This covers Linux-specific networking commands. The concepts (TCP/IP, DNS, HTTP) are in the networking-for-devops repo. This file is about the tools you use daily on a Linux server.

---

## Checking Network Interfaces

```bash
ip a                            # show all interfaces and their IPs (modern)
ip addr show                    # same, verbose
ip addr show eth0               # specific interface
ifconfig                        # older command (install net-tools if needed)

# Interface states
ip link show                    # show link state (UP/DOWN)
ip link set eth0 up             # bring interface up
ip link set eth0 down           # bring interface down
```

---

## Routing

```bash
ip route                        # show routing table
ip route show                   # same
ip route add 10.0.0.0/24 via 192.168.1.1    # add route
ip route del 10.0.0.0/24                    # delete route
ip route get 8.8.8.8            # which route would be used to reach this IP
route -n                        # older way (net-tools)
```

---

## Checking Open Ports and Connections

```bash
ss -tulnp                       # most useful: TCP+UDP, listening, numeric, process
# -t = TCP, -u = UDP, -l = listening only, -n = numeric, -p = show process

ss -tulnp | grep :80            # who is listening on port 80
ss -anp                         # all connections with process info
ss -s                           # summary statistics
ss -tp                          # established TCP connections with process

netstat -tulnp                  # older (same as ss, needs net-tools)
netstat -an | grep ESTABLISHED  # all established connections
```

---

## DNS Lookups

```bash
nslookup google.com             # basic DNS query
nslookup google.com 8.8.8.8    # query specific DNS server

dig google.com                  # detailed DNS query
dig google.com A                # A record (IPv4)
dig google.com MX               # mail exchange records
dig google.com NS               # nameserver records
dig @8.8.8.8 google.com         # query specific DNS server
dig +short google.com           # just the IP, no extra info
dig -x 8.8.8.8                  # reverse DNS lookup

host google.com                 # simple lookup
getent hosts google.com         # use system resolver (respects /etc/hosts)

cat /etc/resolv.conf            # current DNS server config
cat /etc/hosts                  # local hostname overrides
```

---

## Testing Connectivity

```bash
ping google.com                 # test basic connectivity
ping -c 4 google.com            # send only 4 packets
ping -i 0.5 google.com          # interval 0.5 seconds
ping6 google.com                # IPv6 ping

traceroute google.com           # trace route to destination
tracepath google.com            # similar, no root needed
mtr google.com                  # combined ping + traceroute (live)

# Test if a port is open
telnet google.com 80            # old way (Ctrl+] then quit)
nc -zv google.com 80            # netcat — better
nc -zv 10.0.0.1 22              # test SSH port
nc -zv 10.0.0.1 80 443          # test multiple ports
```

---

## curl — HTTP From the Command Line

```bash
curl https://google.com                  # GET request
curl -I https://google.com               # headers only
curl -v https://google.com               # verbose (full request+response)
curl -s https://api.example.com          # silent (no progress)
curl -o output.html https://example.com  # save to file
curl -L https://example.com             # follow redirects

# POST request
curl -X POST https://api.example.com/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"secret"}'

# With auth
curl -u alice:password https://api.example.com
curl -H "Authorization: Bearer TOKEN" https://api.example.com

# Download file
curl -O https://example.com/file.tar.gz          # save with original name
curl -C - -O https://example.com/large.iso       # resume interrupted download

# Check response time
curl -w "\nTime: %{time_total}s\n" -s -o /dev/null https://example.com

# Follow redirects + show final URL
curl -Ls -o /dev/null -w "%{url_effective}" https://short.url/abc
```

---

## wget — Download Files

```bash
wget https://example.com/file.tar.gz        # download file
wget -O custom-name.tar.gz https://...      # save with different name
wget -c https://example.com/large.iso       # resume download
wget -q https://example.com                 # quiet mode
wget -r -np https://example.com/docs/       # recursive download
```

---

## Firewall — ufw (Ubuntu)

```bash
sudo ufw status                             # check firewall status
sudo ufw enable                             # turn on
sudo ufw disable                            # turn off

sudo ufw allow 22                           # allow SSH
sudo ufw allow 80/tcp                       # allow HTTP
sudo ufw allow 443/tcp                      # allow HTTPS
sudo ufw allow from 10.0.0.0/8             # allow entire subnet
sudo ufw allow from 192.168.1.5 to any port 22  # allow specific IP to SSH

sudo ufw deny 8080                          # block a port
sudo ufw delete allow 80                    # remove a rule
sudo ufw reset                              # reset all rules

sudo ufw status numbered                    # list rules with numbers
sudo ufw delete 3                           # delete rule number 3
```

## Firewall — firewalld (RHEL/CentOS)

```bash
sudo systemctl status firewalld
sudo firewall-cmd --state
sudo firewall-cmd --list-all                # current rules
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload                  # apply permanent rules
```

---

## Network Configuration Files

```bash
# DNS resolution order
cat /etc/nsswitch.conf          # controls: hosts files dns mymachines

# Local DNS overrides
cat /etc/hosts
# 127.0.0.1   localhost
# 192.168.1.10  myserver.local

# DNS servers
cat /etc/resolv.conf
# nameserver 8.8.8.8
# nameserver 8.8.4.4

# Netplan (Ubuntu 18.04+)
cat /etc/netplan/00-installer-config.yaml
sudo netplan apply              # apply netplan changes

# NetworkManager
nmcli connection show           # show connections
nmcli device status             # device status
```

---

## Useful One-Liners

```bash
# My external IP
curl -s ifconfig.me

# All listening ports with process names
sudo ss -tulnp

# Who is connecting to my server right now
ss -tn state established

# Block an IP immediately
sudo iptables -A INPUT -s 1.2.3.4 -j DROP

# Test if remote port is open (no telnet needed)
cat /dev/null > /dev/tcp/google.com/443 && echo "open" || echo "closed"

# Bandwidth usage by process
sudo nethogs eth0               # install: apt install nethogs

# Watch network traffic live
sudo tcpdump -i eth0 port 80    # HTTP traffic
sudo tcpdump -i any -n          # all traffic, numeric IPs
```
