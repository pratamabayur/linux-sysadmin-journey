# E10 — Instalasi & Verifikasi Paket Software

**Kategori:** Package Management

## Tujuan
Melakukan instalasi dan verifikasi paket software melalui package manager (dnf).

## Command & Output

**Refresh cache repo:**
```bash
$ sudo dnf makecache
cloudflared-stable                                    4.3 kB/s | 1.5 kB     00:00
Docker CE Stable - x86_64                              41 kB/s | 2.0 kB     00:00
Extra Packages for Enterprise Linux 9 - x86_64         19 kB/s |  11 kB     00:00
Rocky Linux 9 - BaseOS                                5.6 kB/s | 4.3 kB     00:00
Rocky Linux 9 - AppStream                              4.9 kB/s | 4.8 kB     00:00
Rocky Linux 9 - Extras                                 3.4 kB/s | 3.1 kB     00:00
Metadata cache created.
```

**Pencarian paket normal:**
```bash
$ dnf search htop
======== Name Exactly Matched: htop ========
htop.x86_64 : Interactive process viewer

$ dnf search nodejs
=========== Name Exactly Matched: nodejs ===========
nodejs.x86_64 : JavaScript runtime
... (beberapa paket terkait: nodejs-devel, nodejs-docs, nodejs-libs, dll)

$ dnf search cloudflared
=========== Name Exactly Matched: cloudflared ===========
cloudflared.386 : Cloudflare Tunnel daemon
cloudflared.aarch64 : Cloudflare Tunnel daemon
cloudflared.x86_64 : Cloudflare Tunnel daemon
```

**Eksperimen pencarian multi-kata (studi kasus argumen shell):**
```bash
$ dnf search Rocky Linux 9
# "Rocky", "Linux", "9" diperlakukan sebagai 3 kata kunci TERPISAH
# hanya 1 match ditemukan (kebetulan mengandung ketiga kata tsb)
=== Name & Summary Matched: 9, Rocky, Linux ===
lorax-templates-rocky.noarch : Rocky Linux 9 build templates for lorax and livemedia-creator

$ rpm -q Rocky Linux 9
package Rocky is not installed
package Linux is not installed
package 9 is not installed

$ rpm -q "Rocky Linux 9"
package Rocky Linux 9 is not installed
```

## Analisis

**Repo pihak ketiga terkonfirmasi aktif:** `cloudflared` tersedia dalam 3 arsitektur (386, aarch64, x86_64) — mengonfirmasi repo `cloudflared-stable` yang terdaftar di sistem benar-benar berfungsi dan terpelihara oleh vendor untuk multi-platform.

**Studi kasus penting — cara shell dan `rpm`/`dnf` menangani argumen multi-kata:**

| Command | Hasil | Penjelasan |
|---|---|---|
| `dnf search Rocky Linux 9` | 1 match (kebetulan) | 3 argumen terpisah diperlakukan sebagai 3 kata kunci independen |
| `rpm -q Rocky Linux 9` | 3x "not installed" | Dicoba sebagai **3 nama paket terpisah**: "Rocky", "Linux", "9" — semuanya memang tidak ada |
| `rpm -q "Rocky Linux 9"` | 1x "not installed" | Dengan kutip, dianggap 1 frasa — tapi tetap gagal karena bukan nama paket RPM yang valid |

**Kesimpulan konseptual:** `rpm -q` dan `dnf search` dirancang untuk mencari **nama paket individual**, bukan nama produk/OS secara keseluruhan. "Rocky Linux 9" bukan satu paket — melainkan kumpulan ratusan paket (kernel, systemd, bash, dll) yang bersama-sama membentuk sebuah distribusi. Untuk memverifikasi identitas OS, command yang tepat adalah `cat /etc/os-release` atau `hostnamectl` (lihat [E05](E05-server-specs.md)), bukan `rpm -q`.

## Poin Penting

- Alur wajib topik ini: **cari (`dnf search`) → install (`dnf install`) → verifikasi (`rpm -q`)**.
- Gunakan tanda kutip (`"frasa dengan spasi"`) jika ingin argumen multi-kata diperlakukan sebagai satu kesatuan, bukan kata kunci terpisah — meski ini tidak relevan untuk `rpm -q` karena ia memang hanya menerima nama paket individual.
- Repo tambahan (di luar Rocky default) mencerminkan software non-default yang sengaja diinstal untuk kebutuhan spesifik server — konsisten dengan temuan proses (`cloudflared`, `dockerd`) di topik-topik sebelumnya.

---

## Bagian 2: Verifikasi Paket Terinstal (Studi Kasus `cloudflared`)

## Command & Output

```bash
$ dnf list intalled | grep cloudflared
Error: No matching Packages to list
```
*(Typo: seharusnya `installed`. Error yang muncul cukup ambigu — bukan "command not found", melainkan dnf tetap berjalan namun tidak menemukan match apa pun.)*

**Verifikasi alternatif — cek binary & versi langsung:**
```bash
$ which cloudflared && cloudflared --version
/usr/bin/cloudflared
cloudflared version 2026.8.2 (built 2026-08-14-12:17 UTC)
```

**Verifikasi paling lengkap — `dnf info`:**
```bash
$ dnf info cloudflared
Installed Packages
Name         : cloudflared
Version      : 2026.8.2
Repository   : @System
From repo    : cloudflared-stable

Available Packages
Name         : cloudflared
Version      : 2026.9.1
Repository   : cloudflared-stable
Architecture : x86_64 / 386 / aarch64 (tersedia multi-platform)
```

## Analisis

**Temuan penting:** `dnf info` menunjukkan versi **terinstal** (`2026.8.2`) berbeda dengan versi **tersedia di repo** (`2026.9.1`) — mengindikasikan ada pembaruan yang belum diterapkan pada server ini. Update dapat dilakukan dengan:
```bash
sudo dnf update cloudflared -y
```

**Perbandingan metode verifikasi:**

| Metode | Informasi yang didapat | Kelengkapan |
|---|---|---|
| `which` + `--version` | Lokasi binary, versi terinstal | Dasar |
| `rpm -q` | Nama, versi, release, arsitektur | Cukup lengkap |
| `dnf info` | Semua di atas + status update tersedia + repo asal + lisensi | **Paling lengkap** |

## Poin Penting

- Typo pada command (`intalled` vs `installed`) tidak selalu menghasilkan error yang jelas ("command not found") — kadang tool tetap berjalan namun dengan hasil yang salah/nihil, sehingga penting untuk selalu memeriksa kembali penulisan command saat hasil tidak sesuai ekspektasi.
- `dnf info <paket>` adalah metode verifikasi **paling komprehensif** karena sekaligus menunjukkan apakah ada versi lebih baru yang tersedia — informasi yang tidak didapat dari `rpm -q` maupun `which`.
- Software dari repo pihak ketiga (seperti `cloudflared`) perlu dipantau pembaruannya secara manual/berkala, karena tidak selalu ter-update otomatis bersamaan dengan paket sistem Rocky Linux lainnya.
