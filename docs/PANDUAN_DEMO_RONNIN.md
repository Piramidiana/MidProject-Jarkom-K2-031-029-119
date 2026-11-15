# PANDUAN LENGKAP DEMO & TROUBLESHOOTING — RONNIN (5027251119)
**Mid-Project Komunikasi Data dan Jaringan Komputer — Kelompok 2 (K2)**
**Tanggung Jawab**: Soal 4 (DHCP Server & Relay), Soal 6 (GRE Tunnel + IPsec), Soal 8 (Layanan Aplikasi Smart City)

---

## BAGIAN 1: SCRIPT DEMO UJI 1 (PRESENTASI KELOMPOK)

> **File Packet Tracer yang Disiapkan**:
> - `artefacts/K2_CIDR.pkt` (digunakan untuk seluruh demo Soal 4, Soal 6, dan Soal 8)

---

### DEMO 1: SOAL 4 — LAYANAN DHCP SERVER & DHCP RELAY AGENT

#### 1. Pembukaan & Konsep Teknis (Bicara ke Dosen/Asisten)
> *"Selamat pagi/siang Bapak/Ibu Asisten. Saya Ronnin Raditya Putra Partono (Ronnin), akan mendemonstrasikan penyediaan layanan alamat IP dinamis (DHCP) di jaringan Smart City Surabaya Kelompok 2.*
>
> *Pada arsitektur kami, pembagian IP host dilakukan secara otomatis: seluruh router wilayah menjalankan DHCP Server lokal untuk LAN masing-masing dengan mengecualikan IP gateway (`.1`) dan IP server (`.2`).*
>
> *Khusus untuk subnet CCTV Kota (`10.16.0.0/19`), pengelolaan IP dipusatkan langsung di Server Data Center (`SRV-DC`). Karena paket broadcast DHCP Discover dari switch CCTV tidak dapat menembus router secara alami, kami mengonfigurasi **DHCP Relay Agent** menggunakan `ip helper-address 10.16.32.2` pada antarmuka GigabitEthernet0/0 Router Data Center."*

#### 2. Langkah Demo & Perintah CLI
1. **Verifikasi DHCP Pool di Router**:
   Buka CLI **R-BARAT** (atau router wilayah lainnya):
   ```text
   enable
   show running-config | section dhcp
   ```
   **Tunjukkan ke Asisten**:
   - `ip dhcp excluded-address 10.16.64.1` (Gateway aman)
   - Pool name `POOL-WIFI-BARAT`, network `10.16.64.0 255.255.224.0`, default-router `10.16.64.1`, dns-server `10.16.32.2`.

2. **Verifikasi Klien Menerima IP Otomatis**:
   - Buka **PC-PUSKESMAS-BARAT** → Tab **Desktop** → **IP Configuration**.
   - Tunjukkan tombol **DHCP** terpilih dan IP berhasil diperoleh secara otomatis (`10.16.96.2`, Mask `255.255.255.0`, Gateway `10.16.96.1`, DNS `10.16.32.2`).
   - Lakukan toggle: Klik **Static** lalu klik **DHCP** lagi → Muncul notifikasi *"DHCP request successful"*.

3. **Verifikasi DHCP Relay Agent (Subnet CCTV)**:
   - Buka CLI **R-DC**:
     ```text
     show running-config interface GigabitEthernet0/0
     ```
     **Tunjukkan**: Ada baris perintah `ip helper-address 10.16.32.2`.
   - Buka **SRV-DC** → Tab **Config** → Menu **DHCP**:
     **Tunjukkan**: Service DHCP dalam keadaan **ON**, pool `POOL-CCTV` terdefinisi dengan network `10.16.0.0`, gateway `10.16.0.1`, DNS `10.16.32.2`.
   - Buka **PC-CCTV-KOTA** → Tab **Desktop** → **IP Configuration**:
     **Tunjukkan**: PC CCTV berhasil mendapat IP `10.16.0.2` yang disuplai langsung oleh Server DC melewati router berkat DHCP relay.

