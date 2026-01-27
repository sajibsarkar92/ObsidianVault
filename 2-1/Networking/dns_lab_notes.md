# DNS Networking Lab Notes

## Table of Contents
1. [DNS Server Fundamentals](#dns-server-fundamentals)
2. [Types of DNS Servers](#types-of-dns-servers)
3. [Lab Commands Explained](#lab-commands-explained)
4. [Practical Examples](#practical-examples)

---

## DNS Server Fundamentals

### What is DNS?

DNS (Domain Name System) is like the phonebook of the internet. When you type `www.google.com` into your browser, DNS translates that human-readable domain name into an IP address (like `142.250.185.46`) that computers can understand.

**Analogy**: Think of DNS as a giant address book. Instead of remembering that your friend lives at "123 Main Street, Apartment 4B", you just remember their name "John". DNS does the same thing—it lets you remember "google.com" instead of "142.250.185.46".

### How DNS Resolution Works

```
User → Local DNS Resolver → Root Server → TLD Server → Authoritative Server → IP Address
```

**The Journey of a DNS Query**:

1. **You ask**: "What's the IP for www.google.com?"
2. **Local Resolver**: "Let me check my cache... not there, I'll ask around"
3. **Root Server**: "I don't know the exact address, but ask the .com server"
4. **TLD (.com) Server**: "I don't have it, but google.com's nameserver does"
5. **Authoritative Server**: "Here's the IP: 142.250.185.46"

---

## Types of DNS Servers

### 1. Authoritative DNS Server

**What it does**: This is the "source of truth" for a specific domain. It holds the official records for that domain.

**Analogy**: Think of it as the property registry office. If you want to know who owns a house, you go to the official registry—that's the authoritative source.

**Example**: 
- Google's authoritative nameservers for `google.com` are `ns1.google.com`, `ns2.google.com`, etc.
- These servers say: "Yes, I officially control google.com, and here are its records"

**Key characteristics**:
- Provides definitive answers for domains it controls
- Doesn't query other servers
- Answers with the `aa` (authoritative answer) flag

### 2. Caching DNS Server (Recursive Resolver)

**What it does**: This server doesn't own any domains, but it remembers (caches) answers from previous queries to speed things up.

**Analogy**: Like a librarian who doesn't own books but helps you find them. Once they've looked up a book's location, they remember it for next time to save effort.

**Example**:
- Your ISP's DNS server (like 8.8.8.8 - Google Public DNS)
- First time someone asks for "facebook.com" → it queries the authoritative servers
- Second time → "I remember! Here's the answer from my cache"

**Key characteristics**:
- Stores DNS records temporarily (TTL - Time To Live)
- Reduces query time significantly
- Reduces load on authoritative servers
- Cache expires based on TTL values

**Caching Example**:
```
First Query:
User → Caching Server → [Full DNS resolution process] → 500ms

Second Query (within TTL):
User → Caching Server → [Returns from cache] → 5ms
```

### 3. Forwarding DNS Server

**What it does**: This server doesn't resolve queries itself. Instead, it forwards all queries to another DNS server.

**Analogy**: Like a receptionist who doesn't answer questions directly but forwards your call to someone who can. The receptionist doesn't solve your problem but knows who to ask.

**Example**:
- Your office network DNS might forward all queries to your ISP's DNS
- Small networks use forwarding to centralize DNS management

**Configuration**:
```
Your Computer → Company Forwarder (10.0.0.53) → ISP DNS (8.8.8.8) → Internet
```

**Why use forwarding?**
- Centralized control and monitoring
- Bandwidth efficiency (one central cache instead of many)
- Security filtering (block malicious domains)
- Simplified configuration

---

## Lab Commands Explained

### Image 1 Commands

#### 1. DNS Query Commands

```bash
# dig +t a zhjlab.bd
```
**What it does**: Queries the A record (IPv4 address) for domain `zhjlab.bd`

**Breakdown**:
- `dig` = Domain Information Groper (DNS lookup utility)
- `+t a` = Query type A (IPv4 address record)
- `zhjlab.bd` = The domain to look up

**Other record types**:
- `A` - IPv4 address
- `AAAA` - IPv6 address  
- `MX` - Mail server
- `NS` - Nameserver
- `CNAME` - Canonical name (alias)
- `TXT` - Text records

---

```bash
# dig +t soa zhjlab.bd
```
**What it does**: Queries the SOA (Start of Authority) record

**What is SOA?**: The "birth certificate" of a domain. It contains:
- Primary nameserver
- Administrator email
- Serial number (version of the zone file)
- Refresh, retry, expire timers
- Minimum TTL

**Example Output**:
```
zhjlab.bd.  3600  IN  SOA  ns1.zhjlab.bd. admin.zhjlab.bd. (
                              2024012701 ; Serial
                              3600       ; Refresh
                              1800       ; Retry
                              604800     ; Expire
                              86400 )    ; Minimum TTL
```

---

```bash
# host s00.zhjlab.bd
```
**What it does**: Simple DNS lookup tool (simpler than `dig`)

**Output**: Returns IP address(es) for `s00.zhjlab.bd`

**Example**:
```
s00.zhjlab.bd has address 10.100.68.1
```

---

```bash
# host www.yahoo.com
```
**What it does**: Looks up Yahoo's IP addresses

**Why this command?**: Testing external domain resolution to verify internet DNS works

---

#### 2. File Operations

```bash
# cd /etc
```
**What it does**: Changes directory to `/etc` (configuration files location in Linux)

**Why?**: DNS configuration files are stored in `/etc`, particularly:
- `/etc/hosts` - Local hostname to IP mappings
- `/etc/resolv.conf` - DNS resolver configuration
- `/etc/bind/` - BIND DNS server configuration (if installed)

---

```bash
# ls host → double tab
```
**What it does**: 
- Lists files starting with "host"
- Pressing `tab` twice shows all possible completions

**Why?**: Looking for hostname-related files like:
- `hostname`
- `hosts`
- `host.conf`

---

```bash
# cat host.conf
# cat hosts
```
**What they do**: Display contents of configuration files

**`/etc/host.conf`**: Controls hostname resolution order
```
order hosts,bind
multi on
```
- First check `/etc/hosts`, then DNS
- Allow multiple IP addresses per hostname

**`/etc/hosts`**: Local DNS override file
```
127.0.0.1    localhost
192.168.1.10 myserver.local myserver
```
- Format: `IP_ADDRESS  hostname  [aliases]`
- Checked before DNS queries
- Used for local network or development

**Use case**: If you add `127.0.0.1 facebook.com` to `/etc/hosts`, your computer will never reach Facebook (great for blocking distractions or malware domains!)

---

#### 3. System Administration

```bash
# sudo nano s.hosts
```
**What it does**: Opens file `s.hosts` in nano text editor with admin privileges

**Why sudo?**: System files require root permissions to edit

---

```bash
# cat resolv.conf
```
**What it does**: Shows DNS resolver configuration

**Typical content**:
```
nameserver 8.8.8.8
nameserver 8.8.4.4
search localdomain
```
- `nameserver`: DNS servers to query (in order)
- `search`: Domain suffixes to append to short names

**Example**: If you ping "server" and search is "company.com", it tries "server.company.com"

---

```bash
# sudo systemctl status bind9
```
**What it does**: Checks the status of the BIND9 DNS server service

**Breakdown**:
- `systemctl` = System service control utility
- `status` = Show current status
- `bind9` = Berkeley Internet Name Domain (DNS server software)

**Output shows**:
- Active/inactive
- Running time
- Recent logs
- PID (process ID)

---

```bash
# su -
```
**What it does**: Switch user to root (superuser)
- `-` flag loads root's environment

**Alternative**: `sudo -i` (more secure)

---

#### 4. Network Utilities

```bash
# man lsof
```
**What it does**: Opens manual (help) page for `lsof` command

**lsof** = List Open Files
- Shows which processes have which files/ports open
- Super useful for debugging

---

```bash
# sudo lsof -i -n | grep 127.0.0.53
```
**Breaking it down**:

**`lsof -i`**: List network connections
**`-n`**: Don't resolve hostnames (faster)
**`| grep`**: Filter output
**`127.0.0.53`**: Local DNS stub resolver address

**Why this IP?**: Modern Ubuntu uses `systemd-resolved` which listens on `127.0.0.53:53`

**Example output**:
```
systemd-r  751  systemd-resolve   13u  IPv4  23456  UDP 127.0.0.53:53
systemd-r  751  systemd-resolve   14u  IPv4  23457  TCP 127.0.0.53:53 (LISTEN)
```

**Translation**: 
- The `systemd-resolved` service (PID 751)
- Is listening on `127.0.0.53` port 53
- For both UDP and TCP (DNS uses both)

---

```bash
# sudo lsof -i -n
```
**What it does**: Shows ALL network connections without hostname resolution

**Use cases**:
- See what services are listening on which ports
- Detect unexpected network connections
- Troubleshoot "port already in use" errors

**Example output**:
```
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd       850 root    3u  IPv4  15678      0t0  TCP *:22 (LISTEN)
nginx     1234 www     6u  IPv4  24680      0t0  TCP *:80 (LISTEN)
named     5678 bind   20u  IPv4  89012      0t0  UDP *:53
named     5678 bind   21u  IPv4  89013      0t0  TCP *:53 (LISTEN)
```

**Reading this**:
- SSH server listening on port 22
- Nginx web server on port 80
- BIND DNS server on port 53 (both UDP and TCP)

---

### Image 2 Commands

#### 1. SSH Command

```bash
# ssh -p 2233 zhjewel@10...
```
**What it does**: Connects to a remote server via SSH (Secure Shell)

**Breakdown**:
- `ssh` = Secure Shell (encrypted remote login)
- `-p 2233` = Use port 2233 (default is 22)
- `zhjewel` = Username
- `@10...` = IP address (appears to be 10.x.x.x)

**Why port 2233?**: Security through obscurity—non-standard ports reduce automated attacks

**What happens**:
```
Your Computer → [Encrypted SSH Tunnel] → Remote Server
```

---

#### 2. Domain Hierarchy Diagram

The notes show DNS hierarchy:

```
www.zhjlab.bd → N52 (Root)
                ↓
              NS1 (gTLD - Generic Top Level Domain)
                ↓
              ccTLD (Country Code TLD)
                ↓
              NS3
```

**Breaking down the hierarchy**:

1. **Root (.)**: The top of DNS tree
   - 13 root server clusters worldwide (A-M)
   - Know where to find TLD servers

2. **TLD (Top Level Domain)**:
   - **gTLD**: Generic TLD (.com, .org, .net)
   - **ccTLD**: Country Code TLD (.bd, .us, .uk)

3. **Domain levels**:
   - `.bd` → Root domain (Bangladesh)
   - `zhjlab` → Second-level domain
   - `www` → Subdomain/host

**Full resolution example**:
```
www.zhjlab.bd
     ↓
1. Root server: "Ask .bd servers"
2. .bd server: "Ask zhjlab.bd's nameservers (NS)"
3. zhjlab.bd NS: "www is at 10.100.68.50"
```

---

#### 3. DNS Record Types

**From the notes**:

**`.bd`** → Subdomain of root
**`.zhjlab`** → Subdomain of .bd
**`www`** → Domain/host

---

#### 4. Service Categories

**`* CDN, Forwarding, Caching name server`**

These are the server types discussed earlier, all tied together.

---

#### 5. More DNS Queries

```bash
# dig www.yahoo.com
```
**Full DNS query** for Yahoo showing all record types

---

```bash
# cName → alias name / FQDN
```
**CNAME Record**: Canonical Name (alias)

**Example**:
```
www.example.com.    IN    CNAME    server1.example.com.
server1.example.com. IN    A        192.168.1.100
```

**Translation**: 
- "www" is an alias for "server1"
- server1's actual IP is 192.168.1.100

**Real-world use**: 
- Point multiple names to one server
- Easy server migration (change one A record instead of many)

**FQDN** = Fully Qualified Domain Name (complete domain with trailing dot)
- `www.google.com.` ← Note the final dot (represents root)

---

```bash
# dig +trace www.yahoo.com
```
**What it does**: Shows the COMPLETE DNS resolution path

**Output shows**:
1. Root servers query
2. .com TLD query  
3. yahoo.com nameserver query
4. Final answer

**Example output**:
```
.                       NS    a.root-servers.net.
com.                    NS    a.gtld-servers.net.
yahoo.com.              NS    ns1.yahoo.com.
www.yahoo.com.          A     74.6.143.25
```

**Use case**: Debugging DNS issues—see exactly where resolution fails

---

```bash
# dig s00.zhjlab.bd
# dig +t ns zhjlab.bd
```
**Two queries**:
1. Get IP for `s00.zhjlab.bd`
2. Get nameserver records for `zhjlab.bd`

**NS Record output example**:
```
zhjlab.bd.  3600  IN  NS  ns1.zhjlab.bd.
zhjlab.bd.  3600  IN  NS  ns2.zhjlab.bd.
```

---

```bash
# dig +t any s00.zhjlab.bd @dns1.du.ac.bd
# dig +t ns s00.zhjlab.bd @dns1.du.ac.bd
```
**What's new here**: The `@dns1.du.ac.bd` part

**What it does**: Query a SPECIFIC DNS server (instead of your default)

**Breakdown**:
- `+t any` = Get ALL record types
- `+t ns` = Get only NS records
- `@dns1.du.ac.bd` = Use this DNS server for the query

**Use cases**:
- Test if a specific DNS server has updated records
- Bypass your ISP's DNS
- Test authoritative servers directly

**Example**:
```bash
# dig google.com @8.8.8.8    # Use Google DNS
# dig google.com @1.1.1.1    # Use Cloudflare DNS
```

---

## Practical Examples

### Example 1: Setting Up a Caching DNS Server

**Scenario**: You run a small office network and want faster DNS resolution

**Solution**: Set up a caching server

```bash
# Install BIND
sudo apt install bind9

# Configure as caching server
sudo nano /etc/bind/named.conf.options

# Add:
options {
    directory "/var/cache/bind";
    forwarders {
        8.8.8.8;
        8.8.4.4;
    };
    dnssec-validation auto;
};

# Restart service
sudo systemctl restart bind9
```

**How it works**:
1. Your computers query your caching server (192.168.1.1)
2. First query → server forwards to 8.8.8.8 → caches result
3. Subsequent queries → instant response from cache
4. Result: Faster browsing for everyone!

---

### Example 2: Testing DNS Resolution

**Problem**: Website not loading, is it DNS?

**Diagnosis steps**:

```bash
# Step 1: Can you reach the DNS server?
ping 8.8.8.8

# Step 2: Can DNS resolve?
dig google.com

# Step 3: Compare different DNS servers
dig google.com @8.8.8.8
dig google.com @1.1.1.1

# Step 4: Check your local hosts file
cat /etc/hosts | grep google

# Step 5: Flush DNS cache
sudo systemd-resolve --flush-caches
```

---

### Example 3: Understanding TTL (Time To Live)

**What is TTL?**: How long a DNS record can be cached

**Example record**:
```
google.com.  300  IN  A  142.250.185.46
             ↑
            TTL (5 minutes)
```

**What this means**:
- Caching servers can store this for 300 seconds (5 minutes)
- After 5 minutes, they must query again for fresh data

**Why it matters**:
- **Low TTL** (60s): Records update quickly, but more DNS traffic
- **High TTL** (86400s = 1 day): Less traffic, but slow updates

**Real scenario**: 
You're migrating your website to a new server:
1. Day before: Lower TTL to 60 seconds
2. Migration day: Change A record to new IP
3. Within minutes: Everyone sees new server
4. Day after: Raise TTL back to normal

---

### Example 4: Local Development with /etc/hosts

**Scenario**: Testing a website locally before going live

**Edit `/etc/hosts`**:
```bash
sudo nano /etc/hosts

# Add:
127.0.0.1  mywebsite.com
127.0.0.1  www.mywebsite.com
```

**Result**: 
- Your browser thinks `mywebsite.com` is on your computer
- You can test before DNS is configured
- Only affects your computer

**Use cases**:
- Web development
- Testing
- Blocking ads/malware (point bad domains to 127.0.0.1)

---

## Key Concepts Summary

### DNS Query Flow

```
User types: www.example.com

↓

/etc/hosts checked first
(127.0.0.1  localhost)
Not found? Continue...

↓

Local Caching Resolver
Check cache: Found? → Return IP
Not found? Continue...

↓

Query DNS Server (from /etc/resolv.conf)
nameserver 8.8.8.8

↓

Recursive Query Chain:
Root → TLD → Authoritative

↓

IP Address Returned & Cached

↓

Browser connects to IP
```

---

### Important Files & Their Purposes

| File | Purpose | Example |
|------|---------|---------|
| `/etc/hosts` | Local hostname to IP mapping | `127.0.0.1 localhost` |
| `/etc/resolv.conf` | DNS servers to use | `nameserver 8.8.8.8` |
| `/etc/host.conf` | Hostname resolution order | `order hosts,bind` |
| `/var/cache/bind/` | BIND DNS cache storage | (binary cache files) |
| `/etc/bind/named.conf` | BIND main configuration | Server settings |

---

### Common DNS Ports

- **Port 53**: DNS (both UDP and TCP)
  - UDP: Standard queries (fast)
  - TCP: Zone transfers, large responses
- **Port 853**: DNS over TLS (encrypted)
- **Port 443**: DNS over HTTPS (DoH)

---

### Command Quick Reference

| Command | Purpose |
|---------|---------|
| `dig domain.com` | Full DNS query |
| `dig +short domain.com` | Just the IP |
| `dig +trace domain.com` | Show full resolution path |
| `dig @8.8.8.8 domain.com` | Query specific DNS server |
| `host domain.com` | Simple lookup |
| `nslookup domain.com` | Interactive DNS lookup |
| `cat /etc/resolv.conf` | Check your DNS servers |
| `systemctl status bind9` | Check DNS server status |
| `sudo lsof -i :53` | See what's using DNS port |

---

## Troubleshooting Tips

### Problem: DNS Resolution Slow

**Checks**:
1. Ping your DNS server: `ping 8.8.8.8`
2. Try different DNS: `dig @1.1.1.1 google.com`
3. Check `/etc/resolv.conf` for too many nameservers
4. Consider running local caching resolver

---

### Problem: Domain Not Resolving

**Steps**:
```bash
# 1. Check if domain exists
dig +trace domain.com

# 2. Try authoritative servers
dig domain.com @ns1.domain.com

# 3. Check local overrides
cat /etc/hosts | grep domain

# 4. Flush cache
sudo systemd-resolve --flush-caches
```

---

### Problem: NXDOMAIN Error

**Meaning**: Domain doesn't exist

**Possible causes**:
- Typo in domain name
- Domain hasn't propagated yet (new domains take 24-48h)
- Incorrect DNS server
- /etc/hosts override

---

## Real-World Scenarios

### Scenario 1: Website Migration

**Challenge**: Move website to new server without downtime

**Steps**:
1. Lower TTL to 300 seconds (5 minutes) 24 hours before
2. Wait 24 hours (old TTL to expire)
3. Change A record to new server IP
4. Within 5 minutes, most users see new server
5. After 24 hours, raise TTL back to 3600 (1 hour)

---

### Scenario 2: Load Balancing

**Challenge**: Distribute traffic across multiple servers

**Solution**: Multiple A records

```
www.example.com.  300  IN  A  192.168.1.10
www.example.com.  300  IN  A  192.168.1.11  
www.example.com.  300  IN  A  192.168.1.12
```

**Result**: DNS returns IPs in rotating order (round-robin)

---

### Scenario 3: Security Filtering

**Challenge**: Block malicious domains for entire office

**Solution**: Forwarding DNS with blacklist

```bash
# Configure BIND with RPZ (Response Policy Zone)
# Automatically return 0.0.0.0 for bad domains

zone "rpz.example.com" {
    type master;
    file "/etc/bind/db.rpz";
};

# In db.rpz file:
malware-site.com    IN  A  0.0.0.0
phishing-site.com   IN  A  0.0.0.0
```

---

## Advanced Topics Preview

### DNSSEC (DNS Security Extensions)
- Cryptographically signs DNS records
- Prevents cache poisoning attacks
- Ensures DNS responses haven't been tampered with

### Split-Horizon DNS
- Different answers for internal vs external queries
- Example: `server.company.com` → `10.0.0.5` (internal) vs `203.0.113.5` (external)

### DNS-over-HTTPS (DoH)
- Encrypts DNS queries
- Prevents ISP snooping
- Used by browsers like Firefox

---

## Summary

You've learned:
- ✅ How DNS works (the internet's phonebook)
- ✅ Three types of servers: Authoritative, Caching, Forwarding
- ✅ Every command from your lab notes
- ✅ How to troubleshoot DNS issues
- ✅ Real-world DNS scenarios

**Key Takeaway**: DNS is the glue that makes the internet user-friendly. Without it, we'd all be memorizing IP addresses like phone numbers!

---

## Additional Resources

- BIND Documentation: https://bind9.readthedocs.io
- DNS RFC 1034/1035: Original DNS specifications
- DNSViz: Visualize DNS hierarchies
- dig Command Man Page: `man dig`

---

*Notes created for Networking Lab Course*
*Topic: DNS Servers, Caching, and Forwarding*
