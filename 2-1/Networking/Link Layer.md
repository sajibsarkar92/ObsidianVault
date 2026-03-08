
![[Chapter_6_v9.0-1.webp]]
![[Chapter_6_v9.0-2.webp]]
![[Chapter_6_v9.0-3.webp]]
![[Chapter_6_v9.0-4.webp]]

- **Nodes** : Hosts and routers, i.e. any device
- **Links** : Any communication medium between the hosts. Wired or wireless.
- Link layer basically transmits one datagram from one node to another over a link. Hop to hop.
- Layer-2 packets are called frames, which are just datagram with another header.

![[Chapter_6_v9.0-5.webp]]



![[Chapter_6_v9.0-6.webp]]
![[Chapter_6_v9.0-7.webp]]
* *Framing* : This encapsulation actually contains a MAC address header. There are source and destination mac addresses given here, which are updated at every hop.
* 
* *Channel access* : In shared mediums  like WiFi, this protocol controls who gets to send and who is to listen, preventing chaos.
* 
* *Reliability* : This is not guaranteed across whole transfer. Even when a perfect transfer is done across the link, the data can still be dropped in the node, in case of buffer overflow. Therefore the end to end reliability at transport  layer are mandatory to ensure no data loss.

 ### Efficiency (The Local Fix)

Imagine sending a packet across ten hops. If an error occurs on the very first wireless hop (which are notoriously "noisy"), it’s much faster to detect and retransmit that packet immediately between those two adjacent nodes.

- **Without Link-Level Reliability:** The error wouldn't be noticed until the packet reached the destination (or timed out), requiring a retransmission across all ten hops. That's a huge waste of bandwidth and time.

![[Chapter_6_v9.0-8.webp]]
![[Chapter_6_v9.0-9.webp]]



- Unlike application layer and transport layer, where everything is mostly handled by the OS kernel or the software, Network layer is implemented by *NIC*.

- It is a combination of *hardware* (the circuits), *firmware* (code running on the NIC), and *software* (the driver that allows the NIC and CPU to communicate via system bus)

- **Layer Responsibilities**: The NIC doesn't just do the link layer; it usually handles the **physical layer** (converting bits into electrical or optical signals) as well.


![[Chapter_6_v9.0-10.webp]]


## On the sending side:
- *Encapsulation* : NIC takes the datagram and wraps it with a header to create a *frame*.

- *Error Prep* : Adds error checking bits like *CRC* or *Checksum* so the receiver can check for errors

- *Transmission* : This implements the *Flow Control* and reliable delivery protocols we discussed.

## On the Receiving side:


-  *Validation* : Checks for errors on those incoming bits

- *Flow Control* : Controls the incoming rate so that the buffer does not get overwhelmed.

- *Decapsulation* : Removes the link layer header from the frame and extracts the datagram before passing to the OS

![[Chapter_6_v9.0-11.webp]]
![[Chapter_6_v9.0-12.webp]]

Because physical links (especially wireless ones) are prone to interference, the sender includes extra bits specifically for checking the integrity of the data.

- **EDC Bits**: These are redundant bits added to the frame to help the receiver identify if the data was garbled.
    
- **The Process**: The receiver performs a calculation on the received data (D′) and the error-checking bits (EDC′). If the result doesn't match the logic of the protocol, it flags a "detected error".
    
- **Not 100% Reliable**: While a larger EDC field provides better protection, it's technically possible for a protocol to miss certain complex error patterns.

![[Chapter_6_v9.0-13.webp]]

### Single Bit Parity

The sender adds a single bit at the end of the data string to make the total number of "1" bits either even or odd.

- **Detection**: If a single bit flips during transmission, the parity will no longer match (e.g., you expected an even number of 1s but got an odd number), and the receiver knows an error occurred.
    
- **Limitation**: If _two_ bits flip, the parity still looks "correct," and the error goes undetected.
    

### Two-Dimensional Parity

This is a much smarter version where the data is arranged in a grid.

- **Detection & Correction**: Parity bits are calculated for every **row** and every **column**.
    
