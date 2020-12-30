+++
date = "Wed Dec 30 08:53:33 UTC 2020"
title = "Android: 公式でAssisted Injectがサポートされそう"
tags = ["android", "dagger"]
blogimport = true
type = "post"
draft = false
+++

Assisted Injectのコードが公式のDaggerに入ったそうです。

https://twitter.com/JakeWharton/status/1344055984575160321

まだリリースされていないんですが、軽く試してみました。

## サンプルコード

初期値を受け取りたい `Counter`クラスがあるとして、それをAssistedを使って定義してみます。

```kotlin
class Counter @AssistedInject constructor(
  @Assisted private val initialValue: Int
) {
  val count = MutableStateFlow(initialValue)
}

@AssistedFactory
interface CounterFactory {
  fun create(initialValue: Int): Counter
}
```

これで定義は完了です。Factoryクラスを`AssistedFactory`アノテーションで、本体を`AssistedInject`と`Assisted`アノテーションを使って定義します。
[square/AssistedInject](https://github.com/square/AssistedInject) とは違い `AssistedModule` が必要なくなりました。

そして次のように使います。

```kotlin
@Inject lateinit var factory: CounterFactory

val counter = factory.create(100)
...
```

定義した `CounterFactory`をinjectして、 `Counter`インスタンスを生成しています。こちらはいつものDaggerの記法になります。

## まとめ

ついに公式にAssisted Injectionが入ります:D

サンプルコードは [satoshun/daggersample](https://github.com/satoshun-android-example/DaggerSample/blob/master/assisted/src/main/java/com/github/satoshun/example/dagger/Counter.kt)にあります。
