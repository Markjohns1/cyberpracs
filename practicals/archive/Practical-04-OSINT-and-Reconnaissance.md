# Practical 4: OSINT & Reconnaissance

**Atiam College - Cyber Security & Ethical Hacking**
**Module:** Week 6 - Footprinting and Reconnaissance
**Duration:** 2–3 Hours
**Difficulty:** Intermediate

---

## Objective

By the end of this practical, you will be able to:

- Perform passive reconnaissance without touching the target system
- Use WHOIS to find domain registration information
- Use DNS tools to map a target's infrastructure
- Use Google Dorks to find exposed sensitive information
- Use theHarvester to collect emails and subdomains
- Understand the difference between passive and active reconnaissance

---

## Prerequisites

- Kali Linux VM with internet access
- Completed Practicals 1–3

---

## Tools Used

| Tool | What It Does |
|------|-------------|
| `whois` | Looks up domain registration details (owner, registrar, dates) |
| `nslookup` / `dig` | Queries DNS servers for domain-to-IP mappings |
| `theHarvester` | Collects emails, subdomains, and IPs from public sources |
| Google Dorks | Advanced Google search operators to find exposed data |
| `host` | Simple DNS lookup tool |

---

## Real-World Context

Reconnaissance is the **first phase** of any penetration test and the most important phase for attackers. Before touching a target system, professionals gather as much information as possible from **public sources**. This is called OSINT - Open Source Intelligence. The information is freely available on the internet; you just need to know where to look.

A good recon phase can reveal employee emails (for phishing), server locations, technology stacks, and even forgotten test servers that are still exposed.

> **Passive reconnaissance is legal** because you are not interacting with the target system directly. You are only looking at publicly available information. However, always confirm this with your engagement's Rules of Engagement.

---

## Part 1: WHOIS Lookups

WHOIS is a public database that stores information about who registered a domain name.

### Step 1.1 - Look Up a Domain

We will use a public domain for practice. Let's use `example.com` (a domain specifically reserved for documentation and examples):

```bash
whois example.com
```

**What to look for in the output:**

| Field | What It Tells You |
|-------|------------------|
| Registrant | Who registered the domain |
| Admin Contact | Administrative contact person |
| Tech Contact | Technical contact person |
| Name Servers | DNS servers handling the domain |
| Creation Date | When the domain was first registered |
| Expiration Date | When it expires (forgotten domains can be hijacked) |
| Registrar | Which company the domain was registered through |

### Step 1.2 - Look Up Your College's Domain

```bash
whois atiamcollege.co.ke
```

> **Why this matters:** WHOIS reveals organizational structure, contact names (for social engineering), technology providers (for targeted attacks), and domain expiration dates (for domain hijacking). Many organizations now use WHOIS privacy protection to hide this information.

### Step 1.3 - Check Multiple Domains

Try looking up a few well-known domains:

```bash
whois google.com
whois facebook.com
whois safaricom.co.ke
```

Notice how large companies use privacy protection services while smaller organizations often expose their full details.

---

## Part 2: DNS Reconnaissance

DNS (Domain Name System) translates domain names to IP addresses. By querying DNS, you can discover a target's servers, mail servers, and infrastructure.

### Step 2.1 - Basic DNS Lookup

```bash
nslookup example.com
```

This shows the IP address(es) associated with the domain.

### Step 2.2 - Find Mail Servers (MX Records)

```bash
dig example.com MX
```

MX records tell you which servers handle email for a domain. This reveals:
- The email provider (Google Workspace, Microsoft 365, self-hosted)
- Potential targets for email-based attacks

### Step 2.3 - Find Name Servers (NS Records)

```bash
dig example.com NS
```

Name servers tell you who manages the domain's DNS. If an attacker can compromise the DNS provider, they can redirect all traffic.

### Step 2.4 - Find All DNS Records

```bash
dig example.com ANY
```

This attempts to retrieve all record types at once. Many DNS servers restrict this, but it is worth trying.

### Step 2.5 - Reverse DNS Lookup

If you know an IP address, you can find the domain associated with it:

```bash
dig -x 93.184.216.34
```

