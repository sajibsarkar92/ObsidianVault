 

It is a command-line tool used to query **DNS (Domain Name System) Name Servers**. It is the industry standard for troubleshooting because it shows you the "raw" data coming back from the servers, unlike simpler tools like `nslookup`.



## What does it actually do?

At Level Zero, `dig` performs four main tasks for you:

### 1. The Translation (Name to IP)

Its primary job is telling you where a domain lives.

- **Input:** `dig yahoo.com`
    
- **Output:** `74.6.231.21` (The IPv4 address).
    

### 2. Identifying "Ownership" (Name servers)

It tells you which specific server is the "boss" of a domain.

- If you want to know who manages the records for your lab, you ask for the **NS (Name Server)** records.
    

### 3. Checking "Freshness" (TTL)

Every piece of DNS info has an expiration date called **TTL (Time to Live)**. `dig` shows you exactly how many seconds are left before a server is required to throw away that info and ask for a fresh copy. This is why when you change an IP, it doesn't happen instantly for everyone.

### 4. Path Tracing

Using the `+trace` flag (which you had in your notes), `dig` acts like a GPS. It starts at the very top of the internet (the **Root Servers**) and follows the path down through the country code (like `.bd`) until it hits the specific server in your lab.

---

![[Pasted image 20260218193621.png]]
Here is a concise, line-by-line breakdown of your `dig` output:

### **The Request Metadata**

- **`DiG 9.18.44`**: The version of the `dig` tool you are using.
- **`global options: +cmd`**: Confirms that the command-line arguments are being processed.
- **`Got answer:`**: Confirms the DNS server responded successfully.

### **The Header Section**

- **`opcode: QUERY`**: You performed a standard "Question" lookup.
- **`status: NOERROR`**: Success; the domain name exists and was found.
- **`id: 40381`**: A random number used to match this specific answer to your request.
- **`flags: qr rd ra`**:
    - **`qr` (Query Response)**: This is a reply from a server.
    - **`rd` (Recursion Desired)**: You asked the server to do the hard work of finding the IP.
    - **`ra` (Recursion Available)**: The server confirmed it is capable of doing that work.
- **`QUERY: 1, ANSWER: 6, ...`**: You asked 1 question and received 6 different IP addresses in response.
    

---

### **The Question Section**

- **`;yahoo.com. IN A`**: This restates your question: "What is the **A** (IPv4) record for **yahoo.com**?"

---

### **The Answer Section**

- **`yahoo.com.`**: The name being looked up.
- **`1363`**: **TTL (Time to Live)**. The number of seconds remaining before this info expires from your local cache.
- **`IN`**: "Internet" (the class of the network).
- **`A`**: The record type (**Address**).
- **`74.6.231.21`**: The physical IPv4 address of the Yahoo server.


---

### **The Footer (Stats)**

- **`Query time: 157 msec`**: How long the round-trip request took.
    
- **`SERVER: 127.0.0.53#53`**: The IP and port of the machine that answered you (your local system resolver).
    
- **`WHEN: Wed Feb 18...`**: The exact timestamp of the query.
    
- **`MSG SIZE rcvd: 134`**: The size of the DNS response packet in bytes.

# Name server Lookup
```
dig yahoo.com NS
```

![[Pasted image 20260218194022.png]]



# Directed Query



![[Pasted image 20260218195006.png]]


By using `@ns1.yahoo.com`, you told `dig` to ignore your local Fedora settings and send the packet directly across the internet to Yahoo's physical hardware. This is why the **SERVER** line at the bottom shows an external IP (`68.180.131.16`) instead of your usual `127.0.0.53`.

---

### 2. The "aa" Flag (Authoritative Answer)

This is the most important part of your screenshot for your lab.

- Look at the line: `;; flags: qr aa rd;`
    
- The presence of **`aa`** means you are talking to the "Source of Truth."
    
- **The technicality:** The server checked its own local configuration file (its **Zone File**) and found the answer there. It didn't have to ask anyone else or look in its history.
    

---

### 3. The Recursion Warning

`WARNING: recursion requested but not available`

- This is a "security" feature. Yahoo's nameservers are **Authoritative**, not **Recursive**.
    
- **Explanation:** They are designed to answer questions _about_ Yahoo. If you tried to ask this specific server for the IP of `google.com`, it would refuse. It's telling you: "I can give you my own records, but don't ask me to go find other people's records for you."
    

---

### 4. The Response Consistency

Notice the **ANSWER SECTION** still shows 6 different IP addresses.

- These are the same IPs you saw in Step 1.
    
- However, because you asked the "Boss," you are now 100% certain these are the correct, official IPs intended by Yahoo's engineers, with no risk of "cache poisoning" or outdated info from a middleman.
    

---

### 5. Why this is the "Secret Sauce" for your Lab

In your notes, you have the command: `dig @127.0.0.1(my loopback) www.yahoo.com`.

- If you run that, you are asking **your own Bind9 server** to go find Yahoo.
    
- In that case, your server acts as a **Recursive Resolver** (the middleman).
    
- But when you eventually run `dig @127.0.0.1 s16.zhjlab.bd`, you are asking your server about **itself**. You will be looking specifically for that **`aa` flag** to prove you configured your `named.conf.local` correctly.


# Dig Trace




![[Pasted image 20260218200629.png]]![[Pasted image 20260218200430.png


```
┌─────────────────────────────────────────────────────────────────────────┐
│  STEP 1: ROOT SERVERS (.)                                               │
│  ───────────────────────                                                │
│  Query: "Who handles .com?"                                             │
│  Answer: "Ask a.gtld-servers.net, b.gtld-servers.net, ..."              │
│  Source: Your resolver → Root servers (a.root-servers.net)              │
│  IPs: 198.41.0.4, 192.228.79.201, etc.                                  │
└─────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│  STEP 2: TLD SERVERS (.com)                                             │
│  ─────────────────────────                                              │
│  Query: "Who handles yahoo.com?"                                        │
│  Answer: "Ask ns1.yahoo.com, ns2.yahoo.com, ..."                        │
│  Source: Root server → TLD servers (a.gtld-servers.net)                 │
│  IPs: 192.5.6.30, 192.33.14.30, etc.                                    │
└─────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│  STEP 3: AUTHORITATIVE SERVERS (yahoo.com)                              │
│  ─────────────────────────────────────────                              │
│  Query: "What's the A record for yahoo.com?"                            │
│  Answer: "98.137.11.163, 74.6.231.20, ..."                              │
│  Source: TLD server → Yahoo's NS (ns1.yahoo.com)                        │
│  IP: 68.180.131.16                                                      │
└─────────────────────────────────────────────────────────────────────────┘
```


## The process 

```
┌─────────────────────────────────────────────────────────────────┐
│  YOUR MACHINE (running dig +trace)                              │
│  ─────────────────────────────────                              │
│  dig performs ITERATIVE queries directly — NOT recursive!       │
│                                                                 │
│  dig → Root → dig → TLD → dig → Authoritative → Answer to you   │
│       ↑_________↑_________↑                                     │
│       These are separate, direct queries from YOUR machine      │
└─────────────────────────────────────────────────────────────────┘
```