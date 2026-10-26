# Simulasi Jaringan dan Keamanan Cisco ASA

## Deskripsi Project

Project ini merupakan simulasi jaringan dan keamanan menggunakan Cisco Packet Tracer dengan Cisco ASA 5505 sebagai firewall.

Topologi menghubungkan jaringan IT, Finance, dan Server melalui Cisco ASA serta router ISP. Firewall menerapkan Access Control List (ACL) untuk mengontrol trafik tertentu berdasarkan alamat IP dan protokol.

Project ini dibuat untuk mempraktikkan konfigurasi firewall, segmentasi jaringan, static routing, pengendalian trafik ICMP, serta verifikasi keamanan jaringan.

## Tujuan Project

Tujuan dari project ini adalah:

- Mengonfigurasi Cisco ASA 5505 sebagai firewall.
- Memahami konsep interface inside dan outside.
- Menerapkan segmentasi jaringan menggunakan VLAN.
- Mengonfigurasi static routing antara firewall dan router ISP.
- Menerapkan ACL untuk mengizinkan dan menolak trafik tertentu.
- Menguji konektivitas antarperangkat menggunakan `ping`.
- Memverifikasi aturan ACL menggunakan hit counter.
- Melakukan troubleshooting konektivitas jaringan.

## Teknologi yang Digunakan

| Teknologi | Fungsi |
|---|---|
| Cisco Packet Tracer | Simulasi dan pengujian jaringan |
| Cisco ASA 5505 | Firewall dan kontrol akses jaringan |
| Cisco 2911 Router | Routing antara jaringan outside dan server |
| Cisco Catalyst 2960 | Menghubungkan perangkat dalam jaringan |
| IPv4 | Pengalamatan jaringan |
| VLAN | Segmentasi jaringan Layer 2 |
| Static Routing | Menentukan jalur menuju jaringan lain |
| Access Control List (ACL) | Mengontrol trafik berdasarkan aturan |
| ICMP Inspection | Memproses trafik ICMP sesuai kebijakan inspeksi |

## Topologi Jaringan

Topologi terdiri dari Cisco ASA 5505, satu router ISP, dua switch Cisco Catalyst 2960, dua PC, dan satu server.

### Perangkat dan Perannya

- **PC IT** — endpoint pada jaringan IT.
- **Switch0** — menghubungkan PC IT ke sisi inside ASA.
- **PC Finance** — endpoint pada jaringan Finance.
- **Switch1** — menghubungkan PC Finance ke sisi outside ASA.
- **Cisco ASA 5505** — firewall yang mengontrol trafik antarjaringan.
- **Router0 (ISP)** — menghubungkan jaringan outside dengan jaringan server.
- **Server0** — server pada jaringan `192.168.30.0/24`.

### Alur Koneksi

- PC IT → Switch0 → ASA Inside.
- PC Finance → Switch1 → ASA Outside.
- Switch0 ↔ Switch1 melalui link antarswitch.
- ASA ↔ Router ISP.
- Router ISP ↔ Server.

Screenshot topologi dapat disimpan pada `screenshots/topology.png`.

## Skema Alamat IP

### Tabel Pengalamatan IP

| Perangkat | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| Cisco ASA | Inside / VLAN 1 | `192.168.10.1` | `255.255.255.0` |
| Cisco ASA | Outside / VLAN 2 | `192.168.20.1` | `255.255.255.0` |
| Router ISP | G0/0 | `192.168.20.2` | `255.255.255.0` |
| Router ISP | G0/1 | `192.168.30.1` | `255.255.255.0` |
| PC IT | FastEthernet0 | `192.168.10.10` | `255.255.255.0` |
| PC Finance | FastEthernet0 | `192.168.20.10` | `255.255.255.0` |
| Server0 | FastEthernet0 | `192.168.30.10` | `255.255.255.0` |

### Default Gateway

| Perangkat | Default Gateway |
|---|---|
| PC IT | `192.168.10.1` |
| PC Finance | `192.168.20.1` |
| Server0 | `192.168.30.1` |

### Pembagian Jaringan

| Network Address | Fungsi |
|---|---|
| `192.168.10.0/24` | Jaringan IT / Inside |
| `192.168.20.0/24` | Jaringan Finance / Outside |
| `192.168.30.0/24` | Jaringan Server |

## Konfigurasi Cisco ASA

Cisco ASA 5505 menggunakan interface VLAN untuk membedakan jaringan inside dan outside.

### Interface Inside

```text
interface Vlan1
 nameif inside
 security-level 100
 ip address 192.168.10.1 255.255.255.0
```

### Interface Outside

```text
interface Vlan2
 nameif outside
 security-level 0
 ip address 192.168.20.1 255.255.255.0
```

### Pemetaan Port Ethernet