(This is example.com's IP.)

### Step 2.6 - DNS Zone Transfer (Important Attack Technique)

A zone transfer requests ALL DNS records for a domain at once. This should be restricted, but misconfigured servers sometimes allow it:

```bash
dig axfr example.com @ns1.example.com
```

**Expected result:** This should fail (refused). If it succeeds, the DNS server is misconfigured and you have found a vulnerability - you now have a complete map of all subdomains and servers.

> **Why this matters:** A successful zone transfer is a critical finding in a penetration test. It exposes the entire DNS infrastructure - every server, every subdomain, every internal hostname. Always test for this.

---

## Part 3: Google Dorking

Google indexes an enormous amount of information. Using advanced search operators (dorks), you can find things that were never meant to be public.

### Step 3.1 - Basic Google Dorks

Open Firefox on Kali and go to `https://www.google.com`. Try these searches:

**Find login pages:**
```
site:example.com inurl:login
```

**Find exposed documents:**
```
site:example.com filetype:pdf
```

**Find directory listings (misconfigured servers):**
```
site:example.com intitle:"index of"
```

**Find configuration files:**
```
site:example.com filetype:conf
```

**Find exposed databases:**
```
site:example.com filetype:sql
```

### Step 3.2 - More Advanced Dorks

**Find pages with passwords in them:**
```
site:example.com intext:"password" filetype:txt
```

**Find exposed admin panels:**
```
site:example.com inurl:admin
```

**Find Apache server status pages:**
```
inurl:"/server-status" "Apache Server Status"
```

**Find exposed webcams:**
```
inurl:"/view.shtml" "Network Camera"
```

### Step 3.3 - Google Dorking for Your Practice Domain

Try running some of these dorks against a practice target. **Never use these against a system you do not have permission to test.**

> **WARNING:** **Legal warning:** Google Dorking itself is legal - you are just using a search engine. However, accessing systems or data you find through dorking without authorization IS illegal. Finding it is fine. Accessing it without permission is a crime.

> **Why this matters:** Google Dorking is one of the most effective reconnaissance techniques. Bug bounty hunters use it regularly to find exposed sensitive data. During penetration tests, it often reveals forgotten test servers, exposed credentials, and misconfigured systems.

---

## Part 4: theHarvester

theHarvester is a tool specifically designed for gathering emails, subdomains, IPs, and other information from public sources.

### Step 4.1 - Basic Email Harvesting

```bash
theHarvester -d example.com -b google -l 100
```

**What the flags mean:**

| Flag | Meaning |
|------|---------|
| `-d` | The target domain |
| `-b` | The data source (google, bing, linkedin, etc.) |
| `-l` | Limit the number of results |

### Step 4.2 - Try Different Sources

```bash
# Search using Bing
theHarvester -d example.com -b bing -l 100

# Search using all available sources
theHarvester -d example.com -b all -l 200
```

### Step 4.3 - Interpret the Results

theHarvester typically returns:

- **Emails** - Employee email addresses (useful for phishing campaigns)
- **Subdomains** - mail.example.com, vpn.example.com, dev.example.com (hidden infrastructure)
- **IP addresses** - Server IPs associated with the domain
- **Hosts** - Specific hostnames and their IPs

### Step 4.4 - Save Results to a File

```bash
theHarvester -d example.com -b all -l 200 -f ~/cyber-labs/recon-results
```

The `-f` flag saves output to an HTML and XML file for later reference.

> **Why this matters:** Email addresses are used in phishing attacks. Subdomains can reveal hidden services (like internal apps exposed to the internet). This is often where the easiest vulnerabilities are - on forgotten subdomains running old software.

---

## Part 5: Passive vs. Active Reconnaissance Summary

### Step 5.1 - Understand the Difference

| Aspect | Passive Recon | Active Recon |
|--------|-------------|-------------|
| **Interaction with target** | None - you only use public data | Direct - you send packets to the target |
| **Detection risk** | Zero - target cannot detect you | High - target's IDS/firewall may detect you |
| **Legal risk** | Very low | Higher - requires authorization |
| **Tools** | WHOIS, Google, theHarvester | Nmap, vulnerability scanners |
| **Examples** | Reading public DNS records | Scanning ports on a server |
| **When to use** | Always - do this first | After you have authorization |

### Step 5.2 - Build a Reconnaissance Report

Create a structured report from your findings:

```bash
cat > ~/cyber-labs/recon-report.md << 'EOF'
# Reconnaissance Report
## Target: [domain name]
## Date: [today's date]
## Tester: [your name]

### 1. WHOIS Information
- Registrant: 
- Name Servers: 
- Registration Date: 
- Expiration Date: 

### 2. DNS Records
- A Record (IP): 
- MX Records (Mail): 
- NS Records (Name Servers): 
- Zone Transfer Possible: Yes / No

### 3. Emails Discovered
- [list emails found]

### 4. Subdomains Discovered
- [list subdomains found]

### 5. Google Dorking Findings
- [list any interesting findings]

### 6. Risk Assessment
- [summarize what an attacker could do with this information]
EOF

nano ~/cyber-labs/recon-report.md
```

Fill this in with the actual results from your reconnaissance.

> **Why this matters:** Professional penetration testers always document their findings in structured reports. Getting into this habit now will serve you well in the industry.

---

## Challenge Section (Optional)

1. **Research Shodan** (`https://www.shodan.io`). Create a free account and search for devices in Kenya. What can you find? (Do NOT interact with any devices.)
2. **Try `dnsrecon`** - another Kali tool for DNS enumeration:
  ```bash
  dnsrecon -d example.com
  ```
3. **Investigate a real data breach** - Go to `https://haveibeenpwned.com` and search your own email address. Were you affected by any breaches?

---

## Summary & Outcomes

| Skill | Status |
|-------|--------|
| Perform WHOIS lookups to find domain information | Yes |
| Use DNS tools to map a target's infrastructure | Yes |
| Use Google Dorks to find exposed data | Yes |
| Use theHarvester for email and subdomain collection | Yes |
| Distinguish between passive and active reconnaissance | Yes |
| Write a structured reconnaissance report | Yes |

---

## Reflection Questions

1. Why should passive reconnaissance always be done before active scanning?
2. You found that a company's domain expires in 2 weeks and they have not renewed it. What could an attacker do with this information?
3. A Google Dork reveals a company's internal document with employee names and phone numbers. Is this the company's fault or Google's fault? Why?
4. You found three subdomains using theHarvester: `mail.target.com`, `dev.target.com`, and `old.target.com`. Which one would you prioritize investigating and why?
5. How can organizations protect themselves against OSINT reconnaissance?
