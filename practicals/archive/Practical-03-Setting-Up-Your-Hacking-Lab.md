# Practical 3: Setting Up Your Hacking Lab

**Atiam College - Cyber Security & Ethical Hacking**
**Module:** Week 5 - Introduction to Ethical Hacking
**Duration:** 2–3 Hours
**Difficulty:** Beginner

---

## Objective

By the end of this practical, you will be able to:

- Navigate Kali Linux confidently and locate key security tools
- Understand VMware networking modes and configure your lab network
- Download and set up Metasploitable 2 as a vulnerable target machine
- Verify connectivity between your Kali attacker and Metasploitable target
- Understand the legal and ethical boundaries of ethical hacking

---

## Prerequisites

- VMware installed on your Windows host machine
- Kali Linux VM already set up
- At least 2GB of free RAM (for running two VMs simultaneously)
- Internet connection (for downloading Metasploitable 2)

---

## Tools Used

| Tool | What It Does |
|------|-------------|
| VMware | Virtualization software - runs virtual machines |
| Kali Linux | Attacker machine - comes with 600+ security tools pre-installed |
| Metasploitable 2 | Intentionally vulnerable Linux machine - your practice target |
| Terminal / Bash | Command-line interface for interacting with Linux |

---

## Real-World Context

Professional penetration testers never practice on live systems without authorization. They build lab environments that simulate real networks. **Your lab is your training ground.** Every skill you develop here - scanning, exploiting, reporting - is the same skill you will use on real engagements, but in a safe, legal environment.

> **WARNING:** **CRITICAL LEGAL WARNING:** Everything you learn in this course must ONLY be used on systems you have explicit written permission to test. Unauthorized access to computer systems is a criminal offense under the Computer Misuse and Cybercrimes Act (Kenya, 2018) and similar laws worldwide. Your lab environment is the ONLY place where you can freely practice.

---

## Part 1: Getting to Know Kali Linux

### Step 1.1 - Explore the Kali Menu

Click the **Applications** menu in the top-left corner of Kali. You will see categories:

| Category | What It Contains |
|----------|-----------------|
| 01 - Information Gathering | Recon tools (Nmap, theHarvester, Maltego) |
| 02 - Vulnerability Analysis | Scanners (Nessus, OpenVAS, Nikto) |
| 03 - Web Application Analysis | Web testing (Burp Suite, OWASP ZAP, SQLMap) |
| 04 - Database Assessment | Database attack tools |
| 05 - Password Attacks | Cracking tools (John the Ripper, Hashcat, Hydra) |
| 06 - Wireless Attacks | WiFi testing (Aircrack-ng) |
| 07 - Reverse Engineering | Malware analysis tools (Ghidra) |
| 08 - Exploitation Tools | Metasploit Framework, exploits |
| 09 - Sniffing & Spoofing | Traffic interception (Wireshark, Ettercap) |
| 10 - Post Exploitation | Maintaining access tools |

> **Don't be overwhelmed.** You will not use all of these. We will focus on the most important and commonly used tools throughout this course.

### Step 1.2 - Essential Terminal Commands

Open a terminal (`Ctrl+Alt+T`) and practice these commands:

```bash
# Where am I?
pwd

# What's in this directory?
ls -la

# Go to the home directory
cd ~

# Create a working directory for this course
mkdir ~/cyber-labs
cd ~/cyber-labs

# Create a notes file
echo "Cyber Security Lab - Started $(date)" > lab-notes.txt
cat lab-notes.txt

# Check disk space
df -h

# Check memory usage
free -h

# Check running processes
ps aux | head -20

# Find a file
locate nmap

# Update the file database (for locate to work)
sudo updatedb
```

### Step 1.3 - Check What Tools Are Pre-installed

```bash
# Check if Nmap is installed
nmap --version

# Check if Metasploit is installed
msfconsole --version

# Check if Wireshark is installed
wireshark --version

# Check if Burp Suite is available
which burpsuite

# Check if Hydra is installed
hydra -h 2>&1 | head -5
```

All of these should be pre-installed on Kali. If any are missing:

```bash
sudo apt update
sudo apt install <tool-name> -y
```

---

## Part 2: Understanding VMware Networking

This is critical for your lab to work. You need your Kali and Metasploitable VMs to communicate.

### Step 2.1 - VMware Network Modes Explained

VMware offers three main networking modes:

| Mode | How It Works | Use Case |
|------|-------------|----------|
| **NAT** | VMs share the host's IP, can access internet | Default mode, good for internet access |
| **Bridged** | VMs get their own IP on your physical network | Makes VMs visible on your real network |
| **Host-Only** | VMs can only talk to each other and the host | **Best for our lab - isolated and safe** |

### Step 2.2 - Configure Host-Only Networking

We will use **Host-Only** networking. This keeps your attack traffic isolated - it never leaves your computer.

**In VMware:**

1. Go to **Edit** → **Virtual Network Editor**
2. Look for **VMnet1** (Host-Only)
3. Note the subnet - it will be something like `192.168.56.0/24`
4. Make sure **DHCP** is enabled (so VMs get IPs automatically)

**On your Kali VM:**

1. Shut down the Kali VM
2. Go to **VM** → **Settings** → **Network Adapter**
3. Select **Host-Only**
4. Start the Kali VM

After booting, verify:

```bash
ip addr show
```

You should see an IP in the Host-Only range (e.g., `192.168.56.x`).

> **Why Host-Only?** When you start attacking Metasploitable, you do not want that traffic going out on your real network. Host-Only keeps everything contained. This is a best practice for any penetration testing lab.

---

## Part 3: Setting Up Metasploitable 2