Port Ethernet0/0 digunakan pada sisi inside, sedangkan Ethernet0/1 dan Ethernet0/2 dikonfigurasi untuk VLAN 2.

```text
interface Ethernet0/0
!
interface Ethernet0/1
 switchport access vlan 2
!
interface Ethernet0/2
 switchport access vlan 2
```

Pemetaan VLAN tersebut mengikuti konfigurasi yang digunakan pada simulasi.

## Konfigurasi Switch

Dua switch Cisco Catalyst 2960 digunakan untuk menghubungkan PC dengan Cisco ASA.

### Switch0 — Sisi IT

Switch0 menghubungkan PC IT dan interface inside ASA melalui VLAN 1.

Port yang digunakan:

- Fa0/1 — terhubung ke ASA Ethernet0/0.
- Fa0/3 — terhubung ke Switch1 Fa0/3.

### Switch1 — Sisi Finance

Switch1 menghubungkan PC Finance dan interface outside ASA.

Port yang digunakan:

- Fa0/1 — terhubung ke ASA Ethernet0/1 dan berada pada VLAN 2.
- Fa0/2 — terhubung ke PC Finance dan berada pada VLAN 2.
- Fa0/3 — terhubung ke Switch0 Fa0/3 dan berada pada VLAN 1.

VLAN 2 dapat diberi nama `OUTSIDE` agar dokumentasi lebih mudah dibaca.

```text
vlan 2
 name OUTSIDE
```

Contoh konfigurasi port access VLAN 2 pada Switch1:

```text
interface fa0/1
 switchport mode access
 switchport access vlan 2

interface fa0/2
 switchport mode access
 switchport access vlan 2
```

Link antarswitch pada konfigurasi ini berada pada VLAN 1 dan tidak diklaim sebagai trunk. Jika komunikasi beberapa VLAN melalui link antarswitch diperlukan, konfigurasi VLAN dan trunk harus ditinjau kembali.

## Konfigurasi ISP dan Routing

Router ISP menggunakan dua interface GigabitEthernet.

### Konfigurasi Interface ISP

```text
interface GigabitEthernet0/0
 ip address 192.168.20.2 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.30.1 255.255.255.0
 no shutdown
```

### Static Route pada Cisco ASA

ASA memiliki route menuju jaringan server melalui router ISP.

```text
route outside 192.168.30.0 255.255.255.0 192.168.20.2 1
```

### Static Route pada ISP

Router ISP memiliki default route menuju ASA dan route menuju jaringan IT.

```text
ip route 0.0.0.0 0.0.0.0 192.168.20.1
ip route 192.168.10.0 255.255.255.0 192.168.20.1
```

Jaringan `192.168.20.0/24` dan `192.168.30.0/24` merupakan jaringan yang terhubung langsung ke router ISP.

## Kebijakan Keamanan Jaringan

Kebijakan keamanan dalam simulasi ini berfokus pada pengaturan trafik ICMP menggunakan ACL pada Cisco ASA.

| Sumber | Tujuan | Kebijakan | Hasil Pengujian |
|---|---|---|---|
| PC Finance | Router ISP | Izinkan ICMP | Berhasil |
| PC Finance | Server | Izinkan ICMP | Berhasil |
| PC Finance | PC IT | Tolak ICMP | Diblokir |
| Router ISP | ASA Outside | Uji konektivitas ICMP | Berhasil |
| Router ISP | Server | Uji konektivitas ICMP | Berhasil |

Kebijakan ini menunjukkan bahwa ACL dapat digunakan untuk mengizinkan dan menolak trafik berdasarkan alamat IP sumber, alamat IP tujuan, dan protokol.

Pengujian ini secara khusus membuktikan perilaku ICMP. Kebijakan untuk HTTP, HTTPS, atau protokol aplikasi lainnya perlu diuji secara terpisah.

## Konfigurasi Access Control List (ACL)

Cisco ASA menggunakan dua ACL utama, yaitu `INSIDE-FILTER` dan `OUTSIDE-TO-INSIDE`.

### INSIDE-FILTER

ACL ini diterapkan pada interface inside untuk mengatur trafik yang berasal dari jaringan IT.

```text
access-list INSIDE-FILTER extended permit ip 192.168.10.0 255.255.255.0 192.168.30.0 255.255.255.0
access-list INSIDE-FILTER extended permit ip 192.168.10.0 255.255.255.0 192.168.20.0 255.255.255.0

access-group INSIDE-FILTER in interface inside
```

Aturan tersebut mengizinkan trafik IP dari jaringan IT menuju jaringan server dan jaringan outside yang ditentukan.

### OUTSIDE-TO-INSIDE

ACL ini mengatur trafik ICMP tertentu yang masuk melalui interface outside.

