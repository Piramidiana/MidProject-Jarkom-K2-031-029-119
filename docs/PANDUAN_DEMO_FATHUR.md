# PANDUAN LENGKAP DEMO & TROUBLESHOOTING — FATHUR (5027251029)
**Mid-Project Komunikasi Data dan Jaringan Komputer — Kelompok 2 (K2)**
**Tanggung Jawab**: Soal 3 (Routing Statis & CIDR), Soal 5 (NAT Overload), Soal 7 (OSPF & Failover L7)

---

## BAGIAN 1: SCRIPT DEMO UJI 1 (PRESENTASI KELOMPOK)

> **File Packet Tracer yang Disiapkan**:
> 1. `artefacts/K2_CIDR.pkt` (untuk demo Soal 3 dan Soal 5)
> 2. `artefacts/K2_CIDR_OSPF.pkt` (untuk demo Soal 7 OSPF & Failover)

---

### DEMO 1: SOAL 3 — ROUTING STATIS & EFISIENSI CIDR SUPERNETTING

#### 1. Pembukaan & Konsep Teknis (Bicara ke Dosen/Asisten)
> *"Selamat pagi/siang Bapak/Ibu Asisten. Saya Muhammad Rifqi Fathurrahman (Fathur), akan mendemonstrasikan implementasi Routing Statis dengan hierarki CIDR Supernetting di jaringan Smart City Surabaya Kelompok 2.*
>
> *Pada topologi K2 dengan base network `10.16.0.0/14`, kami mengelompokkan subnet-subnet wilayah ke dalam blok supernet CIDR yang rapi. Tanpa CIDR, Router Pusat harus menyimpan minimal 12 entri rute VLSM untuk seluruh LAN cabang. Namun dengan prinsip CIDR Supernetting, Router Pusat cukup menyimpan **5 entri rute ringkas** menuju router wilayah, ditambah 1 default route ke ISP. Ini menghemat memori routing table hingga lebih dari 58% dan mempercepat proses lookup forwarding paket."*

#### 2. Langkah Demo & Perintah CLI
1. Buka jendela CLI **R-PUSAT**:
   ```text
   enable
   show ip route static
   ```
   **Tunjukkan ke Asisten**:
   - `10.16.0.0/18` via `10.16.33.2` (Mencakup CCTV & Server DC)
   - `10.16.64.0/18` via `10.16.97.2` (Mencakup WiFi & Puskesmas Barat)
   - `10.16.128.0/18` via `10.16.164.2` (Mencakup WiFi & Sensor Banjir Timur)
   - `10.16.192.0/18` via `10.16.224.130` (Mencakup WiFi & PJU Selatan)
   - `10.18.0.0/16` via `10.18.130.2` (Mencakup seluruh Surabaya Utara & Pelabuhan)
   - `S* 0.0.0.0/0` via `203.0.113.9` (Default route ke R-ISP)

2. Buka jendela CLI **R-UTARA**:
   ```text
   show ip route static
   ```
   **Tunjukkan**:
   - `10.18.64.0/18` via `10.18.96.130` (Supernet ke arah Pelabuhan)
   - `S* 0.0.0.0/0` via `10.18.130.1` (Default route ke R-PUSAT)

3. Uji Konektivitas Antar-Wilayah:
   Buka CLI **R-BARAT**, jalankan ping ke wilayah yang berbeda:
   ```text
   ping 10.16.32.1
   ping 10.18.96.1
   ```
   **Hasil**: Success rate is 100 percent (5/5).

---

### DEMO 2: SOAL 5 — NAT OVERLOAD (PAT) & ISOLASI PRIVAT ISP

#### 1. Pembukaan & Konsep Teknis (Bicara ke Dosen/Asisten)
> *"Selanjutnya untuk Soal 5, kami mengonfigurasi NAT Overload (Port Address Translation) di Gateway R-PUSAT agar ribuan host internal pemerintah kota dapat mengakses jaringan internet publik menggunakan satu alamat IP publik `203.0.113.10` pada antarmuka GigabitEthernet0/0.*
>
> *Sesuai aturan keamanan dan arsitektur RFC 1918, Router ISP sama sekali tidak memiliki tabel rute ke alamat privat `10.16.0.0/14`. Komunikasi ke internet hanya bisa terjadi jika proses translasi NAT di R-PUSAT berjalan dengan sempurna."*

#### 2. Langkah Demo & Perintah CLI
1. Buka CLI **R-ISP** terlebih dahulu:
   ```text
   show ip route
   ```
   **Poin Penting untuk Ditegaskan**:
   Tunjukkan bahwa di R-ISP **hanya ada network publik** `203.0.113.8/30` dan `8.8.8.0/24`. Sama sekali tidak ada rute `10.16.x.x` atau `10.18.x.x`. Ini membuktikan jaringan ISP bersih dari rute privat.

