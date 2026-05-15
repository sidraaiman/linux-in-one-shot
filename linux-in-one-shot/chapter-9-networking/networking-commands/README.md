# Networking Commands in Linux

## Introduction

Networking is one of the most critical skills for Linux administration. Whether you are configuring a server, debugging connectivity issues, downloading files, or exposing a service — you need to understand how Linux interacts with the network. This chapter covers the essential commands for inspecting, testing, and using network connections from the command line.

---

## 1. Network Interface Inspection

### `ip` — Network Interface and Routing Tool (Modern)

`ip` is the standard tool for all network configuration on modern Linux. It replaces the deprecated `ifconfig`, `route`, and `arp` commands.

```bash
# Show all network interfaces and their IP addresses
ip a
ip addr show

# Show a specific interface
ip a show eth0

# Brief one-line summary per interface
ip -br a

# Show only IPv4 addresses
ip -4 a

# Show only IPv6 addresses
ip -6 a

# Show routing table
ip r
ip route show

# Show default gateway
ip r | grep default

# Show ARP table (MAC address mappings)
ip neigh

# Bring an interface up or down
ip link set eth0 up
ip link set eth0 down

# Show interface link status (UP/DOWN, speed, MTU)
ip link show
ip link show eth0
```

Sample `ip a` output:
```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/24 brd 172.17.0.255 scope global eth0
       valid_lft forever preferred_lft forever
```

| Field | Meaning |
|-------|---------|
| `eth0` | Interface name |
| `UP` | Interface is active |
| `mtu 1500` | Maximum Transmission Unit — largest packet size in bytes |
| `link/ether` | MAC (hardware) address |
| `inet` | IPv4 address with subnet mask in CIDR notation |
| `inet6` | IPv6 address |

### `ifconfig` — Legacy Interface Tool (Deprecated)

```bash
ifconfig               # Show all active interfaces
ifconfig eth0          # Show specific interface
ifconfig eth0 up       # Bring interface up
ifconfig eth0 down     # Bring interface down
```

> `ifconfig` is deprecated and no longer installed by default on many distros. It is part of the `net-tools` package. Use `ip a` instead — it provides more information and is actively maintained.

---

## 2. Connectivity Testing

### `ping` — Test Network Connectivity

`ping` sends ICMP echo requests and measures round-trip time. It is the first tool to reach for when diagnosing connectivity.

```bash
ping google.com              # Continuous ping (Ctrl+C to stop)
ping -c 4 google.com         # Send exactly 4 packets then stop
ping -c 4 -i 0.5 google.com  # Ping every 0.5 seconds
ping -t 64 google.com        # Set TTL (Time To Live)
ping -s 1400 google.com      # Test with large packet (MTU check)
ping 8.8.8.8                 # Ping by IP (bypasses DNS)
```

Sample output:
```
PING google.com (142.250.64.110) 56(84) bytes of data.
64 bytes from lga34s32-in-f14.1e100.net: icmp_seq=1 ttl=117 time=12.3 ms
64 bytes from lga34s32-in-f14.1e100.net: icmp_seq=2 ttl=117 time=11.8 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 11.8/12.0/12.3/0.2 ms
```

| Field | Meaning |
|-------|---------|
| `icmp_seq` | Sequence number — gaps indicate dropped packets |
| `ttl` | Time To Live — decrements at each router hop |
| `time` | Round-trip time in milliseconds |
| `packet loss` | Any loss indicates network problems |
| `rtt min/avg/max` | Latency range — spikes indicate congestion |

**Diagnosing with ping:**
```bash
# Step 1: Can you reach your own machine?
ping 127.0.0.1              # Loopback — always works if TCP/IP stack is up

# Step 2: Can you reach your gateway?
ip r | grep default         # Find gateway IP
ping 192.168.1.1            # Ping the gateway

# Step 3: Can you reach the internet by IP? (bypasses DNS)
ping 8.8.8.8

# Step 4: Can DNS resolve? (if step 3 works but this fails, it's a DNS issue)
ping google.com
```

