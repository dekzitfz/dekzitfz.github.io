---
layout: post
title:  "Implementasi OTP dengan SMS Retriever API"
date:   2021-03-29 09:00:00 
categories: [android,sms,otp]
disqus_identifier: 115
comments: true
---

![](https://developers.google.com/identity/sms-retriever/flow-overview.png)

Pada setiap proses otentikasi biasanya diperlukan verifikasi pengguna, umumnya verifikasi dilakukan via email ataupun SMS OTP (One Time Password). Pada aplikasi android verifikasi via SMS lebih disukai karena prosesnya terbilang lebih mudah dan lebih cepat. Disini saya akan coba sharing cara mengimplementasikan SMS OTP menggunakan SMS Retriever API dari Google.

<!--more-->

> Artikel ini hanya akan lebih fokus membahas implementasi dari sisi Android. Untuk implementasi dari sisi server silahkan baca dokumentasi resmi.


### SMS Retriever API

SMS Retriever API merupakan bagian dari Google Play Services. API ini memungkinkan kita untuk mendeteksi SMS yang masuk secara otomatis untuk kemudian kita gunakan pada proses verifikasi pengguna. Keuntungan menggunakan API ini salah satunya adalah adalah kita tidak perlu menggunakan `android.permission.RECEIVE_SMS` lagi untuk dapat mengakses SMS yang masuk, selain itu metode ini ini juga sudah tidak direkomendasikan karena alasan untuk melindungi privasi pengguna.

### Implementasi Sisi Android

Tambahkan komponen Play Services Auth ke `build.gradle` di dalam folder `app`

{% highlight Groovy %}
implementation 'com.google.android.gms:play-services-auth:19.0.0'
implementation 'com.google.android.gms:play-services-auth-api-phone:17.5.0'
{% endhighlight %}

Buat `BroadcastReceiver` untuk handle konten sms yang akan masuk

{% highlight Kotlin %}
class SMSReceiver : BroadcastReceiver() {

    interface SMSListener{
        fun onOTPReceived(otp: String)
    }

    private var listener: SMSListener? = null

    override fun onReceive(context: Context?, intent: Intent?) {
        if (SmsRetriever.SMS_RETRIEVED_ACTION == intent?.action) {
            val extras = intent.extras
            val status = extras?.get(SmsRetriever.EXTRA_STATUS) as Status
            when (status.statusCode) {
                CommonStatusCodes.SUCCESS -> {
                    val message = extras.get(SmsRetriever.EXTRA_SMS_MESSAGE) as String
                    val otpPattern = Pattern.compile("(\\d{5})") //contoh regex yang mengambil 5 digit kode OTP
                    val matcher = otpPattern.matcher(message)
                    if(matcher.find()){
                        listener?.onOTPReceived(matcher.group(0)!!)
                    }else{
                        Log.w("SMSReceiver", "regex failed to get otp code")
                    }
                }
            }
        }
    }

    fun setListener(listener: SMSListener) {
        this.listener = listener
    }

}
{% endhighlight %}

Konfigurasikan kelas `SMSReceiver` dan jalankan `SmsRetriever` pada `Activity` tempat kita untuk menginput kode OTP

{% highlight Kotlin %}
class OTPActivity: AppCompatActivity(), SMSReceiver.SMSListener {

    private lateinit var smsReceiver: SMSReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_otp)
        ...
        startSMSRetriever()
        ...
    }

    override fun onDestroy() {
        unregisterReceiver(smsReceiver)
        super.onDestroy()
    }

    private fun startSMSRetriever(){
        smsReceiver = SMSReceiver()
        smsReceiver.setListener(this)
        registerReceiver(smsReceiver, IntentFilter(SmsRetriever.SMS_RETRIEVED_ACTION))

        val smsRetrieverClient = SmsRetriever.getClient(this)
        val smsRetrieverTask = smsRetrieverClient.startSmsRetriever()
        smsRetrieverTask.addOnCompleteListener { task->
            if(task.isSuccessful){
                Log.i("OTPActivity", "smsRetrieverClient task success")
            }else{
                Log.e("OTPActivity", task.exception?.message!!)
            }
        }
    }

    override fun onOTPReceived(otp: String) {
        //handle your otp here
    }

}
{% endhighlight %}

Kode diatas melakukan setup pada method `startSMSRetriever` untuk kemudian mengambil kode OTP pada method `onSMSReceived`. Jangan lupa juga untuk melakukan unregister `SMSReceiver` di method `onDestroy`.


