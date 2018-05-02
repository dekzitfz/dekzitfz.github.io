---
layout: post
title:  "Format Gambar WebP Di Android"
date:   2016-12-01 10:21:17
categories: [android,image,webp,tips]
disqus_identifier: 107
comments: true
---


![logo webP](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/webplogo.png)

WebP (dibaca: "weppy") merupakan format file gambar yang menggunakan kompresi lossy dan lossless. Saat ini tengah dikembangkan oleh Google dan sudah didukung oleh beberapa browser seperti Google Chrome dan Opera. Google sendiri membuat format ini sebagai alternatif dari JPEG dan PNG yang semakin menua, untuk itu WebP ini dijanjikan akan mempunyai keunggulan utama, diantaranya yaitu ukuran yang lebih kecil dengan kualitas yang tidak berkurang.

<!--more-->

### Pro dan Kontra

Karena format ini masih tergolong baru, ada beberapa pro dan kontra yang perlu diperhatikan jika ingin menggunakan format ini.

##### Pro
- Ukuran lebih kecil.
- Kualitas tetap, bahkan bisa meningkat.
- satu format WebP bisa menggantikan format JPEG maupun PNG.

##### Kontra
- Belum didukung semua browser.
- Belum didukung secara native oleh sistem operasi manapun, alternatif bisa menggunakan plugin.

Terlepas dari pro dan kontra diatas, bagi saya WebP bisa menjadi pilihan yang bijak jika kita ingin mengurangi ukuran aplikasi android kita sebanyak 20% - 30%. Di android sendiri sudah mendukung format WebP dari versi 4.0 (ICS) keatas (Loosless & Transparance dari versi 4.2.1).

> WebP tidak bisa digunakan sebagai launcher icons

### Konversi ke WebP

Untuk melakukan konfersi dari JPEG ataupun PNG ke WebP, kita bisa menggunakan tool yang sudah disediakan oleh Google, yaitu [WebP Library][cwebp]. Silahkan download  dan install sesuai dengan OS, lalu ikuti langkah2 berikut:

- jalankan command `cwebp -q 100 img\android.png -o img\android.webp`. untuk dokumentasi bisa dilihat [disini][cwebp-doc].
- contoh result nya :

![result cwebp](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_120116_101803_AM.jpg)

- hasil konversinya:

![hasil](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_113016_094353_PM.jpg)

Terlihat hasil dari konversi dengan ukuran 50% lebih kecil, dengan format baru, yaitu `*.webp`. Bayangkan jika semua resource gambar kita menggunakan WebP, ukuran file APK kita tentu bisa menurun secara signifikan.

### Apply ke Android

Untuk menggunakan WebP di android, langkahnya tetap sama seperti kita menggunakan file JPEG ataupun PNG. Bisa di XML dengan `app:srcCompat="@drawable/android"`, atau di JAVA dengan `imageView.setImageResource(R.drawable.android);`.

[cwebp]: https://developers.google.com/speed/webp/docs/precompiled
[cwebp-doc]: https://developers.google.com/speed/webp/docs/cwebp