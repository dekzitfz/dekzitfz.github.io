---
layout: post
title:  "Continuous Integration Project Android Di Gitlab"
date:   2016-09-27 13:43:38 
categories: [ci,integration,build,android,gitlab,git]
disqus_identifier: 104
comments: true
---

![logo gitlabCI](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/icon_gitlab_ci.png)

Beberapa hari yang lalu, saya iseng-iseng melihat source code dari aplikasi [google I/O di github][iosched], di list commitnya terdapat status build yang menggunakan [Travis CI][travisCI]. Travis CI ini sangat terintegrasi dengan GitHub dan sudah digunakan secara luas, lalu bagaimana dengan Gitlab?

<!--more-->

Sebelumnya saya belum pernah sama sekali menggunakan CI(Continuous Integration), bahkan tidak tahu apa sebenarnya dan apa fungsi dari CI. Jadi, setelah menelusuri tentang CI, akhirnya saya jadi lumayan tahu, lumayan ya, bukan tahu banget.

### Apa itu Continuous Integration?
> Continuous Integration is a software development practice where members of a team integrate their work frequently, usually each person integrates at least daily - leading to multiple integrations per day. Each integration is verified by an automated build (including test) to detect integration errors as quickly as possible. Many teams find that this approach leads to significantly reduced integration problems and allows a team to develop cohesive software more rapidly. - [Martin Fowler](http://martinfowler.com/articles/continuousIntegration.html)

Jadi singkatnya, CI ini adalah suatu kegiatan dalam pengembangan software yang mengharuskan seluruh developer untuk mengintegrasikan hasil kerjanya ke sebuah repo dan dari repo itu nantinya akan dilakukan verifikasi dengan build otomatis, sehingga jika ada error akan diketahui secara dini.

### Kenapa menggunakan Gitlab?
Gitlab adalah salah satu layanan penyimpan Git gratis seperti GitHub. salah satu kelebihannya yaitu kita dapat membuat private repo secara cuma-cuma dan sebanyak mungkin. Sehingga saya cenderung menggunakan Gitlab untuk project yang sourcenya tidak boleh diketahui publik. Dan tentu saja karena Gitlab memiliki sistem CI yang bisa digunakan langsung di setiap reponya, detailnya bisa dilihat [disini][aboutgitlabci].

### Start!
Oke, kita mulai saja, pertama-tama kita setup repository di lokal dan tambahkan remote ke repo di Gitlab.

Buat file `.gitlab-ci.yml` di root project android, file ini merupakan file konfigurasi build yang akan dieksekusi oleh Gitlab CI.

{% highlight YAML %}
image: java:openjdk-8-jdk

before_script:
  - apt-get --quiet update --yes
  - apt-get --quiet install --yes wget tar unzip lib32stdc++6 lib32z1
  - wget --quiet --output-document=android-sdk.tgz https://dl.google.com/android/android-sdk_r24.4.1-linux.tgz
  - tar --extract --gzip --file=android-sdk.tgz
  - echo y | android-sdk-linux/tools/android --silent update sdk --no-ui --all --filter android-24
  - echo y | android-sdk-linux/tools/android --silent update sdk --no-ui --all --filter platform-tools
  - echo y | android-sdk-linux/tools/android --silent update sdk --no-ui --all --filter build-tools-24.0.2
  - echo y | android-sdk-linux/tools/android --silent update sdk --no-ui --all --filter extra-android-m2repository
  - echo y | android-sdk-linux/tools/android --silent update sdk --no-ui --all --filter extra-google-google_play_services
  - echo y | android-sdk-linux/tools/android --silent update sdk --no-ui --all --filter extra-google-m2repository
  - export ANDROID_HOME=$PWD/android-sdk-linux
  - chmod +x ./gradlew

build:
  script:
    - ./gradlew assembleDebug
  artifacts:
    paths:
    - app/build/outputs/
{% endhighlight %}

Sedikit penjelasan tentang isi file ini, 

- `image: java:openjdk-8-jdk` digunakan untuk mendefiniskan Docker image yang digunakan, disini saya menggunakan OpenJDK 8.
- selanjutnya di bagian `before_script` kita meng-update beberapa dependencies dengan perintah - perintah `apt-get` dan `wget`.
- file Android SDK yang sudah diunduh akan diekstrak dan diinstall satu-persatu sesuai kebutuhan.
- `export ANDROID_HOME=$PWD/android-sdk-linux` akan membuat environtment variable `ANDROID_HOME`.
- `chmod +x ./gradlew` untuk membuat script gradle wrapper dapat dieksekusi.
- pada block `script` inilah command Gradle akan dieksekusi, kita dapat mengisinya sesuai kebutuhan.
- pada block `paths` mendeskripsikan output dari hasil build yang sudah dilakukan.

Lakukan `add`, `commit`, dan `push` file `.gitlab-ci.yml`. Gitlab akan otomatis melakukan build yang dapat kita lihat progressnya di bagian pipeline project.

![pipeline](http://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_092716_121338_PM.jpg)

Kita juga dapat melihat log build yang sedang berjalan.

![pipeline](http://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_092716_121405_PM.jpg)

Ini tampilan jika build berhasil dilakukan. Hasil build juga bisa di download langsung.

![pipeline](http://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_092716_121847_PM.jpg)

Cukup simpel dan praktis bukan? contoh repo bisa dilihat [disini][gitlabCI].

##### Referensi
- http://www.greysonparrelli.com/post/setting-up-android-builds-in-gitlab-ci-using-shared-runners/
- http://blog.goddchen.de/2016/04/configuration-for-gitlab-ci-android-projects/
- http://software.endy.muhardin.com/manajemen/continuous-integration/


[iosched]: https://github.com/google/iosched
[travisCI]: https://travis-ci.org/
[gitlabCI]: https://gitlab.com/dekzitfz/GitlabCIExample
[aboutgitlabci]: https://about.gitlab.com/gitlab-ci/