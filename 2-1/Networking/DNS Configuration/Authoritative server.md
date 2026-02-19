
           Before Phase 4, let me explain **why we're doing this** and **what we're building**.

---

## The Big Picture: What We've Done So Far

| Phase | What We Did | Role |
|-------|-------------|------|
| **Phase 1** | Used `dig` to query DNS | **Client** (ask questions) |
| **Phase 2** | Ran BIND as recursive resolver | **Resolver** (find answers) |
| **Phase 3** | Understood DNS hierarchy & GLUE | **Analyst** (understand why things break) |

---

## What Phase 4 Adds: Authoritative Server

| Role | Function | Analogy |
|------|----------|---------|
| **Client** (Phase 1) | Asks DNS questions | Customer at restaurant |
| **Resolver** (Phase 2) | Finds answers by asking others | Waiter who checks kitchen |
| **Authoritative** (Phase 4) | **Owns the actual data** | **Chef who prepares the food** |

**Phase 4 = You become the source of truth for your domain**

---

## Why This Matters for Your Lab

### Your Lab Assignment
| Element | Value |
|---------|-------|
| Your machine IP | `10.17.100.RR` (RR = your roll number, e.g., 16) |
| Your domain | `s16.zhjlab.bd` (e.g., if roll 16) |
| Your job | Run DNS server that **authoritatively answers** for this domain |

### The Goal
When someone queries `www.s16.zhjlab.bd`, **your BIND server** answers with the IP address.

```
Someone asks: "What is www.s16.zhjlab.bd?"

    ↓

Your BIND server (at 10.17.100.16) responds:
"www.s16.zhjlab.bd is at 10.17.100.16"

    ↓

Query satisfied from authoritative source!
```

---

## What "Authoritative" Means

| Type | Answers From | Example |
|------|--------------|---------|
| **Recursive query** | Cache or other servers | "Let me look that up for you" |
| **Authoritative query** | **Own zone files** | "I **own** this domain, here's the answer" |

**Key difference:** Authoritative server doesn't ask anyone else. It has the **master copy** of the data.

---

## Why This Is Different from Phase 2

| Phase 2 (Recursive) | Phase 4 (Authoritative) |
|---------------------|------------------------|
| Answers any question | Answers **only for your domain** |
| Queries internet if needed | **Never queries internet** for your zone |
| Caches results | **Owns the original records** |
| `forwarders` configured | `zone` configured with zone file |
| Generic resolver | **Source of truth** for specific domain |

---

## What We're Building

```
┌─────────────────────────────────────────┐
│           YOUR MACHINE                  │
│  (Fedora/Ubuntu with BIND9)             │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  RECURSIVE MODE (Phase 2)       │    │
│  │  • Resolves any domain          │    │
│  │  • Uses forwarders              │    │
│  │  • Caches results               │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  AUTHORITATIVE MODE (Phase 4)   │    │
│  │  • Owns s16.zhjlab.bd zone      │    │
│  │  • Answers from zone file       │    │
│  │  • No forwarding for this zone  │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Zone file: db.s16.zhjlab.bd            │
│  Contains: www, mail, ns1 records       │
└─────────────────────────────────────────┘
              │
              ▼
    "I am the authority for s16.zhjlab.bd"
```

---

## The Zone File Concept

**Zone file = Text file containing DNS records you own**

| Record | Purpose | Example |
|--------|---------|---------|
| `SOA` | Start of Authority - "I am the primary server" | `@ IN SOA ns1.s16.zhjlab.bd. admin.s16.zhjlab.bd. (...)` |
| `NS` | Name Server delegation | `@ IN NS ns1.s16.zhjlab.bd.` |
| `A` | IPv4 address | `www IN A 10.17.100.16` |
| `CNAME` | Alias | `ftp IN CNAME www` |
| `MX` | Mail server | `@ IN MX 10 mail.s16.zhjlab.bd.` |

---

## What Success Looks Like

| Test | Command | Expected Result |
|------|---------|---------------|
| Local test | `dig @127.0.0.1 www.s16.zhjlab.bd` | Returns your IP |
| Network test | `dig @10.17.100.16 www.s16.zhjlab.bd` | Returns your IP |
| Authority check | `dig @10.17.100.16 s16.zhjlab.bd NS` | Shows `aa` flag (Authoritative Answer) |

---

## Prerequisites for Phase 4

| Requirement | Status |
|-------------|--------|
| BIND installed | ✅ Phase 2 |
| BIND running | ✅ Phase 2 |
| Know your roll number | ❓ You tell me |
| Know your assigned IP | ❓ `10.17.100.RR` |

**What is your roll number?** I'll use it in the examples (e.g., if you say 16, domain is `s16.zhjlab.bd`).

---

## Summary: Phase 4 Goals

| Goal | How |
|------|-----|
| Create zone file | `db.s16.zhjlab.bd` with your records |
| Declare zone | Add to `named.conf.local` (Ubuntu) or `named.conf` (Fedora) |
| Test locally | `dig @127.0.0.1` queries |
| Verify authority | Check for `aa` (Authoritative Answer) flag |

**Ready to start? What's your roll number?**