# Dokumentasi Pemeriksaan Koneksi MikroTik untuk RT/RW Net

Dokumentasi ini berisi berbagai metode untuk memeriksa koneksi internet pelanggan dari MikroTik RouterOS, khususnya untuk jaringan RT/RW Net.

Fokus pemeriksaan:

* Koneksi internet umum
* Pelanggan PPPoE
* DNS
* YouTube
* TikTok
* WhatsApp
* Facebook/Instagram
* Game online
* IP server tertentu
* TCP/UDP
* Latency
* Packet loss
* Jitter
* Routing/traceroute
* Connection tracking
* Torch
* Pemeriksaan saat pelanggan mengeluh lag atau internet lambat

> **Catatan:** Perintah dalam dokumentasi ini ditujukan untuk pemeriksaan/diagnosis. Sebagian besar perintah hanya membaca kondisi router dan tidak mengubah konfigurasi.

---

# 1. Prinsip Dasar Troubleshooting

Jangan langsung menyimpulkan bahwa internet bermasalah hanya karena satu aplikasi atau satu server tidak dapat diakses.

Urutan pemeriksaan yang disarankan:

```text
Pelanggan
   ↓
PPPoE
   ↓
MikroTik
   ↓
Gateway/Upstream
   ↓
DNS
   ↓
Routing
   ↓
Server tujuan
   ↓
Aplikasi
```

Contoh:

```text
YouTube tidak bisa dibuka
        ↓
Apakah PPPoE pelanggan aktif?
        ↓
Apakah DNS bekerja?
        ↓
Apakah IP YouTube bisa di-resolve?
        ↓
Apakah IP YouTube reachable?
        ↓
Apakah routing normal?
        ↓
Apakah TCP/443 bisa terhubung?
        ↓
Apakah masalah hanya terjadi pada pelanggan tertentu?
```

---

# 2. Memeriksa Resource MikroTik

Sebelum mendiagnosis internet, periksa kondisi router.

```mikrotik
/system resource print
```

Perhatikan:

```text
cpu-load
free-memory
uptime
version
```

Untuk melihat penggunaan CPU secara real-time:

```mikrotik
/tool profile
```

Jika CPU sangat tinggi, masalah internet bisa berasal dari router sendiri.

---

# 3. Memeriksa Interface

Melihat semua interface:

```mikrotik
/interface print
```

Melihat statistik interface:

```mikrotik
/interface print stats
```

Melihat trafik secara real-time:

```mikrotik
/interface monitor-traffic ether1
```

Ganti `ether1` dengan interface yang ingin diperiksa.

Contoh:

```mikrotik
/interface monitor-traffic sfp-sfpplus1
```

---

# 4. Memeriksa Pelanggan PPPoE

Melihat semua pelanggan PPPoE yang sedang aktif:

```mikrotik
/ppp active print
```

Mencari pelanggan berdasarkan IP:

```mikrotik
/ppp active print where address=192.168.8.13
```

Contoh hasil:

```text
NAME   SERVICE   CALLER-ID          ADDRESS
eli    pppoe     DC:71:37:5D:7C:10  192.168.8.13
```

Dari hasil tersebut diketahui:

```text
Username : eli
IP       : 192.168.8.13
MAC      : DC:71:37:5D:7C:10
```

---

# 5. Menemukan Interface PPPoE Pelanggan

Interface PPPoE biasanya berupa interface dinamis.

Cari berdasarkan username:

```mikrotik
/interface print where name~"eli"
```

Contoh:

```text
<pppoe-eli>
```

Interface tersebut dapat digunakan untuk Torch:

```mikrotik
/tool torch <pppoe-eli>
```

> Torch bekerja berdasarkan interface, bukan langsung berdasarkan IP pelanggan.

Perintah berikut salah:

```mikrotik
/tool torch 192.168.8.13
```

Karena `192.168.8.13` adalah IP pelanggan, bukan nama interface.

---

# 6. Ping Gateway MikroTik

Untuk memeriksa koneksi dasar:

```mikrotik
/ping 10.10.110.1 count=10
```

Ganti dengan gateway sesuai topologi.

Yang diperhatikan:

