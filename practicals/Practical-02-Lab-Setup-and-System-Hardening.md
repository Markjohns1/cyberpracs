# Module 2: Lab Setup & System Hardening

**Atiam College - Cyber Security & Ethical Hacking**
**Module:** 2 of 5 — Build Your Lab & Secure Linux
**Duration:** 3–4 Hours
**Difficulty:** Beginner

---

## What You Will Do in This Module

This module combines two essential foundations:

1. **Part A — Lab Setup:** Install your attack/target environment (Kali + Metasploitable 2).
2. **Part B — System Hardening:** Lock down Linux users, permissions, SSH, and services.

> **Important:** Complete Part A before Part B. You need both VMs running before the rest of the course.

---

## Objectives

By the end of this module, you will be able to:

- Navigate Kali Linux and verify security tools are installed
- Configure VMware Host-Only networking and set up Metasploitable 2
- Verify connectivity between attacker (Kali) and target (Metasploitable)
- Create and manage Linux users with correct file permissions
- Harden SSH and install fail2ban against brute-force attacks
- Audit and disable unnecessary services

---

## Prerequisites

- VMware installed on your Windows host
- Kali Linux VM already created
- At least 2 GB free RAM (to run two VMs at once)
- **Completed Module 1** (Networking Fundamentals)

---

## Tools Used

| Tool | What It Does |
|------|-------------|
| VMware | Runs your virtual machines |
| Kali Linux | Attacker machine with security tools |
| Metasploitable 2 | Intentionally vulnerable practice target |
| `useradd` / `chmod` / `chown` | User and file permission management |
| `sshd_config` | SSH server configuration |
| `fail2ban` | Blocks IPs after repeated failed logins |
| `ss` / `systemctl` | View ports and manage services |

---

## Real-World Context

Professional penetration testers never practice on live systems without authorization. They build isolated labs first. Once the lab works, they also understand **hardening** — because most real breaches exploit weak configuration, not magic hacking skills.

> **WARNING — LEGAL:** Only use these techniques on systems you own or have **written permission** to test. Unauthorized access is a criminal offense (e.g. Kenya Computer Misuse and Cybercrimes Act, 2018).

---

# Part A: Setting Up Your Hacking Lab

## Step A.1 — Explore the Kali Menu

1. On Kali, click **Applications** (top-left).
2. Notice the tool categories (Information Gathering, Vulnerability Analysis, Exploitation, etc.).
3. You will not use all of them — we focus on the core tools over the next modules.

## Step A.2 — Practice Essential Terminal Commands

Open a terminal (`Ctrl+Alt+T`) and run each command below. Read the comment above each one so you know what it does.

```bash
# Show current directory
pwd

# List files (including hidden) with details
ls -la

# Go to your home directory
cd ~

# Create a folder for all course work
mkdir ~/cyber-labs
cd ~/cyber-labs

# Create a notes file with today's date
echo "Cyber Security Lab - Started $(date)" > lab-notes.txt
cat lab-notes.txt

# Check disk space and memory
df -h
free -h
```

**Expected:** Each command runs without errors. `lab-notes.txt` shows today's date.

## Step A.3 — Verify Security Tools Are Installed

```bash
nmap --version
msfconsole --version
wireshark --version
which burpsuite
hydra -h 2>&1 | head -5
```

**Expected:** Version numbers or help text for each tool.

**If a tool is missing:**

```bash
sudo apt update
sudo apt install <tool-name> -y
```

Replace `<tool-name>` with the package you need (e.g. `nmap`).

---

## Step A.4 — Understand VMware Network Modes

| Mode | How It Works | Our Use |
|------|-------------|---------|
| **NAT** | VM shares host internet via translation | Internet access only |
| **Bridged** | VM gets an IP on your real LAN | Not for attacks — traffic leaves your PC |
| **Host-Only** | VMs talk to each other + host only | **We use this — isolated and safe** |

**Why Host-Only:** Attack traffic stays on your computer. It cannot reach the college network or the internet targets.

---

## Step A.5 — Configure Host-Only Networking

**On VMware (while Kali is shut down):**

1. **Edit** → **Virtual Network Editor**
2. Select **VMnet1** (Host-Only)
3. Note the subnet (usually `192.168.56.0/24`)
4. Ensure **DHCP** is enabled

