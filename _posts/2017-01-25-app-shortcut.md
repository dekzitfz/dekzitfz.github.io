---
layout: post
title:  "App Shortcut Pada Android 7.1"
date:   2017-01-25 11:25:15 
categories: [android,nougat,shortcut]
disqus_identifier: 108
comments: true
---

<img src="https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_050218_012730_PM.jpg" width="600">

Android Nougat 7.1 (API 25) membawa beberapa fitur baru seperti yang sudah saya buat pada artikel [sebelumnya][sebelumnya]. Salah satu fitur tersebut adalah App Shortcut.

<!--more-->

App Shortcut adalah sebuah jalan pintas untuk mengakses fitur atau aktivitas tertentu pada aplikasi android kita. App Shortcut ini hanya bisa diakses jika kita menggunakan Android 7.1 dan menggunakan *launcher* dari google maupun pixel. Untuk mengakses jalan pintas ini kita cukup melakukan *tap and hold* pada ikon aplikasi.

Jalan pintas yang ditampilkan terbagi menjadi dua jenis, yaitu statis dan dinamis. Urutan tampilan jalan pintas pun bisa kita urutkan sesuai kebutuhan.

### Mengapa Menggunakan App Shortcut?

Fungsi utama dari fitur ini yaitu mempermudah pengguna aplikasi untuk menjalankan ataupun mengakses aktivitas maupun fitur umum yang biasa dilakukan pada aplikasi kita. Misalnya pada aplikasi chat, maka kita bisa membuat jalan pintas untuk  langsung melakukan chat kepada orang-orang tertentu yang sering pengguna kita ajak mengobrol.

### Static App Shortcut

Jalan pintas statis, seperti namanya, jalan pintas ini mempunyai atribut yang selalu sama di tiap versi dari aplikasi kita. Buat sebuah project dengan sebuah `activity`, lalu tambahkan `meta-data` ini pada file `AndroidManifest.xml` di dalam tag `activity`.

{% highlight XML %}
<meta-data
android:name="android.app.shortcuts"
android:resource="@xml/shortcuts" />
{% endhighlight %}

pada contoh ini nama `activity` adalah `MainActivity`. Sehingga tampilan  di dalam tag `activity` seperti dibawah ini.

{% highlight XML %}
<!-- sisa kode tidak ditampilkan -->
<activity android:name=".MainActivity">
	<intent-filter>
		<action android:name="android.intent.action.MAIN" />
		<category android:name="android.intent.category.LAUNCHER" />
	</intent-filter>
	<meta-data
		android:name="android.app.shortcuts"
		android:resource="@xml/shortcuts" />
</activity>
<!-- sisa kode tidak ditampilkan -->
{% endhighlight %}

Pada tag `meta-data` kita mendefinisikan `android:resource` yang akan mengarah ke folder `res/xml/shortcuts.xml`. Pada file inilah kita menyimpan atribut static shortcut. Contoh isi filenya seperti dibawah ini.

{% highlight XML %}
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
    <shortcut
        android:enabled="true"
        android:icon="@drawable/ic_today"
        android:shortcutDisabledMessage="@string/disabled_message"
        android:shortcutId="shortcut1"
        android:shortcutLongLabel="@string/long_label"
        android:shortcutShortLabel="@string/short_label">
        <intent
            android:action="android.intent.action.VIEW"
            android:targetClass="it.dekz.appshortcut.Activity2"
            android:targetPackage="it.dekz.appshortcut" />
    </shortcut>
</shortcuts>
{% endhighlight %}

Terlihat isi dari file `shortcuts.xml` terdapat tag `shortcuts` yang dapat menyimpan beberapa Static Shortcut pada tag `shortcut`. Berikut penjelasan dari beberapa atribut shortcut:

- `enabled` menandakan apakah shortcut ini aktif(dapat terlihat) atau tidak.
- `icon` untuk menetapkan ikon yang kita gunakan untuk shortcut tersebut.
- `shortcutDisabledMessage` adalah pesan yang ditampilkan jika shortcut berstatus *disabled*, namun pengguna sudah melakukan pin terhadap shortcut tersebut, sehingga ikon shortcut akan berwarna abu-abu dan pengguna melakukan klik ke ikon tersebut.
- `shortcutId` merupakan ID atau pengenal.
- `shortcutLongLabel` label shortcut yang ditampilkan bila *space* mencukupi.
- `shortcutShortLabel` label shortcut *default*.
- `intent` disini kita menambahkan *action* dari shortcut. Pada contoh ini shortcut akan membuka sebuah `activity` yang berlokasi di `it.dekz.appshortcut.Activity2`.

Silahkan buat `Activity2` beserta layout dan data di `AndroidManifest.xml` untuk melakukan tes terhadap Static Shortcut yang sudah dibuat.

