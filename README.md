# Week 2 — Vulnerability Assessment & Penetration Testing (VAPT)

## 📁 Repository Structure

```
week2/
├── README.md                        ← You are here
├── steps.md                         ← Full workflow with ordered steps
├── assets/                          ← Images and screenshots used in reports
├── notes/
│   └── theoretical_knowledge.md    ← All theory: scanning, pentesting, exploit dev
├── logs/
│   ├── scan_logs.md                 ← Nmap / OpenVAS / Nikto scan results
│   ├── recon_logs.md                ← OSINT / Recon activity log
│   └── exploit_logs.md              ← Exploitation & post-exploitation logs
├── reports/
│   └── VAPT_Full_Report.md          ← Full professional VAPT report (PTES-based)
├── screenshots/
│   └── README.md                    ← Screenshot instructions and index
└── workflow/
    └── VAPT_Workflow.md             ← End-to-end workflow diagram & description
```

---

## 🎯 Objectives

| # | Objective | Status |
|---|-----------|--------|
| 1 | Perform vulnerability scanning (Nmap, OpenVAS, Nikto) | ✅ Documented |
| 2 | Conduct OSINT/Reconnaissance (Shodan, Maltego) | ✅ Documented |
| 3 | Execute exploitation lab (Metasploit, Burp Suite, sqlmap) | ✅ Documented |
| 4 | Perform post-exploitation (Meterpreter, Volatility) | ✅ Documented |
| 5 | Complete Capstone: Full VAPT cycle (DVWA) | ✅ Documented |
| 6 | Write professional PTES-based report | ✅ Complete |

---

## 🧪 Lab Environment

| Component | Details |
|-----------|---------|
| Attacker Machine | Kali Linux 2024.x (VM) — `192.168.56.101` |
| Target 1 | Metasploitable2 (Linux) — `192.168.56.104` (DVWA also hosted here) |
| Target 2 | Windows 10 VM — `192.168.56.105` |
| Vulnerability Scanner | OpenVAS (dedicated VM) — `192.168.56.103` — accessed via Host browser UI |
| Network | Host-Only Adapter (isolated lab — all VMs on same subnet) |
| Tools Used | Nmap, OpenVAS, Nikto, Metasploit, Burp Suite, sqlmap, Maltego, Shodan, Meterpreter, Volatility |

---

## 🔗 Quick Navigation

- [📖 Theoretical Notes](notes/theoretical_knowledge.md)
- [🔍 Scan Logs](logs/scan_logs.md)
- [🕵️ Recon Logs](logs/recon_logs.md)
- [💥 Exploit Logs](logs/exploit_logs.md)
- [📋 Full VAPT Report](reports/VAPT_Full_Report.md)
- [⚙️ Workflow](workflow/VAPT_Workflow.md)
- [🪜 Steps](steps.md)
- [📸 Screenshots Index](screenshots/README.md)
- [🖼️ Assets Directory](assets/)

---

## ⚠️ Legal Disclaimer

> All activities documented in this repository were performed in a **controlled, isolated lab environment** on intentionally vulnerable machines (Metasploitable2, Windows 10 VM, DVWA) for **educational purposes only**. No real-world systems were targeted. All testing was conducted with explicit authorization within the scope of the CyArt VAPT training programme.

---