```text
packet-loss
avg-rtt
min-rtt
max-rtt
```

Contoh bagus:

```text
sent=10 received=10 packet-loss=0%
avg-rtt=1ms
```

---

# 7. Ping Internet

Gunakan beberapa tujuan, jangan hanya satu.

Contoh:

```mikrotik
/ping 1.1.1.1 count=10
```

```mikrotik
/ping 8.8.8.8 count=10
```

Jika keduanya normal, koneksi internet dasar kemungkinan tersedia.

---

# 8. Menguji DNS

Resolve sebuah domain:

```mikrotik
/resolve youtube.com
```

Contoh:

```text
142.251.xxx.xxx
```

Resolve domain lain:

```mikrotik
/resolve google.com
```

```mikrotik
/resolve tiktok.com
```

```mikrotik
/resolve facebook.com
```

```mikrotik
/resolve instagram.com
```

Jika domain tidak dapat di-resolve, periksa konfigurasi DNS MikroTik:

```mikrotik
/ip dns print
```

---

# 9. Memeriksa DNS Cache

Melihat cache:

```mikrotik
/ip dns cache print
```

Mencari YouTube:

```mikrotik
/ip dns cache print where name~"youtube"
```

Mencari TikTok:

```mikrotik
/ip dns cache print where name~"tiktok"
```

Mencari Free Fire:

```mikrotik
/ip dns cache print where name~"freefire"
```

Mencari Google:

```mikrotik
/ip dns cache print where name~"google"
```

Mencari Facebook:

```mikrotik
/ip dns cache print where name~"facebook"
```

---

# 10. Memahami DNS Cache

Contoh:

```text
youtubei.googleapis.com
www.youtube.com
rr1---sn-xxxx.c.youtube.com
```

Jika muncul pada DNS cache, berarti MikroTik telah melakukan atau melayani resolusi DNS untuk nama tersebut.

Namun:

> Muncul di DNS cache tidak otomatis berarti pelanggan sedang aktif menggunakan aplikasi tersebut saat ini.

Cache dapat berasal dari:

* aplikasi
* browser
* update aplikasi
* preload
* login
* background service
* cache DNS sebelumnya

Karena itu DNS cache harus dipakai sebagai **indikator**, bukan bukti tunggal.

---

# 11. Memeriksa IP YouTube

Setelah memperoleh IP:

```mikrotik
/ping 142.251.157.4 count=10
```

Contoh:

```text
sent=10
received=10
packet-loss=0%
avg-rtt=20ms
```

Ini menunjukkan IP tersebut dapat dijangkau dari MikroTik.

Namun:

> Ping sukses tidak otomatis berarti seluruh layanan YouTube normal.

YouTube menggunakan banyak server, CDN, domain, dan IP.

---

# 12. Memeriksa Banyak IP YouTube

Contoh:

```mikrotik
/ping 142.251.157.4 count=5
/ping 142.251.155.4 count=5
/ping 142.251.151.4 count=5
```

Bandingkan:

```text
packet-loss
min-rtt
avg-rtt
max-rtt
```

Jika semuanya konsisten, jalur ke IP yang diuji terlihat baik.

---

# 13. Traceroute

Untuk melihat jalur menuju tujuan:

```mikrotik
/tool traceroute 8.8.8.8
```

Untuk YouTube:

```mikrotik
/tool traceroute 142.251.157.4
```

Untuk server game:

```mikrotik
/tool traceroute x.x.x.x
```

Traceroute berguna untuk mengetahui:

* gateway
* router upstream
* hop yang mengalami delay
* jalur internet
* lokasi kemungkinan gangguan routing

> Hop yang tidak menjawab traceroute tidak otomatis berarti jalurnya rusak. Banyak router sengaja tidak merespons ICMP/TTL-expired.

---

# 14. Memeriksa TCP Port 443

Sebagian besar layanan internet modern menggunakan HTTPS.

Contoh:

```mikrotik
/tool telnet 142.251.157.4 443
```

Jika berhasil:

```text
Connected
```

maka TCP menuju port 443 berhasil.

Jika gagal:

```text
timeout
```

maka perlu diperiksa:

* routing
* firewall
* upstream
* server tujuan
* filtering

