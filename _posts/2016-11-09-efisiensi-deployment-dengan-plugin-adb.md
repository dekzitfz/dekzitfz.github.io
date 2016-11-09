---
layout: post
title:  "Efisiensi Deployment Android Dengan Plugin ADB"
date:   2016-11-09 13:46:07
categories: [android,plugin,adb,tips,deployment]
disqus_identifier: 106
comments: true
---

![logo adb](https://s17.postimg.org/4r1st7men/adb.jpg)

Saat kita ingin menguji aplikasi android di suatu perangkat, pada umumnya kita mempunyai dua opsi, yaitu mengujinya melalui emulator atau mengujinya melalui device asli. Namun terkadang untuk laptop/PC yang memiliki spesifikasi pas-pasan cenderung enggan menggunakan emulator dikarenakan dapat membuat performa laptop/PC nya melambat. Alternatifnya tentu saja menggunakan device asli yang terhubung via kabel data. 

Namun tetap saja, jika kita menghubungkan device kita dengan kabel terlalu lama, terkadang daya listrik yang masuk ke device menjadi tidak stabil. Salah satu solusi untuk mengatasi masalah ini yaitu menggunakan plugin pada android studio.

<!--more-->

### Android WIFI ADB

![logo adb wifi](https://raw.githubusercontent.com/pedrovgs/AndroidWiFiADB/master/art/AndroidWiFiADBIcon.png)

Pada android studio ada banyak plugin yang bisa digunakan sesuai kebutuhannya. Pada kasus kali ini saya akan membahas sebuah plugin yang sudah lama saya gunakan dan sangat membantu saya dalam proses pengembangan aplikasi android, yaitu [Android WIFI ADB][pluginADBWIFI].

ADB WIFI merupakan plugin yang dapat menghubungkan device kita melalui wireless untuk dapat melakukan install, run ataupun debug tanpa harus menghubungkan kabel secara terus menerus.

Untuk instalasinya, cukup melalui android studio saja di File -> Settings -> Plugin -> Browse Repositories-> cari 'android wifi adb' -> pilih plugin yang ditemukan -> klik install -> restart android studio.

Jika instalasi berhasil, maka akan muncul tombol ![tombol wifi adb](https://github.com/pedrovgs/AndroidWiFiADB/raw/master/art/sampleButton.png) pada toolbar android studio.

Untuk menggunakannya, silahkan ikuti langkah - langkah ini:

- pastikan devices dan Laptop/PC yang kita gunakan berada dalam 1 jaringan WIFI.
- Hubungkan kabel data dari devices ke PC/Laptop dalam mode debug.
- Klik Icon ![tombol wifi adb](https://github.com/pedrovgs/AndroidWiFiADB/raw/master/art/sampleButton.png)
- Jika muncul pesan bahwa koneksi berhasil, cabut kabel data. Enjoy!

### ADB Idea

![popup adb idea](https://github.com/pbreault/adb-idea/raw/master/website/adb_operations_popup.png)

ADB Idea merupakan plugin yang dapat mempercepat dan mempermudah kita dalam mengakses perintah - perintah ADB yang biasa dilakukan melalui terminal. Beberapa perintah yang dapat dilakukan diantaranya:

- Uninstall Aplikasi
- Kill Aplikasi
- Start Aplikasi
- Restart Aplikadi
- Clear Data Aplikasi
- Clear Data Aplikasi dan Restart

Untuk instalasinya, cukup melalui android studio saja di File -> Settings -> Plugin -> Browse Repositories-> cari 'adb idea' -> pilih plugin yang ditemukan -> klik install -> restart android studio.

Untuk menggunakannya kita dapat menggunakan kombinasi tombol berikut untuk memunculkan popup menu perintah ADB Idea:

- Mac OSX: `Ctrl+Shift+A`
- Windows/Linux: `Ctrl+Alt+Shift+A`

Itulah dua plugin yang selalu saya gunakan. Jika kalian punya refresnsi plugin yang lain, jangan ragu - ragu untuk berkomentar di komentar. 



[pluginADBWIFI]: https://plugins.jetbrains.com/plugin/7983
[pluginADBIdea]: https://plugins.jetbrains.com/plugin/7380?pr=idea