![](http://i.giphy.com/PpYLEOdmEa4Gk.gif)

Jika kita perhatikan, saat kita menekan tombol back, aplikasi justru kembali ke home screen, untuk kita diperlukan *back stack intent* untuk mendefinisikan `MainActivity` sebagai *root* `activity`. Contoh penambahannya seperti dibawah ini.

{% highlight XML %}
<shortcut
        android:enabled="true"
        android:icon="@drawable/ic_today"
        android:shortcutDisabledMessage="@string/disabled_message"
        android:shortcutId="shortcut1"
        android:shortcutLongLabel="@string/long_label"
        android:shortcutShortLabel="@string/short_label">
	<intent
		android:action="android.intent.action.MAIN"
		android:targetClass="it.dekz.appshortcut.MainActivity"
		android:targetPackage="it.dekz.appshortcut"/>
	<intent
		android:action="android.intent.action.VIEW"
		android:targetClass="it.dekz.appshortcut.Activity2"
		android:targetPackage="it.dekz.appshortcut" />
</shortcut>
{% endhighlight %}

Sehingga urutan `activity` menjadi `MainActivity` -> `Activity2` dan saat kita menekan tombol back aplikasi akan menuju ke `MainActivity`.

![](http://i.giphy.com/bHevdDI2lxopa.gif)

### Dynamic App Shortcut

Jalan pintas dinamis, dapat dibuat dan diupdate saat aplikasi sedang berjalan (*runtime*) tanpa harus melakukan deploy aplikasi berulang-ulang. Shortcut ini tidak dibuat dalam file `shortcuts.xml` seperti Static App Shortcut, melainkan pada koding aplikasi.

Untuk membuat jalan pintas dinamis, kita menggunakan `ShortcutManager` API, contoh penggunaannya pada `MainActivity` seperti code dibawah ini.

{% highlight JAVA %}
//buat list untuk menampung shortcut
List<ShortcutInfo> shortcutInfos = new ArrayList<>();

//cek jika os menggunakan API 25
if (android.os.Build.VERSION.SDK_INT >= 25) {
	//inisiasi ShortcutManager
	ShortcutManager shortcutManager = getSystemService(ShortcutManager.class);

	//membuat object shortcutInfo
	ShortcutInfo shortcutInfo = new ShortcutInfo.Builder(this, "web_shortcut")
		.setShortLabel("dekzitfz.github.io")
		.setLongLabel("Visit dekzitfz github pages")
		.setIcon(Icon.createWithResource(this, R.drawable.ic_web))
		.setIntent(new Intent(Intent.ACTION_VIEW, Uri.parse("https://dekzitfz.github.io/")))
		.build();

	//tambahkan objeck ke dalam list
	shortcutInfos.add(shortcutInfo);
	
	//set shortcut dinamis
	shortcutManager.setDynamicShortcuts(shortcutInfos);
}
{% endhighlight %}

Disini kita menggunakan `ShortcutManager` untuk membuat object bertipe `SHortcutInfo`. Dengan menggunakan `ShortcutInfo.Builder` kita dapat menerapkan atribut-atribut *shortcut* seperti yang kita lakukan sebelumnya pada Jalan Pintas Statis. Hasil dari kode diatas akan terlihat seperti dibawah ini.

![](http://i.giphy.com/tSW98Qr9Xy4Uw.gif)

Jika kita ingin menambah *bac stack intent* seperti pada Jalan Pintas Statis, kita bisa menambahkan `intent` menggunakan `setIntents()`. Contohnya kodenya seperti dibawah ini.

{% highlight JAVA %}
ShortcutInfo dynamicShortcut = new ShortcutInfo.Builder(this, "shortcut_dinamis")
                    .setShortLabel("Dinamis")
                    .setLongLabel("buka shortcut dinamis")
                    .setIcon(Icon.createWithResource(this, R.drawable.ic_today))
                    .setIntents(
                            new Intent[]{
                                    new Intent(Intent.ACTION_MAIN, Uri.EMPTY, this, MainActivity.class).setFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK),
                                    new Intent(Intent.ACTION_VIEW, Uri.EMPTY, this, Activity3.class)
                            })
                    .build();
{% endhighlight %}

Tentunya jangan lupa untuk membuat *class* dan *layout* untuk `Activity3`. Untuk data pada `AndroidManifest.xml` tambahkan `intent-filter` seperti dibawah ini.

{% highlight XML %}
<activity
	android:name=".Activity3"
	android:label="Dynamic shortcut activity">
	<intent-filter>
		<action android:name="it.dekz.appshortcut.OPEN_DYNAMIC_SHORTCUT" />
		<category android:name="android.intent.category.DEFAULT" />
	</intent-filter>
</activity>
{% endhighlight %}

![](http://i.giphy.com/tv8chky4Go4cE.gif)

### Mengurutkan Shortcut

Urutan shortcut yang tampil bisa kita urutkan sesuai kebutuhan dengan menggunakan `setRank(int rank)` pada `ShortcutInfo.Builder`. Semakin tinggi nilai *rank*, semakin diatas posisi shortcut tersebut.

Bagi yang ingin melihat contoh dari penerapan App Shortcut ini bisa cek ke repo [AppShortcut di Github][github].

Referensi:

- https://developer.android.com/guide/topics/ui/shortcuts.html

[sebelumnya]: https://dekzitfz.github.io/articles/2016-10/sekilas-android-7-1
[github]: https://github.com/dekzitfz/AppShortcut