---

# 15. TCP 80

Untuk HTTP:

```mikrotik
/tool telnet 1.1.1.1 80
```

Port 80 tidak digunakan sebagai indikator utama layanan modern karena mayoritas website sudah menggunakan HTTPS.

---

# 16. UDP dan Game Online

Game online banyak menggunakan UDP.

Melihat connection tracking UDP:

```mikrotik
/ip firewall connection print where protocol=udp
```

Melihat UDP port 443:

```mikrotik
/ip firewall connection print where protocol=udp and dst-port=443
```

UDP 443 sering digunakan oleh QUIC/HTTP3, tetapi:

> UDP 443 bukan berarti pasti YouTube atau aplikasi tertentu.

Banyak aplikasi modern menggunakan UDP 443.

---

# 17. Connection Tracking

Melihat semua koneksi:

```mikrotik
/ip firewall connection print
```

Melihat koneksi pelanggan tertentu:

```mikrotik
/ip firewall connection print where src-address~"192.168.8.13"
```

Melihat koneksi menuju pelanggan:

```mikrotik
/ip firewall connection print where dst-address~"192.168.8.13"
```

Melihat kedua arah:

```mikrotik
/ip firewall connection print where src-address~"192.168.8.13" or dst-address~"192.168.8.13"
```

---

# 18. Memeriksa Koneksi HTTPS Pelanggan

Contoh pelanggan:

```text
192.168.8.13
```

Perintah:

```mikrotik
/ip firewall connection print where src-address~"192.168.8.13" and dst-port=443
```

Ini dapat memperlihatkan:

* IP tujuan
* port
* protocol
* state
* timeout

Namun karena HTTPS terenkripsi, connection tracking tidak selalu dapat memberitahu aplikasi secara langsung.

---

# 19. Torch

Torch digunakan untuk melihat trafik real-time pada interface.

Dasar:

```mikrotik
/tool torch <interface>
```

Contoh:

```mikrotik
/tool torch <pppoe-eli>
```

Torch dapat menampilkan trafik seperti:

```text
TX
RX
TX-PACKETS
RX-PACKETS
```

Pada tampilan detail dapat terlihat informasi seperti:

```text
SRC-ADDRESS
DST-ADDRESS
PROTOCOL
SRC-PORT
DST-PORT
TX
RX
```

Torch sangat berguna untuk melihat trafik real-time.

---

# 20. Torch Tidak Sama dengan Application Detector

Torch menunjukkan karakteristik trafik:

```text
IP
port
protocol
rate
packet
```

Tetapi bukan berarti MikroTik selalu dapat mengatakan:

```text
YouTube
TikTok
Free Fire
Mobile Legends
```

secara pasti.

HTTPS dan QUIC membuat identifikasi aplikasi menjadi lebih sulit.

---

# 21. Pemeriksaan Free Fire

Jika DNS cache menunjukkan:

```text
freefiremobile.com
```

cari record A:

```mikrotik
/ip dns cache print where name~"freefire" and type=A
```

Contoh:

```text
csoversea.stronghold.freefiremobile.com
35.198.211.155
34.87.24.78
34.126.122.233
```

Tes IP:

```mikrotik
/ping 35.198.211.155 count=10
```

Kemudian:

```mikrotik
/tool traceroute 35.198.211.155
```

---

# 22. Contoh Hasil Ping Game yang Baik

Misalnya:

```text
sent=5
received=5
packet-loss=0%
avg-rtt=18ms
```

Secara koneksi dasar:

```text
Latency     : rendah
Packet loss : 0%
Stabilitas  : baik
```

Contoh hasil:

```text
18.1 ms
18.2 ms
18.3 ms
18.5 ms
18.6 ms
```

menunjukkan variasi latency yang kecil.

---

# 23. Memahami Jitter

Jitter adalah perubahan latency antar paket.

Contoh bagus:

```text
18 ms
18 ms
19 ms
18 ms
19 ms
```

Contoh tidak stabil:

```text
18 ms
65 ms
22 ms
110 ms
19 ms
87 ms
```

Walaupun rata-rata terlihat masih rendah, variasi besar dapat terasa sebagai:

* lag
* delay
* karakter teleport
* tembakan terlambat
* reconnect
* suara patah-patah

---

# 24. Packet Loss

Packet loss sangat penting untuk game.

Contoh:

```text
sent=100
received=100
packet-loss=0%
```

Sangat baik dari sisi ICMP yang diuji.

Contoh:

```text
sent=100
received=98
packet-loss=2%
```

Ada kehilangan paket.

Contoh:

```text
sent=100
received=90
packet-loss=10%
```

Sudah perlu diperiksa lebih lanjut.

Namun batas toleransi tidak boleh disimpulkan hanya dari satu tes ping karena ICMP dan trafik game sebenarnya bisa diperlakukan berbeda.

---

# 25. Jangan Hanya Mengandalkan Ping

Contoh:

```text
Ping server:
20 ms
0% loss
```

tetapi pelanggan tetap lag.

Kemungkinan penyebab:

```text
HP
 ↓
Wi-Fi
 ↓
ONT
 ↓
PPPoE
 ↓
Queue
 ↓
MikroTik
 ↓
Internet
```

Masalah dapat terjadi pada:

* Wi-Fi
* interferensi
* upload pelanggan penuh
* download pelanggan penuh
* bufferbloat
* queue
* packet loss lokal
* CPU ONT
* perangkat pelanggan
* jalur UDP game

---

# 26. Pemeriksaan Saat Pelanggan Mengeluh Lag

Gunakan urutan:

```text
1. Periksa PPPoE
2. Ping IP pelanggan
3. Ping gateway
4. Ping internet
5. Ping server tujuan
6. Periksa packet loss
7. Periksa jitter
8. Traceroute
9. Periksa connection tracking
10. Periksa Torch
11. Periksa queue
12. Bandingkan pelanggan lain
```

---

# 27. Membandingkan Satu Pelanggan dengan Pelanggan Lain

Misalnya:

```text
Pelanggan A → lag
Pelanggan B → normal
Pelanggan C → normal
```

Tetapi:

```text
MikroTik → server game
0% loss
20 ms
```

Kemungkinan masalah tidak berada pada jalur MikroTik → server secara umum.

Periksa pelanggan A:

```text
Wi-Fi
ONT
PPPoE
queue
perangkat
traffic lokal
```

---

# 28. Jika Semua Pelanggan Mengalami Masalah

Misalnya:

```text
A → lag
B → lag
C → lag
D → lag
```

dan tujuan yang sama mengalami:

```text
latency naik
packet loss
jitter tinggi
```

Maka periksa:

```text
MikroTik
   ↓
Gateway
   ↓
Upstream
   ↓
Routing
   ↓
Peering
   ↓
Server/CDN
```

---

# 29. Pemeriksaan Traffic Queue

Melihat simple queue:

```mikrotik
/queue simple print stats
```

Perhatikan:

```text
rate
packet-rate
queued
dropped
```

Jika menggunakan FQ-CoDel, perhatikan apakah terjadi drop ketika trafik tinggi.

Untuk detail:

```mikrotik
/queue simple print stats
```

---

# 30. Upload/Download Drop

Jika:

```text
Upload Dropped = 0
Download Dropped = 0
```

itu berarti pada saat statistik tersebut diamati tidak terlihat paket yang dijatuhkan oleh queue tersebut.

Tetapi:

> Nol drop tidak otomatis membuktikan seluruh jaringan bebas masalah.

Pemeriksaan harus dilakukan ketika pelanggan sedang mengalami masalah.

---

# 31. FastTrack

Periksa:

```mikrotik
/ip firewall filter print
```

Cari rule FastTrack.

FastTrack dapat memengaruhi cara sebagian trafik melewati firewall/mangle/queue.

Jika melakukan troubleshooting queue, perhatikan apakah trafik pelanggan benar-benar melewati mekanisme queue yang dimaksud.

Jangan mengubah FastTrack hanya karena sedang troubleshooting tanpa memahami konfigurasi jaringan.

---

# 32. Mencari Trafik Pelanggan Berdasarkan IP

Misalnya:

```text
192.168.8.13
```

Connection tracking:

```mikrotik
/ip firewall connection print where src-address~"192.168.8.13"
```