### `traceroute` — Trace Network Path

Shows every router (hop) between your machine and the destination:

```bash
# Install
apt install traceroute        # Debian/Ubuntu
dnf install traceroute        # RHEL/Fedora

traceroute google.com         # Trace to destination
traceroute -n google.com      # Skip DNS resolution (faster)
traceroute -m 20 google.com   # Limit to 20 hops
```

Sample output:
```
traceroute to google.com (142.250.64.110), 30 hops max
 1  192.168.1.1      1.2 ms     # Your gateway/router
 2  10.0.0.1         5.4 ms     # ISP router
 3  * * *                       # Router didn't respond (normal)
 4  72.14.215.165    10.1 ms    # Google network
 5  142.250.64.110   12.3 ms    # Destination
```

`* * *` means a hop didn't respond to ICMP — this is common on the internet and doesn't always indicate a problem.

### `mtr` — Continuous Traceroute

Combines `ping` and `traceroute` into a live, refreshing view:

```bash
apt install mtr
mtr google.com              # Interactive (Ctrl+C to quit)
mtr --report google.com     # Run 10 cycles and print report
mtr -n google.com           # No DNS resolution
```

---

## 3. DNS Resolution

### `nslookup` — Basic DNS Query

```bash
nslookup google.com              # Resolve hostname to IP
nslookup 8.8.8.8                 # Reverse lookup — IP to hostname
nslookup google.com 8.8.8.8      # Query using specific DNS server
```

### `dig` — Detailed DNS Lookup

```bash
# Install
apt install dnsutils

dig google.com                   # Full DNS query output
dig +short google.com            # Just the IP address
dig google.com A                 # Get IPv4 address record
dig google.com AAAA              # Get IPv6 address record
dig google.com MX                # Get mail server records
dig google.com NS                # Get nameserver records
dig google.com TXT               # Get TXT records (SPF, DKIM)
dig @8.8.8.8 google.com          # Query against Google's DNS
dig -x 8.8.8.8                   # Reverse DNS lookup
```

Sample `dig` output:
```
;; ANSWER SECTION:
google.com.      299    IN    A    142.250.64.110

;; Query time: 12 msec
;; SERVER: 192.168.1.1#53
```

| Field | Meaning |
|-------|---------|
| `299` | TTL — how many seconds to cache this result |
| `IN` | Internet class |
| `A` | Record type (A = IPv4, AAAA = IPv6, MX = mail, etc.) |
| `SERVER` | DNS server that answered the query |

### DNS Configuration Files

```bash
# Which DNS servers is the system using?
cat /etc/resolv.conf

# Static hostname mappings (checked before DNS)
cat /etc/hosts

# DNS lookup order (files vs DNS)
cat /etc/nsswitch.conf | grep hosts
```

---

## 4. Fetching Data from the Network

### `curl` — Transfer Data to/from URLs

`curl` is the Swiss Army knife of network data transfer. It supports HTTP, HTTPS, FTP, SFTP, and many other protocols.

```bash
# Fetch a webpage
curl https://example.com

# Fetch and save to a file
curl -o output.html https://example.com

# Download file keeping the remote filename
curl -O https://example.com/file.zip

# Follow redirects (HTTP 301/302)
curl -L https://example.com

# Show only HTTP response headers
curl -I https://example.com

# Include headers in output
curl -i https://example.com

# Verbose — show full request and response details
curl -v https://example.com

# Send a GET request with headers
curl -H "Authorization: Bearer token123" https://api.example.com/data

# Send a POST request with JSON body
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name": "alice", "role": "admin"}' \
  https://api.example.com/users

# Send a POST with form data
curl -X POST -d "username=alice&password=secret" https://example.com/login

# Upload a file
curl -F "file=@/path/to/file.txt" https://example.com/upload

# Download with progress bar
curl -# -O https://example.com/largefile.zip

# Limit download speed
curl --limit-rate 1M -O https://example.com/file.zip

# Set timeout
curl --connect-timeout 10 --max-time 30 https://example.com

# Test HTTP status code only
curl -o /dev/null -s -w "%{http_code}\n" https://example.com

# Skip SSL certificate verification (use only for testing)
curl -k https://self-signed.example.com
```

