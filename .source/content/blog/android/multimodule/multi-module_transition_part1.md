+++
date = "Sat Dec 29 08:17:17 UTC 2018"
title = "todo"
tags = ["android", "gradle", "multimodule"]
blogimport = true
type = "post"
draft = true
+++

Androidのトレンドの1つにマルチモジュール構成があります。
マルチモジュールによるメリットとして、

- ビルド時間の短縮
- 依存関係を各モジュールに閉じ込めることでコードをクリーンに保つことが期待できる

などがあります。大規模なプロジェクトでは上記のメリットは大きいため、マルチモジュールに移行していくことになると思います。

この記事は、マルチモジュールにした際のActivity間の遷移について考えたいと思います。Part1では、遷移専用のモジュールを作る方法を考えてみます。

## 遷移専用のモジュールを作る

まず依存関係の構築の原則に、Circular Dependency、循環依存を作り出してはいけないというものがあります。

例えば、メイン画面とサブ画面の2画面があるとします。それらをメイン画面モジュール、サブ画面モジュールとして切り出します。

- メインではサブ画面が必要なのでサブモジュールに依存する
- サブではメイン画面が必要なのでメインモジュールに依存する

-- todo 図を書く --

これでは依存関係が壊れてしまうので駄目です。そこでDIP、依存関係逆転の原則を用います。
直接Activityに参照しているのが問題なので、各画面に遷移できる遷移用のインターフェースを定義することで解決を目指します。

そこで、

- メイン画面に遷移するメインルーターモジュール
- サブ画面に遷移するサブルーターモジュール

の2つのモジュールを作ります。

メインルーターモジュールでは次の遷移専用インターフェースを定義します。

```koltin
interface MainRouter {
  fun routeToMain(context: Context): Intent
}
```

そして、メインモジュールで実装します。また、今回はDaggerを使って依存を解決します。

```kotlin
internal class MainRouterImpl @Inject constructor() : MainRouter {
  override fun routeToMain(context: Context): Intent {
    return Intent(context, MainActivity::class.java)
  }
}

---

@Module
internal interface MainActivityModule {
  @Binds fun bindMainRouter(impl: MainRouterImpl): MainRouter
}
```

これで、使う側であるサブ画面は、メインモジュールに依存するのではなく、メインルーターモジュールに依存し遷移することが出来ます。

```koltin
class SubActivity : AppCompatActivity() {
  @Inject lateinit var router: MainRouter

  ...
}
```

これで、相互に遷移する画面だとしても循環参照になることなく解決することが出来ます！！

## 補足

いちいちルーターモジュールを作るのがめんどうなのであれば、共通のRouterインターフェースを作る方法もあります。

```kotlin
interface Router<T> {
  fun route(context: Context, params: T): Intent
}
```

実装は次のようになります。

```kotlin
---実装

internal class MainRouter2Impl @Inject constructor() : Router<Unit> {
  override fun route(context: Context, params: Unit): Intent {
    return Intent(context, MainActivity::class.java)
  }
}

---Daggerの設定

@Module
internal interface MainActivityModule {
  @Named("main")
  @Binds fun bindMain2Router(impl: MainRouter2Impl): Router<Unit>
}

---使用側

class SubActivity : AppCompatActivity() {
  @field:[Inject Named("main")] lateinit var router: Router<Unit>
  ...
}
```

DaggerのNamedアノテーションと組み合わせることでいい感じに共通Routerを作ることが出来ます。

## まとめ

- 相互に行き来したい画面があったときに、遷移専用のモジュールを作ることで循環参照を防ぐことが出来る
- 基本的に画面を含んだモジュールは遷移したいときがほとんどだと思うので、遷移専用のモジュールを作ることで無駄な依存を作ることを防ぐことが出来る
- モジュールがすごい増えるので微妙なアプローチかも😂😂😂

Part2ではDeeplinkやnavigationを絡めた遷移の方法について考えてみたいと思います😃
