# Simulasi Jaringan dan Keamanan Cisco ASA

## Deskripsi

Proyek ini merupakan simulasi jaringan menggunakan Cisco Packet Tracer yang menerapkan segmentasi jaringan, routing, serta firewall menggunakan Cisco ASA.

Topologi terdiri dari jaringan **IT**, **Finance**, dan **Server**, dengan Cisco ASA sebagai perangkat keamanan yang mengatur lalu lintas antara jaringan internal dan jaringan luar.

Tujuan utama simulasi ini adalah memahami:

- Konfigurasi Cisco ASA sebagai firewall.
- Penggunaan VLAN dan segmentasi jaringan.
- Routing antar jaringan.
- Penerapan Access Control List (ACL).
- Pengujian konektivitas menggunakan `ping`.
- Verifikasi konfigurasi menggunakan perintah `show`.

---

## Network Design Overview

```text
                 +----------------+
                 |   Cisco ASA    |
                 |    Firewall    |
                 +-------+--------+
                         |
                    Outside Network
                         |
                 +-------+--------+
                 |      ISP       |
                 +---+---------+--+
                     |         |
                  IT/Inside   Server
```

---

## Skema Alamat IP

| Perangkat | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| Cisco ASA | Inside | 192.168.10.1 | 255.255.255.0 |
| Cisco ASA | Outside | 192.168.20.1 | 255.255.255.0 |
| ISP | G0/0 | 192.168.20.2 | 255.255.255.0 |
| ISP | G0/1 | 192.168.30.1 | 255.255.255.0 |
| PC IT | - | 192.168.10.10 | 255.255.255.0 |
| PC Finance | - | 192.168.20.10 | 255.255.255.0 |
| Server | - | 192.168.30.10 | 255.255.255.0 |

### Default Gateway

- PC IT → `192.168.10.1`
- PC Finance → `192.168.20.1`
- Server → `192.168.30.1`

---

## Topologi

Topologi menggunakan Cisco ASA sebagai firewall antara jaringan internal dan jaringan luar.

Cisco ASA memiliki dua interface logis:

- **Inside** → `192.168.10.1/24`
- **Outside** → `192.168.20.1/24`

Jaringan server berada di belakang router ISP pada jaringan:

- `192.168.30.0/24`

---

## Konfigurasi Cisco ASA

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

Port Ethernet yang digunakan pada sisi outside dikonfigurasi sebagai VLAN 2.

```text
interface Ethernet0/1
 switchport access vlan 2

interface Ethernet0/2
 switchport access vlan 2
```

---

## Routing

Cisco ASA memiliki route menuju jaringan server melalui ISP:

```text
route outside 192.168.30.0 255.255.255.0 192.168.20.2 1
```

ISP memiliki route menuju jaringan internal melalui Cisco ASA:

```text
ip route 0.0.0.0 0.0.0.0 192.168.20.1
ip route 192.168.10.0 255.255.255.0 192.168.20.1
```

---

## Access Control List

Cisco ASA menggunakan dua ACL utama:

- `INSIDE-FILTER`
- `OUTSIDE-TO-INSIDE`

### INSIDE-FILTER

ACL ini digunakan untuk mengatur traffic yang masuk dari interface inside.

```text
access-list INSIDE-FILTER extended permit ip 192.168.10.0 255.255.255.0 192.168.30.0 255.255.255.0
access-list INSIDE-FILTER extended permit ip 192.168.10.0 255.255.255.0 192.168.20.0 255.255.255.0
```

ACL diterapkan pada interface inside:

```text
access-group INSIDE-FILTER in interface inside
```

### OUTSIDE-TO-INSIDE

ACL ini mengatur traffic tertentu yang berasal dari jaringan outside.

```text
access-list OUTSIDE-TO-INSIDE extended permit icmp 192.168.30.0 255.255.255.0 192.168.10.0 255.255.255.0
access-list OUTSIDE-TO-INSIDE extended permit icmp 192.168.20.0 255.255.255.0 192.168.30.0 255.255.255.0
access-list OUTSIDE-TO-INSIDE extended permit icmp host 192.168.20.2 host 192.168.20.1
access-list OUTSIDE-TO-INSIDE extended permit icmp host 192.168.30.10 host 192.168.10.10
access-list OUTSIDE-TO-INSIDE extended permit icmp 192.168.20.0 255.255.255.0 192.168.10.0 255.255.255.0
```

ACL diterapkan pada interface outside:

```text
access-group OUTSIDE-TO-INSIDE in interface outside
```

---

## ICMP Inspection

Cisco ASA menggunakan inspection untuk traffic ICMP:

```text
class-map inspection_default
 match default-inspection-traffic

policy-map global_policy
 class inspection_default
  inspect icmp

service-policy global_policy global
```

Konfigurasi ini membantu Cisco ASA menangani traffic ICMP yang melewati firewall.

---

## Pengujian Konektivitas

Pengujian dilakukan menggunakan perintah `ping` dari perangkat jaringan.

### IT → ASA Inside

