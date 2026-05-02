# VAPT Workflow — Week 2

> End-to-end workflow for the full VAPT cycle following PTES methodology.

---

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                     VAPT FULL CYCLE — PTES                          │
│                                                                     │
│  ┌──────────────┐    ┌────────────────┐    ┌────────────────────┐  │
│  │  Phase 0     │    │  Phase 1       │    │  Phase 2           │  │
│  │  Environment │───▶│  Vulnerability │───▶│  Reconnaissance    │  │
│  │  Setup       │    │  Scanning      │    │  (OSINT)           │  │
│  │              │    │                │    │                    │  │
│  │ • Start VMs  │    │ • Nmap         │    │ • WHOIS            │  │
│  │ • Check tools│    │ • OpenVAS      │    │ • Sublist3r        │  │
│  │ • Set network│    │ • Nikto        │    │ • Shodan           │  │
│  └──────────────┘    └────────────────┘    │ • Maltego          │  │
│                                            │ • Wappalyzer       │  │
│                                            └────────────────────┘  │
│                                                      │              │
│                                                      ▼              │
│  ┌──────────────┐    ┌────────────────┐    ┌────────────────────┐  │
│  │  Phase 5     │    │  Phase 4       │    │  Phase 3           │  │
│  │  Reporting   │◀───│ Post-          │◀───│  Exploitation      │  │
│  │              │    │ Exploitation   │    │                    │  │
│  │ • PTES Report│    │                │    │ • Metasploit       │  │
│  │ • Exec Brief │    │ • Priv Esc     │    │ • sqlmap           │  │
│  │ • Dev Email  │    │ • Evidence     │    │ • Burp Suite       │  │
│  │ • Remediation│    │ • Hashing      │    │ • Custom PoCs      │  │
│  │ • GitHub Push│    │ • Memory       │    │                    │  │
│  └──────────────┘    └────────────────┘    └────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Phase-by-Phase Workflow

### Phase 0 — Environment Setup

| Step | Action | Command / Tool |
|------|--------|---------------|
| 0.1 | Start Kali Linux VM | VMware / VirtualBox |
| 0.2 | Start Metasploitable2 VM | VMware / VirtualBox |
| 0.3 | Start OpenVAS VM | VMware / VirtualBox |
| 0.4 | Start Windows 10 VM | VMware / VirtualBox |
| 0.5 | Verify network (Host-Only) | `ip a`, `ping 192.168.56.103`, `ping 192.168.56.105` |
| 0.6 | Access OpenVAS from Host browser | `http://192.168.56.104` |
| 0.7 | Confirm tools installed on Kali | `nmap --version`, `msfconsole --version` |

---

### Phase 1 — Vulnerability Scanning

| Step | Action | Command / Tool | Output |
|------|--------|---------------|--------|
| 1.1 | Port scan Metasploitable2 | `nmap -A -sV -sC -O 192.168.56.104` | Open ports + services |
| 1.2 | Access OpenVAS from Host browser | `http://192.168.56.103` | OpenVAS UI |
| 1.3 | Configure scan targets | UI → New Target → 192.168.56.104 | Targets added |
| 1.3 | Configure OpenVAS task | Browser → http://192.168.56.104 | Scan task created |
| 1.4 | Run OpenVAS full scan | UI → Start Task | 47 vulnerabilities |
| 1.5 | Nikto web scan | `nikto -h http://192.168.56.104` | Web vulnerabilities |
| 1.6 | Prioritise findings | CVSS scoring in table | Prioritised list |
| 1.7 | Document in scan_logs.md | Manual | Scan log updated |

---

### Phase 2 — Reconnaissance

| Step | Action | Command / Tool | Output |
|------|--------|---------------|--------|
| 2.1 | WHOIS lookup | `whois example.com` | Registrant info |
| 2.2 | DNS enumeration | `dig example.com ANY` | DNS records |
| 2.3 | Zone transfer attempt | `dig axfr @ns1.example.com example.com` | Refused (pass) |
| 2.4 | Subdomain enum | `python3 sublist3r.py -d example.com` | 7 subdomains |
| 2.5 | Shodan search | `shodan host <IP>` | Exposed services |
| 2.6 | Maltego graph | Maltego CE GUI | Asset relationship map |
| 2.7 | Tech fingerprint | Wappalyzer CLI / extension | Tech stack |
| 2.8 | Email harvest | `theHarvester -d example.com -b google` | 3 emails |
| 2.9 | Google dorks | Google search | Exposed files |
| 2.10 | Document in recon_logs.md | Manual | Recon log updated |

