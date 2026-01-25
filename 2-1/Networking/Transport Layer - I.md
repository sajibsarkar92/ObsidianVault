![[Pasted image 20260121011752.png]]

# Transport Layer: The Big Picture (Slide 2)

## The "Logical" Connection
The Transport Layer provides **logical communication** between processes. 

> [!TIP] What does "Logical" mean?
> It means from the App's perspective, the two hosts are directly connected. It ignores the physical mess of routers, switches, and wires in between.

### Where does it live?
- **Only at the Edges:** Transport protocols (TCP/UDP) run *only* on end systems (Hosts).
- **The Core is Blind:** Routers in the "Network Core" do not look at Transport headers; they only look at Network (IP) headers.

### The Two-Way Street
1. **Sender Side:** - Grabs the huge message from the App.
   - Chops it into **Segments**.
   - Hands it down to the Network Layer (IP).
2. **Receiver Side:**
   - Receives the scattered segments from the Network Layer.
   - Reassembles them into the original message.
   - Hands it up to the specific App.

---

## ⚡ The Quick Summary (For Exam Brain)
- **Network Layer** = Path between **Devices** (IP addresses).
- **Transport Layer** = Path between **Apps** (Port numbers).
- **Unit of Data:** In this layer, we call data a **Segment**.


![[Pasted image 20260121012640.png]]# Transport vs. Network Layer (Slide 3)

## The Core Difference
| Feature | Network Layer | Transport Layer |
| :--- | :--- | :--- |
| **Entity** | Host (Computer/Server) | Process (App/Socket) |
| **Analogy** | The Postal Service | Ann & Bill (The internal distributors) |
| **Logic** | Host-to-Host communication | Process-to-Process communication |

### Relationship: The "Enhancer" Role
The Transport Layer **relies on** the Network Layer, but it also **enhances** it.
* **The Network Layer is like a "Best Effort" mailman:** He might lose a letter, or deliver them out of order.
* **The Transport Layer (TCP) is like a supervisor:** It notices if a letter is missing and asks for a resend. 

> [!IMPORTANT] The Constraint
> Transport protocols are constrained by the Network Layer. If the network (the "Postal Service") has a delay of 1 hour, the Transport layer cannot magically make it 10 minutes. 

---

## 💡 Engineering Intuition
Even if the Network Layer is 100% reliable, we **still** need the Transport Layer to do **Multiplexing** (sorting which data goes to which app). Without it, your computer wouldn't know if an incoming packet belongs to your YouTube video or your system update.




![[Pasted image 20260121012950.png]]![[Pasted image 20260121013006.png]]


# Transport Layer Actions (Slides 4-5)

## 1. The Sender Side (Encapsulation)
When you send data (e.g., hitting "Send" on an email):
1. **Receive:** Gets the raw message from the Application layer.
2. **Segment:** Chops the message into smaller pieces.
3. **Header Creation:** Determines the values for header fields (Source/Dest Ports, Sequence numbers, etc.).
4. **Wrap:** Creates the **Segment** by attaching the header to the data.
5. **Pass:** Hands the segment down to the Network Layer (IP).



## 2. The Receiver Side (Decapsulation)
When data arrives at your computer:
1. **Receive:** Grabs the segment from the Network Layer (IP).
2. **Verify:** Checks the header values (Is the data corrupted? Is it for the right App?).
3. **Extract:** Rips off the transport header.
4. **Reassemble:** Puts the pieces back in the correct order.
5. **Deliver:** Slaps the original message onto the **Socket** for the specific App.

---

## 🛠 Engineering Terminology
- **Message:** Data at the Application Layer.
- **Segment:** Data at the Transport Layer (Message + Transport Header).
- **Datagram/Packet:** Data at the Network Layer (Segment + IP Header).

> [!QUOTE] Ed's Rule of Thumb
> If you see the word **"Segment"**, you are 100% in the Transport Layer.


