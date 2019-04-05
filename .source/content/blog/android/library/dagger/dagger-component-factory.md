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

基本的にはやれることはComponent.Builderクラスと変わりません。

例えば、以下の2つは本質的にはやっていることは変わりません。

```kotlin

@Component
interface AppComponent {
  val presenter: ArticlePresenter

  @Component.Factory
  interface Factory {
    fun create(
      @BindsInstance age: Int
    ): AppComponent
  }
}

val component = DaggerAppComponent
  .factory()
  .create(50)
val presenter = component.presenter
---

@Component
interface AppComponent {
  val presenter: ArticlePresenter

  @Component.Builder
  interface Builder {
    @BindsInstance fun bindInt(age: Int): Builder
    fun build(): AppComponent
  }
}

val component = DaggerAppComponent
  .builder()
  .bindInt(50)
  .build()
val presenter = component.presenter
```

このコードは、AppComponentにIntの値をBindsしています。コードは違いますが、やっていることは変わりません。
コード以外の違いとしては、statelessかどうかという点です。
Builderはセッターメソッドを使ってフィールドの状態を変えていきますが、Factoryはcreateメソッドの引数に必要な値を渡します。

## ユースケース

Factoryのユースケースを考えます。この機能はそもそも[Feature request: factory method in components for assisted injection](https://github.com/google/dagger/issues/935)で話されていたのものです。次のコードを解決したいモチベーションがあります。

```java
class ArticlePresenter {
	...
	ArticlePresenter(long articleId, ArticleService articleService) {
		...
	}
}
```

```kotlin
fun AppComponent.Factory.createPresenter(
  name: String,
  age: Int
): ArticlePresenter {
  return create(AppModule(name), age).presenter
}
```

##

- AutoFactory
- AssistedInject
