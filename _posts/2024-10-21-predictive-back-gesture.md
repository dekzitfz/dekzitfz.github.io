---
layout: post
title: Predictive Back Gesture On Android
date: 2024-10-21

tags: android open-source
splash_img_source: /assets/img/post_predictive_back.png # high resolution images with an aspect ratio close to 4:3 recommended
splash_img_caption: 

# Optional front matter
updated: # Updated date in YYYY-MM-DD format
author: 
  name: # Author name, if not provided defaults to site.author.name
  homepage: # Author link, if not provided defaults to site.author.homepage
pin: false # true if this post must be pinned on top of the page, default is false.
listed: true # false if this post must NOT be included on the posts page, sitemap, and any of the tag pages, default is true
index: true # When false, <meta name="robots" content="noindex"> is added to the page, default is true
---

If you already using Android 15 and exploring the settings menu, you'll may find a brand new behavior when try to swipe back to previous screens. A preview of previous screen is showing, indicates that we will takes to that screen if we swipe back.

### Predictive Back Gesture
Predictive back gesture, as it names is an improved user experience when user try to navigate back to previous page by executing a back swipe. This feature allows users to see the result of a back gesture before completing it, so they can decide whether to stay in the current view or continue. It’s all about making the back gesture more predictive and context-aware.

![Predictive back preview](https://developer.android.com/static/images/about/versions/13/predictive-back-nav-home.gif)

The predictive back gesture was introduced in Android 13 and the back-to-home animation can be previewed by enabling it via `Developer Option > Predictive back animations`. This is also required for Android 14. 

### Predictive Back Gesture Implementation
To implement the Predictive Back Gesture in your Android app, firstly you should ensure that APIs that are already using OnBackPressedDispatcher APIs (such as Fragments and the Navigation Component) work seamlessly with the predictive back gesture, so you upgrade to [AndroidX Activity 1.6.0-alpha05](https://developer.android.com/jetpack/androidx/releases/activity#1.6.0-alpha05) or later.

```groovy
// In your build.gradle file:
dependencies {

    // Add this in addition to your other dependencies
    implementation "androidx.activity:activity:1.8.0"

}
```

Opt-in to the predictive back gesture by update your `AndroidManifest.xml` file by add `android:enableOnBackInvokedCallback="true` flag.

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    ...
    <application
        ...
        android:enableOnBackInvokedCallback="true">
        ...
    </application>

</manifest>
```

### Intercepting Back Gesture
Sometimes we don't want to straightforward bring the user to previous screen when swipe back, for example when we want to show a dialog to confirm user if they really wanna go to previous page, we need to add extra step to intercept the swipe back logic.

To do that, we need to use `OnBackPressedDispatcher` with an implementation of `OnBackPressedCallback`. Firstly create `OnBackPressedCallback` instance.

```kotlin
private val onBackPressedCallback: OnBackPressedCallback = object : OnBackPressedCallback(true) {
    override fun handleOnBackPressed() {
        //your custom onBackPressed logic
        showConfirmationDialog()
    }
}
```

And then add it as callback to `OnBackPressedDispatcher`

```kotlin
onBackPressedDispatcher.addCallback(onBackPressedCallback)
```

Lastly, logic for the Confimation Dialog

```kotlin
private fun showConfirmationDialog() {
    val confirmDialog = AlertDialog.Builder(this)
    confirmDialog.setMessage("Are you sure want to go back?")
    confirmDialog.setCancelable(false)
    confirmDialog.setNegativeButton("Cancel") { d, _ ->
        d.dismiss()
    }
    confirmDialog.setPositiveButton("Confirm") { _, _ ->
        //stop intercepting the back gesture
        onBackPressedCallback.isEnabled = false

        //navigate back
        onBackPressedDispatcher.onBackPressed()
    }
    confirmDialog.show()
}
```

That’s it! That’s how you implement predictive back gesture to your app. Checkout the [Github Repository](https://github.com/dekzitfz/PredictiveBackExample) for more details on code.

##### Reference
- https://codelabs.developers.google.com/handling-gesture-back-navigation
- https://developer.android.com/guide/navigation/custom-back/predictive-back-gesture
