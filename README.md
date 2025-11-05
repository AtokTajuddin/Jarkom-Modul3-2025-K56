# Jarkom-Modul-3-K56-2025

# LAPORAN PRAKTIKUM MODUL-3

## Praktikum Komunikasi data & Jaringan Komputer kelompook K56

| Nama                    | NRP        |
| ----------------------- | ---------- |
| Mochammad Atha Tajuddin | 5027241093 |
| Muhammad Huda Rabbani   | 5027241098 |

---

# Network Topology Documentation

## Switch Distribution

| Switch       | Total Nodes | Connection To | Node Names                |
| ------------ | ----------- | ------------- | ------------------------- |
| **Switch 1** | 3 nodes     | Switch 8      | Elendil, Isildur, Anarion |
| **Switch 2** | 3 nodes     | Switch 5      | -                         |
| **Switch 3** | 1 node      | Switch 6      | -                         |
| **Switch 4** | 3 nodes     | -             | -                         |
| **Switch 5** | 3 nodes     | Switch 2      | -                         |
| **Switch 6** | 2 nodes     | Switch 3      | -                         |
| **Switch 8** | 3 nodes     | Switch 1      | -                         |

## Keluarga Manusia (Dynamic IP)

| Node Name | Role                    | IP Type |
| --------- | ----------------------- | ------- |
| Minastir  | Forward Proxy           | Dynamic |
| Aldarion  | DHCP Server             | Dynamic |
| Erendis   | DNS Master              | Dynamic |
| Elendil   | Laravel Worker-1        | Dynamic |
| Isildur   | Laravel Worker-2        | Dynamic |
| Anarion   | Laravel Worker-3        | Dynamic |
| Miriel    | Client-Static-1         | Static  |
| Pharazon  | Load Balancer (PHP)     | Dynamic |
| Palantir  | Database Server         | Dynamic |
| Elros     | Load Balancer (Laravel) | Dynamic |
| Amandil   | Client-Dynamic-2        | Dynamic |
| Khamul    | Client-Fixed-Address    | Fixed   |

## Keluarga Peri (Dynamic IP)

| Node Name   | Role             | IP Type |
| ----------- | ---------------- | ------- |
| Amdir       | DNS Slave        | Dynamic |
| Galadriel   | PHP Worker-1     | Dynamic |
| Celeborn    | PHP Worker-2     | Dynamic |
| Oropher     | PHP Worker-3     | Dynamic |
| Gilgalad    | Client-Dynamic-1 | Dynamic |
| Celebrimbor | Client-Static-2  | Static  |

## Summary Statistics

| Category                | Total    |
| ----------------------- | -------- |
| **Total Switches**      | 7        |
| **Total Nodes**         | 18       |
| **Dynamic IP Nodes**    | 14       |
| **Static IP Nodes**     | 2        |
| **Fixed Address Nodes** | 1        |
| **Keluarga Manusia**    | 12 nodes |
| **Keluarga Peri**       | 6 nodes  |

---

#### Keluarga Dwarf

Durin (Sudah menjadi dhcp relay sekaligus router)

Khamul (IP Static )

Untuk ubah konfigurasi kita dapat melakukannya via CLI saja tanpa perlu ke interface GNS3

```
nano /etc/network/interfaces
```

GIlgalad & Khamul menjadi client DHCP dalam artian konfigurasi mereka dynamic tidak perlu define static ip

## Nomor 1

#### Durin

```
auto lo
iface lo inet loopback

# ==================================
# KONEKSI KE INTERNET (NAT1)
# ==================================
# Biarkan DHCP bekerja otomatis
# Dia akan mengambil IP dan DNS dari NAT1
auto eth0
iface eth0 inet dhcp

# ==================================
# JARINGAN INTERNAL (Sudah Benar)
# ==================================
auto eth1
iface eth1 inet static
        address 192.239.1.1
        netmask 255.255.255.0

auto eth2
iface eth2 inet static
        address 192.239.2.1
        netmask 255.255.255.0

auto eth3
iface eth3 inet static
        address 192.239.3.1
        netmask 255.255.255.0

auto eth4
iface eth4 inet static
        address 192.239.4.1
        netmask 255.255.255.0

auto eth5
iface eth5 inet static
        address 192.239.5.1
        netmask 255.255.255.0

# ==================================
# NAT UNTUK KLIEN (Sudah Benar)
# ==================================
# Baris ini untuk klien, BUKAN untuk router Durin
up iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.239.0.0/16
```

#### Aldarion

Konfigurasi node

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
        address 192.239.1.1
        netmask 255.255.255.0

auto eth2
iface eth2 inet static
        address 192.239.2.1
        netmask 255.255.255.0

auto eth3
iface eth3 inet static
        address 192.239.3.1
        netmask 255.255.255.0

auto eth4
iface eth4 inet static
        address 192.239.4.1
        netmask 255.255.255.0

auto eth5
iface eth5 inet static
        address 192.239.5.1
        netmask 255.255.255.0

up iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.239.0.0/16
```

Setup awal & setup dhcp relay (Node Durin)

```
apt update && apt-get install -y isc-dhcp-relay

nano /etc/default/isc-dhcp-relay

# /etc/default/isc-dhcp-relay

# 1. Tentukan alamat IP DHCP Server (Aldarion)
# Pastikan ini IP statis Aldarion yang benar.
SERVERS="192.239.4.2"

# 2. Tentukan di interface mana Durin harus "mendengarkan"
#    permintaan DHCP dari klien.
#
#    PENTING:
#    - Masukkan SEMUA interface yang mengarah ke klien.
#    - JANGAN masukkan eth4 (karena Server ada di sana).
#    - JANGAN masukkan eth0 (karena itu ke Internet/NAT1).
INTERFACES="eth1 eth2 eth3 eth5"

# (Di console Durin)
nano /etc/default/isc-dhcp-relay

# What servers should the DHCP relay forward requests to?
SERVERS="192.239.4.2"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth1 eth2 eth3 eth5"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS="-a -i eth4"

service isc-dhcp-relay restart
```

#### Client

Konfig pada GNS3

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
         address 192.239.1.2
         netmask 255.255.255.0
         gateway 192.239.1.1

up echo nameserver 192.168.122.1 > /etc/resolv.conf
```

Semuanya sama tinggal ganti ip belakang dan gateway disesuaikan sesuai konfigurasi router (Durin)

---

## Nomor 2

#### Aldarion

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
         address 192.239.4.2
         netmask 255.255.255.0
         gateway 192.239.4.1

up echo nameserver 192.168.122.1 > /etc/resolv.conf
```

```
# 1. Pastikan Aldarion bisa konek internet (sesuai Soal 1)
apt-get update

# 2. Instal software servernya
apt-get install -y isc-dhcp-server


nano /etc/default/isc-dhcp-server

# Tentukan interface mana yang melayani DHCPv4 (ke eth0 Aldarion)
INTERFACESv4="eth0"



nano /etc/dhcp/dhcpd.conf
# /etc/dhcp/dhcpd.conf (di node ALDARION)

# === OPSI GLOBAL ===
# (Ini masih sama, tapi kita perlu memikirkan
# apakah nameserver sementara masih diperlukan.
# Untuk saat ini, kita biarkan saja.)
option domain-name-servers 192.168.122.1;
option domain-name "praktikum.local";
default-lease-time 7200;
max-lease-time 86400;
authoritative;

# === JARINGAN 1 (Manusia) ===
subnet 192.239.1.0 netmask 255.255.255.0 {
    option routers 192.239.1.1;

    # DEKRIT BARU: Tanah untuk Klien Dinamis Keluarga Manusia
    range 192.239.1.6 192.239.1.34;
    range 192.239.1.68 192.239.1.94;
}

# === JARINGAN 2 (Peri) ===
subnet 192.239.2.0 netmask 255.255.255.0 {
    option routers 192.239.2.1;

    # DEKRIT BARU: Tanah untuk Klien Dinamis Keluarga Peri
    # (Pool lama dari Soal 1 dihapus dan diganti)
    range 192.239.2.35 192.239.2.67;
    range 192.239.2.96 192.239.2.121;
}

# === JARINGAN 3 (Khamul & Amandil) ===
subnet 192.239.3.0 netmask 255.255.255.0 {
    option routers 192.239.3.1;

    # DEKRIT BARU: Tanah Tetap Khamul
    # (Pool lama dari Soal 1 dihapus dan diganti)
    host Khamul {
        # Anda HARUS mengganti MAC Address ini
        # sesuai dengan node 'Khamul' Anda!
        hardware ethernet 02:42:f2:39:a2:00;
        fixed-address 192.239.3.95;
    }

    # (Lihat Analisis Penting di bawah)
}

# === JARINGAN 4 (Server - Sisi Aldarion) ===
subnet 192.239.4.0 netmask 255.255.255.0 {
    option routers 192.239.4.1;
    # (Tidak ada pool dinamis di sini)
}

# === JARINGAN 5 (Minastir) ===
subnet 192.239.5.0 netmask 255.255.255.0 {
    option routers 192.239.5.1;
    # (Tidak ada pool dinamis di sini)
}

service isc-dhcp-server start

Cara testing :

dhcrelay -d -i eth1 -i eth2 -i eth3 -i eth5 192.239.4.2
dhcp -t (Untuk cek sintaks konfigurasi)


Cek dhcp pada  (Di console Aldarion)
Start dulu node yang memiliki dynamic IP

cat /var/lib/dhcp/dhcpd.leases
```

```
#!/bin/bash
# ==========================================================
# SKRIP OTOMATIS KONFIGURASI DHCP SERVER (NODE: ALDARION)
# (Versi Gabungan dengan DNS Final)
# ==========================================================

echo "=== [Aldarion] Memulai Konfigurasi DHCP Server ==="

# 1. Menginstal paket yang diperlukan
echo "--> Mengupdate paket dan menginstal isc-dhcp-server..."
apt-get update
apt-get install -y isc-dhcp-server

# 2. Membuat file /etc/default/isc-dhcp-server
echo "--> Menulis file /etc/default/isc-dhcp-server..."
cat << EOF > /etc/default/isc-dhcp-server
# File ini dibuat otomatis oleh skrip
# Tentukan interface mana yang melayani DHCPv4
INTERFACESv4="eth0"
# Nonaktifkan IPv6 agar tidak 'failed'
INTERFACESv6=""
EOF

# 3. Membuat file konfigurasi utama /etc/dhcp/dhcpd.conf
echo "--> Menulis file konfigurasi utama /etc/dhcp/dhcpd.conf..."
cat << EOF > /etc/dhcp/dhcpd.conf
# File ini dibuat otomatis oleh skrip

# === OPSI GLOBAL ===
# (MODIFIKASI: Langsung menggunakan DNS Minastir & domain k56.com)
option domain-name-servers 192.239.5.2;
option domain-name "k56.com";
max-lease-time 3600; # Batas 1 jam (Soal 6)
authoritative;

# === JARINGAN 1 (Manusia) ===
subnet 192.239.1.0 netmask 255.255.255.0 {
    option routers 192.239.1.1;
    range 192.239.1.6 192.239.1.34;
    range 192.239.1.68 192.239.1.94;
    default-lease-time 1800; # 30 menit (Soal 6)
}

# === JARINGAN 2 (Peri) ===
subnet 192.239.2.0 netmask 255.255.255.0 {
    option routers 192.239.2.1;
    range 192.239.2.35 192.239.2.67;
    range 192.239.2.96 192.239.2.121;
    default-lease-time 600; # 10 menit (Soal 6)
}

# === JARINGAN 3 (Khamul & Amandil) ===
subnet 192.239.3.0 netmask 255.255.255.0 {
    option routers 192.239.3.1;

    # (MAC Address Khamul dari log Anda)
    host Khamul {
        hardware ethernet 02:42:f2:39:a2:00;
        fixed-address 192.239.3.95;
    }
}

# === JARINGAN 4 (Server - Sisi Aldarion) ===
subnet 192.239.4.0 netmask 255.255.255.0 {
    option routers 192.239.4.1;
    # (Tidak ada pool dinamis di sini)
}

# === JARINGAN 5 (Minastir) ===
subnet 192.239.5.0 netmask 255.255.255.0 {
    option routers 192.239.5.1;
    # (Tidak ada pool dinamis di sini)
}
EOF

# 4. Memeriksa sintaks konfigurasi
echo "--> Memeriksa sintaks konfigurasi..."
dhcpd -t

# 5. Merestart layanan untuk menerapkan konfigurasi
echo "--> Merestart layanan isc-dhcp-server..."
service isc-dhcp-server restart

echo "=== [Aldarion] Konfigurasi DHCP Server Selesai ==="

# 6. Menampilkan status akhir
echo "--> Cek status akhir:"
service isc-dhcp-server status
```

Harusnya begini hasilnya jika berhasil

```
root@Aldarion:~# cat /var/lib/dhcp/dhcpd.leases
  starts 4 2025/10/30 08:29:52;
  ends 4 2025/10/30 08:39:52;
  cltt 4 2025/10/30 08:29:52;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 02:42:76:4a:2e:00;
  uid "\001\002BvJ.\000";
  set vendor-class-identifier = "udhcp 1.36.1";
  option agent.circuit-id "eth2";
}
```

```
root@Aldarion:~# cat /var/lib/dhcp/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.4.3-P1

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

lease 192.239.2.35 {
  starts 2 2025/10/28 16:15:07;
  ends 2 2025/10/28 18:15:07;
  tstp 2 2025/10/28 18:15:07;
  cltt 2 2025/10/28 16:15:07;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 02:42:76:4a:2e:00;
  uid "\001\002BvJ.\000";
  set vendor-class-identifier = "udhcp 1.36.1";
  option agent.circuit-id "eth2";
}
server-duid "\000\001\000\0010\223\240E\002B\265K\335\000";

