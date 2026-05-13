# 🖼️ Assets — Week 1 Screenshots

This directory contains all screenshots and evidence images captured during the Week 1 VAPT assessment, extracted from the original PDF submission.

---

## Screenshot Index

| Filename | Description | Phase | PDF Page |
|----------|-------------|-------|----------|
| `nmap1.png` | Nmap `-sV -A` scan — ports 21–111 (FTP, SSH, Telnet, HTTP, RPC) | Scanning | 7 |
| `nmap2.png` | Nmap scan — ports 139–8180 (Samba, MySQL, VNC, IRC, Tomcat) + OS detection | Scanning | 8 |
| `nmap3.png` | Nmap scan — completion, SMB host script results, traceroute | Scanning | 9 |
| `openvas_dashboard.png` | OpenVAS Greenbone Security Assistant dashboard + Metasploitable scan task | Scanning | 10 |
| `openvas_results.png` | OpenVAS results per host — Critical/High port threat level table | Scanning | 11 |
| `nikto_scan.png` | Nikto v2.6.0 web server scan against Apache 2.2.8 (port 80) | Web Testing | 12 |
| `zap_automated_scan.png` | OWASP ZAP 2.17.0 automated scan setup targeting Mutillidae | Web Testing | 14 |
| `zap_active_scan.png` | ZAP spider + active scan in progress with alerts list (XSS, SQLi, RCE, LFI) | Web Testing | 15 |
| `zap_rce_finding.png` | ZAP RCE finding detail — CVE-2012-1823 (PHP CGI) + XSS alerts panel | Web Testing | 16 |
| `msf_php_cgi_exploit.png` | Metasploit PHP CGI exploit (CVE-2012-1823) + Meterpreter session established | Exploitation | 17 |

---

## Image Reference Guide

When referencing these images from logs or reports, use relative paths:

```markdown
# From logs/ directory:
![alt text](../assets/nmap1.png)

# From reports/ directory:
![alt text](../assets/nmap1.png)
```

---