### Pengujian

Sebelum melakukan pengujian pada kode yang sudah kita buat, kita perlu melakukan setup 11-character hash untuk mengidentifikasi sms yang masuk. Penjelasan lengkap tentang ini dapat dilihat di [dokumentasi ini][11-hash-doc], tapi jika menurut kalian menggunakan command-line cukup rumit, disini kita bisa coba cara lain yaitu menggunakan class helper.

buat kelas `AppSignatureHelper`, kelas ini bertujuan untuk melakukan generate 11-caracter hash berdasarkan package-name dan certificate yg kita gunakan.

{% highlight Kotlin %}
class AppSignatureHelper(context: Context?) : ContextWrapper(context) {

    private lateinit var signatures: SigningInfo

    val appSignatures: ArrayList<String>
        @RequiresApi(Build.VERSION_CODES.P)
        get() {
            val appCodes = ArrayList<String>()
            try {
                val packageName = packageName
                val packageManager = packageManager

                signatures = packageManager.getPackageInfo(
                    packageName,
                    PackageManager.GET_SIGNING_CERTIFICATES
                ).signingInfo

                for (signature in signatures.apkContentsSigners) {
                    val hash = hash(packageName, signature.toCharsString())
                    if (hash != null) {
                        appCodes.add(String.format("%s", hash))
                    }
                }
            } catch (e: PackageManager.NameNotFoundException) {
                Log.e(TAG, e.message!!)
            }

            return appCodes
        }

    companion object {
        val TAG = AppSignatureHelper::class.java.simpleName

        private val HASH_TYPE = "SHA-256"
        val NUM_HASHED_BYTES = 9
        val NUM_BASE64_CHAR = 11

        private fun hash(packageName: String, signature: String): String? {
            val appInfo = "$packageName $signature"
            try {
                val messageDigest = MessageDigest.getInstance(HASH_TYPE)
                messageDigest.update(appInfo.toByteArray(StandardCharsets.UTF_8))
                var hashSignature = messageDigest.digest()

                // truncated into NUM_HASHED_BYTES
                hashSignature = Arrays.copyOfRange(hashSignature, 0, NUM_HASHED_BYTES)
                // encode into Base64
                var base64Hash = Base64.encodeToString(hashSignature, Base64.NO_PADDING or Base64.NO_WRAP)
                base64Hash = base64Hash.substring(0, NUM_BASE64_CHAR)

                Log.d(TAG, String.format("pkg: %s -- hash: %s", packageName, base64Hash))
                return base64Hash
            } catch (e: NoSuchAlgorithmException) {
                Log.e(TAG, e.message!!)
            }

            return null
        }
    }
}
{% endhighlight %}

Jalankan kelas ini pada Activity Launcher

{% highlight Kotlin %}
//disable this in production code
val appSignatureHelper = AppSignatureHelper(applicationContext)
appSignatureHelper.appSignatures
{% endhighlight %}

Jalankan aplikasi, kita akan menemukan 11-caracter hash di `logcat`, kira-kira formatnya seperti ini

{% highlight Log %}
D/AppSignatureHelper: pkg: id.adiandrea.otpexample -- hash: aA2MtPvtEex
{% endhighlight %}

Kode `aA2MtPvtEex` inilah yang akan kita gunakan sebagai identifier di konten sms nanti. Selanjutnya kita bisa melakukan simulasi menerima sms masuk dengan emulator bawaan android studio. Konten sms yang kita gunakan pada contoh ini adalah OTP dengan 5 digit.

{% highlight Log %}
<#> your otp: 32451

aA2MtPvtEex
{% endhighlight %}

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/sms-retriever-1.png)

Jalankan aplikasi, lalu kirim sms simulasi pada activity untuk input OTP, kdoe OTP akan langsung terinput.

![](https://raw.githubusercontent.com/dekzitfz/dekzitfz.github.io/master/img/posts/sms-retriever-2.png)

Contoh kode sumber yang lengkap dapat kalian cek pada [repositori ini][repo].

Referensi:
- https://developers.google.com/identity/sms-retriever/
- https://nhkarthick.medium.com/android-verify-otp-automatically-with-sms-retriever-api-177168b623ae

[11-hash-doc]: https://developers.google.com/identity/sms-retriever/verify#1_construct_a_verification_message
[repo]: https://github.com/dekzitfz/SMS-Retriever-API-Sample