```text
C:\>ping 192.168.10.1
```

Koneksi berhasil setelah konfigurasi interface dan jaringan diperbaiki.

### Finance → ASA Outside

```text
C:\>ping 192.168.20.1
```

Koneksi berhasil setelah port switch dan interface outside dikonfigurasi pada VLAN yang sesuai.

### IT → Server

```text
C:\>ping 192.168.30.10
```

Hasil pengujian akhir:

```text
Reply from 192.168.30.10
Reply from 192.168.30.10
Reply from 192.168.30.10
Reply from 192.168.30.10

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

### IT → Finance

```text
C:\>ping 192.168.20.10
```

Hasil pengujian:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

### Finance → IT

Traffic dari jaringan Finance menuju jaringan IT dikontrol oleh ACL pada Cisco ASA.

### Finance → Server

Traffic dari jaringan Finance menuju jaringan Server dikontrol berdasarkan konfigurasi ACL dan jalur routing yang tersedia.

---

## Verifikasi Cisco ASA

### Melihat VLAN

```text
show switch vlan
```

Hasil menunjukkan:

```text
VLAN 1  inside
VLAN 2  outside
```

### Melihat status interface

```text
show interface ip brief
```

Interface utama:

```text
Vlan1    192.168.10.1    up    up
Vlan2    192.168.20.1    up    up
```

### Melihat tabel ARP

```text
show arp
```

Tabel ARP menunjukkan perangkat pada jaringan inside dan outside telah dikenali oleh Cisco ASA.

### Melihat ACL

```text
show access-list
```

Perintah ini digunakan untuk melihat ACL beserta jumlah hit pada setiap rule.

Contoh:

```text
access-list INSIDE-FILTER line 1 extended permit ip 192.168.10.0 255.255.255.0 192.168.30.0 255.255.255.0
access-list INSIDE-FILTER line 2 extended permit ip 192.168.10.0 255.255.255.0 192.168.20.0 255.255.255.0
```

Hit counter digunakan sebagai indikator bahwa rule ACL telah memproses traffic.

---

## Verifikasi Switch

Konfigurasi switch diverifikasi menggunakan beberapa perintah:

```text
show vlan brief
show interfaces status
show mac address-table
show cdp neighbors
show spanning-tree
```

Perintah tersebut digunakan untuk memastikan:

- Port berada pada VLAN yang sesuai.
- Interface dalam kondisi `connected`.
- MAC address perangkat berhasil dipelajari.
- Hubungan antar-switch terdeteksi.
- Spanning Tree Protocol berjalan.

---

## Konfigurasi VLAN Outside

Pada switch yang terhubung dengan jaringan outside, VLAN 2 dibuat sebagai VLAN `OUTSIDE`.

```text
vlan 2
 name OUTSIDE
```

Port yang digunakan untuk perangkat outside dikonfigurasi sebagai access port:

```text
interface fa0/1
 switchport mode access
 switchport access vlan 2

interface fa0/2
 switchport mode access
 switchport access vlan 2
```

Verifikasi:

```text
show vlan brief
```

Hasil menunjukkan:

```text
2    OUTSIDE    active    Fa0/1, Fa0/2
```

---

## Konfigurasi ISP

Interface ISP yang digunakan:

```text
interface GigabitEthernet0/0
 ip address 192.168.20.2 255.255.255.0

interface GigabitEthernet0/1
 ip address 192.168.30.1 255.255.255.0
```

Routing pada ISP:

```text
ip route 0.0.0.0 0.0.0.0 192.168.20.1
ip route 192.168.10.0 255.255.255.0 192.168.20.1
```

---

## File Proyek

File simulasi Cisco Packet Tracer:

```text
*.pkt
```

File konfigurasi dan dokumentasi proyek disimpan bersama repository ini.

Struktur repository:

```text
.
├── README.md
├── project.pkt
└── screenshots/
    ├── topology.png
    ├── asa-config.png
    ├── connectivity-test.png
    └── verification.png
```

---

## Dokumentasi Screenshot

Screenshot digunakan untuk mendokumentasikan:

1. Topologi jaringan.
2. Konfigurasi Cisco ASA.
3. Konfigurasi routing.
4. Konfigurasi ACL.
5. Hasil pengujian konektivitas.
6. Verifikasi VLAN dan MAC address.
7. Status interface perangkat.

---

## Kesimpulan

Simulasi ini menunjukkan penerapan dasar keamanan jaringan menggunakan Cisco ASA sebagai firewall.

Konfigurasi mencakup:

- Segmentasi jaringan menggunakan VLAN.
- Pengalamatan IP.
- Routing antara beberapa jaringan.
- Cisco ASA sebagai firewall.
- Access Control List untuk mengatur traffic.
- ICMP inspection.
- Verifikasi konfigurasi menggunakan perintah `show`.
- Pengujian konektivitas antar jaringan.

Hasil pengujian digunakan untuk memastikan bahwa konfigurasi routing, interface, VLAN, dan ACL berjalan sesuai dengan rancangan jaringan.