---

### DEMO 2: SOAL 6 — SECURE INTER-OPD SITE-TO-SITE GRE TUNNEL + IPSEC

#### 1. Pembukaan & Konsep Teknis (Bicara ke Dosen/Asisten)
> *"Untuk Soal 6, kami membangun terowongan privat virtual (Site-to-Site GRE over IPsec) yang menghubungkan Kantor Balai Kota (R-PUSAT) langsung ke Otoritas Pelabuhan (R-PELABUHAN).*
>
> *Terowongan ini menggunakan subnet khusus `/30` yaitu `192.168.2.0/30`. Seluruh komunikasi data sensitif SCADA dan Kantor Pelabuhan diarahkan melewati antarmuka `Tunnel0`.*
>
> *Untuk meraih poin bonus keamanan, kami mengenkripsi tunnel menggunakan IPsec (ISAKMP AES-256 dan SHA Integrity) sehingga data terlindung dari penyadapan pada jaringan publik. Kami juga menerapkan prinsip **Longest Prefix Match (LPM)** agar rute tunnel tidak menimpa rute fisik underlay, mencegah fatal error recursive routing loop."*

#### 2. Langkah Demo & Perintah CLI
1. **Status Antarmuka Tunnel**:
   Buka CLI **R-PUSAT**:
   ```text
   show interfaces Tunnel0
   ```
   **Tunjukkan ke Asisten**:
   - `Tunnel0 is up, line protocol is up`
   - Internet address is `192.168.2.1/30`
   - Tunnel source `Serial0/2/0`, destination `10.18.96.130`
   - Tunnel protocol/transport `GRE/IP`, IPsec encrypted.

2. **Verifikasi Sesi Enkripsi IPsec**:
   Di **R-PUSAT**:
   ```text
   show crypto isakmp sa
   show crypto session
   ```
   **Tunjukkan**: Status sesi adalah `ACTIVE` / `QM_IDLE` dengan peer `10.18.96.130`.

3. **Uji Jalur Terowongan (Traceroute Bypass)**:
   Buka CLI **R-PUSAT** (atau PC Balai Kota), jalankan traceroute ke LAN Pelabuhan:
   ```text
   traceroute 10.18.64.1
   ```
   **Output**:
   ```text
     1 192.168.2.2 8 msec 9 msec 8 msec
     2 10.18.64.1  10 msec 10 msec 11 msec
   ```
   > *"Bisa dilihat bahwa lompatan pertama paket dari Balai Kota langsung menyentuh IP 192.168.2.2 (antarmuka tunnel R-PELABUHAN). Hop publik perantara seperti R-UTARA dilewati secara transparan di dalam enkripsi terowongan."*

---

### DEMO 3: SOAL 8 — LAYANAN APLIKASI SMART CITY (DNS, WEB, FTP, EMAIL)

#### 1. Pembukaan & Konsep Teknis (Bicara ke Dosen/Asisten)
> *"Terakhir untuk Soal 8, kami menyajikan integrasi layanan aplikasi inti Smart City berbasis domain resmi internal `k2.smartcity.lab`:*
> 1. *DNS Server terpusat di Server Data Center (`10.16.32.2`)*
> 2. *Portal Web Resmi Smart City di Server Balai Kota (`10.17.8.2`)*
> 3. *Storage Server FTP untuk rekaman CCTV di Server Data Center (`10.16.32.2`)*
> 4. *Email Server POP3/SMTP untuk komunikasi antar-OPD di Server Data Center (`10.16.32.2`)."*

#### 2. Langkah Demo & Pembuktian
1. **DNS Server (SRV-DC - 10.16.32.2)**:
   - Buka **SRV-DC** → Tab **Config** → Menu **DNS**.
   - Tunjukkan service **ON** dan daftar Resource Record (RR):
     - `www.k2.smartcity.lab` → `10.17.8.2` (Web Portal)
     - `ftp.k2.smartcity.lab` → `10.16.32.2` (Storage FTP)
     - `mail.k2.smartcity.lab` → `10.16.32.2` (Mail Server)

