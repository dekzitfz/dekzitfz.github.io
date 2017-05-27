---
layout: post
title:  "Menangani Perubahan Konfigurasi Dengan ViewModel"
date:   2017-05-27 21:08:41
categories: [android,lifecycle,arch]
disqus_identifier: 111
comments: true
---

![](https://s23.postimg.org/cuw1hnv63/Screenshot_052717_090000_PM.jpg/)

Pada Google I/O 2017 beberapa hari yang lalu, Google memperkenalkan beberapa komponen yang dapat digunakan untuk mempermudah developer dalam membangun aplikasi android. Salah satu komponen tersebut adalah `ViewModel`.

<!--more-->

### Apa itu ViewModel?

[ViewModel][ViewModel] adalah sebuah kelas yang dirancang untuk menyimpan dan mengelola data yang biasanya berhubungan dengan UI. Sehingga data tersebut dapat digunakan kembali saat terjadi perubahan konfigurasi.

### Apa fungsi dari ViewModel?

Terkadang, terjadi beberapa perubahan konfigurasi pada *device* yang kita gunakan, entah itu saat rotasi layar, munculnya virtual keyboard, dan lain-lain. Saat perubahan itu terjadi, android akan melakukan *restart* terhadap `activity` yang sedang berjalan.

Contoh sederhananya, saat aplikasi kita sedang melakukan *request* data ke server ketika `activity` diakses, ketika data sudah ditampilkan, tiba-tiba *user* melakukan rotasi layar, maka `activity` akan melakukan *restart* dan melakukan *request* data dari awal, yang seharusnya tidak perlu dilakukan lagi.

Untuk itulah `ViewModel` dibuat, `ViewModel` dapat menyimpan dan mengembalikan data yang terikat dengan suatu `activity` maupun `fragment` sehingga aplikasi kita dapat menggunakan data yang sebelumnya sudah dimiliki.

### Menggunakan ViewModel

> **Catatan:** saat artikel ini dibuat, versi komponen saat ini masih berstatus alpha.

Disini saya akan membuat contoh sederhana menggunakan `ViewModel`, yaitu aplikasi *timer*, dimana nantinya *timer* harus tetap berjalan (tidak mengulang) walaupun terjadi rotasi layar.

Langkah pertama, tentu saja kita membuat sebuah project dengan android studio seperti biasa. Lalu tambahkan *dependencies* untuk menggunakan komponen `ViewModel`.

{% highlight Groovy %}
// build.gradle pada level project
...
allprojects {
    repositories {
        jcenter()
        maven { url 'https://maven.google.com' }
    }
}
...
{% endhighlight %}

{% highlight Groovy %}
// build.gradle pada level module
...
dependencies {
    ...

    compile "android.arch.lifecycle:runtime:1.0.0-alpha1"
    compile "android.arch.lifecycle:extensions:1.0.0-alpha1"
    annotationProcessor "android.arch.lifecycle:compiler:1.0.0-alpha1"
}
{% endhighlight %}

Tambahkan widget `Chronometer` pada `activity_main.xml` dan jangan lupa definisikan juga pada file `MainActivity.java`.

{% highlight XML %}
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="id.dekz.viewmodelexample.MainActivity">

    <Chronometer
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:id="@+id/chronometer"
        android:layout_marginTop="8dp"
        android:layout_marginRight="8dp"
        app:layout_constraintRight_toRightOf="parent"
        android:layout_marginLeft="8dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        android:layout_marginBottom="8dp" />

</android.support.constraint.ConstraintLayout>
{% endhighlight %}

{% highlight JAVA %}
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...

        Chronometer chronometer = (Chronometer) findViewById(R.id.chronometer);
        chronometer.start();
    }
}
{% endhighlight %}

Coba jalankan program diatas, lalu lakukan rotasi layar. *Timer* akan mengulang dari awal setiap rotasi layar dilakukan karena `activity` melakukan *restart* saat terjadi perubahan konfigurasi.

Untuk menyimpan data dari *timer* tersebut, buat sebuah kelas `ViewModel`, contohnya `TimerViewModel.java`.

{% highlight JAVA %}
public class TimerViewModel extends ViewModel {

    private Long start;

    public Long getStart() {
        return start;
    }

    public void setStart(Long start) {
        this.start = start;
    }
}
{% endhighlight %}

Lakukan perubahan pada `MainActivity.java`.

{% highlight JAVA %}
public class MainActivity extends LifecycleActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...

        //init ViewModel
        TimerViewModel viewModel = ViewModelProviders.of(this)
                .get(TimerViewModel.class);

        Chronometer chronometer = (Chronometer) findViewById(R.id.chronometer);

        if(viewModel.getStart() == null){
            // jika start pada viewmodel belum diset
            Long start = SystemClock.elapsedRealtime();
            viewModel.setStart(start);
            chronometer.setBase(start);
        }else{
            //jika start menyimpan value, set ke chronometer
            Log.d("start",""+viewModel.getStart());
            chronometer.setBase(viewModel.getStart());
        }

        chronometer.start();
    }
}
{% endhighlight %}

Pada kode diatas, kita mengganti `AppCompatActivity` menjadi [LifecycleActivity][LifecycleActivity] yang nantinya digunakan saat deklarasi `TimerViewModel`. 

{% highlight JAVA %}
TimerViewModel viewModel = ViewModelProviders.of(this)
                .get(TimerViewModel.class);      
{% endhighlight %}

`this` mengarah kepada *instance* dari `LifecycleOwner` karena kita sudah menggunakan `LifecycleActivity`. `ViewModel` masih dapat bertahan walaupun `activity` melakukan *restart* (`onDestroy` lalu `onCreate` kembali) saat terjadi perubahan konfigurasi. Gambaran *Lifecycle* dari `ViewModel` dapat dilihat seperti gambar dibawah ini.

![](https://developer.android.com/images/topic/libraries/architecture/viewmodel-lifecycle.png)

Sekarang coba jalankan kembali aplikasinya, dan coba lakukan rotasi layar, maupun beralih sementara ke aplikasi lain lalu kembali ke aplikasi ini. Untuk detail code bisa cek reponya [disini][repo].

###### Referensi

- https://codelabs.developers.google.com/io2017
- https://developer.android.com/topic/libraries/architecture/index.html


[repo]: https://github.com/dekzitfz/ViewModelExample
[LifecycleActivity]: https://developer.android.com/reference/android/arch/lifecycle/LifecycleActivity.html
[ViewModel]: https://developer.android.com/topic/libraries/architecture/viewmodel.html