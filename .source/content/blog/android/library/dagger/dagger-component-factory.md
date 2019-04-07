+++
date = "Thu Apr  4 23:28:56 UTC 2019"
title = "TODO"
tags = ["android", "dagger", "factory", "di"]
blogimport = true
type = "post"
draft = true
+++

Dagger 2.22からComponent.Factoryクラスが導入されました。この記事では、使い方とユースケースを見ていきたいと思います。

## Component.Factoryとは?

Component.FactoryでやれることはComponent.Builderクラスとほぼ変わりません。

例えば、以下のArticlePresenterインスタンスを生成する、2つのコードは本質的にやっていることは同じです。

```kotlin
@Component
interface AppComponent {
  val presenter: ArticlePresenter

  @Component.Factory
  interface Factory {
    fun create(
      @BindsInstance id: L
    ): AppComponent
  }
}

val component = DaggerAppComponent
  .factory()
  .create(50)
val presenter = component.presentere

---

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

この2つのコードは、AppComponentにIntのインスタンスをBindsしています。渡し方は違えど、やっていることは変わりません。

書き方以外の違いは、statelessかどうかというところです。
Builderはセッターメソッドを使ってフィールドの状態を変えていきますが、Factoryはcreateメソッドから必要な値を渡します。

## ユースケース

Factoryのユースケースを考えます。この機能はそもそも[Feature request: factory method in components for assisted injection](https://github.com/google/dagger/issues/935)を解決したいモチベーションがあります。

例えば、次のコードを解決したい。

```java
class ArticlePresenter {
	...
	ArticlePresenter(long articleId, ArticleService articleService) {
		...
	}
}
```

ここで、longの値を後から決めたいときに、今までだと

- AssistedInject
- AutoFactory

のどちらかを使っていました。これに、dagger.Factoryが加わりました。
ただ、現状のdagger.Factoryだと多くのボイラープレートコードが必要です。

```kotlin
// 定義側
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

// 呼び出し側
val component = DaggerAppComponent
  .factory()
  .create(50)
```

AssistedInjectとAutoFactoryを使えば、ここらへんのボイラープレートコードを緩和することが出来ます。
なので、このようなパターンのコードがよく出てくるようなプロジェクトは、AssistedInject or AutoFactoryの導入を検討しても良いと思います。

## まとめ

- dagger.Factoryが導入された
  - ただし、AutoFactoryやAssistedInjectのほうが多機能

---

内容におかしい点や、もっとこうしたほうがいいよって！！いうのがあれば[Twitter](https://twitter.com/stsn_jp)などから教えてもらえればとても嬉しいです😊
