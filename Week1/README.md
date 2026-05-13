# 🛡️ Week 1 — VAPT Foundations: Vulnerability Assessment Using Open-Source Tools

> **Week:** 1 — Vulnerability Identification, Risk Analysis & Basic Exploitation Validation

---

## 📁 Repository Structure

```
week1/
├── README.md
├── assets/                          ← Screenshots and evidence images
├── notes/
│   └── theoretical_knowledge.md    ← Theory: VAPT methodology, tools, CVSSv3
├── logs/
│   ├── scan_logs.md                 ← Nmap, OpenVAS, Nikto scan results
│   ├── recon_logs.md                ← Reconnaissance & enumeration logs
│   └── exploitation_logs.md         ← Exploitation validation logs
├── reports/
│   ├── VAPT_Week1_Report.md         ← Full professional PTES-based report
│   ├── vapt_task01.pdf              ← Original PDF submission (full report)
│   └── 2026-04-30-ZAP-Report-.pdf  ← OWASP ZAP automated scan findings (exported from ZAP)
└── workflow/
    └── VAPT_Workflow.md             ← End-to-end workflow diagram & description
```

---

## 🖥️ Lab Environment

| Role | OS | IP Address |
|------|----|-----------| 
| Attacker | Kali Linux 2024.x | 192.168.56.101 |
| Vulnerable Target | Metasploitable2 (Linux) | 192.168.56.104 |
| Vulnerability Scanner | OpenVAS VM (dedicated, accessed via host browser) | 192.168.56.103 |
| Host Machine | Windows/Linux Host | Web UI access for OpenVAS |

> **Note:** Due to hardware limitations (limited RAM and CPU), OpenVAS was deployed on a dedicated VM rather than inside Kali Linux. This improved scanning stability and reflects real-world enterprise VAPT architecture.

---

## 🎯 Week 1 Objectives

| # | Objective | Status |
|---|-----------|--------|
| 1 | Set up distributed VAPT lab (Kali + Metasploitable + OpenVAS VM) | ✅ Documented |
| 2 | Perform Nmap port & service enumeration | ✅ Documented |
| 3 | Run OpenVAS full vulnerability scan | ✅ Documented |
| 4 | Conduct Nikto web server scan | ✅ Documented |
| 5 | Perform OWASP ZAP web application scan | ✅ Documented |
| 6 | Validate exploitation with Metasploit (CVE-2012-1823) | ✅ Documented |
| 7 | Conduct CVSS-based risk assessment | ✅ Documented |
| 8 | Write professional PTES-based VAPT report | ✅ Documented |

---

## 🔑 Key Findings Summary

| ID | Vulnerability | Severity | CVSS |
|----|---------------|----------|------|
| V-01 | PHP CGI Argument Injection (CVE-2012-1823) | Critical | 9.8 |
| V-02 | DistCC Remote Code Execution (CVE-2004-2687) | Critical | 9.3 |
| V-03 | vsftpd 2.3.4 Backdoor (CVE-2011-2523) | Critical | 10.0 |
| V-04 | MySQL Default/Blank Root Password | Critical | 8.5 |
| V-05 | rlogin/rexec Services (No Authentication) | Critical | 9.0 |
| V-06 | SQL Injection (DVWA/Mutillidae) | High | 8.8 |
| V-07 | Cross-Site Scripting — 25 instances | High | 7.4 |
| V-08 | Path Traversal — 20 instances | High | 7.5 |

> **Total:** 14 Critical · 8 High · 40 Medium · 6 Low

---

## 🔗 Quick Navigation

- [📖 Theoretical Notes](notes/theoretical_knowledge.md)
- [🔍 Scan Logs](logs/scan_logs.md)
- [🕵️ Recon Logs](logs/recon_logs.md)
- [💥 Exploitation Logs](logs/exploitation_logs.md)
- [📋 Full VAPT Report](reports/VAPT_Week1_Report.md)
- [📄 Original PDF Report](reports/vapt_task01.pdf)
- [🌐 ZAP Findings Report (PDF)](reports/2026-04-30-ZAP-Report-.pdf)
- [⚙️ Workflow](workflow/VAPT_Workflow.md)
- [🖼️ Assets Directory](assets/)

---

## ⚠️ Legal Disclaimer

> All activities documented in this repository were performed in a **controlled, isolated lab environment** on intentionally vulnerable machines (Metasploitable2) for **educational purposes only**. No real-world systems were targeted. All testing was conducted with explicit authorization within the scope of the CyArt VAPT training programme.

---
