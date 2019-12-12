+++
date = "Tue Oct 29 01:32:36 UTC 2019"
title = "Android Studio: Firebaseなどのクラッシュログから関数ジャンプできるようにする"
tags = ["android", "androidstudio", "agp"]
blogimport = true
type = "post"
draft = false
+++

[Android Studio: Debugging Tips n' Tricks (Android Dev Summit '19)](https://youtu.be/rjlhSDhFwzM?t=1043)で、便利な機能があったので、それの紹介です。

---

テキスト形式のクラッシュログを、Android Studio上で関数ジャンプ出来るようになります。 Firebaseのクラッシュログを例にして説明してきます。

まず、 Firebaseからクラッシュログをコピーします。

{{< figure src="/blog/android/agp/android-studio/firebase-crash-log.png" >}}

---

次に、Android StudioのAnalyze Stack Trace or Thread Dumpを使い、さきほどコピーしたクラッシュログをペーストします。

{{< figure src="/blog/android/agp/android-studio/firebase-crash-log-android-studio.png" >}}

---

そうすると、Android Studio上で、関数ジャンプが出来るようになります！

{{< figure src="/blog/android/agp/android-studio/firebase-crash-log-result.png" >}}