- **The "Magic"**: If a single bit flips, it will trigger an error in exactly one row and one column. The intersection of that row and column tells the receiver exactly which bit is wrong, allowing it to **correct the error** without asking for a retransmission.

![[Chapter_6_v9.0-14.webp]]

 The goal is to determine if the data received is identical to the data sent. The sender performs a calculation to produce a fixed-size value (the checksum) that represents the entire data block.

- **16-bit Segments**: The sender treats the contents of the segment (including header fields) as a sequence of **16-bit integers**.
    
- **One's Complement Sum**: The sender calculates the sum of these integers using **one's complement arithmetic**.
    
- **Final Field**: This final sum is placed in the dedicated checksum field within the packet header.
    

---

### **2. The Process: Sender vs. Receiver**

#### **At the Sender:**

1. Divide the data into 16-bit chunks.
    
2. Add all 16-bit integers together.
    
    - _Note: If the addition results in a "carry" bit at the end, that bit is added back to the lowest bit (wraparound)._
        
3. Take the **1's complement** (flip all 0s to 1s and vice-versa) of the result.
    
4. Store this value in the checksum header.
    

#### **At the Receiver:**

1. Compute the checksum of the entire received segment (including the original checksum value sent by the sender).
    
2. **The Result**: If the data is error-free, the sum of the data plus the checksum should result in all 1s (11111111...).
    
3. **Validation**:
    
    - If the result contains any 0s, an error is **detected**.
        
    - If the result is all 1s, **no error is detected**, though there is still a small statistical possibility that compensating errors occurred.
- 
![[Chapter_6_v9.0-15.webp]]
![[Chapter_6_v9.0-16.webp]]

### At the Sender:

1. **Shift the Data**: The sender appends r zeros to the end of the data bits (D⋅2r).
    
2. **Divide by G**: The sender performs binary long division of this padded data by the generator G using XOR.
    
3. **Find the Remainder**: The result of this division is a remainder R.
    
4. **Create the Frame**: The sender replaces those r zeros with R. The final string to be sent is ⟨D,R⟩.
    

### At the Receiver:

1. **Receive the Bits**: The receiver gets the bits ⟨D′,R′⟩.
    
2. **Divide by G**: The receiver divides the entire received string by the same generator G.
    
3. **Check for Zero**:
    
    - If the remainder is **zero**, the data is assumed to be error-free.
        
    - If the remainder is **non-zero**, an error is detected.
- 
![[Chapter_6_v9.0-17.webp]]
![[Chapter_6_v9.0-18.webp]]
![[Chapter_6_v9.0-19.webp]]

## 1. Types of Links: Point-to-Point vs. Broadcast

The first slide distinguishes how devices are physically or logically connected.

- **Point-to-Point:** A direct "private line" between two devices. Think of it like a telephone call between two people.
    
    - _Examples:_ Your PC connected directly to an Ethernet switch, or old-school dial-up (PPP).
        
- **Broadcast (Shared Medium):** Many devices are connected to the same "airwaves" or wire. If everyone speaks at once, no one is heard.
    
    - _Examples:_ WiFi (shared air), Satellite, and 4G/5G.
        
    - **The Cocktail Party Analogy:** This is a great way to visualize it. In a crowded room, everyone shares the same air. To be heard, you have to wait for a gap in the noise or talk louder, but if everyone talks at once, it’s just chaos.
        

---

## 2. The Problem: Collisions

When you have a **shared broadcast channel**, you run into the "Collision" problem.

- **What is a collision?** It’s when two or more nodes transmit at the exact same time. The signals interfere with each other, resulting in "garbage" data that the receiver cannot decode.
    
- **The Challenge:** Unlike a classroom where a teacher (a central controller) points to a student to speak, computer networks often have to decide who speaks next in a **distributed** way.
    

---

## 3. The Solution: Multiple Access Protocols (MAPs)

Because there is no "boss" to tell everyone when it's their turn, nodes use a **Multiple Access Protocol**. This is basically a set of rules (a distributed algorithm) that determines:

1. When a node is allowed to transmit.
    
2. What to do if a collision happens.

