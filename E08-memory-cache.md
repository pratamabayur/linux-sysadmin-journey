# E08 — Mengelola Cache Memory Sistem

**Kategori:** Monitoring

## Tujuan
Mengelola cache memory sistem dan membuktikan perubahan kondisi sebelum-sesudah.

## Command & Output

**Sebelum:**
```bash
$ free -h
           total   used   free   shared  buff/cache  available
Mem:       1.7Gi   531Mi  767Mi  6.0Mi   583Mi       1.1Gi
```

**Proses clear cache:**
```bash
$ sync
$ echo 3 | sudo tee /proc/sys/vm/drop_caches
3
```

**Sesudah:**
```bash
$ free -h
           total   used   free   shared  buff/cache  available
Mem:       1.7Gi   339Mi  1.4Gi  6.0Mi   114Mi       1.3Gi
```

## Analisis

| Metrik | Sebelum | Sesudah |
|---|---|---|
| buff/cache | 583 MB | 114 MB (↓ 469 MB) |
| free | 767 MB | 1.4 GB (↑ signifikan) |
| available | 1.1 GB | 1.3 GB (naik tipis) |

Konsisten dengan observasi sebelumnya: `available` hanya naik sedikit karena kernel Linux sudah menghitung cache sebagai memory yang *dapat dilepas sewaktu-waktu* — bukan memory yang benar-benar "terkunci". Clear cache manual tidak menambah kapasitas memory yang bisa dipakai secara signifikan; fungsinya lebih relevan untuk keperluan *benchmark* disk read murni tanpa pengaruh cache.

## Poin Penting

- Bukti perubahan wajib berupa perbandingan `buff/cache` sebelum-sesudah.
- `sync` sebelum `drop_caches` adalah best practice untuk mencegah kehilangan data yang belum ditulis ke disk.
