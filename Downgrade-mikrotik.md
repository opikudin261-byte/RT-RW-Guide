# mikrotikDowngrade
cara downgrade versi mikrotik
# DOKUMENTASI / SOP

## Downgrade RouterOS MikroTik via Winbox

**Kasus referensi:**
MikroTik CCR1009-7G-1C-1S+
Arsitektur: `tile`
CPU: `tilegx`
RAM: 2 GB

**Downgrade:**
RouterOS `7.20.4` → `7.15.2`

**Metode:** Winbox + file `.npk`

---

# 1. TUJUAN

Dokumentasi ini menjelaskan cara melakukan downgrade RouterOS MikroTik menggunakan Winbox, khususnya ketika versi RouterOS yang sedang digunakan lebih baru daripada versi target.

Contoh:

```text
RouterOS saat ini : 7.20.4
RouterOS target   : 7.15.2
Architecture      : TILE
```

Metode ini tidak menggunakan Netinstall dan konfigurasi router tetap dipertahankan.

---

# 2. HAL PENTING SEBELUM DOWNGRADE

Sebelum melakukan downgrade, WAJIB mengetahui:

1. Versi RouterOS saat ini.
2. Arsitektur perangkat.
3. Versi RouterOS target.
4. Paket RouterOS yang sedang terpasang.
5. Kondisi konfigurasi.
6. Ruang penyimpanan yang tersedia.

Periksa:

```routeros
/system resource print
```

Contoh:

```text
version: 7.20.4
architecture-name: tile
board-name: CCR1009-7G-1C-1S+
```

Dalam kasus ini arsitektur adalah:

```text
tile
```

Maka paket yang digunakan harus sesuai:

```text
routeros-7.15.2-tile.npk
```

Jangan menggunakan paket:

```text
arm
arm64
mipsbe
mmips
```

karena berbeda arsitektur.

---

# 3. BUAT BACKUP TERLEBIH DAHULU

Sebelum downgrade, buat backup binary:

```routeros
/system backup save name=before-downgradeCCR
```

Kemudian buat export konfigurasi:

```routeros
/export file=before-downgradeCCR
```

Hasilnya akan menjadi:

```text
before-downgradeCCR.backup
before-downgradeCCR.rsc
```

Backup sebaiknya disalin ke komputer.

## Perbedaan backup dan export

### `.backup`

Contoh:

```text
before-downgradeCCR.backup
```

Merupakan backup binary konfigurasi MikroTik.

### `.rsc`

Contoh:

```text
before-downgradeCCR.rsc
```

Merupakan konfigurasi dalam bentuk script.

Untuk recovery atau migrasi antar versi/perangkat, file `.rsc` biasanya lebih fleksibel.

---

# 4. CEK PAKET YANG TERPASANG

Jalankan:

```routeros
/system package print
```

Contoh pada CCR1009:

```text
NAME      VERSION
routeros  7.20.4
wireless  7.20.4
```

Catat semua paket yang terpasang.

Jangan hanya melihat versi RouterOS melalui:

```routeros
/system resource print
```

karena proses downgrade berkaitan dengan paket `.npk` yang tersedia.

---

# 5. CEK DEVICE-MODE

Pada RouterOS versi baru, periksa:

```routeros
/system device-mode print
```

Perhatikan:

```text
allowed-versions:
install-any-version:
```

Contoh kondisi awal kasus ini:

```text
allowed-versions: 7.13+,6.49.8+
install-any-version: no
```

Pada kasus downgrade yang kami lakukan, kondisi tersebut menyebabkan proses downgrade tidak berhasil.

---

# 6. AKTIFKAN install-any-version

Jika `install-any-version` masih:

```text
no
```

jalankan:

```routeros
/system device-mode update install-any-version=yes
```

RouterOS akan memberikan pesan seperti:

```text
update: turn off power or reboot by pressing reset or mode button in 4m47s
```

Artinya perubahan belum langsung aktif.

---

# 7. KONFIRMASI DEVICE-MODE SECARA FISIK

Setelah menjalankan:

```routeros
/system device-mode update install-any-version=yes
```

lakukan **power-cycle fisik**.

Caranya:

1. Matikan MikroTik.
2. Cabut power.
3. Tunggu sekitar 5–10 detik.
4. Sambungkan power kembali.
5. Tunggu RouterOS boot.
6. Login kembali melalui Winbox.

Kemudian cek:

```routeros
/system device-mode print
```

Pastikan:

```text
install-any-version: yes
```

Contoh hasil yang benar:

```text
install-any-version: yes
attempt-count: 0
```

Jangan melanjutkan downgrade jika masih:

```text
install-any-version: no
```

---

# 8. DOWNLOAD FILE NPK SESUAI ARSITEKTUR

