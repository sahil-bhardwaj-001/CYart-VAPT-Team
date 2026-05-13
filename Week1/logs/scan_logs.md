# 🔍 Scan Logs — Week 1 VAPT

> **Environment:** Kali Linux (192.168.56.101) → Metasploitable2 (192.168.56.104)
> **OpenVAS Scanner VM:** 192.168.56.103 (accessed via Host browser)

---

## 1. Nmap Port & Service Scan

### Command Executed
```bash
nmap -A -sV -sC -O 192.168.56.104 -oN nmap_metasploitable.txt
```

Full port scan:
```bash
nmap -p- -T4 192.168.56.104
```

> `[SCREENSHOT: Nmap scan — ports 21-111: FTP, SSH, Telnet, HTTP, RPC services]`

![alt text](../assets/nmap1.png)

> `[SCREENSHOT: Nmap scan — ports 139-8180: Samba, MySQL, VNC, IRC, Tomcat + OS detection]`

![alt text](../assets/nmap2.png)

> `[SCREENSHOT: Nmap scan completion — SMB host script results, traceroute, scan summary]`

![alt text](../assets/nmap3.png)

### Discovered Ports & Services

```
Nmap scan report for 192.168.56.104
Host is up (0.00088s latency).

PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 2.3.4
22/tcp    open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp    open  telnet      Linux telnetd
25/tcp    open  smtp        Postfix smtpd
53/tcp    open  domain      ISC BIND 9.4.2
80/tcp    open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X
445/tcp   open  netbios-ssn Samba smbd 3.0.20-Debian
512/tcp   open  exec        netkit-rsh rexecd
513/tcp   open  login
514/tcp   open  shell       Netkit rshd
1099/tcp  open  java-rmi    GNU Classpath grmiregistry
1524/tcp  open  bindshell   Metasploitable root shell
2049/tcp  open  nfs         2-4 (RPC #100003)
2121/tcp  open  ftp         ProFTPD 1.3.1
3306/tcp  open  mysql       MySQL 5.0.51a-3ubuntu5
3632/tcp  open  distccd     distccd v1 ((GNU) 4.2.4)
5432/tcp  open  postgresql  PostgreSQL DB 8.3.0
5900/tcp  open  vnc         VNC (protocol 3.3)
6000/tcp  open  X11         (access denied)
6667/tcp  open  irc         UnrealIRCd
8009/tcp  open  ajp13       Apache Jserv (Protocol v1.3)
8180/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1

OS details: Linux 2.6.9 - 2.6.33
```

### Notable Findings

| Port | Service | Version | Risk |
|------|---------|---------|------|
| 21 | FTP | vsftpd 2.3.4 | **Critical** — Known backdoor (CVE-2011-2523) |
| 80 | HTTP | Apache 2.2.8 + PHP CGI | **Critical** — CVE-2012-1823 RCE |
| 512–514 | rexec/rlogin/rsh | netkit | **Critical** — No authentication |
| 1524 | bindshell | Metasploitable root shell | **Critical** — Root backdoor |
| 3306 | MySQL | 5.0.51a | **Critical** — Blank root password |
| 3632 | distccd | v1 GNU 4.2.4 | **Critical** — Unauthenticated RCE (CVE-2004-2687) |
| 6667 | IRC | UnrealIRCd | **Critical** — Backdoor (CVE-2010-2075) |

---

## 2. OpenVAS Vulnerability Scan

### Scan Configuration

| Setting | Value |
|---------|-------|
| OpenVAS VM IP | 192.168.56.103 |
| Web UI URL | http://192.168.56.103 |
| Target | 192.168.56.104 (Metasploitable2) |
| Scan Profile | Full and Fast |

### Deployment Note

A dedicated OpenVAS VM was deployed due to instability when running OpenVAS directly inside Kali Linux on the assessment hardware. The host machine browser was used to access the Greenbone Security Assistant (GSA) web interface.

**Process Followed:**
1. Boot OpenVAS VM
2. Login to scanner VM terminal
3. Verify scanner services (`gvmd --version`, `openvas --version`)
4. Access Greenbone Security Assistant: `http://192.168.56.103`
5. Configure target: `New Target → 192.168.56.104`
6. Launch scan: `New Task → Full and Fast → Start`
7. Export findings: `Reports → Download XML/PDF`

> `[SCREENSHOT: OpenVAS Greenbone Security Assistant dashboard + Metasploitable scan task]`

![alt text](../assets/openvas_dashboard.png)

> `[SCREENSHOT: OpenVAS results per host — Critical/High threat level port table]`

![alt text](../assets/openvas_results.png)

### Scan Results Summary

| Severity | Count |
|----------|-------|
| **Critical** | **14** |
| **High** | **8** |
| **Medium** | **40** |
| **Low** | **6** |
| **Total** | **68** |

### Top Vulnerabilities Found (Prioritised by CVSS)

