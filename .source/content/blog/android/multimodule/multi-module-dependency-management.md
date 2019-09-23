+++
date = "Mon Sep 23 12:07:59 UTC 2019"
title = "Android マルチモジュール: ライブラリのバージョン管理について"
tags = ["android", "multimodule", "gradle"]
blogimport = true
type = "post"
draft = false
+++

マルチモジュールなアプリを作ることをテーマにいくつかの記事を書いていきたいと思っています。

まだ正確には決まっていないのですが、以下のような内容をまとめていこうと思っております。

- ライブラリのバージョン管理について
- マルチモジュール構築に役立つGradleの設定
- モノリシックなアプリからマルチモジュールへ
- 外部ライブラリとして切り出すタイミングを考える
- DFM、モジュール間の遷移方法
- モジュールの階層について
- ビルドの高速化について

---

今回は、ライブラリのバージョン管理について書いてきます。

## ライブラリのバージョン管理？

Android開発では、Gradleで外部ライブラリの依存を定義するのが一般的です。マルチモジュールプロジェクトの場合、外部ライブラリのバージョンを合わせるため、変数のような形で定義しておくと便利です。

変数の定義方法には、直接記述する方法を除くと、大きく2つの方法があります。

## 1. Gradleのextraプロパティを使う

[Androidの公式ドキュメント: Configure project-wide properties](https://developer.android.com/studio/build/gradle-tips.html#configure-project-wide-properties)で紹介されている方法です。
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

extプロパティにライブラリのバージョンをセットします。

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
    - [Androidの公式ドキュメント](https://developer.android.com/studio/build/gradle-tips.html#configure-project-wide-properties)で紹介されている方法
    - ノウハウが多い
    - Android StudioのProject Structureが使える (Project Structureについては最後に紹介します)
- デメリット
    - IDEによるコード補完が効かない

## 2. buildSrc + Kotlin

buildSrc以下にKotlinファイルを定義し、バージョンを管理することも出来ます。

例えば、[androidx: Dependencies.kt](https://android.googlesource.com/platform/frameworks/support/+/androidx-master-dev/buildSrc/src/main/kotlin/androidx/build/dependencies/Dependencies.kt)では、次のように定義しています。

```kotlin
const val ANDROID_GRADLE_PLUGIN = "com.android.tools.build:gradle:3.4.2"
const val ANDROIDX_TEST_CORE = "androidx.test:core:1.1.0"
const val ANDROIDX_TEST_EXT_JUNIT = "androidx.test.ext:junit:1.1.0"
const val ANDROIDX_TEST_EXT_KTX = "androidx.test.ext:junit-ktx:1.1.0"
const val ANDROIDX_TEST_MONITOR = "androidx.test:monitor:1.1.1"
const val ANDROIDX_TEST_RULES = "androidx.test:rules:1.1.0"
const val ANDROIDX_TEST_RUNNER = "androidx.test:runner:1.1.1"
...
```

Kotlinの定数として定義しています。

そして、次のように使います。

```groovy
dependencies {
    api(KOTLIN_STDLIB)
    androidTestImplementation(JUNIT)
    androidTestImplementation(TRUTH)
    androidTestImplementation(ANDROIDX_TEST_EXT_JUNIT)
    ...
}
```

メリット、デメリットをまとめると次になります。

- メリット
    - IDEによるコード補完が効く
- デメリット
    - extの方法に比べると一般的ではない
    - 現状Gradle KTSを使うと、Android StudioのProject Structureが使えない

## 結局どっちがいいのか？

安定しているのは、extraプロパティを使う方法だと思います。
buildSrc + Kotlinの方法はProject Structureが使えないのと、最近まで、コード補完がうまく効かない問題もあり、もう少し待ったほうが無難かもしれません。
ただ、どちらもクリティカルなものではないので、最終的には好みで決定すれば良いと思います。

## 運用について

Tips的に、運用に役立つ便利ライブラリ、ASの機能を紹介します。

### Gradle Versions Plugin

[Gradle Versions Plugin](https://github.com/ben-manes/gradle-versions-plugin)は、ライブラリにアップデートがあるかをチェックしてくれるライブラリです。超便利です。

### de.fayard.buildSrcVersions

[de.fayard.buildSrcVersions](https://github.com/jmfayard/buildSrcVersions)は、buildSrc以下に、いい感じにバージョン管理用のKotlinファイルを作ってくれるライブラリです。実際に使ったことはないのですが、超便利そうです。

### Android Studio Project Structure

Android Studioには[Project Structure](https://developer.android.com/studio/projects#ProjectStructure)は、各モジュールのライブラリへの依存を一覧で見ることが出来ます。アップデートがあるライブラリを知ることが出来ます。

## まとめ

- バージョン管理には、大きく2つの方法がある
    - Gradle extraプロパティを使う
    - buildSrc + Kotlinを使う
- 最終的には、プロジェクトの構成や好みなどを加味して決めればいいと思います

## リファレンス

- [OkHttp](https://square.github.io/okhttp/)
- [androidx: Dependencies.kt](https://android.googlesource.com/platform/frameworks/support/+/androidx-master-dev/buildSrc/src/main/kotlin/androidx/build/dependencies/Dependencies.kt)
- [Configure project-wide properties](https://developer.android.com/studio/build/gradle-tips.html#configure-project-wide-properties)
- [Gradle buildSrc](https://docs.gradle.org/current/userguide/organizing_gradle_projects.html#sec:build_sources)
