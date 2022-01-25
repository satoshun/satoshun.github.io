+++
date = "Tue Jan 25 12:53:58 UTC 2022"
title = "Gradle: 様々なバージョン管理の方法"
tags = ["gradle", "android"]
blogimport = true
type = "post"
draft = false
+++

Android開発では、Gradleを使ってビルド環境を構築します。その中で、バージョンを管理する必要があります。
Gradleでは、いろいろなバージョンの管理方法があるので、色々と紹介したいと思います。

## 1. バージョン管理しない管理方法

パワー系の管理方法です。
小規模なプロジェクトや、個人プロジェクトなどは、シングルモジュールな構成であったりと、そもそもバージョンを管理する必要がない場合があります。

具体的には、build.gradle(.kts)にベタ書きします。

```groovy
dependencies {
  implementation "com.google.android.material:material:1.4.0"
  ...
}
```

バージョン管理をしない、バージョン管理方法です。これだと、マルチモジュール環境などで、辛くなるので進化させる必要があります。

## 2. dependencies.gradleに定義する

dependencies.gradleファイルを定義し、そこでバージョンを管理する方法です。
ファイル名はなんでもいいのですが、dependencies.gradleという命名が多いような気がします。

具体的には、dependencies.gradleに依存を定義し、それをbuild.gradleで読み込んで使います。

```groovy
// dependencies.gradle
ext {
  material = "com.google.android.material:material:1.4.0"
  ...
}
```

```groovy
// build.gradle
apply from: rootProject.file('dependencies.gradle')
```

```groovy
// app/build.gradle
dependencies {
  implementation rootProject.ext.kotlinStdlib
}
```

参考コード: https://github.com/JakeWharton/RxBinding/tree/version-one

また、わざわざdependencies.gradleにファイルを分けないで、build.gradleに直接書くことも出来ます。

```groovy
// build.gradle
buildscript {
  ext.versions = [
    "material": "com.google.android.material:material:1.4.0"
  ]
}
```

参考コード: https://github.com/JakeWharton/RxBinding


## 3. buildSrcを使う

buildSrcディレクトリを使い、バージョンの管理をする方法です。
GradleではbuildSrcは特別なディレクトリになっており、ビルド実行時に自動で中身を読み込みます。

具体的には、次のようになります。今回は、kotlinでバージョンを管理してみます。

```kotlin
// buildSrc/src/main/java/Dep.kt
package hoge.dependencies

object Dep {
  const val Material = "com.google.android.material:material:1.4.0"
  ...
}
```

```groovy
// app/build.gradle
import hoge.dependencies.Dep

dependencies {
  implementation Dep.Material
}
```

参考コード
- https://github.com/DroidKaigi/conference-app-2021/
- https://github.com/androidx/androidx

## 4. version catalogを使う

Gradleの比較的新しいバージョンで使うことが出来ます。
`gradle/libs.versions.toml` でtoml形式でバージョンを管理します。

```toml
# gradle/libs.versions.toml

[libraries]
material = "com.google.android.material:material:1.4.0"
...
```

```groovy
// settings.gradle
...
enableFeaturePreview("VERSION_CATALOGS")
```

```groovy
dependencies {
  implementation libs.material
}
```

version catalogでは、バージョン管理をする上で必要であろう機能が色々と提供されており、バージョン管理をいい感じにすることが出来ます。

参考コード: https://github.com/chrisbanes/tivi


## まとめ

今回、dependencies.gradle、buildSrc、version catalogを使って管理する方法を紹介しました。
結局どれがいいのっていう話ですが、Gradleが提供しているversion catalogが普及するんじゃないかなと思っています。
また、個人的にもversion catalogを良いと思っています。

しかし、version catalogによるバージョン管理は、Dependabotが対応していないなど、プロジェクトによっては導入が難しかったりするので、ケースバイケースだとは思います。
