# E05 — Mengumpulkan Informasi Spesifikasi Server

**Kategori:** Monitoring

## Tujuan
Mengumpulkan informasi lengkap spesifikasi server: CPU, RAM, disk, dan versi OS.

## Command & Output

```bash
$ lscpu
Architecture:        x86_64
CPU(s):               4
Vendor ID:            AuthenticAMD
Model name:           QEMU Virtual CPU version 2.5+
Hypervisor vendor:    KVM
Virtualization type:  full
Caches (sum of all):
  L1d: 256 KiB (4 instances)   L2: 2 MiB (4 instances)
  L1i: 256 KiB (4 instances)   L3: 64 MiB (4 instances)
Vulnerabilities:      Semua "Not affected" atau "Mitigation" — tidak ada yang berstatus rentan
```

```bash
$ free -h
              total    used    free   shared  buff/cache  available
Mem:          1.7Gi    532Mi   765Mi  6.0Mi    583Mi       1.1Gi
Swap:         3.9Gi     94Mi   3.8Gi
```

```bash
$ df -h
Filesystem                Size  Used Avail Use% Mounted on
/dev/mapper/rl-root         35G   13G   23G  36% /
/dev/sda1                  960M  471M  490M  50% /boot
<nfs-server-ip>:/data/...   300G   68G  233G  23% /mnt/nfs
```

```bash
$ lsblk
NAME        SIZE TYPE MOUNTPOINTS
sda          40G disk
├─sda1        1G part /boot
└─sda2       39G part
  ├─rl-root 35.1G lvm  /
  └─rl-swap  3.9G lvm  [SWAP]
sr0        1024M rom
```

```bash
$ cat /etc/os-release
NAME="Rocky Linux"
VERSION="9.8 (Blue Onyx)"
SUPPORT_END="2032-05-31"
```

## Catatan Proses (Human Error yang Wajar)

Selama praktik, sempat terjadi dua kesalahan pengetikan yang berguna sebagai catatan pembelajaran:
- `lsbkl` (typo dari `lsblk`) → `command not found`, langsung dikoreksi.
- `cat /etc/hostnamectl` → keliru mengira `hostnamectl` adalah file yang bisa dibaca dengan `cat`, padahal itu adalah **command**, bukan file di `/etc/`. Command yang benar untuk melihat status hostname adalah `hostnamectl` (tanpa `cat`), seperti dibahas di [E01](E01-hostname.md).

## Analisis

- **CPU:** 4 core AMD virtual (QEMU), dengan seluruh mitigasi kerentanan CPU (Spectre, Meltdown, dll) berstatus "Not affected" atau "Mitigation" — baseline keamanan hardware yang baik untuk VM ini.
- **Memory:** Total 1.7 GB, dengan `available` 1.1 GB — cukup lega untuk workload ringan, namun perlu diperhatikan jika akan menjalankan service tambahan yang lebih berat.
- **Disk:** Struktur berbasis **LVM** (`rl-root`, `rl-swap`), memudahkan resize di kemudian hari tanpa perlu partisi ulang dari awal.
- **NFS mount aktif** terdeteksi dari server terpisah (`<nfs-server-ip>:/data/.../NFS`) — dikonfirmasi lebih detail di [E11](E11-connectivity-test.md).
- **OS:** Rocky Linux 9.8, dengan `SUPPORT_END` di 2032 — informasi berguna untuk perencanaan upgrade jangka panjang.

## Poin Penting

- Jawaban lengkap topik ini memerlukan kombinasi minimal 4 tool: `lscpu` + `free -h` + `df -h`/`lsblk` + `cat /etc/os-release` — tidak ada satu command tunggal yang mencakup semuanya.
- `hostnamectl` (tanpa `cat`) adalah command, bukan file — kesalahan umum bagi pemula yang baru belajar perbedaan file konfigurasi vs command sistem.
