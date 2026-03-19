# 🚚 Transport Layer — Part I: Complete Study Notes

> **Source:** Kurose & Ross — Computer Networking: A Top-Down Approach  
> **Tags:** #networking #transport-layer #TCP #UDP #rdt #multiplexing #study-notes  
> **Status:** 📚 Complete Reference

---

## 📋 Table of Contents

1. [[#1. Transport Layer Overview]]
2. [[#2. Transport vs Network Layer]]
3. [[#3. Transport Layer Actions — Sender and Receiver]]
4. [[#4. TCP vs UDP — The Two Internet Transport Protocols]]
5. [[#5. Multiplexing and Demultiplexing]]
6. [[#6. How Demultiplexing Works]]
7. [[#7. UDP — User Datagram Protocol]]
8. [[#8. UDP Segment Header]]
9. [[#9. Principles of Reliable Data Transfer (rdt)]]
10. [[#10. rdt1.0 — Reliable Channel (The Easy Case)]]
11. [[#11. rdt2.0 — Channel with Bit Errors]]
12. [[#12. rdt3.0 — Channel with Errors AND Loss]]
13. [[#13. Stop-and-Wait Performance Problem]]
14. [[#14. Pipelining — The Solution to Stop-and-Wait]]
15. [[#15. Go-Back-N (GBN)]]
16. [[#16. Selective Repeat (SR)]]
17. [[#17. GBN vs Selective Repeat — Side-by-Side Comparison]]
18. [[#18. Practice Questions (Kurose & Ross Style)]]

---

## 1. Transport Layer Overview

### What Does the Transport Layer Do?

The transport layer provides **logical communication** between **application processes** running on _different hosts_.

The word **"logical"** is key here. It means that from the application's perspective, it feels like the two processes are directly connected — like there's a private pipe between them — even though in reality, messages travel through a complex web of routers, switches, and links across the entire internet.

### 🧠 Simple Analogy

Imagine you're mailing letters between two friends in different cities. The _postal system_ (network layer) handles getting letters from city to city. But the transport layer is like the _sorting room inside each house_ — it makes sure each letter gets to the right person (process) in that house, and on the sending end, collects all outgoing letters from everyone in the house and hands them to the postal system.

### What the Transport Layer Actually Does

**At the sender:**

- Takes application messages
- Breaks them into **segments** (smaller chunks)
- Adds a transport header to each segment
- Passes segments to the **network layer** (IP) for delivery

**At the receiver:**

- Receives segments from the network layer (IP)
- Checks header values
- Reassembles segments back into the original message
- Passes the message up to the correct application via a **socket**

### The Two Internet Transport Protocols

- **TCP** — Transmission Control Protocol
- **UDP** — User Datagram Protocol

---

## 2. Transport vs Network Layer

This is one of the most important distinctions to understand clearly:

|Layer|Provides Logical Communication Between...|Analogy|
|---|---|---|
|**Network Layer (IP)**|**Hosts** (machines)|Postal system delivers to the _building_|
|**Transport Layer (TCP/UDP)**|**Processes** (running programs)|The _sorting room_ routes mail to the right _person_ in the building|

### Key Point

The transport layer **relies on** and **enhances** the network layer:

- It uses the network layer to actually move bits across the internet
- It adds services the network layer doesn't provide (like reliability, ordering, multiplexing)

### ⚠️ Common Confusion

> "Doesn't the network layer already connect two computers? Why do we need the transport layer too?"

The network layer delivers a packet to the **correct machine**. But that machine might be running 50 different applications simultaneously (web browser, Spotify, Skype, email client...). The transport layer is what says: _"This particular data is for the Firefox browser, not for Spotify."_ It extends host-to-host delivery to process-to-process delivery.

> "Can the transport layer fix all of IP's problems?"

No. The transport layer can provide reliability _on top of_ IP, but it cannot provide guarantees that require the network layer's cooperation — like bandwidth guarantees or delay guarantees. Those would require Quality of Service (QoS) mechanisms in routers. TCP provides reliable delivery even over an unreliable IP network, but it cannot guarantee _when_ the data arrives.

---

## 3. Transport Layer Actions — Sender and Receiver

### Sender Side

```
Application Layer
      ↓  (app message)
Transport Layer
  1. Receives application message
  2. Determines header field values (ports, seq#, checksum...)
  3. Creates segment: [Header | App Message]
  4. Passes segment down to IP (Network Layer)
Network Layer (IP)
```

### Receiver Side

```
Network Layer (IP)
      ↓  (segment arrives)
Transport Layer
  1. Receives segment from IP
  2. Checks header values (is the checksum correct? what port?)
  3. Extracts the application-layer message from segment
  4. Demultiplexes: sends message up to the correct application via socket
Application Layer
```

### The Encapsulation Picture

Each layer **wraps** the data from the layer above it with its own header:

```
Application message:         [    DATA    ]
Transport segment:    [Ht |  DATA    ]         ← Transport header added
Network datagram:  [Hn | Ht |  DATA    ]      ← Network header added
Link frame:    [Hl | Hn | Ht | DATA | Tl]    ← Link header + trailer added
```

When data arrives at the destination, each layer **unwraps** (removes) its header and passes the rest up.

---

## 4. TCP vs UDP — The Two Internet Transport Protocols

### TCP (Transmission Control Protocol)

TCP is the **feature-rich, reliable** protocol. It provides:

|Feature|What It Means|
|---|---|
|**Reliable, in-order delivery**|Every byte sent arrives exactly once, in the right order|
|**Congestion control**|Slows down when the _network_ is overloaded|
|**Flow control**|Slows down when the _receiver_ is overwhelmed|
|**Connection setup**|Requires a 3-way handshake before any data flows|

TCP does **NOT** provide: timing guarantees, minimum throughput guarantees, or built-in security (TLS adds that on top).

### UDP (User Datagram Protocol)

UDP is the **lightweight, best-effort** protocol. It provides:

- **Unreliable, unordered delivery** — packets may be lost, duplicated, or arrive out of order
- No connection setup, no state maintained
- Just "fire and forget"

### ⚠️ Common Confusion

> "If TCP is so much better, why use UDP at all?"

Because "better" depends on your needs. TCP's reliability comes at a _cost_:

- **Connection setup delay** (the 3-way handshake takes at least 1 RTT before any data flows)
- **Head-of-line blocking** — TCP waits for a missing packet before delivering later ones, which causes stalls in real-time apps
- **Congestion control overhead** — TCP throttles itself, which adds unpredictability
- **State maintenance** — TCP keeps track of sequence numbers, buffers, timers

For live video calls, online games, DNS lookups, and similar applications, UDP's "just send it" approach is actually _preferable_.

---

## 5. Multiplexing and Demultiplexing

### The Problem

A host can run **many applications simultaneously** — your laptop might be running a web browser, a music streaming app, a video call, and an email client all at once. Data from the network arrives at the host as a single stream. How does the transport layer know which data belongs to which application?

This is solved by **multiplexing** (at the sender) and **demultiplexing** (at the receiver).

### Multiplexing (at Sender)

**Definition:** Gathering data from _multiple sockets_, encapsulating each piece with a transport header (containing source and destination port numbers), and passing the resulting segments to the network layer.

```
Firefox socket ──→ ┐
Skype socket   ──→ ├──[Transport Layer: adds headers]──→ Network Layer
Netflix socket ──→ ┘
```

Think of it like a post office collecting all outgoing mail from different departments and putting them in one mail truck with address labels.

### Demultiplexing (at Receiver)

**Definition:** Using the header information in received segments to direct each segment to the correct socket (and thus the correct application process).

```
Network Layer ──→ [Transport Layer: reads headers] ──→ Firefox socket
                                                   ──→ Skype socket
                                                   ──→ Netflix socket
```

Think of it like a mail carrier sorting incoming packages to different apartments in a building.

### The Key Insight

The header information that makes this work includes **port numbers**. The segment carries:

- **Source port number** — which port the sender used
- **Destination port number** — which port on the receiver should get this data

### Multiplexing/Demultiplexing Happen at ALL Layers

Not just transport! Every layer performs some form of mux/demux:

- Link layer uses MAC addresses to demux to the right network-layer protocol
- Network layer uses IP addresses to demux to the right transport-layer protocol
- Transport layer uses port numbers to demux to the right socket/application

---

## 6. How Demultiplexing Works

### The Segment Header Fields

Every TCP/UDP segment carries at minimum:

```
┌─────────────────┬─────────────────┐
│  Source Port #  │   Dest Port #   │  ← 16 bits each
├─────────────────┴─────────────────┤
│         Other header fields        │
├───────────────────────────────────┤
│         Application Data           │
│           (Payload)                │
└───────────────────────────────────┘
```

### The Process

1. Host receives an IP datagram (packet)
2. Each datagram carries one transport-layer segment
3. Each datagram has a **source IP address** and **destination IP address**
4. Each segment has a **source port #** and **destination port #**
5. The host uses **IP addresses + port numbers** to direct the segment to the right socket

### UDP Demultiplexing (2-tuple)

UDP uses only **destination IP + destination port** to identify the right socket.

This means: two UDP segments with the _same_ destination IP and port but _different_ source IPs/ports → both go to the **same socket**.

### TCP Demultiplexing (4-tuple)

TCP uses all four: **(source IP, source port, destination IP, destination port)** to identify the socket.

This means: two TCP segments with the same destination IP and port but _different_ source IPs/ports → go to **different sockets** (different connections, possibly different server processes).

### The Critical Example from the Slides

```
Server B (IP: B, port 80) receives three segments:
  1. Source: A:9157, Dest: B:80  → goes to process P4 (socket for connection with A)
  2. Source: C:5775, Dest: B:80  → goes to process P5 (socket for connection with C:5775)
  3. Source: C:9157, Dest: B:80  → goes to process P6 (socket for connection with C:9157)
```

All three segments have the same destination (B:80), but because TCP uses the full 4-tuple, they go to **different sockets**. This is how a web server can handle thousands of simultaneous connections from different clients — each connection gets its own socket.

### ⚠️ Common Confusion

> "If UDP demultiplexes only by destination port, doesn't that mean two different clients talking to the same UDP server get mixed together?"

Yes and no. The data is sent to the _same socket_, but the application code can read the source IP and port from each received packet to distinguish who sent what. DNS servers, for example, handle this — they receive UDP queries from many clients on port 53, and they know which client to respond to because each query arrives with a source IP/port.

> "Why does TCP need all 4 fields but UDP only needs 2?"

Because TCP is _connection-oriented_. Each TCP connection is a unique relationship between a specific client and server. The 4-tuple uniquely identifies each connection. UDP is connectionless — each packet is independent, and there's no "connection" to track.

---

## 7. UDP — User Datagram Protocol

### What is UDP?

UDP is the transport layer's **"bare minimum"** offering. It does just enough to be useful: it adds port numbers (for multiplexing/demultiplexing) and a checksum (for basic error detection), and nothing else.

### UDP's Characteristics

|Property|Detail|
|---|---|
|**"Best effort" service**|Segments may be lost or delivered out of order|
|**Connectionless**|No handshaking before sending|
|**No connection state**|No buffers, no sequence numbers, no congestion control variables at sender or receiver|
|**Small header**|Only 8 bytes of overhead|
|**No congestion control**|UDP can send as fast as it wants — no throttling|

### Why Use UDP? (The Full Answer)

1. **No connection establishment delay:** UDP just sends immediately. No 3-way handshake (which takes 1 RTT). DNS uses UDP for exactly this reason — a query-response is just 2 messages, and the overhead of establishing a TCP connection would double the time.
    
2. **No connection state:** A server using UDP can support many more simultaneous clients than a TCP server, because it doesn't maintain state (buffers, sequence numbers, etc.) for each client.
    
3. **Small header overhead:** UDP adds only 8 bytes. TCP adds 20+ bytes. For small, frequent messages, this matters.
    
4. **No congestion control — sender controls rate:** UDP can send at whatever rate the application desires. Useful for streaming applications that would rather drop frames than buffer. Also useful in the rare case where the application itself implements a smarter congestion control strategy.
    
5. **Can work even when the network is congested:** Because UDP doesn't back off during congestion (unlike TCP), it can "blast through" — though this is actually a concern for network stability.
    

### UDP Applications

|Application|Why UDP|
|---|---|
|**Streaming multimedia**|Loss-tolerant, rate-sensitive; a dropped frame is better than buffering|
|**DNS**|Simple request-response; no need for connection overhead|
|**SNMP** (network management)|Simple, infrequent queries|
|**HTTP/3 (QUIC)**|Implements its own reliability on top of UDP, avoiding TCP's head-of-line blocking|

### ⚠️ Common Confusion

> "Can you get reliability over UDP?"

Yes! If an application needs reliability but doesn't want TCP's specific tradeoffs (like its congestion control behavior), it can implement reliability itself at the _application layer_. HTTP/3 does exactly this with the QUIC protocol — it runs over UDP and implements its own reliability, ordering, and congestion control in a way that avoids TCP's limitations (like head-of-line blocking).

> "Does UDP have no error detection at all?"

UDP does have a **checksum** for error _detection_ (it can tell if bits flipped in transit), but it does nothing about errors — it just detects them. The application may or may not care. There's no _correction_ or _retransmission_ in UDP.

---

## 8. UDP Segment Header

### Structure (Only 8 Bytes Total!)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├───────────────────────────┬───────────────────────────────────────┤
│      Source Port (16)     │       Destination Port (16)           │
├───────────────────────────┼───────────────────────────────────────┤
│       Length (16)         │          Checksum (16)                │
├───────────────────────────┴───────────────────────────────────────┤
│                     Application Data (Payload)                     │
│                              ...                                   │
└───────────────────────────────────────────────────────────────────┘
```

### Field Descriptions

|Field|Size|Purpose|
|---|---|---|
|**Source Port**|16 bits|Port of the sending process|
|**Destination Port**|16 bits|Port of the receiving process|
|**Length**|16 bits|Length of the entire UDP segment (header + data), in bytes. Minimum = 8 (header only)|
|**Checksum**|16 bits|Used to detect errors (bit flips) in the segment|

### The Checksum

The checksum is computed over the UDP header, UDP data, and parts of the IP header. The receiver recomputes it and checks for a match. If they don't match → error detected. What happens then depends on the application.

### ⚠️ Common Confusion

> "The length field says length in bytes — why is the minimum 8?"

Because the header itself is 8 bytes (4 fields × 2 bytes each). Even if there's no data (empty payload), the segment is still 8 bytes. So the minimum value of the length field is 8.

---

## 9. Principles of Reliable Data Transfer (rdt)

### The Core Problem

The **application layer** wants a _reliable_ channel — it hands data to the transport layer and expects it to arrive intact, in order, without loss. But the **underlying network** (IP) is _unreliable_ — it can lose packets, corrupt bits, and reorder things.

```
WHAT THE APP SEES (abstraction):
Sending Process ──────[reliable channel]──────→ Receiving Process
     data                                              data ✓

WHAT ACTUALLY HAPPENS (reality):
Sending Process → [sender-side rdt] → [unreliable IP network] → [receiver-side rdt] → Receiving Process
     data                                 ↑ drops, corrupts,                               data ✓
                                            reorders packets
```

The transport layer's job is to **implement the reliable service abstraction on top of an unreliable channel**.

### The Fundamental Challenge

Sender and receiver do **not know each other's state** unless they explicitly communicate. The sender doesn't know if the receiver got the last packet. The receiver doesn't know if the sender thinks everything is OK. This uncertainty is what makes reliable data transfer hard.

### What Can Go Wrong?

As we build up our rdt protocols, we'll deal with increasingly severe channel problems:

|Version|Channel Problems|Solution Needed|
|---|---|---|
|rdt1.0|None (perfect channel)|Nothing — trivial|
|rdt2.0|Bit errors (corruption)|Error detection + ACK/NAK|
|rdt3.0|Bit errors + packet loss|+ Timeout + retransmission|

### The Building Blocks of Reliable Transfer

|Mechanism|Purpose|
|---|---|
|**Checksum**|Detect bit errors in transmitted packets|
|**Acknowledgements (ACK)**|Receiver tells sender: "Got it, it was correct"|
|**Negative Acknowledgements (NAK)**|Receiver tells sender: "Got it, but it was corrupted"|
|**Sequence numbers**|Number packets so sender/receiver can detect duplicates and reordering|
|**Timeout/Retransmission**|If no ACK received within a time limit, sender retransmits|
|**Pipelining**|Send multiple packets without waiting for each ACK (improves efficiency)|

---

## 10. rdt1.0 — Reliable Channel (The Easy Case)

### Assumption

The underlying channel is **perfectly reliable**:

- No bit errors
- No packet loss

### Behavior

This is trivially easy:

- Sender just sends data
- Receiver just receives data
- No error handling needed at all

```
Sender:                    Receiver:
  call from above(data)      rdt_rcv(packet)
  → make_pkt(data)           → extract(packet, data)
  → udt_send(packet)         → deliver_data(data)
```

### Why Study This?

rdt1.0 is the baseline. We use it to understand what needs to be added as we make the channel worse. Every subsequent version adds mechanisms to handle a new type of channel imperfection.

---

## 11. rdt2.0 — Channel with Bit Errors

### New Channel Assumption

The channel **may flip bits** in a packet (errors can occur), but packets are **not lost**.

### The Problem

How does the receiver tell the sender that a packet arrived corrupted?

### 🧠 Human Analogy

Think about a phone call in a noisy room:

- Speaker says something
- Listener says "**OK!**" (ACK) if they heard it
- Listener says "**What? Say again!**" (NAK) if they didn't understand

### The Solution: ACKs and NAKs

|Message|Meaning|
|---|---|
|**ACK (Acknowledgement)**|"Received correctly — send the next one"|
|**NAK (Negative Acknowledgement)**|"Received with errors — please retransmit"|

### The Stop-and-Wait Paradigm

rdt2.0 uses **stop-and-wait**: the sender sends _one_ packet, then **stops and waits** for the receiver to respond (either ACK or NAK) before sending the next packet.

```
Sender:
  send pkt → [waiting for ACK/NAK]
  receive NAK → retransmit same pkt
  receive ACK → send next pkt

Receiver:
  receive pkt (no errors) → send ACK
  receive pkt (errors detected) → send NAK
```

### The Fatal Flaw of rdt2.0

What if the **ACK or NAK itself gets corrupted**?

```
Sender sends pkt0 → Receiver gets it fine → Sends ACK
But ACK gets corrupted → Sender can't tell if it was ACK or NAK
```

The sender doesn't know what to do! If it retransmits, the receiver might get a duplicate. If it doesn't, the packet might have been corrupted.

### The Fix: Sequence Numbers

Add a **sequence number** to each packet. With just 2 states (sending/retransmitting), we only need **1-bit sequence numbers** (0 or 1).

Now if the sender receives a garbled ACK/NAK, it retransmits. The receiver uses the sequence number to detect duplicates (if it gets the same seq# twice, it knows it's a duplicate).

### ⚠️ Common Confusion

> "Why are only 1-bit sequence numbers needed for stop-and-wait?"

Because in stop-and-wait, only **one packet is "in flight"** at any time. There are only two relevant states: "the current new packet" (0) and "the retransmission of the current packet" (1). You just need to distinguish "is this a new packet or the same one again?" — 1 bit is enough for that.

---

## 12. rdt3.0 — Channel with Errors AND Loss

### New Channel Assumption

The channel can now **both corrupt bits AND lose packets entirely** (data packets or ACK packets can disappear).

### The New Problem

Checksums + ACK/NAK + sequence numbers handle corruption. But what happens when a packet just **vanishes**? The receiver never gets it, so it never sends an ACK. The sender waits forever.

### 🧠 Human Analogy

Imagine sending a letter and waiting for a reply. If the letter or reply gets lost in the mail, you'll wait forever. Real people set a deadline: "If I don't hear back in a week, I'll send another letter."

### The Solution: Timeout and Retransmission

The sender starts a **countdown timer** when it sends a packet. If no ACK arrives within a "reasonable" time (the timeout period), the sender **retransmits** the packet.

```
Sender:
  send pkt, start timer
  if ACK received before timeout → stop timer, send next pkt
  if timeout fires → retransmit same pkt, restart timer
```

### The "Reasonable" Timeout

Setting the right timeout is tricky:

- **Too short:** Retransmit too eagerly — causes unnecessary retransmissions and wastes bandwidth
- **Too long:** Wait too long for a lost packet — hurts throughput

In practice (TCP), the timeout is estimated based on measured round-trip times (RTTs).

### What Happens in Each Scenario

**Case (a): No loss**

```
Sender sends pkt0 → Receiver gets it → ACK0 → Sender gets ACK0 → sends pkt1 → ...
Normal operation, no issues.
```

**Case (b): Packet loss**

```
Sender sends pkt1 → [pkt1 lost in network] → Timeout fires → Sender retransmits pkt1
→ Receiver gets pkt1 → sends ACK1 → Sender continues with pkt0
```

**Case (c): ACK loss**

```
Sender sends pkt1 → Receiver gets it → Sends ACK1 → [ACK1 lost] → Timeout fires
→ Sender retransmits pkt1 → Receiver gets pkt1 AGAIN (duplicate!)
→ Receiver detects duplicate (same seq#) → re-sends ACK1 → ignores the data
→ Sender gets ACK1 → continues
Sequence numbers handle duplicates!
```

**Case (d): Premature timeout / delayed ACK**

```
Sender sends pkt1 → Receiver gets it → Sends ACK1 → ACK1 is delayed (not lost)
→ Timeout fires before ACK arrives → Sender retransmits pkt1
→ Receiver gets pkt1 again (duplicate) → re-sends ACK1 (ignore duplicate data)
→ The delayed ACK1 finally arrives → Sender receives ACK1 (ignores it — already moved on)
→ Sender then receives the second ACK1 → also ignored
```

### ⚠️ Common Confusion

> "If the timeout fires and the original packet was just delayed (not lost), won't retransmitting cause problems?"

Yes — it causes a **duplicate**. But sequence numbers handle this! The receiver sees it received seq#1 already, knows it's a duplicate, discards the duplicate data, but **still sends an ACK** (because maybe the sender didn't get the first ACK). The sender needs the ACK to proceed. So: duplicates are always acknowledged but never delivered to the application twice.

> "Why does rdt3.0 only use 0 and 1 as sequence numbers if packets can be lost and delayed?"

With stop-and-wait (only one packet in flight at a time), you still only need 2 sequence numbers. However, this becomes a problem in extreme cases (very long delays), which is part of why real protocols use larger sequence number spaces.

---

## 13. Stop-and-Wait Performance Problem

### The Utilization Formula

Stop-and-wait is incredibly inefficient. Here's why:

**Sender utilization** = fraction of time the sender is actually sending data (vs. idle, waiting for ACK).

$$U_{sender} = \frac{L/R}{RTT + L/R}$$

Where:

- $L$ = packet size in bits
- $R$ = link bandwidth in bits/second
- $L/R$ = transmission time for one packet
- $RTT$ = round-trip time

### Example Calculation

- Link bandwidth: $R = 1$ Gbps
- Packet size: $L = 8000$ bits (1 KB)
- RTT = 30 ms (cross-continent link)
- $L/R = 8000 / 10^9 = 0.008$ ms (8 microseconds)

$$U_{sender} = \frac{0.008}{30 + 0.008} = \frac{0.008}{30.008} \approx 0.00027 = 0.027%$$

**The sender is idle 99.973% of the time!** The link is almost completely wasted.

### Why Is It So Bad?

Because $RTT >> L/R$. The packet takes only 8 microseconds to transmit, but then the sender waits 30 milliseconds for the ACK. For 30 ms, the link carries nothing. The sender is blocked, twiddling its thumbs.

### 🧠 Analogy

Imagine sending a single truck across the country, waiting for it to arrive and the driver to call back before sending the next truck. The roads (link) are nearly empty most of the time. The solution? Send many trucks at once without waiting for each one to confirm arrival!

---

## 14. Pipelining — The Solution to Stop-and-Wait

### What is Pipelining?

**Pipelining** means the sender is allowed to have **multiple packets "in flight"** simultaneously — sent but not yet acknowledged.

```
Stop-and-Wait:
[pkt0]────→────→────→────→ ACK ←────←────←
          [waiting.....]
[pkt1]────→────→────→────→ ACK ←────←────←
          [waiting.....]

Pipelined:
[pkt0]────→
[pkt1]──────→
[pkt2]────────→
[pkt3]──────────→          ACKs stream back
```

### What Pipelining Requires

1. **Larger sequence number range** — with multiple packets in flight, you need more sequence numbers to uniquely identify each one.
    
2. **Buffering at sender** — the sender must buffer unacknowledged packets in case they need to be retransmitted.
    
3. **Buffering at receiver** — (depending on the protocol) the receiver may need to buffer out-of-order packets.
    

### Pipelining Utilization Formula

With a window of $N$ packets in flight:

$$U_{sender} = \frac{N \times L/R}{RTT + L/R}$$

Using the same example as before with $N = 3$:

$$U_{sender} = \frac{3 \times 0.008}{30.008} = \frac{0.024}{30.008} \approx 0.00081 = 0.081%$$

**3× improvement!** With $N = 3000$ packets in flight, you'd get near 100% utilization.

### Two Pipelined Protocol Approaches

There are two main strategies for handling errors in a pipelined setting:

1. **Go-Back-N (GBN)**
2. **Selective Repeat (SR)**

They differ in _what gets retransmitted_ when a packet is lost.

---

## 15. Go-Back-N (GBN)

### Sender Side

**The Sliding Window:**

The sender maintains a "window" of up to **N consecutive packets** that can be sent but not yet acknowledged.

```
Sequence numbers:
| already | sent, not  | usable,  | not    |
| ACK'd   | yet ACK'd  | not sent | usable |
│▓▓▓▓▓▓▓│▒▒▒▒▒▒▒│░░░░░░░│        │
         ↑           ↑
      send_base   nextseqnum
         ←── window size N ──→
```

- **Already ACK'd (green):** These are done. Window has moved past them.
- **Sent, not yet ACK'd (yellow):** In flight. Waiting for ACKs. These are buffered in case of retransmission.
- **Usable, not yet sent (blue):** Can be sent immediately (window allows it).
- **Not usable (white):** Outside the window — can't send until window moves forward.

**GBN Sender Rules:**

|Event|Action|
|---|---|
|Call from above (new data)|If window has room, make & send packet; else refuse/buffer|
|Receive ACK(n)|Advance window base to n+1 (**cumulative ACK** — acknowledges all ≤ n)|
|Timeout|**Retransmit ALL unACK'd packets** in the window (packet n through last sent)|

### Receiver Side

**GBN Receiver Rules:**

- **ACK-only:** Receiver sends an ACK only for the _highest in-order_ packet received so far.
- **Out-of-order packets:** Can be discarded (no buffering required) _or_ buffered (implementation choice). Either way, the receiver re-ACKs the last correctly received in-order packet.
- **Only need to remember:** `rcv_base` — the sequence number of the next expected in-order packet.

### GBN in Action (from slides — window N=4)

```
Sender sends: pkt0, pkt1, pkt2, pkt3 (window full)
pkt2 is LOST

Receiver:
  rcv pkt0 → ACK0 ✓
  rcv pkt1 → ACK1 ✓
  rcv pkt3 → discard! (out of order), re-send ACK1
  rcv pkt4 → discard! (out of order), re-send ACK1
  rcv pkt5 → discard! (out of order), re-send ACK1

Sender:
  rcv ACK0 → advance window, send pkt4
  rcv ACK1 → advance window, send pkt5
  ... receives duplicate ACK1s (ignores them)
  pkt2 TIMEOUT → retransmit pkt2, pkt3, pkt4, pkt5 (ALL from pkt2 onwards!)

Receiver:
  rcv pkt2 → in-order! ACK2 ✓
  rcv pkt3 → in-order! ACK3 ✓
  rcv pkt4 → in-order! ACK4 ✓
  rcv pkt5 → in-order! ACK5 ✓
```

Notice: pkt3, pkt4, pkt5 were correctly received and discarded the first time, then had to be retransmitted. That's the cost of GBN.

### ⚠️ Common Confusion

> "Does the GBN receiver buffer out-of-order packets or not?"

The slides say "can discard (don't buffer) or buffer." The _classic_ GBN design discards out-of-order packets to keep the receiver simple (no buffering needed). But many real implementations buffer them. The key property is the _ACK behavior_ — GBN uses **cumulative ACKs** (ACK n = "I have everything up to n").

> "What's a cumulative ACK?"

ACK(n) means "I have correctly received all packets through sequence number n." It's like a receipt that says "I have chapters 1 through 7 of your book" — it implicitly covers all chapters up to 7, not just chapter 7.

---

## 16. Selective Repeat (SR)

### The Problem with GBN

GBN retransmits ALL unACK'd packets when one is lost. If the window is large and packet loss is frequent, this wastes bandwidth enormously — you're retransmitting packets the receiver already has!

**Example:** With window N=100 and pkt5 lost, GBN retransmits pkts 5–104 (100 packets!), even though pkts 6–104 were received correctly.

### Selective Repeat's Approach

**Only retransmit the packets that were actually lost or corrupted.**

The receiver **individually acknowledges** each correctly received packet and **buffers** out-of-order packets until the missing ones are retransmitted and the sequence can be delivered in order.

### SR Sender

|Event|Action|
|---|---|
|Call from above|If seq# in window, send packet and start timer for that specific packet|
|Receive ACK(n)|Mark packet n as ACK'd. If n = send_base, advance window to next unACK'd packet|
|Timeout(n)|**Only retransmit packet n** (the specific one that timed out), restart its timer|

**Key difference from GBN:** SR maintains a **separate timer for each unACK'd packet**. On timeout, only that _one_ packet is retransmitted.

### SR Receiver

|Event|Action|
|---|---|
|Receive pkt with seq# in [rcv_base, rcv_base+N-1]|Send ACK(n). If not already received, buffer it. If n=rcv_base, deliver buffered in-order packets up through the gap|
|Receive pkt with seq# in [rcv_base-N, rcv_base-1]|Send ACK(n) — this is a packet the receiver already ACK'd, but the sender re-sent it because its ACK was lost|
|Otherwise|Ignore|

### SR in Action (window N=4)

```
Sender sends: pkt0, pkt1, pkt2, pkt3
pkt2 is LOST

Receiver:
  rcv pkt0 → ACK0 ✓
  rcv pkt1 → ACK1 ✓
  rcv pkt3 → BUFFER it! ACK3 ✓ (not discard like GBN!)
  rcv pkt4 → BUFFER it! ACK4 ✓
  rcv pkt5 → BUFFER it! ACK5 ✓

Sender:
  rcv ACK0 → advance window, send pkt4
  rcv ACK1 → advance window, send pkt5
  rcv ACK3 → mark pkt3 as ACK'd (window still can't advance past pkt2)
  pkt2 TIMEOUT → retransmit ONLY pkt2

Receiver:
  rcv pkt2 → in-order! Deliver pkt2, pkt3, pkt4, pkt5 all at once! ACK2 ✓
```

Only pkt2 was retransmitted (instead of pkt2, pkt3, pkt4, pkt5 in GBN). Much more efficient!

### ⚠️ Common Confusion

> "In SR, the receiver sends individual ACKs. So if ACK3 is received before pkt2, why doesn't the sender advance its window past pkt3?"

The sender window advances only when the **oldest unACK'd packet** is acknowledged. If pkt2 is the oldest unACK'd and we get ACK3, ACK4, ACK5, the window base stays at pkt2 until ACK2 arrives. You can't skip a packet — you need all of them in order to advance the window.

> "What is the required relationship between window size and sequence number space in SR?"

For SR, the window size must be **≤ half the sequence number space**. If you have 2^k sequence numbers, the window size must be ≤ 2^(k-1). This is to prevent ambiguity — without this constraint, the receiver can't tell if a packet is a new one (from the next "wrap" of sequence numbers) or a retransmission of an old one.

For GBN, the window size must be < the sequence number space (≤ 2^k - 1).

---

## 17. GBN vs Selective Repeat — Side-by-Side Comparison

|Feature|Go-Back-N (GBN)|Selective Repeat (SR)|
|---|---|---|
|**On packet loss**|Retransmit packet n AND all higher seq# packets in window|Retransmit **only** packet n|
|**Receiver buffering**|No (typically discards out-of-order)|Yes (buffers out-of-order packets)|
|**ACK type**|Cumulative ACK — ACK(n) = "received all up to n"|Individual ACK — ACK(n) = "received specifically packet n"|
|**Timers**|One timer for the oldest unACK'd packet|Separate timer for each unACK'd packet|
|**Receiver complexity**|Simple (just remember rcv_base)|More complex (must manage buffer)|
|**Sender complexity**|Moderate|Higher (individual timers)|
|**Bandwidth efficiency on loss**|Lower (retransmits many packets)|Higher (retransmits only what's needed)|
|**Window size constraint**|≤ 2^k - 1|≤ 2^(k-1)|

### Which is Better?

Neither is universally better — it depends on the environment:

- **Low loss rate, large window:** SR is better (avoids unnecessary retransmissions)
- **Simple receiver, low memory:** GBN is better
- **High bandwidth-delay product:** SR (fewer wasted retransmissions)
- **TCP in practice:** Uses something closer to SR — it selectively retransmits, and modern TCP uses Selective Acknowledgements (SACK) as an extension

---

## 18. Practice Questions (Kurose & Ross Style)

---

### Section A: Transport Layer Basics

**Q1.** What is the difference between the network layer and the transport layer? Be precise.

> **Answer:**
> 
> The **network layer** provides logical communication between **hosts** (machines). It delivers a packet from one machine to another across the internet, using IP addresses. It doesn't care which process on the host should receive the data.
> 
> The **transport layer** provides logical communication between **application processes** running on different hosts. It extends the network layer's host-to-host delivery to process-to-process delivery by using port numbers to direct data to the correct application (socket).
> 
> In short: Network layer = host-to-host. Transport layer = process-to-process.

**Q2.** What does a transport-layer sender do to an application message? What does the receiver do?

> **Answer:**
> 
> **Sender:** Takes the application message, determines appropriate transport header field values (source/destination ports, sequence number, checksum, etc.), creates a segment by prepending the header to the message, and passes the segment down to the network layer (IP).
> 
> **Receiver:** Receives a segment from the network layer, checks the header values (e.g., verifies checksum), extracts the application-layer message from the segment, and demultiplexes the message up to the correct application process via the appropriate socket.

---

### Section B: Multiplexing and Demultiplexing

**Q3.** What is multiplexing and demultiplexing? Which happens at the sender and which at the receiver?

> **Answer:**
> 
> **Multiplexing (at sender):** The process of gathering data chunks from multiple application sockets, encapsulating each chunk with transport-layer header information (including source and destination port numbers), and passing the resulting segments to the network layer. Multiple data streams are combined ("multiplexed") into one.
> 
> **Demultiplexing (at receiver):** The process of using the header information in received segments to direct each segment to the correct socket (and thus the correct application process). The single incoming stream is split ("demultiplexed") to the appropriate destinations.

**Q4.** A host receives a UDP segment destined for port 80. Two different clients sent segments, both to this host at port 80, but from different source IPs and ports. Do both segments go to the same socket?

> **Answer:** Yes. UDP demultiplexes using only the **destination IP address and destination port number**. Since both segments have the same destination (this host, port 80), they are delivered to the same socket, regardless of where they came from. The application can distinguish between the two clients by reading the source IP and port from each received datagram.

**Q5.** Same scenario as Q4, but with TCP. Do both segments go to the same socket?

> **Answer:** No. TCP demultiplexes using a **4-tuple: (source IP, source port, destination IP, destination port)**. Even though both segments have the same destination IP and port 80, they have _different_ source IPs (or source ports). TCP uses all four values, so each segment goes to a **different socket**, representing a separate TCP connection. This is why a web server can maintain thousands of simultaneous connections — each connection gets its own socket.

---

### Section C: UDP

**Q6.** Suppose an application uses UDP. List and explain three advantages this gives the application compared to using TCP.

> **Answer:**
> 
> 1. **No connection establishment delay:** UDP sends data immediately without a handshake. TCP requires a 3-way handshake (1+ RTT) before any data flows. For short interactions like DNS queries, this overhead is unacceptable.
>     
> 2. **No connection state:** UDP maintains no connection state (no buffers, no sequence number trackers, no congestion control variables). A UDP server can serve many more clients simultaneously since it doesn't track state per client.
>     
> 3. **Finer application-level control over sending rate:** UDP has no congestion control. The application can send data at whatever rate it chooses. This is useful for real-time applications (video calls, games) where consistent timing is more important than reliability, and where the application itself may implement a smarter loss-recovery strategy than TCP.
>     

**Q7.** What is in a UDP segment header? What is the purpose of the "Length" field?

> **Answer:** A UDP segment header contains exactly 4 fields, each 16 bits (2 bytes), for a total of 8 bytes:
> 
> - **Source port:** Port of the sending process
> - **Destination port:** Port of the receiving process
> - **Length:** Total length of the UDP segment (header + data) in bytes. The minimum value is 8 (header only, no data). This field is redundant since the total length could be inferred from the IP datagram's length, but it's there for the receiver to know where the segment ends.
> - **Checksum:** Used to detect bit errors in the transmitted segment

---

### Section D: Reliable Data Transfer

**Q8.** What mechanisms are needed to implement reliable data transfer over a channel that can corrupt bits (but not lose packets)?

> **Answer:** To handle bit errors (rdt2.0 scenario), the following mechanisms are needed:
> 
> 1. **Error detection (checksum):** The sender computes and includes a checksum so the receiver can detect if bits were corrupted in transit.
>     
> 2. **Acknowledgements (ACKs):** The receiver explicitly tells the sender when a packet was received correctly.
>     
> 3. **Negative acknowledgements (NAKs):** The receiver explicitly tells the sender when a packet was received with errors, prompting retransmission.
>     
> 4. **Retransmission:** The sender retransmits the packet upon receiving a NAK.
>     
> 5. **Sequence numbers:** Needed to handle the case where ACK/NAK messages themselves get corrupted (the sender can't tell what the receiver said). With sequence numbers, the sender can safely retransmit and the receiver can detect duplicates.
>     

**Q9.** Why are sequence numbers needed in rdt2.0? Wouldn't checksums alone be enough?

> **Answer:** Checksums detect corruption in the _data packets_, but rdt2.0 also has the problem that **ACK/NAK packets can themselves get corrupted**. If the sender receives a garbled response, it doesn't know whether the receiver said "ACK" or "NAK." The safe choice is to retransmit, but then the receiver might get a _duplicate_ — it can't tell if this is a new packet or a retransmission.
> 
> **Sequence numbers solve this:** The sender numbers each packet. If the receiver gets the same sequence number twice, it knows it's a duplicate — it re-sends its ACK but doesn't deliver the duplicate data to the application.

**Q10.** What does rdt3.0 add over rdt2.0, and why?

> **Answer:** rdt3.0 adds a **timeout mechanism** (countdown timer + retransmission on timeout). rdt2.0 assumed the channel could corrupt packets but not lose them entirely. In reality, packets (data or ACKs) can be lost completely.
> 
> If a packet is lost, the receiver never gets it, so it never sends an ACK. The sender would wait forever. rdt3.0 solves this: the sender sets a timer when it sends a packet. If no ACK arrives within a "reasonable" time, the sender assumes the packet (or its ACK) was lost and retransmits.
> 
> Sequence numbers (already from rdt2.0) handle the case where the original packet was just delayed (not lost) — the retransmission creates a duplicate, which is detected and discarded.

**Q11.** In the rdt3.0 "premature timeout" scenario, the sender retransmits a packet even though the original arrived fine. Walk through what happens. Is correctness maintained?

> **Answer:** Yes, correctness is maintained.
> 
> Timeline:
> 
> 1. Sender sends pkt1, starts timer
> 2. Receiver gets pkt1, sends ACK1
> 3. ACK1 is delayed (takes a long time in the network)
> 4. Timer fires before ACK1 arrives → sender retransmits pkt1
> 5. Receiver gets duplicate pkt1 → recognizes it's a duplicate (same seq# 1 as already received) → discards the data, but sends ACK1 again
> 6. Sender gets the first (delayed) ACK1 or the second ACK1 → either way, advances to pkt0
> 7. Any extra ACK1s that arrive later are ignored
> 
> Correctness is maintained because: (a) the application never gets duplicate data delivered, (b) the sender eventually gets ACK'd and moves on.

---

### Section E: Pipelining

**Q12.** What is stop-and-wait utilization, and why is it so low? Give the formula.

> **Answer:** Sender utilization in stop-and-wait:
> 
> $$U_{sender} = \frac{L/R}{RTT + L/R}$$
> 
> It is low because $RTT >> L/R$ in most real-world scenarios. A 1 Gbps link can transmit a 1 KB packet in about 8 microseconds, but a cross-continent RTT is around 30 ms. So the sender transmits for 8 µs and then sits idle for ~30 ms waiting for the ACK. The link is being used only 0.027% of the time — an extreme waste.

**Q13.** How does pipelining improve utilization? What does it require?

> **Answer:** Pipelining allows the sender to have multiple packets "in flight" simultaneously — sent but not yet acknowledged. With a pipeline window of N packets:
> 
> $$U_{sender} = \frac{N \times L/R}{RTT + L/R}$$
> 
> With N=3, utilization triples. With large N, utilization approaches 100%.
> 
> **Requirements:**
> 
> - Larger sequence number range (to uniquely identify each in-flight packet)
> - Sender buffers unACK'd packets (for possible retransmission)
> - Receiver may need to buffer out-of-order packets (SR), or can discard them (GBN)

---

### Section F: Go-Back-N vs Selective Repeat

**Q14.** Compare Go-Back-N and Selective Repeat in terms of: (a) what gets retransmitted on a loss, (b) receiver buffering, (c) ACK type, (d) window size constraint.

> **Answer:**
> 
> **(a) Retransmission on loss:**
> 
> - GBN: Retransmits packet n AND all higher sequence number packets currently in the sender window.
> - SR: Retransmits ONLY the specific lost/timed-out packet.
> 
> **(b) Receiver buffering:**
> 
> - GBN: Receiver typically discards out-of-order packets (no buffer needed), only remembers the expected next in-order sequence number.
> - SR: Receiver buffers out-of-order packets until the missing one arrives, then delivers them all in sequence.
> 
> **(c) ACK type:**
> 
> - GBN: Cumulative ACKs — ACK(n) means "I have correctly received all packets with sequence number ≤ n."
> - SR: Individual ACKs — ACK(n) means "I have correctly received packet n specifically."
> 
> **(d) Window size constraint:**
> 
> - GBN: Window size ≤ 2^k - 1 (where k = number of bits in sequence number field)
> - SR: Window size ≤ 2^(k-1) — must be at most half the sequence number space, to prevent ambiguity between new packets and retransmissions.

**Q15.** With a 3-bit sequence number (values 0–7) and Go-Back-N, what is the maximum window size? What about Selective Repeat?

> **Answer:**
> 
> With k=3 bits, sequence numbers range from 0 to 7 (2^3 = 8 values).
> 
> - **GBN maximum window size:** 2^3 - 1 = **7**
> - **SR maximum window size:** 2^3 / 2 = 2^(3-1) = **4**
> 
> SR has the stricter constraint because of its receiver window. The receiver window also slides, and if the window size were larger than half the sequence number space, a newly sent packet could have the same sequence number as one the receiver is still waiting to have retransmitted — causing ambiguity.

**Q16.** In Go-Back-N, suppose the sender window is N=4 and packets 0, 1, 2, 3 are sent. Packet 1 is lost. Describe what happens.

> **Answer:**
> 
> 1. Sender sends pkt0, pkt1, pkt2, pkt3 (window full, waiting for ACKs)
> 2. Receiver gets pkt0 → sends ACK0
> 3. pkt1 is lost in the network
> 4. Receiver gets pkt2 → out of order! Discards pkt2, re-sends ACK0 (last correctly received in-order)
> 5. Receiver gets pkt3 → out of order! Discards pkt3, re-sends ACK0
> 6. Sender receives ACK0 → advances window, sends pkt4
> 7. Receiver gets pkt4 → out of order! Discards pkt4, re-sends ACK0
> 8. Sender receives ACK0 (duplicate) → ignores
> 9. pkt1 timeout fires → sender retransmits pkt1, pkt2, pkt3, pkt4 (all in window from pkt1 onward)
> 10. Receiver gets pkt1 → in order! ACK1; gets pkt2 → ACK2; gets pkt3 → ACK3; gets pkt4 → ACK4
> 
> Note: pkt2, pkt3, pkt4 were received correctly the first time but had to be retransmitted — GBN's inefficiency.

---

_End of Notes_

---

> **Created for Obsidian** | Computer Networking: A Top-Down Approach — Kurose & Ross | Chapter 3: Transport Layer (Part I)  
> **Covers:** Transport services, Mux/Demux, UDP, Reliable Data Transfer (rdt1.0–3.0), Pipelining, Go-Back-N, Selective Repeat