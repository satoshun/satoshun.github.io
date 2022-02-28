+++
date = "Sun Feb 27 11:42:50 UTC 2022"
title = "Gradle: Typesafe project dependencies"
tags = ["gradle", "android"]
blogimport = true
type = "post"
draft = false
+++

Type-safe project dependenciesの紹介ブログです。Gradle 7.4で試しています。

Gradleでマルチモジュールプロジェクトを構成すると、例えば、プロジェクトlibAへの依存は、次のようにう書くことが出来ます。

```kotlin
// app/build.gradle.kts

dependencies {
  implementation(project(":libA"))
  ...
}
```

この際に、projectで指定している `:libA` の部分は文字列なのでtypesafeではなく、補完などが効きにくいです。(最近のIntelliJ、Android Studioだと補完が効いたりします)

ここで、今回紹介する Type-safe project dependenciesを使うと、次のように書くことが出来ます。

```groovy
// settings.gradle
enableFeaturePreview("TYPESAFE_PROJECT_ACCESSORS")
```

```kotlin
// app/build.gradle.kts

dependencies {
  implementation(projects.libA)
  ...
}
```

projectsから、プロジェクトlibA参照出来るようになり、補完がいい感じに効きます。

## まとめ

Gradle 7.4段階では、まだFeature Previewですが、ktsと相性が良い技術になっています。
ktsを入れているようなプロジェクトの場合は、導入を検討するのも良いかもしれません。

## 参考

- https://docs.gradle.org/7.4/userguide/declaring_dependencies.html#sec:type-safe-project-accessors
