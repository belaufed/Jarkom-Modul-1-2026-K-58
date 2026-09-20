
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
![Output Soal 1](./bukit%20no%201-13/266Complete.png)
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


**Pengujian:**
```bash
ping 192.240.3.2
```
dari client subnet berbeda.
**Hasil**
### Output Soal 2
Router berhasil melakukan routing antar subnet.
![Output Soal 2](bukit%20no%201-13/DNSNo6.png)

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
![Output Soal 3](bukit%20no%201-13/EchoRep.png)

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
![Output Soal 4](bukit%20no%201-13/MikaDownload.png)

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
![Output Soal 5](bukit%20no%201-13/buktiNo5.png)

![Output Soal 5 Setelah Restart](bukit%20no%201-13/No5TetepAdaSetelahRestart.png)

## Soal 6 - DNS Resolution dan ICMP Analysis Menggunakan Wireshark

### Tujuan

Melakukan pengujian proses resolusi DNS dan komunikasi menggunakan protokol ICMP. 
Capture paket dilakukan menggunakan Wireshark untuk melihat proses request dan response yang terjadi pada jaringan.

---

### Konfigurasi

Pada node Mika dilakukan pengujian koneksi terhadap beberapa domain.

Cara pengerjaan:
1). Buka node mika dan switch 1 lalu start capture. buka console mika, dan buka file yang dari soal di wireshark
2). di console mika
```bash
#!/bin/bash

echo "==========================================="
echo "  Protocol 7 Traffic Generator v2026"
echo "  Node: Mika Iwakura"
echo "==========================================="
echo "[*] Generating DNS & ICMP traffic..."

# ICMP Traffic
ping -c 5 8.8.8.8 &
ping -c 5 1.1.1.1 &
ping -c 3 its.ac.id &

# DNS Queries
nslookup google.com 8.8.8.8 &
nslookup its.ac.id 8.8.8.8 &
nslookup github.com 1.1.1.1 &
dig @8.8.8.8 example.com A &
dig @1.1.1.1 cloudflare.com AAAA &

wait
echo "[*] Traffic generation complete."
echo "[*] Check Wireshark for captured packets."
```
lalu disimpan
```bash
chmod +x traffic_protocol7.sh
./traffic_protocol7.sh
```
Filter wireshark
```bash
dns || icmp
```
harus memperlihatkan
- filter dns || icmp
- paket ICMP
- paket DNS (jika ada)
- Source
- Destination

### Output
![Output Soal 6](bukit%20no%201-13/NMAP12.png)

## Soal 7 - Konfigurasi Hak Akses File Linux
### Tujuan

Mengatur permission file pada Linux sehingga user tertentu memiliki hak akses sesuai kebutuhan.

### Script Konfigurasi
**Pembuatan file:**
```bash
touch /var/wired/data/signal_alice.txt
```
**Mengubah permission:**
```bash
chmod 644 /var/wired/data/signal_alice.txt
```
**Melihat permission:**
```bash
ls -l /var/wired/data
```
### Pengujian User

User Alice mencoba melakukan perubahan file.

Alice:
```bash
echo "Signal from Alice - K58" > /root/signal_alice.txt
```
upload sbg alice :
```bash
curl -v -u alice:alice123 \
-T /root/signal_alice.txt \
ftp://192.240.2.2/signal_alice.txt
```
cek di chisa-deb :
```bash
ls -l /var/wired/data/signal_alice.txt
cat /var/wired/data/signal_alice.txt
```
Berhasil karena Alice memiliki permission write.

User Eiri mencoba mengakses file.
coba di ping
```bash
curl -v -u eiri:eiri123 ftp://192.240.2.2/
```
Hasil:
```bash
Permission denied
```
karena user tidak memiliki hak akses write

### Output
![Output Soal 7 Eri Ditolak](bukit%20no%201-13/No7EriDitolak.png)

![Output Soal 7 Hak Alice Write](bukit%20no%201-13/No7HakAliceWrite.png)

## Soal 8 - Konfigurasi FTP Server dan Transfer File
**Tujuan**

Melakukan konfigurasi FTP server menggunakan vsftpd serta melakukan transfer file antara client Mika dan server Knights.

Konfigurasi FTP Server
Install vsftpd