```text
access-list OUTSIDE-TO-INSIDE extended permit icmp 192.168.30.0 255.255.255.0 192.168.10.0 255.255.255.0
access-list OUTSIDE-TO-INSIDE extended permit icmp host 192.168.20.2 host 192.168.20.1
access-list OUTSIDE-TO-INSIDE extended permit icmp host 192.168.30.10 host 192.168.10.10
access-list OUTSIDE-TO-INSIDE extended permit icmp host 192.168.20.10 host 192.168.20.2
access-list OUTSIDE-TO-INSIDE extended permit icmp host 192.168.20.10 host 192.168.30.10
access-list OUTSIDE-TO-INSIDE extended deny icmp host 192.168.20.10 host 192.168.10.10

access-group OUTSIDE-TO-INSIDE in interface outside
```

### Penjelasan Aturan

- Mengizinkan ICMP dari jaringan server menuju jaringan IT.
- Mengizinkan ICMP dari router ISP menuju interface outside ASA.
- Mengizinkan ICMP dari server menuju PC IT.
- Mengizinkan PC Finance melakukan ping ke router ISP.
- Mengizinkan PC Finance melakukan ping ke server.
- Menolak ICMP dari PC Finance menuju PC IT.

ACL diproses dari atas ke bawah. Urutan aturan perlu diperhatikan karena aturan yang lebih dahulu cocok akan diproses terlebih dahulu.

## ICMP Inspection

Cisco ASA menggunakan ICMP inspection pada kebijakan global.

```text
class-map inspection_default
 match default-inspection-traffic

policy-map global_policy
 class inspection_default
  inspect icmp

service-policy global_policy global
```

Konfigurasi ini mengaktifkan inspeksi ICMP untuk trafik yang sesuai dengan kebijakan inspeksi.

ICMP inspection dan ACL memiliki fungsi berbeda: ACL menentukan trafik yang diizinkan atau ditolak, sedangkan inspection membantu ASA menangani trafik ICMP yang sesuai dengan kebijakan global.

## Pengujian Konektivitas

Pengujian dilakukan menggunakan perintah `ping` pada PC Finance dan router ISP.

### Finance ke ISP

Perintah:

```text
ping 192.168.20.2
```

Hasil:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

**Status: Berhasil.**

PC Finance dapat melakukan ping ke router ISP setelah aturan ACL yang sesuai ditambahkan.

### Finance ke Server

Perintah:

```text
ping 192.168.30.10
```

Hasil:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

**Status: Berhasil.**

PC Finance dapat menjangkau server melalui ASA dan router ISP.

### Finance ke PC IT

Perintah:

```text
ping 192.168.10.10
```

Hasil:

