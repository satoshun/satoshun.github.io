+++
date = "Sun Apr  7 06:04:22 UTC 2019"
title = "Dagger 2.22にFactoryクラスが導入されました"
tags = ["android", "dagger", "factory", "di"]
blogimport = true
type = "post"
draft = false
+++

Dagger 2.22からComponent.Factoryクラスが導入されました。この記事では、簡単な使い方とユースケースを見ていきたいと思います。

## Component.Factoryとは?

実は、Component.FactoryでやれることはComponent.Builderクラスとほぼ変わりません。

例えば、次のArticlePresenterインスタンスを生成する2つのコードは本質的にやっていることは同等です。

```kotlin
// Factoryを使う場合
@Component
interface AppComponent {
  val presenter: ArticlePresenter

  @Component.Factory
  interface Factory {
    fun create(
      @BindsInstance id: Long
    ): AppComponent
  }
}

val component = DaggerAppComponent
  .factory()
  .create(50)
val presenter = component.presentere

---

// Builderを使う場合
@Component
interface AppComponent {
  val presenter: ArticlePresenter

  @Component.Builder
  interface Builder {
    @BindsInstance fun bindId(id: Long): Builder
    fun build(): AppComponent
  }
}

val component = DaggerAppComponent
  .builder()
  .bindId(50)
  .build()
val presenter = component.presenter
```

この2つのコードは、AppComponentにIntをBindsしています。渡し方、定義の仕方は違いますが、やっていることは変わりません。

ただし、Factoryを使うパターンはstatelessです。
Builderはセッターメソッドを使ってフィールドの状態を変えていきますが、Factoryはcreateメソッドから必要な値を渡します。

## ユースケース

次に、Factoryのユースケースを考えます。そもそもこの機能は [Feature request: factory method in components for assisted injection](https://github.com/google/dagger/issues/935) を解決したいモチベーションがあります。

例えば、次のコードをDaggerで解決したい。

```java
class ArticlePresenter {
	...
	ArticlePresenter(long articleId, ArticleService articleService) {
		...
	}
}
```

longの値を後から決めたいときには、今までだと

- AssistedInject
- AutoFactory

のどちらかを使っていました。これに、今回のdagger.Factoryが加わりました。
dagger.Factoryを使うと、このコードを解決することが出来ます！

ただし、現状のdagger.Factoryだと多くのボイラープレートコードが必要です。

```kotlin
@Component
interface AppComponent {
  val presenter: ArticlePresenter

  @Component.Factory
  interface Factory {
    fun create(
      @BindsInstance id: Long
    ): AppComponent
  }
}

val component = DaggerAppComponent
  .factory()
  .create(50)
```

AssistedInjectとAutoFactoryを使えば、ここらへんのボイラープレートコードを緩和することが出来ます。
なので、このようなパターンのコードがよく出てくるようなプロジェクトは、AssistedInject or AutoFactoryの導入を検討しても良いと思います。pure Daggerで運用したい、もしくはそこまでこのパターンが出てこないなら、dagger.Factoryを使うのが良いかと思います。

## まとめ

- dagger.Factoryが導入された
  - ただし、AutoFactoryやAssistedInjectのほうが多機能
- サンプルコードは[satoshun/DaggerFactoryExample](https://github.com/satoshun-android-example/DaggerFactoryExample)にあります😃

---

内容におかしい点や、もっとこうしたほうがいいよって！！いうのがあれば[Twitter](https://twitter.com/stsn_jp)などから教えてもらえればとても嬉しいです😊
