# Serah-terima Piramid → Fathur & Ronnin (topologi + IP selesai)

File: `artefacts/K2_CIDR.pkt`. Tabel alamat: `artefacts/K2_Subnetting_SmartCity.xlsx` (sheet CIDR & Supernet).
Script config router: `perquest-scripts/soal1/`. **Sudah terkonfigurasi:** hostname, IP semua interface, description, clock rate, dan tersimpan (startup-config). **Belum ada rute apa pun** (static/OSPF/NAT/DHCP/GRE).

## Status uji (semua berhasil)
- `show cdp neighbors` di R-PUSAT: 5 router wilayah cocok dengan port rencana.
- Ping antar-router bertetangga: L1–L6 dan R-PUSAT ↔ R-ISP berhasil.
- R-ISP → SRV-INTERNET, R-PUSAT → SRV-BALAIKOTA, R-DC → SRV-DC berhasil.
- Semua port G0/0 dan G0/1 router `up up`; 4 laptop terhubung ke AP wilayahnya.

## Peta port backbone
| Link | Router atas (DCE) | Router bawah | Network /30 | Atas | Bawah |
|---|---|---|---|---|---|
| ISP | R-PUSAT G0/0 | R-ISP G0/0 | 203.0.113.8/30 | .10 (R-PUSAT) | .9 (R-ISP) |
| L1 | R-PUSAT S0/1/0 | R-DC S0/3/0 | 10.16.33.0/30 | .1 | .2 |
| L5 | R-PUSAT S0/1/1 | R-BARAT S0/3/0 | 10.16.97.0/30 | .1 | .2 |
| L2 | R-PUSAT S0/2/0 | R-UTARA S0/3/0 | 10.18.130.0/30 | .1 | .2 |
| L4 | R-PUSAT S0/2/1 | R-TIMUR S0/3/0 | 10.16.164.0/30 | .1 | .2 |
| L3 | R-PUSAT S0/3/0 | R-SELATAN S0/3/0 | 10.16.224.128/30 | .129 | .130 |
| L6 | R-UTARA S0/2/0 | R-PELABUHAN S0/3/0 | 10.18.96.128/30 | .129 | .130 |

Port kosong: R-PUSAT S0/3/1, **R-DC S0/3/1 dan R-UTARA S0/2/1 (disiapkan untuk L7, soal 7)**.

## LAN (gateway = host pertama)
| Router | Port | LAN | Network | Gateway | Perangkat |
|---|---|---|---|---|---|
| R-PUSAT | G0/1 | Pegawai Balai Kota | 10.17.0.0/21 | 10.17.0.1 | PC-PEGAWAI-BK (DHCP) |
| R-PUSAT | G0/2 | Server Balai Kota | 10.17.8.0/27 | 10.17.8.1 | SRV-BALAIKOTA **10.17.8.2** |
| R-DC | G0/0 | CCTV Kota | 10.16.0.0/19 | 10.16.0.1 | PC-CCTV-KOTA (DHCP dari SRV-DC) |
| R-DC | G0/1 | Server dan Storage DC | 10.16.32.0/24 | 10.16.32.1 | SRV-DC **10.16.32.2** |
| R-BARAT | G0/0 | WiFi Publik Barat | 10.16.64.0/19 | 10.16.64.1 | AP + LAPTOP-WIFI-BARAT (DHCP) |
| R-BARAT | G0/1 | Puskesmas dan Sekolah Barat | 10.16.96.0/24 | 10.16.96.1 | PC-PUSKESMAS-BARAT (DHCP) |
| R-UTARA | G0/0 | WiFi Publik Utara | 10.18.0.0/18 | 10.18.0.1 | AP + LAPTOP-WIFI-UTARA (DHCP) |
| R-UTARA | G0/1 | Sensor Lalu Lintas Utara | 10.18.128.0/23 | 10.18.128.1 | PC-SENSOR-UTARA (DHCP) |
| R-TIMUR | G0/0 | WiFi Publik Timur | 10.16.128.0/19 | 10.16.128.1 | AP + LAPTOP-WIFI-TIMUR (DHCP) |
| R-TIMUR | G0/1 | Sensor Banjir Timur | 10.16.160.0/22 | 10.16.160.1 | PC-BANJIR-TIMUR (DHCP) |
| R-SELATAN | G0/0 | WiFi Publik Selatan | 10.16.192.0/19 | 10.16.192.1 | AP + LAPTOP-WIFI-SELATAN (DHCP) |
| R-SELATAN | G0/1 | PJU Pintar Selatan | 10.16.224.0/25 | 10.16.224.1 | PC-PJU-SELATAN (DHCP) |
| R-PELABUHAN | G0/0 | Sensor dan SCADA Pelabuhan | 10.18.64.0/19 | 10.18.64.1 | PC-SCADA-PELABUHAN (DHCP) |
| R-PELABUHAN | G0/1 | Kantor Pelabuhan | 10.18.96.0/25 | 10.18.96.1 | PC-KANTOR-PELABUHAN (DHCP) |

Internet: SRV-INTERNET **8.8.8.8/24**, gateway 8.8.8.1 (R-ISP G0/1).

## Untuk Fathur (soal 3, 5, 7)
- Rute ringkas CIDR ada di sheet **Supernet** (bagian "Rencana rute ringkas"). R-PUSAT cukup 5 rute + default: 10.16.0.0/18 via 10.16.33.2, 10.16.64.0/18 via 10.16.97.2, 10.16.128.0/18 via 10.16.164.2, 10.16.192.0/18 via 10.16.224.130, 10.18.0.0/16 via 10.18.130.2, default via 203.0.113.9. R-UTARA: 10.18.64.0/18 via 10.18.96.130.
- Inside NAT di R-PUSAT: G0/1, G0/2, S0/1/0, S0/1/1, S0/2/0, S0/2/1, S0/3/0. Outside: G0/0. ACL internal: 10.16.0.0 0.3.255.255 (seluruh blok /14).
- L7 pakai port R-DC S0/3/1 ↔ R-UTARA S0/2/1, alamat dari ruang kosong (misal 10.16.34.0/30 masih kosong di sisi DC).

## Untuk Ronnin (soal 4, 6, 8)
- Server statis: SRV-DC 10.16.32.2, SRV-BALAIKOTA 10.17.8.2. Saat membuat pool DHCP, **exclude gateway (.1) dan alamat server (.2)**.
- CCTV: pool di SRV-DC untuk 10.16.0.0/19, `ip helper-address 10.16.32.2` di **R-DC G0/0**.
- Laptop WiFi memakai DHCP lewat interface Wireless0.
- Tunnel GRE: R-PUSAT dan R-PELABUHAN adalah ISR 2911 (lisensi securityk9 belum diaktifkan; perlu untuk bonus IPsec).

## Aturan bersama
- File `.pkt` dipegang bergantian: umumkan 🔒 sebelum membuka dan 🔓 setelah push.
- Jangan ubah IP interface atau pindah kabel tanpa memberi tahu grup.
- Setiap selesai: `copy running-config startup-config` di router, Ctrl+S, lalu commit + push.