Khusus TCP:

```mikrotik
/ip firewall connection print where src-address~"192.168.8.13" and protocol=tcp
```

Khusus UDP:

```mikrotik
/ip firewall connection print where src-address~"192.168.8.13" and protocol=udp
```

Khusus HTTPS:

```mikrotik
/ip firewall connection print where src-address~"192.168.8.13" and dst-port=443
```

---

# 33. Mengidentifikasi Layanan Berdasarkan DNS

DNS dapat digunakan sebagai petunjuk.

Contoh:

```text
youtube.com
googlevideo.com
ytimg.com
```

kemungkinan berkaitan dengan layanan YouTube.

Contoh:

```text
freefiremobile.com
akamaized.net
enacdn.net
```

dapat berkaitan dengan Free Fire atau infrastrukturnya.

Tetapi domain CDN umum seperti:

```text
akamaized.net
cloudfront.net
googleusercontent.com
```

tidak boleh langsung dianggap sebagai satu aplikasi tertentu.

---

# 34. Kenapa IP Tidak Selalu Bisa Menentukan Aplikasi

Satu IP dapat digunakan oleh:

* banyak domain
* banyak aplikasi
* CDN
* cloud service
* backend
* API

Sebaliknya satu aplikasi dapat menggunakan:

* banyak IP
* banyak ASN
* banyak CDN
* TCP
* UDP
* port dinamis

Karena itu:

```text
IP saja
```

tidak selalu cukup untuk identifikasi aplikasi.

---

# 35. Pengujian YouTube

### DNS

```mikrotik
/resolve youtube.com
```

### DNS cache

```mikrotik
/ip dns cache print where name~"youtube"
```

### Ping

Setelah mendapatkan IP:

```mikrotik
/ping IP-YOUTUBE count=20
```

### Traceroute

```mikrotik
/tool traceroute IP-YOUTUBE
```

### TCP HTTPS

```mikrotik
/tool telnet IP-YOUTUBE 443
```

---

# 36. Pengujian TikTok

DNS:

```mikrotik
/resolve tiktok.com
```

Cache:

```mikrotik
/ip dns cache print where name~"tiktok"
```

Setelah memperoleh IP:

```mikrotik
/ping IP-TIKTOK count=20
```

Traceroute:

```mikrotik
/tool traceroute IP-TIKTOK
```

TCP:

```mikrotik
/tool telnet IP-TIKTOK 443
```

---

# 37. Pengujian Google

```mikrotik
/resolve google.com
```

```mikrotik
/ping 8.8.8.8 count=20
```

```mikrotik
/tool traceroute 8.8.8.8
```

---

# 38. Pengujian Cloudflare

```mikrotik
/ping 1.1.1.1 count=20
```

```mikrotik
/tool traceroute 1.1.1.1
```

---

# 39. Membandingkan Beberapa Tujuan

Saat internet terasa bermasalah, lakukan:

```mikrotik
/ping 1.1.1.1 count=20
/ping 8.8.8.8 count=20
```

Kemudian ping IP layanan tertentu.

Contoh:

```text
Cloudflare → 5 ms
Google     → 8 ms
YouTube    → 20 ms
Game       → 20 ms
```

Jika semuanya stabil, koneksi umum terlihat sehat.

Jika:

```text
Cloudflare → 5 ms
Google     → 8 ms
YouTube    → 80 ms
Game       → 90 ms
```

maka perlu diperiksa jalur/routing/peering menuju tujuan tersebut.

---

# 40. Membandingkan MikroTik dengan Pelanggan

Ini sangat penting.

Misalnya dari MikroTik:

```text
server game
20 ms
0% loss
```

Kemudian dari PC pelanggan:

```text
server game
70 ms
5% loss
```

Maka jangan langsung menyalahkan server game.

Periksa:

```text
PC/HP
Wi-Fi
ONT
PPPoE
MikroTik
```

Sebaliknya, jika MikroTik dan banyak pelanggan sama-sama mengalami:

```text
latency tinggi
packet loss
```

maka kemungkinan masalah berada pada jalur bersama.

---

# 41. Pemeriksaan Bandwidth

Interface:

