---
layout: post
title:  "Gradle Untuk Dasar Android. Bagian 1"
date:   2016-09-04 13:48:22
categories: [gradle,android]
disqus_identifier: 101
comments: true
---

Pernahkah kalian mendengar istilah tentang Gradle? Bagi android developer, istilah ini tentu tidak asing, bahkan selalu ditemui saat mengembangkan suatu aplikasi android. Jadi, apa sebenarnya Gradle itu?

<!--more-->

Aplikasi android dibangun menggunakan sistem Gradle Build dengan kode sumber terbuka. Setiap kita melakukan (re)build, ataupun mendeploy app kita ke emulator ataupun devices, Android Studio akan melakukan built terhadap source code kita berdasarkan konfigurasi yang ada pada file Gradle.

Supaya lebih jelas, kita mulai dari saat pertama kali membuat project di Android Studio.

![Membuat Project Baru](https://s16.postimg.org/77094q1p1/create_project.jpg "Membuat Project Baru")

Setelah membuat project baru, akan terbentuk tiga file utama Gradle yang dapat dilihat di panel kiri Android Studio.

![File Gradle yang terbentuk](https://s11.postimg.org/am91ayjw3/file_gradle.jpg "File Gradle yang terbentuk")


#### Gradle Setting
Sebuah project merupakan kumpulan dari beberapa sub-projects yang dikonfigurasi pada Gradle. Daftar sub-projects dapat dilihat pada file `settings.gradle` .

{% highlight Groovy %}
include ':app'
{% endhighlight %}

Syntax `include` menunjukan bahwa folder app merupakan satu-satu nya sub-project yang terdaftar disini, jika kita ingin menambahkan library project kita juga harus menambahkannya ke dalam file `settings.gradle` ini.


#### Top-Level Gradle Build
{% highlight Groovy %}
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.3'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
{% endhighlight %}

Kode pada block `buildscript` menunjukkan dimana Gradle harus mengunduh plugin yang diperlukan. Seperti yang terlihat diatas, secara default plugin akan diunduh dari repo `jcenter`, repo lain yang juga bisa digunakan yaitu `mavenCentral`.

Kode pada block `allprojects` menunjukkan bahwa top-level project dan semua sub-projects yang terdaftar secara default akan mengunduh plugin/library dependencies dari repo `jcenter`.

Kita dapat membuat task di Gradle sesuai kebutuhan, pada file `build.gradle` ini terdapat sebuah clean task dengan perintah `delete`. Pada kasus ini, task ini akan menghapus folder `build` dari root project kita, dimana secara default itu adalah folder `build` di top-level.


#### App Sub-Project Gradle Build
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

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:24.2.0'
}
{% endhighlight %}

Fungsi `apply` digunakan untuk menambahkan plugin Android ke dalam build system.

Pada block `dependencies` terdapat 3 baris kode. Baris pertama, filreTree dependency, menunjukkan bahwa semua file dengan ekstensi `*.jar` didalam folder `libs` akan di-compile.

Baris kedua memberitahu Gradle untuk mengunduh `JUnit` versi 4.12, fungsi `testCompile` akan menempatkan kelas-kelas dari `JUnit` di `src/androidTest` dan `src/test/java` (opsional) dan kita bisa menggunakannya untuk melakukan unit test.

Baris ketiga memberitahu Gradle untuk menambahkan library appcompat-v7 versi 24.2.0 dari Android Support Libraries. Library pendukung disini digunakan sebagai dependency saat melakukan compile.

Cukup sampai sini dulu untuk bagian 1, post ini akan dilanjutkan ke bagian 2. Stay Tuned!

###### Referensi
Gradle Recipes For Android by Ken Kousen