![[Pasted image 20260121013216.png]]


# Internet Transport Protocols: TCP vs. UDP

The Internet gives us two distinct flavors of transport. Choosing between them is the most fundamental architectural decision a Software Engineer makes when building a networked app.



---

## 1. UDP: User Datagram Protocol (The "No-Frills" Speedster)
**Philosophy:** Get the data there as fast as possible. If it breaks, it breaks.

- **Connectionless:** No "Handshaking." The sender just starts blasting data.
- **Unreliable Delivery:** - Packets can arrive **out of order**.
	- Packets can be **lost** (dropped) without the sender ever knowing.
- **No Flow/Congestion Control:** UDP will pump data as fast as the app allows, even if the network is "clogged" or the receiver is overwhelmed.
- **The "Lightweight" Advantage:** Since there are no checks, the header is tiny, and there is zero delay in starting the transmission.

**Best for:** - Streaming Video/Audio (IPTV, Zoom) - *Speed > Perfect Quality*
- DNS (Domain Name System) - *Needs to be lightning fast*
- Online Gaming - *Lag is a bigger enemy than a dropped packet*

---

## 2. TCP: Transmission Control Protocol (The "Perfectionist")
**Philosophy:** Ensure every single bit arrives exactly as sent, no matter what it takes.

- **Connection-Oriented:** Requires a **3-Way Handshake** (SYN, SYN-ACK, ACK) before any data moves.
- **Reliable Data Transfer (RDT):** - **Acknowledgements (ACKs):** Receiver tells the sender "I got packet #4."
	- **Retransmissions:** If an ACK doesn't arrive, the sender sends the data again.
- **In-Order Delivery:** If packet #3 arrives after packet #4, TCP holds #4 in a buffer and waits for #3 so the App gets them in the right sequence.
- **Flow Control:** Sender won't "overwhelm" the receiver (checks the receiver's buffer space).
- **Congestion Control:** Sender slows down if the *network* is congested (prevents "Internet collapse").

**Best for:**
- HTTP/HTTPS (Web Browsing) - *You don't want a "broken" image or missing text.*
- FTP (File Transfer) - *A single flipped bit ruins a .zip file.*
- SMTP (Email) - *Reliability is non-negotiable.*

---

## ⚡ Summary Table for Exams

| Feature | UDP | TCP |
| :--- | :--- | :--- |
| **Connection** | Connectionless | Connection-oriented |
| **Reliability** | Unreliable (Best effort) | Guaranteed |
| **Order** | Can be out of order | Strictly ordered |
| **Speed** | Very High (Low overhead) | Slower (High overhead) |
| **Header Size** | 8 Bytes | 20 Bytes (minimum) |
| **Data Unit** | Datagram | Segment |

> [!TIP] Ed's Pro-Tip
> **What UDP and TCP BOTH provide:** Integrity Check (Checksums). Even "unreliable" UDP will usually discard a packet if the bits are corrupted. It just won't ask for a resend.


![[Pasted image 20260121013432.png]]
# Why Multiplexing & Demultiplexing? (Slide 6-7)

## The Problem: The "One Door" Limitation
The Network Layer (IP) only delivers data to the **Host** (the machine). It has no idea that you have multiple applications running. 

**Multiplexing and Demultiplexing** are the mechanisms that extend host-to-host delivery to **process-to-process** delivery.



---

## 1. Demultiplexing (The Receiver's Job)
**Definition:** Delivering received segments to the correct socket.

**How it works:**
1. The host receives an IP datagram.
2. Each datagram has a **Source IP** and **Destination IP**.
3. Each datagram carries ONE transport-layer **segment**.
4. The segment has a **Destination Port Number**.
5. The host uses that port number to kick the data into the right **Socket**.

---

## 2. Multiplexing (The Sender's Job)
**Definition:** Gathering data from multiple sockets and wrapping them into segments to send out.

