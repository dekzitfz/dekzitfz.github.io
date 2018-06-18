---
layout: post
title:  "Firebase Test Lab via Gitlab Continuous Integration"
date:   2018-05-18 17:31:05 
categories: [android,firebase,test,ci]
disqus_identifier: 114
comments: true
---

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/gitlab-firebase-test.png)

Di android kita biasa melakukan unit test ataupun instrumented (UI) test. Kita bisa melakukan testing sesaat setelah kita melakukan commit dan push code kita melalui git, semua secara otomatis, tanpa kita perlu menjalankan tes secara manual, bagaimana caranya?

<!--more-->

> Peringatan! Artikel ini hanya akan berfokus pada integrasi CI dan Firebase Test Lab. Untuk membuat Instrumented Test kalian bisa dapatkan banyak contoh di internet.

### Firebase Test Lab

Salah satu tool dari firebase yang akan membantu kita menjalankan tes ini adalah Firebase Test Lab. Kita bisa menjalankan [Robo Test][robotest] dan [Instrumented Test][instrumentedtest] ke virtual devices maupun physical device yang tersedia. Lebih lanjut tentang Firebase Test Lab bisa dibaca [disini][firebasetestlab].

### Gitlab Continuous Integration

Kenapa pakai Gitlab CI? Jawawbannya simpel, karena  saya belum coba pakai CI yang lain, Hahahaha. Gitlab CI ini cukup mudah di setup juga, jadi saya lebih suka pakai ini, gratis pula. Tutorial untuk setup Gitlab CI juga bisa kalian banyak temukan di internet.

### Go Automation!

Oke, disini saya asumsikan kita sudah mempunyai repo project android di Gitlab, dan kita sudah pernah menggunakan Gitlab CI, minimal menggunakannya untuk build apk versi debug.

Pada file `.gitlab-ci.yml`, saya menggunakan image docker milik [jangrewe][jangrewe], karena dengan image ini kita tidak perlu melakukan setup lagi pada Android SDK. Untuk initial setup bisa ikuti intruksi yang ada di reponya. Sehingga secara default isi dari configurasi CI kita untuk melakukan build apk debug adalah seperti dibawah ini.

{% highlight YAML %}
image: jangrewe/gitlab-ci-android

stages:
  - build

before_script:
- export GRADLE_USER_HOME=$(pwd)/.gradle
- chmod +x ./gradlew

cache:
  key: ${CI_PROJECT_ID}
  paths:
  - .gradle/

build_dev_debug:
  stage: build
  script:
    - ./gradlew assembleDevDebug
  only:
    - dev
  artifacts:
    paths:
      - app/build/outputs/
{% endhighlight %}

Disini saya juga asumsikan kita telah membuat project di Firebase dan sudah men-setup nya ke project android. Kita perlu mengaktifkan beberapa API dari [Developer Console][gdevconsole], diantaranya yaitu **Google Cloud Testing API**, **Cloud Tool Results API**, dan **Google Cloud Storage JSON API**. Pastikan project yang dipilih sudah sesuai.

Buat Service Account, ini digunakan untuk merepresentasikan identitas kita atau program yang akan menggunakan layanan Google Cloud. Service Account dapat dibuat dari menu `Firebase Console -> Project Settings -> Service accounts -> Manage all service accounts`. Jangan lupa gunakan Role **Editor**.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_061818_035154_PM.jpg)

Pada Service Account yang sudah kita buat, pilih `option -> Create Key`, dan pilih format JSON.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_061818_035313_PM.jpg)

Sebuah file berformat json akan terdownload, kita akan menggunakan isi dari file ini sebagai identitas untuk mengeksekusi Firebase Test Lab. Caranya, kita buka pengaturan CI repo di Gitlab (`Settings -> CI / CD`), Expand `Variables`, buat variabel baru dengan nama `SERVICE_ACCOUNT`, untuk valuenya, copy-paste seluruh isi dari file json yang telah terdownload sebelumnya, kemudian simpan perubahannya.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_061818_035932_PM.jpg)

Jika kalian membuka file json yang terdownload itu, akan ada value dari `project_id`, buat sebuah variabel di CI/CD Gitlab lagi dengan nama `PROJECT_ID`, isikan dengan value dari `project_id` dari file json tersebut.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_061818_044639_PM.jpg)

Kembali ke file `.gitlab-ci.yml`, kita akan mengupdate beberapa scriptnya.

Pada job `build_dev_debug`, tambahkan script untuk generate file APK yang nantinya digunakan untuk menjalankan Instrumented test. Perlu dicatat, untuk melakukan Instrumented test diperlukan dua buah file APK, satu adalah APK versi debug biasa, dan satunya APK untuk menjalankan testnya.

