# Jarkom-Modul-1-2026-K-58
# Jarkom-Modul-1-2026-K-58
## Modul 1 - K-58

Nama Kelompok: K-58  
Mata Praktikum: Jaringan Komputer  
Modul: 1  
Platform: GNS3 Web, Docker Linux, Wireshark

## Soal 1 - Membangun Topologi Jaringan pada GNS3

### Tujuan

Pada soal ini dilakukan pembuatan topologi jaringan menggunakan GNS3 Web dengan beberapa node yang terdiri dari router, switch, dan client. Topologi digunakan sebagai dasar untuk pengujian komunikasi antar jaringan pada soal berikutnya.

### Topologi

Node yang digunakan:

| Node | Keterangan |
|---|---|
| Lain | Router utama |
| Switch1 | Menghubungkan Alice dan Mika |
| Switch2 | Menghubungkan Chisa |
| Switch3 | Menghubungkan Knights dan Eiri |

Pembagian subnet:

| Network | Interface Lain | Client |
|---|---|---|
| 192.240.1.0/24 | eth1 | Alice, Mika |
| 192.240.2.0/24 | eth2 | Chisa |
| 192.240.3.0/24 | eth3 | Knights, Eiri |


### Konfigurasi IP
1). Bikin topologi dulu
#### Output Topologi
terus di run.
2). Router Lain, kita tulis di terminal lain:

```bash
ip link set eth1 up
ip link set eth2 up
ip link set eth3 up

ip addr add 192.240.2.1/24 dev eth1
ip addr add 192.240.3.1/24 dev eth2
ip addr add 192.240.1.1/24 dev eth3

ip -br a
```

### Client:
kemudian di masing2 client setting ini
1). Alice :
```bash
ip addr add 192.240.1.2/24 dev eth0
ip route add default via 192.240.1.1
```
2). Mika
```bash
ip addr add 192.240.1.3/24 dev eth0
ip route add default via 192.240.1.1
```
3). Chisa:
```bash
ip addr add 192.240.2.2/24 dev eth0
ip route add default via 192.240.2.1
```
4). Knights:
```bash
ip addr add 192.240.3.2/24 dev eth0
ip route add default via 192.240.3.1
```
5). Eiri:
```bash
ip addr add 192.240.3.3/24 dev eth0
ip route add default via 192.240.3.1
```
### Penjelasan
Perintah **ip addr add** digunakan untuk memberikan alamat IP pada interface jaringan.
Perintah **ip route add** default via digunakan untuk menentukan gateway yang digunakan agar node dapat berkomunikasi dengan jaringan lain.
sehingga output yang didapatkan adalah seluruh node berhasil mendapatkan alamat IP sesuai subnet masing2.

### Output

## Soal 2 - Konfigurasi Routing Antar Subnet
### Tujuan

Mengaktifkan kemampuan router Lain agar dapat meneruskan paket antar jaringan berbeda.

### Script Konfigurasi Router

**File:**
```bash
/root/router_config.sh
```

Isi script:
```bash
#!/bin/sh

echo 1 > /proc/sys/net/ipv4/ip_forward

sysctl -p


iptables -F
iptables -t nat -F


iptables -A FORWARD -i eth1 -j ACCEPT
iptables -A FORWARD -i eth2 -j ACCEPT
iptables -A FORWARD -i eth3 -j ACCEPT


iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
### Penjelasan
1). Mengaktifkan IP Forwarding
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```
Digunakan untuk mengaktifkan fungsi router agar dapat meneruskan paket.

2). Membaca konfigurasi kernel
```bash
sysctl -p
```
Digunakan untuk menerapkan konfigurasi sistem.

3). Menghapus aturan firewall lama
```bash
iptables -F
iptables -t nat -F
```
Digunakan agar konfigurasi firewall dimulai dari kondisi bersih.

4). Mengizinkan forwarding
```bash
iptables -A FORWARD -i eth1 -j ACCEPT
```
Memperbolehkan paket dari interface eth1 diteruskan.