**How it works:**
1. The Transport layer sits at the "exit" of the application sockets.
2. It takes data from Socket A (Chrome) and Socket B (Spotify).
3. It adds **Headers** (Source/Dest Ports) to create segments.
4. It passes these segments to the Network layer.

---

## 💡 Engineering Intuition: The Socket
Think of a **Socket** as the "Internal Door" of an app. 
- **IP Address** = The Building Address.
- **Port Number** = The Room Number.
- **Socket** = The actual door handle you turn to hand over the data.

![[Pasted image 20260121013442.png]]
# The Multiplexing Process: Step-by-Step Encapsulation

## The Sequential Flow (Sender Side)

1. **API Call:** Application layer passes a "Message" to the Transport Layer via a **Socket**.
2. **Chunking:** Data is divided into chunks (if necessary).
3. **Header Attachment:** The Transport Layer adds a header to create a **Segment**.
4. **Passing Down:** The Segment is passed to the Network Layer (IP).



---

## 📦 Anatomy of the Transport Header
The header is the "Metadata" that makes multiplexing possible. While TCP and UDP headers differ, they **must** both contain these core fields:

| Field | Size | Function |
| :--- | :--- | :--- |
| **Source Port #** | 16 bits | The "return address" (Which app on MY computer sent this?) |
| **Dest Port #** | 16 bits | The "destination door" (Which app on the RECEIVER should get this?) |
| **Length/Other** | Varies | Size of the segment. |
| **Checksum** | 16 bits | Error detection (Was the data mangled in transit?) |

### 🔍 Detailed Look: The Port Numbers
- **Range:** 0 to 65,535.
- **Well-known Ports:** 0-1023 (Reserved for big stuff like HTTP=80, FTP=21).
- **Ephemeral Ports:** 1024-65535 (Temporary ports used by your browser tabs).

---

## 💡 The Key Insight
**Multiplexing** is simply the act of taking multiple data streams from different sockets, slapping unique **Port Headers** on them, and merging them into one stream of segments heading for the Network Layer.

![[Pasted image 20260121013456.png]]![[Pasted image 20260121013529.png]]
# How Servers Differentiate Users: The 4-Tuple Logic

## The Problem
Two clients (Client A and Client B) are both using Firefox to access the same Web Server. 
- Both use **HTTP (Port 80)**.
- Both use the **same Browser (Firefox)**.
- They send requests at the **exact same millisecond**.

How does the server not send Client A's private data to Client B?
## The Solution: TCP Connection-Oriented Demux
Unlike UDP (which is loose), **TCP** is strict. It identifies every single connection using a **4-Tuple**. For a segment to belong to a specific "conversation" (socket), all four of these must match:

1. **Source IP:** (Where is the computer?)
2. **Source Port:** (Which specific tab/process on that computer?)
3. **Destination IP:** (Which server are they hitting?)
4. **Destination Port:** (Which service on that server, e.g., 80 for Web?)



---

## 🛠 Case Study: The Firefox Example

| Data Field | Client A (User 1) | Client B (User 2) |
| :--- | :--- | :--- |
| **Source IP** | 103.11.1.5 | 103.11.1.9 (Different!) |
| **Source Port** | 52341 | 61222 (Different!) |
| **Dest IP** | 142.250.1.1 | 142.250.1.1 |
| **Dest Port** | 80 | 80 |

### Why they are different:
1. **Different IP:** If they are on different networks (e.g., one on Home WiFi, one on Mobile Data), their **Source IP** is different.
2. **Different Source Port:** Even if they are on the *same* WiFi (same Public IP), their Operating Systems will pick a **random ephemeral port** (1024–65535) for the Firefox tab. It is statistically impossible for them to be identical.

---

## 🏗 The Server-Side Architecture
When the server sees a new 4-tuple, it doesn't just shove the data into a general "Port 80" pile. Instead:
1. The **Welcome Socket** (Listening on Port 80) hears the request.
2. The Server OS creates a **New Dedicated Socket** specifically for that 4-tuple.
3. All future data from Client A goes to **Socket #1**.
4. All future data from Client B goes to **Socket #2**.