{% highlight YAML %}
./gradlew assembleDevDebugAndroidTest
{% endhighlight %}

Syntax script ini bisa bervariasi, tergantung setup `productFlavors` dan `buildTypes` yang kalian gunakan. Pada kasus ini saya menggunakan `productFlavors` **`dev`** dan `buildTypes` **`debug`**.

> Catatan: kalian bisa menggunakan `assembleAndroidTest` untuk generate apk di semua build variant.

Selanjutnya kita buat job baru dengan stage test dengan nama `instrumentation_test_dev_debug`.

{% highlight YAML %}
instrumentation_test_dev_debug:
  stage: test
  only:
    - dev
  before_script:
    # Preparing wget
    - apt-get update
    - apt-get upgrade -y
    - apt install python2.7 python-pip -y
    - apt-get install wget

    # Install Google Cloud SDK
    - wget --quiet --output-document=/tmp/google-cloud-sdk.tar.gz https://dl.google.com/dl/cloudsdk/channels/rapid/google-cloud-sdk.tar.gz
    - mkdir -p /opt
    - tar zxf /tmp/google-cloud-sdk.tar.gz --directory /opt
    - /opt/google-cloud-sdk/install.sh --quiet
    - source /opt/google-cloud-sdk/path.bash.inc

    # Setup and configure the project
    - gcloud components update
    - gcloud config set project $PROJECT_ID

    # Activate cloud credentials
    - echo $SERVICE_ACCOUNT > /tmp/service-account.json
    - gcloud auth activate-service-account --key-file /tmp/service-account.json
  script:
    # copy file to root and rename it for easier access
    - cp app/build/outputs/apk/dev/debug/app-dev-debug.apk ./app.apk
    - cp app/build/outputs/apk/androidTest/dev/debug/app-dev-debug-androidTest.apk ./app-test.apk

    # deploy to gcloud and run instrumented test
    - gcloud firebase test android run --type instrumentation --app app.apk --test app-test.apk --device model=Nexus6P,version=26,locale=en,orientation=portrait
  dependencies:
    - build_dev_debug
{% endhighlight %}

Sedikit penjelasan dari `before_script` diatas, pertama kita setup package `wget` yang nantinya digunakan untuk mendownload file instalasi Google Cloud SDK, dan ternyata untuk setup wget diperlukan **python** juga. Setelah mendownload dan menginstall Google CLoud SDK, setup gcloud dengan menentukan project id nya. Setup identitas dengan variable Service Account yang kita buat tadi dan aktifkan service accountnya. 

Kemudian pada `script`, kita copy kedua APK ke root, dan merubah namanya, ini supaya nanti command run terakhir tidak terlalu panjang aja sih. Dan akhirnya, `gcloud firebase test android run ...` akan menjalankan tugasnya.

> Catatan: untuk menjalankan Robo test, kita hanya perlu mengganti `--type`, detail lainnya seperti memilih di device apa test akan dijalankan bisa cek [disini][gcloud-cli].

Jangan lupa tambahkan stage baru, yaitu `test` di file `.gitlab-ci.yml` sehingga kita sekarang memiliki 2 stage, `build` dan `test`. Versi lengkap dari file yang sudah jadi bisa dilihat [disini][gist].

What's next? commit dan push code kita dan biarkan Gitlab CI yang bekerja. Sesaat setelah tes berhasil, kita dapat melihat detail lengkapnya pada firebase console.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_061818_050442_PM.jpg)

Sekian tutorial integrasi Gitlab CI dengan Firebase Test Lab, jika ada pertanyaan ataupun kiritk dan saran silahkan post di kolom komentar yak!

Referensi:
- https://medium.com/evenbit/firebase-test-lab-with-gitlab-continuous-integration-c66b93bf896a
- https://firebase.google.com/docs/test-lab/
- https://developer.android.com/training/testing/unit-testing/instrumented-unit-tests

[robotest]: https://firebase.google.com/docs/test-lab/android/robo-ux-test
[instrumentedtest]: https://developer.android.com/training/testing/unit-testing/instrumented-unit-tests
[firebasetestlab]: https://firebase.google.com/docs/test-lab/
[jangrewe]: https://github.com/jangrewe/gitlab-ci-android
[gdevconsole]: https://console.developers.google.com/
[gist]: https://gist.github.com/dekzitfz/1e85e59beba420e970f666cc456f03a1
[gcloud-cli]: https://firebase.google.com/docs/test-lab/android/command-line