### `wget` — File Downloader

`wget` is purpose-built for downloading files, with better support for resuming interrupted downloads and recursive downloads.

```bash
# Basic download
wget https://example.com/file.zip

# Download to a specific filename
wget -O myfile.zip https://example.com/file.zip

# Resume an interrupted download
wget -c https://example.com/largefile.zip

# Download in the background
wget -b https://example.com/largefile.zip

# Limit download speed
wget --limit-rate=1m https://example.com/file.zip

# Download multiple files from a list
wget -i urls.txt

# Mirror an entire website
wget --mirror --convert-links --no-parent https://example.com

# Retry on failure
wget --tries=3 https://example.com/file.zip

# Download quietly (no output)
wget -q https://example.com/file.zip

# Custom user agent
wget --user-agent="Mozilla/5.0" https://example.com

# Skip SSL certificate verification (testing only)
wget --no-check-certificate https://example.com
```

### `curl` vs `wget`

| Feature | `curl` | `wget` |
|---------|--------|--------|
| Resume downloads | `--continue-at -` | `-c` (simpler) |
| Recursive download | No | Yes (`--mirror`) |
| API testing (POST, headers) | Excellent | Limited |
| Follow redirects | `-L` | Automatic |
| Multiple protocols | HTTP, FTP, SFTP, SMTP... | HTTP, HTTPS, FTP |
| Scripting/piping | Better | Good |

> Use `wget` for straightforward file downloads. Use `curl` for API calls, custom headers, POST requests, and protocol flexibility.

---

## 5. Open Ports and Connections

### `netstat` — Network Connections (Legacy)

```bash
apt install net-tools

netstat -tulnp           # TCP/UDP listening ports with process names
netstat -anp             # All connections with PIDs
netstat -s               # Protocol statistics summary
netstat -r               # Routing table
```

### `ss` — Socket Statistics (Modern)

```bash
ss -tulnp                # Listening TCP/UDP with process info
ss -tnp                  # Established TCP connections
ss -anp                  # All sockets
ss -s                    # Summary
ss -tulnp | grep :80     # Find what's using port 80
ss -tulnp | grep nginx   # Find ports used by nginx
```

`ss` flags:

| Flag | Meaning |
|------|---------|
| `-t` | TCP sockets |
| `-u` | UDP sockets |
| `-l` | Listening sockets only |
| `-n` | Numeric (no DNS resolution) |
| `-p` | Show process name and PID |
| `-a` | All sockets (listening + established) |

### `lsof` — List Open Files/Ports by Process

```bash
# What is using port 80?
lsof -i :80

# What ports is nginx using?
lsof -p $(pidof nginx)

# All network connections
lsof -i

# All TCP connections
lsof -i TCP
```

---

## 6. SSH — Secure Shell

SSH is the standard protocol for remote access to Linux servers.

```bash
# Connect to a remote server
ssh username@hostname
ssh username@192.168.1.100
ssh -p 2222 username@hostname       # Custom port

# Connect with SSH key
ssh -i ~/.ssh/mykey.pem username@hostname

# Run a command remotely without opening a shell
ssh username@hostname "ls -la /var/www"
ssh username@hostname "systemctl status nginx"

# Copy files securely
scp file.txt username@hostname:/remote/path/
scp username@hostname:/remote/file.txt ./local/

# Copy directory recursively
scp -r localdir/ username@hostname:/remote/path/

# Generate SSH key pair
ssh-keygen -t ed25519 -C "your@email.com"
ssh-keygen -t rsa -b 4096

# Copy public key to server (enables passwordless login)
ssh-copy-id username@hostname
ssh-copy-id -i ~/.ssh/mykey.pub username@hostname

# SSH tunneling — forward local port 8080 to remote port 80
ssh -L 8080:localhost:80 username@hostname

# SSH config file — store connection aliases
cat ~/.ssh/config
```