![[Chapter_6_v9.0-20.webp]]
![[Chapter_6_v9.0-21.webp]]
![[Chapter_6_v9.0-22.webp]]
![[Chapter_6_v9.0-23.webp]]
![[Chapter_6_v9.0-24.webp]]
![[Chapter_6_v9.0-25.webp]]

## Slotted ALOHA

Slotted ALOHA introduces a bit of structure to reduce the "collision window" seen in the original (pure) ALOHA.

### Core Assumptions

- **Synchronized Timing**: All nodes are synchronized and time is divided into equal slots.
    
- **Fixed Frame Size**: All frames are the same size, designed to fit perfectly into one time slot.
    
- **Slot-Start Rule**: Nodes can only begin transmitting at the very start of a slot.
    
- **Collision Detection**: If two or more nodes transmit in a slot, all nodes detect the collision.
    


![[Chapter_6_v9.0-26.webp]]

### Operation

1. **Fresh Frame**: When a node gets a new frame, it transmits it in the very next slot.
    
2. **Success**: If there’s no collision, the node is finished with that frame and can send another in the next slot.
    
3. **Collision Recovery**: If a collision occurs, the node **retransmits** the frame in each subsequent slot with **probability p** until it succeeds.
    

> **Why the randomization?** If nodes immediately retransmitted in the next slot after a collision, they would just collide again (and again). Using probability p "staggers" the attempts, giving each node a chance to find an empty slot.

![[Chapter_6_v9.0-27.webp]]
![[Chapter_6_v9.0-28.webp]]

## 1. CSMA: Carrier Sense Multiple Access

The core philosophy of CSMA is simple: **listen before transmit**.

- **Idle Channel**: If the node senses that the channel is idle, it transmits the entire frame.
    
- **Busy Channel**: If the channel is sensed as busy, the node defers its transmission.
    
- **Human Analogy**: This is like a polite person who doesn't interrupt others while they are talking.
    

---



![[Chapter_6_v9.0-29.webp]]

## 2. Why Collisions Still Occur

Despite the "listen before transmit" rule, collisions are still possible due to **propagation delay**.

- **The Problem**: A node might start transmitting, but because the signal takes time to travel through the medium, another node further away might sense the channel as "idle" and start its own transmission.
    
- **Wasted Time**: When a collision occurs in basic CSMA, the entire packet transmission time is wasted because the nodes finish sending their corrupted frames before realizing a collision happened.
    
- **Variables**: The probability of these collisions is determined by the physical distance between nodes and the propagation delay of the medium.
- 
![[Chapter_6_v9.0-30.webp]]
## CSMA/CD: Collision Detection

To address the "wasted time" issue in basic CSMA, **CSMA/CD (Collision Detection)** was developed.

- **Rapid Detection**: Collisions are detected within a very short time.
    
- **Aborting Transmissions**: Once a collision is detected, the colliding nodes immediately abort their transmissions, significantly reducing channel wastage.
    
- **Medium Constraints**: Collision detection is relatively easy to implement in **wired** networks (by measuring signal strength) but is very difficult in **wireless** networks.
    
- **Human Analogy**: This is like "the polite conversationalist" who stops talking the instant they realize someone else started at the same time.

![[Chapter_6_v9.0-31.webp]]
### 1. Preparation

The network interface card (NIC) receives data from the network layer and wraps it into an **Ethernet frame**.

### 2. Carrier Sensing (The "Listen" Phase)

Before sending, the NIC "senses" the wire:

- **Idle:** If no one else is transmitting, it starts sending immediately.
    
- **Busy:** If it detects a signal, it waits until the channel is clear.
    

### 3. Success vs. Collision

- **Success:** If the entire frame is sent without interference, the job is finished.
    
- **Collision:** If another device starts sending at the same time, their signals "collide" on the wire.
    

### 4. Collision Handling

If a collision is detected:

1. **Abort:** The NIC stops sending the current frame.
    
2. **Jam Signal:** It sends a special "jam signal" to ensure all other devices on the network know a collision occurred and should also stop.
    

### 5. Binary Exponential Backoff

To prevent the same two devices from colliding again immediately, they use a randomized waiting period:

- **Choosing K:** After the mth collision, the device picks a random number K from the range {0,1,2,…,2m−1}.
    
