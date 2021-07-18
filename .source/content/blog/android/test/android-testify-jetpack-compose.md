+++
date = "Sun Jul 18 07:44:46 UTC 2021"
title = "Testifyを使って、Jetpack Composeのスクリーンショットテストをする"
tags = ["android", "test", "compose"]
blogimport = true
type = "post"
draft = false
+++

(この記事は、ほぼ、[Testify](https://github.com/Shopify/android-testify]) のREADMEに書いてあることをやっているだけです)

[Testify](https://github.com/Shopify/android-testify]) を使って、Jetpack Composeのスクリーンショットテストをしてみます。

## セットアップ

READMEに記してある通り、build.gradleに下記の設定を追加します。

```groovy
dependencies {
  classpath "com.shopify.testify:plugin:1.1.0-beta2"
}

apply plugin: 'com.shopify.testify'
```

これで完了です。

## テストを書いてみる

JUnit4ベースの`ScreenshotRule` と、`ScreenshotInstrumentation` アノテーションを使います。

```kotlin
class AppActivityTest {
  @get:Rule val rule = ScreenshotRule(TestActivity::class.java)

  @ScreenshotInstrumentation
  @Test
  fun test() {
    rule.activity.runOnUiThread {
      rule.activity.setContent { AppContent() } // AppContent関数は適当なComposeコンポーネント
    }
    Thread.sleep(500) // 一応
    rule.assertSame()
  }
}
```

こんな感じでテストを記述できます。`ScreenshotRule` を使って、適当なActivityを起動する。それに、Jetpack Composeコンポーネントをセットする感じです。

次にテストの実行をします。まず下準備として、baseとなる画像を生成します。

```
./gradlew screenshotRecord
```

これでスクリーンショット(pngファイル)がローカルにコピーされるので、Gitなどにコミットしておきます。

次に、スクリーンショットテストを行います。今回は、連続でコマンドを実行しているので成功します。

```
./gradlew screenshotTest
```

```
------------------------------------------------------------
Run the Testify screenshot tests
------------------------------------------------------------


com.github.satoshun.example.AppActivityTest:.

Time: 2.264

OK (1 test)
```

失敗したケースも見てみたいので、AppContent関数の中身を変えて実行してみます。

```
------------------------------------------------------------
Run the Testify screenshot tests
------------------------------------------------------------


com.github.satoshun.example.AppActivityTest:
Error in test(com.github.satoshun.example.AppActivityTest):
com.shopify.testify.internal.exception.ScreenshotIsDifferentException:
...
```

無事に失敗しました。


## まとめ

Jetpack Compose時代には、Layout時代に比べて、コンポーネントの生成に必要なパラメータが関数に明示的に切り出されていたり、
関数の組み合わせによる、少し複雑なコンポーネントのテストがしやすくなると思います。
そうなったときには、Testifyのようなローカルのみで完結させることが出来る、テストライブラリをとりあえず試してみるのはいいのかなと思いました。
