# E07 — Mengukur Utilisasi I/O Disk + Instalasi Tool Pendukung

**Kategori:** Monitoring

## Tujuan
Mengukur utilisasi I/O disk beserta instalasi tool pendukungnya.

## Command & Output

```bash
$ iostat -x 2 5
avg-cpu:  %user %nice %system %iowait %steal %idle
           0.12   0.01   0.11    0.00   0.07  99.69

Device  r/s   rkB/s  w/s   wkB/s   %util
dm-0    0.14  4.22   0.47  7.90    0.08
dm-1    0.55  2.19   0.60  2.39    0.01
sda     0.45  6.44   0.34  10.33   0.04

# 4 snapshot berikutnya (interval 2 detik):
# seluruh device menunjukkan 0.00 di semua kolom — sistem sepenuhnya idle
```

```bash
$ vmstat 2 5
procs ---memory---  ---swap-- ---io--- ---cpu---
 r b  swpd  free    buff cache   si so  bi bo  us sy id wa st
 2 0  97252 785228  0    597236  1  1   2  3   0  0 100 0  0
 0 0  97252 785228  0    597276  0  0   0  0   0  0  99  0  0
 0 0  97252 785228  0    597276  0  0   0  0   0  0  100 0  0
```

## Analisis

- Snapshot pertama `iostat` menangkap sedikit aktivitas sisa dari command sebelumnya, namun 4 snapshot berikutnya menunjukkan **`%util` 0.00%** secara konsisten — disk dalam kondisi benar-benar idle.
- `vmstat` mengonfirmasi hal yang sama: kolom `bi`/`bo` (block in/out) bernilai 0 di hampir semua baris, dan `id` (CPU idle) berada di 99-100%.
- Kondisi idle ini dicatat sebagai **baseline** — referensi kondisi normal server, berguna sebagai pembanding jika suatu saat terjadi masalah performa.

## Poin Penting

- Instalasi tool pendukung (`sudo dnf install sysstat -y`) wajib disertakan sebagai bagian jawaban, bukan hanya command monitoring.
- Metrik kunci: `%util` (persentase disk sibuk) dan `await` (waktu tunggu I/O) — keduanya indikator utama bottleneck disk.