Metasploitable 2 is a purposely vulnerable Linux virtual machine. It has dozens of known vulnerabilities - perfect for practicing.

### Step 3.1 - Download Metasploitable 2

Download from SourceForge (it is free):

```
https://sourceforge.net/projects/metasploitable/
```

The download is a `.zip` file (~800MB). Extract it - you will get a `.vmdk` file (virtual hard drive).

### Step 3.2 - Create a New VM in VMware

1. In VMware, go to **File** → **New Virtual Machine**
2. Select **Custom** → **Next**
3. Choose **I will install the operating system later** → **Next**
4. Guest OS: **Linux**, Version: **Ubuntu** (or Other Linux) → **Next**
5. Name it **Metasploitable2** → **Next**
6. Memory: **512MB** is enough → **Next**
7. Network: **Host-Only** (same as Kali) → **Next**
8. For the disk, select **Use an existing virtual disk** and browse to the `.vmdk` file you extracted
9. Finish the wizard

### Step 3.3 - Boot Metasploitable 2

Start the VM. You will see a login screen:

```
metasploitable login: msfadmin
Password: msfadmin
```

Yes, the credentials are `msfadmin` / `msfadmin`. This is intentionally insecure.

### Step 3.4 - Find Metasploitable's IP Address

After logging in:

```bash
ifconfig
```

Note the IP address (e.g., `192.168.56.101`). You will need this for all future practicals.

---

## Part 4: Verify Lab Connectivity

### Step 4.1 - Ping from Kali to Metasploitable

On your **Kali** VM:

```bash
ping -c 4 192.168.56.101
```

(Replace with Metasploitable's actual IP.)

**Expected:** You should receive replies. If not, both VMs need to be on the same Host-Only network.

### Step 4.2 - Ping from Metasploitable to Kali

On your **Metasploitable** VM:

```bash
ping -c 4 192.168.56.100
```

(Replace with Kali's actual IP.)

### Step 4.3 - Quick Nmap Scan to See What's Open

From **Kali**, run a fast scan against Metasploitable:

```bash
nmap 192.168.56.101
```

**Expected output (similar to):**

```
PORT   STATE SERVICE
21/tcp  open ftp
22/tcp  open ssh
23/tcp  open telnet
25/tcp  open smtp
80/tcp  open http
139/tcp open netbios-ssn
445/tcp open microsoft-ds
3306/tcp open mysql
5432/tcp open postgresql
```

Look at all those open ports. **This machine is wide open.** Each of those services is a potential way in. Over the coming weeks, you will learn to exploit many of them.

> **Why this matters:** In the real world, your first task as a penetration tester is to discover what services are running. This quick Nmap scan just showed you exactly what Metasploitable is exposing. In a real engagement, this would be your roadmap for the entire test.

### Step 4.4 - Document Your Lab Setup

Create a reference file you can use throughout the course:

```bash
cat > ~/cyber-labs/lab-info.txt << EOF
=== CYBER SECURITY LAB SETUP ===
Date: $(date)

ATTACKER (Kali Linux):
 IP: $(hostname -I | awk '{print $1}')
 
TARGET (Metasploitable 2):
 IP: 192.168.56.101
 Login: msfadmin / msfadmin

NETWORK: Host-Only (VMnet1)
SUBNET: 192.168.56.0/24
EOF

cat ~/cyber-labs/lab-info.txt
```

Update the Metasploitable IP to match your actual setup.

---

## Part 5: Lab Ethics and Rules of Engagement

### Step 5.1 - Read and Understand

Before any engagement (even in a lab), professionals define **Rules of Engagement (RoE)**. These specify:

1. **What you CAN test** - Only Metasploitable 2. Never your host machine, the college network, or any external system.
2. **What you CANNOT do** - Do not attack any machine outside your Host-Only network. Do not use techniques against real systems without written permission.
3. **When you can test** - During lab hours, or on your own machine at home.
4. **Reporting** - Document everything you find. Professional pentesters write reports, they don't just hack and walk away.

### Step 5.2 - The Legal Framework (Kenya)

The **Computer Misuse and Cybercrimes Act, 2018** makes it a criminal offense to:

- Access a computer system without authorization (Section 14)
- Illegally intercept data (Section 15)
- Illegally interfere with computer systems (Section 16)

Penalties include fines up to KES 5 million and imprisonment up to 3 years.

> **WARNING:** **The only thing separating an ethical hacker from a criminal is PERMISSION.** Always have written authorization before testing any system.

---

## Challenge Section (Optional)

1. **Explore Metasploitable's web interface** - Open a browser on Kali and visit `http://<metasploitable-ip>`. Look around at the web applications running. You will attack these in Week 10.
2. **Research three of the open ports** you found in the Nmap scan. What service runs on each? What known vulnerabilities exist?
3. **Take a snapshot** of both VMs in VMware (VM → Snapshot → Take Snapshot). This gives you a clean state to restore if you break something during future practicals.

---

## Summary & Outcomes

| Skill | Status |
|-------|--------|
| Navigate Kali Linux and locate security tools | Yes |
| Understand VMware networking modes | Yes |
| Set up Metasploitable 2 as a target VM | Yes |
| Verify lab connectivity between attacker and target | Yes |
| Understand legal and ethical boundaries | Yes |

---

## Reflection Questions

1. Why do we use Host-Only networking for our lab instead of Bridged or NAT?
2. What are the risks of running attack tools on a Bridged network?
3. Metasploitable has many open ports. In a real-world scenario, how many ports should a properly hardened server have open?
4. What is the difference between a black-box and white-box penetration test?
5. Why is documentation and reporting important in ethical hacking?