[[Pasted   image 20260121013548.png]] 
![[Pasted image 20260121013601.png]]
# The Demultiplexing Process: Step-by-Step Decapsulation

## The Sequential Flow (Receiver Side)

1. **IP Handover:** Network layer passes the segment to Transport Layer.
2. **Identity Check:** - **UDP:** Checks Destination Port only.
   - **TCP:** Checks the 4-tuple (Src IP/Port, Dest IP/Port).
3. **Validation:** Verifies the **Checksum** to ensure no bits were flipped.
4. **Decapsulation:** Removes the Transport Header.
5. **Socket Mapping:** Locates the specific **Socket ID** in the Operating System.
6. **Delivery:** Delivers raw data to the Application Layer via the socket

---

## 💡 Engineering Summary
| Level | Data Unit | Key Identifier used for Delivery |
| :--- | :--- | :--- |
| **Network** | Datagram | Destination IP Address |
| **Transport** | Segment | Destination Port (UDP) / 4-Tuple (TCP) |
| **Application** | Message | Socket File Descriptor |

> [!DANGER] Critical Exam Point
> If a segment arrives with a **Destination Port** that no application is currently "listening" on, the Transport Layer cannot deliver it. It will drop the segment and (usually) send an ICMP "Destination Unreachable" message back to the sender.


![[Pasted image 20260121013617.png]]

# Encapsulation & Demux: TCP vs. UDP

## 1. Encapsulation (Header Differences)
| Feature | UDP Header (8B) | TCP Header (20B+) |
| :--- | :--- | :--- |
| **Simplicity** | Minimalist. Just Ports & Checksum. | Complex. Includes state tracking. |
| **Reliability** | None. Just "best effort." | Sequence & Ack numbers for RDT. |
| **Traffic Control** | Blasts data at app speed. | Uses Window size to prevent overflow. |

---

## 2. Demultiplexing Mechanics

### UDP (Connectionless)
- **Identifier:** Uses **2-Tuple** `(Dest IP, Dest Port)`.
- **Behavior:** Segments from *different* sources but with the *same* Dest Port are directed to the **same socket**.
- **Use Case:** DNS, Video Streaming.

### TCP (Connection-Oriented)
- **Identifier:** Uses **4-Tuple** `(Src IP, Src Port, Dest IP, Dest Port)`.
- **Behavior:** Every unique 4-tuple gets its own **Dedicated Socket**. Even if 10 users hit Port 80, the server maintains 10 separate "conversations."
- **Use Case:** HTTP (Web), SSH, Email.

---

## 💡 The "Aha!" Insight for Exams
- **UDP Demux** is based on the **Destination** only.
- **TCP Demux** is based on the **Relationship** (Source + Destination).

> [!WARNING] Coding Tip
> In a UDP server, your code has to look at the `from_address` manually to know who sent the data. In a TCP server, the `accept()` function gives you a unique socket that is already "locked" to that specific user.

# Demultiplexing: TCP vs. UDP Logic

## 1. UDP: Connectionless Demux
UDP uses a **2-Tuple** to identify the destination socket.

- **The Identifier:** `(Dest IP, Dest Port)`
- **Key Behavior:** Segments with different Source IPs or Source Ports, but the **same Destination Port**, will be directed to the **same socket**.

> [!EXAMPLE]
> If User A (Port 500) and User B (Port 600) both send a UDP packet to Server X (Port 80), the server's code receives both messages through one single `serverSocket`.

---

## 2. TCP: Connection-Oriented Demux
TCP uses a **4-Tuple** to identify the destination socket.

- **The Identifier:** `(Source IP, Source Port, Dest IP, Dest Port)`
- **Key Behavior:** Each unique combination of these four values gets its own **dedicated socket**.
- **Welcome Socket:** Servers have one "Listening" socket (e.g., Port 80) that acts like a receptionist. When a new user arrives, it "spawns" a new, specific socket for that unique 4-tuple.



