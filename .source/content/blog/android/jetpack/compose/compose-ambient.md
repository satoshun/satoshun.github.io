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

---

`Providers(exampleViewModelAmbient provides viewModel)`

Ambientに対して値を注入しています。この場合、定義した`exampleViewModelAmbient`に、ExampleViewModelのインスタンスを注入、セットしています。

---

`val viewModel = exampleViewModelAmbient.current`

注入した値は `.current` から取得することが出来ます。この場合、上記で生成したExampleViewModelのインスタンスの値を取得することが出来ます。

---

`ambientOf`で定義して、Providersで値をセットすることで、コンポーネントに値を渡すことが可能になります。

## 注意点

注意としては、Providersで囲っていない場合にエラーになる点です。例えば、次のように書くことは出来ません。

```kotlin
@Composable
fun ExampleApp() {
  val viewModel = ExampleViewModel()

  Providers(exampleViewModelAmbient provides viewModel) {
    ...
  }

  // ランタイムエラーになる!
  MyView()
}
```

直感的な挙動だと思います。

---

また、複数のProvidersで囲った場合は、直近のProvidersが有効になります。

```kotlin
fun ExampleApp() {
  val viewModel = ExampleViewModel()

  Providers(exampleViewModelAmbient provides viewModel) {

    val viewModel2 = ExampleViewModel()
    Providers(exampleViewModelAmbient provides viewModel2) {
      // viewModel2が渡される
      MyView()
    }
  }
}
```

## まとめ

- Ambientを使うことで、多くの中間コンポーネントがある場合に楽が出来る
  - とはいえ、一般的なデータの渡し方は引数から
- ライブラリから、ContextAmbient、CoroutineContextAmbientなどが提供されているので、それらの実装、書き方を参考にするのも良いと思います。

なにか間違っている点や、疑問があればTwitterなどからご指摘いただければ幸いです😃

ではでは。
