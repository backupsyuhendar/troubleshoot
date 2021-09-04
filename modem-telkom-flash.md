# menyambungkan modem telkom flash di linux

sumber: 
https://t.me/GNULinuxIndonesia/268215


## 1. Cek interface
```bash
cat /proc/net/dev
```
output:
```bash
❯ cat /proc/net/dev
Inter-|   Receive                                                |  Transmit
 face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
    lo: 11103790  141543    0    0    0     0          0         0 11103790  141543    0    0    0     0       0          0
enp5s0:       0       0    0    0    0     0          0         0        0       0    0    0    0     0       2          0
wlp4s0: 40341949   46892    0    0    0     0          0         0  5293016   34565    0    0    0     0       0          0
```
- Kalau ada modemnya biasanya muncul seperti usbnet0 atau usbtty0 dan sejenisnya
Berarti belum terbaca if nya, ada masalah di driver atau firmware
Kudu dipasang dlu mungkin


## 2. Pastikan paket ini terpasang

`usb_modeswitch`
`modem-manager` atau `modemmanager`
tambahan:
`modem-manager-gui`

- cek juga paket ppp-nya pakai 
```bash
dpkg -l | grep ppp
```

- setelah diinstall, coba `lsusb`
- setelah pasang use-modeswitch seharusnya modemnya di kenali sebagai modem
output:
```bash
❯ lsusb
Bus 002 Device 003: ID 064e:a103 Suyin Corp. Acer/HP Integrated Webcam [CN0314]
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 007 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 006 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 005 Device 002: ID 0000:3825   USB OPTICAL MOUSE
Bus 005 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 001 Device 019: ID 05c6:6000 Qualcomm, Inc. Siemens SG75
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 004 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 003 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
```

contoh punya gw itu `Qualcomm, Inc. Siemens SG75`

- Yg terpenting catat `ID VendorID:ProductID` nya nanti bakal digunain buat configurasi selanjutnya 


## 3. Cek di `/dev/tty*` adakah yang semisal `/dev/ttyUSB0` atau sejenisnya?

- Misal modem terdeteksi sebagai `/dev/ttyUSB0`. Maka simlinks ke /dev/modem:
```bash
ln -s /dev/ttyUSB0 /dev/modem
```
- Kalau sudah biasanya akan terdeteksi di ubuntu net-admin kalau pakai ubuntu. 
Atau kalau pakai kppp tinggal arahkan devicenya ke /dev/modem

- Kalau bingung yang mana yang usb modem, coba cabut dulu usb modemnya lalu colokin lagi. 
tunggu beberapa saat lalu jalankan 
```bash
sudo dmesg | grep tty
```
- Cari yang tulisan 0x3f8 atau 0x2f8.  
Lihat itu ada di /dev/tty[apa?]

output gw:
```bash
.
.
.
[26793.895400] option1 ttyUSB0: GSM modem (1-port) converter now disconnected from ttyUSB0
[26793.898173] option1 ttyUSB1: GSM modem (1-port) converter now disconnected from ttyUSB1
[26793.898495] option1 ttyUSB2: GSM modem (1-port) converter now disconnected from ttyUSB2
[26793.899504] option1 ttyUSB3: GSM modem (1-port) converter now disconnected from ttyUSB3
[26818.538334] usb 1-1: GSM modem (1-port) converter now attached to ttyUSB0
[26818.539153] usb 1-1: GSM modem (1-port) converter now attached to ttyUSB1
[26818.539393] usb 1-1: GSM modem (1-port) converter now attached to ttyUSB2
[26818.540649] usb 1-1: GSM modem (1-port) converter now attached to ttyUSB3
```

*note : jd gw sepmpet berhenti sampe diatas doang, karna gk tau lg mana yg kudu di symlink itu*

output lain:
```bash
[number...] console [tty0] enabled
```
Jika seperti diatas tdk aktif modemnya


## 4. Oprek file configuration