```mikrotik
/interface monitor-traffic ether1
```

Queue:

```mikrotik
/queue simple print stats
```

Periksa apakah bandwidth sedang mendekati kapasitas link.

Contoh:

```text
Link 1 Gbps
Traffic 980 Mbps
```

Kondisi seperti ini perlu diperhatikan karena dapat menyebabkan:

* antrean
* latency meningkat
* jitter
* packet loss
* bufferbloat

---

# 42. Bufferbloat

Bufferbloat terjadi ketika perangkat/jalur menyimpan terlalu banyak paket saat link penuh.

Gejala:

```text
Internet idle:
10 ms

Saat download penuh:
150 ms
```

atau:

```text
Idle:
10 ms

Saat upload penuh:
300 ms
```

Untuk RT/RW Net, ini sangat penting karena pelanggan dapat mengatakan:

> "Internet cepat, tetapi game lag."

Kecepatan download mungkin tinggi, tetapi latency ketika link penuh dapat melonjak.

---

# 43. Tes Latency Saat Idle dan Saat Beban Tinggi

Idle:

```mikrotik
/ping 1.1.1.1 count=30
```

Kemudian saat trafik tinggi, jalankan lagi:

```mikrotik
/ping 1.1.1.1 count=30
```

Bandingkan:

```text
Idle:
10 ms

Load:
80 ms
```

Jika latency meningkat drastis saat bandwidth penuh, periksa queue dan bufferbloat.

---

# 44. Menentukan Apakah Masalah Lokal atau Upstream

Gunakan pola berikut:

## Kasus A

```text
Ping gateway       normal
Ping internet      normal
Ping game          normal
Pelanggan tertentu lag
```

Periksa pelanggan/akses lokal.

## Kasus B

```text
Ping gateway       normal
Ping internet      normal
Ping game          tinggi/loss
```

Periksa jalur menuju game.

## Kasus C

```text
Ping gateway       tinggi/loss
Ping internet      tinggi/loss
Ping game          tinggi/loss
```

Periksa jaringan lokal/MikroTik/upstream.

## Kasus D

```text
Semua tujuan normal
Satu aplikasi bermasalah
```

Periksa domain/CDN/routing aplikasi tersebut.

---

# 45. Hal yang Jangan Dilakukan Saat Diagnosis

Jangan langsung:

```text
mengubah DNS
mengubah queue
mengubah MTU
mengubah MSS
mengubah FastTrack
mengubah firewall
mengubah routing
```

hanya karena satu pelanggan mengeluh.

Lakukan pengukuran dahulu.

Catat:

```text
waktu
pelanggan
IP
tujuan
latency
packet loss
jitter
traffic
queue
```

---

# 46. Template Pemeriksaan Pelanggan

Misalnya pelanggan:

```text
Nama : Eli
IP   : 192.168.8.13
```

Gunakan:

```mikrotik
/ppp active print where address=192.168.8.13
```

Cari interface:

```mikrotik
/interface print where name~"eli"
```

Ping internet:

```mikrotik
/ping 1.1.1.1 count=20
```

Ping tujuan:

```mikrotik
/ping IP-TUJUAN count=20
```

Traceroute:

```mikrotik
/tool traceroute IP-TUJUAN
```

Connection tracking:

```mikrotik
/ip firewall connection print where src-address~"192.168.8.13"
```

UDP:

```mikrotik
/ip firewall connection print where src-address~"192.168.8.13" and protocol=udp
```

TCP 443:

```mikrotik
/ip firewall connection print where src-address~"192.168.8.13" and dst-port=443
```

Torch:

```mikrotik
/tool torch <pppoe-eli>
```

---

# 47. Pemeriksaan Cepat 5 Menit

Jika pelanggan menelepon:

> "Internet saya lambat."

Lakukan:

```mikrotik
/ppp active print where address=IP-PELANGGAN
```

Kemudian:

```mikrotik
/ping IP-PELANGGAN count=10
```

```mikrotik
/ping 1.1.1.1 count=10
```

```mikrotik
/ping 8.8.8.8 count=10
```

Jika masalahnya aplikasi tertentu:

```mikrotik
/resolve DOMAIN
```