**On the Kali VM:**

1. Shut down Kali
2. **VM** → **Settings** → **Network Adapter** → **Host-Only**
3. Start Kali
4. In the terminal:

```bash
ip addr show
```

**Expected:** An IP like `192.168.56.x` on your active interface (often `eth0`).

**Write down your Kali IP** — you will need it later.

---

## Step A.6 — Download and Import Metasploitable 2

1. Download from: https://sourceforge.net/projects/metasploitable/
2. Extract the `.zip` — you get a `.vmdk` file (~800 MB download).

**Create the VM in VMware:**

1. **File** → **New Virtual Machine** → **Custom**
2. **I will install the operating system later**
3. Guest OS: **Linux** → **Ubuntu** (or Other Linux)
4. Name: **Metasploitable2**
5. RAM: **512 MB**
6. Network: **Host-Only** (same as Kali)
7. Disk: **Use an existing virtual disk** → select the `.vmdk`
8. Finish

---

## Step A.7 — Boot Metasploitable and Find Its IP

Start the Metasploitable VM. At the login prompt:

```
Login: msfadmin
Password: msfadmin
```

After login:

```bash
ifconfig
```

**Expected:** An IP in the same subnet as Kali (e.g. `192.168.56.101`).

**Write down the Metasploitable IP** — this is your `TARGET` for all future modules.

---

## Step A.8 — Verify Lab Connectivity

**On Kali** (replace `101` with your Metasploitable IP):

```bash
export TARGET=192.168.56.101
ping -c 4 $TARGET
```

**Expected:** Four replies (`64 bytes from ...`). If ping fails, both VMs must use Host-Only on the same VMnet.

**Quick port scan from Kali:**

```bash
nmap $TARGET
```

**Expected:** Many open ports (21, 22, 23, 80, 445, etc.). This confirms the target is reachable and vulnerable-looking.

---

## Step A.9 — Document Your Lab Setup

On Kali:

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

**Action required:** Edit the file if your Metasploitable IP is different:

```bash
nano ~/cyber-labs/lab-info.txt
```

---

## Step A.10 — Take VMware Snapshots

1. With both VMs running cleanly, in VMware: **VM** → **Snapshot** → **Take Snapshot**
2. Name them e.g. `clean-kali` and `clean-metasploitable`

**Why:** If you break something during attacks, you can restore to a clean state in one click.

---

## Step A.11 — Rules of Engagement (RoE)

Before any testing, professionals define:

| Rule | Our Lab |
|------|---------|
| **What you CAN test** | Only Metasploitable 2 on Host-Only |
| **What you CANNOT test** | College network, internet hosts, your Windows host |
| **Documentation** | Save all scan output and notes in `~/cyber-labs/` |

---

# Part B: Linux System Hardening

> **Where to run these steps:** On your **Kali** VM (unless stated otherwise). We harden Kali to practice defense skills.

---

## Step B.1 — Check Your Current User

```bash
whoami
id
```

**Expected:** `root` or `kali`. Note your UID — root is always `0`.

---

## Step B.2 — List System Users

```bash
cat /etc/passwd
```

Each line format: `username:x:UID:GID:comment:home:shell`

Real human users usually have UID ≥ 1000.

---

## Step B.3 — Create Practice Users

```bash
sudo adduser analyst1
```

Set a password when prompted (remember it). Skip optional fields with Enter.

```bash
sudo adduser analyst2
```

---

## Step B.4 — Test User Restrictions

Switch to the new user:

```bash
su - analyst1
```

Try reading the password file:

```bash
cat /etc/shadow
```

**Expected:** `Permission denied` — only root should read password hashes.

Return to your original user:

```bash
exit
```

---

## Step B.5 — File Permissions Practice

```bash
mkdir /tmp/perm-lab
cd /tmp/perm-lab

echo "public content" > public.txt
echo "secret content" > secret.txt
echo '#!/bin/bash
echo Hello from script' > myscript.sh

ls -la
```

**Read permissions** — example `-rw-r--r--`:

| Part | Meaning |
|------|---------|
| `rw-` | Owner: read + write |
| `r--` | Group: read only |
| `r--` | Others: read only |

Make the secret file private:

