# Linux `ip` Command - Network Configuration Guide

## Table of Contents

1. [Introduction to the `ip` Command](#introduction-to-the-ip-command)
2. [Network Interface Management](#network-interface-management)
3. [IP Address Configuration](#ip-address-configuration)
4. [Routing Table Management](#routing-table-management)
5. [ARP Table Management](#arp-table-management)
6. [Complete Command Reference](#complete-command-reference)
7. [Practical Examples and Troubleshooting](#practical-examples-and-troubleshooting)

---

## Introduction to the `ip` Command

### What is the `ip` Command?

The `ip` command is a powerful networking tool in Linux that replaces older commands like `ifconfig`, `route`, and `arp`. It's part of the `iproute2` package and provides a unified interface for network configuration and management.

**Analogy**: Think of the `ip` command as a Swiss Army knife for networking. Instead of carrying separate tools for different jobs (ifconfig for interfaces, route for routing, arp for neighbors), you have one tool that does it all.

### Why Use `ip` Instead of Older Commands?

|Old Command|New `ip` Command|Why Change?|
|---|---|---|
|`ifconfig`|`ip addr`, `ip link`|More features, better output|
|`route`|`ip route`|Handles complex routing better|
|`arp`|`ip neigh`|More control over ARP cache|
|`netstat`|`ip route`, `ss`|Faster, more detailed|

**Important Note**: In the examples, `eth0` is used as the network interface name. Your interface might have a different name like:

- `eth1`, `eth2` (traditional Ethernet naming)
- `enp1s0`, `enp2s0` (PCI slot-based naming)
- `eno1`, `eno2` (onboard device naming)
- `wlan0`, `wlo1` (wireless interfaces)

Use `ip link` to see your actual interface names and substitute accordingly.

---

## Network Interface Management

### Understanding Network Interfaces

**What is a Network Interface?** A network interface is your computer's connection point to a network. It's like a door through which network traffic enters and exits your computer.

**Types of Interfaces**:

- **Physical**: Actual network cards (Ethernet, WiFi)
- **Virtual**: Software-created (loopback, bridges, VPNs)
- **Loopback (lo)**: Special interface for local communication (always `127.0.0.1`)

---

### 1.1. View All Network Interfaces

```bash
$ ip link
```

**What it does**: Lists all network interfaces on your system

**Expected Terminal Output**:

```bash
admin@mypc:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000 
    link/ether 00:1c:c0:b9:1d:2d brd ff:ff:ff:ff:ff:ff
```

**How to Interpret** (line by line):

**Interface 1: Loopback (lo)**

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN
```

- **`1:`** = Interface index number
- **`lo:`** = Interface name (loopback)
- **`<LOOPBACK,UP,LOWER_UP>`** = Flags (more below)
- **`mtu 16436`** = Maximum Transmission Unit (16KB for loopback)
- **`qdisc noqueue`** = Queue discipline (no queue for loopback)
- **`state UNKNOWN`** = Interface state

**Interface 2: Ethernet (eth0)**

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
```

- **`2:`** = Interface index number
- **`eth0:`** = Interface name (Ethernet)
- **`<BROADCAST,MULTICAST,UP,LOWER_UP>`** = Flags
- **`mtu 1500`** = Standard Ethernet MTU (1500 bytes)
- **`qdisc pfifo_fast`** = Queue discipline (first in, first out)
- **`state UP`** = Interface is active
- **`qlen 1000`** = Transmit queue length

**Understanding Flags**:

|Flag|Meaning|Example|
|---|---|---|
|**LOOPBACK**|Virtual loopback interface|Internal communication|
|**BROADCAST**|Can send broadcast packets|Network discovery|
|**MULTICAST**|Supports multicast|Streaming, group communication|
|**UP**|Interface is enabled|Ready to work|
|**LOWER_UP**|Physical layer is up|Cable is connected|
|**NO-CARRIER**|No physical connection|Cable unplugged|

**MAC Address Line**:

```
link/ether 00:1c:c0:b9:1d:2d brd ff:ff:ff:ff:ff:ff
```

- **`link/ether`** = Link layer type (Ethernet)
- **`00:1c:c0:b9:1d:2d`** = MAC address (hardware address, unique)
- **`brd ff:ff:ff:ff:ff:ff`** = Broadcast address (all nodes)

**Real-world analogy**: The MAC address is like your network card's serial number—it's unique and never changes (unless you manually change it). The broadcast address is like shouting in a room where everyone can hear.

---

### 1.2. View Specific Interface Details

```bash
$ ip link show dev eth0
```

**What it does**: Shows details for only the `eth0` interface

**Expected Terminal Output**:

```bash
admin@mypc:~$ ip link show dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000 
    link/ether 00:1c:c0:b9:1d:2d brd ff:ff:ff:ff:ff:ff
```

**When to use**:

- Quick check on one interface
- Writing scripts that need to parse output
- Verifying interface state after changes

**Practical example**:

```bash
# Check if WiFi interface is up
$ ip link show dev wlan0 | grep "state UP"
# If it returns something, WiFi is up. If not, it's down.
```

---

### 1.3. Bring Interface Up (Enable)

```bash
$ sudo ip link set eth0 up
```

**What it does**: Activates the network interface

**Why sudo?**: Changing interface state requires root privileges (system-level change)

**Expected Terminal Output**:

```bash
admin@mypc:~$ sudo ip link set eth0 up
[sudo] password for admin: ****
admin@mypc:~$
```

(No output means success!)

**What happens internally**:

```
Before: eth0: <BROADCAST,MULTICAST> state DOWN
After:  eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> state UP
```

**Verify it worked**:

```bash
$ ip link show dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
         ^^^^^^^^^^^^^^^^^^^^^^^^
         UP flag is now present!
```

**Real-world analogy**: This is like turning on a light switch. The power is there, but the light won't work until you flip the switch to "on".

**Common use cases**:

- After disabling interface for testing
- After changing network configuration
- Troubleshooting network connectivity

---

### 1.4. Bring Interface Down (Disable)

```bash
$ sudo ip link set eth0 down
```

**What it does**: Deactivates the network interface (disconnects from network)

**Expected Terminal Output**:

```bash
admin@mypc:~$ sudo ip link set eth0 down
admin@mypc:~$
```

**What happens**:

```
Before: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> state UP
After:  eth0: <BROADCAST,MULTICAST> state DOWN
                                      ^^^^^^^^^^
                                      No more UP flags!
```

**Verify it worked**:

```bash
$ ip link show dev eth0
2: eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast state DOWN qlen 1000
                                                          ^^^^^^^^^^
```

**Warning**: Running this on your active interface will disconnect you from the network!

**Common use cases**:

- Testing network redundancy
- Forcing interface to renegotiate DHCP
- Security (disconnect suspicious interface)
- Troubleshooting (isolate interface issues)

**Practical scenario**:

```bash
# Restart network interface (useful for applying changes)
$ sudo ip link set eth0 down
$ sudo ip link set eth0 up
# Interface will renegotiate connection, get new DHCP lease, etc.
```

---

## IP Address Configuration

### Understanding IP Addresses

**What is an IP Address?** An IP address is like your computer's mailing address on the network. Just as mail needs your street address to reach you, network packets need your IP address to reach your computer.

**IPv4 Format**: `172.16.1.100/24`

- **`172.16.1.100`** = The actual IP address (32-bit, 4 octets)
- **`/24`** = Subnet mask (CIDR notation)
    - Means: First 24 bits are network, last 8 bits are host
    - Traditional notation: `255.255.255.0`
    - Allows: 256 addresses (0-255) on this network

**IPv6 Format**: `fe80::xxxx:xxxx:xxxx:xxxx/64`

- 128-bit address (much longer than IPv4)
- Hexadecimal notation
- `::` means consecutive zeros (compression)

---

### 2.1. View All IP Addresses

```bash
$ ip addr
```

**What it does**: Shows IP configuration for all interfaces

**Expected Terminal Output**:

```bash
admin@mypc:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000 
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff 
    inet 172.16.1.100/24 brd 172.16.1.255 scope global eth0 
    inet6 fe80::xxxx:xxxx:xxxx:xxxx/64 scope link 
       valid_lft forever preferred_lft forever
```

**Detailed Interpretation**:

**Loopback Interface (lo)**:

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

- Physical layer info (no real hardware, it's virtual)

```
    inet 127.0.0.1/8 scope host lo
```

- **`inet`** = IPv4 address
- **`127.0.0.1/8`** = Loopback IP (always this)
- **`scope host`** = Only accessible from this computer
- **`lo`** = Interface name

```
    inet6 ::1/128 scope host
```

- **`inet6`** = IPv6 address
- **`::1/128`** = IPv6 loopback (equivalent to 127.0.0.1)

```
    valid_lft forever preferred_lft forever
```

- **`valid_lft`** = Valid lifetime (how long IP is valid)
- **`preferred_lft`** = Preferred lifetime (how long preferred for outgoing)
- **`forever`** = Never expires (static configuration)

**Ethernet Interface (eth0)**:

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
```

- Physical interface info

```
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
```

- MAC address and broadcast address

```
    inet 172.16.1.100/24 brd 172.16.1.255 scope global eth0
```

- **`inet 172.16.1.100/24`** = IPv4 address with /24 subnet
- **`brd 172.16.1.255`** = Broadcast address for this subnet
- **`scope global`** = Globally routable (can talk to other networks)
- **`eth0`** = Interface name

**Understanding Scope**:

|Scope|Meaning|Example|Can Access|
|---|---|---|---|
|**host**|Local only|127.0.0.1|Same computer|
|**link**|Local network|169.254.x.x|Same physical network|
|**global**|Routable|172.16.1.100|Internet/other networks|

**Understanding Subnet /24**:

```
IP: 172.16.1.100/24

Breaking down /24:
┌─────────┬─────────┬─────────┬─────────┐
│   172   │   16    │    1    │   100   │
└─────────┴─────────┴─────────┴─────────┘
    ↑         ↑         ↑         ↑
    └─────────┴─────────┴─────────┘───────┐
            24 bits                8 bits  │
         (Network part)          (Host)    │
                                           │
Network: 172.16.1.0                        │
First usable: 172.16.1.1 (gateway)         │
Last usable: 172.16.1.254                  │
Broadcast: 172.16.1.255 ←──────────────────┘
Total hosts: 254 (256 - 2 for network and broadcast)
```

---

### 2.2. View Specific Interface IP Configuration

```bash
$ ip addr show dev eth0
```

**What it does**: Shows IP configuration for only `eth0`

**Expected Terminal Output**:

```bash
admin@mypc:~$ ip addr show dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000 
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff 
    inet 172.16.1.100/24 brd 172.16.1.255 scope global eth0 
    inet6 fe80::xxxx:xxxx:xxxx:xxxx/64 scope link 
       valid_lft forever preferred_lft forever
```

**When to use**:

- Quick IP address lookup for specific interface
- Scripting (parsing is easier with less output)
- Troubleshooting one specific interface

**Quick lookup tricks**:

```bash
# Get just the IPv4 address
$ ip addr show dev eth0 | grep "inet " | awk '{print $2}'
172.16.1.100/24

# Get just the IP without subnet mask
$ ip addr show dev eth0 | grep "inet " | awk '{print $2}' | cut -d/ -f1
172.16.1.100

# Get MAC address
$ ip addr show dev eth0 | grep "link/ether" | awk '{print $2}'
xx:xx:xx:xx:xx:xx
```

---

### 2.3. Add IP Address to Interface

```bash
$ sudo ip addr add 172.16.1.100/24 dev eth0
```

**What it does**: Assigns an IP address to the interface

**Breakdown**:

- **`ip addr add`** = Add IP address command
- **`172.16.1.100/24`** = IP address with subnet mask
- **`dev eth0`** = To interface eth0

**Expected Terminal Output**:

```bash
admin@mypc:~$ sudo ip addr add 172.16.1.100/24 dev eth0
[sudo] password for admin: ****
admin@mypc:~$
```

(No output = success!)

**Verify it worked**:

```bash
$ ip addr show dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000 
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff 
    inet 172.16.1.100/24 brd 172.16.1.255 scope global eth0
         ^^^^^^^^^^^^^^^^
         New IP is now assigned!
       valid_lft forever preferred_lft forever
```

**Important Notes**:

1. **Multiple IPs**: You can add multiple IPs to one interface!

```bash
$ sudo ip addr add 172.16.1.100/24 dev eth0
$ sudo ip addr add 172.16.1.101/24 dev eth0
# Now eth0 has TWO IP addresses!
```

2. **Temporary**: This change is NOT permanent! It's lost on reboot.
    
3. **DHCP Conflict**: If interface uses DHCP, manually added IPs might conflict.
    

**Real-world use cases**:

- Testing network configurations
- Running multiple services on different IPs
- Virtual hosting (web servers)
- Network troubleshooting

**Practical example - Web server with multiple IPs**:

```bash
# Add multiple IPs for different websites
$ sudo ip addr add 192.168.1.100/24 dev eth0  # website1.com
$ sudo ip addr add 192.168.1.101/24 dev eth0  # website2.com
$ sudo ip addr add 192.168.1.102/24 dev eth0  # website3.com
# Each website can now have its own IP!
```

---

### 2.4. Remove IP Address from Interface

```bash
$ sudo ip addr del 172.16.1.100/24 dev eth0
```

**What it does**: Removes an IP address from the interface

**Expected Terminal Output**:

```bash
admin@mypc:~$ sudo ip addr del 172.16.1.100/24 dev eth0
admin@mypc:~$
```

**Verify it's gone**:

```bash
$ ip addr show dev eth0 | grep "172.16.1.100"
# No output = IP successfully removed
```

**Warning**: Removing the active IP will disconnect you from the network!

**Safe way to change IP**:

```bash
# Add new IP first
$ sudo ip addr add 172.16.1.200/24 dev eth0
# Verify it works (test connection)
$ ping -c 1 172.16.1.1
# Only then remove old IP
$ sudo ip addr del 172.16.1.100/24 dev eth0
```

**Common mistake**:

```bash
# WRONG - Will disconnect you immediately
$ sudo ip addr del 172.16.1.100/24 dev eth0

# RIGHT - Add new IP first
$ sudo ip addr add 172.16.1.200/24 dev eth0
$ sudo ip addr del 172.16.1.100/24 dev eth0
```

---

## Routing Table Management

### Understanding Routing

**What is Routing?** Routing is how your computer decides where to send network packets. Think of it like a GPS for network traffic—it determines the best path for data to reach its destination.

**Analogy**: Imagine you're sending a letter:

- **Local delivery**: Address on your street → Hand deliver (same network)
- **Different city**: Address in another city → Send to post office → They forward it (routing through gateway)

**The Routing Table**: A list of rules that tells your computer:

- "For this network, send packets directly"
- "For that network, send through this gateway"
- "For everything else (internet), send to default gateway"

---

### 3.1. View Routing Table

```bash
$ ip route
```

**What it does**: Displays the kernel routing table

**Expected Terminal Output**:

```bash
admin@mypc:~$ ip route
default via 172.16.1.1 dev eno1 proto dhcp metric 100 
169.254.0.0/16 dev eno1 scope link metric 1000 
172.16.1.0/24 dev eno1 proto kernel scope link src 172.16.1.100 metric 100
```

**How to Interpret Each Line**:

**Line 1: Default Route (Gateway to Internet)**

```
default via 172.16.1.1 dev eno1 proto dhcp metric 100
```

- **`default`** = For any destination not in other routes (0.0.0.0/0)
- **`via 172.16.1.1`** = Send packets through this gateway (usually your router)
- **`dev eno1`** = Use interface eno1
- **`proto dhcp`** = Route learned from DHCP server
- **`metric 100`** = Priority (lower = preferred)

**Translation**: "For anything not on local network, send to 172.16.1.1 (my router)"

**Line 2: Link-Local Route**

```
169.254.0.0/16 dev eno1 scope link metric 1000
```

- **`169.254.0.0/16`** = Link-local addressing (automatic when no DHCP)
- **`dev eno1`** = Directly on interface eno1
- **`scope link`** = Only valid on local link (can't route through gateway)
- **`metric 1000`** = Low priority route

**Translation**: "For 169.254.x.x addresses, send directly on local network (fallback addressing)"

**Line 3: Local Network Route**

```
172.16.1.0/24 dev eno1 proto kernel scope link src 172.16.1.100 metric 100
```

- **`172.16.1.0/24`** = Local network (all addresses 172.16.1.0-255)
- **`dev eno1`** = Directly connected on eno1
- **`proto kernel`** = Created automatically by kernel when IP assigned
- **`scope link`** = Local network scope
- **`src 172.16.1.100`** = Use this as source IP for packets
- **`metric 100`** = Priority

**Translation**: "For 172.16.1.0-255, send directly—they're on my local network"

---

### Understanding Metrics

**What is a Metric?** When multiple routes exist to the same destination, metric determines which route to use.

**Lower metric = Higher priority (preferred route)**

**Example with multiple routes**:

```bash
$ ip route
default via 172.16.1.1 dev eth0 metric 100    # Wired connection
default via 192.168.1.1 dev wlan0 metric 600  # WiFi connection
```

**Result**: System prefers wired connection (metric 100) over WiFi (metric 600)

---

### Route Types Explained

|Route Type|Description|Example|
|---|---|---|
|**Default**|Catch-all for unknown destinations|Internet gateway|
|**Network**|Route to specific network|192.168.1.0/24|
|**Host**|Route to specific computer|192.168.1.50/32|
|**Link-local**|Automatic local addressing|169.254.0.0/16|

---

### 3.2. Add Default Route (Gateway)

```bash
$ sudo ip route add 0.0.0.0/0 via 172.16.1.1 dev eth0
```

**OR**

```bash
$ sudo ip route add default via 172.16.1.1 dev eth0
```

**What it does**: Sets the default gateway (router) for internet access

**Breakdown**:

- **`0.0.0.0/0`** or **`default`** = All destinations (entire internet)
- **`via 172.16.1.1`** = Through this gateway IP (your router)
- **`dev eth0`** = Using interface eth0

**Expected Terminal Output**:

```bash
admin@mypc:~$ sudo ip route add default via 172.16.1.1 dev eth0
admin@mypc:~$
```

**Verify it worked**:

```bash
$ ip route | grep default
default via 172.16.1.1 dev eth0
```

**Test connectivity**:

```bash
$ ping -c 2 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=12.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=11.8 ms
# Success! Can reach internet
```

**What happens without default route?**

```bash
$ ping 8.8.8.8
ping: connect: Network is unreachable
# Can't reach internet—no route to get there!
```

**Real-world analogy**: The default route is like knowing where the nearest post office is. If you want to send mail anywhere in the world, you take it to the post office (gateway) and they handle the rest.

---

### 3.3. Delete Default Route

```bash
$ sudo ip route del 0.0.0.0/0 via 172.16.1.1 dev eth0
```

**OR**

```bash
$ sudo ip route del default via 172.16.1.1 dev eth0
```

**What it does**: Removes the default gateway

**Expected Terminal Output**:

```bash
admin@mypc:~$ sudo ip route del default via 172.16.1.1 dev eth0
admin@mypc:~$
```

**Verify it's gone**:

```bash
$ ip route | grep default
# No output = default route removed
```

**Result**: You can no longer reach the internet (or any non-local network)!

**When to use**:

- Testing network issues
- Forcing traffic through specific interface
- Security isolation
- Before adding new default route

---

### 3.4. Add Static Route

```bash
$ sudo ip route add 172.16.100.0/24 via 172.16.1.12
```

**What it does**: Creates a route to a specific network through a gateway

**Breakdown**:

- **`172.16.100.0/24`** = Destination network (all IPs 172.16.100.0-255)
- **`via 172.16.1.12`** = Through this gateway/router

**Expected Terminal Output**:

```bash
admin@mypc:~$ sudo ip route add 172.16.100.0/24 via 172.16.1.12
admin@mypc:~$
```

**Verify it worked**:

```bash
$ ip route | grep "172.16.100"
172.16.100.0/24 via 172.16.1.12 dev eth0
```

**Real-world scenario**:

Imagine this network topology:

```
Your Computer          Router 1           Router 2           Remote Network
172.16.1.100 ────► 172.16.1.12 ────► 172.16.1.254 ────► 172.16.100.0/24
    (you)           (gateway)                           (destination)
```

Without the static route:

```bash
$ ping 172.16.100.50
# Fails! Your computer doesn't know how to reach 172.16.100.0/24
```

With the static route:

```bash
$ sudo ip route add 172.16.100.0/24 via 172.16.1.12
$ ping 172.16.100.50
# Success! Traffic goes through 172.16.1.12 to reach destination
```

**Use cases**:

- Multiple networks in company
- VPN connections
- Complex network topologies
- Traffic engineering

**Example - Multi-site company**:

```bash
# Office 1: 192.168.1.0/24 (your office)
# Office 2: 192.168.2.0/24 (different location)
# Gateway to Office 2: 192.168.1.254

$ sudo ip route add 192.168.2.0/24 via 192.168.1.254
# Now you can access computers in Office 2!
```

---

### 3.5. Delete Static Route

```bash
$ sudo ip route del 172.16.100.0/24 via 172.16.1.12
```

**What it does**: Removes the static route

**Expected Terminal Output**:

```bash
admin@mypc:~$ sudo ip route del 172.16.100.0/24 via 172.16.1.12
admin@mypc:~$
```

**Verify it's gone**:

```bash
$ ip route | grep "172.16.100"
# No output = route removed
```

---

### 3.6. Add Connected Route

```bash
$ sudo ip route add 172.16.1.0/24 dev eth0
```

**What it does**: Creates a direct route to a network (no gateway needed)

**What's "Connected"?** A connected route means the network is directly accessible—physically connected to your interface, no router needed.

**Expected Terminal Output**:

```bash
admin@mypc:~$ sudo ip route add 172.16.1.0/24 dev eth0
admin@mypc:~$
```

**Verify**:

```bash
$ ip route | grep "172.16.1.0"
172.16.1.0/24 dev eth0 scope link
                       ^^^^^^^^^^^
                       "scope link" = directly connected
```

**Difference from Static Route**:

**Connected Route** (no gateway):

```bash
$ sudo ip route add 172.16.1.0/24 dev eth0
# "Traffic to 172.16.1.0/24 goes directly out eth0"
```

**Static Route** (with gateway):

```bash
$ sudo ip route add 172.16.100.0/24 via 172.16.1.12
# "Traffic to 172.16.100.0/24 goes through 172.16.1.12 first"
```

**When is this automatically created?** Usually created automatically when you assign an IP address:

```bash
$ sudo ip addr add 172.16.1.100/24 dev eth0
# Kernel automatically creates:
# 172.16.1.0/24 dev eth0 proto kernel scope link src 172.16.1.100
```

**Manual use cases**:

- After deleting automatic route
- Custom routing scenarios
- Network testing

---

### 3.7. Delete Connected Route

```bash
$ sudo ip route del 172.16.1.0/24 dev eth0
```

**What it does**: Removes the connected route

**Expected Terminal Output**:

```bash
admin@mypc:~$ sudo ip route del 172.16.1.0/24 dev eth0
admin@mypc:~$
```

**Warning**: This will break local network connectivity!

**What breaks**:

```bash
$ ping 172.16.1.1
ping: connect: Network is unreachable
# Can't reach local network anymore!
```

---

### 3.8. Check Route to Specific IP

```bash
$ ip route get 8.8.8.8
```

**What it does**: Shows which route will be used to reach a specific IP

**Expected Terminal Output**:

```bash
admin@mypc:~$ ip route get 8.8.8.8
8.8.8.8 via 172.16.1.1 dev eno1 src 172.16.1.100 uid 1000 
    cache
```

**How to Interpret**:

- **`8.8.8.8`** = Destination IP (Google DNS)
- **`via 172.16.1.1`** = Will go through this gateway
- **`dev eno1`** = Will use interface eno1
- **`src 172.16.1.100`** = Will use this as source IP
- **`uid 1000`** = User ID that made the query
- **`cache`** = Result is cached (faster next lookup)

**Translation**: "To reach 8.8.8.8, I'll send packets from 172.16.1.100 through 172.16.1.1 using eno1"

**Practical troubleshooting examples**:

**Example 1: Check if traffic goes through VPN**

```bash
# Before VPN connection
$ ip route get 8.8.8.8
8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.100
        ^^^^^^^^^^^^^^^^^^^^
        Going through regular gateway

# After VPN connection
$ ip route get 8.8.8.8
8.8.8.8 via 10.8.0.1 dev tun0 src 10.8.0.2
        ^^^^^^^^^^^^^^^^^
        Now going through VPN tunnel!
```

**Example 2: Check local vs remote routing**

```bash
# Local network IP
$ ip route get 172.16.1.50
172.16.1.50 dev eth0 src 172.16.1.100 
    ^^^^^^^^^^^^
    Direct (no gateway) - it's local!

# Remote IP
$ ip route get 8.8.8.8
8.8.8.8 via 172.16.1.1 dev eth0 src 172.16.1.100
        ^^^^^^^^^^^^^^^
        Goes through gateway - it's remote!
```

**Example 3: Verify static route works**

```bash
# After adding: ip route add 172.16.100.0/24 via 172.16.1.12
$ ip route get 172.16.100.50
172.16.100.50 via 172.16.1.12 dev eth0 src 172.16.1.100
              ^^^^^^^^^^^^^^^
              Using our static route! ✓
```

---

## ARP Table Management

### Understanding ARP (Address Resolution Protocol)

**What is ARP?** ARP is how your computer finds out the MAC address (physical hardware address) of another computer when it only knows the IP address.

**Analogy**: You know someone's phone number (IP address) but need to know which physical phone (MAC address) to call. ARP is like a directory that maps phone numbers to physical devices.

**Why do we need both IP and MAC?**

- **IP address** = Logical address (can change, used for routing across networks)
- **MAC address** = Physical address (burned into network card, used for local delivery)

**The ARP process**:

```
1. Computer A wants to send data to 192.168.1.50
2. Computer A checks ARP table: "Do I know the MAC for 192.168.1.50?"
3. Not in table → Send ARP broadcast: "Who has 192.168.1.50?"
4. Computer at 192.168.1.50 replies: "I do! My MAC is aa:bb:cc:dd:ee:ff"
5. Computer A stores this in ARP table
6. Future packets to 192.168.1.50 use MAC aa:bb:cc:dd:ee:ff directly
```

**Real-world analogy**:

- IP address = "123 Main Street" (where you want to send)
- MAC address = "The house with the blue door" (physical identifier)
- ARP = The process of looking up which house has which address

---

### 4.1. View ARP Table

```bash
$ ip neigh
```

(Note: "neigh" is short for "neighbor" - another device on your local network)

**What it does**: Shows the ARP cache (IP to MAC address mappings)

**Expected Terminal Output**:

```bash
admin@mypc:~$ ip neigh
192.168.88.1 dev wlo1 lladdr 4c:5e:0c:2d:1a:25 STALE
192.168.88.1 dev eno1 lladdr 4c:5e:0c:2d:1a:25 STALE
```

**How to Interpret**:

```
192.168.88.1 dev wlo1 lladdr 4c:5e:0c:2d:1a:25 STALE
     ↑          ↑         ↑                      ↑
     IP      Interface  MAC Address          State
```

- **`192.168.88.1`** = IP address of neighbor (probably your router)
- **`dev wlo1`** = Reachable through interface wlo1 (wireless)
- **`lladdr 4c:5e:0c:2d:1a:25`** = Link Layer Address (MAC address)
- **`STALE`** = ARP entry state (more below)

**Understanding ARP States**:

|State|Meaning|What's Happening|
|---|---|---|
|**REACHABLE**|✅ Fresh, recently confirmed|Just communicated with this device|
|**STALE**|⏳ Old but not expired|Haven't talked to it lately, might verify next time|
|**DELAY**|⏲️ Waiting for confirmation|Checking if device is still there|
|**PROBE**|🔍 Actively probing|Sending ARP requests to verify|
|**FAILED**|❌ Unreachable|Can't reach this device|
|**PERMANENT**|📌 Manual entry|Won't expire (manually added)|

**Lifecycle of an ARP entry**:

```
INCOMPLETE → Sending ARP request
     ↓
REACHABLE → Got reply! Entry is fresh (valid for ~30 seconds)
     ↓
STALE → Not used recently (valid for ~60 seconds)
     ↓
DELAY → Used again, verifying it's still valid
     ↓
PROBE → Sending ARP request to confirm
     ↓
REACHABLE (if reply) or FAILED (if no reply)
```

**Why same IP appears twice?**

```bash
192.168.88.1 dev wlo1 lladdr 4c:5e:0c:2d:1a:25 STALE
192.168.88.1 dev eno1 lladdr 4c:5e:0c:2d:1a:25 STALE
```

- Same router (192.168.88.1) reachable through two interfaces
- WiFi (wlo1) and Ethernet (eno1) both connected
- Same MAC address because it's the same physical device

---

### 4.2. View ARP Table for Specific Interface

```bash
$ ip neigh show dev eno1
```

**What it does**: Shows ARP entries only for interface eno1

**Expected Terminal Output**:

```bash
admin@mypc:~$ ip neigh show dev eno1
192.168.88.1 lladdr 4c:5e:0c:2d:1a:25 REACHABLE
```

**Notice the state changed from STALE to REACHABLE**:

- Likely because we just pinged the router
- Or sent some traffic that confirmed it's alive
- Entry is fresh now

**Use cases**:

- Troubleshooting specific interface
- Checking if device is reachable on particular network
- Monitoring specific connection

**Practical example**:

```bash
# Check if gateway is in ARP table
$ ip neigh show dev eth0 | grep "192.168.1.1"
192.168.1.1 lladdr aa:bb:cc:dd:ee:ff REACHABLE
# Found it! And it's reachable.

# If not found:
$ ping -c 1 192.168.1.1  # Generate traffic to populate ARP
$ ip neigh show dev eth0 | grep "192.168.1.1"
# Should appear now
```

---

### 4.3. Add Static ARP Entry

```bash
$ sudo ip neigh add 192.168.88.200 lladdr 1:2:3:4:5:6 dev eno1
```

**What it does**: Manually adds an IP-to-MAC mapping to ARP table

**Breakdown**:

- **`neigh add`** = Add neighbor (ARP entry)
- **`192.168.88.200`** = IP address
- **`lladdr 1:2:3:4:5:6`** = Link layer address (MAC address)
- **`dev eno1`** = On interface eno1

**Expected Terminal Output**:

```bash
admin@mypc:~$ sudo ip neigh add 192.168.88.200 lladdr 1:2:3:4:5:6 dev eno1
admin@mypc:~$
```

**Verify**:

```bash
$ ip neigh show dev eno1 | grep "192.168.88.200"
192.168.88.200 lladdr 01:02:03:04:05:06 PERMANENT dev eno1
                                        ^^^^^^^^^
                                        Won't expire!
```

**When to use static ARP entries**:

**1. Security (ARP Spoofing Prevention)**:

```bash
# Lock down your gateway's MAC address
$ sudo ip neigh add 192.168.1.1 lladdr aa:bb:cc:dd:ee:ff dev eth0
# Now attackers can't fake being your gateway (ARP spoofing attack)
```

**2. Static IP Devices**:

```bash
# For devices with fixed IPs (servers, printers)
$ sudo ip neigh add 192.168.1.50 lladdr 00:11:22:33:44:55 dev eth0
# Avoid ARP lookup every time (slightly faster)
```

**3. Troubleshooting**:

```bash
# Force traffic to specific MAC when testing
```

**Real-world scenario - Prevent ARP Spoofing**:

**Without static ARP (vulnerable)**:

```bash
# Your ARP table
192.168.1.1 lladdr aa:bb:cc:dd:ee:ff  # Real router

# Attacker sends fake ARP reply
# ARP table gets poisoned
192.168.1.1 lladdr 99:99:99:99:99:99  # Attacker's MAC!

# Your traffic now goes to attacker instead of real router!
```

**With static ARP (protected)**:

```bash
$ sudo ip neigh add 192.168.1.1 lladdr aa:bb:cc:dd:ee:ff dev eth0
# ARP entry is PERMANENT, can't be changed by fake replies
# Attacker's fake ARP replies are ignored ✓
```

---

### 4.4. Delete ARP Entry

```bash
$ sudo ip neigh del 192.168.88.200 lladdr 1:2:3:4:5:6 dev eno1
```

**What it does**: Removes ARP entry from table

**Expected Terminal Output**:

```bash
admin@mypc:~$ sudo ip neigh del 192.168.88.200 lladdr 1:2:3:4:5:6 dev eno1
admin@mypc:~$
```

**Verify it's gone**:

```bash
$ ip neigh show dev eno1 | grep "192.168.88.200"
# No output = entry removed
```

**When to use**:

- Remove incorrect manual entries
- Force ARP cache refresh (delete, system will re-learn)
- Troubleshooting connectivity issues

**Troubleshooting with ARP deletion**:

**Problem**: Can't connect to 192.168.1.50

```bash
# Check ARP table
$ ip neigh | grep "192.168.1.50"
192.168.1.50 lladdr aa:bb:cc:dd:ee:ff STALE
# Entry exists but might be wrong

# Delete it
$ sudo ip neigh del 192.168.1.50 dev eth0

# Try connecting again
$ ping 192.168.1.50
# System will send new ARP request
# Might get correct MAC address this time

# Check new entry
$ ip neigh | grep "192.168.1.50"
192.168.1.50 lladdr ff:ee:dd:cc:bb:aa REACHABLE
# Different MAC! Old entry was wrong
```

---

### 4.5. Replace ARP Entry

```bash
$ sudo ip neigh replace 192.168.88.200 lladdr 1:2:3:4:5:7 dev eno1
```

**What it does**: Updates existing ARP entry or creates if it doesn't exist

**Expected Terminal Output**:

```bash
admin@mypc:~$ sudo ip neigh replace 192.168.88.200 lladdr 1:2:3:4:5:7 dev eno1
admin@mypc:~$
```

**Verify**:

```bash
$ ip neigh show dev eno1 | grep "192.168.88.200"
192.168.88.200 lladdr 01:02:03:04:05:07 PERMANENT dev eno1
                              ↑
                              Changed from :06 to :07
```

**Difference between `add`, `del`, and `replace`**:

|Command|Existing Entry|Behavior|
|---|---|---|
|**add**|Exists|❌ Error: "File exists"|
|**add**|Doesn't exist|✅ Creates new entry|
|**del**|Exists|✅ Deletes entry|
|**del**|Doesn't exist|❌ Error: "No such file"|
|**replace**|Exists|✅ Updates entry|
|**replace**|Doesn't exist|✅ Creates new entry|

**When to use `replace`**:

- In scripts (don't know if entry exists)
- Updating security settings
- Changing configuration without checking first

**Example script using replace**:

```bash
#!/bin/bash
# Lock down gateway MAC (works whether entry exists or not)
GATEWAY_IP="192.168.1.1"
GATEWAY_MAC="aa:bb:cc:dd:ee:ff"
INTERFACE="eth0"

sudo ip neigh replace $GATEWAY_IP lladdr $GATEWAY_MAC dev $INTERFACE
echo "Gateway ARP entry secured!"
```

---

## Complete Command Reference

### Quick Command Cheatsheet

#### Link (Interface) Management

```bash
# View all interfaces
ip link

# View specific interface
ip link show dev eth0

# Enable interface
sudo ip link set eth0 up

# Disable interface
sudo ip link set eth0 down

# Change MTU
sudo ip link set eth0 mtu 9000

# Change MAC address (spoofing)
sudo ip link set eth0 address aa:bb:cc:dd:ee:ff
```

#### Address Management

```bash
# View all IP addresses
ip addr
ip a  # Short form

# View specific interface
ip addr show dev eth0

# Add IP address
sudo ip addr add 192.168.1.100/24 dev eth0

# Add IP with broadcast
sudo ip addr add 192.168.1.100/24 brd + dev eth0

# Delete IP address
sudo ip addr del 192.168.1.100/24 dev eth0

# Flush all IPs from interface
sudo ip addr flush dev eth0
```

#### Routing Management

```bash
# View routing table
ip route
ip r  # Short form

# Add default route
sudo ip route add default via 192.168.1.1 dev eth0

# Add network route
sudo ip route add 10.0.0.0/8 via 192.168.1.254

# Add host route
sudo ip route add 192.168.2.50/32 via 192.168.1.254

# Delete route
sudo ip route del 10.0.0.0/8

# Get route to specific IP
ip route get 8.8.8.8

# Flush all routes
sudo ip route flush table main
```

#### ARP/Neighbor Management

```bash
# View ARP table
ip neigh
ip n  # Short form

# View specific interface
ip neigh show dev eth0

# Add static ARP entry
sudo ip neigh add 192.168.1.50 lladdr aa:bb:cc:dd:ee:ff dev eth0

# Delete ARP entry
sudo ip neigh del 192.168.1.50 dev eth0

# Replace ARP entry
sudo ip neigh replace 192.168.1.50 lladdr aa:bb:cc:dd:ee:ff dev eth0

# Flush ARP table
sudo ip neigh flush dev eth0
```

---

## Practical Examples and Troubleshooting

### Example 1: Setting Up a Static IP Configuration

**Scenario**: Configure static IP on eth0: 192.168.1.100/24, gateway 192.168.1.1

```bash
# Step 1: Bring interface up
$ sudo ip link set eth0 up

# Step 2: Assign IP address
$ sudo ip addr add 192.168.1.100/24 dev eth0

# Step 3: Add default route
$ sudo ip route add default via 192.168.1.1 dev eth0

# Step 4: Verify configuration
$ ip addr show dev eth0
$ ip route

# Step 5: Test connectivity
$ ping -c 3 192.168.1.1  # Test gateway
$ ping -c 3 8.8.8.8       # Test internet
```

**Making it permanent** (Ubuntu/Debian):

```bash
$ sudo nano /etc/netplan/01-netcfg.yaml

network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

$ sudo netplan apply
```

---

### Example 2: Multi-Homed Server (Two Networks)

**Scenario**: Server connected to two networks

- eth0: 192.168.1.100/24 (internal network)
- eth1: 10.0.0.100/24 (external network)

```bash
# Configure eth0 (internal)
$ sudo ip link set eth0 up
$ sudo ip addr add 192.168.1.100/24 dev eth0
$ sudo ip route add 192.168.1.0/24 dev eth0

# Configure eth1 (external)
$ sudo ip link set eth1 up
$ sudo ip addr add 10.0.0.100/24 dev eth1
$ sudo ip route add default via 10.0.0.1 dev eth1

# Verify
$ ip addr
$ ip route
```

**Result**:

```
Internal traffic (192.168.1.x) → Goes directly via eth0
External traffic (internet)     → Goes via eth1 to 10.0.0.1
```

---

### Example 3: Temporary Network Testing

**Scenario**: Test configuration without affecting production setup

```bash
# Save current configuration
$ ip addr show eth0 > /tmp/eth0-config.txt
$ ip route > /tmp/routes.txt

# Test new configuration
$ sudo ip addr add 192.168.10.50/24 dev eth0
$ sudo ip route add 192.168.10.0/24 dev eth0
$ ping 192.168.10.1  # Test

# If it doesn't work, restore
$ sudo ip addr del 192.168.10.50/24 dev eth0
$ sudo ip route del 192.168.10.0/24 dev eth0

# Original config automatically restored
```

---

### Example 4: Troubleshooting No Internet Connection

```bash
# Step 1: Check interface is up
$ ip link show eth0
# Look for: state UP

# Step 2: Check IP address assigned
$ ip addr show dev eth0
# Should show: inet 192.168.x.x/24

# Step 3: Check default route exists
$ ip route | grep default
# Should show: default via 192.168.x.1

# Step 4: Test gateway reachable
$ ping -c 3 192.168.1.1
# Should get replies

# Step 5: Check DNS
$ ping -c 3 8.8.8.8
# Works? DNS problem
# Doesn't work? Routing problem

# Step 6: Check ARP for gateway
$ ip neigh | grep 192.168.1.1
# Should show: REACHABLE

# Step 7: Trace route
$ ip route get 8.8.8.8
# Shows the path packets will take
```

---

### Example 5: Creating a Bridge Between Two Networks

**Scenario**: Connect 192.168.1.0/24 and 192.168.2.0/24

**Computer acting as router**:

- eth0: 192.168.1.254/24
- eth1: 192.168.2.254/24

```bash
# Configure interfaces
$ sudo ip addr add 192.168.1.254/24 dev eth0
$ sudo ip addr add 192.168.2.254/24 dev eth1
$ sudo ip link set eth0 up
$ sudo ip link set eth1 up

# Enable IP forwarding
$ sudo sysctl -w net.ipv4.ip_forward=1

# On computers in network 1 (192.168.1.x):
$ sudo ip route add 192.168.2.0/24 via 192.168.1.254

# On computers in network 2 (192.168.2.x):
$ sudo ip route add 192.168.1.0/24 via 192.168.2.254

# Test: From 192.168.1.100 → ping 192.168.2.100
```

---

### Troubleshooting Tips

#### Problem: "Network is unreachable"

```bash
# Usually missing default route
$ ip route | grep default
# If nothing, add one:
$ sudo ip route add default via 192.168.1.1 dev eth0
```

#### Problem: "Cannot assign requested address"

```bash
# Interface might be down
$ sudo ip link set eth0 up
# Then add IP
$ sudo ip addr add 192.168.1.100/24 dev eth0
```

#### Problem: Slow network after configuration

```bash
# Check MTU - might be mismatched
$ ip link show eth0 | grep mtu
# Set correct MTU
$ sudo ip link set eth0 mtu 1500
```

#### Problem: ARP issues ("Destination Host Unreachable")

```bash
# Check ARP table
$ ip neigh
# If entry is FAILED or missing:
$ sudo ip neigh flush dev eth0  # Clear ARP cache
$ ping 192.168.1.1  # Regenerate ARP entry
```

---

## Important Notes

### Persistence Warning

⚠️ **CRITICAL**: All `ip` command changes are **TEMPORARY**!

They are lost when:

- System reboots
- Interface goes down and up
- Network service restarts

**To make permanent**:

- **Ubuntu/Debian**: Use `/etc/netplan/` configuration
- **RHEL/CentOS**: Use `/etc/sysconfig/network-scripts/`
- **All distros**: Use `NetworkManager` or `systemd-networkd`

---

### Common Interface Names

Modern Linux uses "Predictable Network Interface Names":

|Name Pattern|Type|Example|
|---|---|---|
|**lo**|Loopback|lo|
|**eth***|Ethernet (old naming)|eth0, eth1|
|**eno***|Onboard device|eno1, eno2|
|**ens***|Hotplug slot|ens33|
|**enp**_s_**|PCI location|enp0s3, enp2s0|
|**wlan***|Wireless (old naming)|wlan0|
|**wlo***|Wireless (new naming)|wlo1|

**Find your interface names**:

```bash
$ ip link
# Look at the interface names in the output
```

---

### Safety Tips

1. **Always test before making permanent**
2. **Keep a terminal open when configuring remotely** (in case you lock yourself out)
3. **Document your changes** (save output to files)
4. **Use `ip route get` to verify** before making routing changes
5. **Have physical/console access** when experimenting with network config

---

## Summary

### Key Takeaways

✅ **`ip` command is the modern networking tool** - replaces ifconfig, route, arp ✅ **Four main categories**: link (interfaces), addr (IP addresses), route (routing), neigh (ARP) ✅ **Changes are temporary** - use config files for persistence ✅ **Always verify after changes** - use show/get commands ✅ **Safety first** - test before committing, keep backup access

### Command Hierarchy

```
ip
├── link      → Physical interfaces (up/down, MAC, MTU)
├── addr      → IP addresses (add/del IPv4/IPv6)
├── route     → Routing table (default, static, connected)
└── neigh     → ARP table (MAC address resolution)
```

### Learning Path Suggestion

1. **Start with**: `ip link` and `ip addr` (understand your interfaces)
2. **Then**: `ip route` (understand routing)
3. **Finally**: `ip neigh` (understand ARP)
4. **Practice**: Set up test networks in VMs
5. **Master**: Troubleshooting real-world scenarios

---

## Additional Resources

- Man pages: `man ip`, `man ip-link`, `man ip-route`
- iproute2 documentation: https://wiki.linuxfoundation.org/networking/iproute2
- Practice in virtual machines before production
- Join networking communities for help

---

_Notes created for Linux Networking Lab Course_ _Topic: `ip` Command - Network Configuration and Management_ _Reference: Net-ip_COMMAND -- MZH by Mohammad Zulfikar Hafizu (Jewel)_