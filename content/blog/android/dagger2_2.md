+++
date = "2015-05-24T00:00:00Z"
title = "Android: Dagger2でDIをする. u2020から読み解く編 Part2"
tags = ["android", "dagger", "retrofit"]
blogimport = true
type = "post"
+++

## 概要

この記事では, JakeWhartonさんの[u2020](https://github.com/JakeWharton/u2020)から, AndroidでDagger2を使うときの実践的な方法を紹介します.
u2020はDagger1を使っていますが, Dagger2においても同様に使えるテクニックなので, u2020をベースにして説明します.

DI, Dagger2の基本について知りたい方は, [Part1](/2015/05/dagger2/)を見て下さい.

目次

- debugとproductionでModuleを切り替える
- Debug専用のViewを作る
- Mockモードの定義
-　まとめ


## debugとproductionでModuleを切り替える

gradleは, `productFlavors`を設定することで, ソースコード, ビルド設定を切り替えることが出来ます. u2020では, production, internalのflavorがあります.
そして, u2020はflavorの種類によって, DIする対象を切り替えています. production用のComponentとdebug用のComponentを作成することでそれを実現しています.
具体的には,

- /src/production/java/com/jakewharton/u2020/Modules.java
- /src/internalDebug/java/com/jakewharton/u2020/Modules.java
- /src/internalRelease/java/com/jakewharton/u2020/Modules.javaに

それぞれModuleを定義し, それをApplicationクラスから読み込むようにしています.
これで, flavorごとにinjectするインスタンスを切り替えることが出来ます.

こうすると何が嬉しいんでしょうか? 例えば以下のことが可能になります.

- Debugのみログを有効にしたい
- APIのエンドポイントを変えたい
- debug専用のViewを出したい
- Test用のインスタンスをinjectしたい
- etc, etc...

以下では, より細かく説明していきます.


## Debug専用のViewを作る

u2020では, Debug専用のView `DebugAppContainer`があります.
<a href="https://github.com/JakeWharton/u2020/blob/master/u2020.gif" target="\_blank">Debug専用のView</a>
はこんな感じです. Debugビルドの時は, このContainerをinjectしています.

DebugAppContainerは簡単にいえば, `DrawerLayout`を1つ実装し, その中に「データをモックに変更する」, 「social機能を有効にする」などを設定出来るViewをおいています.


## Mockモードの定義

u2020ではMockモードがあり, Mockデータを表示機能があります.


```java
public final class DebugDataModule {
  ...
  ...

  @Provides @Singleton @ApiEndpoint
  StringPreference provideEndpointPreference(SharedPreferences preferences) {
    return new StringPreference(preferences, "debug_endpoint", ApiEndpoints.MOCK_MODE.url);
  }

  @Provides @Singleton @IsMockMode boolean provideIsMockMode(@ApiEndpoint StringPreference endpoint) {
    return ApiEndpoints.isMockMode(endpoint.get());
  }
}
```

上記のコードを見て分かる通り, デバッグビルドの時はMockモードが有効になります. デバッグ時にサーバがなくて困る時がありますが, u2020では, assets/内にデバッグ用のモックデータを入れておくことで,
サーバ問題を解決しています. retrofitの`MockRestAdapter`を組み合わせ, mockからデータを取得しています.


## activityのlifecycleのログを取る

Applicationクラスには, `registerActivityLifecycleCallbacks`メソッドがあり, このメソッドにActivityLifecycleCallbacksインターフェースの実装を登録することで, アプリ内で動いているActivityのライフサイクルのタイミング(onStart, onResume, ...)でイベントを受け取ることが出来ます.
`ActivityLifecycleCallbackss`インターフェースの定義は以下になります.

```java
public void onActivityCreated(Activity activity, Bundle savedInstanceState) {}
public void onActivityStarted(Activity activity) {}
public void onActivityResumed(Activity activity) {}
public void onActivityPaused(Activity activity) {}
public void onActivityStopped(Activity activity) {}
public void onActivitySaveInstanceState(Activity activity, Bundle outState) {}
public void onActivityDestroyed(Activity activity) {}
```

u2020では,　[SocketActivityHierarchyServer](https://github.com/JakeWharton/u2020/blob/master/src/internalDebug/java/com/jakewharton/u2020/ui/debug/SocketActivityHierarchyServer.java)クラスをデバッグ, Activityが正しく振舞っているかを確認しています.

一元的にActivityのlifecycleログを取れるので, デバッグ時にはとても有効な機能だと思います.(u2020を見るまでは存在を知らなかった...)


## まとめ

u2020はDagger以外にもノウハウが多くあり, 非常に勉強になりました. 正直10%くらいしか理解できていないので, 次はu2020全体にフォーカスを当てた記事を書くので楽しみに待っていて下さい.
