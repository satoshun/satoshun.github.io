+++
date = "Sat Apr 10 01:13:45 UTC 2021"
title = "Android: BuildConfigの生成をoffにする"
tags = ["android", "agp", "gradle"]
blogimport = true
type = "post"
draft = false
+++

buildFeaturesから、BuildConfigクラスの生成をするかどうかを設定できるようになりました。

`gradle.properties`から設定できます。

```
android.defaults.buildfeatures.buildconfig=false # defaultはtrue
```

マルチモジュール構成のプロジェクトでは、基本的にはBuildConfigを生成しないと思うので、BuildConfigの生成のデフォルト設定をoffにしておくと良いと思います。

モジュール単位で、生成をonにするには、`build.gradle`に次の記述をします。

```groovy
android {
  buildFeatures {
    buildConfig true
  }
}
```

buildFeaturesブロックから設定を与えることが出来ます。

## まとめ

マルチモジュール構成のプロジェクトの場合は、coreモジュールやappモジュールのみonにして、他をoffにするのが良さそう
