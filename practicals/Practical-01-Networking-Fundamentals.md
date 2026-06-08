# Practical 1: Networking Fundamentals

**Atiam College - Cyber Security & Ethical Hacking**
**Module:** Week 3 - Networking Basics for Ethical Hacking
**Duration:** 2–3 Hours
**Difficulty:** Beginner

---

## Objective

By the end of this practical, you will be able to:

- View and interpret your machine's network configuration
- Understand IP addresses, subnet masks, and default gateways
- Capture and analyze live network traffic using Wireshark
- Identify common protocols (TCP, HTTP, DNS) in packet captures
- Create basic firewall rules using iptables

---

## Prerequisites

- Access to your Kali Linux VM (running in VMware)
- Basic terminal knowledge (opening a terminal, typing commands)
- Internet connection on the VM

---

## Tools Used

| Tool | What It Does |
|------|-------------|
| `ip` / `ifconfig` | Shows your network interface configuration |
| `ping` | Tests if a remote host is reachable |
| `traceroute` | Shows the path packets take to reach a destination |
| Wireshark | Captures and analyzes network traffic visually |
| `iptables` | Linux firewall - controls what traffic is allowed or blocked |

---

## Real-World Context

Every cyber security professional must understand networking. When you investigate a breach, you analyze network traffic. When you set up defenses, you configure firewalls. When you perform a penetration test, you scan networks. **Networking is the foundation of everything in cyber security.** Without it, you are blind.

---

## Part 1: Understanding Your Network Configuration

### Step 1.1 - View Your IP Address

Open a terminal and type:

```bash
ip addr show
```

You will see output similar to this:

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
  inet 192.168.1.105/24 brd 192.168.1.255 scope global dynamic eth0
```

**What to look for:**

- `eth0` - This is your network interface (your "network card")
- `inet 192.168.1.105/24` - This is your IP address. The `/24` is the subnet mask
- `brd 192.168.1.255` - This is the broadcast address

> **Why this matters:** Your IP address is your identity on the network. Attackers look for active IP addresses when scanning a network. Defenders need to know which IPs belong to their systems.

### Step 1.2 - Understand the Subnet Mask

The `/24` you saw means the first 24 bits are the network part. In simple terms:

- `/24` = `255.255.255.0` = 254 usable hosts on this network
- Your network is `192.168.1.0/24`, meaning all devices from `192.168.1.1` to `192.168.1.254` are on your local network

Try this command to see it in the traditional format:

```bash
ifconfig eth0
```

Look for the line that says `netmask 255.255.255.0`.

### Step 1.3 - Find Your Default Gateway

The gateway is the door to the internet - the router that sends your traffic outside your local network.

```bash
ip route show
```

You should see:

```
default via 192.168.1.1 dev eth0
```

This means `192.168.1.1` is your gateway (your router).

### Step 1.4 - Find Your DNS Server

DNS translates domain names (like google.com) to IP addresses.

```bash
cat /etc/resolv.conf
```

You will see something like:

```
nameserver 8.8.8.8
```

> **Why this matters:** Attackers can hijack DNS to redirect users to fake websites (DNS spoofing). Knowing your DNS settings helps you detect this.

---

## Part 2: Testing Connectivity

### Step 2.1 - Ping a Remote Host

Ping sends small packets to a target and measures if they reply.

```bash
ping -c 4 google.com
```

The `-c 4` means send only 4 packets (otherwise it runs forever).

**Expected output:**

```
64 bytes from 142.250.180.14: icmp_seq=1 ttl=117 time=12.3 ms
64 bytes from 142.250.180.14: icmp_seq=2 ttl=117 time=11.8 ms
```

**What to look at:**

- `64 bytes` - the packet size
- `ttl=117` - Time To Live, how many routers the packet can pass through
- `time=12.3 ms` - how long the round trip took (lower = faster)

> **WARNING:** **If ping fails:** Some networks block ICMP (ping) packets. This does not mean the host is down - it means ping is blocked. This is important to remember during penetration testing.

### Step 2.2 - Trace the Route to a Server

Traceroute shows every router (hop) between you and the destination:

```bash
traceroute google.com
```

**Expected output (example):**

```
 1 192.168.1.1 (192.168.1.1)  1.234 ms
 2 10.0.0.1 (10.0.0.1)     5.678 ms
 3 isp-router.example.com    12.345 ms
 ...
