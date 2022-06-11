+++
date = "Sat Jun 11 03:03:47 UTC 2022"
title = "dev版のJetpack Composeを使って、新しいKotlinに対応する"
tags = ["android", "jetpack", "compose"]
blogimport = true
type = "post"
draft = false
+++

Jetpack Composeは、特定のKotlinのバージョンと結びついており、Kotlinをアップデートするには、対応するJetpack Composeのバージョンに合わせる必要があります。
しかし、Jetpack Composeは、やや遅れて最新のKotlinへのサポートが入るので、すぐに最新のKotlinを試せない現状があります。

ただ、devバージョンのJetpack Compose Compilerを指定することで、Kotlinを新しいバージョンに上げることが出来ます。

devバージョンのJetpack Compose Compilerは、下記のリンクから探すことができます。
- https://androidx.dev/storage/compose-compiler/repository

例えば、Kotlin 1.7.0に対応した、Jetpack Compose Compilerは次のように使うことができます。

```
// build.gradle
repositories {
  maven { url "https://androidx.dev/storage/compose-compiler/repository/" }
}

// app/build.gradle
android {
  composeOptions {
    kotlinCompilerExtensionVersion "1.2.0-dev-k1.7.0-53370d83bb1"
  }
}
```

これで、Kotlinを1.7.0に上げることが出来ます。

## 注意

前述したリンクにも書いてあるんですが、devバージョンなので不具合がある可能性が高いです。なので、あくまでテスト、サンプルでのみ使ったほうが無難だと思います。