Kemudian:

```mikrotik
/ping IP-TUJUAN count=20
```

Jika perlu:

```mikrotik
/tool traceroute IP-TUJUAN
```

Terakhir:

```mikrotik
/ip firewall connection print where src-address~"IP-PELANGGAN"
```

---

# 48. Interpretasi Cepat

## Kondisi bagus

```text
Packet loss : 0%
Latency     : stabil
Jitter      : kecil
CPU router  : normal
Queue drop  : normal
Bandwidth   : tidak penuh
```

## Kondisi perlu diperiksa

```text
Packet loss > 0%
Latency naik
Jitter besar
Bandwidth penuh
CPU tinggi
Queue drop meningkat
```

## Kondisi sangat perlu diperiksa

```text
Packet loss tinggi
Latency melonjak
CPU router mendekati 100%
Link upstream penuh
Banyak pelanggan mengalami masalah bersamaan
```

---

# 49. Prinsip Penting untuk RT/RW Net

Jangan bertanya hanya:

> "Internet hidup atau mati?"

Pertanyaan yang lebih berguna:

```text
Apakah PPPoE pelanggan aktif?
Apakah pelanggan bisa mencapai gateway?
Apakah DNS bekerja?
Apakah internet umum reachable?
Apakah tujuan tertentu reachable?
Berapa latency?
Berapa packet loss?
Berapa jitter?
Apakah masalah terjadi saat bandwidth penuh?
Apakah pelanggan lain mengalami hal yang sama?
Apakah TCP normal?
Apakah UDP normal?
Apakah masalah hanya pada satu CDN/server?
```

Dengan pendekatan tersebut, troubleshooting menjadi jauh lebih terarah.

---

# 50. Ringkasan Perintah Penting

## Resource

```mikrotik
/system resource print
/tool profile
```

## Interface

```mikrotik
/interface print
/interface print stats
/interface monitor-traffic INTERFACE
```

## PPPoE

```mikrotik
/ppp active print
/ppp active print where address=IP
```

## DNS

```mikrotik
/resolve DOMAIN
/ip dns print
/ip dns cache print
/ip dns cache print where name~"DOMAIN"
```

## Ping

```mikrotik
/ping IP count=20
```

## Traceroute

```mikrotik
/tool traceroute IP
```

## TCP

```mikrotik
/tool telnet IP 443
```

## Connection Tracking

```mikrotik
/ip firewall connection print
/ip firewall connection print where src-address~"IP-PELANGGAN"
/ip firewall connection print where protocol=udp
/ip firewall connection print where dst-port=443
```

## Torch

```mikrotik
/tool torch INTERFACE
```

## Queue

```mikrotik
/queue simple print stats
```

---

# 51. Kesimpulan

Untuk operasional RT/RW Net, tidak ada satu perintah yang dapat menjawab seluruh masalah.

Gunakan kombinasi:

```text
PPP Active
     ↓
DNS
     ↓
Ping
     ↓
Traceroute
     ↓
TCP/UDP
     ↓
Connection Tracking
     ↓
Torch
     ↓
Queue
     ↓
Perbandingan pelanggan
```

DNS berguna untuk mengetahui domain yang muncul.

Ping berguna untuk mengukur reachability, latency, dan packet loss ICMP.

Traceroute berguna untuk melihat jalur.

Connection tracking berguna untuk melihat koneksi aktif.

Torch berguna untuk melihat trafik real-time pada interface.

Queue berguna untuk melihat kondisi antrean dan drop.

Untuk game dan aplikasi modern, **jangan menentukan aplikasi hanya berdasarkan port atau satu IP**, karena banyak layanan menggunakan CDN, cloud, HTTPS, QUIC, dan IP yang berubah-ubah.

Diagnosis terbaik dilakukan dengan membandingkan:

```text
Pelanggan
    VS
MikroTik
    VS
Tujuan internet
    VS
Pelanggan lain
```

Dengan cara ini, sumber masalah dapat dipersempit apakah berada pada:

```text
Perangkat pelanggan
Wi-Fi
ONT
PPPoE
MikroTik
Queue
Link upstream
Routing
Peering
CDN
Server tujuan
```
