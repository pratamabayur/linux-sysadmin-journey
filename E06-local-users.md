# E06 — Menampilkan & Mendokumentasikan Daftar User Lokal

**Kategori:** User & SSH

## Tujuan
Menampilkan dan mendokumentasikan seluruh user lokal pada sistem.

## Command & Output

```bash
$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
...(system account lainnya, UID < 1000)...
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/sbin/nologin
chrony:x:985:985:chrony system user:/var/lib/chrony:/sbin/nologin
cmk-agent:x:981:980:Checkmk agent system user:/var/lib/cmk-agent:/sbin/nologin
<user1>:x:1000:1000:<user1>:/home/<user1>:/bin/bash
<user2>:x:1001:1001::/home/<user2>:/bin/bash
<user3>:x:1002:1002::/home/<user3>:/bin/bash
<myuser>:x:1003:1003::/home/<myuser>:/bin/bash
```

```bash
$ getent passwd
# (hasil identik dengan cat /etc/passwd — mengonfirmasi tidak ada
#  integrasi ke directory service eksternal seperti LDAP)
```

```bash
$ w
 up 10 days, 5:03, 3 users, load average: 0.00, 0.00, 0.00
USER     TTY    LOGIN@  IDLE   WHAT
<user1>  tty1   Thu20   3days  -bash
<myuser> pts/0  18:44   36:19  -bash
<myuser> pts/1  19:00   0.00s  w
```

## Analisis

- Filter UID ≥ 1000 (lihat command `awk` di versi sebelumnya) mengidentifikasi **4 user manusia** pada sistem ini, terpisah dari puluhan system account (UID < 1000) yang digunakan oleh service seperti `sshd`, `chrony`, `cmk-agent`, dll.
- `getent passwd` menghasilkan output **identik** dengan `cat /etc/passwd` — mengonfirmasi sistem ini tidak terintegrasi ke directory service eksternal (LDAP/NIS/AD); seluruh akun adalah local account murni.
- `w` mengonfirmasi server bersifat **shared/multi-user**: satu user login langsung via console (`tty1`) dengan idle time 3 hari (sesi lama yang masih terbuka), sementara user lain aktif via SSH (`pts/0`, `pts/1`) — dua sesi terpisah dari akun yang sama, konsisten dengan penggunaan `tmux` yang memungkinkan banyak sesi paralel.

## Poin Penting

- "Menampilkan" → cukup `cat`/`getent passwd`.
- "Mendokumentasikan" → hasil idealnya di-redirect ke file, bukan hanya ditampilkan di layar.
- Server shared seperti ini menuntut kehati-hatian ekstra saat melakukan perubahan sistem-wide (misalnya menonaktifkan firewall atau mengubah konfigurasi service), karena berdampak ke seluruh user yang aktif, bukan hanya diri sendiri.
