---
layout: post
title:  "Gradle Untuk Dasar Android. Bagian 2"
date:   2016-09-16 14:06:23 
categories: [gradle,android]
disqus_identifier: 103
comments: true
---

![logo gradle](https://technologyconversations.files.wordpress.com/2014/06/gradle.png)

Pada [bagian 1][bagian_1] sudah membahas file *Top-Level* `build.gradle` dan *App-Level* `build.gradle` walau baru sebagian, untuk itu di artikel bagian ke-2 ini kita akan lanjut membahasnya.

<!--more-->

Di *Top-Level* `build.gradle` terdapat block `buildscript` untuk menambahkan plugin - plugin android untuk kemudian di-*apply* pada *App-Level* `build.gradle` yang menambahkan block `android`. Pada block `android` ini kita dapat menspesifikasikan beberapa *properties* dari project kita, seperti contoh dibawah.

{% highlight Groovy %}
apply plugin: 'com.android.application'

android {
    compileSdkVersion 24
    buildToolsVersion "24.0.2"

    defaultConfig {
        applicationId "id.dekz.gradleexample"
        minSdkVersion 21
        targetSdkVersion 24
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
{% endhighlight %}

Block `android` merupakan titik masuk untuk Android DSL(*Domain Spesific Language*). Disini kita harus menentukan target kompilasi dari project kita pada `compileSdkVersion` dan versi dari `build-tools` yang kita gunakan pada `buildToolsVersion`. Kedua variable ini pada umumnya harus menggunakan versi yang paling baru karena biasanya bug - bug dari versi sebelumnya sudah ditambal.

Pada block `defaultConfig` terdapat beberapa variable, berikut penjelasan dari masing - masing variable tersebut:

- **applicationId** ini adalah nama package dari aplikasi kita. Harus bersifat unik saat diupload ke Play Store karena ini akan menjadi identitas dari aplikasi. Nilai dari variable ini sebaiknya jangan diubah, namun jika diubah secara sengaja, maka aplikasi kita akan dikenal sebagai aplikasi baru, walaupun nama aplikasi tidak berubah.

- **minSdkVersion** adalah versi SDK minimal yang didukung oleh aplikasi. Smartphone yang menggunakan versi dibawahnya tidak akan menemukan aplikasi ini di Play Store.

- **targetSdkVersion** adalah versi android yang ditunjukkan untuk aplikasi ini. Kita bebas memilih versi target asalkan tidak dibawah versi minimal.

- **versionCode** adalah variable integer yang merepresentasikan versi aplikasi ini dibandingkan dari versi yang sebelumnya. Kita bisa merubah nilainya saat ingin mengupgrade aplikasi kita.

- **versionName** adalah variable String yang menunjukkan versi rilis dari aplikasi yang dapat dilihat oleh pengguna. Pada umumnya menggunakan format: `<major>.<minor>.<version>`

Block `buildTypes` digunakan untuk mengkonfigurasi versi Build dari aplikasi kita, baik itu versi *debug*, versi *release*, ataupun kita ingin membuat kustomisasi versi Build sendiri. Versi Build menentukan bagaimana nantinya aplikasi akan di-*compile*.

Secara default, versi build yang ada pada gradle hanya ada untuk versi *release*. Kita dapat menambahkan versi lainnya pada block ini, misalnya versi *debug* dan lain - lain. Tiap block mendukung berbagai macam *properties*, detail *properties* dan methodnya dapat dilihat [disini][DSL_ref].

Pada contoh di block versi *release*, terdapat variable `minifyEnabled` yang jika bernilai *true*, maka gradle akan membuat aplikasi kita aman dari *reverse engineering* sekaligus mengecilkan ukuran dari aplikasi kita nantinya menggunakan ProGuard. Gradle juga dapat mengecilkan *resource* dari *library* yang kita gunakan jika kita menambahkan variable `shrinkResources` yang bernilai *true*.

> Catatan: Android Studio akan menonaktifkan ProGuard jika kita menggunakan Instant Run.

Variable `proguardFiles` mendefinisikan rule untuk menggunakan ProGuard:

- `getDefaultProguardFile('proguard-android.txt')` Method ini digunakan untuk mendapatkan file konfigurasi default dari SDK Android di folder `tools/proguard/`.

- `proguard-rules.pro` Adalah file tempat kita menerapkan beberapa rule untuk penggunaan ProGuard. Secara default, filenya terletak di folder `App`.

Stay Tuned terus ya, bagian 3 akan menyusul! Contoh kode yang digunakan bisa dilihat di repo [GradleExample][GradleExample].


###### Referensi
Gradle Recipes For Android by Ken Kousen

[bagian_1]: https://dekzitfz.github.io/articles/2016-09/gradle-untuk-dasar-android
[DSL_ref]: http://google.github.io/android-gradle-dsl/current/index.html
[GradleExample]: https://github.com/dekzitfz/GradleExample