lease 192.239.1.6 {
  starts 2 2025/10/28 16:30:05;
  ends 2 2025/10/28 18:30:05;
  cltt 2 2025/10/28 16:30:05;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 02:42:45:e7:9f:00;
  uid "\001\002BE\347\237\000";
  set vendor-class-identifier = "udhcp 1.36.1";
  option agent.circuit-id "eth1";
}
```

```
apt update
apt-get install -y dnsmasq iptables-persistent
```

in case dynamic di sini ada Gilgalad dan Amandil

---

## Nomor 3 (Old,untuk dokumentasi saja sayang dibuang :v )

#### Minastir

Konfigurasi

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
    # Alamat IP statis Minastir (sudah benar)
    address 192.239.5.2

    # Netmask (sudah benar)
    netmask 255.255.255.0

    # Gateway (Durin eth5, sudah benar)
    gateway 192.239.5.1

    # Cara yang benar untuk mengatur DNS (agar bisa apt install squid)
    dns-nameservers 192.168.122.1
```

```
apt update -y
apt install squid -y
apt-get install -y iptables-persistent


[!!] Sebelum lanjut pada squid conf jikalau ingin menghilangkan semua confignya maka bisa seperti ini

# (Di console Minastir)
# Backup file konfigurasi utama
mv /etc/squid/squid.conf /etc/squid/squid.conf.BAK
# Nonaktifkan direktori conf.d agar tidak dimuat
mv /etc/squid/conf.d /etc/squid/conf.d.BAK




nano /etc/squid/squid.conf


#  KONFIGURASI PROXY MINASTIR
# 1. PORT LISTENING (Harus di atas)
http_port 0.0.0.0:3128

# 2. ACL (ACCESS CONTROL LIST)
acl localnet src 192.239.0.0/16
acl localhost src 127.0.0.1
acl SSL_ports port 443
acl Safe_ports port 80
acl CONNECT method CONNECT

# 3. ATURAN AKSES (URUTAN SANGAT PENTING)
# ------------------------------
# Izinkan akses dari localhost (untuk Squid sendiri)
http_access allow localhost

# Izinkan jaringan internal kita
http_access allow localnet

# Blokir semua yang lain
http_access deny all

# Tolak port berbahaya (ini bisa diletakkan di atas 'allow')
http_access deny !Safe_ports
# Tolak CONNECT ke port selain SSL
http_access deny CONNECT !SSL_ports

# 4. PENGATURAN LAINNYA
visible_hostname minastir
cache_dir ufs /var/spool/squid 100 16 256
coredump_dir /var/spool/squid
cache_effective_user proxy
cache_effective_group proxy

# (AKHIR DARI FILE)

service squid restart
service squid status

Ada error ? cek dengan ini :
# -N: Jangan jadi daemon (tetap di foreground)
# -d1: Tampilkan log debug level 1
squid -N -d1
Keterangan :
-d1 = debug level 1


tail -f /var/log/squid/access.log
```

```
[Lanjut, pada Gilgalad]
apt-get install -y net-tools
apt-get install -y iptables-persistent




nano /etc/apt/apt.conf.d/01proxy

Acquire::http::Proxy "http://192.239.5.2:3128";



[Cek pada durin & gilgalad]
# (Di console Durin)

iptables -A INPUT -p tcp --dport 3128 -j ACCEPT
netfilter-persistent save


Coba curl pada Gilgalad
curl -v http://example.com

Cek di Minastir :
tail -f /var/log/squid/access.log
```

```
[Bukti kalau Gilgalad bisa forward proxy ?]
.
root@Minastir:~#
root@Minastir:~# squid[4875]: Squid Parent: (squid-1) process 4877 started

root@Minastir:~# tail -f /var/log/squid/access.log
1761672845.515    678 192.239.2.35 TCP_MISS/200 847 GET http://example.com/ - HIER_DIRECT/23.215.0.138 text/html
1761672860.501    221 192.239.2.35 TCP_MISS/304 359 GET http://deb.debian.org/debian/dists/stable/InRelease - HIER_DIRECT/146.75.46.132 -
1761672860.525     22 192.239.2.35 TCP_MISS/304 357 GET http://deb.debian.org/debian/dists/stable-updates/InRelease - HIER_DIRECT/146.75.46.132 -
1761672860.696    170 192.239.2.35 TCP_MISS/304 398 GET http://deb.debian.org/debian-security/dists/stable-security/InRelease - HIER_DIRECT/146.75.46.132 -
```

---

# Fix (DNS Forwarders)

#### Minastir

```
apt-get update -y && apt install bind9 -y
ln -s /etc/init.d/named /etc/init.d/bind9
nano /etc/bind/named.conf.options

// nano /etc/bind/named.conf.options

// nano /etc/bind/named.conf.options

options {
    directory "/var/cache/bind";

    # Dengarkan di IP Minastir
    listen-on { 127.0.0.1; 192.239.5.2; };

    # Izinkan kueri dari semua jaringan Arda
    allow-query { localhost; 192.239.0.0/16; };

    # ==================================
    # INTI FORWARDER (DENGAN PENAMBAHAN)
    # ==================================
    forwarders {
        # 1. Coba tanya DNS GNS3 dulu
        192.168.122.1;

        # 2. Jika gagal, tanya Google DNS
        8.8.8.8;
        8.8.4.4;
    };
    # Ini memaksa bind9 untuk HANYA meneruskan
    forward only;
    # ==================================

    # Aktifkan Response Policy Zone (RPZ) untuk filtering
    response-policy {
        zone "rpz.blacklist.local";
    };

    dnssec-validation auto;
    # Nonaktifkan IPv6 untuk GNS3 agar lebih stabil
    listen-on-v6 { none; };
};


// Jika dibutuhkan bisa tambahkan zona blacklist

nano /etc/bind/db.rpz.blacklist

; File database untuk RPZ Blacklist
$TTL 60
@       IN      SOA     localhost. root.localhost. (
                        1       ; Serial
                        60      ; Refresh
                        60      ; Retry
                        60      ; Expire
                        60 )    ; Negative Cache TTL
        IN      NS      localhost.

; ==================================
; INTI FILTERING
; ==================================
; Arahkan domain ini ke 0.0.0.0
facebook.com      IN      A       0.0.0.0
*.facebook.com    IN      A       0.0.0.0

; Anda juga bisa mengarahkannya ke 'Tidak Ditemukan' (NXDOMAIN)
twitter.com       IN      CNAME   .
*.twitter.com     IN      CNAME   .


named-checkconf -z
chown bind:bind /etc/bind/db.rpz.blacklist
named-checkzone rpz.blacklist.local /etc/bind/db.rpz.blacklist
service bind9 restart

```

---

## Nomor 4

Menggunakan method addressing host id/network id 3.4 karena sesuai atau mengikuti alur dari banyaknya interface eth yang ada di mana eth hanya terbatas hingg eth5 maka dari itu kita harus melakukan pemikiran ektstra terkait ini, jika kita memaksakan menambahkan hingga eth7 sesuai jumlah switch maka Router Durin akan mengalami error di mana tidak menemukan interfacen eth6 & eth7.

---

#### Erendis

```
# (Di console Erendis)
auto eth0
iface eth0 inet static
    address 192.239.3.3
    netmask 255.255.255.0
    gateway 192.239.3.1
    # DNS sementara untuk instalasi
    dns-nameservers 192.168.122.1
```

#### Erendis

```
# (Di console Erendis)
apt-get update
apt-get install -y bind9 dnsutils
ln -s /etc/init.d/named /etc/init.d/bind9

# (Di console Erendis)

nano /etc/bind/named.conf.options

options {
    directory "/var/cache/bind";

    // ... baris lain ...

    // Izinkan Amdir (IP Slave) untuk mengambil salinan
    allow-transfer { 192.239.4.12; };

    dnssec-validation auto;
    listen-on-v6 { any; };
};


nano /etc/bind/named.conf.local


zone "k56.com" {
    type master;
    file "/etc/bind/zones/db.k56.com"; // Lokasi file peta kita
    allow-update { none; };
};


mkdir /etc/bind/zones
# (Di console Erendis)
nano /etc/bind/zones/db.k56.com


; === FILE ZONA UNTUK k56.com ===

$TTL 604800
@   IN  SOA  ns1.k56.com. root.k56.com. (
            1         ; Serial (Ubah ini setiap kali Anda mengedit!)
            604800    ; Refresh
            86400     ; Retry
            2419200   ; Expire
            604800 )  ; Negative Cache TTL

; --- Penjaga Peta (Nameserver) ---
@           IN  NS  ns1.k56.com.
@           IN  NS  ns2.k56.com.

; --- Alamat Penjaga Peta (Wajib) ---
ns1         IN  A   192.239.3.3  ; IP Erendis
ns2         IN  A   192.239.3.4  ; IP Amdir

; --- Lokasi Penting (Sesuai Soal) ---
palantir    IN  A   192.239.4.4
elros       IN  A   192.239.1.7
pharazon    IN  A   192.239.2.7
elendil     IN  A   192.239.1.2
isildur     IN  A   192.239.1.3
anarion     IN  A   192.239.1.4
galadriel   IN  A   192.239.2.2
celeborn    IN  A   192.239.2.3
oropher     IN  A   192.239.2.4

named-checkconf
named-checkzone k56.com /etc/bind/zones/db.k56.com
```

---

#### Amdir

```
# (Di console Amdir)
auto eth0
iface eth0 inet static
    address 192.239.3.4
    netmask 255.255.255.0
    gateway 192.239.3.1
    dns-nameservers 192.168.122.1
```

```
# (Di console Amdir)
apt-get update
apt-get install -y bind9 dnsutils
ln -s /etc/init.d/named /etc/init.d/bind9

nano /etc/bind/named.conf.local

// (Di /etc/bind/named.conf.local)
zone "k56.com" {
    type slave;
    file "db.k56.com"; // Nama file lokal untuk menyimpan salinan
    masters { 192.239.3.3; }; // IP Erendis (Master)
};

named-checkconf
service bind9 restart
```

---

#### Aldarion

```
nano /etc/dhcp/dhcpd.conf

# (Di dhcpd.conf Aldarion)
option domain-name-servers 192.239.3.3, 192.239.3.4;
option domain-name "k56.com";


# -R = Rekursif (termasuk semua file di dalamnya)
[Pada node erendis]
chown -R bind:bind /etc/bind/zones


[Perhatikan !!!, setup nameserver untuk semua client]

search k56.com
nameserver 192.239.3.3
nameserver 192.239.3.4
search praktikum.local
nameserver 192.168.122.1
```

## Nomor 5

#### Erendis

```
nano /etc/bind/zones/db.k56.com

Kita ubah serial number
@   IN  SOA  ns1.k56.com. root.k56.com. (
            2         ; <<< UBAH INI (misalnya dari 1 menjadi 2)
            ...


Masukkan (Tambahkan) script berikut di dalam db.k56.com

; ... (catatan 'oropher' Anda ada di sini) ...

; --- Alias (Soal 5) ---
; Alamat untuk domain utama (menunjuk ke Elros)
@           IN  A   192.239.1.7
; Alias www (menunjuk ke domain utama '@')
www         IN  CNAME k56.com.

; --- Pesan Rahasia (Soal 5) ---
@           IN  TXT   "Cincin Sauron: Elros"
@           IN  TXT   "Aliansi Terakhir: Pharazon"

nano /etc/bind/named.conf.local

zone "3.239.192.in-addr.arpa"{
 type master;
 file "/etc/bind/zones/db.192.239.3";
};


# (Di console Erendis)
nano /etc/bind/zones/db.192.239.3

; === FILE ZONA REVERSE UNTUK 192.239.3.x ===

$TTL 604800
@   IN  SOA  ns1.k56.com. root.k56.com. (
            1         ; Serial (mulai dari 1)
            604800
            86400
            2419200
            604800 )

; --- Nameserver ---
@   IN  NS  ns1.k56.com.
@   IN  NS  ns2.k56.com.

; --- Catatan PTR (Pointer) ---
; (Hanya angka terakhir dari IP)
3   IN  PTR ns1.k56.com.  ; 192.239.3.3
4   IN  PTR ns2.k56.com.  ; 192.239.3.4 (Asumsi IP Amdir)
10  IN  PTR elros.k56.com.  ; 192.239.1.7


# (Di console Erendis)
chown bind:bind /etc/bind/zones/db.192.239.3


[Cek sintaks]
named-checkconf
named-checkzone k56.com /etc/bind/zones/db.k56.com
named-checkzone 3.239.192.in-addr.arpa /etc/bind/zones/db.192.239.3


service bind9 restart

```

---

#### Amdir

```
# (Di console Amdir)
Tambahkan scriptnya ini

nano /etc/bind/named.conf.local


zone "3.239.192.in-addr.arpa" {
type slave;
file "db.192.239.3";
masters { 192.239.3.3; }; // IP Erendis (Master)
};
```

### Pengujian

```



[Pada erendis]

root@Erendis:~# dig k56.com TXT

; <<>> DiG 9.20.15-1~deb13u1-Debian <<>> k56.com TXT
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64172
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 9de1b8e57105061b0100000069031bff634b493f8fa4276a (good)
;; QUESTION SECTION:
;k56.com.                       IN      TXT

;; ANSWER SECTION:
k56.com.                604800  IN      TXT     "Cincin Sauron: Elros"
k56.com.                604800  IN      TXT     "Aliansi Terakhir: Pharazon"

;; Query time: 1 msec
;; SERVER: 192.239.3.3#53(192.239.3.3) (UDP)
;; WHEN: Thu Oct 30 08:04:15 UTC 2025
;; MSG SIZE  rcvd: 136

root@Erendis:~# dig -x 192.239.3.3

; <<>> DiG 9.20.15-1~deb13u1-Debian <<>> -x 192.239.3.3
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19193
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: fe26c02a1bfa28e60100000069031c05b3c2cdfe1da5bdd5 (good)
;; QUESTION SECTION:
;3.3.239.192.in-addr.arpa.      IN      PTR

;; ANSWER SECTION:
3.3.239.192.in-addr.arpa. 604800 IN     PTR     ns1.k56.com.

;; Query time: 0 msec
;; SERVER: 192.239.3.3#53(192.239.3.3) (UDP)
;; WHEN: Thu Oct 30 08:04:21 UTC 2025
;; MSG SIZE  rcvd: 106

[Pada amdir]

root@Amdir:~# dig -x 192.239.3.4

; <<>> DiG 9.20.15-1~deb13u1-Debian <<>> -x 192.239.3.4
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18238
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 8eff74ac069a32240100000069031c2eb6d65141b9891b66 (good)
;; QUESTION SECTION:
;4.3.239.192.in-addr.arpa.      IN      PTR

;; ANSWER SECTION:
4.3.239.192.in-addr.arpa. 604800 IN     PTR     ns2.k56.com.

;; Query time: 0 msec
;; SERVER: 192.239.3.3#53(192.239.3.3) (UDP)
;; WHEN: Thu Oct 30 08:05:02 UTC 2025
;; MSG SIZE  rcvd: 106

```

