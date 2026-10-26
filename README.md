# Mid-Project Jarkom K2 – Implementasi Jaringan Smart City Kota Surabaya

Mid-Project Komunikasi Data dan Jaringan Komputer, Semester Gasal 2026/2027, Departemen Teknologi Informasi ITS.

## Anggota

| Nama                         | NRP        | Bagian                                                                |
| ---------------------------- | ---------- | --------------------------------------------------------------------- |
| Dian Piramidiana Rachmatika  | 5027251031 | Repo, tabel subnetting (soal 2), topologi (soal 1), dokumentasi akhir |
| Muhammad Rifqi Fathurrahman  | 5027251029 | Static routing (soal 3), NAT (soal 5), OSPF (soal 7), uji end-to-end  |
| Ronnin Raditya Putra Partono | 5027251119 | DHCP (soal 4), tunnel GRE + IPsec (soal 6), layanan aplikasi (soal 8) |

## Ringkasan pengalamatan

| Keperluan                                  | Alamat                                      |
| ------------------------------------------ | ------------------------------------------- |
| Base network kelompok (oktet ke-2 = 8 × 2) | 10.16.0.0                                   |
| Skema yang diimplementasikan               | CIDR                                        |
| Link R-PUSAT – R-ISP                       | 203.0.113.8/30                              |
| Server internet                            | 8.8.8.8/24, gateway 8.8.8.1                 |
| Tunnel GRE                                 | 192.168.2.0/30 (R-PUSAT .1, R-PELABUHAN .2) |
| Domain internal                            | k2.smartcity.lab                            |

## Struktur repo

| Folder                    | Isi                                               |
| ------------------------- | ------------------------------------------------- |
| `artefacts/`              | File Packet Tracer (.pkt) dan spreadsheet (.xlsx) |
| `assets/`                 | Screenshot bukti pengujian                        |
| `nodes/`                  | Running-config akhir tiap perangkat               |
| `perquest-scripts/soalN/` | Script konfigurasi per soal per perangkat         |
| `docs/`                   | Catatan serah-terima antar-anggota                |

Format nama screenshot: `JARKOM_MID_[nomor soal]_[isi].png`.

---

## Soal 1 – Perancangan Topologi

_Belum dikerjakan._

## Soal 2 – Tabel Subnetting

_Belum dikerjakan._

## Soal 3 – Static Routing

_Belum dikerjakan._

## Soal 4 – DHCP

_Belum dikerjakan._

## Soal 5 – NAT

_Belum dikerjakan._

## Soal 6 – Tunnel GRE

_Belum dikerjakan._

## Soal 7 – Dynamic Routing OSPF

_Belum dikerjakan._

## Soal 8 – Layanan Aplikasi Smart City

_Belum dikerjakan._
