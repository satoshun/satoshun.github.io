+++
date = "Mon Sep 23 12:07:59 UTC 2019"
title = "Android マルチモジュール: Gradleプロパティについて"
tags = ["android", "multimodule", "gradle"]
blogimport = true
type = "post"
draft = true
+++

マルチモジュールなアプリを作ることをテーマにいくつかの記事を書いていきたいと思っています。

マルチモジュール環境における便利なGradleプロパティについて紹介します。

## resourcePrefix

[resourcePrefix](https://google.github.io/android-gradle-dsl/current/com.android.build.gradle.LibraryExtension.html#com.android.build.gradle.LibraryExtension:resourcePrefix)は、リソースファイルのプレフィックスに制限を掛けるプロパティです。

例えば、次のように書くと、このモジュール内のリソースファイル（Layout、Drawableなど）は`home_`から始まる必要があります。

```groovy
android {
  resourcePrefix 'home_'
}
```

これを定義することで、リソースファイルがどのモジュールで定義されているかを、名前から予想することが可能になります。

## generateBuildConfigProvider

モジュールのBuildConfigの生成を無効にすることが出来ます。例えば、[uber/AudoDispose](https://github.com/uber/AutoDispose/blob/1.4.0/build.gradle#L229)では、次のように設定しています。

```groovy
project.android {
  libraryVariants.all {
    it.generateBuildConfigProvider.configure {
      it.enabled = false
    }
  }
}
```

僕の経験的に、BuildConfigが必要なモジュールは多くないので、不要なモジュールに対してつけておくと良いと思います。

