+++
date = "Sun Sep 22 11:55:11 UTC 2019"
title = "Android マルチモジュール: ライブラリのバージョン管理について"
tags = ["android", "multimodule", "gradle"]
blogimport = true
type = "post"
thumbnail = "https://bit.ly/2DqyKVb"
draft = true
+++

マルチモジュールなアプリを作ることをテーマにいくつかの記事を書いていきたいと思っています。

まだ正確には決まっていないのですが、以下をまとめていこうと考えています。

- ライブラリのバージョン管理について
- マルチモジュール構築に役立つGradleの設定
- モノリシックなアプリからマルチモジュールへ
- 外部ライブラリとして切り出すタイミングを考える
- DFM、モジュール間の遷移方法
- モジュールの階層構造について
- ビルドの高速化について

---

最初は、ライブラリのバージョン管理について書いてきます。

## ライブラリのバージョン管理？

Android開発では、Gradleを使い、外部ライブラリの依存を定義するのが一般的です。マルチモジュールなプロジェクトの場合、外部ライブラリのバージョンを合わせるため、設定ファイルを作り、変数のような形で定義しておくのが良いとされています。

設定ファイルの定義方法には、大きく2つの方法があります。

## 1. Gradleのextraプロパティを使う

[公式のドキュメント](https://developer.android.com/studio/build/gradle-tips.html#configure-project-wide-properties)で紹介されている方法です。
extにバージョンを定義します。

例えば、[OkHttp](https://square.github.io/okhttp/)では、次のように定義しています。

```groovy
buildscript {
  ext.versions = [
      'animalSniffer': '1.17',
      'assertj': '3.11.0',
      'bouncycastle': '1.62',
      'brotli': '0.1.2',
  ...

  ext.deps = [
      'picocli': "info.picocli:picocli:${versions.picocli}",
      'android': "org.robolectric:android-all:9-robolectric-4913185-2",
      'animalSniffer': "org.codehaus.mojo:animal-sniffer-annotations:${versions.animalSniffer}",
      'assertj': "org.assertj:assertj-core:${versions.assertj}",
      'bouncycastle': "org.bouncycastle:bcprov-jdk15on:${versions.bouncycastle}",
      'brotli': "org.brotli:dec:${versions.brotli}",
  ...
```

そして、次のように使います。

```groovy
dependencies {
  api deps.okio
  api deps.kotlinStdlib
  compileOnly deps.android
  compileOnly deps.conscrypt
  ...
```

メリット、デメリットをまとめると次になります。

- メリット
    - ノウハウが多い
    - 公式で紹介されている方法
    - ASのProject Structureが使える (ASのProject Structureについては最後に紹介します)
- デメリット
    - IDEによる補完が効かない

## 2. buildSrc + Kotlin

Kotlinを使って、