- **Waiting:** It waits for K×512 bit times (the time it takes to transmit 512 bits).
    
- **Escalation:** The more collisions that occur (increasing m), the larger the possible range for K becomes. This exponentially increases the potential wait time, which helps clear heavy congestion.

![[Chapter_6_v9.0-32.webp]]
## **Comparing MAC Protocol Families**

MAC (Medium Access Control) protocols are generally categorized by how they handle channel access:

| Protocol Type            | Pros                                                                         | Cons                                                                                                     |
| ------------------------ | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **Channel Partitioning** | Efficient and fair under **high load**.                                      | Inefficient at **low load**; introduces delay and fixed bandwidth (1/N) even if only one node is active. |
| **Random Access**        | Efficient at **low load**; a single node can use the full channel bandwidth. | High collision overhead under **high load**.                                                             |
| **"Taking Turns"**       | Aims for the "best of both worlds."                                          | Subject to overhead and single points of failure.                                                        |

![[Chapter_6_v9.0-33.webp]]

In a polling setup, a **centralized controller** (the "master") "invites" each node to transmit its data in a specific order.

- **Common Use**: Often used with "dumb" devices that don't have complex internal logic.
    
- **Real-world Example**: Bluetooth technology utilizes polling.
    
- **Primary Concerns**:
    
    - **Polling Overhead**: The control messages themselves consume bandwidth.
        
    - **Latency**: Nodes must wait for their "turn" to be invited.
        
    - **Single Point of Failure**: If the centralized controller fails, the entire network goes down.
    
![[Chapter_6_v9.0-34.webp]]

### **Token Passing**

In this decentralized approach, a special control message called a **token** is passed sequentially from one node to the next.

- **Mechanism**: A node can only transmit data while it is holding the token. Once finished (or if it has nothing to send), it passes the token to the next node in the ring.
    
- **Primary Concerns**:
    
    - **Token Overhead**: Bandwidth is used to pass the token even when no data is being sent.
        
    - **Latency**: A node must wait for the token to circulate back to it.
        
    - **Single Point of Failure**: If the token is lost or a node holding it fails, the network must have a recovery mechanism to regenerate it.

![[Chapter_6_v9.0-35.webp]]  

## 1. Physical Architecture

The network uses a **shared medium**, meaning multiple homes are connected to the same physical cable.

