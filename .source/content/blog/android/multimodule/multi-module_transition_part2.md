+++
date = "Fri Feb 15 00:14:56 UTC 2019"
title = "マルチモジュールの遷移について考える Part2"
tags = ["android", "multimodule", "gradle", "navigation"]
blogimport = true
type = "post"
thumbnail = "https://bit.ly/2TPjiZz"
+++

マルチモジュール遷移方法Part2です。[Part1](https://satoshun.github.io/2018/12/multi-module_transition_part1/)はここになります😌

Part2では、Navigation Componentを使うパターンを考えてみます。今回はDynamic Feature(DFM)には触れません。いわゆる一般的なライブラリモジュールでの遷移になります。

また、今回の検証に用いたコードは[satoshun/MultiModuleNavigationComponentExample](https://github.com/satoshun-android-example/MultiModuleNavigationComponentExample)にあります。

## モジュール構成について

細かい実装に入る前に、全体的なモジュール構成を説明します。今回はappモジュールがトップにあり、2つのfeatureモジュールがあるとします。

<img src="https://www.plantuml.com/plantuml/img/SoWkIImgAStDuU8goIp9ILLutBpeSTEEnyrB7pVlUToy-kdipLnS1Od9sOdfgGfAYGK5yMcfYIMbHQbA2jLS2WhHG95O45sKNrgIMXJBLOkakhWqoH1DEKWe5iQ8nw7925EJ4KoJ4RAcvFpSWloyrBmIi3lGN1wha5Yi01H6LWNHYqqXH0PPxUF6kOyRrptPFGqi3t8likpBnktFb-z-tBJaSVFcnqtxmIPDVToq7CHesWdN4a-4kKQacmiB1Iuka2KAkdOebe4KGCKG2e4XeQ2Rab-U1rCC3MDq2IEi4Z1Jk20Cg7WDghrOv13sEwJcfG2J6G00" width=600>

各featureモジュールでは遷移用インターフェースを持っており、それを用いて他のfeature画面へ遷移をします。遷移用インターフェースの実装はapp内のrouterモジュールで行います。

このモジュール構成のポイントは、各featureモジュール内で自身が使う遷移インターフェースを定義し、appがそのインターフェースの実装を行う点です。このようにすることで、feature間で直接の依存を持つことを防ぐことができます。これは循環依存を避けるためです。

では、実装に入っていきます。今回はDagger2を使って実装をします。

## featureモジュール側の遷移用インターフェースの定義

前述の図の通り、各featureモジュール内で遷移用のインターフェースを定義します。ここでは、featureモジュール内で使用するインターフェースを定義します。

main画面からsub1画面に移動したいとします。次のようなインターフェース定義になります。

```kotlin
interface MainModuleRouter {
  // sub1画面へ移動する
  fun routeToSub1()
}
```

Mainモジュール用のインターフェースなので、`MainModuleRouter`という名前にし、sub1画面へ遷移するためのメソッドを定義しています。

そしてこのインターフェースを、MainFragmentで使います。

```kotlin
class MainFragment : Fragment() {
  @Inject lateinit var moduleRouter: MainModuleRouter

  ...

  override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)

    view.findViewById<View>(R.id.route).setOnClickListener {
      moduleRouter.routeToSub1()
    }
  }
}
```

これでfeatureモジュールでの遷移用インターフェースの定義は完了です。

次にこのインターフェースの実装をします。

## routerモジュール側の遷移用インターフェースの実装

今回は、遷移用インタフェースの実装をrouterモジュールで行います。まずは、Navigation Componentを用いて、Graphを作ります。

```xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:app="http://schemas.android.com/apk/res-auto"
  android:id="@+id/nav_graph"
  app:startDestination="@id/nav_main_frag">

  <fragment
    android:id="@+id/nav_main_frag"
    android:name="com.github.satoshun.example.feature.main.MainFragment">

    <action
      android:id="@+id/main_to_sub1"
      app:destination="@id/nav_sub1_frag" />
  </fragment>

  ...
</navigation>
```

そして、これを用いて遷移用インターフェースを実装します。

```kotlin
class MainModuleRouterImpl @Inject constructor(
  private val controller: NavController
) : MainModuleRouter {
  override fun routeToSub1() {
    // NavComponentで自動生成されるコードを用いて遷移
    controller.navigate(MainFragmentDirections.mainToSub1())
  }
}
```

これで実装は完了です。Navigation Componentを使っているため、実装はかなり楽です。

あとはDaggerで配るだけです。Daggerで配る部分はサンプルコードを見ていただけたらと思います。サンプルでは、Dagger Androidを用いています。

## メモ

### routerモジュールをわざわざ作る必要はないかも

構成図を見てほしいのですが、実はrouterモジュールをわざわざ作る必要はなく、appモジュールに含めても良いです。routerモジュールを作るのがめんどう、もしくは意味がないと感じるなら、appモジュールで実装しても良いと思います。

## まとめ

- Navigation Componentでモジュール間の遷移を宣言的にXMLで書けるのは見やすくて非常に良いと思いました。
    - またNavigation Componentはactivityの記述もできるので、既存アプリへの導入も比較的しやすいと思います
- app/routerでは各featureモジュールへの依存を持つことができるので、クラスへの参照を持ちながら、navgationのgraphを作ることができる
    - navigation graphはActivity/Fragmentへの参照を持たなくても作ることが可能だが、持ったほうがLintなどの兼ね合いで安全
- サンプルコードは[satoshun/MultiModuleNavigationComponentExample](https://github.com/satoshun-android-example/MultiModuleNavigationComponentExample)にあります。

次は最終章になる予定です。DFMの遷移について書きます😃
