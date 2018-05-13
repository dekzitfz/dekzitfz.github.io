---
layout: post
title:  "Tensor Flow Lite Android"
date:   2018-05-13 12:56:49 
categories: [android,tensorflow,machinelearning]
disqus_identifier: 113
comments: true
---

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/tflite.png)

Machine learning adalah cabang aplikasi dari Artificial Intelligence (Kecerdasan Buatan) yang fokus pada pengembangan sebuah sistem yang mampu belajar "sendiri" tanpa harus berulang kali di program oleh manusia. Tensor Flow merupakan salah satu library open source yang dapat kita gunakan untuk melakukan eksperimen dengan Machine Learning.

<!--more-->

### Tensor Flow Lite

Tensor Flow Lite adalah library machine learning yang dirancang khusus untuk perangkat mobile. Ini memungkinkan mesin untuk "belajar" di perangkat dengan latensi rendah dan ukuran binary yang kecil. Lebih detail tentang Tensor Flow Lite dapat dilihat [disini][tflite].

### Object Detection Di Android

Salah satu contoh sederhana dalam studi kasus Machine Learning, yaitu pengenalan suatu objek. Kita akan coba membuat sebuah aplikasi android yang akan mengambil gambar dari sebuah benda dan mencoba menebak benda apakah yang ada dalam gambar tersebut. Ini mungkin bisa dibilang aplikasi Tebak Gambar (?).

Salah satu syarat untuk menggunakan Tensor Flow Lite ini adalah kita harus menyediakan Model dari Tensor Flow Lite yang sudah terlatih, maksudnya terlatih disini artinya Model tersebut sudah belajar dari beberapa contoh kasus yang diberikan kepadanya. Disini kita akan menggunakan Model dari Mobilenet yang bisa kalian [download][mobilenet]. Kalian juga bisa mencoba model lainnya pada [daftar model][listmodels].

Setelah mendownload modelnya, akan ada dua buah file, yaitu file `labels.txt` yang berisi daftar objek yang sudah dipelajari oleh model tersebut, dan file `mobilenet_quant_v1_224.tflite` yang merupakan model tensor flow lite yang sudah terlatih.

Buat project baru di Android Studio, kemudian tambahkan library Tensor Flow Lite.

{% highlight Groovy %}
implementation 'org.tensorflow:tensorflow-lite:+'
{% endhighlight %}

Karena kita akan menggunakan kamera, untuk mempermudah dan mempersingkat waktu setup kita akan menggunakan library [Camera Kit][camerakit]. Silahkan baca sekilas [dokumentasinya][camerakitdoc] terlebih dahulu.

Buat folder `assets` di `src/main`, taruh file `labels.txt` dan `mobilenet_quant_v1_224.tflite` di dalamnya. Untuk saat ini kita akan mengakses model tensor flow dari folder ini.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_050918_044305_PM.jpg)

Sekarang kita memerlukan sebuah class yang akan membantu kita untuk mengenal objek yang telah dicapture nanti, kita beri nama class tersebut dengan `TensorFlowImageClassifier`, class ini akan mengadopsi sifat - sifat dari interface `Classifier` yang berisi fungsi untuk membantu kita melakukan parsing hasil dari pengenalan objek.

interface `Classifier` :

{% highlight JAVA %}
public interface Classifier {
    class Recognition {

        private final String id;
        private final String title;
        private final Float confidence;

        public Recognition(final String id, final String title, final Float confidence) {
            this.id = id;
            this.title = title;
            this.confidence = confidence;
        }

        public String getId() {
            return id;
        }

        public String getTitle() {
            return title;
        }

        public Float getConfidence() {
            return confidence;
        }

        @Override
        public String toString() {
            String resultString = "";
            if (id != null) {
                resultString += "[" + id + "] ";
            }

            if (title != null) {
                resultString += title + " ";
            }

            if (confidence != null) {
                resultString += String.format(
                        Locale.getDefault(),"(%.1f%%) ", confidence * 100.0f);
            }

            return resultString.trim();
        }
    }

    List<Recognition> recognizeImage(Bitmap bitmap);

    void close();
}
{% endhighlight %}

Pada class [`TensorFlowImageClassifier`][TensorFlowImageClassifier], kita menggunakan `Interpreter` untuk mengakses model dan label yang ada dalam folder `assets` dan menggunakannya untuk menebak gambar yang telah diambil menggunakan syntax `interpreter.run()`. 