- **CMTS (Cable Modem Termination System):** Located at the "cable headend" (the ISP's central office), this acts as the primary controller for all modems in the neighborhood.
    
- **Cable Modem:** The device in the home that communicates with the CMTS.
    
- **Splitter:** Divides the signal within the home to both the TV and the internet modem.
    

---

## 2. Multiplexing Strategies

The network handles traffic differently depending on the direction of data:

### **Downstream (ISP to User)**

- **FDM (Frequency Division Multiplexing):** The cable is divided into different frequency bands. One band might carry a TV channel, while another carries internet data.
    
- **Broadcast Nature:** The CMTS is the only sender. It broadcasts data to all modems, and your specific modem "listens" for the packets addressed to it.
    
- **Speed:** Up to **1.6 Gbps** per channel.
    

### **Upstream (User to ISP)**

This is more complex because multiple users are trying to talk back to the ISP at the same time on the same wire.

- **TDM (Time Division Multiplexing):** The CMTS divides the upstream time into "slots."
    
- **The MAP Frame:** The second slide shows a **MAP frame**. This is a schedule sent downstream by the CMTS that tells each modem exactly which time slot it is allowed to use to send data.


![[Chapter_6_v9.0-36.webp]]

## 3. Random Access & Contention

Since the CMTS needs to know you have data to send before it can assign you a slot, there is a "chicken and egg" problem. DOCSIS solves this with two types of slots:

- **Assigned Slots:** Reserved for specific modems to send their actual data (no collisions).
    
- **Contention Slots (Random Access):** These are used by modems to send "requests" for data slots.
    
    - Because any modem can try to use these slots at any time, collisions can occur.
        
    - If a collision happens, the modem uses a **binary backoff** algorithm (waiting a random amount of time) before trying to request a slot again.
        

### Summary Table

| Direction      | Method              | Management                                 |
| -------------- | ------------------- | ------------------------------------------ |
| **Downstream** | FDM                 | Centralized (CMTS sends to all)            |
| **Upstream**   | TDM + Random Access | Scheduled via MAP frames and request slots |
![[Chapter_6_v9.0-37.webp]]
![[Chapter_6_v9.0-38.webp]]
![[Chapter_6_v9.0-39.webp]]
![[Chapter_6_v9.0-40.webp]]
![[Chapter_6_v9.0-41.webp]]
![[Chapter_6_v9.0-42.webp]]
![[Chapter_6_v9.0-43.webp]]
![[Chapter_6_v9.0-44.webp]]
![[Chapter_6_v9.0-45.webp]]
![[Chapter_6_v9.0-46.webp]]
![[Chapter_6_v9.0-47.webp]]
![[Chapter_6_v9.0-48.webp]]
![[Chapter_6_v9.0-49.webp]]
![[Chapter_6_v9.0-50.webp]]
![[Chapter_6_v9.0-51.webp]]
![[Chapter_6_v9.0-52.webp]]
![[Chapter_6_v9.0-53.webp]]
![[Chapter_6_v9.0-54.webp]]
![[Chapter_6_v9.0-55.webp]]
![[Chapter_6_v9.0-56.webp]]
![[Chapter_6_v9.0-57.webp]]
![[Chapter_6_v9.0-58.webp]]
![[Chapter_6_v9.0-59.webp]]
![[Chapter_6_v9.0-60.webp]]
![[Chapter_6_v9.0-61.webp]]
![[Chapter_6_v9.0-62.webp]]
![[Chapter_6_v9.0-63.webp]]
![[Chapter_6_v9.0-64.webp]]
![[Chapter_6_v9.0-65.webp]]
![[Chapter_6_v9.0-66.webp]]
![[Chapter_6_v9.0-67.webp]]
![[Chapter_6_v9.0-68.webp]]
![[Chapter_6_v9.0-69.webp]]
![[Chapter_6_v9.0-70.webp]]
![[Chapter_6_v9.0-71.webp]]
![[Chapter_6_v9.0-72.webp]]
![[Chapter_6_v9.0-73.webp]]
![[Chapter_6_v9.0-74.webp]]
![[Chapter_6_v9.0-75.webp]]
![[Chapter_6_v9.0-76.webp]]
![[Chapter_6_v9.0-77.webp]]
![[Chapter_6_v9.0-78.webp]]
![[Chapter_6_v9.0-79.webp]]
![[Chapter_6_v9.0-80.webp]]
![[Chapter_6_v9.0-81.webp]]
![[Chapter_6_v9.0-82.webp]]
![[Chapter_6_v9.0-83.webp]]
![[Chapter_6_v9.0-84.webp]]
![[Chapter_6_v9.0-85.webp]]
![[Chapter_6_v9.0-86.webp]]
![[Chapter_6_v9.0-87.webp]]
![[Chapter_6_v9.0-88.webp]]
![[Chapter_6_v9.0-89.webp]]
![[Chapter_6_v9.0-90.webp]]
![[Chapter_6_v9.0-91.webp]]
![[Chapter_6_v9.0-92.webp]]
![[Chapter_6_v9.0-93.webp]]
![[Chapter_6_v9.0-94.webp]]
![[Chapter_6_v9.0-95.webp]]
![[Chapter_6_v9.0-96.webp]]
![[Chapter_6_v9.0-97.webp]]
![[Chapter_6_v9.0-98.webp]]
![[Chapter_6_v9.0-99.webp]]
![[Chapter_6_v9.0-100.webp]]
![[Chapter_6_v9.0-101.webp]]
![[Chapter_6_v9.0-102.webp]]
![[Chapter_6_v9.0-103.webp]]
![[Chapter_6_v9.0-104.webp]]
![[Chapter_6_v9.0-105.webp]]
![[Chapter_6_v9.0-106.webp]]
![[Chapter_6_v9.0-107.webp]]
![[Chapter_6_v9.0-108.webp]]
![[Chapter_6_v9.0-109.webp]]