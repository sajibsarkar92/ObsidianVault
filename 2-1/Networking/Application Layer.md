# 🌐 Application Layer — Complete Study Notes

> **Source:** Kurose & Ross — Computer Networking: A Top-Down Approach  
> **Tags:** #networking #application-layer #HTTP #DNS #SMTP #FTP #study-notes  
> **Status:** 📚 Complete Reference

---

## 📋 Table of Contents

1. [[#1. Application Architectures]]
2. [[#2. Client-Server Paradigm]]
3. [[#3. Peer-to-Peer (P2P) Architecture]]
4. [[#4. Processes Communicating]]
5. [[#5. Sockets]]
6. [[#6. Addressing Processes]]
7. [[#7. Application-Layer Protocols]]
8. [[#8. Transport Service Requirements]]
9. [[#9. Internet Transport Protocols — TCP vs UDP]]
10. [[#10. Web and HTTP]]
11. [[#11. HTTP Connections — Persistent vs Non-Persistent]]
12. [[#12. HTTP Messages — Request and Response]]
13. [[#13. Cookies]]
14. [[#14. Web Caches (Proxy Servers)]]
15. [[#15. Conditional GET]]
16. [[#16. FTP — File Transfer Protocol]]
17. [[#17. E-mail — SMTP, POP3, IMAP]]
18. [[#18. DNS — Domain Name System]]
19. [[#19. Practice Questions (Kurose & Ross Style)]]

---

## 1. Application Architectures

### What is it?

An **application architecture** defines _how_ an application is structured across the network — specifically, how the different machines and processes involved are organized and who does what.

There are two main architectures:

|Architecture|Description|
|---|---|
|**Client-Server**|One always-on server answers requests from many clients|
|**Peer-to-Peer (P2P)**|Peers communicate directly with each other, no dedicated server|

### 🧠 Simple Analogy

Think of a **restaurant** (Client-Server) vs a **potluck dinner** (P2P):

- In a restaurant, you (client) order from a waiter (server) who brings food from the kitchen.
- At a potluck, everyone brings food _and_ eats food. There's no dedicated chef.

---

## 2. Client-Server Paradigm

### How It Works

**The Server:**

- Is **always on** — it never sleeps. It's perpetually waiting for requests.
- Has a **permanent (fixed) IP address** — clients always know where to find it.
- Is often hosted in **data centers** so it can scale (handle millions of clients).

**The Clients:**

- **Initiate** communication — they reach out first.
- May have **dynamic IP addresses** — their IP can change (like your home internet).
- **Do NOT communicate directly with each other** — all traffic goes through the server.

### Examples of Client-Server Apps

- **HTTP** (web browsing)
- **IMAP** (email retrieval)
- **FTP** (file transfer)

### ⚠️ Common Confusion

> "If the server is always-on, does it mean my laptop server counts?"

No! A "server" in this context refers to a machine specifically designed to _serve_ requests, running 24/7, typically in a data center. Your laptop runs a _client_, not a server (unless you explicitly run server software on it).

---

## 3. Peer-to-Peer (P2P) Architecture

### How It Works

- **No always-on server** — there's no single machine waiting for requests.
- **Arbitrary end systems** (regular computers, phones, etc.) communicate **directly** with each other.
- Each peer **both requests** services from other peers **and provides** services to them.
- This gives P2P **self-scalability** — when a new peer joins, it brings _new service demand_ (it wants files), but also _new service capacity_ (it can share files). The system grows with itself.

### Example

**BitTorrent** — when you download a movie via torrent, you're downloading pieces from many peers simultaneously, and uploading pieces to others at the same time.

### Key Characteristic: Self-Scalability

```
More peers = More demand BUT ALSO More supply
Net effect: System doesn't collapse under load
```

### ⚠️ Common Confusion

> "Is P2P always leaderless?"

Mostly, but not always. Some P2P systems have a _tracker_ (a central server that just keeps track of who has what), while actual data transfer is P2P. True "pure" P2P has no central component at all.

---

## 4. Processes Communicating

### Key Concepts

A **process** is simply a program that is currently running on a host (computer).

**Two scenarios:**

1. **Same host:** Two processes communicate via **inter-process communication (IPC)** — defined by the operating system (pipes, shared memory, etc.).
2. **Different hosts:** Processes communicate by **exchanging messages** across the network.

### Client vs Server Process

Even in P2P apps, we still use this distinction at the _process_ level:

|Process Type|Definition|
|---|---|
|**Client process**|The one that **initiates** communication|
|**Server process**|The one that **waits to be contacted**|

### ⚠️ Common Confusion

> "Does a P2P app have client and server processes?"

Yes! In P2P, each peer runs **both** a client process (when requesting data) and a server process (when providing data). The labels refer to the _role_ in a specific communication, not the machine.

---

## 5. Sockets

### What is a Socket?

A **socket** is the interface between the application layer and the transport layer. Think of it as a **door** or **mailbox** through which your application sends and receives data.

```
Application Layer  ←→  [SOCKET]  ←→  Transport Layer  ←→  Network  ←→  Transport Layer  ←→  [SOCKET]  ←→  Application Layer
   (Your Code)                        (OS Handles This)                   (OS Handles This)                    (Their Code)
```

### The Door Analogy (from the slides)

- Sending process **"shoves a message out the door"** (into the socket)
- It then relies on the transport infrastructure (TCP/UDP) to deliver it to the socket at the other end
- The receiving process picks it up from its own socket

### Who Controls What?

|Component|Controlled By|
|---|---|
|The socket itself|App developer (you choose TCP or UDP, set some options)|
|Everything below the socket|The Operating System|

### Two Sockets Involved

Every communication involves **exactly two sockets** — one on each side (sender and receiver).

### ⚠️ Common Confusion

> "Is a socket the same as a port?"

No! A socket is a _software abstraction_ (an endpoint for communication). A port is a _number_ that identifies which process/application on a host should receive a message. A socket is bound _to_ a port, but they're not the same thing. Think: port = apartment number, socket = the actual door of that apartment.

---

## 6. Addressing Processes

### The Problem

On a network, we need to identify **which process** on **which machine** to send data to.

### Why IP Address Alone Isn't Enough

- An IP address identifies the **host** (the machine).
- But one machine can run **many processes** at the same time (a web server, an email server, an FTP server, etc.).
- So we need a second piece of information: the **port number**.

### The Full Identifier

$$\text{Process Identifier} = \text{IP Address} + \text{Port Number}$$

### Well-Known Port Numbers

|Service|Port Number|
|---|---|
|HTTP (web)|**80**|
|HTTPS (secure web)|**443**|
|SMTP (email sending)|**25**|
|FTP (file transfer)|**21**|
|DNS|**53**|

### Example

To send an HTTP request to `gaia.cs.umass.edu`:

- IP Address: `128.119.245.12`
- Port Number: `80`

### ⚠️ Common Confusion

> "Are port numbers fixed?"

Only **well-known** ports (0–1023) are standardized for common services. Ports 1024–65535 can be used by applications dynamically. When you run a custom app, it typically uses a random high port.

---

## 7. Application-Layer Protocols

### What Does an Application-Layer Protocol Define?

An application-layer protocol is like a **rulebook** for how two processes talk to each other. It specifies:

|Element|What It Means|
|---|---|
|**Types of messages**|e.g., request messages, response messages|
|**Message syntax**|What fields exist in the message and how they're formatted|
|**Message semantics**|What the information in each field _means_|
|**Rules**|When and how to send/respond to messages|

### Example

HTTP defines:

- **Message types:** GET request, POST request, HTTP response
- **Syntax:** Request line + headers + body
- **Semantics:** Status code 200 means OK, 404 means Not Found
- **Rules:** Client always initiates; server waits for requests

---

## 8. Transport Service Requirements

### What Do Apps Need From the Transport Layer?

Different apps have wildly different needs. The four key dimensions are:

### 1. Data Integrity (Reliability)

- **Some apps MUST have 100% reliable delivery:** File transfer, email, web documents — you can't have half a file or a corrupted document.
- **Some apps can tolerate loss:** Audio/video streaming — a dropped packet causes a brief glitch, which is acceptable.

### 2. Timing (Latency)

- **Time-sensitive apps:** Internet telephony, interactive games — need low delay (milliseconds). A 500ms delay in a phone call makes it feel like a satellite call.
- **Non-time-sensitive apps:** Email, file download — a few extra seconds don't matter.

### 3. Throughput (Bandwidth)

- **Bandwidth-hungry apps:** Video streaming needs a minimum data rate (e.g., 5 Mbps for HD video). Below that, the video buffers.
- **Elastic apps:** Email, file transfer — they use whatever bandwidth is available and don't have a minimum requirement.

### 4. Security

- Encryption, data integrity verification, end-point authentication.

### Summary Table

|Application|Data Loss|Throughput|Time Sensitive?|
|---|---|---|---|
|File transfer/download|No loss|Elastic|No|
|E-mail|No loss|Elastic|No|
|Web documents|No loss|Elastic|No|
|Real-time audio/video|Loss-tolerant|Audio: 5Kbps–1Mbps; Video: 10Kbps–5Mbps|Yes, tens of ms|
|Streaming audio/video|Loss-tolerant|Same as above|Yes, few secs|
|Interactive games|Loss-tolerant|Kbps+|Yes, tens of ms|
|Text messaging|No loss|Elastic|Yes and no|

---

## 9. Internet Transport Protocols — TCP vs UDP

### TCP (Transmission Control Protocol)

TCP is the **"reliable workhorse"** of the internet. It provides:

|Feature|What It Means|
|---|---|
|**Reliable transport**|Every byte sent is guaranteed to arrive in order, with no corruption|
|**Flow control**|Sender won't overwhelm the receiver (adjusts speed based on receiver's capacity)|
|**Congestion control**|Slows down when the _network_ is overloaded (not just receiver)|
|**Connection-oriented**|A handshake must happen before any data flows (setup phase)|

**TCP does NOT provide:** timing guarantees, minimum throughput guarantees, or security (those are add-ons like TLS).

### UDP (User Datagram Protocol)

UDP is the **"bare-bones, fire-and-forget"** protocol. It provides:

- Just sends packets — **no guarantees** of delivery, order, or integrity.
- No connection setup overhead.

### Why Does UDP Exist? (The Classic Question)

UDP is useful when:

1. **Speed matters more than reliability** — Live video streaming. A dropped frame is better than pausing to wait for a retransmission.
2. **App handles reliability itself** — DNS uses UDP for quick single-request/response lookups.
3. **Overhead of TCP is too high** — Real-time games need low latency; TCP's congestion control adds unpredictable delays.
4. **Multicast/Broadcast** — TCP is point-to-point; UDP can send to many at once.

### Which Protocol Do Common Apps Use?

|Application|Protocol|
|---|---|
|File transfer/download (FTP)|TCP|
|E-mail (SMTP)|TCP|
|Web documents (HTTP)|TCP|
|Internet telephony|TCP or UDP|
|Streaming audio/video|TCP|
|Interactive games|UDP or TCP|

### ⚠️ Common Confusion

> "If UDP is unreliable, why not always use TCP?"

Because reliability comes with a **cost**:

- TCP has **connection setup overhead** (3-way handshake takes time).
- TCP has **retransmission delays** — if a packet is lost, TCP waits and resends, which can cause jitter in real-time apps.
- TCP's **congestion control** can throttle your app unpredictably. For real-time apps where a slightly old/lost packet is useless anyway, UDP's "best effort" is actually better.

---

## 10. Web and HTTP

### Web Basics

A **web page** consists of **objects** — these can be HTML files, JPEG images, video clips, audio files, JavaScript files, etc.

Every web page has:

- A **base HTML file** — the main document with the page structure
- **Referenced objects** — images, scripts, stylesheets, etc., each stored at a specific URL

A **URL (Uniform Resource Locator)** has two parts:

```
www.someschool.edu / someDept/pic.gif
      ↑                    ↑
  host name            path name
```

### HTTP (HyperText Transfer Protocol)

**HTTP** is the application-layer protocol that the web runs on. It defines how web clients and servers communicate.

**HTTP is a client/server protocol:**

- **Client:** Your browser (Chrome, Firefox, Safari) — it requests web objects and displays them.
- **Server:** A web server (Apache, Nginx) — it listens for requests and sends back the objects.

### HTTP Uses TCP

The process:

1. Client initiates a **TCP connection** to the server on **port 80**.
2. Server **accepts** the TCP connection.
3. HTTP messages are exchanged between browser and web server.
4. TCP connection is **closed**.

### HTTP is "Stateless"

This is crucial: **HTTP servers maintain NO information about past client requests.**

Each request is treated as completely independent. The server has no memory of you.

### Why Stateless? Why Not Stateful?

**Stateless is simpler:**

- No need to keep track of millions of clients' states.
- If the server crashes, no state is lost from the server's perspective.

**But stateful is complex:**

- Server must remember past history.
- If either side crashes, their views of "state" become inconsistent → hard to reconcile.
- This is why cookies were invented — to simulate state on top of a stateless protocol.

### ⚠️ Common Confusion

> "If HTTP is stateless, how does Amazon remember I'm logged in?"

Through **cookies** and **session tokens** — these are stored client-side and sent with each request, allowing the server to _look up_ your state in a database. The protocol itself is stateless, but the _application_ can maintain state using these mechanisms.

---

## 11. HTTP Connections — Persistent vs Non-Persistent

### Non-Persistent HTTP (HTTP/1.0)

**Rule:** At most **one object** is sent per TCP connection. After the object is sent, the connection is **closed**.

**Steps to fetch a web page with 10 images (11 objects total):**

1. Open TCP connection → Get base HTML file → Close TCP connection
2. Open TCP connection → Get image 1 → Close TCP connection
3. Open TCP connection → Get image 2 → Close TCP connection
4. ... (repeat 10 more times)

**Total TCP connections needed:** 11 (one per object)

### Response Time: RTT

**RTT (Round Trip Time)** = time for a small packet to travel from client to server and back.

**Non-Persistent HTTP response time per object:**

```
= 1 RTT (to initiate TCP connection)
+ 1 RTT (for HTTP request + first bytes of response)
+ File transmission time
= 2 RTT + file transmission time
```

**For a page with 10 images (11 objects):**

```
Total = 11 × (2 RTT + file transmission time)
```

This is slow and wasteful!

### Persistent HTTP (HTTP/1.1)

**Rule:** The TCP connection remains **open** after sending a response. Multiple objects can be sent over the same connection.

**With pipelining (HTTP/1.1):**

- Client sends requests back-to-back without waiting for responses.
- Server responds in order.

**Non-Persistent Issues Solved:**

|Problem|Solution|
|---|---|
|2 RTTs per object|Only 1 RTT per object after first|
|OS overhead for each TCP connection|Single connection = single OS overhead|
|Sequential downloads|Can pipeline requests|

### Visual Timeline Comparison

```
Non-Persistent:            Persistent:
                           
[TCP Setup RTT]            [TCP Setup RTT]
[Request RTT + file]       [Request1 RTT + file1]
[TCP Setup RTT]            [Request2 RTT + file2]
[Request RTT + file]       [Request3 RTT + file3]
[TCP Setup RTT]            ...all on same connection
[Request RTT + file]
...
```

### Pipelining in Persistent HTTP — Deep Dive

#### How Pipelining Works

Without pipelining (persistent but sequential), the client must wait for each response before sending the next request:

```
Client          Server
  |──GET obj1──→|
  |←──obj1 ─────|   (wait for response)
  |──GET obj2──→|
  |←──obj2 ─────|   (wait for response)
  |──GET obj3──→|
  |←──obj3 ─────|
```

With pipelining, the client fires all requests back-to-back without waiting:

```
Client          Server
  |──GET obj1──→|
  |──GET obj2──→|
  |──GET obj3──→|
  |←──obj1 ─────|
  |←──obj2 ─────|
  |←──obj3 ─────|
```

#### RTT Calculations

**Non-Persistent HTTP** (per object):

- 1 RTT — TCP handshake (SYN + SYN-ACK)
- 1 RTT — HTTP request + first bytes of response arrive
- + file transmission time

> **Cost per object = 2 RTT + transmission time** **Total for N objects = N × (2 RTT + transmission time)**

**Persistent HTTP without pipelining** (sequential):

- 1 RTT — TCP handshake (once, for the whole session)
- 1 RTT per object — request + response

> **Total = 1 RTT (setup) + N × (1 RTT + transmission time)**

**Persistent HTTP with pipelining:**

- 1 RTT — TCP handshake
- 1 RTT — to request and receive the first object
- All remaining objects: pipeline fills the connection; ideally ~1 RTT for all remaining objects combined (assuming small transmission times relative to RTT)

> **Total ≈ 1 RTT (setup) + 1 RTT (first object) + remaining transmission times** In the ideal case with N objects: approximately **2 RTT + all transmission times**

#### Comparison Table

|Feature|Non-Persistent|Persistent (No Pipeline)|Persistent + Pipelining|
|---|---|---|---|
|TCP connections needed|1 per object|1 for entire session|1 for entire session|
|RTT cost per object|2 RTT|1 RTT (after setup)|~0 RTT (after first)|
|Total RTT for N objects|N × 2 RTT|1 + N RTT|~2 RTT (all objects)|
|OS overhead|High (many connections)|Low|Low|
|Server must respond before next request|N/A|Yes|No|
|HTTP version|HTTP/1.0|HTTP/1.1|HTTP/1.1|
|Efficiency|❌ Worst|✅ Better|✅✅ Best|

> ⚠️ **Common Confusion** "Does pipelining mean responses can arrive out of order?" No — HTTP/1.1 pipelining still requires the server to respond **in order** (head-of-line blocking). If object 1 is slow to generate, objects 2 and 3 are delayed even if they're ready. This limitation was a major motivation for **HTTP/2**, which introduced true multiplexing — responses can arrive in any order using stream IDs.

---

## 12. HTTP Messages — Request and Response

### HTTP Request Message

**Format:** ASCII text (human-readable)

```
GET /index.html HTTP/1.1\r\n          ← Request Line
Host: www-net.cs.umass.edu\r\n        ←
User-Agent: Firefox/3.6.10\r\n        ←
Accept: text/html\r\n                 ← Header Lines
Accept-Language: en-us\r\n            ←
Connection: keep-alive\r\n            ←
\r\n                                  ← Blank line = end of headers
                                      ← Body (empty for GET)
```

**Parts of the Request Line:**

```
METHOD   SP   URL   SP   HTTP-VERSION   CR LF
  GET         /page.html   HTTP/1.1     \r\n
```

### HTTP Methods

**HTTP/1.0 Methods:**

|Method|What It Does|
|---|---|
|**GET**|Retrieve a resource|
|**POST**|Submit data to the server (data goes in the body)|
|**HEAD**|Like GET, but server only returns headers, not the object itself (useful for checking if a file changed)|

**HTTP/1.1 Adds:**

|Method|What It Does|
|---|---|
|**PUT**|Upload a file to a specific path on the server|
|**DELETE**|Delete a file at the specified URL|

### Uploading Form Data: POST vs GET

**POST Method:**

- Form data is sent in the **entity body** of the request
- Not visible in the URL
- Used for sensitive data, large data, file uploads

**GET Method (URL method):**

- Data is appended to the URL: `www.somesite.com/search?q=monkeys&type=banana`
- Visible in the URL bar — bookmarkable, shareable
- Not suitable for passwords or large amounts of data

### ⚠️ Common Confusion

> "Is GET always 'safe' and POST always 'dangerous'?"

In the HTTP specification, GET is meant to be **idempotent** (calling it multiple times has the same effect as calling it once) and **safe** (doesn't modify server state). POST is meant to _create or modify_ resources. But in practice, many developers misuse these (GET requests that delete things, POST that just queries). The security difference people think of (POST hides data, GET exposes it) is a UI concern — both are visible in network traffic without HTTPS.

---

### HTTP Response Message

```
HTTP/1.1 200 OK\r\n                     ← Status Line
Date: Sun, 26 Sep 2010 20:09:20 GMT\r\n ←
Server: Apache/2.0.52 (CentOS)\r\n      ←
Last-Modified: Tue, 30 Oct 2007...\r\n  ← Header Lines
Content-Length: 2652\r\n                ←
Content-Type: text/html\r\n             ←
\r\n                                    ← Blank line
<html>...</html>                        ← Data (response body)
```

### HTTP Status Codes

These appear in the first line of the server-to-client response:

|Code|Meaning|When Used|
|---|---|---|
|**200 OK**|Request succeeded|Everything worked|
|**301 Moved Permanently**|Object moved to new URL (in Location: header)|URL has changed permanently|
|**400 Bad Request**|Server couldn't understand the request|Malformed request|
|**404 Not Found**|Object not found on this server|Wrong URL|
|**505 HTTP Version Not Supported**|Server doesn't support the HTTP version requested|Client uses too new/old HTTP|

### ⚠️ Common Confusion

> "What's the difference between 301 and 302?"

- **301 Moved Permanently:** The resource has a _new permanent URL_. Browsers and search engines should update their links/indices.
- **302 Found (Temporary Redirect):** The resource is _temporarily_ at a different URL. Keep using the original URL for future requests.

---

## 13. Cookies

### The Problem Cookies Solve

HTTP is stateless — the server doesn't remember you. But websites need to:

- Keep you logged in
- Remember your shopping cart
- Personalize your experience

### How Cookies Work (4 Components)

1. **Cookie header in HTTP _response_** — server sets a cookie when you first visit
2. **Cookie header in HTTP _request_** — your browser sends the cookie back on every subsequent request
3. **Cookie file on your computer** — browser stores cookies locally
4. **Backend database at the website** — server stores your data keyed by cookie ID

### Step-by-Step Example

```
FIRST VISIT:
Client                              Amazon Server
  |--- HTTP Request (no cookie) ------→|
  |                                    | Creates user ID: 1678
  |                                    | Stores 1678 in database
  |←-- HTTP Response: set-cookie:1678--|
  | (browser saves cookie: amazon=1678)|

SECOND VISIT (same session):
  |--- HTTP Request: cookie: 1678 ----→|
  |                                    | Looks up 1678 in database
  |                                    | Knows it's YOU
  |←-- Personalized HTTP Response -----|

ONE WEEK LATER:
  |--- HTTP Request: cookie: 1678 ----→|
  |                                    | Still knows it's you!
  |←-- Personalized HTTP Response -----|
```

### What Cookies Are Used For

- **Authorization** — staying logged in
- **Shopping carts** — remembering what you added
- **Recommendations** — "based on your history"
- **User session state** — webmail, preferences

### ⚠️ Common Confusion

> "Are cookies the same as cache?"

No! They're completely different:

- **Cookies** = Small pieces of data _identifying you_ or storing preferences, sent between client and server in HTTP headers.
- **Cache** = A local copy of web objects (HTML, images) stored to avoid re-downloading them.

> "Are cookies a security risk?"

They can be. **Session hijacking** occurs when someone steals your cookie and impersonates you. This is why:

- Cookies should be sent only over HTTPS
- Sensitive cookies should have the `HttpOnly` flag (can't be read by JavaScript)
- Session cookies should expire after logout

---

## 14. Web Caches (Proxy Servers)

### What is a Web Cache?

A **web cache** (also called a **proxy server**) is an intermediary server that stores copies of recently requested web objects. When a client requests an object, the cache checks if it has a recent copy — if yes, it returns it immediately without contacting the origin server.

**Goal:** Satisfy client requests without involving the origin server.

### How It Works

```
Client → [Web Cache] → Origin Server
              ↑
         Stores copies
         of responses
```

1. User configures browser to point to a local web cache.
2. Browser sends _all_ HTTP requests to the cache.
3. **Cache HIT:** Object is in cache → returned immediately to client.
4. **Cache MISS:** Object not in cache → cache fetches from origin server, stores a copy, returns to client.

### The Cache Acts as Both Client and Server

- It's a **server** to the requesting client (responds with the object)
- It's a **client** to the origin server (makes requests to get objects)

### Why Use Web Caching?

1. **Reduce response time** — cache is physically closer to the client (same campus/ISP), so lower RTT.
2. **Reduce traffic on access links** — less data needs to travel over expensive wide-area links.
3. **Reduces load on origin servers** — popular sites don't have to serve every request themselves.

### Caching Example: The Numbers

**Scenario:**

- Access link rate: **1.54 Mbps**
- RTT from institutional router to server: **2 seconds**
- Web object size: **100K bits**
- Average request rate: **15 requests/second**
- Average data rate to browsers: **1.50 Mbps**

**Without cache:**

```
Access link utilization = 1.50 / 1.54 = 0.97 (97%!)
→ Massive queueing delays at near-saturation
End-end delay = 2 sec + MINUTES (queuing) + microseconds
```

**Option 1: Upgrade the access link to 154 Mbps:**

```
Cost: EXPENSIVE
Access link utilization = 1.50 / 154 = 0.0097 (< 1%)
End-end delay = 2 sec + milliseconds
```

**Option 2: Install a local web cache (assume 40% hit rate):**

```
60% of requests go to origin server
Data rate over access link = 0.6 × 1.50 = 0.9 Mbps
Access link utilization = 0.9 / 1.54 = 0.58 (58%) → low queueing

Average end-end delay:
= 0.6 × (delay from origin) + 0.4 × (delay from cache)
= 0.6 × 2.01 sec      + 0.4 × ~msecs
= ~1.2 seconds

Cost: CHEAP (just a local cache server)
Result: BETTER than the 154 Mbps upgrade!
```

**Cache-Control Headers (Server → Cache):**

```http
Cache-Control: max-age=3600       # Cache this for 3600 seconds
Cache-Control: no-cache           # Don't cache this object
```

### ⚠️ Common Confusion

> "Isn't the browser already a cache?"

Yes! Your **browser** has its own local cache. Web cache/proxy servers are different — they're **shared** caches serving many users (e.g., all users at a university or ISP). Browser cache is private to you; proxy cache is shared.

---

## 15. Conditional GET

### The Problem

Even if an object is in the cache, it might be **outdated** (the origin server has a newer version). Re-downloading unnecessarily wastes bandwidth. But we also don't want to serve stale content.

### The Solution: Conditional GET

The client (or cache) sends an HTTP request with a special header:

```http
GET /page.html HTTP/1.1
Host: www.example.com
If-Modified-Since: Tue, 30 Oct 2007 17:00:02 GMT
```

This says: **"Give me this object ONLY if it has changed since this date."**

### Server Responses

**If NOT modified (cache is still fresh):**

```http
HTTP/1.0 304 Not Modified
(No body — just headers)
```

→ Cache serves its stored copy. No data transferred!

**If modified (cache is stale):**

```http
HTTP/1.0 200 OK
[full object in body]
```

→ Cache downloads new version, stores it, serves to client.

### ⚠️ Common Confusion

> "Who sends the If-Modified-Since header — the browser or the cache?"

Both can! When your **browser** checks its local cache, it may send a conditional GET to the web server/proxy. When a **proxy cache** checks with the origin server, it also sends a conditional GET. It's a mechanism that works at multiple levels.

---

## 16. FTP — File Transfer Protocol

### What is FTP?

**FTP (File Transfer Protocol)** is used to transfer files between a client and a remote host. Defined in **RFC 959**. Uses **port 21** for control.

### Architecture

```
User ↔ FTP User Interface ↔ FTP Client ←→ FTP Server ↔ Remote File System
                              (local file system)
```

**Client:** The side that _initiates_ the transfer (either uploading or downloading). **Server:** The remote host where files are stored.

### FTP's Unique Feature: Two Separate Connections

This is what makes FTP distinctive — it uses **two separate TCP connections**:

|Connection|Port|Purpose|
|---|---|---|
|**Control Connection**|Port 21|Commands (user login, directory browsing, file transfer commands)|
|**Data Connection**|Port 20|Actual file data transfer|

### How It Works

1. FTP client connects to FTP server on **port 21** (control channel — persistent throughout session).
2. Client authenticates over the control connection.
3. Client browses remote directory, issues commands over control connection.
4. When a file transfer is requested, server opens a **second TCP connection on port 20** (data channel).
5. After one file is transferred, the **data connection is closed**.
6. For the next file, a new data connection is opened.

### Key FTP Concepts

**Control connection is "out of band":** The term "out of band" means the control information (commands) travels on a _separate_ channel from the data. This contrasts with HTTP, where commands and data travel on the same connection ("in band").

**FTP is stateful:** Unlike HTTP, FTP _does_ maintain state:

- Current directory (where you are on the remote file system)
- Authentication status (are you logged in?)

### ⚠️ Common Confusion

> "Why does FTP use two connections instead of one?"

It's a design choice from the early internet. Having separate control and data channels allows:

- Commands to be sent while a transfer is in progress
- The server to initiate the data connection (the server _calls back_ to the client's data port)

This "active mode" (server initiates data connection) causes problems with firewalls (the client's firewall blocks the incoming connection). That's why "passive mode" FTP was invented — the client initiates both connections.

> "Is FTP secure?"

No! FTP sends credentials (username/password) and data in **plaintext** — anyone sniffing the network can read them. Use **SFTP** (SSH File Transfer Protocol) or **FTPS** (FTP over SSL/TLS) for security.

---

## 17. E-mail — SMTP, POP3, IMAP

### E-mail Architecture: Three Components

1. **User Agents (UA)** — "mail readers" (Outlook, Gmail app, iPhone Mail)
    
    - Used to compose, edit, read email
    - Outgoing and incoming messages stored on the mail server
2. **Mail Servers** — the backbone of email
    
    - **Mailbox:** Stores incoming messages for each user
    - **Message queue:** Holds outgoing messages waiting to be sent
3. **SMTP** — the protocol used between mail servers
    

### How Email Travels (Alice sends to Bob)

```
Alice's UA → [SMTP] → Alice's Mail Server → [SMTP] → Bob's Mail Server → [IMAP/HTTP] → Bob's UA
    1              2           3                   4             5                  6
```

**Step by step:**

1. Alice uses her UA (e.g., Outlook) to compose email to `bob@someschool.edu`
2. Alice's UA sends the message to Alice's mail server via SMTP; placed in outgoing queue
3. Client side of SMTP at Alice's mail server opens TCP connection to Bob's mail server
4. SMTP client sends Alice's message over the TCP connection
5. Bob's mail server places the message in Bob's mailbox
6. Bob uses his UA (e.g., iPhone Mail) to read the message via IMAP

---

### SMTP (Simple Mail Transfer Protocol)

**RFC 5321** — the protocol for mail _transfer_ between mail servers.

**Key Characteristics:**

- Uses **TCP** for reliable delivery on **port 25**
- **Direct transfer:** Sending server connects directly to receiving server
- **Push protocol:** The _sender_ pushes mail to the receiver (vs HTTP, which is pull)

**Three Phases of SMTP Transfer:**

1. **Handshaking (Greeting)** — client and server introduce themselves
2. **Message Transfer** — actual email content is sent
3. **Closure** — connection is politely terminated

**SMTP Uses 7-bit ASCII:** Email messages (both headers and body) must be in 7-bit ASCII. This is why attachments (binary files) must be encoded (using Base64) before sending — they need to be converted to ASCII.

**End of Message Marker:** SMTP uses `CRLF.CRLF` (a line with just a period) to signal the end of a message.

### Sample SMTP Conversation

```
S: 220 hamburger.edu                          ← Server greeting
C: HELO crepes.fr                             ← Client introduces itself
S: 250 Hello crepes.fr, pleased to meet you  ← Server acknowledges
C: MAIL FROM: <alice@crepes.fr>               ← Sender
S: 250 alice@crepes.fr... Sender ok          ← OK
C: RCPT TO: <bob@hamburger.edu>               ← Recipient
S: 250 bob@hamburger.edu ... Recipient ok    ← OK
C: DATA                                       ← Ready to send body
S: 354 Enter mail, end with "." on a line    ← Go ahead
C: Do you like ketchup?                       ← Email body
C: How about pickles?
C: .                                          ← End of message
S: 250 Message accepted for delivery
C: QUIT
S: 221 hamburger.edu closing connection
```

---

### Mail Access Protocols: POP3 vs IMAP

After SMTP delivers mail to the recipient's mail server, the _user_ needs to retrieve it. This is where **POP3** and **IMAP** come in.

```
Sender UA → [SMTP] → Sender's Server → [SMTP] → Receiver's Server → [POP3 or IMAP] → Receiver UA
```

### POP3 (Post Office Protocol version 3)

**Port:** 110

**Two Phases:**

**Phase 1: Authorization**

```
S: +OK POP3 server ready
C: user bob
S: +OK
C: pass hungry
S: +OK user successfully logged on
```

**Phase 2: Transaction**

```
C: list          → S: 1 498 (message 1 is 498 bytes)
                 → S: 2 912
C: retr 1        → S: [message 1 contents]
C: dele 1        → marks message 1 for deletion
C: retr 2        → S: [message 2 contents]
C: dele 2
C: quit          → S: +OK POP3 server signing off (deletes marked messages)
```

**POP3 Modes:**

- **Download-and-delete:** Messages deleted from server after download. If you switch devices, messages are gone from server.
- **Download-and-keep:** Messages stay on server. Downloaded to every client (your laptop AND phone get copies).

**POP3 Limitations:**

- Stateless across sessions — once you end the session, the server forgets everything.
- No folder organization on the server.
- Downloading the same messages to multiple devices leads to inconsistency.

---

### IMAP (Internet Mail Access Protocol)

**RFC 3501** — **Port:** 143

**Key Advantages over POP3:**

|Feature|POP3|IMAP|
|---|---|---|
|Message storage|Client (downloaded)|Server|
|Folders|Local only|On server, synced everywhere|
|State across sessions|Stateless|Stateful|
|Multi-device|Messy|Clean (same view everywhere)|
|Partial download|No|Yes (download headers first)|

**IMAP keeps all messages on the server** — when you read, organize, or delete on your phone, those changes are reflected on your laptop too.

### HTTP for Email (Webmail)

Gmail, Outlook.com, Yahoo! Mail all use:

- **HTTP** between browser and mail server (the web interface)
- Under the hood, they still use **SMTP** (to send) and **IMAP** (to retrieve internally)

### ⚠️ Common Confusion

> "What's the difference between SMTP and IMAP?"

||SMTP|IMAP|
|---|---|---|
|**Direction**|Sending/pushing mail|Receiving/accessing mail|
|**Between**|Mail servers (and UA → server)|Mail server ↔ User Agent|
|**Analogy**|Post office delivering your letter|You checking your mailbox|

> "Why can't I use SMTP to receive mail?"

SMTP is a **push** protocol — it pushes mail _to_ servers. But to _retrieve_ (pull) mail from a server, you need a pull protocol (POP3 or IMAP). It's like a postal worker can deliver to your mailbox, but you need to open the mailbox yourself (POP3/IMAP) to get the mail out.

---

## 18. DNS — Domain Name System

### The Problem

Humans use names like `www.google.com`. Routers use IP addresses like `142.250.80.46`. How do we convert between them?

### DNS: The Solution

**DNS (Domain Name System)** is:

- A **distributed database** implemented in a hierarchy of name servers
- An **application-layer protocol** that hosts and DNS servers use to communicate
- The internet's "phone book" — translates names to IP addresses

### DNS Services

|Service|Description|
|---|---|
|**Hostname-to-IP translation**|`www.amazon.com` → `205.251.242.103`|
|**Host aliasing**|A complex canonical name can have simpler aliases (`www.ibm.com` → `servereast.backup2.ibm.com`)|
|**Mail server aliasing**|MX records let mail servers have aliases|
|**Load distribution**|One hostname → multiple IPs (traffic distributed across server farm)|

### Why Not Centralize DNS?

A single DNS server for the entire internet would be catastrophic:

- **Single point of failure** — one crash = entire internet breaks
- **Traffic volume** — billions of queries per day (Comcast alone: 600 billion/day!)
- **Distant centralized database** — high latency for users far away
- **Maintenance** — impossible to update a single server for the entire internet in real-time

**Conclusion:** DNS is distributed. It doesn't scale centrally.

---

### DNS Hierarchy: Three Levels

```
                    [Root DNS Servers]
                   /        |         \
          .com servers   .org servers   .edu servers
          /      \           |           /        \
   yahoo.com  amazon.com  pbs.org   nyu.edu  umass.edu
   DNS server  DNS server  DNS server  DNS server  DNS server
```

**Level 1: Root DNS Servers**

- There are **13 logical root name servers** (labeled A–M) worldwide.
- Each "logical" server is actually replicated many times (~200+ physical servers in the US alone for redundancy).
- Root servers know where to find TLD servers.
- Managed by **ICANN** (Internet Corporation for Assigned Names and Numbers).
- Supports **DNSSEC** for security (authentication and message integrity).

**Level 2: Top-Level Domain (TLD) Servers**

- Responsible for top-level domains: `.com`, `.org`, `.net`, `.edu`, `.gov`, `.uk`, `.cn`, etc.
- **Network Solutions** manages `.com` and `.net` TLDs.
- **Educause** manages `.edu` TLD.
- TLD servers know where to find authoritative servers for each domain.

**Level 3: Authoritative DNS Servers**

- Each organization runs its own authoritative DNS server.
- Provides the definitive hostname-to-IP mappings for that organization.
- e.g., `dns.cs.umass.edu` is the authoritative server for `cs.umass.edu` — it knows the IP of `gaia.cs.umass.edu`.

**Local DNS Servers (not in the hierarchy):**

- Each ISP has a local DNS server.
- When your computer makes a DNS query, it goes to the local DNS server first.
- Local DNS acts as a proxy — it caches results and forwards queries up the hierarchy.
- To find yours: Mac: `scutil --dns`; Windows: `ipconfig /all`

---

### DNS Resolution: Two Methods

**Example:** Host at `engineering.nyu.edu` wants IP for `gaia.cs.umass.edu`.

#### Method 1: Iterated Query

The local DNS server does all the work, asking each server in turn:

```
engineering.nyu.edu                  local DNS (dns.nyu.edu)
      1 ──────────────────────────────→|
      |   (Who is gaia.cs.umass.edu?)  |
      |                                |──2──→ Root DNS Server
      |                                |←3──── "Ask .edu TLD server"
      |                                |──4──→ TLD DNS Server (.edu)
      |                                |←5──── "Ask umass.edu DNS"
      |                                |──6──→ Authoritative DNS (umass.edu)
      |                                |←7──── "IP is 128.119.245.12"
      8 ←──────────────────────────────|
      (Here's the IP: 128.119.245.12)
```

Each contacted server replies with _"I don't know, but ask this server"_. The local DNS server iterates through the hierarchy.

#### Method 2: Recursive Query

Each server passes the query up to the next level itself:

```
engineering.nyu.edu → local DNS → Root DNS → TLD DNS → Authoritative DNS
                                                                  ↓
engineering.nyu.edu ← local DNS ← Root DNS ← TLD DNS ← Answer
```

**Recursive puts burden on higher-level servers** → can cause heavy load. **Iterated is more common** for this reason.

---

### DNS Caching

**Once any name server learns a mapping, it caches it.**

- Cached entries are returned immediately, without querying further.
- Entries **expire** after a TTL (Time To Live) period set by the authoritative server.
- TLD server addresses are typically cached in local name servers (so root servers are rarely queried).
- **Stale cache entries** — if a server changes its IP, cached mappings may be wrong until TTL expires. Best-effort!

---

### DNS Records (Resource Records — RR)

DNS stores information as **Resource Records (RR)** in the format:

```
(name, value, type, ttl)
```

|Type|Name|Value|Use Case|
|---|---|---|---|
|**A**|Hostname|IP Address|`gaia.cs.umass.edu` → `128.119.245.12`|
|**NS**|Domain|Hostname of authoritative name server|`umass.edu` → `dns.cs.umass.edu`|
|**CNAME**|Alias hostname|Canonical (real) hostname|`www.ibm.com` → `servereast.backup2.ibm.com`|
|**MX**|Domain|Mail server hostname|`foo.com` → `mail.foo.com`|

### Getting Into the DNS

**Registering a new domain (e.g., networkutopia.com):**

1. Go to a **DNS registrar** (e.g., GoDaddy, Network Solutions)
2. Provide names and IP addresses of your authoritative name servers
3. Registrar inserts these records into the `.com` TLD server:
    
    ```
    (networkutopia.com, dns1.networkutopia.com, NS)(dns1.networkutopia.com, 212.212.212.1, A)
    ```
    
4. On your authoritative server, add:
    - Type A record: `www.networkutopia.com → 212.212.212.1`
    - Type MX record: `networkutopia.com → mail.networkutopia.com`

### ⚠️ Common Confusion

> "What's the difference between CNAME and A records?"

- **A record:** Directly maps a hostname to an IP address. Use for actual servers.
- **CNAME record:** Maps one hostname to another hostname (an alias). Useful when you want multiple names to point to the same server without duplicating IP addresses everywhere.

Example: `www.example.com` CNAME → `loadbalancer.example.com` A → `1.2.3.4`

> "If DNS is cached, can attackers poison the cache?"

Yes! **DNS cache poisoning** is a real attack where an attacker injects fake DNS records into a cache, redirecting users to malicious servers. **DNSSEC** (DNS Security Extensions) was designed to prevent this by cryptographically signing DNS records.

---

## 19. Practice Questions (Kurose & Ross Style)

---

### Section A: Application Architectures

**Q1.** What are the two main application architectures discussed in the chapter? Describe the key difference between them.

> **Answer:** The two architectures are **client-server** and **peer-to-peer (P2P)**.
> 
> In client-server, there is always an always-on server with a permanent IP address that responds to requests from many clients. Clients do not communicate directly with each other.
> 
> In P2P, there is no always-on server. Any end system can communicate directly with any other. Each peer both requests and provides services. P2P exhibits self-scalability: new peers bring new service demand _and_ new service capacity.

**Q2.** What is meant by "self-scalability" in P2P architectures?

> **Answer:** Self-scalability means that as new peers join the system, they simultaneously bring new demand (they want services) _and_ new capacity (they can provide services). So the system's ability to handle load grows naturally as the user base grows, without the operator needing to add central infrastructure.

---

### Section B: Processes and Sockets

**Q3.** Why is an IP address alone insufficient to identify a process to which data should be delivered?

> **Answer:** Because a single host (identified by one IP address) can run many processes simultaneously (e.g., a web server, an email server, an FTP server). To identify the _specific_ process, we also need a **port number**. The complete identifier is: IP address + port number.

**Q4.** What is a socket? Why is it analogous to a "door"?

> **Answer:** A socket is the interface (software endpoint) between the application layer and the transport layer. It is analogous to a door because:
> 
> - A _sending_ process "shoves" a message out through its socket (door)
> - The transport infrastructure delivers it to the socket (door) at the receiving end
> - The receiving process "picks up" the message from its socket (door) The application developer controls what goes in/out of the socket; the OS controls everything below it.

---

### Section C: HTTP

**Q5.** What does it mean for HTTP to be "stateless"? What is the advantage of this design?

> **Answer:** HTTP is stateless because the server maintains no information about past client requests. Each request is processed independently.
> 
> **Advantages:** Simpler server design (no need to track millions of client states), easier crash recovery (no inconsistent state to reconcile), easier to scale.
> 
> **Disadvantage:** Applications that need state (like shopping carts or login sessions) must implement state tracking on top of HTTP using mechanisms like cookies.

**Q6.** Suppose a web page consists of a base HTML file and 5 JPEG images. How many RTTs are required to fully load this page using non-persistent HTTP? Persistent HTTP with pipelining?

> **Answer:**
> 
> **Non-persistent HTTP:**
> 
> - 1 RTT to initiate TCP connection (base HTML)
> - 1 RTT for the HTTP request/response (base HTML) + transmission time
> - 5 × 2 RTT for the 5 JPEG images (each needs its own TCP connection)
> - Total: approximately **2 + 5×2 = 12 RTTs** + transmission times (ignoring parallel connections)
> 
> **Persistent HTTP with pipelining:**
> 
> - 1 RTT to initiate TCP connection
> - 1 RTT for base HTML
> - 1 RTT for all 5 images (pipelined together)
> - Total: approximately **3 RTTs** + transmission times

**Q7.** What is the difference between the POST and GET methods? When should each be used?

> **Answer:**
> 
> - **GET:** Retrieves a resource. Data (if any) is appended to the URL as query parameters (e.g., `?q=search+term`). Idempotent and safe. Suitable for search queries, pagination, bookmarkable actions. Not suitable for sensitive data.
>     
> - **POST:** Submits data to the server. Data is in the message body (not the URL). Not idempotent. Suitable for form submissions, file uploads, login credentials, creating resources on the server.
>     

**Q8.** What is the purpose of the `If-Modified-Since` header? What are the possible server responses?

> **Answer:** The `If-Modified-Since` header is used in a **Conditional GET** to avoid re-downloading an object that hasn't changed. The client (or cache) sends the date of its cached copy; the server responds:
> 
> - **HTTP/1.0 304 Not Modified** — the cached copy is still current; no object is sent. Zero bandwidth wasted.
> - **HTTP/1.0 200 OK** — the object has changed; the new version is included in the response body.

---

### Section D: Caching

**Q9.** An institutional network has an access link rate of 1.54 Mbps. The average data rate from browsers to origin servers is 1.50 Mbps. What is the access link utilization, and what is the problem?

> **Answer:**
> 
> Utilization = 1.50 / 1.54 = **0.97 (97%)**
> 
> At near-saturation (approaching 100% utilization), **queueing delays become extremely large** (potentially minutes). This is because queuing delay grows non-linearly as utilization approaches 1. The link becomes a severe bottleneck.

**Q10.** With a local web cache having a hit rate of 0.4, what is the new access link utilization and average end-to-end delay (using the scenario from Q9, RTT from institutional router to server = 2 sec)?

> **Answer:**
> 
> With 40% hit rate, only 60% of requests go to origin server:
> 
> - Rate over access link = 0.6 × 1.50 = **0.9 Mbps**
> - Access link utilization = 0.9 / 1.54 = **0.58 (58%)** → low queueing delay
> 
> Average end-to-end delay:
> 
> - 60% served from origin: delay ≈ 2.01 sec (2 sec RTT + minimal access link delay)
> - 40% served from cache: delay ≈ milliseconds
> - Average = 0.6 × 2.01 + 0.4 × ~0.01 = **~1.21 seconds**

---

### Section E: FTP

**Q11.** How does FTP differ from HTTP in terms of its connections?

> **Answer:**
> 
> - **HTTP** uses a single TCP connection for both control (commands) and data (file content). HTTP is "in band."
>     
> - **FTP** uses **two separate TCP connections**:
>     
>     - **Control connection (port 21):** Persistent throughout the session; carries commands and replies (ASCII text)
>     - **Data connection (port 20):** Opened fresh for each file transfer; carries the actual file data; closed after each file
> 
> FTP is described as "out of band" because its control information travels on a different channel from its data.

---

### Section F: Email

**Q12.** Is SMTP a push or pull protocol? Explain.

> **Answer:** SMTP is a **push** protocol. The sending mail server _pushes_ the email to the receiving mail server. The sender initiates the connection and sends the data. This contrasts with HTTP, which is a pull protocol (clients pull resources from servers on demand).

**Q13.** Compare POP3 and IMAP. In what situation would you prefer IMAP?

> **Answer:**
> 
> ||POP3|IMAP|
> |---|---|---|
> |Storage|Downloads to client; can delete from server|Keeps all mail on server|
> |Multi-device|Problematic (messages on one device only in delete mode)|Excellent (same view on all devices)|
> |Folders|Local only|Server-side, synced everywhere|
> |State|Stateless across sessions|Stateful|
> 
> **Prefer IMAP when:**
> 
> - Using multiple devices (phone + laptop + web browser)
> - You want to access mail from anywhere without losing messages
> - You want server-side folder organization
> 
> **Prefer POP3 when:**
> 
> - You want mail stored locally (privacy, offline access)
> - You only use one device for email
> - Low server storage quota

**Q14.** Why does SMTP require messages to be in 7-bit ASCII? How does this affect binary attachments?

> **Answer:** SMTP was designed in the early days of the internet when 7-bit ASCII was the universal character encoding. The protocol interprets all data as ASCII text.
> 
> Binary files (images, documents, executables) cannot be represented in 7-bit ASCII. To send them as email attachments, they must be **encoded into ASCII** using schemes like **Base64**. Base64 converts every 3 bytes of binary data into 4 ASCII characters, increasing size by ~33%. When the recipient's mail client receives the message, it decodes the Base64 back to binary. This encoding/decoding is handled automatically by modern email clients (MIME — Multipurpose Internet Mail Extensions).

---

### Section G: DNS

**Q15.** Why is a distributed, hierarchical DNS better than a single centralized DNS server?

> **Answer:** A centralized DNS would have these critical problems:
> 
> 1. **Single point of failure:** If it goes down, the entire internet's name resolution fails.
> 2. **Traffic volume:** Billions of DNS queries per day — no single server can handle this load.
> 3. **Geographic delay:** Users far from the central server would experience high latency.
> 4. **Maintenance bottleneck:** Every new domain and IP change must be registered to one server in real-time.
> 
> The distributed hierarchical design solves all these: it's resilient (no single point of failure), scalable (distributed load), fast (local caching, geographically close servers), and manageable (each organization manages its own authoritative server).

**Q16.** What is the difference between iterative and recursive DNS resolution?

> **Answer:**
> 
> **Iterative:** The local DNS server contacts each DNS server in the hierarchy _itself_. When a server can't resolve the name, it replies "I don't know, but ask this server." The local DNS server then contacts that next server, and so on. **The local DNS server does all the work.**
> 
> **Recursive:** Each DNS server, when it can't answer, takes responsibility for contacting the next server in the hierarchy and returning the final answer. **The work is distributed up the hierarchy.** This puts heavy load on root and TLD servers, which is why iterated queries are more common in practice.

**Q17.** What are the four types of DNS resource records? Give an example of each.

> **Answer:**
> 
> |Type|Example|
> |---|---|
> |**A** (Address)|`gaia.cs.umass.edu` → `128.119.245.12`|
> |**NS** (Name Server)|`umass.edu` → `dns.cs.umass.edu`|
> |**CNAME** (Canonical Name / Alias)|`www.ibm.com` → `servereast.backup2.ibm.com`|
> |**MX** (Mail Exchange)|`foo.com` → `mail.foo.com`|

**Q18.** Explain DNS caching. What is a TTL and why does it matter?

> **Answer:** Once any name server learns a name-to-IP mapping, it stores it in its **cache** and returns this cached answer to future queries — without contacting higher-level servers. This greatly speeds up DNS resolution.
> 
> **TTL (Time To Live)** is a value (in seconds) set by the authoritative server indicating how long the cached entry is valid. When the TTL expires, the cache entry is deleted and must be re-fetched.
> 
> **Why TTL matters:**
> 
> - Too short → frequent re-queries, more traffic to authoritative servers
> - Too long → if the authoritative server changes the IP (e.g., the host moves), cached entries will be stale, pointing to the wrong IP. Changes can take the full TTL duration to propagate across the internet.
> 
> This is the **"best-effort" nature** of DNS caching — there's no guarantee that cached entries are current.

---

### Section H: Tricky/Conceptual Questions

**Q19.** Why does HTTP run over TCP rather than UDP?

> **Answer:** HTTP is used to transfer web pages, which must arrive **completely and correctly**. A missing or corrupted byte in an HTML file or image renders it useless. Therefore, HTTP requires **reliable data transfer**, which is provided by TCP (guarantees delivery, order, and integrity). UDP provides no such guarantees, so it would require HTTP itself to implement reliability — defeating the purpose of using UDP's simplicity. The latency cost of TCP setup is acceptable for web browsing. (Modern HTTP/3 runs over QUIC, which is built on UDP but implements its own reliability mechanisms.)

**Q20.** Suppose you type `http://www.google.com` in your browser. Describe all the steps that happen, from DNS lookup to receiving the webpage.

> **Answer:**
> 
> 1. **DNS Resolution:**
>     - Browser checks its own DNS cache. If not found, asks the OS.
>     - OS checks its DNS cache. If not found, sends query to **local DNS server** (configured by ISP/router).
>     - Local DNS server checks its cache. If not found, queries **root DNS server** → **TLD (.com) DNS server** → **Google's authoritative DNS server**.
>     - Google's DNS server returns the IP address (e.g., `142.250.80.46`).
>     - Result cached at each level with respective TTLs.
> 2. **TCP Connection:**
>     - Browser initiates a **TCP three-way handshake** with `142.250.80.46` on port 80 (or 443 for HTTPS).
>     - SYN → SYN-ACK → ACK
> 3. **HTTP Request:**
>     - Browser sends HTTP GET request: `GET / HTTP/1.1\r\nHost: www.google.com\r\n...`
> 4. **HTTP Response:**
>     - Google's web server sends back `HTTP/1.1 200 OK` followed by the HTML of the homepage.
> 5. **Rendering + Fetching Sub-objects:**
>     - Browser parses the HTML, finds referenced objects (images, CSS, JavaScript).
>     - For each object, sends additional HTTP GET requests (possibly over the same persistent TCP connection).
> 6. **Display:**
>     - Browser renders the page as objects arrive.

---

_End of Notes_

---

> **Created for Obsidian** | Computer Networking: A Top-Down Approach — Kurose & Ross | Chapter 2: Application Layer