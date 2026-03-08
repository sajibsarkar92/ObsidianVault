This guide outlines the complete process of transforming an **Ubuntu** server into an **Authoritative DNS Name Server** using BIND9.

---

## 1. Installation & Initial Setup

First, install the BIND9 suite. In the Debian/Ubuntu ecosystem, the service name is `bind9`.

Bash

```
sudo apt update
sudo apt install bind9 bind9utils bind9-doc -y
```

### Enable Recursive Resolution (Optional but Recommended)

To allow your server to resolve external domains (like google.com) while also managing your local zone, configure **forwarders**.

**File:** `/etc/bind/named.conf.options`

Plaintext

```
options {
    directory "/var/cache/bind";

    forwarders {
        8.8.8.8;
        1.1.1.1;
    };

    dnssec-validation auto;
    listen-on-v6 { any; };
};
```

- **Forwarders:** Upstream servers BIND asks when it doesn't know an answer.
    
- **Restart Service:** `sudo systemctl restart bind9`
    

---

## 2. Declaring the Custom Zone

You must tell BIND that it is the "Master" (the primary source of truth) for your specific domain. This is done in the local configuration file.

**File:** `/etc/bind/named.conf.local`

Plaintext

```
zone "zhjlab.bd" {
    type master;
    file "/etc/bind/db.zhjlab.bd";
};
```

> **Note:** The `file` path points to where we will create our actual database in the next step.

---

## 3. Creating and Configuring the Zone File

The Zone File is the actual "phonebook" containing the names and IP addresses.

**Step:** Copy the template to your new file location.

Bash

```
sudo cp /etc/bind/db.local /etc/bind/db.zhjlab.bd
sudo nano /etc/bind/db.zhjlab.bd
```

### The Zone File Structure

Plaintext

```
$TTL    604800
@       IN      SOA     ns1.zhjlab.bd. admin.zhjlab.bd. (
                              2026030801 ; Serial (Format: YYYYMMDDNN)
                              604800     ; Refresh
                              86400      ; Retry
                              2419200    ; Expire
                              604800 )   ; Negative Cache TTL
;
; --- Name Servers (NS) ---
@       IN      NS      ns1.zhjlab.bd.

; --- Glue Records & Address Records (A) ---
ns1     IN      A       192.168.1.10      ; The IP of this DNS server
s16     IN      A       192.168.1.16      ; A standard host record

; --- Aliases (CNAME) ---
web     IN      CNAME   s16               ; Points web.zhjlab.bd to s16
```

### Record Definitions & Syntax

- **`@` (Origin):** A shorthand symbol representing the domain name defined in `named.conf.local` (`zhjlab.bd`).
    
- **`SOA` (Start of Authority):** The primary record that specifies the authoritative server and administrative contact.
    
- **`A` (Address):** Maps a hostname to an IPv4 address.
    
- **`CNAME` (Canonical Name):** Maps one name to another name (an alias).
    
- **`NS` (Name Server):** Identifies the servers that are authoritative for this zone.
    
- **The Trailing Dot (`.`):** **Critical.** Every Fully Qualified Domain Name (FQDN) inside the zone file must end with a dot (e.g., `ns1.zhjlab.bd.`). If omitted, BIND will append the origin, resulting in `ns1.zhjlab.bd.zhjlab.bd`.
    

---

## 4. Understanding Glue Records

A **Glue Record** is an A record that provides the IP address of a Name Server.

**The "Catch-22" Problem:** If you tell the internet "Ask `ns1.zhjlab.bd` for information," the internet needs to find the IP of `ns1.zhjlab.bd` first. But to find that IP, it has to ask `ns1.zhjlab.bd`.

**The Solution:**

By including the `ns1 IN A 192.168.1.10` line in the same file as the NS record, you "glue" the IP to the name. The parent server (or the local BIND process) provides both the name and the IP simultaneously, breaking the circular dependency.

---

## 5. Verification and Testing

Never restart BIND without checking your syntax first, as a single missing semicolon can crash the service.

### Syntax Check Commands

1. **Verify Configuration:** `sudo named-checkconf` (Returns nothing if successful).
    
2. **Verify Zone File:** `sudo named-checkzone zhjlab.bd /etc/bind/db.zhjlab.bd`
    

### Applying Changes

Bash

```
sudo systemctl restart bind9
```

### Interrogating the Server with `dig`

Use the `dig` tool to verify your server is providing Authoritative answers.

1. **Test Local Record:** `dig @127.0.0.1 s16.zhjlab.bd`
    
2. **Verify Authority:** In the output header, look for the **`aa`** flag (**Authoritative Answer**). This proves your server is the "Boss" and not just a middleman.
    
3. **Check CNAME:** `dig @127.0.0.1 web.zhjlab.bd` (The answer should show it pointing to `s16`).
    

---

**Would you like me to show you how to set up "Wildcard" records (`*.zhjlab.bd`) so that any sub-domain someone types automatically points to your main web server?**