```text
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

**Status: Diblokir sesuai target pengujian ICMP.**

Setelah pengujian, aturan deny ICMP untuk sumber `192.168.20.10` dan tujuan `192.168.10.10` menunjukkan `hitcnt=4`.

### ISP ke Server

Perintah pada router ISP:

```text
ping 192.168.30.10
```

Hasil:

```text
Success rate is 100 percent (5/5)
```

**Status: Berhasil.**

Router ISP dapat menjangkau server yang terhubung langsung ke jaringannya.

### ISP ke ASA Outside

Perintah:

```text
ping 192.168.20.1
```

Hasil:

```text
Success rate is 100 percent (5/5)
```

**Status: Berhasil.**

Router ISP dapat menjangkau interface outside ASA.

### Ringkasan Pengujian

| Sumber | Tujuan | Hasil |
|---|---|---|
| PC Finance | `192.168.20.2` — ISP | Berhasil, 4/4 |
| PC Finance | `192.168.30.10` — Server | Berhasil, 4/4 |
| PC Finance | `192.168.10.10` — PC IT | Diblokir, 0/4 diterima |
| Router ISP | `192.168.30.10` — Server | Berhasil, 5/5 |
| Router ISP | `192.168.20.1` — ASA Outside | Berhasil, 5/5 |

Hasil di atas mendokumentasikan pengujian yang telah dilakukan. Hasil pengujian lainnya sebaiknya dicatat setelah dijalankan ulang dan diverifikasi.

## Verifikasi Konfigurasi

### Status Interface ASA

```text
show interface ip brief
```

Digunakan untuk memeriksa status interface ASA dan alamat IP yang dikonfigurasi.

### Routing ASA

```text
show route
```

Digunakan untuk memastikan connected route dan static route menuju jaringan server tersedia.

### Tabel ARP

```text
show arp
```

Digunakan untuk memeriksa alamat IP dan MAC address perangkat yang telah dikenali ASA.

### ACL dan Hit Counter

```text
show access-list
```

Digunakan untuk melihat aturan ACL beserta hit counter. Hit counter membantu membuktikan bahwa trafik pengujian cocok dengan aturan tertentu.

### Verifikasi Switch

```text
show vlan brief
show interfaces status
show mac address-table
show cdp neighbors
show spanning-tree
```

Perintah tersebut dapat digunakan untuk memeriksa VLAN, status port, MAC address, tetangga perangkat, dan status spanning tree.

Ketersediaan perintah tertentu dapat berbeda sesuai perangkat dan versi simulasi Packet Tracer.

## Dokumentasi Pengujian

Screenshot yang disarankan untuk dokumentasi project:

1. Topologi lengkap Cisco Packet Tracer.
2. Konfigurasi interface ASA.
3. Output `show route` pada ASA.
4. Output `show access-list` beserta hit counter.
5. Hasil ping PC Finance ke server.
6. Hasil ping PC Finance ke PC IT.
7. Hasil ping router ISP ke server.
8. Verifikasi VLAN dan status port switch.

Gunakan screenshot dari pengujian aktual agar dokumentasi sesuai dengan kondisi simulasi.

## Troubleshooting

### Ping Finance ke ISP Gagal

**Gejala:** PC Finance tidak dapat melakukan ping ke router ISP.

**Pemeriksaan:** Alamat IP, default gateway, interface ASA, serta ACL pada interface outside diperiksa.

**Perbaikan:** Aturan ACL ICMP yang sesuai ditambahkan untuk mengizinkan trafik dari PC Finance menuju router ISP.

**Hasil:** Ping Finance ke `192.168.20.2` berhasil dengan 4/4 paket diterima.

### Ping Finance ke Server Gagal

**Gejala:** PC Finance tidak dapat melakukan ping ke server.

**Pemeriksaan:** Routing ASA menuju jaringan server, route pada router ISP, dan ACL outside diperiksa.

**Perbaikan:** Aturan ICMP yang sesuai ditambahkan untuk mengizinkan trafik dari PC Finance menuju server.

**Hasil:** Ping Finance ke `192.168.30.10` berhasil dengan 4/4 paket diterima.

### Ping Finance ke IT Tidak Berhasil

**Gejala:** PC Finance tidak menerima balasan ping dari PC IT.

**Pemeriksaan:** Aturan ACL `OUTSIDE-TO-INSIDE` diperiksa menggunakan `show access-list`.

**Hasil:** Aturan deny ICMP untuk trafik dari `192.168.20.10` menuju `192.168.10.10` menunjukkan `hitcnt=4`.

Hal tersebut sesuai dengan target pengujian untuk memblokir ICMP dari Finance ke IT.

## Ringkasan Keterampilan

Project ini mendemonstrasikan keterampilan dasar jaringan dan keamanan berikut:

- Konfigurasi Cisco ASA 5505.
- Pengalamatan IPv4 dan default gateway.
- Pemahaman interface inside dan outside.
- Segmentasi jaringan menggunakan VLAN.
- Konfigurasi static routing.
- Penerapan ACL berbasis alamat IP dan protokol.
- ICMP inspection.
- Pengujian konektivitas jaringan.
- Verifikasi routing, ARP, dan ACL.
- Troubleshooting konektivitas menggunakan pendekatan bertahap.
- Dokumentasi konfigurasi dan hasil pengujian.

## Struktur File Project

Struktur repository yang disarankan:

```text
Cisco_ASA_Enterprise_Network/
├── README.md
├── Cisco_ASA_Enterprise_Network.pkt
└── screenshots/
    ├── topology.png
    ├── asa-config.png
    ├── acl-verification.png
    ├── finance-to-server.png
    ├── finance-to-it.png
    └── routing-verification.png
```

Pastikan file `.pkt` dan screenshot benar-benar tersedia sebelum repository dipublikasikan. Nama file dapat disesuaikan dengan file yang digunakan.

## Status Project

**Status:** Simulasi dasar selesai dan pengujian ICMP utama telah dilakukan.

Fitur yang telah dikonfigurasi dan diuji:

- Cisco ASA sebagai firewall.
- Interface inside dan outside.
- Static routing antara ASA dan ISP.
- ACL untuk mengizinkan ICMP ke tujuan tertentu.
- ACL untuk menolak ICMP dari Finance ke IT.
- ICMP inspection.
- Verifikasi ACL menggunakan hit counter.

Pengembangan selanjutnya dapat mencakup pengujian trafik HTTP/HTTPS, penerapan kebijakan least privilege yang lebih lengkap, dan pengujian tambahan untuk setiap jalur komunikasi.

## Kesimpulan

Project ini menunjukkan penerapan dasar firewall dan pengendalian trafik menggunakan Cisco ASA 5505 di Cisco Packet Tracer.

Hasil pengujian menunjukkan bahwa PC Finance dapat melakukan ping ke router ISP dan server, sedangkan ICMP dari PC Finance menuju PC IT ditolak oleh aturan ACL yang sesuai.

Simulasi ini menjadi dasar untuk mempelajari firewall, segmentasi jaringan, routing, ACL, dan troubleshooting dalam lingkungan jaringan yang lebih kompleks.