Example `~/.ssh/config`:
```
Host myserver
    HostName 192.168.1.100
    User alice
    Port 2222
    IdentityFile ~/.ssh/mykey.pem

# Then connect with just:
# ssh myserver
```

---

## 7. Temporary Network Configuration

These changes are not persistent — they reset on reboot. For permanent config, use the distro's network manager.

```bash
# Assign an IP address to an interface
sudo ip addr add 192.168.1.50/24 dev eth0

# Remove an IP address
sudo ip addr del 192.168.1.50/24 dev eth0

# Add a default gateway
sudo ip route add default via 192.168.1.1

# Add a static route
sudo ip route add 10.0.0.0/8 via 192.168.1.1

# Flush all IP addresses on an interface
sudo ip addr flush dev eth0
```

---

## 8. Firewall — `ufw` (Uncomplicated Firewall)

`ufw` is the user-friendly interface to `iptables`, default on Ubuntu:

```bash
# Check status
ufw status
ufw status verbose

# Enable/disable firewall
ufw enable
ufw disable

# Allow a port
ufw allow 22            # Allow SSH
ufw allow 80            # Allow HTTP
ufw allow 443           # Allow HTTPS
ufw allow 8080/tcp      # Allow specific port and protocol

# Allow by service name
ufw allow ssh
ufw allow http
ufw allow https

# Deny a port
ufw deny 23             # Block Telnet

# Delete a rule
ufw delete allow 80

# Allow from a specific IP
ufw allow from 192.168.1.100
ufw allow from 192.168.1.0/24 to any port 22

# Reset all rules
ufw reset
```

---

## 9. Common Networking Scenarios

### Check if a Port is Open on a Remote Host

```bash
# Using nc (netcat)
nc -zv hostname 80
nc -zv hostname 22

# Using curl
curl -v telnet://hostname:80

# Using /dev/tcp (bash built-in)
timeout 3 bash -c 'cat < /dev/null > /dev/tcp/hostname/80' && echo "open" || echo "closed"
```

### Find Your Public IP Address

```bash
curl ifconfig.me
curl icanhazip.com
curl api.ipify.org
```

### Find Your Local/Private IP

```bash
hostname -I
ip -br a | grep UP
```

### Test DNS Resolution Speed

```bash
time dig google.com
time nslookup google.com
```

### Download and Run a Script

```bash
# Pipe directly to bash (verify source before doing this)
curl -fsSL https://get.docker.com | bash

# Safer — download first, inspect, then run
curl -fsSL https://get.docker.com -o install-docker.sh
cat install-docker.sh         # Review it
bash install-docker.sh
```

---

## Summary

| Task | Command |
|------|---------|
| Show IP addresses | `ip a` |
| Show routing table | `ip r` |
| Test connectivity | `ping google.com` |
| Trace network path | `traceroute google.com` |
| DNS lookup | `dig +short google.com` |
| Fetch URL / test API | `curl https://example.com` |
| Download a file | `wget https://example.com/file.zip` |
| Resume download | `wget -c https://example.com/file.zip` |
| Show listening ports | `ss -tulnp` |
| What's on port 80? | `lsof -i :80` |
| SSH to server | `ssh user@host` |
| Copy file to server | `scp file.txt user@host:/path/` |
| My public IP | `curl ifconfig.me` |
| Check port on remote | `nc -zv host 80` |
| Allow port in firewall | `ufw allow 80` |

---

**← Back to [Chapter 8: System Monitoring](../../chapter-8-system-monitoring/monitoring/README.md)**  
**Next → Chapter 10: Shell Scripting**