---

## ⚡ Summary Table for Exam

| Protocol | Demux Identifier | Result of same Dest Port |
| :--- | :--- | :--- |
| **UDP** | 2-Tuple (Dest IP/Port) | Multiple users share 1 socket. |
| **TCP** | 4-Tuple (Src & Dest IP/Port) | Every user gets a private socket. |

### 💡 Software Engineering Key Insight
In **TCP**, the 4-tuple allows the server to track the "state" of each user individually (e.g., what page they are on). In **UDP**, since everyone shares a socket, the programmer must manually write code to check the sender's IP to tell them apart.


![[tcp-header-format.jpeg]]


![[images.png]]



![[Pasted image 20260121020741.png]]

# UDP: User Datagram Protocol (The Connectionless Speedster)

## 1. The "Best Effort" Philosophy
UDP is often called a "Best Effort" service. In plain English, this means the protocol tries its best to deliver data, but it makes **zero promises**.

### ⚠️ Possible Failure Modes:
- **Loss:** Segments may simply vanish in the network core.
- **Out-of-Order:** Segment #2 might arrive before Segment #1.
- **No Recovery:** If data is lost, UDP does not ask for a retransmission.

---

## 2. Why use UDP? (The Advantages)
If UDP is unreliable, why do we use it for some of the world's most important apps (DNS, Gaming, Streaming)?

| Advantage | Technical Reason | Benefit |
| :--- | :--- | :--- |
| **No Latency** | No Connection Establishment. | No RTT delay for "handshaking." Data flows instantly. |
| **Simplicity** | No Connection State. | No need to track Sequence #s, ACKs, or Buffer sizes. |
| **Small Overhead** | Small Header Size (8 bytes). | More room for actual data; less bandwidth wasted on headers. |
| **No Restraint** | No Congestion Control. | UDP can "blast away" at any speed. It doesn't slow down just because the network is crowded. |

[Image of UDP vs TCP performance comparison]

---

## 3. Key Characteristics (Slide 16-18)

### Connectionless Nature
- There is **no handshaking** between the sender and receiver.
- Each UDP segment is handled **independently** of the others. The receiver doesn't even know if the segment it just got is part of the same file as the previous one.

### The "Wild West" Protocol
- UDP can function even when the network is severely congested. While TCP will slow down to be a "good citizen," UDP will continue to pump data as fast as the application allows.

> [!INFO] One Key Insight
> UDP is the choice for **Real-Time Applications**. In a Zoom call, if a packet of your voice is lost, you don't want the app to "pause" everything to re-fetch it—you just want the *next* packet of audio immediately.

---

## 🛠 Exam Focus: The "No-State" Rule
In exams, you may be asked: *"Why is UDP more scalable for a server than TCP?"*
**Answer:** Because the server doesn't have to store **Connection State**. A TCP server must remember the Sequence # and Window Size for *every* client. A UDP server just receives a packet, processes it, and forgets it instantly.



# **==The UDP Ordering Problem: "No Sequence, No Order"==**

## The Reality of UDP
UDP headers (8 bytes) **do not** contain Sequence Numbers or Acknowledgement Numbers. 

### What happens when packets are jumbled?
1. **The Transport Layer:** Does nothing. It blindly passes the data up to the Application Layer as soon as it arrives.
2. **The Result:** If "World" arrives before "Hello", the app sees "World Hello".

---

## 🛠 How Engineers Fix This (Application Layer Reliability)
If an application needs the speed of UDP but the order of TCP (e.g., **HTTP/3** or **QUIC**), the logic must be built into the **Application Layer**.

