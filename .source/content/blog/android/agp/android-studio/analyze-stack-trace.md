+++
date = "Tue Oct 29 01:32:36 UTC 2019"
title = "Android Studio: Firebaseなどのクラッシュログをダンプして関数ジャンプできるようにする"
tags = ["android", "androidstudio", "agp"]
blogimport = true
type = "post"
draft = false
+++

テキスト形式のクラッシュログを、Android Studio上で関数ジャンプ出来るようにする方法です。
Firebaseのクラッシュログを例にして説明してきます。

まず、 Firebaseからクラッシュログをコピーします。

<img src="/blog/android/agp/android-studio/firebase-crash-log.png" />

---

次に、Android StudioのAnalyze Stack Trace or Thread Dumpを使い、さきほどコピーしたクラッシュログをペーストします。

<img src="/blog/android/agp/android-studio/firebase-crash-log-android-studio.png" />

---

そうすると、Android Studio上で、関数ジャンプが出来るようになります！

<img src="/blog/android/agp/android-studio/firebase-crash-log-result.png" />
