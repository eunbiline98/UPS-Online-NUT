# How to configure gateway NUT &amp; conversion data with SNMP Protocol V1 or V2c
- Apa? Skrip dasar yang menginstal Network UPS Tools (NUT) dan beberapa paket lainnya.
- Bagaimana? Dengan mengunduh skrip kami dan menjalankannya di Raspberry Pi atau sistem Linux serupa.
- Mengapa? Jadi Anda dapat memiliki pemantauan HTTP dan SNMP v1 / v2c dalam beberapa menit setelah menjalankan skrip.
Tentu saja, ada lebih banyak yang mungkin ingin Anda lakukan dengan perangkat lunak UPS Anda seperti menambahkan konfigurasi shutdown, memantau beberapa UPS, atau memperkuat pengaturan dengan protokol terenkripsi dan konfigurasi aman. 

# Why are we doing this?
Proyek Network UPS Tools (NUT) melakukan banyak pekerjaan baik di industri maupun ISP internet. Banyak integrasi UPS yang Anda lihat di berbagai sistem (misalnya, perangkat NAS dan server konsol) dimungkinkan oleh NUT. Sederhananya, ada banyak hal keren yang dapat Anda lakukan dengan NUT.

Tapi, kita sering melihat masalah ini:

- Orang tidak tahu semua hal keren yang bisa dilakukan NUT
- NUT bisa rumit untuk dikonfigurasi jika Anda tidak tahu harus mulai dari mana
Jadi, saya ingin memberi tahu Anda tentang NUT dan membuatnya mudah untuk memulai. saya juga ingin mendapatkan beberapa percakapan seputar apa yang ingin Anda lakukan dengan sistem Linux dan UPS Anda

# Supported UPS list
Proses ini harus bekerja dengan berbagai sistem UPS dengan port USB. Yang mengatakan, pengujian saya telah dengan UPS bermerek APC dan Eaton tertentu. Berikut adalah daftar UPS yang direkomendasikan:

- APC BX-xxxx Series
- Eaton 5E Series

Selain itu saya juga melakukan RND dengan sistem UPS menggunakan port ethernet dari card management, pengujian saya telah dengan UPS bermerek Aplus tertentu. Berikut adalah daftar UPS yang direkomendasikan:

- Aplus PlusIII-3KLRB

# How to use the basic script
⚠️ Disini saya melakukan configure menggunakan OrangePI PC dengan OS Armbian, dan tidak menjelaskan cara install OS armbian di OrangePI serta memahami dasar command CLI pada Linux
## Prasyarat
- UPS yang didukung dapat dilihat di -> https://networkupstools.org/stable-hcl.html, atau kalian bisa mengulik dengam mencocokan driver yang support
- Komputer Linux yang menjalankan OS Raspberry Pi, Ubuntu, atau sistem operasi serupa yang menggunakan apt
- root/sudo privleges
- Koneksi USB antara UPS dan komputer Linux. Anda dapat mengonfirmasi bahwa UPS terhubung menggunakan perintah `lsusb`. Pada contoh di bawah ini, kita melihat UPS `American Power Conversion Uninterruptible Power Supply
`, itu driver milik APC UPS
  ![image](https://github.com/eunbiline98/UPS-Online-NUT/assets/50385294/8896aebf-4ca0-49b2-8b3d-1ac919b76fcc)
- Ports 161 (UDP), 80 (TCP), and 3493 (TCP) open
- None of these packages installed:
  -  nut
  -  nut-cgi
  -  snmp
  -  snmpd
  -  libsnmp-dev
  -  snmp-mibs-downloader
  -  net-snmp
 ### (Optional) Update your system
Jika Anda belum memperbarui sistem Anda dalam beberapa saat, itu ide yang baik untuk mengakses terminal komputer Anda (misalnya, melalui SSH) dan menjalankan:
```
sudo apt update
```
Untuk memastikan Anda mendapatkan paket terbaru yang tersedia di langkah berikutnya.  

### Install NUT and nut-cgi
1. Akses terminal komputer Anda (misalnya, melalui SSH)
2. Gunakan 'apt' untuk menginstal paket 'nut' dan 'nut-cgi'
```
sudo apt install nut nut-cgi
```
![image](https://user-images.githubusercontent.com/17661803/160892838-22814911-19a2-423c-94d9-132545433a09.png)

### Configure NUT
1. Run this command:
```
nut-scanner -UNq
```
The output should look similar to:
```
SNMP library not found. SNMP search disabled.
Neon library not found. XML search disabled.
IPMI library not found. IPMI search disabled.
[nutdev1]
        driver = "usbhid-ups"
        port = "auto"
        vendorid = "09AE"
        productid = "4004"
        bus = "003"
```
You'll need everything under (and including) `[nutdev1]` for the next step.
![image](https://user-images.githubusercontent.com/17661803/160893706-69e5ffbf-3a77-446e-a602-b82fe8ee51cc.png)
