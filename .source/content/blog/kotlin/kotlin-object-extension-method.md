+++
date = "Thu Jan  7 12:16:07 UTC 2021"
title = "Kotlin: objectクラスで拡張関数をグループ化する"
tags = ["kotlin"]
blogimport = true
type = "post"
draft = false
+++

最近firebase ktxを使っていて、良い書き方だと思ったので紹介します。

まず、firebase-ktxの一部コードを抜粋。より詳しいコードは [firebase-common/Firebase.kt](https://github.com/firebase/firebase-android-sdk/blob/master/firebase-common/ktx/src/main/kotlin/com/google/firebase/ktx/Firebase.kt) を見てください。

```kotlin
package com.google.firebase.ktx

object Firebase

val Firebase.app: FirebaseApp
    get() = FirebaseApp.getInstance()

...
```

このコードは、トップに空の `object Firebase`を定義し、そこに `app` 拡張関数を生やしています。

さらに、firebase-remoteconfig-ktx 等でも、`object Firebase` に対して、拡張関数を生やしています。

```kotlin
package com.google.firebase.remoteconfig.ktx

val Firebase.remoteConfig: FirebaseRemoteConfig
    get() = FirebaseRemoteConfig.getInstance()

...
```

こうすることのメリットは、共通の `object Firebase`上に拡張関数を定義しているので、IDE上で `Firebase.`とタイピングして自動補完をすれば一覧で全てを参照できる点です。（ライブラリの依存をGradleで指定している場合）

## まとめ

トップに空のobjectクラスを定義して、そこに拡張関数を生やすことで、似たものをまとめる（グループ化）することができる。トップレベル関数等を多用しすぎている場合は、このパターンを検討して、自動補完空間をキレイにすると良さそう。