2. **Web Server Portal Smart City (SRV-BALAIKOTA - 10.17.8.2)**:
   - Buka perangkat klien manapun di ujung jaringan, misal **PC-PUSKESMAS-BARAT** atau **LAPTOP-WIFI-UTARA**.
   - Buka tab **Desktop** → **Web Browser**.
   - Ketik URL: `http://www.k2.smartcity.lab` lalu tekan **Go**.
   - **Hasil Tampil**: Halaman HTML resmi portal Smart City dengan judul *"Portal Smart City Surabaya"* dan tertulis jelas identitas *"Kelompok 2 (K2)"*.

3. **FTP Storage Server CCTV (SRV-DC - 10.16.32.2)**:
   - Buka **PC-CCTV-KOTA** → Tab **Desktop** → **Command Prompt**.
   - Jalankan perintah:
     ```text
     ftp ftp.k2.smartcity.lab
     ```
   - Masukkan username: `cctv`
   - Masukkan password: `cctv123`
   - Ketik:
     ```text
     dir
     quit
     ```
   - **Tunjukkan**: Muncul pesan *"Logged in"* dan daftar file direktori FTP sukses terbaca.

4. **Email Server Inter-OPD (SRV-DC - 10.16.32.2)**:
   - Buka **PC-PEGAWAI-BK** → Tab **Desktop** → **Email** → Klik **Compose**:
     - To: `pelabuhan@k2.smartcity.lab`
     - Subject: `Laporan Smart City Surabaya`
     - Body: `Koordinasi arus logistik pelabuhan dan balai kota berjalan lancar.`
     - Klik **Send** → Di status bar bawah muncul *"Send Success"*.
   - Buka **PC-KANTOR-PELABUHAN** → Tab **Desktop** → **Email**:
     - Klik tombol **Receive**.
     - **Tunjukkan**: Email dari `pegawai@k2.smartcity.lab` langsung masuk ke inbox dan dapat dibaca secara utuh!

---

## BAGIAN 2: PANDUAN TROUBLESHOOTING UJI 2 (10 MENIT INDIVIDU)

Jika dosen/asisten sengaja "merusak" jaringan Anda, ikuti **SOP 3 Menit Pertama**:

```text
[LANGKAH 1] Cek IP Configuration Klien: Apakah DHCP dapat IP atau 169.254.x.x (APIPA)?
            -> Cek apakah DNS Server terisi 10.16.32.2 atau kosong (255.255.255.255).
[LANGKAH 2] Cek Interface Status:       show ip interface brief di router terkait.
[LANGKAH 3] Cek Service di Server:       Buka Server -> Config -> Cek tombol ON/OFF service.
```

### 1. KASUS TROUBLESHOOTING: PC MENDAPATKAN IP APIPA (169.254.x.x)
* **Gejala**: PC klien gagal meminta IP ke DHCP, muncul IP 169.254.x.x.
* **Pemeriksaan & Solusi**:
  1. **Jika terjadi pada PC biasa (WiFi / Puskesmas / Sensor)**:
     - Buka router wilayah tersebut, ketik: `show running-config | section dhcp`
     - Cek apakah network atau default-router salah ketik.
     - Contoh perbaikan:
       ```text
       conf t
       ip dhcp pool POOL-PUSKESMAS-BARAT
       network 10.16.96.0 255.255.255.0
       default-router 10.16.96.1
       dns-server 10.16.32.2
       end
       ```
     - Toggle kembali DHCP di PC klien.
  2. **Jika terjadi pada PC-CCTV-KOTA**:
     - Buka **R-DC**, cek interface G0/0:
       ```text
       show running-config interface GigabitEthernet0/0
       ```
     - Jika baris `ip helper-address 10.16.32.2` hilang:
       ```text
       conf t
       interface GigabitEthernet0/0
       ip helper-address 10.16.32.2
       end
       ```
     - Buka **SRV-DC** → Tab **Config** → **DHCP**: Pastikan Service berstatus **ON**.