Untuk contoh CCR1009 TILE dengan target RouterOS 7.15.2:

```text
routeros-7.15.2-tile.npk
```

Pastikan file benar-benar untuk:

```text
TILE
```

Bukan paket arsitektur lain.

## Catatan nama file

Dalam percobaan awal terdapat file:

```text
routeros-7.15.2.npk
```

Kemudian digunakan:

```text
routeros-7.15.2-tile.npk
```

Untuk perangkat TILE, gunakan paket yang secara jelas sesuai dengan arsitektur perangkat.

---

# 9. UPLOAD NPK MELALUI WINBOX

Login Winbox.

Buka:

```text
Files
```

Drag & drop file:

```text
routeros-7.15.2-tile.npk
```

ke jendela Files.

Kemudian cek:

```routeros
/file print
```

Contoh:

```text
NAME
routeros-7.15.2-tile.npk
```

---

# 10. TIDAK PERLU DOUBLE-CLICK FILE NPK

File `.npk` tidak perlu dibuka atau dieksekusi secara manual.

Cukup letakkan file `.npk` di:

```text
Files
```

Kemudian RouterOS akan memprosesnya melalui perintah downgrade.

---

# 11. TENTANG PAKET WIRELESS

Periksa:

```routeros
/system package print
```

Jika perangkat memang menggunakan paket tambahan tertentu, pastikan paket target tersedia dan sesuai versi.

Namun pada CCR1009-7G-1C-1S+ yang menjadi contoh dokumentasi ini, tidak ada radio Wi-Fi bawaan.

Pada kasus ini paket utama yang digunakan untuk downgrade adalah:

```text
routeros-7.15.2-tile.npk
```

Jangan menambahkan paket secara sembarangan hanya karena nama paket tersebut ada pada router.

Prinsipnya:

> Paket target harus sesuai dengan paket dan fungsi yang memang digunakan perangkat.

---

# 12. JALANKAN PROSES DOWNGRADE

Setelah:

```text
install-any-version: yes
```

dan file `.npk` sudah berada di Files:

```text
routeros-7.15.2-tile.npk
```

jalankan:

```routeros
/system package downgrade
```

Jika muncul konfirmasi:

```text
Do you really want to downgrade? [y/N]:
```

jawab:

```text
y
```

RouterOS akan melakukan reboot.

---

# 13. JANGAN PANIK JIKA FILE NPK HILANG

Setelah proses downgrade/reboot, file:

```text
routeros-7.15.2-tile.npk
```

dapat hilang dari:

```text
/file print
```

Hal ini **bukan berarti proses gagal**.

File `.npk` merupakan file instalasi sementara yang diproses oleh RouterOS.

Jadi kondisi:

```text
sebelum downgrade:
routeros-7.15.2-tile.npk
```

kemudian setelah reboot:

```text
file tersebut tidak ada
```

adalah hal yang dapat terjadi.

Yang menentukan berhasil atau tidak adalah **versi RouterOS setelah reboot**, bukan keberadaan file `.npk`.

---

# 14. VERIFIKASI SETELAH REBOOT

Setelah MikroTik kembali online, jalankan:

```routeros
/system resource print
```

Target kasus ini:

```text
version: 7.15.2
architecture-name: tile
board-name: CCR1009-7G-1C-1S+
```

Contoh hasil:

```text
version: 7.15.2 (stable)
build-time: 2024-06-26 11:42:37
architecture-name: tile
board-name: CCR1009-7G-1C-1S+
```

Jika sudah seperti itu, downgrade berhasil.

---

# 15. VERIFIKASI PAKET

Jalankan:

```routeros
/system package print
```

Pastikan paket yang terpasang sesuai dengan versi target.

Contoh:

```text
NAME      VERSION
routeros  7.15.2
```

Jika ada paket tambahan, periksa juga versinya.

---

# 16. CEK KONFIGURASI SETELAH DOWNGRADE

Setelah downgrade, jangan langsung mengubah konfigurasi.

Periksa:

### IP Address

```routeros
/ip address print
```

### Routing

```routeros
/ip route print
```

### VLAN

```routeros
/interface vlan print
```

### Bridge

```routeros
/interface bridge print
```

### PPPoE Server

```routeros
/interface pppoe-server server print
```

### DHCP

```routeros
/ip dhcp-server print
```

### NAT

```routeros
/ip firewall nat print
```

### Firewall Filter

```routeros
/ip firewall filter print
```

### Queue

```routeros
/queue simple print
```

---

# 17. TES FUNGSI UTAMA

Setelah downgrade, lakukan pengujian:

1. Internet dari LAN.
2. PPPoE login.
3. VLAN.
4. DHCP.
5. NAT.
6. DNS.
7. Hotspot jika digunakan.
8. Simple Queue.
9. Firewall.
10. Akses Winbox.
11. Akses service yang diperlukan.
12. Monitoring traffic.

Jangan langsung menganggap downgrade berhasil hanya karena versi sudah berubah.

Pastikan fungsi jaringan tetap normal.

---

# 18. JIKA DOWNGRADE GAGAL

Jika setelah:

```routeros
/system package downgrade
```

router reboot tetapi masih:

```text
version: 7.20.4
```

**Jangan langsung mengulang proses berkali-kali.**

Periksa:

```routeros
/system device-mode print
```

Pastikan:

```text
install-any-version: yes
```

Kemudian cek:

```routeros
/file print
```

dan:

```routeros
/system package print
```

Jika perlu, periksa log:

```routeros
/log print where message~"package|upgrade|downgrade|install"
```

Log dapat menunjukkan alasan paket tidak dipasang.

---

# 19. PELAJARAN DARI PERCOBAAN PERTAMA

Pada kasus CCR1009 ini, percobaan awal gagal walaupun file:

```text
routeros-7.15.2-tile.npk
```

sudah di-upload.

Setelah reboot:

```text
file .npk hilang
```

tetapi:

```text
RouterOS tetap 7.20.4
```

Awalnya hal tersebut terlihat seperti paket hilang tanpa digunakan.

Setelah diperiksa, ternyata:

```text
install-any-version: no
```

Kemudian dilakukan:

```routeros
/system device-mode update install-any-version=yes
```

dan dikonfirmasi melalui **power-cycle fisik**.

Setelah:

```text
install-any-version: yes
```

proses:

```routeros
/system package downgrade
```

berhasil dan RouterOS akhirnya menjadi:

```text
7.15.2
```

---

# 20. URUTAN SOP SINGKAT

Untuk kebutuhan di masa depan, gunakan checklist berikut.

```text
[ ] 1. Cek versi sekarang
      /system resource print

[ ] 2. Cek architecture
      architecture-name

[ ] 3. Cek paket
      /system package print

[ ] 4. Backup
      /system backup save name=backup-sebelum-downgrade

[ ] 5. Export
      /export file=export-sebelum-downgrade

[ ] 6. Cek device-mode
      /system device-mode print

[ ] 7. Jika perlu:
      /system device-mode update install-any-version=yes

[ ] 8. Power-cycle fisik

[ ] 9. Pastikan:
      install-any-version: yes

[ ] 10. Download NPK sesuai architecture

[ ] 11. Upload NPK ke Files

[ ] 12. Jalankan:
       /system package downgrade

[ ] 13. Jawab:
       y

[ ] 14. Tunggu reboot

[ ] 15. Cek:
       /system resource print

[ ] 16. Pastikan versi target

[ ] 17. Cek:
       /system package print

[ ] 18. Tes seluruh fungsi jaringan
```

---

# 21. SETELAH DOWNGRADE BERHASIL

Setelah router dipastikan stabil, periksa kembali:

```routeros
/system device-mode print
```

Jika memang tidak membutuhkan kemampuan instalasi versi apa pun, pertimbangkan mengembalikan:

```text
install-any-version: no
```

dengan:

```routeros
/system device-mode update install-any-version=no
```

Kemudian lakukan konfirmasi fisik sesuai instruksi RouterOS.

**Jangan melakukan perubahan ini sebelum router benar-benar dipastikan stabil.**

---

# 22. KESIMPULAN

Untuk downgrade RouterOS melalui Winbox, inti prosesnya adalah:

```text
BACKUP
   ↓
CEK ARCHITECTURE
   ↓
DOWNLOAD NPK YANG SESUAI
   ↓
CEK DEVICE-MODE
   ↓
AKTIFKAN install-any-version JIKA DIPERLUKAN
   ↓
KONFIRMASI POWER-CYCLE
   ↓
UPLOAD NPK
   ↓
/system package downgrade
   ↓
REBOOT
   ↓
CEK VERSION
   ↓
TES KONFIGURASI DAN JARINGAN
```

### Kasus nyata CCR1009

```text
Awal:
RouterOS 7.20.4
TILE

Target:
RouterOS 7.15.2
TILE

Penghalang awal:
install-any-version = no

Solusi:
install-any-version = yes
+ power-cycle fisik

Kemudian:
routeros-7.15.2-tile.npk
→ Files
→ /system package downgrade

Hasil:
RouterOS 7.15.2 (stable)
```

**Catatan terpenting:** keberadaan atau hilangnya file `.npk` setelah reboot bukan indikator keberhasilan downgrade. Indikator utamanya adalah hasil:

```routeros
/system resource print
```

dan:

```routeros
/system package print
```

yang harus menunjukkan versi target.
