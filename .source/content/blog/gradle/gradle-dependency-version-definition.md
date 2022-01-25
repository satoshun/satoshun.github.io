+++
date = "Mon Jan 24 12:59:44 UTC 2022"
title = "Gradle: 様々なバージョン管理の方法"
tags = ["gradle", "android"]
blogimport = true
type = "post"
draft = true
+++

この記事を書いている段階で、Gradleの最新バージョンは7.3.3になり、いろいろなバージョン管理の方法が生まれたので、
どんなバージョン管理方法があるかを紹介したいと思います。


## 1. バージョン管理しない

パワー系の管理方法です。
小規模なプロジェクトや、個人プロジェクトなどは、シングルモジュールな構成であったりと、そもそもバージョン管理をする必要がない場合があります。

具体的には、build.gradle(.kts)にベタ書きします。

```groovy
dependencies {
  implementation "com.google.android.material:material:1.4.0"
  ...
}
```

バージョン管理をしない、バージョン管理方法です。

## 2. dependencies.gradleに定義する

dependencies.gradleファイルを定義し、そこでバージョンを管理する方法です。
ファイル名はなんでもいいのですが、dependencies.gradleという命名が多いような気がします。

具体的は、dependencies.gradleに依存を定義し、それをbuild.gradleで読み込んで使います。

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

参考
- https://github.com/JakeWharton/RxBinding/tree/version-one

また、わざわざdependencies.gradleに分けないで、build.gradleに直接バージョン定義を書くことも出来ます。

```groovy
buildscript {
  ext.versions = [
    "material": "com.google.android.material:material:1.4.0"
  ]
}
```

## 3. buildSrcを使う

buildSrcディレクトリ内で、バージョンの管理をする方法です。
GradleにとってbuildSrcは特別なディレクトリになっており、実行時に自動で中身を読み込みます。

今回の例では、kotlinでバージョンを管理してみます。

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

参考
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

version catalogでは、バージョン管理をする上で必要であろう機能が最初から提供されています。

参考
- https://github.com/chrisbanes/tivi

