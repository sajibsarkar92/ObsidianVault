
![[Transport_Layer_Part_II_MSI.pptx 1-1.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-2.webp]]

![[Transport_Layer_Part_II_MSI.pptx 1-3.webp]]
# [[Transport_Layer_Part_II_MSI.pdf#page=3|TCP Segment Structure (Slide 3)]]


## The Header Breakdown

Think of the TCP header as the "control panel" of the segment. It’s significantly beefier than UDP (20 bytes vs 8 bytes) because it has much more to keep track of.

### 1. The Basics (Top Row)

- **Source & Dest Port:** Just like UDP. This is the "Room Number" (Process) on the Host.
    

### 2. The Heavy Lifters (Middle Rows)

- **Sequence Number (32 bits):** The "ID" of the first byte of data in this segment.
    
- **Acknowledgement Number (32 bits):** The ID of the **next** byte expected from the other side.
    

### 3. The Flags (The "Alphabet Soup")

These 1-bit flags tell the receiver what kind of segment this is:

- **A (ACK):** "The Ack number in this header is valid."
    
- **R (RST), S (SYN), F (FIN):** Used for connection setup and teardown (The Handshake).
    
- **P (PSH):** Tells the receiver to push data to the app immediately.
    
- **U (URG):** Marks data as urgent (rarely used).
    

### 4. Management Fields

- **Receive Window:** This is for **Flow Control**. It tells the sender: "I only have this much room left in my buffer, don't overwhelm me!"
    
- **Checksum:** Error detection (to see if bits flipped during transit).
    
- **Head Len:** Since the "Options" field varies in size, this tells the receiver where the actual data (payload) starts.
    

---

## ⚡ Key Engineering Detail

> [!IMPORTANT] Header Size A standard TCP header is **20 bytes**. If you see a question about overhead, remember that every time you send even 1 byte of data via TCP, you are attaching at least 20 bytes of "paperwork" to it.

---

## 💡 Intuitive Summary

> [!TIP] The "Package Label" Intuition If a UDP header is just a sticky note with a name on it, a **TCP Header is a full manifest**:
> 
> - It says where it's going (**Ports**).
>     
> - It says where it fits in the puzzle (**Sequence #**).
>     
> - It asks if previous packages arrived (**ACK #**).
>     
> - It warns the sender if the mailbox is getting full (**Receive Window**).


![[Transport_Layer_Part_II_MSI.pptx 1-4.webp]]

![[Transport_Layer_Part_II_MSI.pptx 1-5.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-6.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-7.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-8.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-9.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-10.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-11.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-12.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-13.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-14.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-15.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-16.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-17.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-18.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-19.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-20.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-21.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-22.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-23.webp]]
![[Transport_Layer_Part_II_MSI.pptx 1-24.webp]]