Command:
```bash
apt install vsftpd
Konfigurasi vsftpd
```
File:
```bash
/etc/vsftpd.conf
```
Isi konfigurasi:
```bash
listen=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES

local_umask=022

chroot_local_user=YES

allow_writeable_chroot=YES

pasv_enable=YES
pasv_min_port=30000
pasv_max_port=30010
```
### Restart Service

**Command:**
```bash
service vsftpd restart
```

### Upload File FTP

Dari node Mika:
```bash
curl -v \
-u alice:alice123 \
-T knights_status_report.txt \
ftp://192.240.2.2/
```
### script 
ketik di console knights
```bash
cat > /root/knights_status_report.txt <<'EOF'
====================================================
   KNIGHTS OF THE EASTERN CALCULUS - STATUS REPORT
   Protocol 7 Surveillance Network
   Classification: LEVEL 7 - EYES ONLY
====================================================

Date: [CLASSIFIED]
Agent: Knights Unit Alpha
Node: Switch 3 - Subnet 192.240.3.0/24

---

SUBJECT: Network Reconnaissance Report

The Wired has been successfully infiltrated through
Protocol 7 channels. Current observations:

1. Router "Lain" has been identified as the central
   gateway node connecting all three subnet segments.

2. Switch 1 (192.240.1.0/24) hosts Alice and Mika.
   Both nodes show standard traffic patterns.

3. Switch 2 (192.240.2.0/24) hosts Chisa alone.
   Isolated subnet - minimal cross-traffic observed.

4. Switch 3 (192.240.3.0/24) - our operational base.
   Knights and Eiri coexist on this segment.

RECOMMENDATION:
Continue monitoring FTP and Telnet sessions for
plaintext credential exposure. SSH tunnels remain
impenetrable without keylog access.

--- END OF REPORT ---
Knights of the Eastern Calculus
"Let's all love Lain."
EOF
```
Blacklist Eiri + buat Mika read-only:
```bash
Blacklist Eiri + buat Mika read-only:
```
nyalakan FTP
```bash
pkill vsftpd 2>/dev/null
/usr/sbin/vsftpd /etc/vsftpd.conf &
sleep 1
ps aux | grep '[v]sftpd'
```
di console knights :
```bash
pkill vsftpd 2>/dev/null
/usr/sbin/vsftpd /etc/vsftpd.conf &
sleep 1
ps aux | grep '[v]sftpd'
```
di console chisa :
```bash
curl -v --disable-epsv -u alice:alice123 \
-T /root/knights_status_report.txt \
ftp://192.240.2.2/knights_status_report.txt
```
### Wireshark Analysis
1). buka file **no8_knights_ftp.pcap** lalu filter 
```bash
ftp || ftp-data
```
Setelah itu cari paket yang info-nya mengandung:
- PASV
- 227 Entering Passive Mode
- STOR knights_status_report.txt
- 226 Transfer complete
filter
```bash
ftp.request.command == "PASV"
ftp.request.command == "PASV"
ftp.response.code == 226
```
(diketik satu2)

Packet yang terlihat:

- FTP Request PASV
- FTP Request STOR
- FTP Response 226 Transfer Complete
### Output
![Output Soal 8](bukit%20no%201-13/PASV.png)

## Soal 9 - Analisis Telnet Menggunakan Wireshark
**Tujuan**

Melakukan analisis keamanan protokol Telnet dengan melihat bahwa data komunikasi dapat terbaca dalam bentuk plaintext.

