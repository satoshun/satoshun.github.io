+++
date = "Sun Mar 28 06:48:44 UTC 2021"
title = "Gradle: 新しいバージョン管理方法 Central declaration of dependenciesの紹介"
tags = ["gradle", "android"]
blogimport = true
type = "post"
draft = false
+++

Gradleの7.0-RC01に、Central declaration of dependenciesという機能が入りました。まだ、experimentalな機能で今後どうなるか分かりませんが、気になったので紹介します。

この記事を書くのに使ったサンプルコードは [github/satoshun-android-example](https://github.com/satoshun-android-example/Template) にあります。

## Central declaration of dependenciesが導入された背景

[Gradle 7.0 Release Notes](https://docs.gradle.org/7.0-rc-1/release-notes.html) に次のように書いてあります。

```text
There are a number of ways to share dependency versions between projects in multi-project builds. For example, users can declare versions or dependency coordinates directly in build scripts (in the ext block), external files (e.g dependencies.gradle), in buildSrcor even dedicated plugins. There wasn’t, however, any standard mechanism to do this which would combine the advantages of each approach.
```

既存のバージョン管理方法には、extを使う方法、buildSrcを使う方法などがありましたが、それらの利点を組み合わせた標準的な方法は無かったので、新しいバージョン管理方法を作った。ということらしいです。

## 簡単な使い方

`settings.gradle` にGroovy(or Kotlin)で書く方法と、tomlファイルに記述する方法が提供されています。この記事ではtomlでのやり方を紹介します。

まず、最初に、gradle/ ディレクトリ以下に、libs.versions.toml ファイルを作成します。そこに使うライブラリのバージョンを記述します。

```toml
[versions]
kotlin = "1.4.31"

[libraries]
kotlin = { module = "org.jetbrains.kotlin:kotlin-stdlib", version.ref = "kotlin" }
```

[versions] にバージョンを、[libraries] にライブラリの情報を記述します。

これで、定義は完了です。`build.gradle` から、libs.xxx のように参照出来るようになります。

```groovy
dependencies {
  implementation libs.kotlin
}
```

さらにCentralized dependency versionsでは、bundlesを使うことでライブラリをひとまとめにすることが出来ます。

```toml
[versions]
kotlin = "1.4.31"
kotlin_coroutine = "1.4.1"

[libraries]
kotlin-stdlib = { module = "org.jetbrains.kotlin:kotlin-stdlib", version.ref = "kotlin" }
kotlin-coroutine-core = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "kotlin_coroutine" }
kotlin-coroutine-ui = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-android", version.ref = "kotlin_coroutine" }

[bundles]
kotlin-android = ["kotlin-stdlib", "kotlin-coroutine-core", "kotlin-coroutine-ui"]
```

[budnles] には、[libraries] で定義した、ライブラリを指定できます。

`build.gradle` からは、libs.bundles.xxx のように参照します。

```groovy
dependencies {
  implementation libs.bundles.kotlinAndroid
}
```

bundlesを使うことで、複数のライブラリの記述をシンプルにすることが出来ました。マルチモジュールなプロジェクトの場合、同じようなライブラリの依存になることが多いと思うので、その際に便利に使うことが出来ると思います。

## まとめ

マルチモジュールなプロジェクトのバージョン管理方法に、新しいメカニズムが誕生しました。まだexperimentalなので、なんともいえないですが、bundlesは便利だなと思いました

より詳しい使い方を知りたい人は、[公式のドキュメント](https://docs.gradle.org/7.0-rc-1/userguide/platforms.html#sub:central-declaration-of-dependencies) を見て下さい。
