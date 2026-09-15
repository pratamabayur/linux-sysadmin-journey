# E02 — Analisis Proses CPU & Memory

**Kategori:** Monitoring

## Tujuan
Mengidentifikasi proses yang paling banyak menggunakan resource CPU dan memory pada sistem.

## Command & Output

**Top proses berdasarkan CPU:**
```bash
$ ps aux --sort=-%cpu | head -n 11
USER    PID     %CPU %MEM    VSZ    RSS   TTY STAT START  TIME COMMAND
bayu    xxxxxxx  0.5  0.4   18336   7944  ?   S    18:44  0:00 sshd-session: bayu@pts/0
root    xxxx     0.3  2.3  1295300 41156  ?   Ssl  Sep04  45:35 /usr/bin/cloudflared --no-autoupdate tunnel run --token <disensor>
root    xxx      0.1  1.3  1969396 22984  ?   Ssl  Sep04  16:13 /usr/bin/containerd
root    xxxxxxx  0.1  0.0       0     0   ?   I    18:42  0:00 [kworker/2:0-events]
root    1        0.0  0.6  173528 10748  ?   Ss   Sep04  1:49  /usr/lib/systemd/systemd --switched-root ...
root    2-6      0.0  0.0       0     0   ?   S/I< Sep04  0:00  [kthreadd] / [kworker/R-*] (proses kernel)
```

**Top proses berdasarkan Memory:**
```bash
$ ps aux --sort=-%mem | head -n 11
USER    PID     %CPU %MEM    VSZ    RSS   TTY STAT START  TIME COMMAND
root    xxxx     0.3  2.3  1295300 41156  ?   Ssl  Sep04  45:36 /usr/bin/cloudflared --no-autoupdate tunnel run --token <disensor>
root    1062     0.0  2.0  3132724 36108  ?   Ssl  Sep04  5:26  /usr/bin/dockerd -H fd:// --containerd=...
root    819      0.1  1.3  1969396 22984  ?   Ssl  Sep04  16:13 /usr/bin/containerd
bayu    xxxxxx   0.0  0.7   22692  12276  ?   Ss   Sep12  0:00  /usr/lib/systemd/systemd --user
root    xxxxxxx  0.0  0.6   18076  11956  ?   Ss   18:44  0:00  sshd-session: bayu [priv]
root    1        0.0  0.6  173528 10748  ?   Ss   Sep04  1:49  /usr/lib/systemd/systemd --switched-root ...
root    751      0.0  0.5  260980  8964   ?   Ssl  Sep04  0:26  /usr/sbin/NetworkManager --no-daemon
root    807      0.0  0.4  258604  7812   ?   Ssl  Sep04  2:45  /usr/bin/python3 -Es /usr/sbin/tuned -l -P
bayu    xxxxxxx  0.5  0.4   18336   7944   ?   S    18:44  0:00  sshd-session: bayu@pts/0
```

## Analisis

- **`cloudflared`** konsisten muncul di puncak kedua daftar (CPU maupun memory) — wajar karena proses ini berjalan terus-menerus (long-running tunnel service) sejak `Sep04`, meski beban CPU aktualnya kecil (0.3%).
- Terlihat relasi proses `dockerd` → `containerd`: Docker daemon memanggil containerd sebagai runtime aktual untuk menjalankan container — keduanya konsisten muncul berdekatan di daftar memory.
- Proses `sshd-session: bayu@pts/0` dan `sshd-session: bayu [priv]` menunjukkan sesi SSH aktif milik user sendiri saat command dijalankan — proses ini muncul karena sedang login, bukan indikasi anomali.
- Proses kernel (`kworker`, `kthreadd`) menggunakan 0% CPU/memory — normal untuk sistem yang idle.

## Poin Penting

- **VSZ** (virtual size, memory yang dialokasikan) berbeda dengan **RSS** (memory fisik yang benar-benar dipakai) — RSS lebih relevan untuk menjawab "siapa yang paling boros memory sungguhan".
- Mode non-interaktif (`ps aux --sort`) cocok untuk dokumentasi/laporan; mode interaktif (`top`/`htop`) untuk observasi real-time.
- ⚠️ **Catatan keamanan pribadi:** command asli `cloudflared` di server latihan sempat menampilkan token autentikasi tunnel di output `ps aux` (karena token dilewatkan sebagai argumen command line, sehingga terlihat oleh siapa pun yang bisa menjalankan `ps`). Ini adalah contoh nyata risiko keamanan — kredensial sebaiknya tidak dilewatkan sebagai argumen command line yang terlihat di process list, melainkan lewat environment variable atau file konfigurasi dengan permission terbatas.