### The Sequence:
1. **Sender App:** Wraps the message with a custom "App-level Sequence Number" before handing it to the UDP socket.
2. **Network:** Routes the UDP segments; they might take different paths and arrive out of order.
3. **Receiver App:** - Collects segments.
	- Reads the "App-level Sequence Number".
	- **Buffers** the data until the missing pieces arrive.
	- Reassembles the message manually.

| Logic Level | TCP | UDP |
| :--- | :--- | :--- |
| **Transport Layer** | Handles ordering automatically. | Ignores ordering entirely. |
| **Application Layer** | Just receives clean data. | Must handle reordering/loss if needed. |

> [!DANGER] Exam Fact
> If a DU exam asks: *"Does UDP provide in-order delivery?"*
> **Answer:** No. UDP is a connectionless service that handles each segment independently. It has no mechanism for reassembly or reordering.



![[Pasted image 20260121022701.png]]
# Abstraction
Here the app sees the transport as a reliable medium of transfer when it isn't.

So we have to implement protocols that ensures the reliability in the unreliable medium.





![[Pasted image 20260121022712.png]]


![[Pasted image 20260121022724.png]]![[Pasted image 20260121022742.png]]

# Principles of Reliable Data Transfer (rdt)

## 1. The Core Challenge
The goal of **rdt** is to implement a reliable data transfer protocol over a channel that is inherently unreliable (IP).

| Layer | Function |
| :--- | :--- |
| **rdt_send()** | Called by App to send data to the "reliable" pipe. |
| **udt_send()** | The actual unreliable transfer over the network. |
| **rdt_rcv()** | Called when a packet arrives at the receiver side. |
| **deliver_data()** | Hands the clean, ordered data to the App. |

---
![[Pasted image 20260121024843.png]]
# rdt 1.0: Reliable Transfer over a Reliable Channel

## 🎯 Context
The underlying channel is 100% reliable. 
- No bit errors (corruption).
- No packet loss.

## ⚙️ How it Works
Since the channel is perfect, the sender just sends and the receiver just receives. Neither side needs to provide any feedback.



### Sender FSM:
1. **State:** Wait for call from above.
2. **Action:** `rdt_send(data)` $\rightarrow$ `make_pkt(data)` $\rightarrow$ `udt_send(packet)`.

### Receiver FSM:
1. **State:** Wait for call from below.
2. **Action:** `rdt_rcv(packet)` $\rightarrow$ `extract(packet, data)` $\rightarrow$ `deliver_data(data)`.

> [!NOTE] Ed's Insight
> In rdt 1.0, the receiver can never receive data faster than the sender sends it, and no data is ever "wrong." It is the simplest possible state machine.

![[Pasted image 20260121024926.png]]

![[Pasted image 20260121024902.png]]
# rdt 2.0: Channel with Bit Errors

## 🎯 Context
Packets can be **corrupted** (bits flipped) due to noise in the channel, but packets are **not lost**.

## 🛠 New Tools: ARQ (Automatic Repeat Request)
To handle corruption, we introduce "Feedback":
1. **Error Detection:** Checksum added to the header.
2. **ACK (Positive Acknowledgement):** Receiver tells sender "Got it!"
3. **NAK (Negative Acknowledgement):** Receiver tells sender "Garbled, resend!"



## 🔄 The Protocol Flow
1. **Sender** sends `pkt1` and enters **Wait for ACK/NAK** state.
2. **Sender** is blocked (cannot send new data) until it hears back.
3. If **NAK**, sender retransmits `pkt1`.
4. If **ACK**, sender moves to the next packet.

> [!DANGER] The "Fatal Flaw"
> What if the **ACK or NAK itself is corrupted**? 
> If the sender receives a garbled ACK/NAK, it doesn't know what the receiver said. 
> - If it resends the packet: The receiver might get a **Duplicate**.
> - This leads us to **rdt 2.1** (Sequence Numbers).


![[Pasted image 20260121024950.png]]
![[Pasted image 20260121024959.png]]
![[Pasted image 20260121025014.png]]


![[Pasted image 20260121053150.png]]
# rdt 3.0: Channel with Errors AND Loss

