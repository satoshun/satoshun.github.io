+++
date = "Tue Feb 18 13:08:10 UTC 2020"
title = "Gradle: BOMを使って依存を指定する"
tags = ["gradle", "android", "bom"]
blogimport = true
type = "post"
draft = false
+++

Gradleの5から、Bill Of Materials(BOM)が使えるようになりました。これが、結構いいものだと思ったので紹介します。

ついでに[dependabot](https://dependabot.com/)の話もちょっとします。

## BOM?

[BOM](https://docs.gradle.org/5.0/userguide/managing_transitive_dependencies.html#sec:bom_import)を使うことで、 複数のライブラリのバージョンを省略することが出来ます。

ドキュメントの例にはspring-bootが挙げられています。

```groovy
dependencies {
  // import a BOM
  implementation platform('org.springframework.boot:spring-boot-dependencies:1.5.8.RELEASE')

  // define dependencies without versions
  implementation 'com.google.code.gson:gson'
  implementation 'dom4j:dom4j'
}
```

gson、dom4jのバージョンを省略していることが分かると思います。

Androidでよく使うライブラリでBOMに対応しているライブラリには、例えば次があります。

- [OkHttp](https://square.github.io/okhttp/)
- [Kotlin Coroutine](https://github.com/Kotlin/kotlinx.coroutines)
- [Firebase](https://firebase.google.com/docs/android/setup#firebase-bom)

例えば、OkHttpならこんな感じで書けます。

```groovy
dependencies {
   implementation platform("com.squareup.okhttp3:okhttp-bom:4.4.0")
   implementation "com.squareup.okhttp3:okhttp"
   implementation "com.squareup.okhttp3:logging-interceptor"
}
```

BOMを使うことで、関連ライブラリをまとめてアップデートすることが出来るのでとても便利です。

## dependabot?

直接はBOMに関係ないんですけど、最近、[dependabot](https://dependabot.com/)が便利だと自分の中で話題になっていて、これはライブラリのアップデートを自動でやってくれるbotになります。

例えば、こんな感じのPRを作ってくれます。

[Bump versions.retrofit from 2.7.0 to 2.7.1](https://github.com/satoshun-android-example/dependabot/pull/4)

---

{{< figure src="/blog/gradle/dependabot-pr.png" >}}

---

それでdependabotって、`versions.retrofit = '2.7.1'`って感じで、変数で定義すると検知できないって思っていたんですけど、
いろいろ試してみたら、普通に出来ました:D

BOMとdependabotは相性いいぞ！！って書こうと思ったんですけど、変数の場合でもアップデートを検知してくれたので、特に関係なかったです😅

とはいえ、わざわざ自前で変数定義する必要はなくなるので、BOMは便利です！

## まとめ

- BOMはいいぞ〜
- dependabotはいいぞ〜