| ID | Vulnerability | CVE | CVSS | Severity | Port |
|----|---------------|-----|------|----------|------|
| 001 | vsftpd 2.3.4 Backdoor | CVE-2011-2523 | 10.0 | Critical | 21 |
| 002 | Samba Username Map Script RCE | CVE-2007-2447 | 10.0 | Critical | 445 |
| 003 | UnrealIRCd Backdoor | CVE-2010-2075 | 10.0 | Critical | 6667 |
| 004 | Bindshell Root Backdoor | — | 10.0 | Critical | 1524 |
| 005 | PHP CGI Argument Injection | CVE-2012-1823 | 9.8 | Critical | 80 |
| 006 | DistCC Remote Code Execution | CVE-2004-2687 | 9.3 | Critical | 3632 |
| 007 | MySQL No Root Password | — | 8.5 | Critical | 3306 |
| 008 | rlogin/rexec/rsh Services | — | 9.0 | Critical | 512–514 |
| 009 | SQL Injection | CVE-2021-41773 | 9.1 | Critical | 80 |
| 010 | VNC No Authentication | — | 7.5 | High | 5900 |
| 011 | Tomcat Manager Exposed | — | 7.3 | High | 8180 |
| 012 | Java RMI Registry Exposed | — | 6.8 | Medium | 1099 |
| 013 | NFS World-Readable Export | — | 5.0 | Medium | 2049 |
| 014 | Telnet Service Running | — | 6.4 | Medium | 23 |

---

## 3. Nikto Web Server Scan

### Command Executed
```bash
nikto -h http://192.168.56.104 -output logs/nikto_scan.txt
```

> `[SCREENSHOT: Nikto v2.6.0 web server scan against Apache 2.2.8 on port 80]`

![alt text](../assets/nikto_scan.png)

### Key Findings

```
+ Target Host: 192.168.56.104
+ Target Port: 80
+ GET /: Retrieved x-powered-by header: PHP/5.2.4-2ubuntu5.10
+ HEAD Apache/2.2.8 appears to be outdated (current is at least 2.4.66)
+ HEAD PHP/5.2.4-2ubuntu5.10 appears to be outdated (current is at least 8.5.1)
+ GET /icons/: Directory indexing found
+ TRACE /: HTTP TRACE method is active — host is vulnerable to XST
+ GET /phpinfo.php: phpinfo() output exposed — system info leak
+ GET /doc/: The /doc/ directory is browsable
+ GET /: Missing security headers: CSP, HSTS, X-Content-Type-Options, Referrer-Policy
+ GET /phpMyAdmin/changelog.php: phpMyAdmin exposed — database management accessible
+ GET /phpMyAdmin/ChangeLog: ETags reveal inode information (CVE-2003-1418)
```

### Summary of Issues Found

| Issue | Detail | Risk |
|-------|--------|------|
| Outdated Apache | 2.2.8 (EOL) | High |
| Outdated PHP | 5.2.4 (EOL) | High |
| Directory indexing | `/icons/`, `/doc/`, `/test/` | Medium |
| HTTP TRACE enabled | XST attack vector | Medium |
| Missing security headers | CSP, HSTS, X-Frame-Options | Medium |
| phpMyAdmin exposed | No access control | High |
| phpinfo() accessible | Full system info leak | Medium |

---

## 4. OWASP ZAP Web Application Scan

### Scan Configuration

| Setting | Value |
|---------|-------|
| Target URL | http://192.168.56.104 |
| Spider | Full crawl of all linked pages |
| Active Scan | Enabled — injects test payloads |

### Activities Performed

1. Spider scan — crawled all pages, discovered hidden endpoints
2. Active scan — injected SQL, XSS, path traversal, command injection payloads
3. Vulnerability detection — auto-classified findings by severity

> `[SCREENSHOT: OWASP ZAP 2.17.0 automated scan setup targeting Mutillidae]`

![alt text](../assets/zap_automated_scan.png)

> `[SCREENSHOT: ZAP spider + active scan running with full alerts list]`

![alt text](../assets/zap_active_scan.png)

> `[SCREENSHOT: ZAP RCE finding detail — CVE-2012-1823 PHP CGI confirmed]`

![alt text](../assets/zap_rce_finding.png)

### Vulnerability Summary

| Vulnerability | Count | Severity |
|---------------|-------|----------|
| Cross-Site Scripting (XSS) | 25 | High |
| SQL Injection (MySQL + SQLite) | Multiple | High |
| Path Traversal | 20 | High |
| Remote Code Execution (CVE-2012-1823) | Confirmed | Critical |
| Command Injection | Multiple | High |
| Missing CSRF Tokens | Multiple | Medium |
| Information Disclosure | Multiple | Low |

---

## Prioritisation Summary

### Critical — Immediate Action Required

| Vulnerability | CVSS | Port | Recommended Action |
|---------------|------|------|--------------------|
| vsftpd 2.3.4 Backdoor | 10.0 | 21 | Upgrade vsftpd immediately |
| Samba RCE | 10.0 | 445 | Patch or disable Samba |
| UnrealIRCd Backdoor | 10.0 | 6667 | Remove service |
| PHP CGI RCE | 9.8 | 80 | Patch PHP, disable CGI handler |
| DistCC RCE | 9.3 | 3632 | Disable distccd |
| MySQL No Password | 8.5 | 3306 | Set strong root password |
| rlogin/rexec | 9.0 | 512–514 | Disable deprecated r-services |

### High — Remediate Within 7 Days

| Vulnerability | CVSS | Port | Recommended Action |
|---------------|------|------|--------------------|
| VNC No Auth | 7.5 | 5900 | Require VNC password |
| Tomcat Manager | 7.3 | 8180 | Restrict access, change credentials |
| SQL Injection | 8.8 | 80 | Parameterise all queries |
| XSS | 7.4 | 80 | Sanitise all user inputs |

---
