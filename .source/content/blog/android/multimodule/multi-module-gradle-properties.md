+++
date = "Mon Nov 11 00:50:18 UTC 2019"
title = "Android マルチモジュール: Gradle周りで便利だと思う設定"
tags = ["android", "multimodule", "gradle"]
blogimport = true
type = "post"
draft = false
+++

マルチモジュールなアプリを作ることをテーマにブログを書いていこうの、2本目です。

1本目はこちらになります。

- [Android マルチモジュール: ライブラリのバージョン管理について](https://satoshun.github.io/2019/09/multi-module-dependency-management/)

今回は、マルチモジュール環境における、Gradle周りの便利であろう設定について、次の4つを紹介します。

- モジュール内のリソース名にルールを持たせる
- BuildConfigを作らない
- モジュール内でProGuard/R8の設定をする
- Rファイルを小さく保つ

## モジュール内のリソース名にルールを持たせる

[resourcePrefix](https://google.github.io/android-gradle-dsl/current/com.android.build.gradle.LibraryExtension.html#com.android.build.gradle.LibraryExtension:resourcePrefix)は、リソース名のプレフィックスにルールを設けるプロパティです。

例えば、次のように書くと、このモジュール内のリソース（レイアウト、Drawable、Stringなど）は`home_`から始まる必要があります。

```groovy
// build.gradle
android {
  ...
  resourcePrefix 'home_'
}

// strings.xml
<resource>
  // home_から始まる必要がある
  <string name="hoge_app_name">Hoge</string>
</resource>
```

これを定義することで、名前のコンフリクトを防ぐことが出来ます。また、名前からリソースがどのモジュールで定義されているかを推測すること出来ます。


## BuildConfigを作らない

モジュールのBuildConfigの生成を無効にすることが出来ます。例えば、[uber/AudoDispose](https://github.com/uber/AutoDispose/blob/1.4.0/build.gradle#L229)では、次のように設定しています。

```groovy
// build.gradle
project.android {
  libraryVariants.all {
    it.generateBuildConfigProvider.configure {
      it.enabled = false
    }
  }
}
```

モジュールで、BuildConfigが必要になることは稀なので、基本つけておくと良いと思います。


## モジュール内でProGuard/R8の設定をする

[consumerProguardFiles](https://developer.android.com/studio/projects/android-library#Considerations)を使うことで、モジュールのProguard/R8の設定を定義することが出来ます。

例えば、次のように使います。

```groovy
// build.gradle
android {
  defaultConfig {
    consumerProguardFiles 'consumer-rules.pro'
  }
}

// consumer-rules.pro
-keep class * implements com.google.gson.TypeAdapterFactory
```

このように書くことで、ProGuardを有効にしてビルドした時に、`consumer-rules.pro`の設定も考慮して、コード最適化を行ってくれます。


## Rファイルを小さく保つ

R.javaは依存関係にあるモジュールのR.javaを、マージしていくような動作をするため、マルチモジュール環境だと各モジュール内で生成されるR.javaのサイズが馬鹿にならないことがあります。そこで、gradle.propertiesに次の設定をすることで、モジュールのR.javaのサイズを抑えることが出来ます。

```groovy
// gradle.properties
android.namespacedRClass=true
```

[Jake Wharton/Twitter](https://twitter.com/JakeWharton/status/1032396431787794432)に書いてあるのですが、debugビルドで約20%のフィールドを削減することが出来たようです。

注意としては、各モジュールで、マージされた大きなR.javaが生成されなくなるため、次のようにimport文を書き直さなければいけない点です。

```kotlin
import com.jakewharton.sdksearch.roboto.R as RobotoR
```
[SdkSearch/SearchUiBinder](https://github.com/JakeWharton/SdkSearch/blob/abb9ee2845382fd8448fe4831d1911a01c1976b2/search/ui-android/src/main/java/com/jakewharton/sdksearch/search/ui/SearchUiBinder.kt#L21)

また、この設定は、experimentalなフラグなので、今後挙動が変わる可能性があります。


## まとめ

- ざっと4つの設定を紹介しました😃
- 他に何かあれば教えていただけると幸いでございます:D
