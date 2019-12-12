+++
date = "Mon Dec 31 06:19:39 UTC 2018"
title = "マルチモジュールの遷移について考える Part1"
tags = ["android", "multimodule", "gradle"]
blogimport = true
type = "post"
+++

Androidのトレンドの1つにマルチモジュール構成があります。
マルチモジュールによるメリットとして、

- ビルド時間の短縮
- 依存関係を各モジュールに閉じ込めることでコードをクリーンに保つことが期待できる

などがあります。大規模なプロジェクトでは上記のメリットは大きいため、マルチモジュールに移行していくことになると思います。

この記事は、マルチモジュールにした際のActivity間の遷移について考えたいと思います。目指すゴールとしては、

- 型安全にしたい、もしくはコンパイル時にチェックする機構が欲しい
- コード量を減らしたい

Part1では、遷移専用のモジュールを作る方法を考えてみます。

サンプルコード: [satoshun-android-example/MultiActivityRouterExample](https://github.com/satoshun-android-example/MultiActivityRouterExample)

## 遷移専用のモジュールを作る

まず依存関係の構築の原則に、Circular Dependency、循環依存を作り出してはいけないというものがあります。

例えば、メイン画面とサブ画面の2画面があり、それらの画面は相互に行き来するとします。それらをメイン画面モジュール、サブ画面モジュールとして切り出すと次のようになります。

- メインではサブ画面が必要なのでサブモジュールに依存する
- サブではメイン画面が必要なのでメインモジュールに依存する

---

{{< figure src="https://www.plantuml.com/plantuml/svg/SoWkIImgAStDuU8goIp9ILLutBpmSTEInysRTHytRNtSFEtvbDqlvovwic_kqxKpdixUpCMLd9zRa9-NcbUY40rN2r4Kgv1OhE0UwecY1CaGcBmH5nSNa5BGBSfCpoZHjOE8WGW5tPpKDAW85vT3QbuAq6K0" width="300" >}}

---

これでは循環参照になり、依存関係が壊れてしまうので駄目です。そこでDIP、依存関係逆転の原則を用います。
直接Activityを参照しているのが問題なので、各画面に遷移できる遷移用のインターフェースを定義することで解決を目指します。

そこで、

- メイン画面に遷移するメインルーターモジュール
- サブ画面に遷移するサブルーターモジュール

の2つのモジュールを作ります。

メインルーターモジュールでは次の遷移専用インターフェースを定義します。

```kotlin
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

```kotlin
class SubActivity : AppCompatActivity() {
  @Inject lateinit var router: MainRouter

  ...
}
```

最終的な依存図は次のようになります。

---

{{< figure src="https://www.plantuml.com/plantuml/svg/SoWkIImgAStDuU8goIp9ILLutBpmSTEInysRTHytRNtSFEtvbDqlvovwic_kqxKpdixUpCMLd9zRa9-NcbUY40rN2r4Kgv1OhE0UwebLoUFcrO-RzpnksWyaOGgDKLGYMGTJO8If09iv9bnSN41AGRUqGBV63c8oZ6y7LG0o3Kc12K80ge7B8JKl1HWG0000" width="300" >}}

---

これで、相互に遷移する画面だとしても循環参照になることなく解決することが出来ます😃

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
- 遷移用のモジュールが増える😂😂😂

Part2ではDeeplinkやnavigationを絡めた遷移の方法について考えてみたいと思います😃

サンプルコード: [satoshun-android-example/MultiActivityRouterExample](https://github.com/satoshun-android-example/MultiActivityRouterExample)
