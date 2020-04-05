+++
date = "2016-11-27"
title = "Android: Dagger2のScopeについてあれこれ"
tags = ["android", "dagger"]
blogimport = true
type = "post"
draft = false
+++

結論: Application(Singleton), Activity, Fragment, Viewの4つのScopeを作るのが良いと思います.


## Scopeとは?

とあるprovideするインスタンスの生存期間を指定するアノテーションです. SingletonアノテーションもScopeの一種です.
例えば, Singletonは, その名の表すとおり, Singletonで表現したいインスタンスに対してアノテートします.

Androidなら, OkHttpClientなどはSingletonで管理すべきなので, 下のように表現できます.

```java
@Singleton @Provides public static OkHttpClient provideOkHttpClient() {
  return new OkHttpClient.Builder()
        .connectTimeout(10, TimeUnit.SECONDS)
        .readTimeout(20, TimeUnit.SECONDS)
        .build();
}
```

Singletonアノテーションを付けないと, Injectするたびに新しいインスタンスが作られます.

また, ScopeアノテーションはDagger特有のアノテーションではなく, JSR-330で定義されているスペックなので,
Daggerが廃れたとしても, JSR-330が廃れてない限り使える知識, 技術になります.


## Androidに適したScope

Androidには大きく, Application, Activity, Fragment, Viewの4つのライフサイクルがあります.
なので, それらに対してScopeを定義すればいいと思ってます.

- Application: Singleton
- Activity: ActivityScope
- Fragment: FragmentScope
- View: ViewScope

ViewScopeはRecylerView, ListViewなどで有効に使えると思います.
