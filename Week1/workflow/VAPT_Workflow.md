# ⚙️ VAPT Workflow — Week 1

> End-to-end workflow for the Week 1 vulnerability assessment and penetration testing engagement.

---

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│               WEEK 1 — VAPT FULL CYCLE (PTES)                       │
│                                                                     │
│  ┌──────────────┐    ┌────────────────┐    ┌────────────────────┐  │
│  │  Phase 0     │    │  Phase 1       │    │  Phase 2           │  │
│  │  Lab Setup   │───▶│  Vulnerability │───▶│  Reconnaissance    │  │
│  │              │    │  Scanning      │    │  & Enumeration     │  │
│  │ • Kali VM    │    │                │    │                    │  │
│  │ • Meta2 VM   │    │ • Nmap         │    │ • Port enumeration │  │
│  │ • OpenVAS VM │    │ • OpenVAS      │    │ • Service versions │  │
│  │ • Network    │    │ • Nikto        │    │ • Web apps         │  │
│  │   config     │    │ • OWASP ZAP    │    │ • Tech stack       │  │
│  └──────────────┘    └────────────────┘    └────────────────────┘  │
│                                                      │              │
│                                                      ▼              │
│  ┌──────────────┐    ┌────────────────┐    ┌────────────────────┐  │
│  │  Phase 5     │    │  Phase 4       │    │  Phase 3           │  │
│  │  Reporting   │◀───│ Risk           │◀───│  Exploitation      │  │
│  │              │    │ Assessment     │    │  Validation        │  │
│  │ • PTES Report│    │                │    │                    │  │
│  │ • Risk Matrix│    │ • CVSS scoring │    │ • Metasploit       │  │
│  │ • Remediation│    │ • Risk matrix  │    │ • PHP CGI RCE      │  │
│  │ • Conclusion │    │ • Prioritise   │    │ • vsftpd backdoor  │  │
│  └──────────────┘    └────────────────┘    └────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Phase-by-Phase Workflow

### Phase 0 — Lab Environment Setup

| Step | Action | Tool / Command |
|------|--------|---------------|
| 0.1 | Start Kali Linux VM | VirtualBox |
| 0.2 | Start Metasploitable2 VM | VirtualBox |
| 0.3 | Start dedicated OpenVAS VM | VirtualBox |
| 0.4 | Configure all VMs on Host-Only or NAT Network | VirtualBox network settings |
| 0.5 | Verify connectivity | `ping 192.168.56.104` from Kali |
| 0.6 | Access OpenVAS from host browser | `http://192.168.56.103` |
| 0.7 | Confirm Kali tools available | `nmap --version`, `msfconsole --version` |

> **Architecture Note:** Due to hardware limitations, OpenVAS ran on a dedicated VM. This avoided RAM exhaustion and service crashes when running it inside Kali, and reflects real enterprise VAPT infrastructure design.

---

### Phase 1 — Vulnerability Scanning

| Step | Action | Command / Tool | Output |
|------|--------|---------------|--------|
| 1.1 | Nmap service scan | `nmap -A -sV -sC -O 192.168.56.104` | Open ports + service versions |
| 1.2 | Nmap full port scan | `nmap -p- -T4 192.168.56.104` | All 65535 ports checked |
| 1.3 | Configure OpenVAS target | Browser UI → New Target | 192.168.56.104 added |
| 1.4 | Run OpenVAS full scan | Browser UI → New Task → Start | 68 vulnerabilities found |
| 1.5 | Nikto web scan | `nikto -h http://192.168.56.104` | Web server issues |
| 1.6 | OWASP ZAP spider | ZAP GUI → Spider → http://192.168.56.104 | All endpoints mapped |
| 1.7 | OWASP ZAP active scan | ZAP GUI → Active Scan | XSS, SQLi, path traversal found |
| 1.8 | Export OpenVAS report | Browser UI → Reports → PDF | Findings exported |
| 1.9 | Document in scan_logs.md | Manual | Scan log updated |

---

### Phase 2 — Reconnaissance & Enumeration

