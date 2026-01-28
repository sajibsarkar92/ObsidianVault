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

### Understanding the Three Main Types

Before diving deep, here's a quick overview:

| Type | Role | Has Own Data? | Queries Others? | Stores Cache? |
|------|------|---------------|-----------------|---------------|
| **Authoritative** | Source of truth | ✅ Yes | ❌ No | ❌ No |
| **Caching/Recursive** | Smart helper | ❌ No | ✅ Yes | ✅ Yes |
| **Forwarding** | Simple middleman | ❌ No | ✅ Yes (one server only) | ⚠️ Optional |

---

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

---

### 2. Caching DNS Server (Recursive Resolver)

**What it does**: This server doesn't own any domains, but it actively resolves queries by asking multiple servers and remembers (caches) answers from previous queries to speed things up.

**Analogy**: Like a resourceful librarian who doesn't own books but:
1. Knows how to find any book (by asking other libraries)
2. Remembers where books are after finding them once
3. Can answer your question faster next time because of their notes

**Example**:
- Google Public DNS (8.8.8.8)
- Cloudflare DNS (1.1.1.1)
- Your ISP's DNS server

**How it works**:
```
First Query for facebook.com:
User → Caching Server → "I don't know, let me find out"
                     → [Queries Root → .com TLD → facebook.com NS]
                     → Gets answer: 157.240.241.35
                     → Stores in cache for 5 minutes (TTL)
                     → Returns to user
Time taken: 150ms

Second Query for facebook.com (within 5 min):
User → Caching Server → "I remember! It's 157.240.241.35"
                     → Returns from cache immediately
Time taken: 2ms
```

**Key characteristics**:
- **Actively resolves**: Queries root → TLD → authoritative servers
- **Stores cache**: Remembers answers for the TTL period
- **Reduces latency**: Subsequent queries are instant
- **Reduces load**: Fewer queries to authoritative servers
- **Intelligence**: Knows the full DNS resolution process

**Cache behavior**:
- Each record stored with its TTL (Time To Live)
- After TTL expires, must query again for fresh data
- Can be flushed manually if needed

---

### 3. Forwarding DNS Server

**What it does**: This server doesn't resolve queries itself or cache extensively. It simply forwards ALL queries to another DNS server (usually a caching server) and passes back the answer.

**Analogy**: Like a receptionist at a company:
- Doesn't solve problems directly
- Forwards your call to the right department
- Waits for the answer and relays it back to you
- Doesn't remember the answers (or only briefly)