---

### Phase 3 — Exploitation

| Step | Action | Command / Tool | Result |
|------|--------|---------------|--------|
| 3.1 | Start Metasploit | `msfconsole` | MSF console open |
| 3.2 | Exploit Samba RCE | `use exploit/multi/samba/usermap_script` | Root shell |
| 3.3 | Exploit vsftpd backdoor | `use exploit/unix/ftp/vsftpd_234_backdoor` | Shell |
| 3.4 | Exploit Tomcat | `use exploit/multi/http/tomcat_mgr_login` | Login success |
| 3.5 | SQL inject DVWA | `sqlmap -u "[url]" --dbs --dump` | DB dumped |
| 3.6 | Test XSS DVWA | Burp Suite + browser | Alert confirmed |
| 3.7 | Validate via Exploit-DB | https://exploit-db.com | PoC cross-checked |
| 3.8 | Document in exploit_logs.md | Manual | Exploit log updated |

---

### Phase 4 — Post-Exploitation

| Step | Action | Command / Tool | Result |
|------|--------|---------------|--------|
| 4.1 | Check privileges | `meterpreter > getuid` | uid=0 (root) |
| 4.2 | System info | `meterpreter > sysinfo` | OS details |
| 4.3 | Download /etc/passwd | `meterpreter > download /etc/passwd` | File saved |
| 4.4 | Download /etc/shadow | `meterpreter > download /etc/shadow` | File saved |
| 4.5 | Hash all evidence | `sha256sum <file>` | Hashes recorded |
| 4.6 | BypassUAC (if Windows) | `use exploit/windows/local/bypassuac` | Elevated session |
| 4.7 | Update evidence table | Manual | Chain of custody |

---

### Phase 5 — Reporting

| Step | Action | Tool | Output |
|------|--------|------|--------|
| 5.1 | Compile all findings | Markdown | VAPT_Full_Report.md |
| 5.2 | Write executive summary | Markdown | Section 1 of report |
| 5.3 | Write remediation plan | Markdown | Section 9 of report |
| 5.4 | Draft developer email | Markdown | Section 10 of report |
| 5.5 | Write non-tech briefing | Markdown | Section 11 of report |
| 5.6 | Add screenshots | PNG files | screenshots/ folder |
| 5.7 | Commit to GitHub | `git push origin main` | Repository updated |

---

## Data Flow Diagram

```
[ Kali Linux — Attacker ]
         │
         ├──── Nmap ──────────────────────▶ [ Metasploitable2 ]
         │                                        │
         ├──── OpenVAS ───────────────────▶       │ Open ports
         │                                        │ Vulnerabilities
         ├──── Metasploit (Samba RCE) ───▶        │
         │         └─── Root Shell ◀──────────────┘
         │
         ├──── sqlmap ────────────────────▶ [ DVWA ]
         │         └─── DB Dump ◀──────────────────┘
         │
         ├──── Burp Suite ────────────────▶ [ DVWA ]
         │         └─── XSS Confirmed ◀─────────────┘
         │
         └──── Report ─────────────────── [ GitHub Repo ]
```

---

## Time Estimate Per Phase

| Phase | Estimated Time | Actual Time |
|-------|---------------|-------------|
| Environment Setup | 15 min | `[FILL IN]` |
| Vulnerability Scanning | 45 min | `[FILL IN]` |
| Reconnaissance | 30 min | `[FILL IN]` |
| Exploitation | 45 min | `[FILL IN]` |
| Post-Exploitation | 20 min | `[FILL IN]` |
| Reporting | 60 min | `[FILL IN]` |
| **Total** | **~3.5 hours** | `[FILL IN]` |

---

