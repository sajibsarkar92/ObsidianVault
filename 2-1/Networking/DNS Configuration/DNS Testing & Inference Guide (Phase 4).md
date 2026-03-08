

In this phase, we move beyond simply getting an answer. We are looking for **Proof of Authority**.

---

## 1. The Real-Time Logic Check: `tail -f /var/log/syslog`

**Purpose:** To observe the BIND9 engine as it attempts to "swallow" your configuration files. This is the first thing you should do after any change.

- **Command:** `sudo tail -f /var/log/syslog`
    
- **Alternative (Fedora/Modern Ubuntu):** `sudo journalctl -u bind9 -f`
    

### Inference Rules:

|Message Observed|Inference (What it means)|
|---|---|
|`loaded serial 2026021901`|**SUCCESS.** BIND has accepted your zone file and is now using the version from your serial number.|
|`zone s16.zhjlab.bd/IN: NS 'ns1...' has no address record (A)`|**ERROR.** You defined a Name Server in Step 1/2 but forgot to create the `A` record for it at the bottom of the file.|
|`unexpected end of input`|**SYNTAX ERROR.** You likely forgot a closing bracket `}` in `named.conf.local` or a parenthesis `)` in the SOA record.|
|`permission denied`|**OS ERROR.** BIND doesn't have permission to read your file. (Check `ls -l` permissions).|

Export to Sheets

---

## 2. The Master Record Query: `dig -t soa s16.zhjlab.bd`

**Purpose:** To verify the "Birth Certificate" of the domain. This command asks for the **Start of Authority** record.

- **Command:** `dig @127.0.0.1 -t soa s16.zhjlab.bd`
    

### Inference Rules:

- **The `aa` Flag (Crucial):** Look at the header line: `flags: qr aa rd ra`.
    
    - **Inference:** If `aa` (Authoritative Answer) is present, your server is the "Chef." If it is missing, your server is just a "Waiter" forwarding the question elsewhere.
        
- **The Serial Number:** Check the first number in the **ANSWER SECTION**.
    
    - **Inference:** If it matches your `YYYYMMDDNN` format, you are looking at live data. If it is a small number (like `2`), the server is likely using an old cached version or a default template.
        
- **The Status:** Look for `status: NOERROR`.
    
    - **Inference:** Anything else (like `SERVFAIL`) means your zone file exists but is "broken" internally.
        

---

## 3. The Delegation Query: `dig NS s16.zhjlab.bd`

**Purpose:** To confirm the "Signposting." This command asks: "Who is the boss of this domain?"

- **Command:** `dig @127.0.0.1 s16.zhjlab.bd NS`
    

### Inference Rules:

- **The ANSWER SECTION:** It should list `ns1.s16.zhjlab.bd.`.
    
    - **Inference:** This confirms that your server is correctly advertising itself as the primary name server for this domain.
        
- **The ADDITIONAL SECTION:** Look for an `A` record for `ns1` showing `10.17.100.16`.
    
    - **Inference:** This is a "Glue Record." It proves the server is smart enough to provide the IP address of the name server it just recommended, preventing a circular logic loop.
        
- **The Trailing Dot:** Check if the names end in a `.`.
    
    - **Inference:** If you see `ns1.s16.zhjlab.bd.s16.zhjlab.bd`, you forgot the trailing dot in your zone file.
        

---

## Summary Checklist for Lab Submission

When your teacher asks you to prove your lab works, show these three things in order:

1. **Status Check:** `systemctl status bind9` (Shows it is running).
    
2. **Authority Check:** `dig @127.0.0.1 www.s16.zhjlab.bd` (Show the **`aa` flag** and your **IP `10.17.100.16`**).
    
3. **Consistency Check:** `dig @127.0.0.1 -t soa s16.zhjlab.bd` (Show that the **Serial Number** matches your latest update).