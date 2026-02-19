

---


---

### 1. What is a GLUE Record?

| Concept | Explanation |
|---------|-------------|
| **Definition** | An A record in the **parent zone** that provides the IP address of a child name server |
| **Purpose** | Breaks circular dependency in DNS resolution |
| **Why needed** | Without IP, resolver cannot reach the name server it's told to query |

**The Circular Problem Without GLUE:**
```
To find s01.zhjlab.bd:
    Ask ns1.s01.zhjlab.bd
    
To find ns1.s01.zhjlab.bd:
    Need to resolve s01.zhjlab.bd first!
    
Back to start → INFINITE LOOP! 💥
```

**With GLUE:**
```
To find s01.zhjlab.bd:
    Ask ns1.s01.zhjlab.bd at 172.17.100.1  ← IP provided!
    
Query 172.17.100.1 directly → Get answer! ✅
```

---

### 2. Commands to Test DNS Hierarchy

| Purpose | Command |
|---------|---------|
| **Trace full resolution path** | `dig +trace domain.com` |
| **Query specific server for NS** | `dig @server domain NS` |
| **Query specific server for A** | `dig @server hostname A` |
| **Check if GLUE exists** | Look for "ADDITIONAL SECTION" with IP |

**Fedora/Ubuntu:** Same commands on both.

---

### 3. The Resolution Chain

| Step | Server Type | Example | Role |
|------|-------------|---------|------|
| 1 | Root (`.`) | `a.root-servers.net` | Points to TLD servers |
| 2 | TLD (`.com`, `.bd`) | `surma.btcl.net.bd` | Points to authoritative servers |
| 3 | Authoritative | `dns1.du.ac.bd` | Owns the zone data |
| 4 | **GLUE needed here** | `ns1.s01.zhjlab.bd = 172.17.100.1` | IP to reach child zone |

---

### 4. Testing GLUE: Working vs Broken

#### Working Example (Yahoo):
```bash
dig +trace yahoo.com
```
**Output shows:**
```
yahoo.com.  NS  ns1.yahoo.com.
            +   ns1.yahoo.com. A 68.180.131.16  ← GLUE present!
```

| Check | Result |
|-------|--------|
| NS record | ✅ Present |
| GLUE (A record) | ✅ Present |
| Final query succeeds | ✅ Yes |

---

#### Broken Example (s01.zhjlab.bd from home):
```bash
dig +trace s01.zhjlab.bd
```
**Output shows:**
```
s01.zhjlab.bd.  NS  ns1.s01.zhjlab.bd.
couldn't get address for 'ns1.s01.zhjlab.bd': not found  ← NO GLUE!
```

| Check | Result |
|-------|--------|
| NS record | ✅ Present |
| GLUE (A record) | ❌ Missing OR unreachable |
| Final query succeeds | ❌ No |

---

### 5. Direct Server Testing Commands

| Test | Command | Purpose |
|------|---------|---------|
| Query TLD server | `dig @surma.btcl.net.bd s01.zhjlab.bd NS` | Check if parent has delegation |
| Query authoritative | `dig @dns1.du.ac.bd s01.zhjlab.bd NS` | Check if child zone has GLUE |
| Compare answers | Run both, check "AUTHORITY" and "ADDITIONAL" sections | Find where GLUE exists/missing |

---

### 6. Interpreting dig Output Sections

| Section | Meaning | GLUE Location |
|---------|---------|---------------|
| `QUESTION SECTION` | What you asked for | — |
| `ANSWER SECTION` | Direct answer (if cached) | — |
| `AUTHORITY SECTION` | Who is authoritative (NS records) | Parent delegation |
| `ADDITIONAL SECTION` | Extra info (IP addresses) | **GLUE records here!** |

**Example with GLUE:**
```
;; AUTHORITY SECTION:
s01.zhjlab.bd.  NS  ns1.s01.zhjlab.bd.

;; ADDITIONAL SECTION:
ns1.s01.zhjlab.bd.  A  172.17.100.1  ← GLUE!
```

---

### 7. Why GLUE Can Still Fail

| Scenario | Explanation | Example |
|----------|-------------|---------|
| **No GLUE record** | Parent didn't provide IP | `+trace` fails with "couldn't get address" |
| **Wrong GLUE IP** | IP in parent doesn't match actual server | Points to non-existent host |
| **Unreachable GLUE IP** | IP is private, querier on different network | `172.17.100.1` from home internet |
| **Stale GLUE** | IP changed, parent not updated | Old IP no longer responds |

**Your case:** Unreachable private IP from home network.

---

### 8. Private vs Public IPs

| Type | Range | Reachable From |
|------|-------|--------------|
| **Public** | Any not in private ranges | Anywhere on internet |
| **Private - Class A** | `10.0.0.0 - 10.255.255.255` | Same private network only |
| **Private - Class B** | `172.16.0.0 - 172.31.255.255` | Same private network only |
| **Private - Class C** | `192.168.0.0 - 192.168.255.255` | Same private network only |

**Lab IPs:**
- `10.17.100.1` → Private Class A (university internal)
- `172.17.100.1` → Private Class B (university internal)

**From home:** ❌ Unreachable  
**From university:** ✅ Reachable

---

### 9. Key Takeaways

| Concept | Understanding |
|---------|---------------|
| **GLUE necessity** | Required to break circular dependency in DNS |
| **GLUE location** | Stored in parent zone, not child zone |
| **GLUE format** | A record mapping name server hostname to IP |
| **Reachability** | GLUE must point to publicly reachable IP (or same private network) |
| **Troubleshooting** | Use `+trace` to find where chain breaks, `@server` to test specific servers |

---

### 10. Common Errors & Meanings

| Error | Meaning | Cause |
|-------|---------|-------|
| `couldn't get address for 'ns1.X'`: not found | No GLUE record | Parent zone missing A record for NS |
| `couldn't get address for 'ns1.X'`: failure | NS name doesn't resolve | NS hostname invalid or non-existent |
| `NXDOMAIN` | Domain doesn't exist | Wrong domain name or no delegation |
| `SERVFAIL` | Server failed to answer | DNSSEC error, server overload, network issue |
| `REFUSED` | Server refused query | ACL restriction, recursion disabled |

---

### 11. Summary: What We Learned

| Before Phase 3 | After Phase 3 |
|----------------|---------------|
| Knew `+trace` shows path | Understand each step: root → TLD → auth |
| Saw "couldn't get address" error | Know it means **missing GLUE** |
| Thought DNS always works | Understand **private IPs break external queries** |
| Used dig for queries | Can **diagnose delegation problems** |

---

Ready for **Phase 4: Configuring BIND as Authoritative Server**?