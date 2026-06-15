# Practical 6: Network Scanning & Enumeration

**Atiam College - Cyber Security & Ethical Hacking**
**Module:** Week 8 - Scanning and Enumeration
**Duration:** 2–3 Hours
**Difficulty:** Intermediate

---

## Objective

By the end of this practical, you will be able to:

- Perform TCP and UDP port scans and understand the differences
- Use different Nmap scan types (SYN, Connect, Stealth)
- Perform banner grabbing with Netcat
- Enumerate SMB shares, users, and services
- Understand and apply stealth scanning techniques

---

## Prerequisites

- Completed Practical 5 (Vulnerability Scanning)
- Kali + Metasploitable 2 running on Host-Only network

---

## Tools Used

| Tool | What It Does |
|------|-------------|
| `nmap` | Network scanner - discovers hosts, ports, and services |
| `netcat` (`nc`) | Swiss army knife of networking - connects to ports, transfers data |
| `enum4linux` | SMB/Samba enumeration tool - extracts users, shares, and policies |
| `nbtscan` | Scans for NetBIOS name information |
| `smbclient` | Connects to SMB shares (like a command-line file explorer) |

---

## Real-World Context

After reconnaissance (gathering info from public sources), the next step is **active scanning** - directly probing the target to discover exactly what is running and how it is configured. This is where you build your attack map. Every open port is a potential entry point, and every service gives you more information to work with.

---

## Setup

```bash
export TARGET=192.168.56.101
ping -c 2 $TARGET
```

---

## Part 1: Advanced Nmap Scanning

### Step 1.1 - SYN Scan (Stealth Scan)

```bash
sudo nmap -sS $TARGET
```

The SYN scan is called "stealth" because it never completes the TCP handshake:

1. Your machine sends **SYN**
2. If the port is open, target responds with **SYN-ACK**
3. Instead of completing with ACK, Nmap sends **RST** (reset)

Because the connection is never fully established, many older logging systems do not record it.

> **Why this matters:** During real penetration tests, you may want to avoid detection by the target's intrusion detection system (IDS). SYN scans are harder to detect than full connect scans.

### Step 1.2 - TCP Connect Scan

```bash
nmap -sT $TARGET
```

This completes the full TCP three-way handshake. It is more reliable but also more easily detected and logged.

**When to use each:**

| Scan Type | Command | Speed | Stealth | Reliability |
|-----------|---------|-------|---------|-------------|
| SYN Scan | `-sS` | Fast | Higher | High |
| Connect Scan | `-sT` | Slower | Lower | Highest |

### Step 1.3 - UDP Scan

```bash
sudo nmap -sU --top-ports 20 $TARGET
```

UDP scanning is much slower than TCP because UDP does not send acknowledgments. We limit to top 20 ports to save time.

**Why UDP matters:** Many important services use UDP:

| Service | Port | Why It Matters |
|---------|------|---------------|
| DNS | 53 | DNS poisoning, zone transfers |
| SNMP | 161 | Often has default community strings, reveals system info |
| TFTP | 69 | No authentication, file uploads possible |
| NTP | 123 | NTP amplification DDoS attacks |

### Step 1.4 - Comprehensive Scan (All Ports)

```bash
sudo nmap -sS -sV -O -p- --open $TARGET
```

**Flags:**

| Flag | Meaning |
|------|---------|
| `-sS` | SYN scan |
| `-sV` | Detect service versions |
| `-O` | Detect operating system |
| `-p-` | Scan ALL 65,535 ports (not just top 1000) |
| `--open` | Only show open ports |

This scan takes longer but finds services running on non-standard ports - a common hiding technique.

### Step 1.5 - Scan Specific Port Ranges

```bash
# Scan only common web ports
nmap -p 80,443,8080,8443 $TARGET

# Scan a range
nmap -p 1-1024 $TARGET

# Scan top N ports
nmap --top-ports 100 $TARGET
```

### Step 1.6 - Output Results to a File

Always save your scan results:

```bash
# Save as normal text
nmap -sV $TARGET -oN ~/cyber-labs/nmap-scan.txt

# Save as XML (can be imported into other tools)
nmap -sV $TARGET -oX ~/cyber-labs/nmap-scan.xml

# Save in all formats
nmap -sV $TARGET -oA ~/cyber-labs/nmap-full
```

---

## Part 2: Banner Grabbing with Netcat

Banner grabbing is connecting to a service and reading the initial response. Services often announce their name and version - useful information for finding exploits.

### Step 2.1 - Grab the FTP Banner

```bash
nc $TARGET 21
```

**Expected output:**

```
220 (vsFTPd 2.3.4)
```

The FTP server just told you exactly what software and version it is running. Type `QUIT` to disconnect.

### Step 2.2 - Grab the SSH Banner

```bash
nc $TARGET 22
```

**Expected output:**

```
SSH-2.0-OpenSSH_4.7p1 Debian-8ubuntu1
```

Now you know the exact SSH version and the operating system.

### Step 2.3 - Grab the SMTP Banner

```bash
nc $TARGET 25
```

**Expected output:**

```
220 metasploitable.localdomain ESMTP Postfix (Ubuntu)
```

### Step 2.4 - Grab the HTTP Banner

```bash
nc $TARGET 80
```

