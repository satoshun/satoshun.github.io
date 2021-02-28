+++
date = "Sun Feb 28 07:53:27 UTC 2021"
title = "Paparazziを使って、実機を使わずにsnapshotを取る"
tags = ["android", "test"]
blogimport = true
type = "post"
draft = true
+++

今回は、[Paparazzi](https://github.com/cashapp/paparazzi)の使い方について紹介します。

また、今回の記事のサンプルコードは [github/satoshun/paparazzi](https://github.com/satoshun-android-example/paparazzi)にあります。

## Paparazzi?

square社がメンテしているライブラリです。これを使うことで、実機を使わずにスナップショットを取ることが出来ます。

https://github.com/cashapp/paparazzi


## 使い方

トップのbuild.gradleにGradleプラグインをセットします。

```groovy
buildscript {
  repositories {
    mavenCentral()
    google()
  }
  dependencies {
    classpath 'app.cash.paparazzi:paparazzi-gradle-plugin:0.7.0'
  }
}
```

ライブラリのbuild.gradleにプラグインを適用します。

```groovy
apply plugin: 'app.cash.paparazzi'
```

これで準備は完了です。

次に、サンプルのレイアウトファイルとして、`lib_item.xml` を準備したので、これをPaparazziを使ってスナップショットを取っていきます。

```kotlin
class LayoutTest {
  @get:Rule
  var paparazzi = Paparazzi(deviceConfig = DeviceConfig.NEXUS_5)

  @Test
  fun sample() {
    val launch = paparazzi.inflate<LinearLayout>(R.layout.lib_item)
    paparazzi.snapshot(view = launch)
  }
}
```

Paparazziには、Junit4 Ruleがあるので、それを使います。引数から、デバイスを指定します。今回は、Nexus5を指定しています。

次に、Paparazziクラスに定義されている、inflate、snapshotメソッドをコールします。これでテストを実行すると、次のようなスナップショット(png)が作成されます。

{{< figure src="/blog/android/test/paparazzi-sample-screenshot.png" >}}

実機を使わずに、スクリーンショットを取ることが出来ました。


## まとめ

結構お手軽に使うことが出来ました。まだAPIが安定していない?そうなので、プロダクションに入れるのは時期尚早かもしれませんが、安定してきたら、試してみるのも良さそうです(こなみかん)
