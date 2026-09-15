# E11 — Menguji Konektivitas ke SSH/NFS/DNS/NTP

**Kategori:** Networking

## Tujuan
Menguji konektivitas jaringan ke berbagai service: SSH, NFS, DNS, dan NTP.

## Studi Kasus 1: Membedakan Jenis Kegagalan Koneksi

```bash
$ nc -zv 254.255.255.255 22
Ncat: TIMEOUT.

$ nc -zv <public-ip> <forwarded-port>
Ncat: Connected to <public-ip>:<forwarded-port>.
```

| Hasil | Artinya |
|---|---|
| **TIMEOUT** | Tidak ada balasan sama sekali — host tidak reachable atau firewall silent-drop |
| **Connected** | Host reachable dan port terbuka |

TIMEOUT berbeda dari "Connection refused": TIMEOUT berarti tidak ada respon apa pun (indikasi masalah routing/firewall silent-drop), sedangkan "refused" berarti host merespon aktif namun menolak koneksi di port tersebut.

## Studi Kasus 2: Kesalahan Sintaks Command (Pembelajaran dari Typo)

```bash
$ ssh -0 ConnectTimeout=5 <user>@<host> echo "OK"
unknown option -- 0
```
Typo `-0` (angka nol) seharusnya `-o` (huruf O) — kesalahan visual umum karena kemiripan bentuk karakter pada banyak font.

## Studi Kasus 3: Membuktikan Arsitektur Client-Server NFS via Dummy Interface

Sebagai eksplorasi tambahan, dibuat *dummy network interface* untuk memahami lebih dalam perbedaan antara service yang berjalan secara lokal:

```bash
$ sudo modprobe dummy
$ sudo ip link add dummy0 type dummy
$ sudo ip addr add 192.168.50.10/24 dev dummy0
$ ip addr show dummy0
13: dummy0: <BROADCAST,NOARP> mtu 1500 ... state DOWN
    inet 192.168.50.10/24 scope global dummy0
```

**Pengujian port lokal melalui IP dummy:**
```bash
$ nc -zv 192.168.50.10 111
Connected.                    # rpcbind MERESPON

$ nc -zv 192.168.50.10 2049
Connection refused.           # NFS server TIDAK berjalan lokal
```

### Analisis Temuan Kunci

Server ini berperan sebagai **NFS client** (mount dari server eksternal, lihat [E05](E05-server-specs.md)), **bukan NFS server**. Eksperimen di atas membuktikan hal ini secara langsung:
- **Port 111 (rpcbind) aktif secara lokal** — dibutuhkan meski hanya berperan sebagai NFS client.
- **Port 2049 (NFS daemon) tidak aktif secara lokal** — karena tidak ada NFS server daemon (`nfsd`) yang dijalankan di server ini.

Ini dikonfirmasi ulang dengan:
```bash
$ showmount -e 127.0.0.1
# (output kosong — tidak ada yang di-export dari server ini sendiri)
```

**Kesalahan sintaks CIDR** juga ditemukan dalam eksperimen ini — mencoba menggunakan notasi `/24` pada tools yang mengharapkan IP polos:
```bash
$ rpcinfo -p 192.168.50.10/24
192.168.50.10/24: RPC: Unknown host

$ nslookup 192.168.50.10/24
** server can't find 192.168.50.10/24: NXDOMAIN

$ dig 192.168.50.10/24
;; ->>HEADER<<- ... status: NXDOMAIN
```
**Pembelajaran:** notasi CIDR (`/24`) hanya valid digunakan pada command konfigurasi alamat (`ip addr`), bukan pada tools yang mengharapkan IP address polos atau hostname (`rpcinfo`, `nslookup`, `dig`, `showmount`). Sistem memperlakukan input tersebut sebagai nama domain yang harus di-resolve, sehingga gagal dengan NXDOMAIN.

## Pengujian Service Sesungguhnya

**NFS (ke server eksternal):**
```bash
$ showmount -e <nfs-server-ip>
Export list for <nfs-server-ip>:
/data/share <allowed-subnet>

$ rpcinfo -p <nfs-server-ip>
   100000 4 tcp 111 portmapper
   100003 3 tcp 2049 nfs
```

**DNS:**
```bash
$ nslookup google.com
$ dig @8.8.8.8 google.com
```

**NTP:**
```bash
$ chronyc sources
MS Name/IP address       Stratum Poll Reach LastRx
^* time.citra.net.id         2    9   377    33  +1967us +/- 24ms
^+ 0.ntp.lambda.net.id       2    8   377    39  -1754us +/- 49ms

$ chronyc tracking
Reference ID    : CA4172CA (time.citra.net.id)
Stratum         : 3
Leap status     : Normal
```

## Tabel Referensi Port

| Service | Port | Protokol |
|---|---|---|
| SSH | 22 | TCP |
| DNS | 53 | TCP/UDP |
| NTP | 123 | UDP |
| RPC (NFS) | 111 | TCP/UDP |
| NFS | 2049 | TCP/UDP |

## Tabel Jenis Error Konektivitas

| Error | Artinya |
|---|---|
| `TIMEOUT` | Tidak ada respon sama sekali |
| `Could not resolve hostname` / `NXDOMAIN` | Nama host tidak valid/tidak ditemukan |
| `Connection refused` | Host reachable, port tertutup |
| `Unknown host` (rpcinfo) | Format alamat tidak valid (misal menyertakan CIDR yang tidak seharusnya) |
| `Permission denied` | Koneksi berhasil, kredensial salah |

## Poin Penting

- Membedakan jenis kegagalan koneksi (TIMEOUT vs refused vs unknown host) adalah keterampilan diagnostik penting — masing-masing mengarah ke akar masalah berbeda.
- Notasi CIDR (`/24`) hanya digunakan pada konfigurasi alamat IP (`ip addr add`), bukan pada tools yang mengharapkan IP polos.
- Eksperimen dengan dummy interface adalah cara efektif untuk membuktikan perilaku service secara lokal tanpa bergantung pada jaringan eksternal — dalam kasus ini berhasil membuktikan server berperan sebagai NFS client, bukan NFS server.
