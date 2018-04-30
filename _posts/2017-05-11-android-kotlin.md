---
layout: post
title:  "Memulai Development Aplikasi Android Dengan Kotlin"
date:   2017-05-11 15:16:05 
categories: [android,kotlin,plugin]
disqus_identifier: 110
comments: true
---

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/kotlinlogo.png)

Seperti yang kita ketahui selama ini, android sangat identik dengan bahasa JAVA. Namun belakangan ini telah muncul bahasa pemrograman baru di kalangan komunitas android, yaitu Kotlin. Jadi seperti apa sebenarnya bahasa Kotlin ini? Apa kelebihannya dibanding dengan JAVA?

<!--more-->

### Apa itu Kotlin?

[Kotlin][kotlin] adalah bahasa pemrograman yang berjalan pada JVM dan juga dapat di *compile* ke JavaScript. Dibuat oleh tim dari para programmer JetBrains yang berlokasi di Rusia. Kotlin dapat berjalan dimanapun seperti layaknya JAVA dan tentu saja dapat digunakan untuk mengembangkan aplikasi android.

### Apa saja keunggulan Kotlin?

Seperti yang dijelaskan pada websitenya, kotlin memberikan beberapa keunggulan utama, diantaranya:

- Mengurangi *boilerplate* pada code.
- Mengurangi dan mencegah error pada kelas-kelas seperti `NullPointerException`
- Dapat digunakan untuk membuat aplikasi *server-side*, aplikasi android, maupun *frontend* yang dapat berjalan di browser
- 100% interoperabilitas dengan JAVA
- Sudah mendukung tool seperti *Command Line Compiler* dan IDE 

#### Membuat project

Langkah pertama yang perlu kita lakukan adalah membuat project android seperti biasa pada Android Studio. Pada contoh ini saya memberi nama project dengan nama AndroidKotlin.

Selanjutnya kita perlu menginstall plugin yang diperlukan, melalui `File -> Settings -> Plugins -> Browse Repositories` cari dengan keyword "Kotlin". Install plugin lalu restart Android Studio.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_051117_040505_PM.jpg)

Setelah berhasil menginstall plugin, selanjutnya kita set konfigurasi kotlin pada project. Caranya pilih menu `Tools -> Kotlin -> Configure Kotlin in Project`.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_051117_042316_PM.jpg)

Pilih android with gradle.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_051117_042423_PM.jpg)

Pilih all module lalu klik OK.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_051117_042531_PM.jpg)

Selanjutnya file `build.gradle` pada level project dan module akan mengalami update seperti code dibawah dan tentu saja kita perlu melakukan *sync* project.

{% highlight Groovy %}
// perubahan pada file build.gradle pada level project
buildscript {
    ext.kotlin_version = '1.1.2-3'
    ...
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.1'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

// perubahan pada file build.gradle pada level module
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'

android {
    ...
}

dependencies {
    ...
    compile "org.jetbrains.kotlin:kotlin-stdlib-jre7:$kotlin_version"
}
{% endhighlight %}

Sampai tahap ini, kita sudah berhasil mengkonfigurasi keseluruhan project untuk menggunakan kotlin plugin. Langkah selanjutnya yaitu merubah file ekstensi.java ke ekstensi .kt yang tidak lain adalah format ekstensi untuk kotlin.

### Konversi JAVA ke Kotlin

Pada file `MainActivity.java`, tekan tombol `shift` sebanyak dua kali untuk menampilkan field pencarian, lalu lakukan pencarian dengan keyword "convert java file to kotlin file".

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_051117_045241_PM.jpg)

Selain dengan cara diatas, kita juga bisa menggunakan kombinasi key `Ctrl + Alt + Shift + K` ataupun melalui menu `Code -> Convert Java File to Kotlin File`.

Berikut hasil konversi dari JAVA ke Kotlin pada file `MainActivity`.

{% highlight Kotlin %}
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
{% endhighlight %}

Jika kita ingin membuat kelas dengan ektensi `*.kt` lainnya, kita hanya perlu klik kanan pada package tempat kelas akan dibuat pilih `new -> Kotlin File/Class`.

### Penutup

Seperti yang sudah kita lihat, ternyata cukup mudah bukan untuk konfigurasi menggunakan plugin Kotlin. Buat yang masih bingung juga bisa lihat reponya [disini][repo].

[repo]: https://github.com/dekzitfz/KotlinAndroid
[kotlin]: https://kotlinlang.org/