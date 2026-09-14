# E01 — Mengubah Hostname Secara Permanen

**Kategori:** Basic System

## Tujuan
Mengubah nama host sistem secara permanen (bertahan setelah reboot), bukan hanya sementara di memory.

## Command & Output

**Cek hostname sebelum diubah:**
```bash
$ hostnamectl
 Static hostname: <hostname-lama>
       Icon name: computer-vm
         Chassis: vm
      Machine ID: <disamarkan>
         Boot ID: <disamarkan>
  Virtualization: kvm
Operating System: Rocky Linux 9.8 (Blue Onyx)
     CPE OS Name: cpe:/o:rocky:rocky:9::baseos
          Kernel: Linux 5.14.0-687.39.1.el9_8.x86_64
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC (i440FX + PIIX, 1996)
Firmware Version: rel-1.16.3-0-ga6ed6b701f0a-prebuilt.qemu.org
```

**Mengubah hostname secara permanen:**
```bash
$ sudo hostnamectl set-hostname <hostname-baru>
```

**Verifikasi setelah perubahan:**
```bash
$ hostnamectl status
 Static hostname: <hostname-baru>
       Icon name: computer-vm
         Chassis: vm
      Machine ID: <disamarkan>
         Boot ID: <disamarkan>
  Virtualization: kvm
Operating System: Rocky Linux 9.8 (Blue Onyx)
     CPE OS Name: cpe:/o:rocky:rocky:9::baseos
          Kernel: Linux 5.14.0-687.39.1.el9_8.x86_64
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC (i440FX + PIIX, 1996)
Firmware Version: rel-1.16.3-0-ga6ed6b701f0a-prebuilt.qemu.org

$ hostname
<hostname-baru>

$ cat /etc/hostname
<hostname-baru>
```

## Analisis

Tiga command verifikasi (`hostnamectl status`, `hostname`, `cat /etc/hostname`) menunjukkan hasil yang **konsisten** — ini membuktikan perubahan benar-benar tersimpan secara permanen di file `/etc/hostname`, bukan hanya nilai sementara di memory kernel.

`hostnamectl status` juga sekaligus menampilkan detail hardware virtual (vendor QEMU, model "Standard PC i440FX + PIIX"), yang mengonfirmasi bahwa sistem ini berjalan sebagai **virtual machine** di atas hypervisor KVM/QEMU — informasi yang berguna juga untuk inventarisasi sistem di [E05](E05-server-specs.md).

## Poin Penting

- Selalu gunakan `hostnamectl set-hostname`, **bukan** command `hostname <nama>` biasa, ketika task meminta perubahan yang **permanen** — `hostnamectl` menulis ke `/etc/hostname`, sedangkan `hostname` saja hanya mengubah nilai runtime di memory (hilang setelah reboot).
- Verifikasi idealnya dilakukan lewat **lebih dari satu cara** (`hostnamectl status`, `hostname`, `cat /etc/hostname`) untuk memastikan konsistensi hasil, bukan hanya satu command saja.
- `hostnamectl` sekaligus menampilkan info OS, kernel, arsitektur, dan hardware — satu command yang serba guna untuk dokumentasi sistem.