## 🎯 Context
This is the realistic model. Packets can be **corrupted** OR completely **lost** (dropped by a router).

## 🛠 The New Tool: The Countdown Timer
In rdt 2.x, if a packet is lost, the sender waits for an ACK forever. In rdt 3.0, we introduce a **Timer**.



## ⚙️ The Logic
1. **Sender** starts a timer when sending a packet.
2. **Success:** ACK arrives before the timer expires $\rightarrow$ Timer stops, move to next packet.
3. **Failure (Timeout):** If the timer runs out, the sender assumes the packet (or its ACK) was lost.
4. **Action:** Sender **Retransmits** the packet.

### Handling Duplicates
If the packet wasn't lost, just slow (delayed ACK), the receiver might get a duplicate. rdt 3.0 uses **Sequence Numbers (0 and 1)** (inherited from rdt 2.1) to discard these duplicates.

> [!KEY_INSIGHT]
> rdt 3.0 is a **correct** protocol, but it is **"Stop-and-Wait."** This means it is incredibly slow because the sender spends most of its time waiting for ACKs rather than sending data.

| Protocol | Fixes |
| :--- | :--- |
| **rdt 1.0** | Baseline (Perfect) |
| **rdt 2.0** | Bit Errors (ACK/NAK) |
| **rdt 2.1/2.2** | Corrupted ACKs (Seq #s) |
| **rdt 3.0** | Packet Loss (Timers) |

![[Pasted image 20260121053237.png]]

# The Inefficiency of Stop-and-Wait (rdt 3.0)

## 📉 The Performance Bottleneck
The core problem with **rdt 3.0** is that it is a "Sequential" protocol. The sender is blocked until an acknowledgment (ACK) travels the full distance of the network and back.

### Why it fails in High-Speed Networks:
1. **Propagation Delay vs. Transmission Delay:** In modern fiber optics, the time it takes to "push" bits into the wire ($L/R$) is microscopic compared to the time it takes for those bits to travel across the ocean ($RTT$).
2. **The "Empty Pipe" Problem:** Under Stop-and-Wait, the network "pipe" is mostly empty. We are limited by the speed of light (latency), not the width of the wire (bandwidth).



## 🛠 Utilization Calculation ($U_{sender}$)
Utilization is the fraction of time the sender is actually sending.

$$U_{sender} = \frac{\text{Time spent transmitting}}{\text{Total cycle time}} = \frac{L/R}{RTT + L/R}$$

### Key Takeaway for Exams:
As the **Bandwidth-Delay Product** (Bandwidth $\times$ RTT) increases, the efficiency of Stop-and-Wait decreases.
- **Low RTT (Local Network):** Stop-and-Wait is "okay."
- **High RTT (Satellites/Global Fiber):** Stop-and-Wait is "broken."

---

## 🚀 The Solution: Pipelining
To fix this, we allow the sender to send multiple packets without waiting for an ACK.
- If we send **$N$ packets** at once, our utilization increases by a factor of $N$.
- **New Formula:** $U_{sender} = \frac{N \times (L/R)}{RTT + (L/R)}$.

> [!KEY_INSIGHT]
> Pipelining fills the "pipe." Instead of one brick per trip, we send a convoy of $N$ trucks. This is the logic used by **Go-Back-N** and **Selective Repeat**.


![[Pasted image 20260121053612.png]]

# Pipelined Protocols: GBN vs. Selective Repeat

## 🚀 The Core Concept
Pipelining increases **Utilization** by allowing the sender to transmit multiple packets before receiving the first ACK.

### Comparison Table
| Feature | Go-Back-N (GBN) | Selective Repeat (SR) |
| :--- | :--- | :--- |
| **Window Size** | Up to $N$ packets | Up to $N$ packets |
| **ACK Type** | **Cumulative** (ACK $n$ covers all up to $n$) | **Individual** (ACK $n$ covers only $n$) |
| **Retransmission** | All un-ACKed packets in window | Only the specific lost packet |
| **Receiver Buffer** | None (Discards out-of-order) | Buffers out-of-order packets |
| **Complexity** | Simple (Low memory) | Complex (More timers/memory) |

---

## 🛠 Utilization Math ($U_{sender}$)
By sending $N$ packets instead of 1, we multiply our efficiency by $N$.

$$U_{sender} = \frac{N \times (L/R)}{RTT + (L/R)}$$

> [!KEY_INSIGHT]
> **Why use GBN?** If the network rarely loses packets, GBN is better because it’s simple and doesn't require a receiver buffer.
> **Why use SR?** If the network is "lossy," GBN wastes too much bandwidth re-sending data that already arrived. SR is the "smarter" but "heavier" solution.

---
![[Pasted image 20260121054034.png]]![[Pasted image 20260121054046.png]]# Go-Back-N (GBN) Protocol

## 🎯 Core Concept
A "Pipelined" protocol that allows sending up to $N$ packets before an ACK. It uses **Cumulative ACKs** and a **single timer**.

## 📤 Sender Logic
- **Window Size (N):** Max number of un-ACKed packets allowed.
- **Cumulative ACK:** Receiving `ACK(n)` confirms all packets up to $n$.
- **Timeout:** If the oldest packet times out, the sender retransmits **ALL** unacknowledged packets in the window.

## 📥 Receiver Logic
- **ExpectedSeqNum:** Only accepts the *exact* next packet in order.
- **No Buffer:** Discards any out-of-order packets.
- **ACK Strategy:** Always sends an ACK for the last **correctly received in-order** packet.



---

## ⚡ Pros & Cons
| Pros | Cons |
| :--- | :--- |
| Much faster than Stop-and-Wait. | A single lost packet triggers many retransmissions. |
| Receiver is very simple (no memory needed). | Wastes bandwidth on "lossy" networks. |

> [!TIP] Ed's Exam Tip
> If an exam question asks: "If packet 3 is lost in a GBN window of 10, how many packets are resent?"
> **Answer:** Packet 3 and everything sent after it (4, 5, 6...).

## ⚡ Visualizing the Window
The "Window" slides forward as ACKs come in.
- **Left Edge:** Oldest un-ACKed packet.
- **Right Edge:** The next sequence number we can use.

![[Pasted image 20260121053735.png]]
![[Pasted image 20260121054137.png]]
# Selective Repeat (SR): Efficient Pipelining

## 🎯 The Philosophy
"Retransmit ONLY what is lost." SR avoids the waste of Go-Back-N by acknowledging packets individually and buffering out-of-order data.

## 🛠 Key Mechanisms

### 1. Sender Side
- **Individual Timers:** Every packet in the window has its own countdown.
- **Selective Retransmission:** When a timer for packet $n$ expires, only $n$ is resent.
- **Window Management:** The window only slides forward when the **oldest** unacknowledged packet is finally ACKed.

### 2. Receiver Side
- **Individual ACKs:** Sends `ACK n` for every valid packet $n$.
- **Buffering:** If a packet is out-of-order, store it. Do NOT discard it (like GBN would).
- **Delivery:** Only delivers to the App Layer when it has a "continuous" sequence of packets.



---

## ⚖️ Trade-offs: GBN vs. Selective Repeat

| Feature | Go-Back-N (GBN) | Selective Repeat (SR) |
| :--- | :--- | :--- |
| **Complexity** | Simple. 1 Timer. | High. Many Timers. |
| **Receiver Memory** | None. Discards out-of-order. | High. Must Buffer. |
| **Bandwidth Use** | Wasted on retransmissions. | Optimized (Minimal). |

> [!KEY_INSIGHT]
> **Selective Repeat** is the "Gentleman's Protocol." It takes more effort (memory and timers) to maintain, but it is far more polite to the network by not clogging it with duplicate data.


