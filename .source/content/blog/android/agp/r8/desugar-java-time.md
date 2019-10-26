+++
date = "Sat Oct 26 02:21:55 UTC 2019"
title = "Android Gradle Plugin 4.0でjava.timeがバックポートされるようになりました"
tags = ["android", "desugar", "agp"]
blogimport = true
type = "post"
draft = false
+++

[Java 8 library desugaring in D8 and R8](https://developer.android.com/studio/preview/features#j8-desugar)で、desugarが`A subset of java.time`に対応したとのことで、[ThreeTenABP](https://github.com/JakeWharton/ThreeTenABP)が置き換えられるのでは？と思い、ウキウキで試してみました。

環境は `com.android.tools.build:gradle:4.0.0-alpha01`になります。

## セットアップ

build.gradleに、coreLibraryDesugaringEnabledを追加します。

```groovy
compileOptions {
  sourceCompatibility JavaVersion.VERSION_1_8
  targetCompatibility JavaVersion.VERSION_1_8

  coreLibraryDesugaringEnabled true
}
```

これで完了です。

## java.timeのAPIを呼び出してみる

29, 21のエミュレーターで次のコードを試したところ、クラッシュすることなく、無事に実行することが出来ました！

```kotlin
// Instant API
val date = Date()
val instant = date.toInstant()
binding.instant.text = instant.epochSecond.toString()

// ZoneId API
val zoneId = ZoneId.systemDefault()
binding.zoneId.text = zoneId.id
println(instant.atZone(zoneId).dayOfMonth)
println(instant.atZone(zoneId).month)

// LocalDate API
val now = LocalDate.now()
binding.localDate.text = now.dayOfMonth.toString()
println(now.dayOfMonth)

// ZoneOffset API
val zoneOffset = ZoneOffset.ofHours(10)
println(zoneOffset)
```

## どのようにして実現しているか？

apkの中身を見たところ、`java.time`用のバックポートライブラリを準備しておいて、それをapkの中に組み込んでいるようでした。

以下、apkの中身になります。

<img src="/blog/android/agp/r8/desugar-time-apk.png" width="100%" />

`java.time` は `j$.time`に変換されていました。

また、`Date#toInstant`など、既存クラスに新しく追加された`date.time`のAPIも使えます。R8のコードを読んだところ、おそらく次のAPIが対応しています。

```
"retarget_lib_member": {
  "java.util.Calendar#toInstant": "java.util.DesugarCalendar",
  "java.util.Date#from": "java.util.DesugarDate",
  "java.util.Date#toInstant": "java.util.DesugarDate",
  "java.util.GregorianCalendar#from": "java.util.DesugarGregorianCalendar",
  "java.util.GregorianCalendar#toZonedDateTime": "java.util.DesugarGregorianCalendar"
},
```

このあたりのAPIが使えるのは便利そうです。

## ThreeTenABPへの置き換え

バックポートされた`j$.time`のクラスと、`ThreeTenABP`のクラスを比較したところ、ほとんどのクラスが同じだったので、パッケージパスを変えるだけで、問題なく置き換えが出来ると思います。
ただもちろん、パフォーマンス、バグなどの問題はある可能性があります。

## まとめ

- 新しく追加されたdesugarを使うと、低いminversionでも`java.time`が使えるようになります😃
- 簡単に使える!!
- ThreeTenABPの置き換えも簡単に出来そう
- ただ、正式4.0のリリースはまだ先...

今回の検証に用いたサンプルコードは [DesugarTimeFragment.kt](https://github.com/satoshun-android-example/AndroidStudioFeature/blob/master/app/src/main/java/com/github/satoshun/example/desugartime/DesugarTimeFragment.kt) にあります:D