### Konfigurasi
unduh file nomor 9 di console mika
```bash
ping -c 2 192.240.2.2
```
lalu di **chisa-deb** buat file nya
```bash
cat > /var/wired/data/protocol7_manifesto.txt <<'EOF'
====================================================
   PROTOCOL 7 - THE MANIFESTO
   A Declaration of Digital Consciousness
   Serial Experiments Lain - Year 2026
====================================================

ARTICLE I: THE NATURE OF THE WIRED
----------------------------------
The Wired is not merely a network of interconnected
machines. It is the collective unconscious of
humanity, rendered in packets and protocols.

Every TCP handshake is a conversation.
Every DNS query is a question.
Every encrypted tunnel is a whispered secret.

ARTICLE II: THE SEVEN PRINCIPLES
--------------------------------
1. All nodes are equal in the eyes of the router.
2. No packet shall be dropped without cause.
3. Encryption is the right of every connection.
4. Plaintext protocols expose the vulnerable.
5. The firewall protects, but also imprisons.
6. NAT masquerade hides truth behind a single face.
7. The Wired remembers everything - packet loss
   is merely a temporary forgetting.

ARTICLE III: THE PROPHECY OF LAIN
---------------------------------
"If you're not remembered, then you never existed."

In the world of networking, persistence is survival.
A configuration that vanishes upon restart is a
thought that was never truly committed to memory.

Therefore: Save your iptables. Write your interfaces.
Let your routing tables endure beyond the power cycle.

ARTICLE IV: CONCERNING SECURITY
--------------------------------
Telnet is the glass house of protocols - transparent
to any observer with a packet sniffer.

SSH is the steel vault - its contents visible only
to those who possess the key.

Choose wisely which door you open to The Wired.

---
"No matter where you go, everyone's connected."
- Lain Iwakura
EOF
```
lalu
```bash
chown alice:alice /var/wired/data/protocol7_manifesto.txt
chmod 644 /var/wired/data/protocol7_manifesto.txt
ls -lh /var/wired/data/protocol7_manifesto.txt
```
nyalakan/run mika, lalu ketik ini di console nya:
```bash
ip -br a
ping -c 2 192.240.1.1
ping -c 2 192.240.2.2
command -v curl
```
Setelah itu, masih di Mika, buat file baru untuk membuktikan bahwa Mika tidak punya hak write:
```bash
echo "Mika mencoba upload" > /root/mika_upload_test.txt
```
Lalu coba upload:
```bash
curl -v -u mika:mika123 \
-T /root/mika_upload_test.txt \
ftp://192.240.2.2/mika_upload_test.txt
```
Karena akun Mika sudah kita konfigurasi read-only, targetnya harus ada respons seperti:
```bash
550 Permission denied
```
### Output
![Output Soal 9](bukit%20no%201-13/Paket77.png)
## Soal 10 - ICMP Capture Knights
### Tujuan
Melakukan capture komunikasi ICMP antara node Knights dengan node tujuan.

### Command
Pada Knights:
```bash
ping 192.240.2.2
```
lalu start capture, dan kembali ke knights dan jalankan command 
```bash
ping -c 77 -s 128 -i 0.3 192.240.2.2
```
lalu stop capture, dan download capture tsb
Buka **no10_knights_icmp.pcap** di Wireshark dan pakai filter:
```bash
icmap
```
### Hasil

Capture Wireshark menunjukkan:

|Paket|Fungsi|
|Echo Request|	Paket permintaan dari pengirim|
|Echo Reply|	Balasan dari penerima|
### Output
![Output Soal 10](bukit%20no%201-13/STOR.png)

## Soal 11 - Analisis Telnet Capture Alice
**Tujuan**

Melakukan capture sesi Telnet dari node Alice dan melihat informasi paket yang dikirimkan.

### pengerjaan
Buka console chisa-deb dulu dan jalankan cuma ini:
```bash
command -v telnetd
command -v in.telnetd
```
harus mengeluarkan **/usr/sbin/telnetd**

Di chisa deb bikin akun yg diminta soal 
```bash
id phantom_user >/dev/null 2>&1 || useradd -m -s /bin/bash phantom_user
echo 'phantom_user:wired_ghost' | chpasswd
```
lalu cek
```bash
grep -v '^#' /etc/inetd.conf | grep telnet
```
Setelah itu nyalakan inetd:
```bash
pkill inetutils-inetd 2>/dev/null
/usr/sbin/inetutils-inetd
```
Lalu cek port 23:
```bash
ss -ltnp | grep ':23'
```
setelah itu capture di link eiri dan switch3, lalu di console eiri login menggunakan unsername: **phantom_user** dan password: **wired_ghost**:
```bash
telnet 192.240.2.2
```
lalu ketik **whoami** dan **exit**
setelah itu dimatikan capturenya dan di download, lalu buka **wireshark**
filter menggunakan:
```bash
telnet
```
Klik salah satu paket TELNET dulu (misalnya baris No. 34 yang biru sekarang sudah oke).
Klik menu atas:
```bash
Analyze
  ↓
Follow
  ↓
TCP Stream
```
Nanti muncul isi percakapan Telnet. Cari **Follow** dan pilih **TCP Stream**

### Output
![Output Soal 11](bukit%20no%201-13/No11.png)

Kalau ada LISTEN di port 23, berarti Telnet server Chisa sudah hidup ✅.
## Soal 14