Then type:

```
HEAD / HTTP/1.0

```

(Press Enter twice after the last line.)

**Expected output:**

```
HTTP/1.1 200 OK
Server: Apache/2.2.8 (Ubuntu)
```

### Step 2.5 - Automated Banner Grabbing with Nmap

```bash
nmap -sV --script banner -p 21,22,25,80 $TARGET
```

This does what you just did manually, but for multiple ports at once.

> **Why this matters:** Banners reveal exactly what version of software is running. With that version number, you can search for known exploits. Many organizations disable or modify banners to prevent this (banner hardening), but many forget.

---

## Part 3: SMB Enumeration

SMB (Server Message Block) is used for file sharing on networks. It is one of the most commonly exploited services because it often leaks sensitive information.

### Step 3.1 - Enumerate with enum4linux

```bash
enum4linux -a $TARGET
```

The `-a` flag runs ALL enumeration checks. This will take a moment.

**What enum4linux discovers:**

| Information | Why It Matters |
|-------------|---------------|
| Usernames | Targets for brute-force attacks |
| Share names | Files you might access |
| Password policies | How strong passwords must be |
| Group memberships | Who has admin access |
| OS information | What operating system and version |

### Step 3.2 - List SMB Shares

```bash
smbclient -L //$TARGET -N
```

The `-N` flag means no password (try anonymous access).

**Expected output:**

```
Sharename    Type   Comment
---------    ----   -------
print$     Disk   Printer Drivers
tmp       Disk   oh nance!
opt       Disk
IPC$      IPC    IPC Service
```

### Step 3.3 - Connect to a Share

```bash
smbclient //$TARGET/tmp -N
```

If this connects, you are now browsing files on the target system without any credentials. Try:

```
smb: \> ls
smb: \> pwd
smb: \> exit
```

### Step 3.4 - NetBIOS Scanning

```bash
nbtscan $TARGET
```

This reveals the machine's NetBIOS name, workgroup, and MAC address.

> **Why this matters:** SMB enumeration is critical in corporate network penetration testing. Windows networks heavily rely on SMB, and misconfigurations are extremely common. Finding accessible shares with sensitive data is one of the most frequent penetration test findings.

---

## Part 4: Putting It All Together - Building an Attack Map

### Step 4.1 - Create Your Attack Map

Based on everything you have scanned and enumerated, create a comprehensive map:

```bash
cat > ~/cyber-labs/attack-map.md << 'EOF'
# Attack Map - Metasploitable 2

## Target: [IP Address]
## Scan Date: [date]

## Open Ports and Services

| Port | Service | Version | Potential Attack |
|------|---------|---------|-----------------|
| 21 | FTP | vsftpd 2.3.4 | Known backdoor exploit |
| 22 | SSH | OpenSSH 4.7p1 | Brute force, old version |
| 23 | Telnet | Linux telnetd | Credentials sent in plaintext |
| 25 | SMTP | Postfix | User enumeration |
| 80 | HTTP | Apache 2.2.8 | Web app vulnerabilities |
| 139 | NetBIOS | Samba 3.x | SMB enumeration, share access |
| 445 | SMB | Samba 3.x | EternalBlue-type attacks |
| 3306 | MySQL | MySQL 5.0 | Default credentials, injection |
| 5432 | PostgreSQL | PostgreSQL 8.3 | Default credentials |

## Users Discovered
- [list from enum4linux]

## Shares Accessible
- [list from smbclient]

## Priority Targets (What to attack first)
1. [highest risk service]
2. [second highest]
3. [third]

## Attack Strategy
[Brief plan for exploitation phase - which services will you target and why?]
EOF

nano ~/cyber-labs/attack-map.md
```

### Step 4.2 - Fill In Your Map

Go through all your scan results and complete the attack map. This document will be your guide for the exploitation practicals in the coming weeks.

---

## Challenge Section (Optional)

1. **Scan for specific NSE scripts:**
  ```bash
  ls /usr/share/nmap/scripts/ | grep smb
  ```
  Run 3 SMB-specific scripts and record what they find.

2. **Try OS fingerprinting** - Research how Nmap detects operating systems. What TCP/IP stack characteristics does it look at?

3. **Stealth challenge** - Research and try the following Nmap evasion techniques:
  ```bash
  # Fragment packets
  nmap -f $TARGET
  
  # Use decoys (makes it look like multiple IPs are scanning)
  nmap -D RND:5 $TARGET
  
  # Slow scan (timing template)
  nmap -T1 $TARGET
  ```

---

## Summary & Outcomes

| Skill | Status |
|-------|--------|
| Perform SYN, Connect, and UDP scans | Yes |
| Scan all 65,535 ports | Yes |
| Grab service banners with Netcat | Yes |
| Enumerate SMB shares and users | Yes |
| Build a comprehensive attack map | Yes |

---

## Reflection Questions

1. Why is a SYN scan called "stealth"? Can it still be detected by modern security tools?
2. You discovered that anonymous SMB access is allowed on a corporate server. How would you rate this finding and why?
3. What is the difference between a port being "closed" and "filtered" in Nmap results?
4. Why should you always save scan results to files instead of just reading the terminal output?
5. A server has 3 open ports. Another server has 30 open ports. Which one is likely more vulnerable and why?