**Example**:
- Your home router (forwards to ISP's DNS)
- Office network DNS (forwards to company's main DNS or public DNS)
- Small organization DNS (forwards to 8.8.8.8)

**How it works**:
```
Query for google.com:
User → Forwarding Server → "I just forward everything"
                        → Forwards to 8.8.8.8
                        → 8.8.8.8 does the real work
                        → Gets answer back
                        → Passes answer to user
```

**Key characteristics**:
- **Passive**: Doesn't actively resolve DNS queries
- **Delegation**: Relies entirely on another DNS server
- **Simple**: Easy to configure and maintain
- **Limited caching**: May cache briefly, but not primary function
- **No intelligence**: Doesn't know DNS resolution process

**Configuration example**:
```
options {
    forwarders {
        8.8.8.8;
        8.8.4.4;
    };
    forward only;  # Don't try to resolve yourself, only forward
};
```

---

### 🔥 KEY DIFFERENCE: Caching vs Forwarding

**The Critical Distinction**:

**Caching Server (Recursive Resolver)**:
- **Active agent**: "I will figure this out myself"
- Queries root servers, TLD servers, authoritative servers
- Builds the answer step-by-step
- Caching is core functionality
- Example path: User → Cache → Root → TLD → Auth → Cache → User

**Forwarding Server**:
- **Passive relay**: "I'll ask someone else"
- Only knows one or two servers to ask
- Doesn't participate in DNS resolution
- Forwarding is core functionality (caching is secondary)
- Example path: User → Forward → External Cache → Forward → User

**Analogy Comparison**:

| Aspect | Caching Server | Forwarding Server |
|--------|----------------|-------------------|
| **Problem solving** | Detective who investigates | Secretary who forwards calls |
| **Knowledge** | Learns the DNS hierarchy | Only knows one phone number |
| **Independence** | Can work alone | Needs another server |
| **Speed** | Fast after first query | Depends on forwarding target |
| **Use case** | ISPs, public DNS | Small networks, home routers |

---

### Data Flow Comparison

#### Scenario 1: Using a Caching Server

```
                    FIRST QUERY
┌──────────┐
│   USER   │ "What's the IP of example.com?"
└─────┬────┘
      │
      ▼
┌─────────────────┐
│ CACHING SERVER  │ "Not in cache, I'll resolve it"
│   (8.8.8.8)     │
└────┬────────────┘
     │
     ├─► [1] Root Servers (.)
     │   "Ask .com TLD"
     │
     ├─► [2] .com TLD Servers
     │   "Ask example.com's NS"
     │
     └─► [3] example.com Authoritative NS
         "Here's the IP: 93.184.216.34"
         
         ↓ (Stores in cache with TTL)
         
┌─────────────────┐
│ CACHING SERVER  │ Returns IP + caches it
│   (8.8.8.8)     │
└────┬────────────┘
     │
     ▼
┌──────────┐
│   USER   │ Gets IP: 93.184.216.34
└──────────┘
Time: 150ms

                    SECOND QUERY (within TTL)
┌──────────┐
│   USER   │ "What's the IP of example.com?"
└─────┬────┘
      │
      ▼
┌─────────────────┐
│ CACHING SERVER  │ "Found in cache!"
│   (8.8.8.8)     │ Returns: 93.184.216.34
└────┬────────────┘
     │
     ▼
┌──────────┐
│   USER   │ Gets IP: 93.184.216.34
└──────────┘
Time: 2ms ⚡
```

---

#### Scenario 2: Using a Forwarding Server

```
                    EVERY QUERY (same process)
┌──────────┐
│   USER   │ "What's the IP of example.com?"
└─────┬────┘
      │
      ▼
┌──────────────────┐
│ FORWARDING SERVER│ "I just forward to 8.8.8.8"
│  (192.168.1.1)   │
└────┬─────────────┘
     │
     ▼
┌─────────────────┐
│ CACHING SERVER  │ Does the actual work:
│   (8.8.8.8)     │ • Checks cache OR
└────┬────────────┘ • Resolves recursively
     │
     ▼
┌──────────────────┐
│ FORWARDING SERVER│ Receives answer
│  (192.168.1.1)   │
└────┬─────────────┘
     │
     ▼
┌──────────┐
│   USER   │ Gets IP: 93.184.216.34
└──────────┘
Time: 2-150ms (depends on 8.8.8.8's cache)
```

**Notice**: The forwarding server adds a hop but doesn't do the resolution work.

---

#### Scenario 3: Hybrid Setup (Common in Organizations)

```
        CORPORATE NETWORK EXAMPLE

┌──────────────┐
│   Employee   │ Query: company.internal
│  Computer    │
└──────┬───────┘
       │
       ▼
┌────────────────────┐
│  Office Forwarding │ "Is this internal or external?"
│      Server        │
│   (10.0.0.1)       │
└────────┬───────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
Internal?  External?
    │         │
    ▼         └─► Forward to 8.8.8.8 (Caching Server)
┌───────────────┐
│ Authoritative │ Company's own domains
│   Server      │
│  (10.0.0.53)  │
└───────────────┘

Benefits:
✅ Internal domains resolved locally (fast + private)
✅ External domains cached by Google DNS (fast + reliable)
✅ Centralized control and monitoring
✅ Can block malicious domains at forwarding level
```

---

### Where They Sit in the Query Flow

**Complete Query Flow with All Server Types**:

```
1. USER COMPUTER
   └─ Checks /etc/hosts first
   └─ If not found, queries DNS server from /etc/resolv.conf
      │
      ▼
      
2. FIRST DNS SERVER (varies by network)
   
   Option A: FORWARDING SERVER (home/office router)
      │
      ├─ Simply forwards to configured server (ISP or public DNS)
      ├─ May cache briefly (optional)
      └─ Adds one hop but provides centralized control
      │
      ▼
      
   Option B: CACHING/RECURSIVE SERVER (ISP or public DNS)
      │
      ├─ Checks local cache first
      │   └─ HIT: Return immediately ⚡
      │   └─ MISS: Continue to resolution ▼
      │
      
3. RECURSIVE RESOLUTION (only if cache miss)
      │
      ├─► ROOT SERVERS (13 clusters worldwide)
      │   └─ "Don't know exact answer, ask TLD servers"
      │
      ├─► TLD SERVERS (.com, .org, .bd, etc.)
      │   └─ "Don't know exact answer, ask authoritative servers"
      │
      └─► AUTHORITATIVE SERVER (final answer)
          └─ "Here's the definitive answer" (aa flag set)
          
      ▼
      
4. ANSWER PROPAGATES BACK
   
   Authoritative → Caching (stores with TTL)
                → Forwarding (optional brief cache)
                → User Computer
                
5. BROWSER GETS IP
   └─ Connects to web server
   └─ Page loads
```

---

### Real-World Deployment Scenarios

#### Home Network
```
Your PC → Home Router (Forwarding) → ISP DNS (Caching) → Internet
           192.168.1.1                  8.8.8.8
```
- Router forwards everything to ISP or Google DNS
- Simple, no maintenance needed

---

#### Small Business
```
Employee PC → Office Forwarding Server → Google DNS (Caching)
                    10.0.0.1                 8.8.8.8
```
- Central point for monitoring and control
- Can add blacklist for malicious sites
- Easy to switch upstream DNS if needed

---

#### Large Enterprise
```
Employee PC → Regional Forwarding → Central Caching Cluster
              (10.x.x.1)              (10.0.0.53-55)
                                           │
                    ┌──────────────────────┼──────────────────────┐
                    ▼                      ▼                      ▼
            Internal Authoritative   External Cache        External Forward
            (company.com)            (8.8.8.8)             (to ISP if needed)
```
- Complex but optimized
- Internal domains stay internal (security + speed)
- External domains cached centrally
- Redundancy at every level

---

#### ISP Setup
```
Customer → ISP Caching Servers → Root/TLD/Authoritative
           (Massive caches)
           (Millions of users benefit)
```
- Huge cache hit rates (many users query same popular sites)
- Direct connections to root servers
- Reduced internet traffic

---

### Performance Implications

**Caching Server Advantages**:
- ⚡ Sub-5ms response for cached queries
- 🎯 Direct resolution without extra hops
- 📊 Better for high-query environments
- 🧠 Intelligent, can handle complex scenarios

**Caching Server Disadvantages**:
- 🔧 More complex to configure
- 💾 Requires more resources (RAM for cache)
- 🔄 Must implement recursion properly

**Forwarding Server Advantages**:
- 😊 Simple configuration
- 📦 Minimal resources needed
- 🎚️ Easy centralized control
- 🔀 Easy to change upstream DNS

**Forwarding Server Disadvantages**:
- 🐌 Extra hop adds 5-20ms latency
- 🔗 Dependency on upstream server
- 📉 No benefit if upstream is slow/down
- ❌ No cache = repeated queries cost same time

---

### When to Use Which?

**Use Caching Server when**:
- You're an ISP or large organization
- High query volume justifies the cache
- You need full control over resolution
- Performance is critical
- You want to reduce external dependencies

**Use Forwarding Server when**:
- Small network (home, small office)
- You want simple, centralized control
- You trust your upstream DNS provider
- You need quick setup with minimal maintenance
- You want to leverage someone else's caching infrastructure

**Use Both (Hybrid) when**:
- Large enterprise with internal and external domains
- You need internal privacy + external performance
- You want redundancy and load balancing
- Different regions need different configurations

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

### `/etc/host.conf`
**Purpose**: Controls hostname resolution order

**Expected Terminal Output**:
```bash
$ cat /etc/host.conf
# The "order" line is only used by old versions of the C library.
order hosts,bind
multi on
```

**What this means**:
- **`order hosts,bind`**: Check `/etc/hosts` file FIRST, then DNS (bind)
- **`multi on`**: Allow multiple IP addresses per hostname

**Why this matters**: If you add `127.0.0.1 facebook.com` to `/etc/hosts`, your computer will use that instead of DNS, effectively blocking Facebook!

---

### `/etc/hosts`
**Purpose**: Local hostname to IP mapping (overrides DNS)

**Expected Terminal Output**:
```bash
$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       your-computer-name
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

# Custom entries (if any)
192.168.1.10    server.local server
10.0.0.5        database.local db
```

**Format**: `IP_ADDRESS  hostname  [aliases...]`

**Reading this**:
- `127.0.0.1 localhost` = "localhost" always means "this computer"
- `192.168.1.10 server.local server` = Both "server.local" and "server" point to 192.168.1.10
- IPv6 addresses use `::` notation

**Practical use cases**:

**1. Block websites** (parental control / productivity):
```bash
$ sudo nano /etc/hosts
# Add these lines:
127.0.0.1  facebook.com
127.0.0.1  www.facebook.com
127.0.0.1  twitter.com
127.0.0.1  www.twitter.com
```
Now those sites won't load (they'll try to connect to your computer instead)!

**2. Development/testing**:
```bash
$ sudo nano /etc/hosts
# Add:
127.0.0.1  mywebsite.local
```
Now you can visit `http://mywebsite.local` in your browser and it connects to your local development server!

**3. Override DNS** (temporary):
```bash
$ sudo nano /etc/hosts
# Add:
93.184.216.34  test.example.com
```
Useful when migrating websites—test the new server before updating DNS.

---

```bash
# sudo nano s.hosts
```
**What it does**: Opens file `s.hosts` in nano text editor with admin privileges

**Why sudo?**: System files require root permissions to edit

**Expected behavior**:
```bash
$ sudo nano s.hosts
[sudo] password for user: ****

  GNU nano 6.2             /etc/s.hosts

# Custom DNS mappings
10.100.68.1     s00.zhjlab.bd
10.100.68.2     s01.zhjlab.bd
10.100.68.3     s02.zhjlab.bd

^G Help     ^O Write Out    ^W Where Is    ^K Cut
^X Exit     ^R Read File    ^\ Replace     ^U Paste
```

**Nano editor shortcuts**:
- `Ctrl+O` then `Enter`: Save file
- `Ctrl+X`: Exit
- `Ctrl+W`: Search
- `Ctrl+K`: Cut line
- `Ctrl+U`: Paste

**What this file does**: 
- Similar to `/etc/hosts`, maps hostnames to IPs
- Lab-specific DNS mappings
- Could be included in system configuration

---

```bash
# cat resolv.conf
```
**What it does**: Shows DNS resolver configuration

**Expected Terminal Output**:
```bash
$ cat /etc/resolv.conf
# This file is managed by man:systemd-resolved(8). Do not edit.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0 trust-ad
search localdomain
```

**How to interpret**:

**`nameserver 127.0.0.53`**:
- This is the DNS server your computer will query
- `127.0.0.53` = local systemd-resolved stub resolver
- systemd-resolved then forwards to real DNS servers

**`options edns0 trust-ad`**:
- `edns0`: Extension mechanisms for DNS (allows larger responses)
- `trust-ad`: Trust Authenticated Data flag (for DNSSEC)

**`search localdomain`**:
- Domain suffix to append to short names
- If you type `ping server` → tries `server.localdomain`

**Traditional `/etc/resolv.conf`** (older systems or manual configuration):
```bash
$ cat /etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
search company.com example.local
options timeout:2 attempts:3
```

**Reading this**:
- **First nameserver**: Try 8.8.8.8 (Google DNS) first
- **Second nameserver**: If first fails, try 8.8.4.4 (Google backup)
- **Third nameserver**: Last resort, try 1.1.1.1 (Cloudflare)
- **search domains**: Append these to short hostnames
- **timeout**: Wait 2 seconds before trying next server
- **attempts**: Try each server max 3 times

**Practical example**:
```
You type: ping database
System tries:
  1. database.company.com (from search domain)
  2. database.example.local (from search domain)
  3. database (as-is)
```

**How to check what systemd-resolved is actually using**:
```bash
$ resolvectl status
Global
       Protocols: +LLMNR +mDNS +DNSOverTLS DNSSEC=no/unsupported
resolv.conf mode: stub
Current DNS Server: 8.8.8.8
       DNS Servers: 8.8.8.8
                    8.8.4.4
```

This shows the REAL DNS servers being used (not just the stub resolver).

---

```bash
# sudo systemctl status bind9
```
**What it does**: Checks the status of the BIND9 DNS server service

**Expected Terminal Output** (when running):
```bash
$ sudo systemctl status bind9
● named.service - BIND Domain Name Server
     Loaded: loaded (/lib/systemd/system/named.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-01-27 14:30:15 +06; 2h 15min ago
       Docs: man:named(8)
   Main PID: 5678 (named)
      Tasks: 5 (limit: 4915)
     Memory: 42.5M
        CPU: 1.234s
     CGroup: /system.slice/named.service
             └─5678 /usr/sbin/named -f -u bind

Jan 27 14:30:15 server named[5678]: zone localhost/IN: loaded serial 2
Jan 27 14:30:15 server named[5678]: zone zhjlab.bd/IN: loaded serial 2026012701
Jan 27 14:30:15 server named[5678]: all zones loaded
Jan 27 14:30:15 server named[5678]: running
Jan 27 14:30:15 server systemd[1]: Started BIND Domain Name Server.
Jan 27 16:15:30 server named[5678]: client @0x7f8b4c001640 192.168.1.100#54321: query: www.example.com IN A +E(0) (192.168.1.1)
```

**How to interpret**:

| Line | Meaning |
|------|---------|
| **Loaded: enabled** | Service will start automatically on boot |
| **Active: active (running)** | Service is currently running ✅ |
| **since Tue 2026-01-27 14:30:15** | Started at this time |
| **2h 15min ago** | Has been running for 2 hours 15 minutes |
| **Main PID: 5678** | Process ID (for killing if needed) |
| **Memory: 42.5M** | RAM usage |
| **CPU: 1.234s** | Total CPU time used |

**Recent log entries show**:
- Zones loaded successfully (localhost, zhjlab.bd)
- DNS server is running and answering queries
- Recent query: 192.168.1.100 asked for www.example.com

**Expected Terminal Output** (when NOT running):
```bash
$ sudo systemctl status bind9
○ named.service - BIND Domain Name Server
     Loaded: loaded (/lib/systemd/system/named.service; enabled; preset: enabled)
     Active: inactive (dead) since Tue 2026-01-27 16:45:00 +06; 2min ago
       Docs: man:named(8)
    Process: 5678 ExecStart=/usr/sbin/named -f $OPTIONS (code=exited, status=0/SUCCESS)
   Main PID: 5678 (code=exited, status=0/SUCCESS)
        CPU: 1.234s

Jan 27 16:45:00 server systemd[1]: Stopping BIND Domain Name Server...
Jan 27 16:45:00 server named[5678]: shutting down: flushing changes
Jan 27 16:45:00 server named[5678]: stopping command channel on 127.0.0.1#953
Jan 27 16:45:00 server named[5678]: no longer listening on 0.0.0.0#53
Jan 27 16:45:00 server systemd[1]: named.service: Deactivated successfully.
Jan 27 16:45:00 server systemd[1]: Stopped BIND Domain Name Server.
```

**How to interpret**:
- **Active: inactive (dead)** = Service is NOT running ❌
- **since Tue 2026-01-27 16:45:00** = Stopped at this time
- **code=exited, status=0/SUCCESS** = Stopped cleanly (no errors)

**Expected Terminal Output** (when failed):
```bash
$ sudo systemctl status bind9
× named.service - BIND Domain Name Server
     Loaded: loaded (/lib/systemd/system/named.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Tue 2026-01-27 17:00:00 +06; 30s ago
       Docs: man:named(8)
    Process: 6789 ExecStart=/usr/sbin/named -f $OPTIONS (code=exited, status=1/FAILURE)
   Main PID: 6789 (code=exited, status=1/FAILURE)
        CPU: 125ms

Jan 27 17:00:00 server named[6789]: loading configuration from '/etc/bind/named.conf'
Jan 27 17:00:00 server named[6789]: /etc/bind/named.conf:15: missing ';' before 'zone'
Jan 27 17:00:00 server named[6789]: loading configuration: failure
Jan 27 17:00:00 server named[6789]: exiting (due to fatal error)
Jan 27 17:00:00 server systemd[1]: named.service: Main process exited, code=exited, status=1/FAILURE
Jan 27 17:00:00 server systemd[1]: named.service: Failed with result 'exit-code'.
Jan 27 17:00:00 server systemd[1]: Failed to start BIND Domain Name Server.
```

**How to interpret**:
- **Active: failed** = Service failed to start ❌
- **status=1/FAILURE** = Exited with error code
- **Error message**: `/etc/bind/named.conf:15: missing ';'` 
- **Fix needed**: Edit config file, line 15, add missing semicolon

**Common systemctl commands**:
```bash
# Start the service
$ sudo systemctl start bind9

# Stop the service
$ sudo systemctl stop bind9

# Restart (stop + start)
$ sudo systemctl restart bind9

# Reload config without stopping
$ sudo systemctl reload bind9

# Enable (start on boot)
$ sudo systemctl enable bind9

# Disable (don't start on boot)
$ sudo systemctl disable bind9

# View full logs
$ sudo journalctl -u bind9

# View recent logs (last 50 lines)
$ sudo journalctl -u bind9 -n 50

# Follow logs in real-time
$ sudo journalctl -u bind9 -f
```

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
**`-n`**: Don't resolve hostnames (faster, shows IPs not names)
**`| grep`**: Filter output to show only lines containing...
**`127.0.0.53`**: Local DNS stub resolver address

**Why this IP?**: Modern Ubuntu uses `systemd-resolved` which listens on `127.0.0.53:53`

**Expected Terminal Output**:
```bash
$ sudo lsof -i -n | grep 127.0.0.53
systemd-r  751  systemd-resolve   13u  IPv4  23456      0t0  UDP 127.0.0.53:53
systemd-r  751  systemd-resolve   14u  IPv4  23457      0t0  TCP 127.0.0.53:53 (LISTEN)
```

**How to Interpret** (column by column):

| Column | Value | Meaning |
|--------|-------|---------|
| **COMMAND** | `systemd-r` | Process name (systemd-resolved, truncated) |
| **PID** | `751` | Process ID (unique identifier) |
| **USER** | `systemd-resolve` | User running the process |
| **FD** | `13u` | File Descriptor: 13 = number, u = read/write mode |
| **TYPE** | `IPv4` | IP version 4 |
| **DEVICE** | `23456` | Kernel device number |
| **SIZE/OFF** | `0t0` | Size/Offset (0 for network sockets) |
| **NODE** | `UDP` or `TCP` | Protocol type |
| **NAME** | `127.0.0.53:53` | IP address and port |
| **STATE** | `(LISTEN)` | TCP is listening for connections |

**What this tells you**:
1. **systemd-resolved** (PID 751) is running
2. It's listening on IP `127.0.0.53`, port `53` (DNS port)
3. It uses **both UDP and TCP**:
   - UDP: For standard DNS queries (fast, connectionless)
   - TCP: For large responses or zone transfers
4. Your system's DNS is working!

**Why both UDP and TCP?**
- **UDP 53**: 99% of DNS queries (single packet request/response)
- **TCP 53**: Large responses >512 bytes, zone transfers, or when UDP fails

**Troubleshooting with this**:
```bash
# Nothing returned? DNS service not running!
$ sudo systemctl status systemd-resolved

# Different IP/port? Custom DNS configuration
# Multiple entries? Multiple DNS services (conflict!)
```

---

```bash
# sudo lsof -i -n
```
**What it does**: Shows ALL network connections without hostname resolution (much more output!)

**Expected Terminal Output** (partial, it's usually very long):
```bash
$ sudo lsof -i -n
COMMAND     PID        USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd-r   751 systemd-resolve   13u  IPv4  23456      0t0  UDP 127.0.0.53:53
systemd-r   751 systemd-resolve   14u  IPv4  23457      0t0  TCP 127.0.0.53:53 (LISTEN)
sshd        850        root    3u  IPv4  15678      0t0  TCP *:22 (LISTEN)
sshd        850        root    4u  IPv6  15679      0t0  TCP *:22 (LISTEN)
nginx      1234     www-data    6u  IPv4  24680      0t0  TCP *:80 (LISTEN)
nginx      1234     www-data    7u  IPv4  24681      0t0  TCP *:443 (LISTEN)
named      5678        bind   20u  IPv4  89012      0t0  UDP *:53
named      5678        bind   21u  IPv4  89013      0t0  TCP *:53 (LISTEN)
chrome     7890        user   45u  IPv4  92345      0t0  TCP 192.168.1.100:54321->142.250.185.46:443 (ESTABLISHED)
chrome     7890        user   46u  IPv4  92346      0t0  TCP 192.168.1.100:54322->151.101.1.140:443 (ESTABLISHED)
```

**How to Read Different Connection Types**:

**1. Listening Services** (LISTEN state):
```
sshd    850   root   3u  IPv4  15678  0t0  TCP *:22 (LISTEN)
```
- **sshd**: SSH server
- **Port 22**: Standard SSH port
- **\*:22**: Listening on ALL network interfaces
- **(LISTEN)**: Waiting for incoming connections
- **Meaning**: SSH server is ready to accept connections

---

**2. Established Connections** (ESTABLISHED state):
```
chrome  7890  user  45u  IPv4  92345  0t0  TCP 192.168.1.100:54321->142.250.185.46:443 (ESTABLISHED)
```
- **chrome**: Browser process
- **192.168.1.100:54321**: Your local IP:port
- **142.250.185.46:443**: Remote server (Google) IP:port (HTTPS)
- **(ESTABLISHED)**: Active connection in use
- **Meaning**: Your browser has an active HTTPS connection to Google

---

**3. DNS Services** (UDP and TCP):
```
named   5678  bind  20u  IPv4  89012  0t0  UDP *:53
named   5678  bind  21u  IPv4  89013  0t0  TCP *:53 (LISTEN)
```
- **named**: BIND DNS server process
- **Port 53**: Standard DNS port
- **Both UDP and TCP**: Ready for all DNS query types
- **Meaning**: Full DNS server running and ready

---

**4. Web Server**:
```
nginx   1234  www-data  6u  IPv4  24680  0t0  TCP *:80 (LISTEN)
nginx   1234  www-data  7u  IPv4  24681  0t0  TCP *:443 (LISTEN)
```
- **nginx**: Web server process
- **Port 80**: HTTP (unencrypted web)
- **Port 443**: HTTPS (encrypted web)
- **Meaning**: Web server ready to serve websites

---

**Use Cases for `lsof -i -n`**:

**1. Find what's using a port**:
```bash
$ sudo lsof -i -n | grep :80
nginx   1234  www-data  6u  IPv4  24680  0t0  TCP *:80 (LISTEN)
```
Translation: nginx is using port 80

---

**2. Troubleshoot "Address already in use"**:
```bash
$ sudo lsof -i -n | grep :8080
python  9876  user  3u  IPv4  45678  0t0  TCP *:8080 (LISTEN)
```
Translation: Python process 9876 is already using port 8080. Kill it: `sudo kill 9876`

---

**3. See all your connections**:
```bash
$ lsof -i -n | grep ESTABLISHED
chrome  7890  user  45u  IPv4  92345  0t0  TCP 192.168.1.100:54321->142.250.185.46:443 (ESTABLISHED)
slack   8901  user  22u  IPv4  93456  0t0  TCP 192.168.1.100:54322->54.230.45.67:443 (ESTABLISHED)
```
Translation: You're connected to Google and Slack right now

---

**4. Check if DNS server is running**:
```bash
$ sudo lsof -i -n | grep :53
systemd-r  751  systemd-resolve  13u  IPv4  23456  0t0  UDP 127.0.0.53:53
named     5678  bind             20u  IPv4  89012  0t0  UDP *:53
```
Translation: Two DNS services running (might be a conflict!)

---

**5. Security audit - see unexpected connections**:
```bash
$ sudo lsof -i -n | grep ESTABLISHED
malware  1337  root  5u  IPv4  66666  0t0  TCP 192.168.1.100:52341->45.123.45.67:8080 (ESTABLISHED)
```
Translation: Unknown process connecting to suspicious IP! Investigate!

---

**Common Filters**:
```bash
# Only TCP connections
$ sudo lsof -i tcp -n

# Only UDP connections  
$ sudo lsof -i udp -n

# Only IPv4
$ sudo lsof -i 4 -n

# Only IPv6
$ sudo lsof -i 6 -n

# Specific port
$ sudo lsof -i :80 -n

# Specific IP
$ sudo lsof -i @192.168.1.100 -n

# Specific user
$ lsof -i -n -u username
```

---

**Connection States Explained**:

| State | Meaning | Example |
|-------|---------|---------|
| **LISTEN** | Waiting for connections | Web server ready for browsers |
| **ESTABLISHED** | Active connection | You're browsing a website |
| **TIME_WAIT** | Connection closing | Browser finished loading page |
| **CLOSE_WAIT** | Waiting to close | Server waiting to confirm close |
| **SYN_SENT** | Trying to connect | Browser attempting to reach server |
| **SYN_RECEIVED** | Server got connection request | Server responding to your browser |

---

**Practical Example - Full Investigation**:

**Problem**: Website not loading, is it the server?

```bash
# Step 1: Is the web server running?
$ sudo lsof -i :80 -n
# Nothing? Server is down! Start it:
$ sudo systemctl start nginx

# Step 2: Is it listening?
$ sudo lsof -i :80 -n | grep LISTEN
nginx  1234  www-data  6u  IPv4  24680  0t0  TCP *:80 (LISTEN)
# Yes! Server is listening

# Step 3: Are there active connections?
$ sudo lsof -i :80 -n | grep ESTABLISHED
nginx  1234  www-data  8u  IPv4  24690  0t0  TCP 192.168.1.1:80->192.168.1.100:54567 (ESTABLISHED)
# Yes! Someone is connected

# Step 4: Is DNS working?
$ sudo lsof -i :53 -n
systemd-r  751  systemd-resolve  13u  IPv4  23456  0t0  UDP 127.0.0.53:53
# Yes! DNS is running

# Conclusion: Server is fine, problem is elsewhere (maybe firewall, routing, etc.)
```

---

### Image 2 & 3 Commands

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

**Expected Terminal Output**:
```bash
$ ssh -p 2233 zhjewel@10.100.68.50
The authenticity of host '[10.100.68.50]:2233 ([10.100.68.50]:2233)' can't be established.
ECDSA key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[10.100.68.50]:2233' (ECDSA) to the list of known hosts.
zhjewel@10.100.68.50's password: 
Welcome to Ubuntu 22.04.3 LTS
Last login: Mon Jan 27 14:23:15 2026 from 10.100.68.1
zhjewel@server:~$
```

**Interpretation**:
- First time connecting: Asked to verify server fingerprint (prevents man-in-the-middle attacks)
- Password prompt appears (or uses SSH key if configured)
- Successful login shows welcome message and new prompt
- You're now controlling the remote machine!

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

**Expected Terminal Output**:
```bash
$ dig www.yahoo.com

; <<>> DiG 9.18.12-Ubuntu <<>> www.yahoo.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;www.yahoo.com.			IN	A

;; ANSWER SECTION:
www.yahoo.com.		1800	IN	CNAME	new-fp-shed.wg1.b.yahoo.com.
new-fp-shed.wg1.b.yahoo.com. 60	IN	A	74.6.143.25
new-fp-shed.wg1.b.yahoo.com. 60	IN	A	74.6.231.20

;; Query time: 45 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Tue Jan 27 15:30:22 +06 2026
;; MSG SIZE  rcvd: 112
```

**How to Interpret**:
- **HEADER section**: 
  - `status: NOERROR` = Query successful (vs NXDOMAIN = domain not found)
  - `flags: qr rd ra` = Query Response, Recursion Desired, Recursion Available
  - `ANSWER: 3` = Got 3 records back
  
- **QUESTION SECTION**: What you asked for (A record for www.yahoo.com)

- **ANSWER SECTION**: The actual answer
  - `www.yahoo.com` is a CNAME (alias) pointing to `new-fp-shed.wg1.b.yahoo.com`
  - That domain has 2 A records (2 IP addresses for load balancing)
  - `1800` = TTL in seconds (30 minutes)

- **Query time**: 45 milliseconds to get answer
- **SERVER**: Which DNS server answered (127.0.0.53 = local systemd-resolved)

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
**What it does**: Shows the COMPLETE DNS resolution path from root servers to final answer

**Expected Terminal Output**:
```bash
$ dig +trace www.yahoo.com

; <<>> DiG 9.18.12-Ubuntu <<>> +trace www.yahoo.com
;; global options: +cmd
.			86258	IN	NS	a.root-servers.net.
.			86258	IN	NS	b.root-servers.net.
.			86258	IN	NS	c.root-servers.net.
[... more root servers ...]
;; Received 228 bytes from 127.0.0.53#53(127.0.0.53) in 5 ms

com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
[... more TLD servers ...]
;; Received 1170 bytes from 198.41.0.4#53(a.root-servers.net) in 178 ms

yahoo.com.		172800	IN	NS	ns1.yahoo.com.
yahoo.com.		172800	IN	NS	ns2.yahoo.com.
yahoo.com.		172800	IN	NS	ns3.yahoo.com.
;; Received 523 bytes from 192.12.94.30#53(e.gtld-servers.net) in 45 ms

www.yahoo.com.		1800	IN	CNAME	new-fp-shed.wg1.b.yahoo.com.
new-fp-shed.wg1.b.yahoo.com. 60	IN	A	74.6.143.25
new-fp-shed.wg1.b.yahoo.com. 60	IN	A	74.6.231.20
;; Received 142 bytes from 68.180.131.16#53(ns1.yahoo.com) in 23 ms
```

**How to Interpret** (step-by-step):

**Step 1 - Root Servers** (the `.`):
```
.    IN    NS    a.root-servers.net.
```
- Started at root servers
- They don't know about yahoo.com directly
- They say: "Ask the .com TLD servers"

**Step 2 - TLD Servers** (.com):
```
com.    IN    NS    a.gtld-servers.net.
Received from 198.41.0.4 (a.root-servers.net)
```
- Queried a root server at IP 198.41.0.4
- Got list of .com nameservers (Generic TLD servers)
- They say: "Ask yahoo.com's nameservers"

**Step 3 - Authoritative Servers** (yahoo.com):
```
yahoo.com.    IN    NS    ns1.yahoo.com.
Received from 192.12.94.30 (e.gtld-servers.net)
```
- Queried .com TLD server at 192.12.94.30
- Got Yahoo's authoritative nameservers
- These know the actual answer

**Step 4 - Final Answer**:
```
www.yahoo.com.    IN    CNAME    new-fp-shed.wg1.b.yahoo.com.
new-fp-shed.wg1.b.yahoo.com.    IN    A    74.6.143.25
Received from 68.180.131.16 (ns1.yahoo.com)
```
- Queried Yahoo's nameserver at 68.180.131.16
- Got the real answer: www is an alias, points to actual servers
- Final IP addresses: 74.6.143.25 and 74.6.231.20

**Timing Analysis**:
- Root query: 178 ms (international servers)
- TLD query: 45 ms 
- Authoritative: 23 ms
- Total: ~250 ms for full resolution

**Use case**: Perfect for debugging DNS propagation issues—see exactly where the chain breaks!

---

```bash
# dig s00.zhjlab.bd
```
**What it does**: Standard query for A record of `s00.zhjlab.bd`

**Expected Terminal Output**:
```bash
$ dig s00.zhjlab.bd

; <<>> DiG 9.18.12-Ubuntu <<>> s00.zhjlab.bd
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54321
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; QUESTION SECTION:
;s00.zhjlab.bd.			IN	A

;; ANSWER SECTION:
s00.zhjlab.bd.		3600	IN	A	10.100.68.1

;; AUTHORITY SECTION:
zhjlab.bd.		3600	IN	NS	ns1.zhjlab.bd.
zhjlab.bd.		3600	IN	NS	ns2.zhjlab.bd.

;; ADDITIONAL SECTION:
ns1.zhjlab.bd.		3600	IN	A	10.100.68.10
ns2.zhjlab.bd.		3600	IN	A	10.100.68.11

;; Query time: 12 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Tue Jan 27 15:35:10 +06 2026
;; MSG SIZE  rcvd: 142
```

**Interpretation**:
- **ANSWER**: `s00.zhjlab.bd` resolves to `10.100.68.1`
- **AUTHORITY**: Shows which nameservers are authoritative for zhjlab.bd
- **ADDITIONAL**: Helpful bonus info—the IPs of those nameservers
- **Query time**: 12ms (very fast, likely cached or local network)

---

```bash
# dig +t ns zhjlab.bd
```
**What it does**: Query specifically for NS (nameserver) records

**Expected Terminal Output**:
```bash
$ dig +t ns zhjlab.bd

; <<>> DiG 9.18.12-Ubuntu <<>> +t ns zhjlab.bd
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11223
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 3

;; QUESTION SECTION:
;zhjlab.bd.			IN	NS

;; ANSWER SECTION:
zhjlab.bd.		3600	IN	NS	ns1.zhjlab.bd.
zhjlab.bd.		3600	IN	NS	ns2.zhjlab.bd.

;; ADDITIONAL SECTION:
ns1.zhjlab.bd.		3600	IN	A	10.100.68.10
ns2.zhjlab.bd.		3600	IN	A	10.100.68.11

;; Query time: 8 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Tue Jan 27 15:36:45 +06 2026
;; MSG SIZE  rcvd: 108
```

**Interpretation**:
- Shows the domain `zhjlab.bd` has 2 nameservers (redundancy!)
- `ns1.zhjlab.bd` at 10.100.68.10
- `ns2.zhjlab.bd` at 10.100.68.11
- This is a local network setup (10.x.x.x are private IPs)

---

```bash
# dig +t any s00.zhjlab.bd @dns1.du.ac.bd
# dig +t ns s00.zhjlab.bd @dns1.du.ac.bd
```
**What's new here**: The `@dns1.du.ac.bd` part

**What it does**: Query a SPECIFIC DNS server (instead of your default)

**Breakdown**:
- `+t any` = Get ALL record types (A, AAAA, MX, TXT, CNAME, NS, etc.)
- `+t ns` = Get only NS records
- `@dns1.du.ac.bd` = Use this DNS server for the query (Dhaka University's DNS)

**Expected Terminal Output** (for `+t any`):
```bash
$ dig +t any s00.zhjlab.bd @dns1.du.ac.bd

; <<>> DiG 9.18.12-Ubuntu <<>> +t any s00.zhjlab.bd @dns1.du.ac.bd
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33445
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1
                      ^^
                      |
               NOTICE THE 'aa' FLAG!

;; QUESTION SECTION:
;s00.zhjlab.bd.			IN	ANY

;; ANSWER SECTION:
s00.zhjlab.bd.		3600	IN	A	10.100.68.1
s00.zhjlab.bd.		3600	IN	NS	ns1.zhjlab.bd.
s00.zhjlab.bd.		3600	IN	NS	ns2.zhjlab.bd.
s00.zhjlab.bd.		3600	IN	SOA	ns1.zhjlab.bd. admin.zhjlab.bd. 2026012701 3600 1800 604800 86400

;; Query time: 15 msec
;; SERVER: 202.51.200.100#53(dns1.du.ac.bd)
;; WHEN: Tue Jan 27 15:40:22 +06 2026
;; MSG SIZE  rcvd: 185
```

**Critical Interpretation Point**:

Notice the **`aa` flag** in the flags section: `qr aa rd ra`

- `aa` = **Authoritative Answer**
- This means `dns1.du.ac.bd` is THE authoritative server for `zhjlab.bd`
- It's not giving you a cached answer—this is the official, definitive answer
- If the flag was just `qr rd ra` (no `aa`), it would be a cached/recursive answer

**Why query specific servers?**
1. **Testing**: "Has this DNS server updated its records?"
2. **Comparison**: Compare Google DNS vs Cloudflare DNS
3. **Troubleshooting**: "Is the authoritative server working?"
4. **Bypassing**: Skip your ISP's potentially censored DNS

**Example comparison**:
```bash
# Your default DNS (might be cached/old)
$ dig example.com
example.com.    3600    IN    A    93.184.216.34

# Query authoritative directly (always current)
$ dig example.com @ns1.example.com
example.com.    3600    IN    A    93.184.216.99  # Different! Just updated!
```

**Use cases**:
- Test if a specific DNS server has updated records
- Bypass your ISP's DNS
- Test authoritative servers directly
- Verify DNS propagation across different servers

**More examples**:
```bash
# Query Google's DNS
$ dig google.com @8.8.8.8

# Query Cloudflare's DNS  
$ dig google.com @1.1.1.1

# Query the authoritative server directly
$ dig google.com @ns1.google.com
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

---

## How All These Commands Tie Together

### Understanding the Complete Workflow

The commands in your lab notes tell a story of DNS server setup, configuration, and testing. Let's see how they all connect:

```
┌─────────────────────────────────────────────────────────────────┐
│                    DNS LAB WORKFLOW                              │
└─────────────────────────────────────────────────────────────────┘

PHASE 1: REMOTE ACCESS
├─ ssh -p 2233 zhjewel@10.x.x.x
│  └─> Connect to remote lab server
│  └─> Why? DNS servers usually run on dedicated machines
│  └─> Non-standard port (2233) for security

PHASE 2: UNDERSTANDING THE HIERARCHY  
├─ Domain structure learned:
│  └─> www.zhjlab.bd breakdown:
│      ├─ Root (.)
│      ├─ .bd (Bangladesh ccTLD)
│      ├─ zhjlab (second-level domain)
│      └─ www (hostname)

PHASE 3: SYSTEM EXPLORATION
├─ cd /etc
│  └─> Navigate to configuration directory
│
├─ ls host [TAB][TAB]
│  └─> Find host-related files:
│      ├─ hosts
│      ├─ host.conf
│      └─ hostname
│
├─ cat host.conf
│  └─> Check resolution order (hosts first, then DNS)
│
├─ cat hosts
│  └─> View local hostname mappings
│  └─> Understanding: These override DNS!
│
├─ cat resolv.conf
│  └─> See which DNS servers system uses
│  └─> Connection: This is WHERE queries go

PHASE 4: DNS SERVER STATUS
├─ sudo systemctl status bind9
│  └─> Check if DNS server is running
│  └─> Why important? Need server running to test!
│
├─ sudo lsof -i -n | grep 127.0.0.53
│  └─> Verify DNS service listening on port 53
│  └─> Connection: Confirms systemd-resolved active
│
└─ sudo lsof -i -n
   └─> See ALL network services
   └─> Connection: Comprehensive network service audit

PHASE 5: DNS QUERY TESTING
├─ dig +t a zhjlab.bd
│  └─> Get IPv4 address
│  └─> Purpose: Basic functionality test
│
├─ dig +t soa zhjlab.bd
│  └─> Get Start of Authority record
│  └─> Purpose: See zone metadata & authority info
│
├─ dig +t ns zhjlab.bd
│  └─> Get nameserver records
│  └─> Purpose: See which servers are authoritative
│  └─> Connection: These are the "authority" in SOA!
│
├─ host s00.zhjlab.bd
│  └─> Simple lookup (easier than dig)
│  └─> Purpose: Quick IP check
│
├─ host www.yahoo.com
│  └─> Test external domain
│  └─> Purpose: Verify internet DNS works
│  └─> Connection: Tests forwarding to external DNS
│
├─ dig www.yahoo.com
│  └─> Detailed external query
│  └─> Shows: CNAME records, multiple A records
│  └─> Learning: Real sites use load balancing!
│
├─ dig +trace www.yahoo.com
│  └─> Show COMPLETE resolution path
│  └─> Purpose: Educational - see root → TLD → auth flow
│  └─> Connection: Demonstrates DNS hierarchy visually
│
├─ dig s00.zhjlab.bd
│  └─> Query local domain
│  └─> Connection: Compare with host command above
│
├─ dig +t any s00.zhjlab.bd @dns1.du.ac.bd
│  └─> Query specific server for ALL records
│  └─> Purpose: Test authoritative server directly
│  └─> Connection: Bypass caching, get fresh data
│  └─> Why @dns1.du.ac.bd? It's the authoritative server!
│
└─ dig +t ns s00.zhjlab.bd @dns1.du.ac.bd
   └─> Get NS records from authoritative server
   └─> Purpose: Confirm authoritative designation
   └─> Connection: Verify 'aa' flag in response

PHASE 6: EDIT & CUSTOMIZE
└─ sudo nano s.hosts
   └─> Create/edit custom hostname mappings
   └─> Purpose: Local overrides for lab environment
   └─> Connection: Combines with /etc/hosts concept
```

---

### The Hidden Connections

Let me show you how these commands relate to each other in ways that might not be obvious:

#### Connection 1: Resolution Order Chain

```
/etc/host.conf  ──→  Defines: "Check hosts, then bind (DNS)"
       │
       ├──→ /etc/hosts  ──→  Local mappings (checked FIRST)
       │                     └─> If match found, STOP here
       │
       └──→ /etc/resolv.conf  ──→  DNS servers to query (checked SECOND)
                  │                └─> nameserver 127.0.0.53
                  │
                  └──→ systemd-resolved (127.0.0.53)
                         │
                         └──→ Forwards to real DNS
                              └─> dig commands query HERE
```

**Practical example**:
```bash
# Scenario: You type "ping s00.zhjlab.bd"

Step 1: Check /etc/host.conf
        → Says: "order hosts,bind"

Step 2: Check /etc/hosts
        → If found: "10.100.68.1  s00.zhjlab.bd"
        → DONE! Use 10.100.68.1, never touch DNS

Step 3: If NOT in /etc/hosts:
        → Check /etc/resolv.conf
        → Query nameserver 127.0.0.53
        → Which forwards to real DNS (e.g., 8.8.8.8)

Step 4: DNS resolution
        → dig +trace shows this path
        → Returns IP address
```

---

#### Connection 2: Server Types in Action

**Your lab uses ALL THREE types**:

```
┌─────────────────────────────────────────────────────────┐
│ LOCAL MACHINE (Your computer in lab)                    │
│                                                          │
│ /etc/resolv.conf shows: nameserver 127.0.0.53          │
│                                                          │
│ This is systemd-resolved (FORWARDING type)              │
│ ├─ Verified by: lsof -i -n | grep 127.0.0.53          │
│ └─ Forwards to: Configured DNS servers                  │
└──────────────────────┬───────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│ CACHING SERVER (Google DNS or similar)                  │
│                                                          │
│ IP: 8.8.8.8 (example)                                   │
│                                                          │
│ Role: Resolves recursively, caches results              │
│ ├─ Used by: dig www.yahoo.com (no @ specified)         │
│ └─ Behavior: Queries root → TLD → authoritative         │
└──────────────────────┬───────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│ AUTHORITATIVE SERVER (dns1.du.ac.bd)                    │
│                                                          │
│ IP: 202.51.200.100 (example)                            │
│                                                          │
│ Role: Owns zhjlab.bd domain records                     │
│ ├─ Used by: dig @dns1.du.ac.bd s00.zhjlab.bd           │
│ ├─ Response has: 'aa' flag (authoritative answer)       │
│ └─ Loaded via: systemctl status bind9                   │
│     └─> Shows: "zone zhjlab.bd/IN: loaded"              │
└─────────────────────────────────────────────────────────┘
```

**Testing this flow**:

```bash
# Test 1: Default query (uses forwarding → caching)
$ dig s00.zhjlab.bd
# Goes: Your PC → systemd-resolved → 8.8.8.8 → dns1.du.ac.bd
# Response: qr rd ra (recursion desired & available)

# Test 2: Direct to authoritative (bypasses caching)
$ dig s00.zhjlab.bd @dns1.du.ac.bd
# Goes: Your PC → dns1.du.ac.bd directly
# Response: qr aa rd ra (authoritative answer!)
```

---

#### Connection 3: The SOA → NS → A Record Chain

```bash
# Command 1: Get SOA (Start of Authority)
$ dig +t soa zhjlab.bd

# Returns something like:
zhjlab.bd.  3600  IN  SOA  ns1.zhjlab.bd. admin.zhjlab.bd. ...
                          ^^^^^^^^^^^^^^^^
                          Primary nameserver

# Command 2: Get NS (Nameservers)
$ dig +t ns zhjlab.bd

# Returns:
zhjlab.bd.  3600  IN  NS  ns1.zhjlab.bd.
zhjlab.bd.  3600  IN  NS  ns2.zhjlab.bd.
                        ^^^^^^^^^^^^^^^^
                        These are the authoritative servers!

# Command 3: Get A records for those nameservers
$ dig +t a ns1.zhjlab.bd

# Returns:
ns1.zhjlab.bd.  3600  IN  A  10.100.68.10
                           ^^^^^^^^^^^^^^
                           This is WHERE the authoritative server lives!

# Command 4: Query that server directly
$ dig s00.zhjlab.bd @ns1.zhjlab.bd
# Or equivalently:
$ dig s00.zhjlab.bd @10.100.68.10
```

**The chain reveals**:
1. SOA tells you WHO is in charge (ns1.zhjlab.bd)
2. NS records tell you ALL servers in charge
3. A records tell you WHERE those servers are
4. @ syntax lets you query them directly

---

#### Connection 4: BIND9 Configuration → Active Queries

```bash
# Check BIND9 status
$ sudo systemctl status bind9

# Output shows:
Jan 27 14:30:15 server named[5678]: zone zhjlab.bd/IN: loaded serial 2026012701
                                    ^^^^^^^^^^^^^^^^^^^^
                                    This zone is configured!

# Now when you query:
$ dig +t soa zhjlab.bd

# The SOA record shows:
zhjlab.bd.  3600  IN  SOA  ns1.zhjlab.bd. admin.zhjlab.bd. 2026012701 ...
                                                            ^^^^^^^^^^
                                                            SAME serial number!
```

**This proves**: The BIND9 server you checked with `systemctl` is the SAME server responding to your queries!

---

#### Connection 5: lsof Shows The Listeners

```bash
# Command sequence:

# 1. Check what's listening on port 53
$ sudo lsof -i :53 -n
systemd-r  751  systemd-resolve  UDP 127.0.0.53:53
named     5678  bind             UDP *:53
named     5678  bind             TCP *:53 (LISTEN)

# 2. Try to query those servers
$ dig @127.0.0.53 zhjlab.bd  # Works! systemd-resolved responds
$ dig @0.0.0.0 zhjlab.bd     # Works! BIND on all interfaces

# 3. Check BIND is actually running
$ sudo systemctl status bind9
# Shows PID 5678 - SAME as lsof output!

# 4. See recent queries in BIND logs
$ sudo journalctl -u bind9 -n 20
# Shows your dig commands being processed!
```

**The connection**: `lsof` shows WHAT'S listening, `dig` proves it works, `systemctl` confirms the service, `journalctl` shows the activity!

---

### Typical Lab Session Flow

Here's how a real DNS troubleshooting session ties all these commands together:

```bash
# ====== SCENARIO: DNS not working for zhjlab.bd ======

# Step 1: Can we even connect to the server?
$ ssh -p 2233 zhjewel@10.100.68.50
# If this fails, network problem, not DNS!

# Step 2: Is the DNS service running?
$ sudo systemctl status bind9
# If inactive → sudo systemctl start bind9

# Step 3: Is it actually listening?
$ sudo lsof -i :53 -n | grep named
# Nothing? Service isn't binding to port!
# Check /etc/bind/named.conf for errors

# Step 4: Try a local query
$ dig @localhost zhjlab.bd
# Works? Server is fine, problem is elsewhere

# Step 5: Check local DNS configuration
$ cat /etc/resolv.conf
# Does it point to the right server?
# Should show: nameserver 10.100.68.50 (or similar)

# Step 6: Check for local overrides
$ cat /etc/hosts | grep zhjlab
# Any entries here? They override DNS!

# Step 7: Test different query types
$ dig +t soa zhjlab.bd   # Authority info
$ dig +t ns zhjlab.bd    # Nameservers
$ dig +t a s00.zhjlab.bd # Actual host

# Step 8: Verify authoritative server responds
$ dig +t any s00.zhjlab.bd @dns1.du.ac.bd
# Check for 'aa' flag - confirms authoritative

# Step 9: Trace full resolution
$ dig +trace s00.zhjlab.bd
# See where it breaks: Root? TLD? Authoritative?

# Step 10: Check for recent errors
$ sudo journalctl -u bind9 -n 50
# Any error messages?

# ====== PROBLEM FOUND AND FIXED! ======
```

---

### Command Categories & Their Purposes

| Purpose | Commands | What They Teach |
|---------|----------|-----------------|
| **Remote Access** | `ssh -p 2233` | Access remote servers securely |
| **System Navigation** | `cd /etc`, `ls host[TAB]` | Find configuration files |
| **Config Reading** | `cat hosts`, `cat resolv.conf`, `cat host.conf` | Understand system DNS setup |
| **Service Management** | `systemctl status bind9` | Manage DNS server service |
| **Network Inspection** | `lsof -i -n`, `lsof -i :53` | See active network services |
| **Basic DNS Queries** | `host`, `dig` | Get DNS records |
| **Advanced Queries** | `dig +trace`, `dig @server`, `dig +t any` | Understand DNS hierarchy |
| **Record Types** | `+t soa`, `+t ns`, `+t a` | Learn different DNS record purposes |
| **Testing Authority** | `@dns1.du.ac.bd` | Query specific servers, verify authority |

---

## Summary

You've learned:
- ✅ How DNS works (the internet's phonebook)
- ✅ Three types of servers: Authoritative, Caching, Forwarding
- ✅ Every command from your lab notes
- ✅ How to troubleshoot DNS issues
- ✅ Real-world DNS scenarios
- ✅ **NEW**: How forwarding and caching servers differ
- ✅ **NEW**: Where each server type sits in the data flow
- ✅ **NEW**: Expected terminal outputs and how to interpret them
- ✅ **NEW**: How all commands connect to form a complete workflow

**Key Takeaway**: DNS is the glue that makes the internet user-friendly. Without it, we'd all be memorizing IP addresses like phone numbers!

The commands in your lab aren't isolated—they form a complete story of how DNS servers are configured, tested, and debugged. Each command reveals a piece of the puzzle, and together they show you the complete picture of DNS infrastructure.

---

## Additional Resources

- BIND Documentation: https://bind9.readthedocs.io
- DNS RFC 1034/1035: Original DNS specifications
- DNSViz: Visualize DNS hierarchies
- dig Command Man Page: `man dig`

---

*Notes created for Networking Lab Course*
*Topic: DNS Servers, Caching, and Forwarding*