#### Hal yang sama dilakukan untuk eth2 dan eth3.
NAT
```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
Digunakan agar jaringan internal dapat mengakses jaringan luar.

### Output Soal 2
Router berhasil melakukan routing antar subnet.

**Pengujian:**
```bash
ping 192.240.3.2
```
dari client subnet berbeda.
**Hasil**

## Soal 3 - Konfigurasi DNS dan Internet Access
**Tujuan**
Melakukan konfigurasi DNS agar client dapat melakukan resolusi domain.

### Konfigurasi DNS
**File:**
```bash
/etc/resolv.conf
```
Isi:
```bash
nameserver 8.8.8.8
```
#### Pengujian
Command:
```bash
ping google.com 
```
atau
```bash
nslookup google.com
```
#### Penjelasan
**nameserver 8.8.8.8** menggunakan Google Public DNS sebagai server DNS.

Ketika melakukan ping terhadap domain, sistem akan melakukan DNS lookup terlebih dahulu untuk mendapatkan alamat IP tujuan.

**Output**

DNS berhasil melakukan resolusi domain.
contoh : google.com | Address : 64.xxx.xxx.xxx
[output soal 3]

## Soal 4 - Membuat Script Monitoring Status Router
**Tujuan**
Membuat script untuk melihat status interface jaringan dan konfigurasi NAT.

#### **Script**
Di terminal Lain, jalankan satu-satu:
```bash
sysctl -w net.ipv4.ip_forward=1
```
lalu:
```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
kemudian:
```bash
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth2 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth3 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```
Setelah itu cek:
```bash
iptables -t nat -L -v -n
```
Kalau tidak ada error, buka console Alice dan tes:
```bash
ping -c 4 8.8.8.8
```
lalu
```bash
echo "nameserver 8.8.8.8" > /etc/resolv.conf
ping -c 4 google.com
```
#### Penjelasan
Penjelasan
1). Melihat interface
```bash
ip -br a
```
2). Menampilkan interface jaringan secara ringkas.
Melihat NAT
```bash
iptables -t nat -L
```
Menampilkan aturan NAT yang aktif.

3). Melihat forwarding
```bash
iptables -L FORWARD
```
#### Output
[Output Soal 4]

## Soal 5 - Membuat Konfigurasi Persisten Setelah Restart
**Tujuan**
Memastikan konfigurasi jaringan dan script tetap tersedia setelah node dilakukan restart.

**Backup Konfigurasi**
Konfigurasi disimpan dalam bentuk script pada directory:
```bash
/root
```
Contoh:
```bash
/root/cek_status.sh
/root/router_config.sh
```
### Script 
**1. Buat konfigurasi jaringan permanen di lain**
Ketik:
```bash
nano /etc/network/interfaces
```
isi menjadi :
```bash
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
    address 192.240.2.1
    netmask 255.255.255.0

auto eth2
iface eth2 inet static
    address 192.240.3.1
    netmask 255.255.255.0

auto eth3
iface eth3 inet static
    address 192.240.1.1
    netmask 255.255.255.0
```
simpan
**2. Buat IP Forwarding permanen**
```bash
nano /etc/sysctl.conf
net.ipv4.ip_forward=1
```
simpan lalu
```bash
sysctl -p
```
**3. Buat NAT otomatis setelah boot**
```bash
nano /etc/local.d/nat.start
```
isi :
```bash
#!/bin/sh
iptables -t nat -C POSTROUTING -o eth0 -j MASQUERADE 2>/dev/null || iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -C FORWARD -i eth1 -o eth0 -j ACCEPT 2>/dev/null || iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -C FORWARD -i eth2 -o eth0 -j ACCEPT 2>/dev/null || iptables -A FORWARD -i eth2 -o eth0 -j ACCEPT
iptables -C FORWARD -i eth3 -o eth0 -j ACCEPT 2>/dev/null || iptables -A FORWARD -i eth3 -o eth0 -j ACCEPT
iptables -C FORWARD -i eth0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT 2>/dev/null || iptables -A FORWARD -i eth0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```
kemudian :
```bash
chmod +x /etc/local.d/nat.start
rc-update add local default
```
**4. Buat script yang diminta soal, isi di lain**
```bash
nano /root/cek_status.sh
```
isi:
```bash
#!/bin/sh
echo "=== INTERFACE STATUS ==="
ip -br a
echo
echo "=== NAT TABLE ==="
iptables -t nat -L -v -n
```
lalu
```bash
chmod +x /root/cek_status.sh
/root/cek_status.sh
```
### Output
[output soal 5]


## Soal 14
