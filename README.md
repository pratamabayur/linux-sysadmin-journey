# Linux System Administration Journey — Rocky Linux 9

Dokumentasi hands-on pembelajaran Linux system administration, dipraktikkan langsung di server Rocky Linux 9 (bukan sandbox), mencakup 3 tingkat kompetensi: **Easy**, **Medium**, dan **Hard**.

> ⚠️ **Catatan keamanan:** Seluruh IP address, hostname, username, password, dan token pada dokumentasi ini adalah **placeholder/disamarkan**. Struktur command dan output dipertahankan seaslinya untuk keperluan pembelajaran.

## Tentang Proyek Ini

Alih-alih belajar lewat tutorial pasif, proyek ini disusun dari praktik langsung di server yang punya karakteristik dunia nyata: multi-user, ada riwayat konfigurasi sebelumnya, dan tidak steril seperti sandbox latihan pada umumnya. Setiap topik didokumentasikan dengan:
- **Command** yang digunakan
- **Output aktual** (disamarkan)
- **Analisis/insight** dari hasil tersebut
- **Troubleshooting**, jika ada kendala nyata yang ditemui

## Progress

| Level | Status | Jumlah Topik |
|---|---|---|
| 🟢 Easy | ✅ Selesai | 16/16 |
| 🟡 Medium | 🔄 Sedang berjalan | 0/13 |
| 🔴 Hard | ⏳ Belum dimulai | 0/5 |

## Struktur Repo

```
├── easy/      → E01-E16: Dasar sistem, monitoring, user & SSH, networking
├── medium/    → M01-M13: Security, firewall, SELinux, NFS, ACL, disk quota
└── hard/      → H01-H05: LVM, mountpoint, NTP/chrony, DNS client, NFS client
```

## Daftar Isi — Level Easy

| Kode | Topik |
|---|---|
| [E01](E01-hostname.md) | Mengubah hostname secara permanen |
| [E02](E02-process-monitoring.md) | Analisis proses CPU & memory |
| [E03](E03-service-status.md) | Verifikasi status service |
| [E04](E04-network-ports.md) | Identifikasi port jaringan listening |
| [E05](E05-server-specs.md) | Spesifikasi server (CPU, RAM, disk, OS) |
| [E06](E06-local-users.md) | Daftar user lokal |
| [E07](E07-disk-io.md) | Utilisasi I/O disk |
| [E08](E08-memory-cache.md) | Manajemen cache memory |
| [E09](E09-packet-capture.md) | Packet capture jaringan |
| [E10](E10-package-management.md) | Instalasi & verifikasi paket software |
| [E11](E11-connectivity-test.md) | Uji konektivitas SSH/NFS/DNS/NTP |
| [E12](E12-ssh-key-auth.md) | SSH key-based authentication |
| [E13](E13-group-management.md) | Manajemen group & keanggotaan |
| [E14](E14-ownership-permission.md) | Ownership & permission direktori |
| [E15](E15-log-audit.md) | Log sistem untuk audit |
| [E16](E16-password-management.md) | Manajemen password & status akun |

## Insight Utama

Beberapa temuan paling berharga dari proses ini didokumentasikan lebih detail di masing-masing file topik, di antaranya:
- Perbedaan VSZ vs RSS dalam analisis memory ([E02](easy/E02-process-monitoring.md))
- Investigasi root cause service yang gagal berulang ([E03](easy/E03-service-status.md))
- Temuan anomali keamanan lewat audit log — percobaan eskalasi privilege yang ditolak sistem ([E15](easy/E15-log-audit.md))
- Debugging SSH key authentication yang tidak berjalan sesuai ekspektasi awal ([E12](easy/E12-ssh-key-auth.md))

## Tentang

Dokumentasi ini adalah bagian dari personal learning project. Update berkala mengikuti progress pembelajaran level Medium dan Hard.

**Connect:** [LinkedIn](#) *(isi link kamu di sini)*