## Nomor 6

#### Aldarion

```
# (Di console Aldarion)
nano /etc/dhcp/dhcpd.conf

Ubah lease time jadi 3600 (3600 Detik)

# === OPSI GLOBAL ===
option domain-name-servers 192.239.3.3, 192.239.3.4;
option domain-name "k56.com";
# default-lease-time 7200; # Komentari atau hapus default global
max-lease-time 3600;      # <<< UBAH INI (1 jam)
authoritative;

# === JARINGAN 1 (Manusia) ===
subnet 192.239.1.0 netmask 255.255.255.0 {
    option routers 192.239.1.1;
    range 192.239.1.6 192.239.1.34;
    range 192.239.1.68 192.239.1.94;

    # Aturan Waktu Baru (Setengah Jam)
    default-lease-time 1800;  # <<< TAMBAHKAN INI (30 menit)
}

# === JARINGAN 2 (Peri) ===
subnet 192.239.2.0 netmask 255.255.255.0 {
    option routers 192.239.2.1;
    range 192.239.2.35 192.239.2.67;
    range 192.239.2.96 192.239.2.121;

    # Aturan Waktu Baru (Seperenam Jam)
    default-lease-time 600;   # <<< TAMBAHKAN INI (10 menit)
}

service isc-dhcp-server restart

```

#### Hasil uji

```
root@Aldarion:~# cat /var/lib/dhcp/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.4.3-P1

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

lease 192.239.2.35 {
  starts 4 2025/10/30 07:53:29;
  ends 4 2025/10/30 09:53:29;
  tstp 4 2025/10/30 09:53:29;
  cltt 4 2025/10/30 07:53:29;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 02:42:76:4a:2e:00;
  uid "\001\002BvJ.\000";
  set vendor-class-identifier = "udhcp 1.36.1";
  option agent.circuit-id "eth2";
}
server-duid "\000\001\000\0010\225\325\346\002B\265K\335\000";

```

## Nomor 7

#### Elendil

```
# (Di console Elendil)
nano /etc/network/interfaces


apt-get update
apt-get install -y nginx
apt-get install -y ca-certificates


curl -sS https://getcomposer.org/installer -o composer-setup.php

# (Di console Elendil)
apt-get install -y php8.4-fpm php8.4-cli php8.4-mbstring php8.4-xml php8.4-bcmath php8.4-curl php8.4-zip php8.4-mysql php8.4-gd php8.4-intl # Sesuaikan ekstensi jika perlu


# (On Elendil)

php composer-setup.php --install-dir=/usr/local/bin --filename=composer


# (On Elendil)
cd /var/www

composer create-project --prefer-dist laravel/laravel benteng-laravel
# Then navigate into the project directory
cd benteng-laravel

apt-get install -y php8.4-sqlite3
service php8.4-fpm restart

php artisan migrate
chown -R www-data:www-data storage bootstrap/cache
chmod -R 775 storage bootstrap/cache

ln -s /etc/nginx/sites-available/benteng-laravel /etc/nginx/sites-enabled/
nano /etc/nginx/sites-available/benteng-laravel


server {
    listen 80;
    listen [::]:80;
    server_name elendil.k56.com 192.239.1.2; # Ganti domain & IP jika perlu
    root /var/www/benteng-laravel/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}

[Proses selanjutnya]
nginx -t
chown -R www-data:www-data database/
chmod -R u+w database/

apt-get install -y lynx

lynx http://elendil.k56.com
lynx http://192.239.1.2 (Sesuaikan dengan alamat ip pada node yang menjadi tempat nginx dan laravel)
```

#### Anarion

```

server {
    listen 80;
    listen [::]:80;
    server_name anrion.k56.com 192.239.1.3; # Ganti domain & IP jika perlu
    root /var/www/benteng-laravel/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

#### Isildur

```

