# How to konfigurasi gateway NUT &amp; conversion data with SNMP Protocol V1 or V2c
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
⚠️ Disini saya melakukan konfigurasi menggunakan OrangePI PC dengan OS Armbian, dan tidak menjelaskan cara install OS armbian di OrangePI serta memahami dasar command CLI pada Linux
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
2. Gunakan `apt` untuk menginstal paket `nut` dan `nut-cgi`
```
sudo apt install nut nut-cgi
```
![image](https://github.com/eunbiline98/UPS-Online-NUT/assets/50385294/b99d7acc-e232-457d-a2dd-cb4eaedd2ec7)

### konfigurasi NUT
1. Jalankan perintah ini:
```
nut-scanner -UNq
```
Output akan terlihat mirip dengan:
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
Anda akan membutuhkan semuanya di bawah (dan termasuk) `[nutdev1]` untuk langkah berikutnya.
![image](https://github.com/eunbiline98/UPS-Online-NUT/assets/50385294/572ba910-b4a5-4d03-a665-f3c598ee7331)

2. Gunakan editor teks (misalnya, 'nano' atau 'vi') untuk mengedit /etc/nut/ups.conf dan tambahkan output dari langkah 1. Menggunakan contoh output kami, bagian file ups.conf yang tidak dikomentari akan terlihat seperti ini:
```
maxretry = 3
[ups-apc]
        driver = "usbhid-ups"
        port = "auto"
        vendorid = "051D"
        productid = "0002"
        bus = "006"
```
Akan tetapi kalian bisa merubah value ID `[nutdev1]` dengan value ID yang kalian ingin kan sebagai contoh `[ups-apc]`

3.  Gunakan editor teks (misalnya 'nano' atau 'vi') untuk mengedit /etc/nut/nut.conf, agar gateway NUT kalian di posisi mode standalone untuk memudahkan dalam kustomisasi dalam GET data
```
MODE=standalone
```
4.  Gunakan editor teks (misalnya 'nano' atau 'vi') untuk mengedit /etc/nut/upsd.conf
```
# =======================================================================
# LISTEN <address> [<port>]
 LISTEN 127.0.0.1 3493
 LISTEN 192.168.8.47 3493

# LISTEN ::1 3493
#
# This defaults to the localhost listening addresses and port 3493.
# In case of IP v4 or v6 disabled kernel, only the available one will be used.
#
# You may specify each interface you want upsd to listen on for connections,
# optionally with a port number.
#
# You may need this if you have multiple interfaces on your machine and
# you don't want upsd to listen to all interfaces (for instance on a
# firewall, you may not want to listen to the external interface).
#
# This will only be read at startup of upsd.  If you make changes here,
# you'll need to restart upsd, reload will have no effect.

# =======================================================================
[root]
password = (isi sesuai pass root kalian)
actions = set
actions = fsd
instcmds = all

[monmaster]
password = monmaster
upsmon master

[monslave]
password = monslave
upsmon slave
```
5.  Gunakan editor teks (misalnya 'nano' atau 'vi') untuk mengedit /etc/nut/upsd.users
```
[ups-apc]
password = (isi sesuai pass root kalian)
upsmon = master

```
6.  Gunakan editor teks (misalnya 'nano' atau 'vi') untuk mengedit /etc/nut/host.conf, untuk konfigurasi alamat host serta ID ups pada gateway NUT kalian
```
# This file is used to control the CGI programs.  If you have not
# installed them, you may safely ignore or delete this file.
#
# -----------------------------------------------------------------------
#
# upsstats will use the list of MONITOR entries when displaying the
# default template (upsstats.html).  The "FOREACHUPS" directive in the
# template will use this file to find systems running upsd.
#
# upsstats and upsimage also use this file to determine if a host may be
# monitored.  This keeps evil people from using your system to annoy
# others with unintended queries.
#
# upsset presents a list of systems that may be viewed and controlled
# using this file.
#
# -----------------------------------------------------------------------
#
# Usage: list systems running upsd that you want to monitor
#
# MONITOR <system> "<host description>"
#
# Examples:
#
 MONITOR ups-apc@localhost "UPS APC-1600 RND"
# MONITOR su2200@10.64.1.1 "Finance department"
# MONITOR matrix@shs-server.example.edu "Sierra High School data room #1"

```
7. Jalankan perintah ini, untuk mengaktifkan konfigurasi yang telah kalian lakukan 
```
sudo upsdrvctl start
```
![image](https://github.com/eunbiline98/UPS-Online-NUT/assets/50385294/c191fff4-a67c-4576-a41a-7945952844d5)
⚠️ Jangan khawatir jika ada pesan `Communications with UPS ups-apc@localhost lost`, kalian bisa melakukan dengan merestart service NUT-Server dengan command
```
sudo systemctl restart nut-server.service
```
Jika sudah melakukan restart service NUT-Server lakukan kembali step 7 hingga muncul pesan `Communications with UPS ups-apc@localhost established`, jika belum berhasil cek kembali konfigurasi dari step 1 hingga step 6

8. Pastikan UPS Anda sekarang berkomunikasi dengan menjalankan perintah `upsc ups-apc@localhost`. Output akan terlihat mirip dengan:
```
Init SSL without certificate database
battery.charge: 100
battery.charge.low: 10
battery.mfr.date: 2001/01/01
battery.runtime: 3320
battery.runtime.low: 120
battery.type: PbAc
battery.voltage: 27.3
battery.voltage.nominal: 24.0
device.mfr: American Power Conversion
device.model: Back-UPS BX1600MI
device.serial: 9B2317A17742
device.type: ups
driver.name: usbhid-ups
driver.parameter.bus: 006
driver.parameter.pollfreq: 30
driver.parameter.pollinterval: 2
driver.parameter.port: auto
driver.parameter.productid: 0002
driver.parameter.synchronous: no
driver.parameter.vendorid: 051D
driver.version: 2.7.4
driver.version.data: APC HID 0.96
driver.version.internal: 0.41
input.sensitivity: medium
input.transfer.high: 295
input.transfer.low: 145
input.voltage: 234.0
input.voltage.nominal: 230
ups.beeper.status: enabled
ups.delay.shutdown: 20
ups.firmware: 378600G -302202G
ups.load: 0
ups.mfr: American Power Conversion
ups.mfr.date: 2023/04/28
ups.model: Back-UPS BX1600MI
ups.productid: 0002
ups.realpower.nominal: 900
ups.serial: 9B2317A17742
ups.status: OL
ups.test.result: Done and passed
ups.timer.reboot: 0
ups.timer.shutdown: -1
ups.vendorid: 051d
```
### Configure Apache
1. Apache diinstal ketika kami intsalled paket `nut-cgi`. Aktifkan modul cgi dengan perintah ini:
  ```
  a2enmod cgi
  ```
   Anda akan melihat output yang mirip dengan:
```
Your MPM seems to be threaded. Selecting cgid instead of cgi.
Enabling module cgid.
To activate the new configuration, you need to run:
  systemctl restart apache2
```

2. Mulai ulang Apache dengan perintah ini:
```
systemctl restart apache2
```
![image](https://github.com/eunbiline98/UPS-Online-NUT/assets/50385294/0d005f76-b61f-4f4e-8976-789403a265e7)
### Install Net-SNMP
1.  Gunakan `apt` untuk menginstal paket `snmp`, `snmpd`, `libsnmp-dev`, dan 'snmp-mibs-downloader'
```
sudo apt install snmp snmpd libsnmp-dev snmp-mibs-downloader
```
![image](https://github.com/eunbiline98/UPS-Online-NUT/assets/50385294/dd9d61d1-b18f-4a84-8f77-43a9f792fb9c)

### Configure Net-SNMP
1. Gunakan editor teks (misalnya, `nano` atau `vi`) untuk membuat file /etc/snmp/snmpd.conf dan tambahkan string komunitas SNMP v2c Anda ke awal. Untuk membuat `upsapc` hanya-baca, tambahkan entri ini (edit string komunitas Anda menjadi sesuatu yang hanya Anda yang tahu):
```
rocommunity upsapc
```
Jangan simpan perubahan Anda dulu, saya memiliki beberapa pengeditan lagi yang harus dilakukan. 
```
###########################################################################
#
# snmpd.conf
# An example configuration file for configuring the Net-SNMP agent ('snmpd')
# See snmpd.conf(5) man page for details
#
###########################################################################
# SECTION: System Information Setup
#

# syslocation: The [typically physical] location of the system.
#   Note that setting this value here means that when trying to
#   perform an snmp SET operation to the sysLocation.0 variable will make
#   the agent return the "notWritable" error code.  IE, including
#   this token in the snmpd.conf file will disable write access to
#   the variable.
#   arguments:  location_string
sysLocation    Sitting on the Dock of the Bay
sysContact     Me <me@example.org>

# sysservices: The proper value for the sysServices object.
#   arguments:  sysservices_number
sysServices    72
###########################################################################
# SECTION: Agent Operating Mode
#
#   This section defines how the agent will operate when it
#   is running.

# master: Should the agent operate as a master agent or not.
#   Currently, the only supported master agent type for this token
#   is "agentx".
#
#   arguments: (on|yes|agentx|all|off|no)

master  agentx

# agentaddress: The IP address and port number that the agent will listen on.
#   By default the agent listens to any and all traffic from any
#   interface on the default SNMP port (161).  This allows you to
#   specify which address, interface, transport type and port(s) that you
#   want the agent to listen on.  Multiple definitions of this token
#   are concatenated together (using ':'s).
#   arguments: [transport:]port[@interface/address],...

agentaddress  xxx.xxx.xxx.xxx,[::1] (sesuaikan IP gateway NUT kalian)
###########################################################################
# SECTION: Access Control Setup
#
#   This section defines who is allowed to talk to your running
#   snmp agent.

# Views
#   arguments viewname included [oid]

#  system + hrSystem groups only
view   systemonly  included   .1.3.6.1.2.1.1 
view   systemonly  included   .1.3.6.1.2.1.25.1


# rocommunity: a SNMPv1/SNMPv2c read-only access community name
#   arguments:  community [default|hostname|network/bits] [oid | -V view]

# Read-only access to everyone to the systemonly view
rocommunity  public default -V systemonly
rocommunity6 public default -V systemonly

# SNMPv3 doesn't use communities, but users with (optionally) an
# authentication and encryption string. This user needs to be created
# with what they can view with rouser/rwuser lines in this file.
# createUser username (MD5|SHA|SHA-512|SHA-384|SHA-256|SHA-224) authpassphrase [DES|AES] [privpassphrase]
# e.g.
# createuser authPrivUser SHA-512 myauthphrase AES myprivphrase
#
# This should be put into /var/lib/snmp/snmpd.conf
#
# rouser: a SNMPv3 read-only access username
#    arguments: username [noauth|auth|priv [OID | -V VIEW [CONTEXT]]]
rouser authPrivUser authpriv -V systemonly

# include a all *.conf files in a directory
includeDir /etc/snmp/snmpd.conf.d
rocommunity upsapc
extend-sh upsmodel "/bin/upsc ups-apc ups.model"
extend-sh upsserial "/bin/upsc ups-apc ups.serial"
extend-sh upsstatus "/bin/upsc ups-apc ups.status"
extend-sh battcharge "/bin/upsc ups-apc battery.charge"
extend-sh battruntimeest "/bin/upsc ups-apc battery.runtime"
extend-sh battvolts "/bin/upsc ups-apc battery.voltage"
extend-sh inputvolt "/bin/upsc ups-apc input.voltage"
extend-sh inputHZ "/bin/upsc ups-apc input.frequency"
extend-sh outputvolt "/bin/upsc ups-apc output.voltage"
extend-sh outputHZ "/bin/upsc ups-apc output.frequency"
```
data yang akan di konnversi snmp bisa disesuaikan sesuai dengan kebutuhan / yang dapat dibaca oleh gateway NUT, dan untuk OID jangan di ubah atau kalian ingin mengulik sendiri silahkan

3. Aktifkan layanan snmpd
```
sudo systemctl enable snmpd
```

4. Mulai ulang layanan snmpd
```
sudo  systemctl restart snmpd
```
5. Jalankan perintah ini, untuk memastikan bahwa data kalian berhasil di konversi ke SNMP 
```
snmpwalk -v2c -c upsapc xxxxx.xxxx.xxx.xxx .1.3.6.1.4.1.8072.1.3.2.4.1.2
```
kalian juga bisa snmpwalk dengan versi v1 dengan cara
```
snmpwalk -v1 -c upsapc xxx.xxx.xxx.xxx .1.3.6.1.4.1.8072.1.3.2.4.1.2
```
data success di konversi ke SNMP
![image](https://github.com/eunbiline98/UPS-Online-NUT/assets/50385294/d398f555-79d5-4b89-85e3-69a80bc81ca5)
### OID Library
⚠️ Pastikan kalian mengikuti konfigurasi Net-SNMP sesuai panduan, jika kalian custom silahkan research bersama 
Kalian bisa cek OID dari masing - masing parameter di sini -> https://github.com/eunbiline98/UPS-Online-NUT/blob/main/OIDexplainer.md

# Hasil
### HAVE A LOT OF FUN!
Kalian bisa memonitoring dengan cara mengakses gateway NUT di browser untuk mengtahui kondisi parameter saat ini -> http://xxx.xxx.xxx.xxx/cgi-bin/nut/upsstats.cgi
![image](https://github.com/eunbiline98/UPS-Online-NUT/assets/50385294/9d2d181b-aaf1-42aa-8ad6-52d44a602b1f)
Kalian juga bisa memasukan data SNMP yang sudah kalian konversi ke NMS untuk menyimpan record data, alert dan fungsi lain yang ada di NMS, seperti contoh saya menggunakan NMS PRTG 
![image](https://github.com/eunbiline98/UPS-Online-NUT/assets/50385294/720e2eb4-2cc8-4c00-a1eb-bcf7f3cbef53)




