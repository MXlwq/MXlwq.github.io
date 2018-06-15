---
title: Fragment getContext() NoSuchMethodError
date: 2018-03-26 10:00:31
tags:
---

# Fragment getContext() 在Android低版本（API<=22）中出现NoSuchMethodError

## Log日志

```
Exception java.lang.NoSuchMethodError: No virtual method getContext()Landroid/content/Context; in class Lcom/android/browser/preferences/MainPreferenceFragment; or its super classes (declaration of 'com.android.browser.preferences.MainPreferenceFragment' appears in /system/priv-app/Browser/Browser.apk)
com.android.browser.preferences.MainPreferenceFragment.onPreferenceChange (SourceFile:203)
android.preference.Preference.callChangeListener (Preference.java:928)
miui.support.preference.ListPreference.onDialogClosed (SourceFile:300)
miui.support.preference.a.onDismiss (SourceFile:130)
android.app.Dialog$ListenersHandler.handleMessage (Dialog.java:1263)
android.os.Handler.dispatchMessage (Handler.java:102)
android.os.Looper.loop (Looper.java:135)
android.app.ActivityThread.main (ActivityThread.java:5296)
java.lang.reflect.Method.invoke (Method.java)
java.lang.reflect.Method.invoke (Method.java:372)
com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run (ZygoteInit.java:912)
com.android.internal.os.ZygoteInit.main (ZygoteInit.java:707)
```
## 切记
Fragment中的getContext()方法是在Android6.0中才出现的，在低于该版本中使用会报NoSuchMethodError，可以使用getActivity()来替换。
