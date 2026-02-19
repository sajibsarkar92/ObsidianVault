
           Perfect! Roll 16 means:

| Your Setting | Value |
|-------------|-------|
| **Your IP** | `10.17.100.16` |
| **Your domain** | `s16.zhjlab.bd` |
| **Your name server** | `ns1.s16.zhjlab.bd` |

---

## Phase 4: Making BIND Authoritative for `s16.zhjlab.bd`

We'll create your zone file and configure BIND to answer authoritatively for your domain.

---

## Step 1: Telling BIND about your Zone

First, we have to tell the BIND software: _"Hey, you are now the boss of the domain `s16.zhjlab.bd`."_

Since you are likely on **Ubuntu** for this lab, we use a specific configuration file to declare this.

### The Action:

1. Open your terminal.
    
2. Open the file `/etc/bind/named.conf.local` with `sudo` and a text editor (like `nano` or `gedit`).
    
3. Add the following block to the bottom of the file:
    

Bash

```
zone "s16.zhjlab.bd" {
    type master;
    file "/etc/bind/db.s16.zhjlab.bd";
};
```

### What this code means:

- **`zone "s16.zhjlab.bd"`**: This defines the name of your "kingdom."
    
- **`type master;`**: This means your server is the **Authoritative** source (the Master). It doesn't ask others for this info.
    
- **`file "/etc/bind/db.s16.zhjlab.bd";`**: This tells BIND exactly where the list of records (the "Zone File") is located.


This is a much safer way to do it because starting from a template like `db.local` ensures you keep the correct formatting that BIND expects.

---

## Step 2: Creating the Zone File (The Database)

**Why we are doing this:**

Instead of typing everything from scratch (where a single missing semicolon can break the whole server), we use a "skeleton" file provided by Ubuntu. We then perform "surgery" on that skeleton to make it represent your specific domain and IP.

### 1. The Command: Copy the Template

First, we make a copy of the default local loopback template and name it after your domain.

Bash

```
sudo cp /etc/bind/db.local /etc/bind/db.s16.zhjlab.bd
```

### 2. The Action: Open and Modify

Open your new file:

Bash

```
sudo nano /etc/bind/db.s16.zhjlab.bd
```

### 3. What to Modify (The "Surgery")

When you open the file, it will look like a generic template. You need to change three specific things:

#### **A. Update the SOA (The Domain Name)**

Find the line that says `localhost.` and replace it with your domain name.

- **Change from:** `localhost. root.localhost.`
    
- **Change to:** `ns1.s16.zhjlab.bd. admin.s16.zhjlab.bd.`
    
- **Why:** You are telling BIND that `ns1.s16.zhjlab.bd` is now the authority, and `admin@s16.zhjlab.bd` is the contact.
    

#### **B. Update the Serial Number**

Find the number `2` (usually under "Serial").

- **Change to:** `2026021901`
    
- **Why:** This tells other servers "This is a fresh update from today (Feb 19, 2026)." If you change this file later, increment this to `...02`.
    

#### **C. Update the Records (The bottom of the file)**

Delete the lines at the bottom (the ones with `127.0.0.1` and `::1`) and add your specific server records.

- **Change from:**
    
    Plaintext
    
    ```
    @   IN  NS  localhost.
    @   IN  A   127.0.0.1
    @   IN  AAAA ::1
    ```
    
- **Change to:**
    
    Plaintext
    
    ```
    @       IN      NS      ns1.s16.zhjlab.bd.
    ns1     IN      A       10.17.100.16
    www     IN      A       10.17.100.16
    mail    IN      A       10.17.100.16
    ```
    
- **Why:** * The `NS` line tells the world _who_ the boss is (`ns1`).
    
    - The `A` lines tell the world _where_ the boss is located (your machine's IP `10.17.100.16`).
        

---

### The Final Result

Your file should now look exactly like this:

Plaintext

```
$TTL    604800
@       IN      SOA     ns1.s16.zhjlab.bd. admin.s16.zhjlab.bd. (
                        2026021901      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.s16.zhjlab.bd.
ns1     IN      A       10.17.100.16
www     IN      A       10.17.100.16
mail    IN      A       10.17.100.16
```


### Step 3: Syntax Verification (The "Spell-Check")

**Why we are doing this:** BIND is extremely sensitive. If you forgot a semicolon `;`, a bracket `}`, or a tiny dot `.` at the end of a domain name, the service will simply refuse to start. Instead of restarting the server and having it crash, we use "pre-flight" tools to check our work.

Think of this like a compiler for your code—it finds the bugs before you "run" the program.

---

### The Action:

We need to check two things: the **main configuration** and the **specific zone file**.

#### 1. Check the Main Configuration

Run this command to check `/etc/bind/named.conf.local` (the file from Step 1):

Bash

```
sudo named-checkconf
```

- **What to look for:** If it returns **nothing**, that is perfect. It means your syntax is correct. If it shows an error, it will tell you exactly which line has the mistake.
    

#### 2. Check the Zone File

Run this command to check the "database" you just created (the file from Step 2):

Bash

```
sudo named-checkzone s16.zhjlab.bd /etc/bind/db.s16.zhjlab.bd
```

- **What to look for:** You want to see: `OK` and `Loaded serial 2026021901`.
    

---

### Troubleshooting (The "Why" if it fails)

If you get an error, it is usually one of these three things:

1. **Missing Semicolon:** Every line in the configuration (Step 1) must end in a `;`.
    
2. **Missing Dot:** In the Zone file (Step 2), full domains like `ns1.s16.zhjlab.bd.` **must** end with a dot.
    
3. **Permissions:** If the tool says "Permission denied," make sure you are using `sudo`.


---

## Step 4: Activating the Authority

**Why we are doing this:** Even though you've written the files and verified they are correct, BIND is still running with its **old** instructions from Phase 2. To make it "load" your new role as the Chef/Authority, you have to restart the service. This forces BIND to read `/etc/bind/named.conf.local` and your new zone file.

### The Action:

1. **Restart BIND:**
    
    Bash
    
    ```
    sudo systemctl restart bind9
    ```
    
2. **Check the status:**
    
    Bash
    
    ```
    sudo systemctl status bind9
    ```
    

**What to look for:** You want to see `active (running)`. If you see `failed`, it usually means there was a mistake that the check-tools missed (like a file path error).

---

## Step 5: The Final Test (Verification)

**Why we are doing this:** This is the "moment of truth." We need to prove that your server is actually answering for `s16.zhjlab.bd` and, most importantly, that it is doing so as an **Authoritative** source.

### The Action:

Run the `dig` command against your own local server:

Bash

```
dig @127.0.0.1 www.s16.zhjlab.bd
```

### How to Read the Output (The "Proof"):

Look closely at the **flags** section of the output.

1. **The Answer:** You should see `www.s16.zhjlab.bd` pointing to `10.17.100.16` in the **ANSWER SECTION**.
    
2. **The Authority Flag (`aa`):** This is the most important part. In the header, you should see `flags: qr aa rd ra;`.
    
    - **`aa` stands for "Authoritative Answer."** * If you see `aa`, it means your server is saying: _"I didn't ask anyone else; I own this data."_
        

---

### What if it doesn't work?

- **No `aa` flag:** You might be hitting a cached result or your zone didn't load. Check `tail -f /var/log/syslog` while restarting BIND to see specific errors.
    
- **Connection Refused:** BIND might have crashed. Re-run Step 3 checks.
    

**Try the `dig` command now. Do you see the `aa` flag in the output?**