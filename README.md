# Cyber Security Practicals — Atiam College

**5 consolidated modules** covering networking, lab setup, reconnaissance, exploitation, and incident response.

> **Deadline note:** If you are catching up before Thursday, follow the **Suggested Schedule** below. Module 1 is already complete for your class — start at Module 2.

---

## Module Overview

| Module | File | Topics (merged from original 11 practicals) | Time |
|--------|------|-----------------------------------------------|------|
| **1** | [Practical-01-Networking-Fundamentals.md](practicals/Practical-01-Networking-Fundamentals.md) | IP config, ping, Wireshark, iptables | 2–3 hrs |
| **2** | [Practical-02-Lab-Setup-and-System-Hardening.md](practicals/Practical-02-Lab-Setup-and-System-Hardening.md) | Kali lab, Metasploitable, VMware, Linux hardening, fail2ban | 3–4 hrs |
| **3** | [Practical-03-Recon-Scanning-and-Vulnerability-Assessment.md](practicals/Practical-03-Recon-Scanning-and-Vulnerability-Assessment.md) | OSINT, Nmap, SMB enum, NSE, Nikto, attack map | 3–4 hrs |
| **4** | [Practical-04-Exploitation-Web-Attacks-and-Post-Exploitation.md](practicals/Practical-04-Exploitation-Web-Attacks-and-Post-Exploitation.md) | Metasploit, DVWA SQLi/XSS, Burp, post-exploit | 4–5 hrs |
| **5** | [Practical-05-Incident-Response-and-Capstone.md](practicals/Practical-05-Incident-Response-and-Capstone.md) | Log forensics, IR timeline, full pentest report | 3–4 hrs |

**Total:** ~15–20 hours (down from ~30+ hours across 11 separate practicals).

---

## Suggested Schedule (Finish by Thursday)

| Day | Focus | Deliverable |
|-----|-------|-------------|
| **Day 1** | Module 2 — Lab must work before anything else | `~/cyber-labs/lab-info.txt` + VMware snapshots |
| **Day 2** | Module 3 — Scanning & vuln assessment | `attack-map.md` + `vulnerability-report.md` |
| **Day 3** | Module 4 — Exploitation & web attacks | Screenshots / notes of successful exploits |
| **Day 4** | Module 5 — IR exercise + capstone report | `incident-timeline.md` + `penetration-test-report.md` |

---

## How the Original 11 Practicals Were Combined

| New Module | Original Practical(s) |
|------------|----------------------|
| 1 | Practical 1 (unchanged) |
| 2 | Practical 2 + Practical 3 |
| 3 | Practical 4 + Practical 5 + Practical 6 |
| 4 | Practical 7 + Practical 8 + Practical 9 |
| 5 | Practical 10 + Practical 11 |

Original full-length files are preserved in [`practicals/archive/`](practicals/archive/) for reference.

---

## Before You Start Module 2

- [ ] VMware installed
- [ ] Kali Linux VM ready
- [ ] Module 1 complete
- [ ] At least 2 GB free RAM for two VMs

---

## Lab Quick Reference

```
Kali (attacker)     →  Host-Only network  →  Metasploitable 2 (target)
Default target IP   →  Often 192.168.56.101 (verify with ifconfig)
Target credentials  →  msfadmin / msfadmin
DVWA login          →  admin / password
```

Always set your target at the start of each module:

```bash
export TARGET=192.168.56.101   # use your real IP
ping -c 2 $TARGET
```

---

## Ethical Use

These practicals are for **authorized lab use only**. Never scan or attack systems without written permission.