```

Each line is a router your traffic passes through. The first hop is your local gateway.

> **Why this matters:** During reconnaissance, traceroute helps you map out a target's network infrastructure. It reveals what ISP they use, whether they have multiple layers of protection, and potential points of attack.

---

## Part 3: Capturing Traffic with Wireshark

This is where it gets real. You are going to capture live network traffic and see exactly what is happening on the wire.

### Step 3.1 - Launch Wireshark

```bash
sudo wireshark
```

If prompted about running as root, click OK. (In a lab environment, this is fine.)

### Step 3.2 - Start Capturing

1. In the Wireshark main screen, you will see a list of network interfaces
2. Double-click on **eth0** (or whatever your active interface is)
3. Wireshark will immediately start capturing packets - you will see them scrolling

### Step 3.3 - Generate Some Traffic

Open a **second terminal** (keep Wireshark running) and type:

```bash
curl http://example.com
```

This fetches a web page over HTTP (unencrypted), which means Wireshark can read it.

### Step 3.4 - Stop the Capture

Go back to Wireshark and click the **red square button** (Stop) in the toolbar.

### Step 3.5 - Analyze the Capture

Now you have captured packets. Let's filter and understand them.

**Filter for HTTP traffic only:**

In the filter bar at the top of Wireshark, type:

```
http
```

Press Enter. Now you only see HTTP packets.

**Find your request:**

- Look for a packet with `GET / HTTP/1.1` in the Info column
- Click on it
- In the bottom pane, expand **Hypertext Transfer Protocol**
- You can see the full HTTP request your machine sent

**Filter for DNS:**

Clear the filter and type:

```
dns
```

- Find the DNS query for `example.com`
- See the response with the IP address it resolved to

> **Why this matters:** This is exactly what security analysts do during incident response. When a machine is compromised, they capture traffic to see what data the attacker is sending out, what servers the malware is contacting, and what commands are being received.

### Step 3.6 - Observe the TCP Three-Way Handshake

Clear the filter and type:

```
tcp.flags.syn == 1
```

This shows SYN packets - the first step of every TCP connection. Find one and follow the sequence:

1. **SYN** - Your machine says "I want to connect"
2. **SYN-ACK** - The server says "OK, I accept"
3. **ACK** - Your machine says "Great, let's talk"

Right-click any packet → **Follow** → **TCP Stream** to see the entire conversation.

> **Why this matters:** Understanding the TCP handshake is essential for understanding SYN flood attacks (a type of DDoS), port scanning (which is based on SYN packets), and firewall rules.

---

## Part 4: Basic Firewall with iptables

A firewall controls what traffic enters and leaves your machine. You are going to create rules manually.

### Step 4.1 - View Current Rules

```bash
sudo iptables -L -n -v
```

If the output shows `policy ACCEPT` and no rules, your firewall is completely open - everything is allowed.

### Step 4.2 - Block Incoming Ping (ICMP)

```bash
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

**What this does:**

- `-A INPUT` - Add a rule to incoming traffic
- `-p icmp` - Protocol is ICMP (ping)
- `--icmp-type echo-request` - Only block ping requests
- `-j DROP` - Silently drop the packet (no response sent back)

**Test it:** From your host machine (Windows), try pinging your Kali VM's IP address. It should now fail or timeout.

### Step 4.3 - Block a Specific Port

Let's block incoming traffic on port 23 (Telnet - an insecure remote access protocol):

```bash
sudo iptables -A INPUT -p tcp --dport 23 -j DROP
```

### Step 4.4 - Allow Only SSH (Port 22) and Block Everything Else

This is how real servers are configured - deny by default, allow only what you need:

```bash
# First, allow established connections to continue
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow loopback (localhost communication)
sudo iptables -A INPUT -i lo -j ACCEPT

# Drop everything else
sudo iptables -A INPUT -j DROP
```

> **WARNING:** **Be careful:** If you are accessing your VM via SSH, make sure the SSH allow rule is added BEFORE the drop-all rule, or you will lock yourself out.

### Step 4.5 - View Your Rules

```bash
sudo iptables -L -n -v --line-numbers
```

You should now see your rules listed with line numbers.

### Step 4.6 - Remove All Rules (Reset)

When you are done experimenting:

```bash
sudo iptables -F
```

This flushes (deletes) all rules, returning to the default open state.

> **Why this matters:** Firewalls are the first line of defense in any network. As a security professional, you must be able to configure them. During penetration testing, understanding firewall rules helps you know which ports are blocked and find ways around them.

---

## Challenge Section (Optional)

If you finished early, try these:

1. **Capture HTTPS traffic** - Visit `https://google.com` while Wireshark is running. Can you read the content? Why not? (Hint: TLS encryption)
2. **Create a firewall rule** that allows HTTP (port 80) and HTTPS (port 443) but blocks everything else
3. **Use `ss -tulnp`** to see all listening ports on your machine. Research what each service is

---

## Summary & Outcomes

After completing this practical, you can now:

| Skill | Status |
|-------|--------|
| View and interpret IP configuration on Linux | Yes |
| Test connectivity with ping and traceroute | Yes |
| Capture and analyze network traffic with Wireshark | Yes |
| Identify HTTP, DNS, and TCP packets in a capture | Yes |
| Create and manage iptables firewall rules | Yes |

---

## Reflection Questions

1. Why is it important to understand your own network configuration before testing others?
2. What is the difference between dropping a packet (`DROP`) and rejecting it (`REJECT`)? Which is more secure and why?
3. If you captured traffic on a network and saw unencrypted HTTP requests with login credentials, what would that tell you about the security of that application?
4. How could an attacker use traceroute information against a target?
5. Why do some organizations block ICMP (ping) on their firewalls?