2. Buka CLI **R-BARAT** (atau PC klien internal, misal PC-PUSKESMAS-BARAT):
   ```text
   ping 8.8.8.8
   ```
   **Hasil**: Berhasil terhubung ke Server Internet (`8.8.8.8`)!

3. Kembali ke CLI **R-PUSAT**, periksa tabel translasi aktif:
   ```text
   show ip nat translations
   ```
   **Tunjukkan ke Asisten**:
   ```text
   Pro  Inside global         Inside local          Outside local      Outside global
   icmp 203.0.113.10:1        10.16.97.2:1          8.8.8.8:1          8.8.8.8:1
   ```
   *Jelaskan: "Alamat privat 10.16.97.2 ditranslasikan menjadi Inside Global 203.0.113.10 dengan mapping port/ICMP identifier dinamis."*

4. Tampilkan konfigurasi NAT di **R-PUSAT**:
   ```text
   show running-config | include nat
   ```
   Tunjukkan:
   - `ip nat inside source list 1 interface GigabitEthernet0/0 overload`
   - Access-list 1: `permit 10.16.0.0 0.3.255.255` (mencakup seluruh blok /14)
   - Interface inside: G0/1, G0/2, S0/1/0, S0/1/1, S0/2/0, S0/2/1, S0/3/0.
   - Interface outside: G0/0.

---

### DEMO 3: SOAL 7 — OSPF DYNAMIC ROUTING & FAILOVER REDUNDANSI L7

#### 1. Pembukaan & Konsep Teknis (Bicara ke Dosen/Asisten)
> *"Untuk Soal 7, kami beralih ke dynamic routing OSPF Area 0 (file `K2_CIDR_OSPF.pkt`). Kami menambahkan link redundansi L7 berkabel Serial antara R-DC dan R-UTARA (`10.16.33.4/30`).*
>
> *Kami juga mengamankan jaringan dengan mengonfigurasi `passive-interface` pada seluruh port LAN host agar tidak membocorkan paket OSPF Hello ke segmen pengguna, serta menyebarkan default route internet secara otomatis dari R-PUSAT menggunakan perintah `default-information originate always`.*
>
> *Sekarang kami akan mendemonstrasikan ketahanan jaringan: ketika link utama L2 (R-PUSAT ke R-UTARA) putus, OSPF secara otomatis melakukan konvergensi dan mengalihkan paket melalui jalur redundansi L7 lewat R-DC tanpa menghentikan layanan."*

#### 2. Langkah Demo & Perintah CLI
1. Buka CLI **R-PUSAT**:
   ```text
   show ip ospf neighbor
   ```
   **Tunjukkan**: Ada 5 tetangga OSPF (R-DC, R-BARAT, R-UTARA, R-TIMUR, R-SELATAN), semuanya berstatus **FULL/-**.

2. Buka CLI **R-PELABUHAN**:
   ```text
   show ip route ospf
   ```
   **Tunjukkan**: Terdapat entri default route otomatis `O*E2 0.0.0.0/0 [110/1] via 10.18.96.129` yang berasal dari R-PUSAT.

3. **Uji Rute Normal**:
   Buka Command Prompt pada **PC-KANTOR-PELABUHAN** (atau CLI R-PELABUHAN):
   ```text
   traceroute 10.17.0.1
   ```
   **Output Kondisi Normal**:
   - Hop 1: `10.18.96.1` (Gateway R-PELABUHAN)
   - Hop 2: `10.18.96.129` (R-UTARA via link L6)
   - Hop 3: `10.18.130.1` (**R-PUSAT langsung via link utama L2**)
   - Hop 4: `10.17.0.1` (Tujuan Balai Kota)

4. **Simulasi Kegagalan Link Utama (Failover Test)**:
   Buka CLI **R-UTARA**, matikan interface link L2:
   ```text
   configure terminal
   interface Serial0/3/0
   shutdown
   end
   ```
   *Tunggu 3-5 detik hingga OSPF mendeteksi Dead Timer / Link Down.*

5. **Uji Rute Failover Otomatis**:
   Kembali ke Command Prompt **PC-KANTOR-PELABUHAN**, jalankan traceroute lagi:
   ```text
   traceroute 10.17.0.1
   ```
   **Output Kondisi Failover**:
   - Hop 1: `10.18.96.1` (Gateway R-PELABUHAN)
   - Hop 2: `10.18.96.129` (R-UTARA via L6)
   - Hop 3: `10.16.33.5` (**R-DC via link cadangan L7!**)
   - Hop 4: `10.16.33.1` (R-PUSAT via L1)
   - Hop 5: `10.17.0.1` (Tujuan Balai Kota)
   
   > *"Bisa kita saksikan, paket otomatis berbelok melewati Router Data Center (10.16.33.5) melalui link L7. Tidak ada packet loss permanen, membuktikan high-availability jaringan Smart City."*