server {
    listen 80;
    listen [::]:80;
    server_name isildur.k56.com 192.239.1.4; # Ganti domain & IP jika perlu
    root /var/www/benteng-laravel/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

---

# Scripting

### Script 2

Sekaligus ada kaitannya pada nomor 4.

```
#!/bin/bash
# ==========================================================
# SKRIP OTOMATIS KONFIGURASI DHCP SERVER (NODE: ALDARION)
# (Versi Gabungan dengan DNS Final)
# ==========================================================
# Gabungan nomor 2 dan 4
echo "=== [Aldarion] Memulai Konfigurasi DHCP Server ==="

# 1. Menginstal paket yang diperlukan
echo "--> Mengupdate paket dan menginstal isc-dhcp-server..."
apt-get update
apt-get install -y isc-dhcp-server

# 2. Membuat file /etc/default/isc-dhcp-server
echo "--> Menulis file /etc/default/isc-dhcp-server..."
cat << EOF > /etc/default/isc-dhcp-server
# File ini dibuat otomatis oleh skrip
# Tentukan interface mana yang melayani DHCPv4
INTERFACESv4="eth0"
# Nonaktifkan IPv6 agar tidak 'failed'
INTERFACESv6=""
EOF

# 3. Membuat file konfigurasi utama /etc/dhcp/dhcpd.conf
echo "--> Menulis file konfigurasi utama /etc/dhcp/dhcpd.conf..."
cat << EOF > /etc/dhcp/dhcpd.conf
# File ini dibuat otomatis oleh skrip

# === OPSI GLOBAL ===
# (MODIFIKASI: Langsung menggunakan DNS Minastir & domain k56.com)
option domain-name-servers 192.239.5.2;
option domain-name-servers 192.239.3.3, 192.239.3.4;
option domain-name "k56.com";

max-lease-time 3600; # Batas 1 jam (Soal 6)
authoritative;

# === JARINGAN 1 (Manusia) ===
subnet 192.239.1.0 netmask 255.255.255.0 {
    option routers 192.239.1.1;
    range 192.239.1.6 192.239.1.34;
    range 192.239.1.68 192.239.1.94;
    default-lease-time 1800; # 30 menit (Soal 6)
}

# === JARINGAN 2 (Peri) ===
subnet 192.239.2.0 netmask 255.255.255.0 {
    option routers 192.239.2.1;
    range 192.239.2.35 192.239.2.67;
    range 192.239.2.96 192.239.2.121;
    default-lease-time 600; # 10 menit (Soal 6)
}

# === JARINGAN 3 (Khamul & Amandil) ===
subnet 192.239.3.0 netmask 255.255.255.0 {
    option routers 192.239.3.1;

    # (MAC Address Khamul dari log Anda)
    host Khamul {
        hardware ethernet 02:42:f2:39:a2:00;
        fixed-address 192.239.3.95;
    }
}

# === JARINGAN 4 (Server - Sisi Aldarion) ===
subnet 192.239.4.0 netmask 255.255.255.0 {
    option routers 192.239.4.1;
    # (Tidak ada pool dinamis di sini)
}

# === JARINGAN 5 (Minastir) ===
subnet 192.239.5.0 netmask 255.255.255.0 {
    option routers 192.239.5.1;
    # (Tidak ada pool dinamis di sini)
}
EOF

# 4. Memeriksa sintaks konfigurasi
echo "--> Memeriksa sintaks konfigurasi..."
dhcpd -t

# 5. Merestart layanan untuk menerapkan konfigurasi
echo "--> Merestart layanan isc-dhcp-server..."
service isc-dhcp-server restart

echo "=== [Aldarion] Konfigurasi DHCP Server Selesai ==="

# 6. Menampilkan status akhir
echo "--> Cek status akhir:"
service isc-dhcp-server status
```

### Script 4_1 (Erendis)

```
#!/bin/bash
# ==========================================================
# SKRIP OTOMATIS KONFIGURASI DNS MASTER (NODE: ERENDIS)
# (Versi Perbaikan Karakter Spasi)
# ==========================================================

# --- Variabel Konfigurasi ---
AMDIR_IP="192.239.3.4"
DOMAIN_NAME="k56.com"
ZONES_DIR="/etc/bind/zones"
ZONE_FILE_PATH="${ZONES_DIR}/db.${DOMAIN_NAME}"

echo "=== [Erendis] Memulai Konfigurasi DNS Master (BIND9) ==="

# 1. Menginstal paket
echo "--> Menginstal BIND9 dan dnsutils..."
apt-get update
apt-get install -y bind9 dnsutils

# 2. Link kompatibilitas service
echo "--> Membuat symlink (ln -s /etc/init.d/named /etc/init.d/bind9)..."
ln -s /etc/init.d/named /etc/init.d/bind9 || true

# 3. Konfigurasi /etc/bind/named.conf.options
echo "--> Mengkonfigurasi named.conf.options (allow-transfer untuk ${AMDIR_IP})..."
# (PERBAIKAN: Menggunakan spasi standar, bukan '   ')
sed -i "/^};/i \        allow-transfer { ${AMDIR_IP}; };" /etc/bind/named.conf.options

# 4. Konfigurasi /etc/bind/named.conf.local
echo "--> Mengkonfigurasi named.conf.local (zona master)..."
# (PERBAIKAN: Menggunakan spasi standar untuk indentasi)
cat << EOF >> /etc/bind/named.conf.local

# === Zona ${DOMAIN_NAME} (Dibuat oleh skrip) ===
zone "${DOMAIN_NAME}" {
    type master;
    file "${ZONE_FILE_PATH}";
    allow-update { none; };
};
EOF

# 5. Membuat direktori /etc/bind/zones
echo "--> Membuat direktori zona: ${ZONES_DIR}..."
mkdir -p ${ZONES_DIR}

# 6. Membuat file zona master (db.k56.com)
echo "--> Menulis file zona: ${ZONE_FILE_PATH}..."
# (PERBAIKAN: Menggunakan spasi/tab standar untuk indentasi)
cat << EOF > ${ZONE_FILE_PATH}
; === FILE ZONA UNTUK ${DOMAIN_NAME} ===
; (Dibuat oleh skrip)

\$TTL 604800
@   IN  SOA  ns1.${DOMAIN_NAME}. root.${DOMAIN_NAME}. (
            1         ; Serial
            604800    ; Refresh
            86400     ; Retry
            2419200   ; Expire
            604800 )  ; Negative Cache TTL

; --- Penjaga Peta (Nameserver) ---
@           IN  NS  ns1.${DOMAIN_NAME}.
@           IN  NS  ns2.${DOMAIN_NAME}.

; --- Alamat Penjaga Peta (Wajib) ---
ns1         IN  A   192.239.3.3  ; IP Erendis
ns2         IN  A   192.239.3.4  ; IP Amdir

; --- Lokasi Penting (Sesuai Soal) ---
palantir    IN  A   192.239.4.4
elros       IN  A   192.239.1.7
pharazon    IN  A   192.239.2.7
elendil     IN  A   192.239.1.2
isildur     IN  A   192.239.1.3
anarion     IN  A   192.239.1.4
galadriel   IN  A   192.239.2.2
celeborn    IN  A   192.239.2.3
oropher     IN  A   192.239.2.4
EOF

# 7. (PENTING) Memperbaiki izin untuk menghindari SERVFAIL
echo "--> Memperbaiki izin file (chown -R bind:bind ${ZONES_DIR})..."
chown -R bind:bind ${ZONES_DIR}

# 8. Memvalidasi semua file konfigurasi
echo "--> Memvalidasi konfigurasi..."
named-checkconf
named-checkzone ${DOMAIN_NAME} ${ZONE_FILE_PATH}

# 9. Merestart BIND9 untuk menerapkan
echo "--> Merestart layanan BIND9 (named)..."
service named restart

echo "=== [Erendis] Konfigurasi DNS Master Selesai ==="
echo "--> Cek status akhir (seharusnya 'running'):"
service named status


[!Perhatikan, jika ada error, failed saat running]
# (Di console Erendis)
# Hentikan layanan (meskipun sudah gagal)
service named stop || true

# Hapus total paket DAN semua file konfigurasinya
rm -rf /etc/bind/zones
apt-get purge -y bind9 bind9-dnsutils

```

### Script4_2

#### Amdir

```
#!/bin/bash
# ==========================================================
# SKRIP OTOMATIS KONFIGURASI DNS SLAVE (NODE: AMDIR)
# ==========================================================

# --- Variabel Konfigurasi ---
DOMAIN_NAME="k56.com"
MASTER_IP="192.239.3.3" # IP Erendis (Master)

echo "=== [Amdir] Memulai Konfigurasi DNS Slave (BIND9) ==="

# 1. Menginstal paket
echo "--> Menginstal BIND9 dan dnsutils..."
apt-get update
apt-get install -y bind9 dnsutils

# 2. (Sesuai permintaan Anda) Link kompatibilitas service
echo "--> Membuat symlink (ln -s /etc/init.d/named /etc/init.d/bind9)..."
# '|| true' mencegah skrip berhenti jika file sudah ada
ln -s /etc/init.d/named /etc/init.d/bind9 || true

# 3. Konfigurasi /etc/bind/named.conf.local
# Menambahkan (append) blok slave zone baru ke akhir file
echo "--> Mengkonfigurasi named.conf.local (slave zone)..."
cat << EOF >> /etc/bind/named.conf.local

# === Slave Zone ${DOMAIN_NAME} (Dibuat oleh skrip) ===
zone "${DOMAIN_NAME}" {
    type slave;
    file "db.${DOMAIN_NAME}";
    masters { ${MASTER_IP}; };
};
EOF

# 4. Memvalidasi semua file konfigurasi
echo "--> Memvalidasi konfigurasi..."
named-checkconf

# 5. Merestart BIND9 untuk menerapkan dan memulai transfer
echo "--> Merestart layanan BIND9..."
service bind9 restart

echo "=== [Amdir] Konfigurasi DNS Slave Selesai ==="
echo "--> Cek status akhir (seharusnya 'running'):"
service named status

echo "---"
echo "Verifikasi transfer zona berhasil (dig axfr ${DOMAIN_NAME} @localhost)"

```

### Script5_1

#### Erendis

```
#!/bin/bash
# ==========================================================
# SKRIP OTOMATIS KONFIGURASI DNS SOAL 5 (NODE: ERENDIS)
# (Alias, TXT, dan Reverse PTR)
# ==========================================================

# --- Variabel Konfigurasi ---
DOMAIN_NAME="k56.com"
ZONES_DIR="/etc/bind/zones"
ZONE_FILE_PATH="${ZONES_DIR}/db.${DOMAIN_NAME}"
REVERSE_ZONE_NAME="3.239.192.in-addr.arpa"
REVERSE_ZONE_FILE_PATH="${ZONES_DIR}/db.192.239.3"

echo "=== [Erendis] Memulai Konfigurasi Soal 5 ==="

# 1. Memperbarui File Zona Forward (db.k56.com)
echo "--> Memperbarui Serial Number di ${ZONE_FILE_PATH}..."
# Menggunakan 'sed' untuk mengubah Serial dari 1 menjadi 2
# (Hanya akan mengubah baris yang berisi "1         ; Serial")
sed -i 's/1\s*;\s*Serial/            2         ; Serial/' ${ZONE_FILE_PATH}

echo "--> Menambahkan record CNAME dan TXT ke ${ZONE_FILE_PATH}..."
# Menambahkan (append) record baru ke akhir file
cat << EOF >> ${ZONE_FILE_PATH}

; --- Alias & Pesan Rahasia (Soal 5) ---
@           IN  A   192.239.1.7
www         IN  CNAME ${DOMAIN_NAME}.
@           IN  TXT   "Cincin Sauron: Elros"
@           IN  TXT   "Aliansi Terakhir: Pharazon"
EOF

# 2. Menambahkan Zona Reverse Baru (named.conf.local)
echo "--> Menambahkan Zona Reverse ke /etc/bind/named.conf.local..."
# Menambahkan (append) blok zona reverse baru
cat << EOF >> /etc/bind/named.conf.local

# === Zona Reverse ${REVERSE_ZONE_NAME} (Soal 5) ===
zone "${REVERSE_ZONE_NAME}" {
    type master;
    file "${REVERSE_ZONE_FILE_PATH}";
    allow-update { none; };
};
EOF

# 3. Membuat File Zona Reverse (db.192.239.3)
echo "--> Membuat file zona reverse: ${REVERSE_ZONE_FILE_PATH}..."
cat << EOF > ${REVERSE_ZONE_FILE_PATH}
; === FILE ZONA REVERSE UNTUK 192.239.3.x ===
; (Dibuat oleh skrip)

\$TTL 604800
@   IN  SOA  ns1.${DOMAIN_NAME}. root.${DOMAIN_NAME}. (
            1         ; Serial (mulai dari 1)
            604800
            86400
            2419200
            604800 )

; --- Nameserver ---
@   IN  NS  ns1.${DOMAIN_NAME}.
@   IN  NS  ns2.${DOMAIN_NAME}.

; --- Catatan PTR (Pointer) ---
; (Hanya IP di Jaringan 192.239.3.x yang dicatat di sini)
3   IN  PTR ns1.${DOMAIN_NAME}.  ; 192.239.3.3
4   IN  PTR ns2.${DOMAIN_NAME}.  ; 192.239.3.4
EOF

# 4. Memperbaiki Izin (Permissions) untuk file baru
echo "--> Memperbaiki izin untuk file zona reverse baru..."
chown bind:bind ${REVERSE_ZONE_FILE_PATH}

# 5. Memvalidasi semua file konfigurasi
echo "--> Memvalidasi semua konfigurasi..."
named-checkconf
echo "--> Memvalidasi zona ${DOMAIN_NAME} (Forward)..."
named-checkzone ${DOMAIN_NAME} ${ZONE_FILE_PATH}
echo "--> Memvalidasi zona ${REVERSE_ZONE_NAME} (Reverse)..."
named-checkzone ${REVERSE_ZONE_NAME} ${REVERSE_ZONE_FILE_PATH}

# 6. Merestart BIND9 untuk menerapkan
echo "--> Merestart layanan BIND9 (named)..."
service named restart

echo "=== [Erendis] Konfigurasi Soal 5 Selesai ==="
echo "--> Cek status akhir (seharusnya 'running'):"
service named status
```

Final

```
#!/bin/bash
# ==========================================================
# SKRIP FINAL KONFIGURASI DNS MASTER (NODE: ERENDIS)
# (Gabungan Soal 4 & 5, Anti-Duplikat, Perbaikan Spasi)
# ==========================================================

# --- Variabel Konfigurasi ---
AMDIR_IP="192.239.3.4"
DOMAIN_NAME="k56.com"
ZONES_DIR="/etc/bind/zones"
ZONE_FILE_PATH="${ZONES_DIR}/db.${DOMAIN_NAME}"
REVERSE_ZONE_NAME="3.239.192.in-addr.arpa"
REVERSE_ZONE_FILE_PATH="${ZONES_DIR}/db.192.239.3"

echo "=== [Erendis] Memulai Konfigurasi DNS Master (BIND9) ==="

# 1. Menginstal paket
echo "--> Menginstal BIND9 dan dnsutils..."
apt-get update
apt-get install -y bind9 dnsutils

# 2. Konfigurasi /etc/bind/named.conf.options
echo "--> Menulis file /etc/bind/named.conf.options..."
# (Menggunakan 'cat >' untuk MENIMPA, bukan 'sed -i')
cat << EOF > /etc/bind/named.conf.options
options {
    directory "/var/cache/bind";
    allow-transfer { ${AMDIR_IP}; };
    dnssec-validation auto;
    listen-on-v6 { any; };
};
EOF

# 3. Konfigurasi /etc/bind/named.conf.local
echo "--> Menulis file /etc/bind/named.conf.local..."
# (Menggunakan 'cat >' untuk MENIMPA)
cat << EOF > /etc/bind/named.conf.local
// File ini dibuat otomatis oleh skrip
// (Menghapus 'include' default agar bersih)

# === Zona ${DOMAIN_NAME} (Dibuat oleh skrip) ===
zone "${DOMAIN_NAME}" {
    type master;
    file "${ZONE_FILE_PATH}";
    allow-update { none; };
};

# === Zona Reverse ${REVERSE_ZONE_NAME} (Soal 5) ===
zone "${REVERSE_ZONE_NAME}" {
    type master;
    file "${REVERSE_ZONE_FILE_PATH}";
    allow-update { none; };
};
EOF

# 4. Membuat direktori /etc/bind/zones
echo "--> Membuat direktori zona: ${ZONES_DIR}..."
mkdir -p ${ZONES_DIR}

# 5. Membuat file zona master (db.k56.com) - GABUNGAN
echo "--> Menulis file zona: ${ZONE_FILE_PATH}..."
cat << EOF > ${ZONE_FILE_PATH}
; === FILE ZONA UNTUK ${DOMAIN_NAME} ===
; (Dibuat oleh skrip - Gabungan Soal 4 & 5)

\$TTL 604800
@   IN  SOA  ns1.${DOMAIN_NAME}. root.${DOMAIN_NAME}. (
            2         ; Serial (langsung versi 2)
            604800    ; Refresh
            86400     ; Retry
            2419200   ; Expire
            604800 )  ; Negative Cache TTL

; --- Penjaga Peta (Nameserver) ---
@           IN  NS  ns1.${DOMAIN_NAME}.
@           IN  NS  ns2.${DOMAIN_NAME}.

; --- Alamat Penjaga Peta (Wajib) ---
ns1         IN  A   192.239.3.3  ; IP Erendis
ns2         IN  A   192.239.3.4  ; IP Amdir

; --- Lokasi Penting (Sesuai Soal) ---
palantir    IN  A   192.239.4.4
elros       IN  A   192.239.1.7
pharazon    IN  A   192.239.2.7
elendil     IN  A   192.239.1.2
isildur     IN  A   192.239.1.3
anarion     IN  A   192.239.1.4
galadriel   IN  A   192.239.2.2
celeborn    IN  A   192.239.2.3
oropher     IN  A   192.239.2.4

; --- Alias & Pesan Rahasia (Soal 5) ---
@           IN  A   192.239.1.7
www         IN  CNAME ${DOMAIN_NAME}.
@           IN  TXT   "Cincin Sauron: Elros"
@           IN  TXT   "Aliansi Terakhir: Pharazon"
EOF

# 6. Membuat File Zona Reverse (db.192.239.3)
echo "--> Membuat file zona reverse: ${REVERSE_ZONE_FILE_PATH}..."
cat << EOF > ${REVERSE_ZONE_FILE_PATH}
; === FILE ZONA REVERSE UNTUK 192.239.3.x ===
; (Dibuat oleh skrip)
\$TTL 604800
@   IN  SOA  ns1.${DOMAIN_NAME}. root.${DOMAIN_NAME}. (
            1         ; Serial (mulai dari 1)
            604800
            86400
            2419200
            604800 )

; --- Nameserver ---
@   IN  NS  ns1.${DOMAIN_NAME}.
@   IN  NS  ns2.${DOMAIN_NAME}.

; --- Catatan PTR (Pointer) ---
3   IN  PTR ns1.${DOMAIN_NAME}.  ; 192.239.3.3
4   IN  PTR ns2.${DOMAIN_NAME}.  ; 192.239.3.4
EOF

# 7. Memperbaiki izin untuk menghindari SERVFAIL
echo "--> Memperbaiki izin file (chown -R bind:bind ${ZONES_DIR})..."
chown -R bind:bind ${ZONES_DIR}

# 8. Memvalidasi semua file konfigurasi
echo "--> Memvalidasi konfigurasi..."
named-checkconf
named-checkzone ${DOMAIN_NAME} ${ZONE_FILE_PATH}
named-checkzone ${REVERSE_ZONE_NAME} ${REVERSE_ZONE_FILE_PATH}

# 9. Merestart BIND9 untuk menerapkan
echo "--> Merestart layanan BIND9 (named)..."
service named restart

echo "=== [Erendis] Konfigurasi DNS Master Selesai ==="
echo "--> Cek status akhir (seharusnya 'running'):"
service named status
```

### Script5_2

#### Amdir

```
#!/bin/bash
# ==========================================================
# SKRIP UNTUK MENAMBAHKAN REVERSE ZONE (NODE: AMDIR)
# ==========================================================

# --- Variabel Konfigurasi ---
REVERSE_ZONE_NAME="3.239.192.in-addr.arpa"
MASTER_IP="192.239.3.3" # IP Erendis (Master)

echo "=== [Amdir] Menambahkan Konfigurasi Reverse Zone Slave ==="

# 1. Konfigurasi /etc/bind/named.conf.local
# Menambahkan (append) blok slave zone baru ke akhir file
echo "--> Menambahkan (append) ${REVERSE_ZONE_NAME} ke named.conf.local..."
cat << EOF >> /etc/bind/named.conf.local

# === Slave Zone ${REVERSE_ZONE_NAME} (Dibuat oleh skrip) ===
zone "${REVERSE_ZONE_NAME}" {
    type slave;
    file "db.192.239.3";
    masters { ${MASTER_IP}; };
};
EOF

# 2. Memvalidasi semua file konfigurasi
echo "--> Memvalidasi konfigurasi..."
named-checkconf

# 3. Merestart BIND9 untuk menerapkan dan memulai transfer
echo "--> Merestart layanan BIND9 (named)..."
service named restart

echo "=== [Amdir] Konfigurasi Reverse Zone Selesai ==="
echo "--> Cek status akhir (seharusnya 'running'):"
service named status
```

### Script 6

Sama seperti script nomor 2 dan 5

### Script 7_1.sh

#### Elendil

```
#!/bin/bash
# ==========================================================
# SKRIP MASTER KONFIGURASI LARAVEL WORKER (Soal 7 & 8)
# NODE: ELENDIL
# ==========================================================

# --- [1] SESUAIKAN VARIABEL INI ---
NODE_NAME="elendil"
NODE_IP="192.239.1.2"
LISTEN_PORT="8001"
DOMAIN_NAME="k56.com"

# --- Variabel Database (Soal 8) ---
DB_HOST="192.239.4.4" # IP Palantir
DB_NAME="benteng_db"
DB_USER="ksatria"
DB_PASS="SandiRahasia123"

# --- Variabel Internal ---
PROJECT_DIR="/var/www/benteng-laravel"
PHP_VERSION="8.4"

echo "=== [${NODE_NAME}] Memulai Instalasi Tools (Soal 7) ==="

# 1. Instalasi Prasyarat
apt-get update
echo "--> Menginstal Nginx, ca-certificates, curl, git..."
apt-get install -y nginx ca-certificates curl git lsb-release apt-transport-https software-properties-common gnupg2

# 2. Instalasi PPA PHP
echo "--> Menambahkan PPA PHP (ppa:ondrej/php)..."
add-apt-repository ppa:ondrej/php -y
apt-get update

# 3. Instalasi PHP & Ekstensi (termasuk php-mysql untuk Soal 8)
echo "--> Menginstal PHP ${PHP_VERSION} dan ekstensinya..."
apt-get install -y \
    php${PHP_VERSION}-fpm \
    php${PHP_VERSION}-cli \
    php${PHP_VERSION}-mbstring \
    php${PHP_VERSION}-xml \
    php${PHP_VERSION}-bcmath \
    php${PHP_VERSION}-curl \
    php${PHP_VERSION}-zip \
    php${PHP_VERSION}-mysql \
    php${PHP_VERSION}-gd \
    php${PHP_VERSION}-intl \
    php${PHP_VERSION}-sqlite3

# 4. Instalasi Composer
echo "--> Menginstal Composer..."
curl -sS https://getcomposer.org/installer -o composer-setup.php
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"

echo "=== [${NODE_NAME}] Membuat Proyek Laravel (Soal 7) ==="

# 5. Membuat Proyek (PERBAIKAN: mkdir -p /var/www)
# Ini memperbaiki 'mv' error Anda sebelumnya
mkdir -p /var/www
cd /var/www

if [ ! -d "${PROJECT_DIR}" ]; then
    echo "--> Menjalankan composer create-project..."
    composer create-project --prefer-dist laravel/laravel benteng-laravel
else
    echo "--> Direktori ${PROJECT_
```

### Script 7_2.sh

#### Isildur

```
#!/bin/bash
# ==========================================================
# SKRIP OTOMATIS KONFIGURASI LARAVEL WORKER
# (Diadopsi untuk Isildur)
# ==========================================================

# --- [1] SESUAIKAN VARIABEL INI ---
NODE_NAME="isildur"
NODE_IP="192.239.1.3"
DOMAIN_NAME="k56.com"
# -----------------------------------

# --- Variabel Internal ---
PROJECT_DIR="/var/www/benteng-laravel"
PHP_VERSION="8.4"

echo "=== [${NODE_NAME}] Memulai Instalasi Tools ==="

# 1. Instalasi Prasyarat
apt-get update
echo "--> Menginstal Nginx, ca-certificates, curl, git..."
apt-get install -y nginx ca-certificates curl git lsb-release apt-transport-https software-properties-common gnupg2
apt install nginx -y
apt install lynx -y

# 2. Instalasi PPA PHP
echo "--> Menambahkan PPA PHP (ppa:ondrej/php)..."
add-apt-repository ppa:ondrej/php -y
apt-get update

# 3. Instalasi PHP & Ekstensi
echo "--> Menginstal PHP ${PHP_VERSION} dan ekstensinya..."
apt-get install -y \
    php${PHP_VERSION}-fpm \
    php${PHP_VERSION}-cli \
    php${PHP_VERSION}-mbstring \
    php${PHP_VERSION}-xml \
    php${PHP_VERSION}-bcmath \
    php${PHP_VERSION}-curl \
    php${PHP_VERSION}-zip \
    php${PHP_VERSION}-mysql \
    php${PHP_VERSION}-gd \
    php${PHP_VERSION}-intl \
    php${PHP_VERSION}-sqlite3 # Penting untuk migrasi

# 4. Instalasi Composer
echo "--> Menginstal Composer..."
curl -sS https://getcomposer.org/installer -o composer-setup.php
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"

echo "=== [${NODE_NAME}] Membuat Proyek Laravel ==="

# 5. Membuat Proyek (jika belum ada)
if [ ! -d "${PROJECT_DIR}" ]; then
    echo "--> Menjalankan composer create-project..."
    cd /var/www
    composer create-project --prefer-dist laravel/laravel benteng-laravel
else
    echo "--> Direktori ${PROJECT_DIR} sudah ada, lanjut..."
fi

cd ${PROJECT_DIR}

# 6. Menjalankan Migrasi (membuat tabel sessions, dll)
echo "--> Menjalankan artisan migrate (membuat tabel)..."
# (Pastikan file .env ada sebelum migrate)
if [ ! -f ".env" ]; then
    cp .env.example .env
    php artisan key:generate
fi
php artisan migrate

# 7. Mengatur Izin (Permissions)
echo "--> Mengatur izin untuk storage, bootstrap, dan database..."
chown -R www-data:www-data storage bootstrap/cache database/
chmod -R 775 storage bootstrap/cache
chmod -R u+w database/ # Memperbaiki 'readonly database' error

echo "=== [${NODE_NAME}] Mengkonfigurasi Nginx ==="

# 8. Membuat file konfigurasi Nginx
echo "--> Menulis file /etc/nginx/sites-available/${NODE_NAME}..."
cat << EOF > /etc/nginx/sites-available/${NODE_NAME}
server {
    listen 80;
    listen [::]:80;
    server_name ${NODE_NAME}.${DOMAIN_NAME} ${NODE_IP};
    root ${PROJECT_DIR}/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php${PHP_VERSION}-fpm.sock;
        fastcgi_param SCRIPT_FILENAME \$realpath_root\$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
EOF

# 9. Mengaktifkan Situs
echo "--> Mengaktifkan situs ${NODE_NAME}..."
rm /etc/nginx/sites-enabled/${NODE_NAME} || true
ln -s /etc/nginx/sites-available/${NODE_NAME} /etc/nginx/sites-enabled/${NODE_NAME}
rm /etc/nginx/sites-enabled/default || true

# 10. Tes & Restart
echo "--> Tes konfigurasi Nginx..."
nginx -t

echo "--> Merestart Nginx dan PHP-FPM..."
service nginx restart
service php${PHP_VERSION}-fpm restart

echo "=== [${NODE_NAME}] Setup Selesai ==="
echo "Tes dari node lain dengan:"
echo "lynx http://${NODE_NAME}.${DOMAIN_NAME}"
echo "atau"
echo "lynx http://${NODE_IP}"
```

### Script7_3.sh

#### Anarion

```
#!/bin/bash
# ==========================================================
# SKRIP OTOMATIS KONFIGURASI LARAVEL WORKER
# (Diadopsi untuk Anarion)
# ==========================================================

# --- [1] SESUAIKAN VARIABEL INI ---
NODE_NAME="anarion"
NODE_IP="192.239.1.4"
DOMAIN_NAME="k56.com"
# -----------------------------------

# --- Variabel Internal ---
PROJECT_DIR="/var/www/benteng-laravel"
PHP_VERSION="8.4"

echo "=== [${NODE_NAME}] Memulai Instalasi Tools ==="

# 1. Instalasi Prasyarat
apt-get update
echo "--> Menginstal Nginx, ca-certificates, curl, git..."
apt-get install -y nginx ca-certificates curl git lsb-release apt-transport-https software-properties-common gnupg2
apt install nginx -y
apt install lynx -y

# 2. Instalasi PPA PHP
echo "--> Menambahkan PPA PHP (ppa:ondrej/php)..."
add-apt-repository ppa:ondrej/php -y
apt-get update

# 3. Instalasi PHP & Ekstensi
echo "--> Menginstal PHP ${PHP_VERSION} dan ekstensinya..."
apt-get install -y \
    php${PHP_VERSION}-fpm \
    php${PHP_VERSION}-cli \
    php${PHP_VERSION}-mbstring \
    php${PHP_VERSION}-xml \
    php${PHP_VERSION}-bcmath \
    php${PHP_VERSION}-curl \
    php${PHP_VERSION}-zip \
    php${PHP_VERSION}-mysql \
    php${PHP_VERSION}-gd \
    php${PHP_VERSION}-intl \
    php${PHP_VERSION}-sqlite3 # Penting untuk migrasi

# 4. Instalasi Composer
echo "--> Menginstal Composer..."
curl -sS https://getcomposer.org/installer -o composer-setup.php
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"

echo "=== [${NODE_NAME}] Membuat Proyek Laravel ==="

# 5. Membuat Proyek (jika belum ada)
if [ ! -d "${PROJECT_DIR}" ]; then
    echo "--> Menjalankan composer create-project..."
    cd /var/www
    composer create-project --prefer-dist laravel/laravel benteng-laravel
else
    echo "--> Direktori ${PROJECT_DIR} sudah ada, lanjut..."
fi

cd ${PROJECT_DIR}

# 6. Menjalankan Migrasi (membuat tabel sessions, dll)
echo "--> Menjalankan artisan migrate (membuat tabel)..."
# (Pastikan file .env ada sebelum migrate)
if [ ! -f ".env" ]; then
    cp .env.example .env
    php artisan key:generate
fi
php artisan migrate

# 7. Mengatur Izin (Permissions)
echo "--> Mengatur izin untuk storage, bootstrap, dan database..."
chown -R www-data:www-data storage bootstrap/cache database/
chmod -R 775 storage bootstrap/cache
chmod -R u+w database/ # Memperbaiki 'readonly database' error

echo "=== [${NODE_NAME}] Mengkonfigurasi Nginx ==="

# 8. Membuat file konfigurasi Nginx
echo "--> Menulis file /etc/nginx/sites-available/${NODE_NAME}..."
cat << EOF > /etc/nginx/sites-available/${NODE_NAME}
server {
    listen 80;
    listen [::]:80;
    server_name ${NODE_NAME}.${DOMAIN_NAME} ${NODE_IP};
    root ${PROJECT_DIR}/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php${PHP_VERSION}-fpm.sock;
        fastcgi_param SCRIPT_FILENAME \$realpath_root\$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
EOF

# 9. Mengaktifkan Situs
echo "--> Mengaktifkan situs ${NODE_NAME}..."
rm /etc/nginx/sites-enabled/${NODE_NAME} || true
ln -s /etc/nginx/sites-available/${NODE_NAME} /etc/nginx/sites-enabled/${NODE_NAME}
rm /etc/nginx/sites-enabled/default || true

# 10. Tes & Restart
echo "--> Tes konfigurasi Nginx..."
nginx -t

echo "--> Merestart Nginx dan PHP-FPM..."
service nginx restart
service php${PHP_VERSION}-fpm restart

echo "=== [${NODE_NAME}] Setup Selesai ==="
echo "Tes dari node lain dengan:"
echo "lynx http://${NODE_NAME}.${DOMAIN_NAME}"
echo "atau"
echo "lynx http://${NODE_IP}"
```

#### Troubleshoot (Kalau ada error)

```
rm /etc/nginx/sites-enabled/benteng-laravel
ls -l /etc/nginx/sites-enabled/

Contoh :
isildur -> /etc/nginx/sites-available/isildur (return dari ls -l.... memunculkan ini)

nginx -t
service nginx restart
service php8.4-fpm restart
```

### Script 8_1.sh

#### Palantir

```
#!/bin/bash
# ==========================================================
# SKRIP OTOMATIS KONFIGURASI DATABASE (NODE: PALANTIR)
# (PERBAIKAN: Menambahkan Firewall port 3306)
# ==========================================================

# --- Variabel Database ---
DB_NAME="benteng_db"
DB_USER="ksatria"
DB_PASS="SandiRahasia123"
# -----------------------------------

echo "=== [Palantir] Memulai Instalasi MariaDB ==="

# 1. Instalasi MariaDB dan Iptables
apt-get update
apt-get install -y mariadb-server iptables-persistent

# 2. Konfigurasi MariaDB agar bisa diakses dari luar
echo "--> Mengkonfigurasi MariaDB (bind-address = 0.0.0.0)..."
sed -i "s/bind-address\s*=\s*127.0.0.1/bind-address = 0.0.0.0/" /etc/mysql/mariadb.conf.d/50-server.cnf

# 3. (PERBAIKAN) Membuka Port Firewall 3306
echo "--> Membuka port firewall 3306 (MariaDB)..."
# (Menggunakan -I INPUT 1 untuk memasukkan aturan di baris paling atas)
iptables -I INPUT 1 -p tcp --dport 3306 -j ACCEPT

# 4. Menyimpan aturan firewall
echo "--> Menyimpan aturan firewall..."
# (Ini mungkin akan menanyakan 'Yes/No' saat pertama kali)
netfilter-persistent save

# 5. Restart layanan database
echo "--> Merestart layanan MariaDB..."
service mariadb restart

# 6. Membuat database dan user
sleep 2 # Beri waktu 2 detik agar server siap
echo "--> Membuat database '${DB_NAME}' dan user '${DB_USER}'..."
mysql -e "CREATE DATABASE IF NOT EXISTS ${DB_NAME};"
mysql -e "CREATE USER IF NOT EXISTS '${DB_USER}'@'%' IDENTIFIED BY '${DB_PASS}';"
mysql -e "GRANT ALL PRIVILEGES ON ${DB_NAME}.* TO '${DB_USER}'@'%';"
mysql -e "FLUSH PRIVILEGES;"

echo "=== [Palantir] Setup Database Selesai ==="
echo "--> Cek status akhir (seharusnya 'running'):"
service mariadb status
```

### Script 8_2.sh

```
#!/bin/bash
# ==========================================================
# SKRIP UPDATE LARAVEL WORKER (NODE: ELENDIL)
# (PERBAIKAN: Perintah 'sed' yang lebih pintar untuk .env)
# ==========================================================

# --- Variabel Konfigurasi ---
NODE_NAME="elendil"
NODE_IP="192.239.1.2"
LISTEN_PORT="8001"
DOMAIN_NAME="k56.com"
PROJECT_DIR="/var/www/benteng-laravel"
PHP_VERSION="8.4"

# --- Variabel Database ---
DB_HOST="192.239.4.4" # IP Palantir
DB_NAME="benteng_db"
DB_USER="ksatria"
DB_PASS="SandiRahasia123"

echo "=== [${NODE_NAME}] Mengkonfigurasi .env ==="

# 1. Konfigurasi file .env (Menggunakan 'sed' yang aman)
cd ${PROJECT_DIR}

# (PERBAIKAN: 'sed' ini mencari baris yang mungkin dikomentari)
sed -i "s,^#\s*DB_CONNECTION=.*,DB_CONNECTION=mysql," .env
sed -i "s,^#\s*DB_HOST=.*,DB_HOST=${DB_HOST}," .env
sed -i "s,^#\s*DB_PORT=.*,DB_PORT=3306," .env
sed -i "s,^#\s*DB_DATABASE=.*,DB_DATABASE=${DB_NAME}," .env
sed -i "s,^#\s*DB_USERNAME=.*,DB_USERNAME=${DB_USER}," .env
sed -i "s,^#\s*DB_PASSWORD=.*,DB_PASSWORD=${DB_PASS}," .env

# (Jalankan lagi untuk baris yang tidak dikomentari, untuk keamanan)
sed -i "s,^DB_CONNECTION=.*,DB_CONNECTION=mysql," .env
sed -i "s,^DB_HOST=.*,DB_HOST=${DB_HOST}," .env
sed -i "s,^DB_PORT=.*,DB_PORT=3306," .env
sed -i "s,^DB_DATABASE=.*,DB_DATABASE=${DB_NAME}," .env
sed -i "s,^DB_USERNAME=.*,DB_USERNAME=${DB_USER}," .env
sed -i "s,^DB_PASSWORD=.*,DB_PASSWORD=${DB_PASS}," .env

echo "--> Verifikasi .env:"
cat .env | grep DB_HOST

echo "=== [${NODE_NAME}] Mengkonfigurasi Nginx (Port ${LISTEN_PORT}) ==="

# 2. Menulis ulang file konfigurasi Nginx (Spasi diperbaiki)
cat << EOF > /etc/nginx/sites-available/${NODE_NAME}
# BLOK 1: Tangkap akses via IP dan TOLAK
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_IP};
    return 404;
}

# BLOK 2: Izinkan akses via DOMAIN
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_NAME}.${DOMAIN_NAME};
    root ${PROJECT_DIR}/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    index index.php;
    charset utf-8;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php${PHP_VERSION}-fpm.sock;
        fastcgi_param SCRIPT_FILENAME \$realpath_root\$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
EOF

# 3. Tes & Restart
echo "--> Tes konfigurasi Nginx..."
nginx -t
echo "--> Merestart Nginx..."
service nginx restart

echo "=== [${NODE_NAME}] Update Selesai ==="
echo "PENTING: Jalankan 'php artisan migrate --seed' di node ini!"


Kemudian masuk ke var
cd /var/www/benteng-laravel && php artisan migrate --seed
```

### Script 8_3.sh

#### Isildur

```
#!/bin/bash
# ==========================================================
# SKRIP UPDATE LARAVEL WORKER (NODE: ISILDUR)
# (PERBAIKAN: Variabel, Konfig Nginx, & Cleanup Symlink)
# ==========================================================

# --- [1] Variabel Konfigurasi (SUDAH DIPERBAIKI) ---
NODE_NAME="isildur"
NODE_IP="192.239.1.3"
LISTEN_PORT="8002"
DOMAIN_NAME="k56.com"
PROJECT_DIR="/var/www/benteng-laravel"
PHP_VERSION="8.4"

# --- Variabel Database ---
DB_HOST="192.239.4.4" # IP Palantir
DB_NAME="benteng_db"
DB_USER="ksatria"
DB_PASS="SandiRahasia123"

echo "=== [${NODE_NAME}] Mengkonfigurasi .env ==="

# [2] Konfigurasi file .env (Menggunakan 'sed' yang aman)
# (Memastikan kita ada di direktori yang benar)
cd ${PROJECT_DIR}

sed -i "s,^#\s*DB_CONNECTION=.*,DB_CONNECTION=mysql," .env
sed -i "s,^#\s*DB_HOST=.*,DB_HOST=${DB_HOST}," .env
sed -i "s,^#\s*DB_PORT=.*,DB_PORT=3306," .env
sed -i "s,^#\s*DB_DATABASE=.*,DB_DATABASE=${DB_NAME}," .env
sed -i "s,^#\s*DB_USERNAME=.*,DB_USERNAME=${DB_USER}," .env
sed -i "s,^#\s*DB_PASSWORD=.*,DB_PASSWORD=${DB_PASS}," .env

sed -i "s,^DB_CONNECTION=.*,DB_CONNECTION=mysql," .env
sed -i "s,^DB_HOST=.*,DB_HOST=${DB_HOST}," .env
sed -i "s,^DB_PORT=.*,DB_PORT=3306," .env
sed -i "s,^DB_DATABASE=.*,DB_DATABASE=${DB_NAME}," .env
sed -i "s,^DB_USERNAME=.*,DB_USERNAME=${DB_USER}," .env
sed -i "s,^DB_PASSWORD=.*,DB_PASSWORD=${DB_PASS}," .env

echo "--> Verifikasi .env (seharusnya ${DB_HOST}):"
cat .env | grep DB_HOST

echo "=== [${NODE_NAME}] Mengkonfigurasi Nginx (Port ${LISTEN_PORT}) ==="

# [3] Menulis ulang file konfigurasi Nginx
cat << EOF > /etc/nginx/sites-available/${NODE_NAME}
# BLOK 1: Tangkap akses via IP dan TOLAK
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_IP};
    return 404;
}

# BLOK 2: Izinkan akses via DOMAIN
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_NAME}.${DOMAIN_NAME};
    root ${PROJECT_DIR}/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    index index.php;
    charset utf-8;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php${PHP_VERSION}-fpm.sock;
        fastcgi_param SCRIPT_FILENAME \$realpath_root\$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
EOF

# [4] (PERBAIKAN) Membersihkan dan Mengaktifkan Symlink
echo "--> Membersihkan symlink Nginx yang salah..."
rm /etc/nginx/sites-enabled/default || true
rm /etc/nginx/sites-enabled/anarion || true
rm /etc/nginx/sites-enabled/benteng-laravel || true
rm /etc/nginx/sites-enabled/elendil || true

echo "--> Mengaktifkan situs ${NODE_NAME}..."
# Hapus link lama (jika ada) dan buat yang baru
rm /etc/nginx/sites-enabled/${NODE_NAME} || true
ln -s /etc/nginx/sites-available/${NODE_NAME} /etc/nginx/sites-enabled/${NODE_NAME}

# [5] Tes & Restart
echo "--> Tes konfigurasi Nginx..."
nginx -t

echo "--> Merestart Nginx..."
service nginx restart

echo "=== [${NODE_NAME}] Update Selesai ==="
echo "PENTING: Jalankan 'php artisan migrate --seed' di node ini (jika belum)!"
echo "Tes dari node klien: lynx http://${NODE_NAME}.${DOMAIN_NAME}:${LISTEN_PORT}"
```

### Script 8_4.sh

#### Anarion

```
#!/bin/bash
# ==========================================================
# SKRIP UPDATE LARAVEL WORKER (NODE: ANARION)
# (PERBAIKAN: IP & EOF)
# ==========================================================

# --- [1] Variabel Konfigurasi (SUDAH DIPERBAIKI) ---
NODE_NAME="anarion"
NODE_IP="192.239.1.4"
LISTEN_PORT="8003"
DOMAIN_NAME="k56.com"
PROJECT_DIR="/var/www/benteng-laravel"
PHP_VERSION="8.4"

# --- Variabel Database ---
DB_HOST="192.239.4.4" # IP Palantir
DB_NAME="benteng_db"
DB_USER="ksatria"
DB_PASS="SandiRahasia123"

echo "=== [${NODE_NAME}] Mengkonfigurasi .env ==="

# [2] Konfigurasi file .env (Menggunakan 'sed' yang aman)
# (Memastikan kita ada di direktori yang benar)
cd ${PROJECT_DIR}

sed -i "s,^#\s*DB_CONNECTION=.*,DB_CONNECTION=mysql," .env
sed -i "s,^#\s*DB_HOST=.*,DB_HOST=${DB_HOST}," .env
sed -i "s,^#\s*DB_PORT=.*,DB_PORT=3306," .env
sed -i "s,^#\s*DB_DATABASE=.*,DB_DATABASE=${DB_NAME}," .env
sed -i "s,^#\s*DB_USERNAME=.*,DB_USERNAME=${DB_USER}," .env
sed -i "s,^#\s*DB_PASSWORD=.*,DB_PASSWORD=${DB_PASS}," .env

sed -i "s,^DB_CONNECTION=.*,DB_CONNECTION=mysql," .env
sed -i "s,^DB_HOST=.*,DB_HOST=${DB_HOST}," .env
sed -i "s,^DB_PORT=.*,DB_PORT=3306," .env
sed -i "s,^DB_DATABASE=.*,DB_DATABASE=${DB_NAME}," .env
sed -i "s,^DB_USERNAME=.*,DB_USERNAME=${DB_USER}," .env
sed -i "s,^DB_PASSWORD=.*,DB_PASSWORD=${DB_PASS}," .env

echo "--> Verifikasi .env (seharusnya ${DB_HOST}):"
cat .env | grep DB_HOST

echo "=== [${NODE_NAME}] Mengkonfigurasi Nginx (Port ${LISTEN_PORT}) ==="

# [3] Menulis ulang file konfigurasi Nginx
cat << EOF > /etc/nginx/sites-available/${NODE_NAME}
# BLOK 1: Tangkap akses via IP dan TOLAK
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_IP};
    return 404;
}

# BLOK 2: Izinkan akses via DOMAIN
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_NAME}.${DOMAIN_NAME};
    root ${PROJECT_DIR}/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    index index.php;
    charset utf-8;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php${PHP_VERSION}-fpm.sock;
        fastcgi_param SCRIPT_FILENAME \$realpath_root\$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
EOF

# [4] (PERBAIKAN) Membersihkan dan Mengaktifkan Symlink
echo "--> Membersihkan symlink Nginx yang salah..."
rm /etc/nginx/sites-enabled/default || true
rm /etc/nginx/sites-enabled/isildur || true
rm /etc/nginx/sites-enabled/benteng-laravel || true
rm /etc/nginx/sites-enabled/elendil || true

echo "--> Mengaktifkan situs ${NODE_NAME}..."
# Hapus link lama (jika ada) dan buat yang baru
rm /etc/nginx/sites-enabled/${NODE_NAME} || true
ln -s /etc/nginx/sites-available/${NODE_NAME} /etc/nginx/sites-enabled/${NODE_NAME}

# [5] Tes & Restart
echo "--> Tes konfigurasi Nginx..."
nginx -t

echo "--> Merestart Nginx..."
service nginx restart

echo "=== [${NODE_NAME}] Update Selesai ==="
echo "PENTING: Jalankan 'php artisan migrate --seed' di node ini (jika belum)!"
echo "Tes dari node klien: lynx http://${NODE_NAME}.${DOMAIN_NAME}:${LISTEN_PORT}"
```

### Script9.sh

```
#!/bin/bash
# ==========================================================
# SKRIP PEMBENAHAN FINAL LARAVEL (Soal 9)
# (Memindahkan rute ke web.php & Membersihkan Semua Cache)
# ==========================================================
PROJECT_DIR="/var/www/benteng-laravel"

cd ${PROJECT_DIR}
if [ $? -ne 0 ]; then
    echo "ERROR: Direktori ${PROJECT_DIR} tidak ditemukan."
    exit 1
fi

echo "--> Menambahkan route /api/airing ke routes/web.php..."

# 1. Menambahkan (append) rute ke web.php (Lokasi yang benar)
cat << 'EOF' >> ${PROJECT_DIR}/routes/web.php

// === TAMBAHAN UNTUK SOAL 9 ===
Route::get('/api/airing', function (Illuminate\Http\Request $request) {
    // Mengambil semua data 'users' untuk membuktikan koneksi DB
    return \App\Models\User::all();
});
EOF

echo "--> Membersihkan SEMUA cache Laravel..."
php artisan route:clear
php artisan config:clear
php artisan cache:clear
php artisan view:clear
php artisan event:clear

echo "--> Membangun ulang 'peta' autoloader Composer..."
# (Menggunakan path lengkap untuk menghindari 'command not found')
/usr/local/bin/composer dump-autoload

echo "--> Merestart PHP-FPM untuk membersihkan OpCache..."
service php8.4-fpm restart

echo "--> Semua cache telah dibersihkan dan layanan di-restart."
```

### Script10.sh

#### Elros

```
#!/bin/bash
# ==========================================================
# SKRIP 2: KONFIGURASI NGINX LOAD BALANCER (NODE: ELROS)
# (PERBAIKAN: Karakter Spasi)
# ==========================================================

# --- [1] Variabel Konfigurasi ---
NODE_NAME="elros"
DOMAIN_NAME="k56.com"

# --- IP Worker (Dari Soal 8 & DNS) ---
ELENDIL_IP="192.239.1.2:8001"
ISILDUR_IP="192.239.1.3:8002"
ANARION_IP="192.239.1.4:8003"

echo "=== [Elros] Mengkonfigurasi Nginx Load Balancer ==="

# 1. Membuat file konfigurasi Load Balancer
echo "--> Menulis file /etc/nginx/sites-available/elros-lb..."
cat << EOF > /etc/nginx/sites-available/elros-lb
# (Soal 10) Mendefinisikan grup worker "kesatria_numenor"
# (Algoritma default adalah Round Robin)
upstream kesatria_numenor {
    server ${ELENDIL_IP};
    server ${ISILDUR_IP};
    server ${ANARION_IP};
}

# (Soal 10) Server Reverse Proxy
server {
    listen 80;
    server_name ${NODE_NAME}.${DOMAIN_NAME};

    # Mengatur header agar worker tahu siapa klien aslinya
    proxy_set_header Host \$host;
    proxy_set_header X-Real-IP \$remote_addr;
    proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto \$scheme;

    location / {
        # Meneruskan semua lalu lintas ke grup 'upstream'
        proxy_pass http://kesatria_numenor;
    }
}
EOF

# 2. Mengaktifkan Situs Load Balancer
echo "--> Mengaktifkan situs elros-lb..."
rm /etc/nginx/sites-enabled/default || true
rm /etc/nginx/sites-enabled/elros-lb || true
ln -s /etc/nginx/sites-available/elros-lb /etc/nginx/sites-enabled/elros-lb

# 3. Tes & Restart Nginx
echo "--> Tes konfigurasi Nginx..."
nginx -t

echo "--> Merestart Nginx..."
service nginx restart

echo "=== [Elros] Setup Load Balancer Selesai ==="
echo "--> Cek status akhir (seharusnya 'running'):"
service nginx status
```

#### Elendil

```
#!/bin/bash
# ==========================================================
# SKRIP PEMBENAHAN FINAL (NODE: ELENDIL)
# (Menggabungkan Soal 8 & 9 + Perbaikan Soal 10)
# ==========================================================

# --- [1] Variabel Konfigurasi ---
NODE_NAME="elendil"
NODE_IP="192.239.1.2"
LISTEN_PORT="8001"
DOMAIN_NAME="k56.com"
PROJECT_DIR="/var/www/benteng-laravel"
PHP_VERSION="8.4"

# --- Variabel Database ---
DB_HOST="192.239.4.4" # IP Palantir
DB_NAME="benteng_db"
DB_USER="ksatria"
DB_PASS="SandiRahasia123"

echo "=== [${NODE_NAME}] Memulai Pembenahan (Soal 8 & 9) ==="

# [2] Pastikan proyek ada di /var/www/
echo "--> Memastikan proyek ada di ${PROJECT_DIR}..."
mkdir -p /var/www
if [ -d "/root/benteng-laravel" ]; then
    echo "--> Memindahkan proyek dari /root ke /var/www..."
    mv /root/benteng-laravel /var/www/
fi
cd ${PROJECT_DIR}

# [3] Konfigurasi file .env (Soal 8)
echo "--> Mengkonfigurasi .env untuk ${DB_HOST}..."
sed -i "s,^#\s*DB_CONNECTION=.*,DB_CONNECTION=mysql," .env
sed -i "s,^#\s*DB_HOST=.*,DB_HOST=${DB_HOST}," .env
sed -i "s,^#\s*DB_PORT=.*,DB_PORT=3306," .env
sed -i "s,^#\s*DB_DATABASE=.*,DB_DATABASE=${DB_NAME}," .env
sed -i "s,^#\s*DB_USERNAME=.*,DB_USERNAME=${DB_USER}," .env
sed -i "s,^#\s*DB_PASSWORD=.*,DB_PASSWORD=${DB_PASS}," .env
sed -i "s,^DB_CONNECTION=.*,DB_CONNECTION=mysql," .env
sed -i "s,^DB_HOST=.*,DB_HOST=${DB_HOST}," .env
sed -i "s,^DB_PORT=.*,DB_PORT=3306," .env
sed -i "s,^DB_DATABASE=.*,DB_DATABASE=${DB_NAME}," .env
sed -i "s,^DB_USERNAME=.*,DB_USERNAME=${DB_USER}," .env
sed -i "s,^DB_PASSWORD=.*,DB_PASSWORD=${DB_PASS}," .env
echo "--> Verifikasi .env:"
cat .env | grep DB_HOST

# [4] Menambahkan Rute API (Soal 9)
echo "--> Menambahkan route /api/airing ke routes/web.php..."
cat << 'EOF' >> ${PROJECT_DIR}/routes/web.php
// === TAMBAHAN UNTUK SOAL 9 ===
Route::get('/api/airing', function (Illuminate\Http\Request $request) {
    return \App\Models\User::all();
});
EOF

# [5] Konfigurasi Nginx (Soal 8 + Perbaikan Soal 10)
echo "--> Mengkonfigurasi Nginx (Port ${LISTEN_PORT})..."
cat << EOF > /etc/nginx/sites-available/${NODE_NAME}
# BLOK 1: Tangkap akses via IP dan TOLAK (Soal 8)
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_IP};
    return 404;
}

# BLOK 2: Izinkan akses via DOMAIN (Soal 8 & 10)
server {
    listen ${LISTEN_PORT};
    # (Menambahkan elros.k56.com agar Load Balancer bisa mengakses)
    server_name ${NODE_NAME}.${DOMAIN_NAME} elros.${DOMAIN_NAME};
    root ${PROJECT_DIR}/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    index index.php;
    charset utf-8;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }
    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php${PHP_VERSION}-fpm.sock;
        fastcgi_param SCRIPT_FILENAME \$realpath_root\$fastcgi_script_name;
        include fastcgi_params;
    }
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
EOF

# [6] Membersihkan dan Mengaktifkan Symlink Nginx
echo "--> Membersihkan symlink Nginx yang salah..."
rm /etc/nginx/sites-enabled/default || true
rm /etc/nginx/sites-enabled/benteng-laravel || true
rm /etc/nginx/sites-enabled/isildur || true
rm /etc/nginx/sites-enabled/anarion || true

echo "--> Mengaktifkan situs ${NODE_NAME}..."
rm /etc/nginx/sites-enabled/${NODE_NAME} || true
ln -s /etc/nginx/sites-available/${NODE_NAME} /etc/nginx/sites-enabled/${NODE_NAME}

# [7] Tes & Restart Nginx
echo "--> Tes konfigurasi Nginx..."
nginx -t
service nginx restart

# [8] Membersihkan Semua Cache Laravel
echo "--> Membersihkan SEMUA cache Laravel..."
php artisan route:clear
php artisan config:clear
php artisan cache:clear
php artisan view:clear
php artisan event:clear

echo "--> Membangun ulang autoloader Composer..."
/usr/local/bin/composer dump-autoload

echo "--> Merestart PHP-FPM (OpCache)..."
service php${PHP_VERSION}-fpm restart

echo "=== [${NODE_NAME}] Pembenahan Selesai ==="
echo "Tes dengan: curl http://${NODE_NAME}.${DOMAIN_NAME}:${LISTEN_PORT}/api/airing"
```

#### Validasi

```
root@Elendil:~# tail -f /var/log/nginx/access.log
192.239.1.2 - - [04/Nov/2025:04:14:13 +0000] "GET /api/airing HTTP/1.1" 404 6672 "-" "curl/8.14.1"
192.239.1.2 - - [04/Nov/2025:04:17:38 +0000] "GET /api/airing HTTP/1.1" 200 203 "-" "curl/8.14.1"
192.239.1.2 - - [04/Nov/2025:04:20:34 +0000] "GET /api/airing HTTP/1.1" 200 203 "-" "curl/8.14.1"
192.239.1.7 - - [04/Nov/2025:04:42:09 +0000] "GET / HTTP/1.0" 404 146 "-" "Lynx/2.9.2 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.8.5"
192.239.1.2 - - [04/Nov/2025:04:42:38 +0000] "GET / HTTP/1.0" 200 80650 "-" "Lynx/2.9.2 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.8.5"
192.239.4.4 - - [04/Nov/2025:04:42:45 +0000] "GET / HTTP/1.0" 200 80650 "-" "Lynx/2.9.2 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.8.5"
192.239.1.7 - - [04/Nov/2025:04:42:58 +0000] "GET / HTTP/1.0" 404 146 "-" "Lynx/2.9.2 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.8.5"
192.239.1.2 - - [04/Nov/2025:04:50:19 +0000] "GET /api/airing HTTP/1.1" 200 203 "-" "curl/8.14.1"
192.239.1.2 - - [04/Nov/2025:04:50:31 +0000] "GET / HTTP/1.0" 200 80650 "-" "Lynx/2.9.2 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.8.5"
192.239.1.7 - - [04/Nov/2025:04:53:48 +0000] "GET / HTTP/1.0" 200 80650 "-" "Lynx/2.9.2 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.8.5"
192.239.1.7 - - [04/Nov/2025:04:55:47 +0000] "GET / HTTP/1.0" 200 80650 "-" "Lynx/2.9.2 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.8.5"
192.239.1.7 - - [04/Nov/2025:04:56:00 +0000] "GET / HTTP/1.0" 200 80650 "-" "Lynx/2.9.2 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.8.5"
```

![[image-310.png]]

### Nomor 11

#### Erendis (atau node lainnya)

```
# (Di console Miriel)

# 1. Pastikan Miriel memiliki DNS (jika belum)
# (Edit /etc/network/interfaces, tambahkan dns-nameservers,
#  dan reboot node Miriel jika Anda belum melakukannya)

# 2. Instal apache2-utils (yang berisi 'ab')
apt-get update
apt-get install -y apache2-utils
```

#### Semua worker

```
# (Jalankan di Elendil, Isildur, DAN Anarion)
apt-get update
apt-get install -y htop
```

```
ab -n 100 -c 10 http://elros.k56.com/api/airing
```

```
#!/bin/bash
# ==========================================================
# SKRIP OPTIMASI APLIKASI (SOAL 11) - MENGGUNAKAN CACHING
# (Jalankan di Elendil, Isildur, dan Anarion)
# ==========================================================
PROJECT_DIR="/var/www/benteng-laravel"

cd ${PROJECT_DIR}
if [ $? -ne 0 ]; then
    echo "ERROR: Direktori ${PROJECT_DIR} tidak ditemukan."
    exit 1
fi

echo "--> (Soal 11) Mengoptimasi route /api/airing dengan Caching..."

# 1. MENIMPA (Overwrite) file web.php dengan versi Cache
# (Ini memperbaiki N+1 query bottleneck)
cat << 'EOF' > ${PROJECT_DIR}/routes/web.php
<?php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use Illuminate\Support\Facades\Cache; // (PENTING)
use App\Models\User; // (PENTING)

// === TAMBAHAN UNTUK SOAL 9 & 11 (VERSI CACHE) ===
Route::get('/api/airing', function (Request $request) {
    // Jalankan query 'User::all()' HANYA JIKA 'all_users'
    // tidak ada di cache. Simpan hasilnya selama 60 detik.
    return Cache::remember('all_users', 60, function () {
        return User::all();
    });
});
EOF

echo "--> Membersihkan SEMUA cache Laravel..."
php artisan route:clear
php artisan config:clear
php artisan cache:clear # (Menghapus 'all_users' yang lama)
php artisan view:clear
php artisan event:clear

echo "--> Membangun ulang 'peta' autoloader Composer..."
/usr/local/bin/composer dump-autoload

echo "--> Merestart PHP-FPM untuk membersihkan OpCache..."
service php8.4-fpm restart

echo "=== [${HOSTNAME}] Optimasi Caching Selesai ==="
```

### Nomor 12

#### Galadriel (Script 12_1.sh)

```
#!/bin/bash
# ==========================================================
# SKRIP OTOMATIS KONFIGURASI PHP WORKER (Soal 12)
# NODE: GALADRIEL
# ==========================================================

# --- [1] SESUAIKAN VARIABEL INI ---
NODE_NAME="galadriel"
NODE_IP="192.239.2.2"
DOMAIN_NAME="k56.com"
# -----------------------------------

# --- Variabel Internal ---
PHP_VERSION="8.4"
WEB_ROOT="/var/www/html"

echo "=== [${NODE_NAME}] Memulai Instalasi Nginx + PHP ==="

# 1. Instalasi Nginx & PHP-FPM
apt-get update
# (Kita tidak perlu PPA ondrej/php karena sudah ditambahkan oleh skrip Laravel)
apt-get install -y nginx php${PHP_VERSION}-fpm lynx

# 2. Membuat Direktori & File index.php
echo "--> Membuat file ${WEB_ROOT}/index.php..."
mkdir -p ${WEB_ROOT}
# (Perintah 'hostname' akan otomatis mengambil nama 'galadriel')
cat << EOF > ${WEB_ROOT}/index.php
<?php
echo "<h1>Ini adalah Taman Digital ${NODE_NAME}</h1>";
echo "Hostname: " . gethostname();
?>
EOF

# 3. Mengatur Izin (Permissions)
echo "--> Mengatur izin di ${WEB_ROOT}..."
chown -R www-data:www-data ${WEB_ROOT}

# 4. Konfigurasi Nginx (Blokir IP, Izinkan Domain)
echo "--> Mengkonfigurasi Nginx..."
cat << EOF > /etc/nginx/sites-available/${NODE_NAME}
# BLOK 1: Tangkap akses via IP dan TOLAK (Soal 12)
server {
    listen 80;
    server_name ${NODE_IP};
    return 404;
}

# BLOK 2: Izinkan akses via DOMAIN (Soal 12)
server {
    listen 80;
    server_name ${NODE_NAME}.${DOMAIN_NAME};
    root ${WEB_ROOT};
    index index.php;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php${PHP_VERSION}-fpm.sock;
    }
}
EOF

# 5. Membersihkan dan Mengaktifkan Symlink
echo "--> Membersihkan symlink Nginx yang salah..."
rm /etc/nginx/sites-enabled/default || true
rm /etc/nginx/sites-enabled/benteng-laravel || true
rm /etc/nginx/sites-enabled/elendil || true
# (Tambahkan node lain jika perlu)

echo "--> Mengaktifkan situs ${NODE_NAME}..."
rm /etc/nginx/sites-enabled/${NODE_NAME} || true
ln -s /etc/nginx/sites-available/${NODE_NAME} /etc/nginx/sites-enabled/${NODE_NAME}

# 6. Tes & Restart
echo "--> Tes konfigurasi Nginx..."
nginx -t

echo "--> Merestart Nginx dan PHP-FPM..."
service nginx restart
service php${PHP_VERSION}-fpm restart

echo "=== [${NODE_NAME}] Setup Selesai ==="
echo "Tes dari node klien (misal Miriel) dengan:"
echo "lynx http://${NODE_NAME}.${DOMAIN_NAME}"
echo "(Tes IP 'lynx http://${NODE_IP}' akan gagal 404)"
```

Validasi
![[image-312.png]]

#### Celeborn (Script12_2.sh)

```
#!/bin/bash
# ==========================================================
# SKRIP OTOMATIS KONFIGURASI PHP WORKER (Soal 12)
# NODE: CELEBORN
# ==========================================================

# --- [1] SESUAIKAN VARIABEL INI ---
NODE_NAME="celeborn"
NODE_IP="192.239.2.3"
DOMAIN_NAME="k56.com"
# -----------------------------------

# --- Variabel Internal ---
PHP_VERSION="8.4"
WEB_ROOT="/var/www/html"

echo "=== [${NODE_NAME}] Memulai Instalasi Nginx + PHP ==="

# 1. Instalasi Nginx & PHP-FPM
apt-get update
apt-get install -y nginx php${PHP_VERSION}-fpm lynx

# 2. Membuat Direktori & File index.php
echo "--> Membuat file ${WEB_ROOT}/index.php..."
mkdir -p ${WEB_ROOT}
cat << EOF > ${WEB_ROOT}/index.php
<?php
echo "<h1>Ini adalah Taman Digital ${NODE_NAME}</h1>";
echo "Hostname: " . gethostname();
?>
EOF

# 3. Mengatur Izin (Permissions)
echo "--> Mengatur izin di ${WEB_ROOT}..."
chown -R www-data:www-data ${WEB_ROOT}

# 4. Konfigurasi Nginx (Blokir IP, Izinkan Domain)
echo "--> Mengkonfigurasi Nginx..."
cat << EOF > /etc/nginx/sites-available/${NODE_NAME}
# BLOK 1: Tangkap akses via IP dan TOLAK (Soal 12)
server {
    listen 80;
    server_name ${NODE_IP};
    return 404;
}

# BLOK 2: Izinkan akses via DOMAIN (Soal 12)
server {
    listen 80;
    server_name ${NODE_NAME}.${DOMAIN_NAME};
    root ${WEB_ROOT};
    index index.php;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php${PHP_VERSION}-fpm.sock;
    }
}
EOF

# 5. Membersihkan dan Mengaktifkan Symlink
echo "--> Membersihkan symlink Nginx yang salah..."
rm /etc/nginx/sites-enabled/default || true
rm /etc/nginx/sites-enabled/benteng-laravel || true
rm /etc/nginx/sites-enabled/elendil || true
# (Tambahkan node lain jika perlu)

echo "--> Mengaktifkan situs ${NODE_NAME}..."
rm /etc/nginx/sites-enabled/${NODE_NAME} || true
ln -s /etc/nginx/sites-available/${NODE_NAME} /etc/nginx/sites-enabled/${NODE_NAME}

# 6. Tes & Restart
echo "--> Tes konfigurasi Nginx..."
nginx -t

echo "--> Merestart Nginx dan PHP-FPM..."
service nginx restart
service php${PHP_VERSION}-fpm restart

echo "=== [${NODE_NAME}] Setup Selesai ==="
echo "Tes dari node klien (misal Miriel) dengan:"
echo "lynx http://${NODE_NAME}.${DOMAIN_NAME}"
echo "(Tes IP 'lynx http://${NODE_IP}' akan gagal 404)"
```

Validasi (Contoh)
![[image-311.png]]

#### Oropher (script12_3.sh)

```
#!/bin/bash
# ==========================================================
# SKRIP OTOMATIS KONFIGURASI PHP WORKER (Soal 12)
# NODE: OROPHER
# ==========================================================

# --- [1] SESUAIKAN VARIABEL INI ---
NODE_NAME="oropher"
NODE_IP="192.239.2.4"
DOMAIN_NAME="k56.com"
# -----------------------------------

# --- Variabel Internal ---
PHP_VERSION="8.4"
WEB_ROOT="/var/www/html"

echo "=== [${NODE_NAME}] Memulai Instalasi Nginx + PHP ==="

# 1. Instalasi Nginx & PHP-FPM
apt-get update
apt-get install -y nginx php${PHP_VERSION}-fpm lynx

# 2. Membuat Direktori & File index.php
echo "--> Membuat file ${WEB_ROOT}/index.php..."
mkdir -p ${WEB_ROOT}
cat << EOF > ${WEB_ROOT}/index.php
<?php
echo "<h1>Ini adalah Taman Digital ${NODE_NAME}</h1>";
echo "Hostname: " . gethostname();
?>
EOF

# 3. Mengatur Izin (Permissions)
echo "--> Mengatur izin di ${WEB_ROOT}..."
chown -R www-data:www-data ${WEB_ROOT}

# 4. Konfigurasi Nginx (Blokir IP, Izinkan Domain)
echo "--> Mengkonfigurasi Nginx..."
cat << EOF > /etc/nginx/sites-available/${NODE_NAME}
# BLOK 1: Tangkap akses via IP dan TOLAK (Soal 12)
server {
    listen 80;
    server_name ${NODE_IP};
    return 404;
}

# BLOK 2: Izinkan akses via DOMAIN (Soal 12)
server {
    listen 80;
    server_name ${NODE_NAME}.${DOMAIN_NAME};
    root ${WEB_ROOT};
    index index.php;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php${PHP_VERSION}-fpm.sock;
    }
}
EOF

# 5. Membersihkan dan Mengaktifkan Symlink
echo "--> Membersihkan symlink Nginx yang salah..."
rm /etc/nginx/sites-enabled/default || true
rm /etc/nginx/sites-enabled/benteng-laravel || true
rm /etc/nginx/sites-enabled/elendil || true
# (Tambahkan node lain jika perlu)

echo "--> Mengaktifkan situs ${NODE_NAME}..."
rm /etc/nginx/sites-enabled/${NODE_NAME} || true
ln -s /etc/nginx/sites-available/${NODE_NAME} /etc/nginx/sites-enabled/${NODE_NAME}

# 6. Tes & Restart
echo "--> Tes konfigurasi Nginx..."
nginx -t

echo "--> Merestart Nginx dan PHP-FPM..."
service nginx restart
service php${PHP_VERSION}-fpm restart

echo "=== [${NODE_NAME}] Setup Selesai ==="
echo "Tes dari node klien (misal Miriel) dengan:"
echo "lynx http://${NODE_NAME}.${DOMAIN_NAME}"
echo "(Tes IP 'lynx http://${NODE_IP}' akan gagal 404)"
```

Validasi
![[image-314.png]]

### Nomor 13

#### Galadriel (Script13_1.sh)

```
#!/bin/bash
# ==========================================================
# SKRIP UPDATE NGINX PHP WORKER (Soal 13)
# NODE: GALADRIEL
# ==========================================================

# --- [1] Variabel Konfigurasi ---
NODE_NAME="galadriel"
NODE_IP="192.239.2.2"
LISTEN_PORT="8004" # <-- Port Baru
DOMAIN_NAME="k56.com"
PHP_VERSION="8.4"
WEB_ROOT="/var/www/html"

echo "=== [${NODE_NAME}] Mengkonfigurasi Nginx (Port ${LISTEN_PORT}) ==="

# 1. Konfigurasi Nginx (Blokir IP, Izinkan Domain, Port Baru)
cat << EOF > /etc/nginx/sites-available/${NODE_NAME}
# BLOK 1: Tangkap akses via IP dan TOLAK
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_IP};
    return 404;
}

# BLOK 2: Izinkan akses via DOMAIN
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_NAME}.${DOMAIN_NAME};
    root ${WEB_ROOT};
    index index.php;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php${PHP_VERSION}-fpm.sock;
    }
}
EOF

# 2. Tes & Restart Nginx
echo "--> Tes konfigurasi Nginx..."
nginx -t
echo "--> Merestart Nginx..."
service nginx restart

echo "=== [${NODE_NAME}] Update Selesai ==="
echo "Tes dengan: lynx http://${NODE_NAME}.${DOMAIN_NAME}:${LISTEN_PORT}"
```

Validasi

![[image-313.png]]
![[image-315.png]]

---

#### Celeborn

```
#!/bin/bash
# ==========================================================
# SKRIP UPDATE NGINX PHP WORKER (Soal 13)
# NODE: CELEBORN
# ==========================================================

# --- [1] Variabel Konfigurasi ---
NODE_NAME="celeborn"
NODE_IP="192.239.2.3"
LISTEN_PORT="8005" # <-- Port Baru
DOMAIN_NAME="k56.com"
PHP_VERSION="8.4"
WEB_ROOT="/var/www/html"

echo "=== [${NODE_NAME}] Mengkonfigurasi Nginx (Port ${LISTEN_PORT}) ==="

# 1. Konfigurasi Nginx (Blokir IP, Izinkan Domain, Port Baru)
cat << EOF > /etc/nginx/sites-available/${NODE_NAME}
# BLOK 1: Tangkap akses via IP dan TOLAK
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_IP};
    return 404;
}

# BLOK 2: Izinkan akses via DOMAIN
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_NAME}.${DOMAIN_NAME};
    root ${WEB_ROOT};
    index index.php;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php${PHP_VERSION}-fpm.sock;
    }
}
EOF

# 2. Tes & Restart Nginx
echo "--> Tes konfigurasi Nginx..."
nginx -t
echo "--> Merestart Nginx..."
service nginx restart

echo "=== [${NODE_NAME}] Update Selesai ==="
echo "Tes dengan: lynx http://${NODE_NAME}.${DOMAIN_NAME}:${LISTEN_PORT}"
```

![[image-316.png]]

#### Oropher

```
#!/bin/bash
# ==========================================================
# SKRIP UPDATE NGINX PHP WORKER (Soal 13)
# NODE: OROPHER
# ==========================================================

# --- [1] Variabel Konfigurasi ---
NODE_NAME="oropher"
NODE_IP="192.239.2.4"
LISTEN_PORT="8006" # <-- Port Baru
DOMAIN_NAME="k56.com"
PHP_VERSION="8.4"
WEB_ROOT="/var/www/html"

echo "=== [${NODE_NAME}] Mengkonfigurasi Nginx (Port ${LISTEN_PORT}) ==="

# 1. Konfigurasi Nginx (Blokir IP, Izinkan Domain, Port Baru)
cat << EOF > /etc/nginx/sites-available/${NODE_NAME}
# BLOK 1: Tangkap akses via IP dan TOLAK
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_IP};
    return 404;
}

# BLOK 2: Izinkan akses via DOMAIN
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_NAME}.${DOMAIN_NAME};
    root ${WEB_ROOT};
    index index.php;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php${PHP_VERSION}-fpm.sock;
    }
}
EOF

# 2. Tes & Restart Nginx
echo "--> Tes konfigurasi Nginx..."
nginx -t
echo "--> Merestart Nginx..."
service nginx restart

echo "=== [${NODE_NAME}] Update Selesai ==="
echo "Tes dengan: lynx http://${NODE_NAME}.${DOMAIN_NAME}:${LISTEN_PORT}"
```

Validasi
![[image-318.png]]
![[image-317.png]]

### Nomor 14

#### Galadriel

```
#!/bin/bash
# ==========================================================
# SKRIP UPDATE NGINX PHP WORKER (Soal 14 - Basic Auth)
# NODE: GALADRIEL
# ==========================================================

# --- [1] Variabel Konfigurasi ---
NODE_NAME="galadriel"
NODE_IP="192.239.2.2"
LISTEN_PORT="8004" # <-- Port dari Soal 13
DOMAIN_NAME="k56.com"
PHP_VERSION="8.4"
WEB_ROOT="/var/www/html"

# --- (BARU) Variabel Basic Auth ---
AUTH_USER="noldor"
AUTH_PASS="silvan"
AUTH_FILE_PATH="/etc/nginx/htpasswd"

echo "=== [${NODE_NAME}] Menambahkan Basic Auth (Port ${LISTEN_PORT}) ==="

# 1. Instalasi 'htpasswd' (bagian dari apache2-utils)
apt-get update
apt-get install -y apache2-utils

# 2. (BARU) Membuat file kata sandi
# '-c' = Create (buat file baru)
# '-b' = Batch mode (ambil password dari command line)
echo "--> Membuat file kata sandi di ${AUTH_FILE_PATH}..."
htpasswd -c -b ${AUTH_FILE_PATH} ${AUTH_USER} ${AUTH_PASS}

# 3. Konfigurasi Nginx (Timpa file dari Soal 13)
echo "--> Menulis ulang file /etc/nginx/sites-available/${NODE_NAME}..."
cat << EOF > /etc/nginx/sites-available/${NODE_NAME}
# BLOK 1: Tangkap akses via IP dan TOLAK
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_IP};
    return 404;
}

# BLOK 2: Izinkan akses via DOMAIN (DENGAN AUTH)
server {
    listen ${LISTEN_PORT};
    server_name ${NODE_NAME}.${DOMAIN_NAME};
    root ${WEB_ROOT};
    index index.php;

    # === (BARU) PERINTAH BASIC AUTH SOAL 14 ===
    auth_basic "Taman Terlarang Noldor";
    auth_basic_user_file ${AUTH_FILE_PATH};
    # ==========================================

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php${PHP_VERSION}-fpm.sock;
    }
}
EOF

# 4. Tes & Restart Nginx
echo "--> Tes konfigurasi Nginx..."
nginx -t
echo "--> Merestart Nginx..."
service nginx restart

echo "=== [${NODE_NAME}] Update Basic Auth Selesai ==="
echo "Tes dengan: lynx http://${AUTH_USER}:${AUTH_PASS}@${NODE_NAME}.${DOMAIN_NAME}:${LISTEN_PORT}"
```

---

# Out of topic (Catatan Belajar Tambahan)

Round robin membagi weight secara merata, salah satu worker melambat jatah eksekusi tetap sama akibatnya antrian pada worker yang lama menumpuk (No live upstream,error rate )

Resource : https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjGj4ShuMeQAxXN9zgGHbj7JN4QFnoECCQQAQ&url=https%3A%2F%2Fserverfault.com%2Fquestions%2F992837%2Fconnecting-to-another-server-through-squid&usg=AOvVaw0tPzW9w0sH_LoduptIE5yr&opi=89978449

Resource : https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjGj4ShuMeQAxXN9zgGHbj7JN4QFnoECCQQAQ&url=https%3A%2F%2Fserverfault.com%2Fquestions%2F992837%2Fconnecting-to-another-server-through-squid&usg=AOvVaw0tPzW9w0sH_LoduptIE5yr&opi=89978449
