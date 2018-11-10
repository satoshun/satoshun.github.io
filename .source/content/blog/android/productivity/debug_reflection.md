+++
date = "2018-11-10T00:00:00Z"
title = "Android: デバッグ時にMoshi、Daggerリフレクションライブラリを使いビルド時間を短縮する"
tags = ["android"]
blogimport = true
type = "post"
+++

ビルド時間の短縮は開発効率を上げる大きな要素です。
極力kapt（アノテーションプロセッサー）を使わなくすることで、ビルド時間を短縮出来ます。
アノテーションプロセッサーを使ったライブラリは、アノテーションプロセッサーを使わない、
リフレクションを用いたデバッグ用の機能を提供していることがあります。

今回は、MoshiとDaggerリフレクションライブラリの紹介をします。
Daggerリフレクションは絶賛開発中 + 公式ではないので、機能がかなり制限されている、どういう未来が待っているかわからない点に注意してください。

検証に用いた[サンプルコードはここ](https://github.com/satoshun-android-example/DebugReflectExample)にあります。

## Moshiリフレクション

Moshiにはmoshi-codegenと呼ばれる、アノテーションプロセッサーでコード生成してくれるライブラリがあります。
これは実行時のパフォーマンスには優れているのですが、アノテーションプロセッサーを使っているため、ビルドに時間がかかってしまいます。

そこで、Moshiではmoshi-reflectionと呼ばれるライブラリを提供しており、これはアノテーションプロセッサーを使うことなく、
moshi-codegenと同等の機能を提供してくれます。
ただし、moshi-reflectionは内部でリフレクションを使っているため、実行時のパフォーマンスには優れていません。あくまでデバッグ用、という立ち位置だと思います。

デバッグ時にmoshi-reflectionを使い、リリース時にmoshi-codegenを使うことで、ビルド速度と実効速度の天秤を勝ち取ることが出来ます。

具体的には、デバッグ時、リリース時に`build.gradle`で指定するライブラリを、Moshiに登録するAdapterを変更します。

```groovy
// bulid.gradle

implementation "com.squareup.moshi:moshi:1.8.0"
debugImplementation "com.squareup.moshi:moshi-kotlin:1.8.0"
kaptRelease "com.squareup.moshi:moshi-kotlin-codegen:1.8.0"
```

```kotlin
// debug時
fun createMoshiBuilder() = Moshi.Builder()
    .add(KotlinJsonAdapterFactory()) // KotlinJsonAdapterFactoryを指定する
    .build()

// release時
fun createMoshiBuilder() = Moshi.Builder()
    .build()
```

このように書くことで、リリース時のみkaptを実行するようになります。

## Daggerリフレクション

これはSdkSearch内で開発が行われているライブラリです。有名なAndroidエンジニアであるJake Whartonさんが開発をしています。

これもMoshiと同様に、リフレクションを使うことでアノテーションプロセッサーを極力使わないようにしています。
現状、最低限のクラスはアノテーションプロセッサーで作成するようになっているので、完全除去というわけではありません。
例えば、アノテーションプロセッサーで`DaggerAppComponent`は作られます。

最初にも書いたのですが、Daggerリフレクションは絶賛開発中で、Scope、セッターインジェクションなど、多くの機能が使えない状態です。
ここが揃ってくればデバッグ時にはDaggerリフレクションを使うことで、ビルド時間の短縮が可能になってくると思います。

具体的なコードは、以下のようになります。
Mavenにアップデートされていないので、SdkSearchからコードをコピーして使っています。

```groovy
implementation "com.google.dagger:dagger:2.18"
debugImplementation project(':dagger-reflect:reflect') // SdkSearchからコピー
kaptDebug project(':dagger-reflect:reflect-compiler') // SdkSearchからコピー
kaptRelease "com.google.dagger:dagger-compiler:2.18"
```

デバッグ時にはDaggerリフレクションを使い、リリース時にはdagger-compilerを使っています。
コードは全く修正することなく、ビルド時間の短縮が可能になります。

## まとめ

- デバッグ時にはkapt、アノテーションプロセッサーを抑制しビルド時間を短縮しよう😃
  - butterknifeもreflectionを使うことができる
- Daggerリフレクションには期待しかない😄
  - 何か進みがあったらまたまとめようと思います

今回の記事を検証するために作った[サンプルコードはここ](https://github.com/satoshun-android-example/DebugReflectExample)にあります。

## 参考

- https://github.com/JakeWharton/SdkSearch
- https://github.com/square/moshi
- https://jakewharton.com/helping-dagger-help-you/
