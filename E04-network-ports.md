# E04 — Identifikasi Port Jaringan yang Listening

**Kategori:** Monitoring/Networking

## Tujuan
Mengidentifikasi port jaringan yang sedang listening beserta proses pemiliknya.

## Command & Output

**Semua port listening (TCP & UDP):**
```bash
$ sudo ss -tulnp
Netid State  Local Address:Port   Peer Address:Port   Process
udp   UNCONN 0.0.0.0:111          0.0.0.0:*           rpcbind, systemd
udp   UNCONN 127.0.0.1:323        0.0.0.0:*           chronyd
udp   UNCONN *:55211              *:*                 cloudflared
udp   UNCONN [::]:111             [::]:*              rpcbind, systemd
udp   UNCONN [::1]:323            [::]:*              chronyd
udp   UNCONN *:52667              *:*                 cloudflared
udp   UNCONN *:53736              *:*                 cloudflared
udp   UNCONN *:48617              *:*                 cloudflared
tcp   LISTEN 127.0.0.1:20241      0.0.0.0:*           cloudflared
tcp   LISTEN 0.0.0.0:111          0.0.0.0:*           rpcbind, systemd
tcp   LISTEN 0.0.0.0:22           0.0.0.0:*           sshd
tcp   LISTEN *:6556               *:*                 cmk-agent-ctl
tcp   LISTEN [::]:111             [::]:*              rpcbind, systemd
tcp   LISTEN [::]:22              [::]:*              sshd
```

**Filter port tertentu:**
```bash
$ sudo ss -tulnp | grep :22
tcp   LISTEN 0.0.0.0:22   0.0.0.0:*   sshd
tcp   LISTEN [::]:22      [::]:*      sshd
```

**Alternatif (lsof):**
```bash
$ sudo lsof -i -P -n | grep LISTEN
systemd    1     root      TCP *:111 (LISTEN)
rpcbind    718   rpc       TCP *:111 (LISTEN)
cmk-agent  799   cmk-agent TCP *:6556 (LISTEN)
sshd       805   root      TCP *:22 (LISTEN)
cloudflar  1057  root      TCP 127.0.0.1:20241 (LISTEN)
```

## Analisis

| Port | Service | Binding | Keterangan |
|---|---|---|---|
| 22 | sshd | `0.0.0.0` | SSH — dapat diakses dari semua interface |
| 111 | rpcbind | `0.0.0.0` | Portmapper, terkait NFS (lihat E11) |
| 6556 | cmk-agent-ctl | `*` (semua) | Agent monitoring terpusat (Checkmk) |
| 20241 | cloudflared | `127.0.0.1` saja | Hanya dapat diakses secara lokal, tidak dari luar server |
| 323 (UDP) | chronyd | `127.0.0.1` | NTP client, komunikasi lokal untuk query waktu |
| Port ephemeral (UDP) | cloudflared | `*` | Port dinamis untuk komunikasi tunnel keluar |

**Validasi silang** dua tool berbeda (`ss` dan `lsof`) menghasilkan temuan yang **konsisten** — sama-sama mengonfirmasi proses dan port yang sama, meningkatkan keyakinan terhadap akurasi hasil.

## Poin Penting

- **`0.0.0.0`** = listening di semua interface (dapat diakses dari luar server).
- **`127.0.0.1`** = listening hanya di localhost (tidak dapat diakses dari luar) — penting dipahami untuk analisis keamanan (relevan di topik hardening SSH, level Medium).
- Menggunakan lebih dari satu tool (`ss` + `lsof`) untuk validasi silang adalah praktik baik saat melakukan audit konfigurasi jaringan.
