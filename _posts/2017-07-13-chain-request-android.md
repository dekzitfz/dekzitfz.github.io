---
layout: post
title:  "Chain Network Request Di Android Dengan ReactiveX"
date:   2017-07-13 15:05:58 
categories: [android,rx,retrofit,reactive]
disqus_identifier: 112
comments: true
---

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/rx-chain2.png)

Melakukan *network request* ke suatu API sudah menjadi hal yang umum pada aplikasi android. Aktivitas ini digunakan untuk mengakses data yang tersedia untuk selanjutnya diolah dan ditampilkan pada UI aplikasi. Namun, bagaimana jika parameter yang kita butuhkan untuk melakukan *request*, bergantung pada hasil *request* yang lain?

<!--more-->

### Studi Kasus

Kita ingin mengakses data berupa daftar harga produk dari suatu toko, tapi untuk melakukan *request* tersebut, kita memerlukan parameter ID dari toko tersebut. Disini kita ingin menampilkan toko lengkap dengan daftar harga produknya.

### Chain Request

Dari studi kasus tersebut, kita akan melakukan *request* untuk mengambil ID toko, dan gunakan data tersebut sebagai parameter untuk melakukan request daftar harga produk.

Disini saya menggunakan library [retrofit][retrofit] sebagai *REST client* dan [Gson][gson] sebagai *converter*nya.

{% highlight Groovy %}
compile 'com.squareup.retrofit2:converter-gson:2.3.0'
{% endhighlight %}

Untuk melakukan *chain request*, saya menggunakan library [RxAndroid][rxandroid]. Jangan lupa menambahkan adapternya untuk digunakan pada retrofit.

{% highlight Groovy %}
//rx
compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
compile 'io.reactivex.rxjava2:rxjava:2.1.0'

//rx adapter untuk retrofit
compile 'com.squareup.retrofit2:adapter-rxjava2:2.3.0'
{% endhighlight %}

Buat `instance` retrofit menggunakan *adapter* RxJava.

{% highlight JAVA %}
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl(YOUR_BASE_URL)
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .addConverterFactory(GsonConverterFactory.create())
                .build();
{% endhighlight %}

Berikut `Interface` retrofit yang digunakan untuk menyimpan *endpoint* API. Saya menggunakan `Single`, yaitu semacam `Observable` yang hanya menerima sebuah *value*.

{% highlight JAVA %}
@GET("toko")
Single<Toko> getToko(YOUR_PARAM_HERE);

@GET("harga")
Single<HargaProduk> getHarga(@Query("id") long id);
{% endhighlight %}

Selanjutnya kita lakukan *chain request* menggunakan operator `FlatMap`. Operator `FlatMap` membuat sebuah `Observable` dengan menerapkan fungsi yang kita tentukan untuk setiap item yang dikirim oleh `Observable` lainnya. Untuk lebih jelasnya bisa dilihat [disini][single].

{% highlight JAVA %}
Single<Toko> getTokoID = getRestClient().getToko(YOUR_PARAM_HERE);

Single<HargaProduk> dataHargaProduk = getTokoID.flatMap(new Function<Toko>, SingleSource<? extends HargaProduk>>() {
            @Override
            public SingleSource<? extends HargaProduk> apply(@NonNull Toko toko) throws Exception {
                return getRestClient().getHarga(toko.getID());
            }
        });
{% endhighlight %}

Pada contoh diatas, kita melakukan *request* pertama untuk mendapatkan data toko, kemudian dari data toko yang sudah didapat kita menggunakan ID toko tersebut untuk melakukan *request* kedua untuk mendapatkan data harga produk.

Langkah terakhir adalah melakukan `subscribe` dataHargaProduk.

{% highlight JAVA %}
dataHargaProduk.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new DisposableSingleObserver<HargaProduk>() {
                    @Override
                    public void onSuccess(@NonNull HargaProduk harga) {
                        //olah data
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        //handle error
                    }
                });
{% endhighlight %}

###### Referensi
- http://reactivex.io/documentation/operators/flatmap.html
- http://reactivex.io/documentation/single.html
- https://square.github.io/retrofit/

[single]: http://reactivex.io/documentation/single.html
[retrofit]: https://square.github.io/retrofit/
[rx]: http://reactivex.io/
[gson]: https://github.com/google/gson
[rxandroid]: https://github.com/ReactiveX/RxAndroid