- coba catat ID dari hasil lsusb buat modem. lalu masukkan ke dalam konfigurasi `/etc/usb_modeswitch.conf`. Isinya:
```bash
# Modem Saya (tulis merek modem biar jelas)
DefaultVendor = VendorID
DefaultProduct = ProductID
TargetVendor = VendorID
TargetProduct = ProductID
MessageContent="55534243123456780000000000000a11062000000000000100000000000000"
```
yg gw isi:
```bash
# Modem Saya (Advan JETZ - Telkomsel Flash)
DefaultVendor = "05c6"
DefaultProduct = "6000"
TargetVendor = "05c6"
TargetProduct = "6000"
MessageContent="55534243123456780000000000000a11062000000000000100000000000000"
```
- lalu jalankan 
```bash
sudo usb_modeswitch -c /etc/usb_modeswitch.conf
```
- Coba gunakan network-manager atau connman, seharusnya sudah terdeteksi di situ. Semoga berhasil


output gw:
```bash
❯ sudo usb_modeswitch -c /etc/usb_modeswitch.conf
Look for target devices ...
 Found devices in target mode or class (1)
Look for default devices ...
 Found devices in default mode (1)
Access device 021 on bus 001
Get the current device configuration ...
Current configuration number is 1
Use interface number 0
 with class 255
Error: can't use storage command in MessageContent with interface 0; interface class is 255, expected 8. Abort
```
Ada kesalahan di `MessageContent` nya

## 5. Cari MessageContent-nya

- Skrg cek ada tidak berkas `/etc/usb_modswitch.d/[ProductID]:[VendorID]` yang sesuai dengan ID dari modem hasil output lsusb. 
- Cek isinya dan cari baris yang ada MessageContent di situ. 
- Copas isinya ke `/etc/usb_modswitch.conf` . 
(Maaf terlewat beri tahu isi dari MessageContent ambilnya di mana. Sesuaikan dengan modem masing-masing)

Jika tidak ada

- coba isi yang di `/etc/usb_modswitch.conf` yang ditambahka dicomment dulu. 
- Lalu modem cabut dan colokkan. Kemudian cek di `/etc/usb_modswitch.d`. Ada file apa di situ setelahnya

Hasil nya tidak ada juga (di gw)


## 6. Eject 

Kemungkinan serial port nya belum aktif

- Sekarang begini. Pas modem dicolokkan. Cek apakah ada berkas `/dev/sr0`, `/dev/sr1` dst. 
Kalau ada sudo eject `/dev/sr1` atau 
`sudo eject /dev/sr0` . lakukan untuk /dev/sr... Yang ada.


## 7. Modeprobe

- Kalau sudah. Jalankan 
```bash
modprobe -rvf usbserial
```
atau
```bash
modprobe -rvf usb-serial 
```
mana yang cocok tergantung sistem.

output gw:
```bash
❯ modprobe -rvf usbserial
modprobe: FATAL: Module usbserial is builtin.
```
output lain:
```bash
❯ modprobe -rvf usbserial
modprobe: FATAL: Module usbserial in use.
```

- Kalau sudah. Jalankan 
```bash
modprobe -vf usb-serial -V VendorID -P productID
```
- Cek pakai dmesg apakah usbserialnya aktif
```bash
sudo dmesg| grep usbserial
```
output:
```bash
❯ sudo dmesg| grep usbserial
[sudo] password for dar729:
[17527.116787] usbserial: USB Serial support registered for GSM modem (1-port)
```

_note: di grup linux terakhir sampai situ aja, karna si TS kondisinya beda ama gw (butuh firmware tambahan dan blacklist
apa gitu di `/etc/modules-load.d/` nya) selebihnya ini gw nemu dr browsing_

## 8. Bikin rules

 - Bikin rules seperti ini di `/etc/udev/rules.d/advan-jetz-3.rules`
```bash
# /etc/udev/rules.d/advan-jetz-3.rules
#
# Advan Jetz 3
#

SUBSYSTEM=="usb", SYSFS{idVendor}=="05c6",
SYSFS{idProduct}=="9013", RUN+="/usr/sbin/usb_modeswitch -default-vendor 0X05c6 -default-product 0X6000 -message-content 5553424312345678c00000008000069f030000000000000000000000000000"
```


## Jalankan `wvdial`

- Langkah terakhir coba pake `wvdial`
- Pokokknya configurasiin di `/etc/wvdial.conf`

