+++
date = "Fri Feb 15 00:14:56 UTC 2019"
title = "マルチモジュールの遷移について考える Part2"
tags = ["android", "multimodule", "gradle", "navigation]
blogimport = true
type = "post"
draft = true
+++

マルチモジュール遷移方法Part2です。[Part1](https://satoshun.github.io/2018/12/multi-module_transition_part1/)はここになります😌

Part2では、Navigation Componentを使うパターンを考えてみます。今回はDynamic Feature(DFM)には触れません。いわゆる一般的なライブラリモジュールでの遷移になります。

また、今回の検証に用いたコードは[satoshun/MultiModuleNavigationComponentExample](https://github.com/satoshun-android-example/MultiModuleNavigationComponentExample)にあります。

## モジュール構成について

細かい実装に入る前に、全体的なモジュール構成を説明します。

<img src="https://www.plantuml.com/plantuml/img/SoWkIImgAStDuU8goIp9ILLutBpeSTEEnyrB7pVlUToy-kdipLnS1Od9sOdfgGfAYGK5yMcfYIMbHQbA2jLS2WhHG95O45sKNrgIMXJBLOkakhWqoH1DEKWe5iQ8nw7925EJ4KoJ4RAcvFpSWloyrBmIi3lGN1wha5Yi01H6LWNHYqqXH0PPxUF6kOyRrptPFGqi3t8likpBnktFb-z-tBJaSVFcnqtxmIPDVToq7CHesWdN4a-4kKQacmiB1Iuka2KAkdOebe4KGCKG2e4XeQ2Rab-U1rCC3MDq2IEi4Z1Jk20Cg7WDghrOv13sEwJcfG2J6G00" width=400>

トップにappがあり、その下に各featureモジュールが紐付いています。featureモジュールでは遷移用インターフェースを持っており、それを用いて他のfeatureへ遷移を行い、遷移用インターフェースの実装はapp内のrouterモジュールで行います。

このモジュール構成のポイントは、各feature内で自身が使う遷移インターフェースを定義し、appがそのインターフェースの実装を行う点です。このような構成にすることで、各feature間で依存を持つことを防ぐことができます。

では、実装に入っていきます。今回はDaggerを使って実装をします。

## featureモジュール側の遷移用インターフェースの定義

前述の図の通り、各featureモジュール内で遷移用のインターフェースを定義します。ここでは、そのfeatureモジュールが必要としているインターフェースを定義します。

mmain画面からsub1画面に移動したいときは、次のようにインターフェースを定義します。

```kotlin
interface MainModuleRouter {
  fun routeToSub1()
}
```

Mainモジュール用のインターフェースなので、`MainModuleRouter`という名前にしました。統一が取れていれば何でもいいと思います。

そしてこれを、MainFragmentで使います。

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

インタフェースの定義だけでは駄目なので、次はこのインターフェースの実装を作ります。

## appモジュール側の遷移用インターフェースの実装

今回は遷移インタフェースの実装用のrouterモジュールで実装を行います。まずは、Navigation Componentを用いて、Graphを作ります。

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

これで実装は完了です。あとはDaggerで配るだけです。Daggerで配る部分はサンプルコードを見ていただけたらと思います。サンプルでは、Dagger Androidを用いています。

## メモ

### routerモジュールをわざわざ作る必要はないかも

構成図を見てほしいのですが、実はrouterモジュールをわざわざ作る必要はなく、appモジュールに含めても良いです。routerモジュールを作るのがめんどう、もしくは意味がないと感じるなら、appモジュールで実装しても良いと思います。

## まとめ

- Navigation Componentでモジュール間の遷移を宣言的にXMLで書けるのは見やすくて非常に良いと思いました。
    - またNavigation Componentはactivityの記述もできるので、既存アプリへの導入も比較的しやすいと思います
- サンプルコードは[satoshun/MultiModuleNavigationComponentExample](https://github.com/satoshun-android-example/MultiModuleNavigationComponentExample)にあります。

次は最終章になる予定です。DFMの遷移について書きます😃
