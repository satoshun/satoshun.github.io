+++
date = "2018-07-12T00:00:00Z"
title = "Kotlin: typealiasを使ってめんどうなアノテーションを省略する"
tags = ["android","kotlin"]
blogimport = true
type = "post"
+++

Kotlinではtypealiasと呼ばれる、既存のタイプに対して別名をつける機能があります。
https://kotlinlang.org/docs/reference/type-aliases.html

典型的な使い方として、下のような複雑な型に名前をつけると可読性が高くなります。

```kotlin
typealias MyHandler = (Int, String, Any) -> Unit
```

さらにtypealiasは、アノテーションをつけることも出来ます。

```kotlin
typealias NonWildcardList<T> = List<@JvmSuppressWildcards T>
```

`JvmSuppressWildcards`はDagger([参考リンク](https://github.com/google/dagger/issues/668))やretrofit([参考リンク](https://github.com/square/retrofit/issues/1805))を使っていると、使わざるをえないケースがあるので、typealiasで定義していおけば可読性高く、楽することが出来ます。
