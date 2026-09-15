# E09 — Melakukan Capture Trafik Jaringan

**Kategori:** Networking

## Tujuan
Melakukan capture trafik jaringan menggunakan tool packet capture.

## Command & Output

**Daftar interface:**
```bash
$ tcpdump -D
1.eth-interface-A [Up, Running, Connected]
2.eth-interface-B [Up, Running, Connected]
3.any (Pseudo-device that captures on all interfaces) [Up, Running]
4.lo [Up, Running, Loopback]
5.docker0 [Up, Disconnected]
...
```

```bash
$ ip a
2: eth-interface-A: ... inet <internal-ip-A>/24 ...
3: eth-interface-B: ... inet <internal-ip-B>/24 ...
4: docker0: ... inet 172.17.0.1/16 ...
```

**Capture di interface tanpa trafik:**
```bash
$ sudo tcpdump -i lo -c 20
0 packets captured
```

**Capture di interface dengan trafik NFS & broadcast jaringan lokal:**
```bash
$ sudo tcpdump -i eth-interface-A -c 20
<time> IP <gateway-ip>.rrac > 255.255.255.255.rrac: UDP, length 137 (broadcast switch/router)
<time> IP <hostname>.fcp-udp > <nfs-server-ip>.nfs: NFS request ... getattr
<time> IP <nfs-server-ip>.nfs > <hostname>.fcp-udp: NFS reply ok
<time> ARP, Request who-has <nfs-server-ip> tell <hostname>
<time> ARP, Reply <nfs-server-ip> is-at <mac-disamarkan>
```

**Capture khusus port 22 (tidak ada trafik saat itu):**
```bash
$ sudo tcpdump -i eth-interface-A port 22 -c 10
0 packets captured
```

**Capture di interface SSH aktif (dengan -n):**
```bash
$ sudo tcpdump -i eth-interface-B -n -c 20
<time> IP <client-ip>.xxxxx > <server-ip>.ssh: Flags [.], ack ...
<time> IP <server-ip>.ssh > <client-ip>.xxxxx: Flags [P.], seq ..., length 220
```

## Analisis

- **Interface `lo` dan `eth-interface-A` untuk port 22** tidak menangkap paket — logis, karena SSH session berjalan lewat interface **kedua** (`eth-interface-B`), bukan interface pertama.
- Trafik broadcast (`255.255.255.255`) dari perangkat jaringan (device ID terekam di paket CDP) menunjukkan adanya **switch/router Cisco atau kompatibel** yang mengirim pesan discovery protocol secara berkala — ini traffic normal infrastruktur jaringan, bukan trafik aplikasi.
- Trafik **NFS request/reply** (`getattr`) terekam berjalan lewat **UDP**, mengonfirmasi mount NFS yang ditemukan di [E05](E05-server-specs.md) benar-benar aktif berkomunikasi secara berkala (bukan hanya konfigurasi statis).
- Percakapan ARP (`who-has` / `is-at`) menunjukkan proses resolusi MAC address standar antara server dan server NFS — bagian normal dari komunikasi jaringan lokal.
- Capture di `eth-interface-B` (dengan `-n`) menangkap trafik SSH aktual — pola `Flags [P.]` bolak-balik antara client dan server, representasi normal dari sesi interaktif SSH terenkripsi.

## Poin Penting

- Server dengan **lebih dari satu network interface** (dual-homed) perlu dipilih interface yang tepat saat capture — trafik yang dicari mungkin tidak lewat interface pertama yang dicoba.
- Trafik broadcast dari infrastruktur jaringan (CDP, ARP) adalah "noise" normal yang perlu dibedakan dari trafik aplikasi saat menganalisis hasil capture.
- Opsi `-n` tetap direkomendasikan untuk analisis yang presisi dan cepat.
