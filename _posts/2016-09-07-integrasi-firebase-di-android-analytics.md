---
layout: post
title:  "Integrasi Firebase Di Android: Analytics"
date:   2016-09-08 10:05:37
categories: [firebase,analytics,android]
disqus_identifier: 102
comments: true
---

![Logo Firebase](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/firebase2.png "Logo Firebase")

Firebase merupakan platform yang menyediakan berbagai layanan layaknya sebagai Back-End aplikasi kita. Pada postingan ini saya akan membahas bagaimana cara mengintegrasikan salah satu layanan dari Firebase, yaitu Firebase Analytics.

<!--more-->

Firebase Analytics merekam kegiatan dari aplikasi kita saat digunakan oleh pengguna. Ada 2 macam informasi yang direkam:

- **Events**: Apa yang terjadi pada aplikasi, seperti apa yang di-klik oleh pengguna, terjadinya error, dan lain-lain
- **User Properties**: Atribut yang kita gunakan untuk mendeskripsikan basis penngguna aplikasi kita, seperti setelan bahasa maupun lokasi.

Pada dasarnya, Firebase Analytics secara otomatis merekam beberapa [Event][event] dan [User Properties][user_properties], sehingga kita tidak perlu kode tambahan untuk melakukannya.

##### 1. Setup Firebase SDK

Sebelum mulai, harap pastikan beberapa hal berikut:

- Pastikan minimum API untuk app yang dibuat adalah 2.3(Gingerbread) keatas dan juga menggunakan Google Play Services versi 9.4.0 keatas.
- Sudah menginstall Google Repository dari [SDK Manager][SDK_manager]
- Android Studio versi 1.5 keatas

Setelah semua sudah sesuai, langkah pertama, kita tambahkan Firebase ke dalam project kita:

- Buat project di Android Studio seperti biasa.
- Buat Project baru di [Firebase Console][firebase_console]. Set nama project dan  negara.

  ![buat project firebase](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_090816_031142_PM.jpg "buat project firebase")

- pilih **Add Firebase to your Android app**, masukkan package name dari aplikasi kita & SHA-1. untuk mendapatkan SHA-1 bisa lihat [disini][SHA1].

  ![add package & sha1](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_090816_032218_PM.jpg "add package & sha1")

- Setelah itu kita akan otomatis mengunduh `google-services.json`.Taruh file tersebut ke dalam folder `app` di project android kita.

  ![unduh json](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_090816_032443_PM.jpg "unduh json")

- Tambahkan dependency firebase ke project kita. Pada Project-level `build.gradle`:

{% highlight Groovy %}
buildscript {
  dependencies {
    // tambahkan baris ini
    classpath 'com.google.gms:google-services:3.0.0'
  }
}
{% endhighlight %}

- tambahkan plugin pada App-Level `build.gradle`

{% highlight Groovy %}
// taruh di baris paling bawah
apply plugin: 'com.google.gms.google-services'
{% endhighlight %}

- Klik Finish pada wizard Firebase, lalu sync Gradle di Android Studio untuk mendownload plugin yang sudah kita tambahkan.


##### 2. Setup Firebase Analytics

Pada langkah sebelumnya kita sudah menginstall Firebase SDK ke aplikasi kita, sekarang kita akan setup untuk dapat menggunakan Firebase Analytics.

- Pada App-Level `build.gradle`, tambahkan dependency dari Firebase Analytics. Untuk list dependency lainnya dari firebase bisa dilihat [disini][list_firebase_dependency].

{% highlight Groovy %}
compile 'com.google.firebase:firebase-core:9.2.0'
{% endhighlight %}

- Pada `Application Class` dari aplikasi kita, deklarasikan object bertipe `FirebaseAnalytics`.

{% highlight Java %}
private FirebaseAnalytics mFirebaseAnalytics;
{% endhighlight %}

- Inisialisasikan object tersebut pada method `onCreate` di `Application Class`

{% highlight Java %}
mFirebaseAnalytics = FirebaseAnalytics.getInstance(this);
{% endhighlight %}

- Berikut tampilan kode langkap dari `Application Class`.

{% highlight Java %}
public class FirebaseExample extends Application {
    private FirebaseAnalytics mFirebaseAnalytics;

    @Override
    public void onCreate(){
        super.onCreate();
        mFirebaseAnalytics = FirebaseAnalytics.getInstance(this);
    }
}
{% endhighlight %}

##### Done!

Dengan ini kita sudah mengintegrasikan Firebase Analytics pada aplikasi android kita. Kita bisa melihat grafik penggunaan aplikasi kita pada sesi **Analytics** di console Firebase. Perlu diperhatikan, butuh sekitar 1x24 jam untuk grafik dapat aktif sejak kita mengintegrasikannya, sehingga saat kita membukanya akan terlihat seperti tampilan dibawah.

![view analytics](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_090916_094703_AM.jpg "view analytics")

Data grafik yang sudah muncul akan terlihat seperti ini.

![graph analytics](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_090916_095734_AM.jpgg "graph analytics")

Untuk lebih jelasnya, silahkan lihat repo [FirebaseExample][FirebaseExample] di GitHub.


[event]: https://support.google.com/firebase/answer/6317485
[user_properties]: https://support.google.com/firebase/answer/6317486
[SDK_manager]: https://developer.android.com/studio/intro/update.html
[firebase_console]: https://console.firebase.google.com/
[SHA1]: https://developers.google.com/android/guides/client-auth
[list_firebase_dependency]: https://firebase.google.com/docs/android/setup#available_libraries
[FirebaseExample]: https://github.com/dekzitfz/FirebaseExample