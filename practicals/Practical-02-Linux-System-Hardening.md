# Practical 2: Linux System Hardening

**Atiam College - Cyber Security & Ethical Hacking**
**Module:** Week 4 - System Security
**Duration:** 2–3 Hours
**Difficulty:** Beginner–Intermediate

---

## Objective

By the end of this practical, you will be able to:

- Create and manage Linux user accounts with proper permissions
- Understand and configure file permissions (read, write, execute)
- Secure the SSH service against common attacks
- Install and configure fail2ban to block brute-force attempts
- Audit and disable unnecessary services

---

## Prerequisites

- Completed Practical 1 (Networking Fundamentals)
- Kali Linux VM running in VMware
- Basic terminal comfort

---

## Tools Used

| Tool | What It Does |
|------|-------------|
| `useradd` / `adduser` | Creates new user accounts |
| `passwd` | Sets or changes user passwords |
| `chmod` / `chown` | Changes file permissions and ownership |
| `ssh` / `sshd_config` | Secure Shell - remote access service and its configuration |
| `fail2ban` | Automatically blocks IPs that fail login too many times |
| `systemctl` | Manages running services |
| `ss` | Shows listening network ports and services |

---

## Real-World Context

Most cyber attacks succeed because systems are poorly configured, not because of some advanced zero-day exploit. Default passwords, unnecessary services running, files with wrong permissions - these are the easy wins for attackers. **System hardening is how you take those easy wins away.** Every server you deploy in the real world must be hardened before going live.

---

## Part 1: User Account Management

### Step 1.1 - See Who You Are

```bash
whoami
```

This shows your current username. On Kali, you are likely `root` or `kali`.

```bash
id
```

This shows your user ID (UID), group ID (GID), and group memberships. Root always has UID 0.

### Step 1.2 - List All Users on the System

```bash
cat /etc/passwd
```

Each line is a user. The format is:

```
username:x:UID:GID:comment:home_directory:shell
```

Real users typically have UIDs above 1000. System accounts (like `www-data`, `nobody`) have lower UIDs.

> **Why this matters:** During a penetration test, one of the first things you do after gaining access is enumerate users. Each user is a potential target for privilege escalation.

### Step 1.3 - Create a New User

Let's create a standard user (not an admin):

```bash
sudo adduser analyst1
```

You will be prompted to set a password and fill in optional details (you can press Enter to skip the optional ones).

Now create a second user:

```bash
sudo adduser analyst2
```

### Step 1.4 - Switch Between Users

```bash
su - analyst1
```

Enter the password you set. Notice the prompt changes - you are now `analyst1`.

Try accessing root's files:

```bash
cat /etc/shadow
```

**Expected result:** Permission denied. This file contains password hashes and only root can read it. This is proper access control.

Go back to your original user:

```bash
exit
```

### Step 1.5 - Give a User sudo Access

Sometimes a user needs admin privileges. Add `analyst1` to the sudo group:

```bash
sudo usermod -aG sudo analyst1
```

Now `analyst1` can use `sudo` to run commands as root.

> **WARNING:** **Real-world warning:** Never give sudo access unless absolutely necessary. The principle of **least privilege** - give users only the minimum permissions they need. Every sudo user is a potential escalation path for an attacker.

---

## Part 2: File Permissions Deep Dive

### Step 2.1 - Create Test Files

```bash
mkdir /tmp/perm-lab
cd /tmp/perm-lab

echo "This is a public file" > public.txt
echo "This is a secret file" > secret.txt
echo "#!/bin/bash
echo 'Hello from script'" > myscript.sh
```

### Step 2.2 - View Permissions

```bash
ls -la
```

You will see output like:

```
-rw-r--r-- 1 root root 22 Jun 8 10:00 public.txt
-rw-r--r-- 1 root root 23 Jun 8 10:00 secret.txt
-rw-r--r-- 1 root root 39 Jun 8 10:00 myscript.sh
```

**Reading the permission string `-rw-r--r--`:**

| Position | Meaning |
|----------|---------|
| `-` | File type (- = file, d = directory) |
| `rw-` | Owner permissions: read + write |
| `r--` | Group permissions: read only |
| `r--` | Others permissions: read only |

### Step 2.3 - Make a File Private (Owner Only)

```bash
chmod 600 secret.txt
ls -la secret.txt
```

Output: `-rw-------` - Only the owner can read and write. Nobody else can even see the contents.

**Understanding the numbers:**

| Number | Permission |
|--------|-----------|
| 0 | No permission |
| 1 | Execute only |
| 2 | Write only |
| 4 | Read only |
| 5 | Read + Execute (4+1) |
| 6 | Read + Write (4+2) |
| 7 | Read + Write + Execute (4+2+1) |

So `chmod 600` = Owner: read+write (6), Group: nothing (0), Others: nothing (0).

### Step 2.4 - Make a Script Executable

Try running the script:

```bash
./myscript.sh
```

**Expected result:** Permission denied - the file has no execute permission.

Fix it:

```bash
chmod 755 myscript.sh
./myscript.sh
```

Now it runs. `755` means: Owner can do everything (7), Group and Others can read and execute (5).

### Step 2.5 - Change File Ownership

```bash
chown analyst1:analyst1 secret.txt
ls -la secret.txt
```

Now `analyst1` owns the file. Combined with `chmod 600`, only `analyst1` can access it - even other users with sudo would need to explicitly use sudo to read it.

### Step 2.6 - The Dangerous 777

```bash
chmod 777 public.txt
ls -la public.txt
```

Output: `-rwxrwxrwx` - **Everyone can read, write, and execute.** This is the most dangerous permission setting.