6. **Pemulihan Jalur (Recovery)**:
   Di **R-UTARA**, hidupkan kembali interface L2:
   ```text
   configure terminal
   interface Serial0/3/0
   no shutdown
   end
   ```

---

## BAGIAN 2: PANDUAN TROUBLESHOOTING UJI 2 (10 MENIT INDIVIDU)

Jika dosen/asisten sengaja "merusak" jaringan Anda, ikuti **SOP 3 Menit Pertama**:

```text
[LANGKAH 1] Cek Interface fisik: show ip interface brief
            -> Cari port yang statusnya 'administratively down' atau 'down'.
[LANGKAH 2] Cek Tabel Rute:      show ip route
            -> Apakah ada route ke subnet tujuan? Apakah ada default route?
[LANGKAH 3] Cek Konfigurasi:     show running-config
```

### 1. KASUS TROUBLESHOOTING: NAT TIDAK BERJALAN (PC Gagal Ping 8.8.8.8)
* **Gejala**: Ping dari PC internal ke `8.8.8.8` timeout / Destination Host Unreachable.
* **Kemungkinan Kerusakan & Solusi di R-PUSAT**:
  1. **Interface NAT Inside / Outside tertukar atau hilang**:
     - Cek: `show running-config interface g0/0` (Harus `ip nat outside`).
     - Cek: Interface internal G0/1, G0/2, S0/1/0, S0/1/1, S0/2/0, S0/2/1, S0/3/0 (Semua harus `ip nat inside`).
     - Perbaikan: Masuk ke interface, ketik `ip nat inside` atau `ip nat outside`.
  2. **Access-List 1 salah atau hilang**:
     - Cek: `show access-lists`
     - Perbaikan:
       ```text
       conf t
       access-list 1 permit 10.16.0.0 0.3.255.255
       end
       ```
  3. **Command NAT Overload hilang**:
     - Perbaikan:
       ```text
       conf t
       ip nat inside source list 1 interface GigabitEthernet0/0 overload
       end
       ```
  4. **Interface G0/0 mati**:
     - Perbaikan: `int g0/0` lalu `no shutdown`.

### 2. KASUS TROUBLESHOOTING: ROUTING STATIS PUTUS
* **Gejala**: Ping antar-wilayah (misal Barat ke Timur) gagal.
* **Diagnosa Cepat**:
  1. Jalankan `traceroute <ip-tujuan>` dari host pengirim untuk melihat di router mana paket berhenti.
  2. Buka router tempat paket terhenti, jalankan `show ip route`.
* **Solusi**:
  - Jika tidak ada rute ke tujuan: Tambahkan static route yang hilang.
  - Jika rute ada tapi next-hop salah IP:
    ```text
    conf t
    no ip route <subnet-salah> <mask-salah> <next-hop-salah>
    ip route <subnet-benar> <mask-benar> <next-hop-benar>
    end
    ```
  - Pastikan di router cabang terdapat default route ke arah R-PUSAT: `ip route 0.0.0.0 0.0.0.0 <ip-pusat>`.

### 3. KASUS TROUBLESHOOTING: OSPF NEIGHBOR TIDAK TERBENTUK (NOT FULL)
* **Gejala**: `show ip ospf neighbor` kosong atau berstatus selain FULL.
* **Cek 4 Hal Utama**:
  1. **Passive interface salah pasang**:
     - Asisten sering memasang `passive-interface Serial...` pada link inter-router!
     - Perbaikan:
       ```text
       conf t
       router ospf 1
       no passive-interface Serial0/x/x
       end
       ```
  2. **Network statement salah atau subnet mask / wildcard salah**:
     - Contoh: `network 10.16.33.0 0.0.0.3 area 0`. Jika wildcard salah (misal `0.0.0.255`), interface serial bisa tidak ter-cover.
  3. **Area ID Mismatch**:
     - Pastikan seluruh router menggunakan `area 0`.
  4. **Interface Serial mati**:
     - Ketik `no shutdown` pada interface yang down.

### 4. KASUS TROUBLESHOOTING: FAILOVER L7 TIDAK BERJALAN
* **Gejala**: Saat L2 dimatikan, ping ke Balai Kota putus total dan tidak beralih ke L7.
* **Penyebab & Solusi**:
  - Periksa apakah link L7 (`10.16.33.4/30`) sudah diikutsertakan ke dalam proses OSPF di **R-DC** dan **R-UTARA**:
    ```text
    ! Di R-DC:
    router ospf 1
    network 10.16.33.4 0.0.0.3 area 0

    ! Di R-UTARA:
    router ospf 1
    network 10.16.33.4 0.0.0.3 area 0
    ```
  - Pastikan port serial L7 (R-DC S0/3/1 dan R-UTARA S0/2/1) berstatus `up up`.