- Gw isi buat kartu tri, klo Sim-card lain sesuaikan aja, bisa dicari di google
```bash
[Dialer three]
Init1 = ATZ
Init2 = ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0
Stupid Mode = 1
Modem Type = Analog Modem
Command Line = ATDT
ISDN = 0
New PPPD = yes
Phone = *99#
Modem = /dev/ttyUSB0
Username = 3data
Password = 3data
Baud = 460800
```

- Nemu yg lain dr gugel:

```bash
[Dialer Defaults]
Init2 = ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0
Modem Type = Analog Modem
Phone = *99#
ISDN = 0
Username = 3data (tergantung kartu gsm/cdma kita, misal saya pakai 3)
Init1 = ATZ
Password = 3data (tergantung kartu gsm/cdma kita, misal saya pakai 3)
Modem = /dev/ttyUSB1
Baud = 9600
Stupid Mode = yes
```

```bash
Telkomsel
Phone = *99#
Username = wap
Password = wap123

XL
Phone = *99#
Username: xlgprs
Password: proxl

Indosat
Phone = *99#
1. Time-besed
Username = indosat@durasi
Password = indosat@durasi
2. Volume-besed
Username = indosat
Password = indosat

Axis
Phone = *99#
Username: axis
Password: 123456

3
Phone = *99#
Username = 3data
Password = 3data
```

- Config lain (buat smartfren dan mobi)
```bash
[Dialer smart] 
Init1 = ATZ
Init2 = ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0
Modem Type = Analog Modem
Phone = #777
ISDN = 0
Username = smart
Password = smart
Modem = /dev/ttyUSB0
Baud = 460800
Command Line = ATDT
Stupid Mode = 1

[Dialer mobi]
Init1 = ATZ
Init2 = ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0
Modem Type = Analog Modem
Phone = #777
ISDN = 0
Username = m8
Password = m8
Modem = /dev/ttyUSB0
Baud = 460800
Command Line = ATDT
Stupid Mode = 1

```


- Lalu buat script utk jalaninnya, sabeb namanya (klo gw `modemgw`)
```bash
#!/bin/bash
# sudo eject /dev/sr1
sleep 2
sudo modprobe usbserial vendor=0X05c6 product=0X6000
sleep 2
sudo wvdial three
```
_note: yg `eject` itu sesuaikan aja kalau memang selalu ada (pas baru colok) di-uncomment_

- Trus masukin script itu ke `PATH`, default utk local user ada di `~/.local/bin/`
- Trus `chmod +x filenya` eksekusi dah
seperti ini klo berhasil:
```bash
❯ modemgw
[sudo] password for dar729:
--> WvDial: Internet dialer version 1.61
--> Initializing modem.
--> Sending: ATZ
OK
--> Sending: ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0
OK
--> Modem initialized.
--> Sending: ATDT*99#
--> Waiting for carrier.
ATDT*99#
CONNECT 28800000
--> Carrier detected.  Starting PPP immediately.
--> Starting pppd at Wed Sep  1 07:52:03 2021
--> Pid of pppd: 31843
--> Using interface ppp0
--> local  IP address 10.28.200.6
--> remote IP address 10.64.64.64
--> primary   DNS address 10.4.246.116
--> secondary DNS address 10.0.27.115

```

Saya tau itu dari kata beliau:
_"Kalau dak salah tangkap pakai ubuntu 18.04? Hanya modemnya baru dengar namanya. Saya seringnya pakai huawei dan pernah juga sih zte. Dak rewel di linux debian. Colok, `usb_modeswitch` jalan. Run `wvdial`. Kelar. Wus.....meluncur berselancar."_

Clue-nya hanya : "Colok, `usb_modeswitch` jalan. Run `wvdial`" 

## Kesimpulan

### Sebetulnya cuma gini aja sih yg penting
- pasang `usb_modeswitch`, `modemmanager`, `wvdial`, `ppp`
- trus liat `lsusb` ingat `vendorID:ProductID` nya
- trus liat `/dev/ttyUSB*` biasanya di `USB0`
- klo mau pastiin pake `dmesg | grep tty` liat attached dimana modem nya
- trus configurasiin `/etc/usb_modeswitch.conf` 
- trus configurasiin `/etc/wvdial.conf` sesuai dgn sim-card nya
- trus buat script, jalanin deh scriptnya



