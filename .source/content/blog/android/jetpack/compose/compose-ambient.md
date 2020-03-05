+++
date = "Thu Mar  5 10:38:47 UTC 2020"
title = "Jetpack Compose: Ambientを使ってインスタンスを注入、取得する"
tags = ["android", "jetpack", "compose"]
blogimport = true
type = "post"
draft = false
+++

こんにちわっふる🍰

今回は、ComposeにあるAmbientを説明したいと思います。この記事はdev06で検証しています。

## 前提・課題

一般的に、Composableなコンポーネントにデータを渡すとき、関数の引数から渡します。

```kotlin
@Composable
fun MyView(someData: Data) {
  ...
}
```

これがシンプル かつ 分かりやすい方法です。
しかし、中間に多くのComposableなコンポーネントがある場合、全てのコンポーネントの引数でデータを受け取り、さらに渡す必要があります。これは少々面倒で冗長です。

ここで、Ambientを使うとすっきりと書くことが出来ます。

## Ambientを使う

まず、最初にAmbientの定義をします。Ambientの定義は`ambientOf`メソッドから行えます。

今回は例として、ExampleViewModelをAmbientで定義して、コンポーネントから使用します。

まず、定義を行います。

```kotlin
val exampleViewModelAmbient = ambientOf<ExampleViewModel>()
```

これで定義は完了です。`ambientOf`メソッドを呼び出すだけです。

次に、これをコンポーネントから使ってみます。

```kotlin
val exampleViewModelAmbient = ambientOf<ExampleViewModel>()

@Composable
fun ExampleApp() {
  val viewModel = ExampleViewModel()

  Providers(exampleViewModelAmbient provides viewModel) {
    MyView()
  }
}

@Composable
private fun MyView() {
  val viewModel = exampleViewModelAmbient.current
  ...
}
```

これで完了です。コードを説明してきます。

Providers(exampleViewModelAmbient provides viewModel)

これは、Ambientに対して値を注入しています。この場合、`exampleViewModelAmbient`に、ExampleViewModelのインスタンスを注入しています。

val viewModel = exampleViewModelAmbient.current

注入した値は `.current` で取得することが出来ます。この場合、上記で生成した、ExampleViewModelのインスタンスを取得することが出来ます。

注意としては、Providersで囲っていない場合にエラーになる点です。例えば、次のように書くことは出来ません。

```
@Composable
fun ExampleApp() {
  val viewModel = ExampleViewModel()

  Providers(exampleViewModelAmbient provides viewModel) {
    ...
  }

  // ランタイムエラーになる!
  MyView()
}

@Composable
private fun MyView() {
  val viewModel = exampleViewModelAmbient.current
  ...
}
```

直感的な挙動だと思います。
