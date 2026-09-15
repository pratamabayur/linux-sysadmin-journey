# E03 — Verifikasi Status/Keberjalanan Service

**Kategori:** Monitoring

## Tujuan
Memverifikasi status berjalannya service sistem menggunakan systemd.

## Command & Output

**Status service SSH:**
```bash
$ systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-09-04 07:19:28 WIB; 1 week 3 days ago
   Main PID: 805 (sshd)
      Tasks: 1 (limit: 10636)
     Memory: 7.6M (peak: 18.1M)
        CPU: 5.003s

Sep 14 18:44:02 <hostname> sshd-session[xxxxxxx]: Accepted publickey for bayu from <internal-ip> port xxxxx
Sep 14 18:44:02 <hostname> sshd-session[xxxxxxx]: pam_unix(sshd:session): session opened for user bayu(uid...
```

**Status service Docker (dengan riwayat error):**
```bash
$ systemctl status docker
● docker.service - Docker Application Container Engine
     Active: active (running) since Fri 2026-09-04 07:19:31 WIB; 1 week 3 days ago
   Main PID: 1062 (dockerd)
      Tasks: 25
     Memory: 36.2M (peak: 198.0M)

Sep 11 20:19:35 <hostname-lama> dockerd[1062]: level=error msg=...
Sep 12 03:56:46 <hostname-lama> dockerd[1062]: level=info msg=...
```

**Cek cepat (yes/no):**
```bash
$ systemctl is-active sshd
active
$ systemctl is-enabled sshd
enabled
```

**Daftar semua service yang berjalan:**
```bash
$ systemctl list-units --type=service --state=running
  atd.service                  loaded active running Deferred execution scheduler
  auditd.service               loaded active running Security Auditing Service
  check-mk-agent-async.service loaded active running Checkmk agent - Asynchronous background tasks
  chronyd.service              loaded active running NTP client/server
  cloudflared.service          loaded active running cloudflared
  cmk-agent-ctl-daemon.service loaded active running Checkmk agent controller daemon
  containerd.service           loaded active running containerd container runtime
  docker.service                loaded active running Docker Application Container Engine
  sshd.service                  loaded active running OpenSSH server daemon
  ... (26 unit total)
```

**Cek service yang gagal:**
```bash
$ systemctl --failed
  UNIT LOAD ACTIVE SUB DESCRIPTION
0 loaded units listed.
```

## Analisis

**Temuan penting — perbandingan dengan observasi sebelumnya:**
Pada pemeriksaan awal (lihat catatan di [E15](E15-log-audit.md)), `dnf-makecache.service` tercatat berstatus **failed**. Namun pada pemeriksaan kali ini, `systemctl --failed` menunjukkan **0 unit gagal** — tidak ada lagi entri yang bermasalah.

Ini konsisten dengan sifat `dnf-makecache` sebagai *timer-based service* (dijadwalkan berkala, bukan service yang terus berjalan) — kegagalan pada satu eksekusi tidak selalu tercermin di eksekusi berikutnya. Ini pengingat penting: status `--failed` merefleksikan kondisi **saat ini**, bukan riwayat lengkap. Untuk menilai apakah sebuah service memiliki masalah berulang, verifikasi historis lewat `journalctl -u <service>` (lihat [E15](E15-log-audit.md)) tetap diperlukan sebagai pelengkap.

**Temuan lain:** Log `docker.service` menunjukkan hostname sistem sempat tercatat sebagai `<hostname-lama>` pada 11-12 September, sementara log `sshd.service` yang lebih baru (14 September) sudah menampilkan `<hostname-baru>` — ini adalah bukti langsung dari perubahan hostname yang dilakukan di [E01](E01-hostname.md), terekam otomatis oleh systemd di log berbagai service.

Terlihat juga beberapa service monitoring aktif berjalan (`check-mk-agent-async`, `cmk-agent-ctl-daemon`) — mengonfirmasi server ini terhubung ke sistem monitoring terpusat, relevan dengan topik arsitektur monitoring perusahaan yang dibahas terpisah.

## Poin Penting

- Nama service SSH di Rocky/RHEL adalah `sshd` (bukan `ssh`).
- `systemctl --failed` hanya menunjukkan kondisi **saat perintah dijalankan** — untuk service berbasis timer/jadwal, kegagalan sebelumnya bisa saja tidak lagi terlihat di pemeriksaan berikutnya. Selalu silangkan dengan `journalctl` untuk gambaran historis yang lebih akurat.
- Log dari berbagai service bisa menjadi bukti tidak langsung dari perubahan konfigurasi sistem lain (seperti perubahan hostname) — cross-referencing log antar service adalah teknik audit yang berguna.