### 2. KASUS TROUBLESHOOTING: GRE TUNNEL DOWN / LINE PROTOCOL DOWN
* **Gejala**: `show interfaces Tunnel0` menampilkan `Tunnel0 is up, line protocol is down`.
* **Penyebab & Solusi**:
  1. **Underlay tidak bisa di-reach (Destination Unreachable)**:
     - Tes ping dari R-PUSAT ke IP destination tunnel: `ping 10.18.96.130`.
     - Jika ping gagal, periksa interface fisik di jalur L2 & L6 (apakah ada interface yang shutdown?).
  2. **Tunnel Source salah format di Packet Tracer**:
     - Ingat! Di Packet Tracer Cisco 2911, tunnel source **harus berupa nama interface** (bukan IP):
       ```text
       conf t
       interface Tunnel0
       tunnel source Serial0/2/0
       tunnel destination 10.18.96.130
       end
       ```
  3. **Recursive Routing Crash**:
     - Jangan pernah merouting seluruh network underlay melewati `Tunnel0`. Pastikan hanya LAN tujuan (`10.18.64.0/19` dan `10.18.96.0/25`) yang diarahkan ke tunnel.

### 3. KASUS TROUBLESHOOTING: IPSEC ENKRIPSI GAGAL (CRYPTO SESSION DOWN)
* **Gejala**: Paket tunnel lewat tetapi tidak terenkripsi, atau status crypto session kosong.
* **Solusi**:
  - Samakan konfigurasi preshared-key di R-PUSAT dan R-PELABUHAN:
    ```text
    crypto isakmp key cisco123 address <ip-peer-lawan>
    ```
  - Pastikan transform-set namanya persis sama:
    ```text
    crypto ipsec transform-set TS esp-aes esp-sha-hmac
    ```
  - Pastikan crypto profile menempel di Tunnel0:
    ```text
    interface Tunnel0
    tunnel protection ipsec profile IPSEC-TUNNEL
    ```

### 4. KASUS TROUBLESHOOTING: WEB / FTP / EMAIL "SERVER NOT FOUND" ATAU DNS GAGAL
* **Gejala**: Browser muncul "Server Not Found" atau pesan error DNS resolving.
* **Urutan Diagnosa 2 Menit**:
  1. **Cek DNS Setting pada PC Klien**:
     - Buka Desktop PC → IP Configuration.
     - Apakah kolom **DNS Server** kosong atau bernilai `255.255.255.255`?
     - Jika iya, ketikkan IP DNS Server: `10.16.32.2` (atau perbaiki perintah `dns-server 10.16.32.2` pada DHCP pool di router).
  2. **Cek Service DNS di SRV-DC**:
     - Buka **SRV-DC** → **Config** → **DNS**.
     - Pastikan tombol service dalam keadaan **ON**.
     - Pastikan tidak ada typo pada nama domain (`www.k2.smartcity.lab` mengarah ke `10.17.8.2`).
  3. **Cek Service Web di SRV-BALAIKOTA**:
     - Buka **SRV-BALAIKOTA** → **Config** → **HTTP**.
     - Pastikan HTTP dan HTTPS dalam keadaan **ON**.
  4. **Cek Akun FTP**:
     - Buka **SRV-DC** → **Config** → **FTP**: Service ON, user `cctv`, pass `cctv123`, seluruh izin (Read/Write/Delete/Rename/List) dicentang.
  5. **Cek Pengaturan Email Client**:
     - Buka Email Client di PC → Klik **Configure Mail**:
     - Incoming & Outgoing Mail Server harus: `mail.k2.smartcity.lab` (atau langsung IP `10.16.32.2`).
     - Pastikan domain name di Server DC adalah `k2.smartcity.lab` dan user account sudah terdaftar.
