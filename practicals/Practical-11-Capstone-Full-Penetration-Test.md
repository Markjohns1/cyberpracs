# Practical 11: Capstone - Full Penetration Test

**Atiam College - Cyber Security & Ethical Hacking**
**Module:** Week 13 - Capstone Project
**Duration:** 4–6 Hours (Self-paced over a week)
**Difficulty:** Advanced (Final Exam Practical)

---

## Objective

By the end of this practical, you will be able to:

- Perform a full-cycle penetration test independently on a target system
- Apply reconnaissance, network scanning, vulnerability assessment, exploitation, and post-exploitation techniques
- Document your findings in a structured, professional-grade penetration testing report
- Deliver clear remediation recommendations to secure a vulnerable system

---

## Prerequisites

- Completed Practicals 1 to 10
- Kali Linux VM and Metasploitable 2 VM running on Host-Only network
- Standard word processor or Markdown editor for drafting the final report

---

## Real-World Context

A penetration test is not just about typing exploit commands. It is a structured, professional process. When companies hire a penetration tester, they pay for the **report**, not the hack. The report details the vulnerabilities found, the business risk, and the instructions on how to patch them.

This capstone project is your chance to put together everything you have learned over the last 13 weeks. You will act as an external security consultant hired by "Atiam Corp" to assess their web server (represented by Metasploitable 2).

---

## The Engagement Scenario

- **Client:** Atiam Corp
- **Scope:** One IP Address (your Metasploitable 2 virtual machine)
- **Goal:** Identify security weaknesses, exploit them to demonstrate risk (up to gaining root access), and write a formal report.
- **Allowed Methods:** Active scanning, web vulnerability assessment, database exploitation, system exploitation, privilege escalation testing.
- **Forbidden Methods:** Denial of Service (DoS) attacks, brute-forcing external networks.

---

## Part 1: The Penetration Testing Cycle

You must execute the 5 phases of hacking in order:

```
 +------------------------+
 | 1. Reconnaissance   | <- Gather information without active scans
 +-----------+------------+
       v
 +------------------------+
 | 2. Scanning & Enum   | <- Nmap, port sweeps, version detection, banners
 +-----------+------------+
       v
 +------------------------+
 | 3. Vuln Assessment   | <- Analyze ports for known CVEs and weak software
 +-----------+------------+
       v
 +------------------------+
 | 4. Exploitation    | <- Gain access (Metasploit, web attacks)
 +-----------+------------+
       v
 +------------------------+
 | 5. Post-Exploitation  | <- Privilege escalation, file proof, clean up
 +------------------------+
```

### Phase 1: Reconnaissance (Passive)
- Document the environment setup (virtual networks, operating systems).
- State your rules of engagement: what boundaries did you set to prevent breaking your lab environment?

### Phase 2: Scanning & Enumeration (Active)
- Run network scans using Nmap. Find all open ports, active services, and operating system versions.
- Run banner grabbing against critical services to verify versions.
- Record the raw output of your scans.

### Phase 3: Vulnerability Analysis
- Research the services you found.
- Use `searchsploit` or search online databases (like CVE Details or Exploit-DB) for vulnerabilities matching the software versions you detected.
- Highlight at least **three (3) distinct vulnerabilities** that have high or critical severity.

### Phase 4: Exploitation
- Attempt to exploit the vulnerabilities you identified.
- You must demonstrate at least **two (2) successful compromises** of the system:
 1. One network-based exploit (e.g., FTP backdoor, Samba exploit, or IRC backdoor).
 2. One web-based exploit (e.g., SQL Injection extracting credentials, or Command Injection).
- Document each step with terminal outputs.

### Phase 5: Post-Exploitation
- Once inside, check your user privileges.
- Identify at least one pathway you could use to escalate privileges to `root` (e.g., SUID binaries, misconfigured files).
- Take a screenshot or copy the output of the `id` command showing you have achieved root access.
- List the proof of compromise (e.g., reading a file only root can access, like `/etc/shadow`).

---

## Part 2: Writing the Report

You must write a professional-grade report. Use the template below to draft your document in Markdown or save it as a PDF/Word document.

```markdown
# Penetration Test Report
**Client:** Atiam Corp 
**Assessor:** [Your Name] 
**Date:** [Date] 

---

## 1. Executive Summary
Provide a high-level summary of the assessment for non-technical managers.
- What was the overall security posture of the target system?
- What is the most critical vulnerability found?
- What are the top three recommendations to secure the target?

## 2. Scope & Methodology
Explain what was tested and how it was tested.
- Scope: [Target IP Address]
- Date of testing: [Date]
- Methodology: OSSTMM / OWASP / NIST standard cycle

## 3. Vulnerability Findings Table

| ID | Finding Name | Severity | Service & Port | CVSS / Impact |
|----|--------------|----------|----------------|---------------|
| 01 | [Vulnerability 1] | Critical | [e.g., FTP Port 21] | Remote Code Execution |
| 02 | [Vulnerability 2] | High   | [e.g., HTTP Port 80]| Database Compromise |
| 03 | [Vulnerability 3] | Medium  | [e.g., SSH Port 22] | Brute-force Risk |

*(Add more rows if necessary)*

---

## 4. Detailed Findings & Exploitation Walkthrough

### Finding 01: [Vulnerability Name]
- **Severity:** Critical
- **CVSS Score:** [e.g., 10.0]
- **Port/Service:** [e.g., Port 21 / vsftpd]
- **Description:** Provide a technical description of what the vulnerability is.
- **Exploitation Steps:**
 1. Explain step-by-step how you accessed it.
 2. Include code/command blocks showing the exploit execution.
 3. Show the proof of compromise (e.g., command output showing shell access).
- **Remediation:** Provide exact steps on how the system administrator can fix this vulnerability.

---

### Finding 02: [Vulnerability Name]
- **Severity:** High
- **Port/Service:** [e.g., Port 80 / Apache]
- **Description:** Technical description of the vulnerability.
- **Exploitation Steps:**
 1. Detail the attack path.
 2. Show command/script used.
 3. Show proof of execution (e.g., SQL database dump).
- **Remediation:** Actionable recommendations on how to patch or secure it.

---

## 5. Security Recommendations
Provide a roadmap for securing the server:
1. **Short-Term Actions (First 48 hours):** What needs to be patched immediately.
2. **Medium-Term Actions (Next 30 days):** Policy updates, service configurations, user access control.
3. **Long-Term Actions (Next 90 days):** System hardening, periodic vulnerability scanning, training.
```

---

## Part 3: Cleaning Up the Environment

When you complete the capstone test, you must leave the client's systems exactly as you found them.

- Remove any backdoor users you created during testing.
- Clean up the `/tmp` directory.
- Revert the Metasploitable virtual machine to its clean snapshot state in VMware.
- Verify that your host computer network settings are reverted to normal.

---

## Graduation Checklist

Congratulations! Once you submit your Penetration Test Report to your instructor, you have completed the practical segment of this course. You now possess hands-on skills in:

- [ ] Core Network Analysis & Traffic Monitoring
- [ ] Operating System Hardening & Configuration
- [ ] Open Source Intelligence Gathering (OSINT)
- [ ] Active Network Profiling & Scanning
- [ ] Exploitation using Professional Frameworks
- [ ] Web Application Testing & OWASP Mitigation
- [ ] Post-Exploitation, Privilege Escalation, and Incident Response
- [ ] Professional security reporting