```bash
chmod 600 secret.txt
ls -la secret.txt
```

**Expected:** `-rw-------` — only the owner can read/write.

**Permission numbers:** 4=read, 2=write, 1=execute. So `600` = owner read+write (6), group nothing (0), others nothing (0).

Make the script executable and run it:

```bash
chmod 755 myscript.sh
./myscript.sh
```

**Expected:** `Hello from script`

> **WARNING:** Never use `chmod 777` in production. World-writable files are a common penetration test finding.

---

## Step B.6 — Check SSH Status

```bash
sudo systemctl status ssh
```

If not running:

```bash
sudo systemctl start ssh
sudo systemctl enable ssh
```

---

## Step B.7 — Back Up SSH Configuration

Always back up before editing:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

---

## Step B.8 — Harden SSH Settings

```bash
sudo nano /etc/ssh/sshd_config
```

Find and change these lines (remove `#` at the start if present):

```
PermitRootLogin no
Port 2222
```

**What this does:**

- `PermitRootLogin no` — attackers must guess a username AND password
- `Port 2222` — reduces automated bot noise on port 22 (not true security alone, but helps)

Save: `Ctrl+O`, Enter, then `Ctrl+X`.

Apply changes:

```bash
sudo systemctl restart ssh
```

---

## Step B.9 — Test SSH Hardening

Root login should fail:

```bash
ssh root@localhost -p 2222
```

**Expected:** Permission denied or connection refused for root.

Regular user should work:

```bash
ssh analyst1@localhost -p 2222
```

Enter `analyst1`'s password. Type `exit` to disconnect.

---

## Step B.10 — Restore SSH Config (End of Lab)

So future modules are not confused by port 2222:

```bash
sudo cp /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
sudo systemctl restart ssh
```

---

## Step B.11 — Install and Configure fail2ban

```bash
sudo apt update
sudo apt install fail2ban -y
```

Create local config (never edit the main file directly):

```bash
sudo nano /etc/fail2ban/jail.local
```

Paste this exactly:

```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 600
findtime = 600
```

| Setting | Meaning |
|---------|---------|
| `maxretry = 3` | Ban after 3 failed attempts |
| `bantime = 600` | Ban for 600 seconds (10 minutes) |
| `findtime = 600` | Count failures within 10 minutes |

Start fail2ban:

```bash
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

---

## Step B.12 — Test fail2ban

Deliberately fail login 3 times:

```bash
ssh analyst1@localhost
```

Enter a **wrong** password three times, then `Ctrl+C`.

Check ban status:

```bash
sudo fail2ban-client status sshd
```

**Expected:** Your IP may appear in the banned list.

Unban localhost if needed:

```bash
sudo fail2ban-client set sshd unbanip 127.0.0.1
```

---

## Step B.13 — Audit Listening Services

```bash
sudo ss -tulnp
```

For each listening port, ask:

1. Do I know what this service is?
2. Do I need it running?

Example — stop Apache if you do not need a web server on Kali:

```bash
sudo systemctl stop apache2
sudo systemctl disable apache2
```

Verify port 80 is gone:

```bash
sudo ss -tulnp | grep ':80'
```

**Expected:** No output (nothing listening on 80).

---

## Module 2 Checklist

Before moving to Module 3, confirm:

- [ ] Kali and Metasploitable both boot on Host-Only network
- [ ] `ping $TARGET` works from Kali
- [ ] `~/cyber-labs/lab-info.txt` has correct IPs
- [ ] VMware snapshots taken
- [ ] You understand `chmod` numbers and SSH hardening basics
- [ ] fail2ban is installed and running

---

## Summary & Outcomes

| Skill | Status |
|-------|--------|
| Set up Kali + Metasploitable lab on Host-Only network | Yes |
| Verify connectivity and document lab IPs | Yes |
| Manage users and file permissions | Yes |
| Harden SSH and use fail2ban | Yes |
| Audit and disable unnecessary services | Yes |

---

## Reflection Questions

1. Why is Host-Only networking safer than Bridged for a hacking lab?
2. What is the principle of least privilege?
3. A file has permissions `644`. Who can read, write, and execute it?
4. How does fail2ban know someone is brute-forcing SSH?
5. Why should penetration testers document Rules of Engagement before testing?
