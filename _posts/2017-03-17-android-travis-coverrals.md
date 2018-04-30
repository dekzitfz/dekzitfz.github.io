---
layout: post
title:  "Integrasi Project Android Di Github Dengan Travis Dan Coverrals"
date:   2017-03-17 17:53:31 
categories: [android,git,github,ci,coverrals,unit test]
disqus_identifier: 109
comments: true
---

<img src="https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/travis_coverrals.png" width="600">

Pada beberapa project di Github, saya sering melihat beberapa badge yang menunjukan status dari repository tersebut. Kebetulan saya juga menggunakan salah satunya, yaitu [Travis CI][travis].

<!--more-->

Beberapa hari yang lalu, saya sedang iseng - iseng menelusuri github dan melihat badge yang cukup menarik, badge tersebut berisi sejumlah persentase. Karena penasaran, saya mencari infonya dan ternyata itu adalah [Coverrals.io][coverrals].

### Coverrals.io

[Coveralls][coverrals] adalah sebuah tool yang dapat kita gunakan untuk menganalisa repository kita dan menemukan bagian - bagian kode yang belum tercover oleh unit tes. Presentase pada badge itulah yang menunjukan berapa persen dari seluruh kode kita yang sudah tercover.

### Travis CI

[Travis][travis] merupakan layanan CI (Continous Integration) online gratis yang bisa kita pakai pada project di Github. Kita bisa melakukan tes dan build secara otomatis saat kita melakukan `push`.

Disini saya menggunakan contoh aplikasi kalkulator sederhana, dimana ada fungsi penjumlahan, pengurangan, perkalian, dan pembagian dari dua angka yang diinput oleh user.

Fungsi kalkulator saya buat pada kelas `Calculator`, contoh dari fungsi penjumlahan seperti kode dibawah ini.

{% highlight JAVA %}
public class Calculator {
    public double add(double num1,double num2){
        return num1 + num2;
    }
}
{% endhighlight %}

Fungsi penjumlahan ini dipanggil pada `activity` saat tombol di klik.

{% highlight JAVA %}
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        final ActivityMainBinding binding = DataBindingUtil.setContentView(this,R.layout.activity_main);

        final Calculator calculator = new Calculator();

        binding.btnTambah.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(!TextUtils.isEmpty(binding.etNum1.getText())){
                    binding.tvHasil.setText(String.valueOf(
                            calculator.add(
                                    Double.parseDouble(binding.etNum1.getText().toString()),
                                    Double.parseDouble(binding.etNum2.getText().toString()))));
                }
            }
        });
    }
}
{% endhighlight %}

Login ke Travis dengan akun Github dan authorize agar Travis dapat mengakses repositori kita. Kemudian kita dapat menambah repositori untuk diaktifkan.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_031617_043326_PM.jpg)

Buat file `.travis.yml` yang berisi konfigurasi untuk melakukan trigger build di Travis. Letakkan di `root` project.

{% highlight YML %}
language: android
sudo: required
jdk: oraclejdk8
android:
  components:
    - tools
    - platform-tools
    - tools
    - build-tools-25.0.1
    - android-25
    - android-22
    - extra-google-google_play_services
    - extra-google-m2repository #Google Play Services
    - extra-android-m2repository #Design Support Library
    - extra-android-support
    - addon-google_apis-google-25
    - sys-img-armeabi-v7a-android-22

before_install: 
  - "chmod +x gradlew"

# Emulator Management: Create, Start and Wait
before_script:
  - echo no | android create avd --force -n test -t android-22 --abi armeabi-v7a
  - emulator -avd test -no-skin -no-audio -no-window &
  - android-wait-for-emulator
  - adb shell input keyevent 82 &

script:
   - ./gradlew clean build connectedCheck coveralls -PdisablePreDex --stacktrace

after_failure:
  # Customize this line, 'android' is the specific app module name of this project. Shows log.
  - export MY_MOD="app"
  - export MY_LOG_DIR="$(pwd)/${MY_MOD}/build/outputs/reports/androidTests/connected/"
  - pwd && cd "${MY_LOG_DIR:-.}" && pwd && ls -al
  - sudo apt-get install -qq lynx && lynx --dump index.html > myIndex.log
  - lynx --dump com.android.builder.testing.ConnectedDevice.html > myConnectedDevice.log
  - lynx --dump com.android.builder.testing.html > myTesting.log
  - for file in *.log; do echo "$file"; echo "====================="; cat "$file"; done || true
{% endhighlight %}

### Coverrals Plugin

Sebelum menggunakan Coverrals, kita perlu melakukan beberapa setup agar Gradle dapat mengirim coverage reports kepada Coverrals setiap build Travis sukses. Tambahkan `classpath dependency` pada file `build.gradle` di `root` Project.

{% highlight Groovy %}
buildscript {
    ...
    dependencies {
        ...
        classpath 'org.kt3k.gradle.plugin:coveralls-gradle-plugin:2.4.0'
    }
}
{% endhighlight %}

apply plugin pada file `build.gradle` di folder `app`.

{% highlight Groovy %}
apply plugin: 'com.github.kt3k.coveralls'
{% endhighlight %}

{% highlight Groovy %}
android {
    ...
    buildTypes {
        debug {
            testCoverageEnabled true
        }
    }
}
{% endhighlight %}

{% highlight Groovy %}
dependencies {
    ...
}

coveralls {
    jacocoReportPath = "${buildDir}/reports/coverage/debug/report.xml"
}

tasks.coveralls {
    dependsOn 'connectedAndroidTest'
    onlyIf { System.env.'CI' }
}
{% endhighlight %}

Buat file `.coveralls.yml` yang berisi konfigurasi `token` untuk mengakses repositori kita. Tiap repo akan memiliki `token` yang berbeda.

{% highlight YML %}
service_name: travis-ci
repo_token: <token-here>
{% endhighlight %}

Login ke Coverrals dengan akun github, pilih menu `ADD REPOS`, tambahkan repo yang ingin kita lakukan coverage.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_031617_050758_PM.jpg)

Coba lakukan `push` ke Github, dan jika build berhasil pada Travis CI, Coverrals akan menunjukan persentase pada dashboard repo yang sudah kita aktifkan.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_031617_051010_PM.jpg)

Terlihat persentase Coverage adalah 0%, karena kita belum membuat unit test sama sekali.

### Membuat Unit Test

Buat kelas `CalculatorTest` di `app\src\androidTest\java\<package-name>`. Contoh nya seperti kode dibawah ini.

{% highlight JAVA %}
@RunWith(AndroidJUnit4.class)
public class CalculatorTest {
    @Test
    public void addTest() throws Exception {
        Calculator calculator = new Calculator();
        double result = calculator.add(3.0,2.5);
        assertEquals(5.5,result,0);
    }
}
{% endhighlight %}

Kode diatas untuk memastikan fungsi dari `calculator.add()` menghasilkan output nilai yang benar. Sekarang coba `push` perubahan yang telah kita lakukan dan lihat persentase coverage yang baru.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_031617_053838_PM.jpg)

Terlihat persentase coverage naik menjadi 14%. Kita juga bisa melihat bagian kode yang sudah tercover oleh unit test.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_031617_054220_PM.jpg)

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/Screenshot_031617_054324_PM.jpg)

Bagian yang berwarna hijau menandakan kode yang sudah tercover unit test, sedangkan yang berwarna merah yang belum tercover.

Contoh repo dapat dilihat [disini][repo].

[travis]: https://travis-ci.org/
[coverrals]: https://coveralls.io/
[repo]: https://github.com/dekzitfz/TravisCoveralls