> **WARNING:** **Never use 777 in production.** If you find files with 777 permissions during a penetration test, that is a finding you must report. Attackers look for world-writable files to modify scripts, plant backdoors, or escalate privileges.

---

## Part 3: Securing SSH

SSH is how administrators remotely access Linux servers. It is also the number one target for brute-force attacks.

### Step 3.1 - Check if SSH is Running

```bash
sudo systemctl status ssh
```

If it is not running:

```bash
sudo systemctl start ssh
```

### Step 3.2 - View the Current SSH Configuration

```bash
sudo cat /etc/ssh/sshd_config
```

This is the main configuration file. Let's make it more secure.

### Step 3.3 - Create a Backup First

Always back up configuration files before changing them:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

### Step 3.4 - Disable Root Login via SSH

Edit the SSH configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Find the line `#PermitRootLogin` and change it to:

```
PermitRootLogin no
```

(Remove the `#` if present - `#` means the line is commented out/disabled.)

> **Why this matters:** If root can log in via SSH, an attacker only needs to guess one password to get full control. By disabling root login, attackers must first guess a username AND a password, then find a way to escalate to root.

### Step 3.5 - Change the Default SSH Port

Still in the same file, find `#Port 22` and change it to:

```
Port 2222
```

This does not make SSH more secure by itself, but it stops most automated bots that only target port 22.

### Step 3.6 - Disable Password Authentication (Optional but Recommended)

If you want maximum security, you can disable passwords entirely and use SSH keys only:

```
PasswordAuthentication no
```

> **WARNING:** **Only do this if you have already set up SSH keys.** Otherwise you will lock yourself out. For this lab, leave password authentication enabled.

### Step 3.7 - Apply Changes

```bash
sudo systemctl restart ssh
```

### Step 3.8 - Test the New Configuration

From another terminal, try connecting:

```bash
ssh root@localhost -p 2222
```

This should be rejected because we disabled root login. Now try with a regular user:

```bash
ssh analyst1@localhost -p 2222
```

This should work (enter the password you set earlier).

### Step 3.9 - Restore Original Config When Done

```bash
sudo cp /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
sudo systemctl restart ssh
```

---

## Part 4: Blocking Brute Force with fail2ban

fail2ban monitors log files and automatically bans IPs that show malicious behavior (like too many failed login attempts).

### Step 4.1 - Install fail2ban

```bash
sudo apt update
sudo apt install fail2ban -y
```

### Step 4.2 - Configure fail2ban for SSH

Create a local configuration file (never edit the main config directly):

```bash
sudo nano /etc/fail2ban/jail.local
```

Add the following content:

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

**What this means:**

| Setting | Meaning |
|---------|---------|
| `maxretry = 3` | After 3 failed login attempts... |
| `bantime = 600` | ...ban the IP for 600 seconds (10 minutes) |
| `findtime = 600` | Count failures within a 10-minute window |

### Step 4.3 - Start fail2ban

```bash
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

### Step 4.4 - Test It

Deliberately fail SSH logins multiple times:

```bash
ssh analyst1@localhost
# Enter wrong password 3 times
```

Then check the fail2ban status:

```bash
sudo fail2ban-client status sshd
```

You should see your IP in the banned list.

### Step 4.5 - Unban an IP

```bash
sudo fail2ban-client set sshd unbanip 127.0.0.1
```

> **Why this matters:** Brute-force attacks against SSH are extremely common on the internet. Any server with SSH open to the internet receives thousands of login attempts per day. fail2ban is a simple, effective defense that every server should have.

---

## Part 5: Auditing Running Services

### Step 5.1 - List All Listening Ports

```bash
sudo ss -tulnp
```

**What the flags mean:**

| Flag | Meaning |
|------|---------|
| `-t` | Show TCP connections |
| `-u` | Show UDP connections |
| `-l` | Show only listening (waiting for connections) |
| `-n` | Show port numbers instead of service names |
| `-p` | Show which program is using the port |

### Step 5.2 - Identify Unnecessary Services

For each listening port, ask yourself:

- **Do I know what this service is?** If not, research it.
- **Do I need this service?** If not, disable it.
- **Is this service up to date?** Old versions may have known vulnerabilities.

### Step 5.3 - Disable an Unnecessary Service

For example, if Apache web server is running but you don't need it:

```bash
sudo systemctl stop apache2
sudo systemctl disable apache2
```

`stop` shuts it down now. `disable` prevents it from starting automatically on boot.

### Step 5.4 - Verify It's Gone

```bash
sudo ss -tulnp | grep 80
```

Port 80 should no longer appear.

> **Why this matters:** Every running service is a potential attack surface. The fewer services running, the fewer ways an attacker can get in. This is called **reducing the attack surface** - a core principle of system security.

---

## Challenge Section (Optional)

1. **Create a restricted user** that can only access their home directory and nothing else. Research `rbash` (restricted bash).
2. **Find all SUID files** on the system with `find / -perm -u=s -type f 2>/dev/null`. Research why SUID files are a security risk.
3. **Set up SSH key-based authentication** between two user accounts on your VM. Disable password authentication and verify only key auth works.

---

## Summary & Outcomes

| Skill | Status |
|-------|--------|
| Create and manage Linux user accounts | Yes |
| Configure file permissions (chmod, chown) | Yes |
| Harden SSH configuration | Yes |
| Install and configure fail2ban | Yes |
| Audit and disable unnecessary services | Yes |

---

## Reflection Questions

1. What is the principle of least privilege and why is it important?
2. A file has permissions `644`. Who can read it? Who can write to it? Who can execute it?
3. Why should you change the default SSH port, even though it does not add real security?
4. If you find a web server running on a machine that is not supposed to be a web server, what could this indicate?
5. How does fail2ban know that someone is trying to brute-force SSH? What log file does it monitor?