| Step | Action | Command / Tool | Output |
|------|--------|---------------|--------|
| 2.1 | Host discovery | `nmap -sn 192.168.56.0/24` | 3 live hosts |
| 2.2 | FTP anonymous login test | `ftp 192.168.56.104` | Login confirmed |
| 2.3 | Web server headers | `curl -I http://192.168.56.104` | Apache 2.2.8, PHP 5.2.4 |
| 2.4 | MySQL blank root test | `mysql -h 192.168.56.104 -u root -p` | Full DB access |
| 2.5 | Samba share enumeration | `smbclient -L //192.168.56.104 -N` | Shares listed |
| 2.6 | VNC authentication check | `vncviewer 192.168.56.104:5900` | No auth — direct access |
| 2.7 | Bindshell check | `nc 192.168.56.104 1524` | Root shell direct |
| 2.8 | Web app discovery | Browser + curl | DVWA, Mutillidae, phpMyAdmin |
| 2.9 | Document in recon_logs.md | Manual | Recon log updated |

---

### Phase 3 — Exploitation Validation

| Step | Action | Command / Tool | Result |
|------|--------|---------------|--------|
| 3.1 | Launch Metasploit | `msfconsole` | MSF console open |
| 3.2 | PHP CGI exploit | `use exploit/multi/http/php_cgi_arg_injection` | www-data shell ✅ |
| 3.3 | vsftpd backdoor | `use exploit/unix/ftp/vsftpd_234_backdoor` | Root shell ✅ |
| 3.4 | DistCC RCE | `use exploit/unix/misc/distcc_exec` | Daemon shell ✅ |
| 3.5 | MySQL root access | `mysql -u root` (no password) | Full DB access ✅ |
| 3.6 | Collect evidence | `meterpreter > download /etc/passwd` | Files saved |
| 3.7 | Hash evidence | `sha256sum` | Chain of custody |
| 3.8 | Document in exploitation_logs.md | Manual | Exploit log updated |

---

### Phase 4 — Risk Assessment

| Step | Action | Tool | Output |
|------|--------|------|--------|
| 4.1 | Score each vulnerability | CVSS v3.1 Calculator | CVSS scores assigned |
| 4.2 | Build risk matrix | Manual (Likelihood × Impact) | Prioritised findings |
| 4.3 | Categorise by severity | CVSS ranges | Critical/High/Medium/Low |
| 4.4 | Map to remediation | Manual | Fix recommendations |

---

### Phase 5 — Reporting

| Step | Action | Tool | Output |
|------|--------|------|--------|
| 5.1 | Compile all findings | Markdown | VAPT_Week1_Report.md |
| 5.2 | Write executive summary | Markdown | Section 1 of report |
| 5.3 | Document lab architecture | Markdown + diagram | Section 3 of report |
| 5.4 | Write vulnerability findings | Markdown | Sections 5–8 |
| 5.5 | Write remediation plan | Markdown | Section 10 of report |
| 5.6 | Write conclusion | Markdown | Section 12 of report |
| 5.7 | Commit to repository | `git add . && git commit -m "Week1 complete"` | Repository updated |

---

## Data Flow Diagram

```
[ Kali Linux — Attacker ]
         │
         ├──── Nmap ────────────────────────▶ [ Metasploitable2 ]
         │                                         │
         │                                    Open ports discovered
         │
         ├──── OpenVAS VM ──────────────────▶ [ Metasploitable2 ]
         │         └─── 68 Vulnerabilities ◀────────────────────┘
         │
         ├──── Nikto ───────────────────────▶ [ Port 80 / Apache ]
         │         └─── Web issues found ◀────────────────────────┘
         │
         ├──── OWASP ZAP ───────────────────▶ [ DVWA / Mutillidae ]
         │         └─── XSS, SQLi, LFI ◀──────────────────────────┘
         │
         ├──── Metasploit ──────────────────▶ [ Port 80 / PHP CGI ]
         │         └─── www-data shell ◀────────────────────────────┘
         │
         ├──── Metasploit ──────────────────▶ [ Port 21 / vsftpd ]
         │         └─── Root shell ◀──────────────────────────────────┘
         │
         └──── Report ──────────────────────▶ [ GitHub Repository ]
```

---

## Time Estimate Per Phase

| Phase | Estimated Time |
|-------|---------------|
| Lab Setup | 30 min |
| Vulnerability Scanning | 60 min |
| Reconnaissance & Enumeration | 30 min |
| Exploitation Validation | 45 min |
| Risk Assessment | 20 min |
| Reporting | 60 min |
| **Total** | **~3.5 hours** |

---