{% highlight JAVA %}
public List<Recognition> recognizeImage(Bitmap bitmap) {
    ByteBuffer byteBuffer = convertBitmapToByteBuffer(bitmap);
    byte[][] result = new byte[1][labelList.size()];
    interpreter.run(byteBuffer, result);
    return getSortedResult(result);
}
{% endhighlight %}

Pada `MainActivity`, kita lakukan inisialisasi Tensor Flow lite, namun proses tidak bisa dilakukan pada Main Thread. Untuk itu kita bisa menggunakan bantuan library [ReactiveX][rx] untuk menghandle threadnya. Jangan lupa untuk melakukan `classifier.close()` pada `onDestroy()`.

{% highlight JAVA %}
public class MainActivity extends AppCompatActivity {

    ....

    private static final String MODEL_PATH = "mobilenet_quant_v1_224.tflite";
    private static final String LABEL_PATH = "labels.txt";
    private static final int INPUT_SIZE = 224;
    private Classifier classifier;

    ....

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ....

        initTensorFlow().subscribe(...);

        ...
    }

    @Override
    protected void onDestroy() {

        closeClassifier().subscribe(...);
        super.onDestroy();

        ...
    }

    private Completable initTensorFlow(){
        return Completable.fromAction(new Action() {
            @Override
            public void run() throws IOException {
                classifier = TensorFlowImageClassifier.create(
                        getAssets(),
                        MODEL_PATH,
                        LABEL_PATH,
                        INPUT_SIZE);
            }
        }).subscribeOn(Schedulers.newThread()).observeOn(AndroidSchedulers.mainThread());
    }

    private Completable closeClassifier(){
        return Completable.fromAction(new Action() {
            @Override
            public void run() {
                classifier.close();
            }
        }).subscribeOn(Schedulers.newThread());
    }

    ....

}
{% endhighlight %}

Pada Callback Camera Kit, kita generate Bitmap dan gunakan `classifier.recognizeImage(bitmap)` untuk memulai menebak gambar tersebut.

{% highlight JAVA %}
public void onImage(CameraKitImage cameraKitImage) {
    Bitmap bitmap = cameraKitImage.getBitmap();
    bitmap = Bitmap.createScaledBitmap(bitmap, INPUT_SIZE, INPUT_SIZE, false);
    final List<Classifier.Recognition> results = classifier.recognizeImage(bitmap);
    showPreview(true, bitmap, generateResults(results));
}
{% endhighlight %}

Kita coba jalankan aplikasinya, foto beberapa objek dan lihat apakah aplikasi ini berhasil menebak dengan benar.

<img src="http://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_20180513-122720.png" width="300" height="533" />

<img src="http://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_20180513-122733.png" width="300" height="533" />

Terlihat disana aplikasi berhasil menebak objek Mouse dengan tepat, dengan tingkat keyakinan 100%. Dan juga menebak objek Laptop dengan keyakinan 57.6%. Keakuratan dan keyakinan nya masih dapat berubah sewaktu waktu tergantung sudut pengambilan gambar.

### Penutup

Sekian eksperimen kita dengan TensorFlow Lite di Android, berikutnya mungkin kita perlu melatih Model TensorFlow ini untuk meningkatkan keakuratannya. Buat yang mau melihat Source Codenya bisa cek [disini][repo].

Referensi:

- https://www.tensorflow.org/mobile/tflite/
- https://medium.com/mindorks/android-tensorflow-lite-machine-learning-example-b06ca29226b6
- https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/lite


[tflite]: https://www.tensorflow.org/mobile/tflite/
[mobilenet]: https://storage.googleapis.com/download.tensorflow.org/models/tflite/mobilenet_v1_224_android_quant_2017_11_08.zip
[listmodels]: https://github.com/tensorflow/tensorflow/blob/master/tensorflow/contrib/lite/g3doc/models.md
[camerakit]: https://github.com/CameraKit/camerakit-android
[camerakitdoc]: http://docs.camerakit.website
[TensorFlowImageClassifier]: https://github.com/dekzitfz/TensorFlowLite-Android/blob/master/app/src/main/java/dekz/id/tensorflowandroid/TensorFlowImageClassifier.java
[rx]: http://reactivex.io/
[repo]: https://github.com/dekzitfz/TensorFlowLite-Android