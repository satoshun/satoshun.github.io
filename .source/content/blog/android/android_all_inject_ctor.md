+++
date = "2018-11-14"
title = "Activity、Fragment、Viewでコンストラクタインジェクション使う"
tags = ["android", "factory", "dagger"]
blogimport = true
type = "post"
draft = true
+++

コンストラクタインジェクションしたい、そんな夢をみたAndroidエンジニアは数多くいると思います。

Activity + Daggerのとき以下のようなコードになると思います。

```kotlin
class MainActivity : Activity() {
  @Inject lateinit var presenter: UserPresenter  
  @Inject lateinit var analytics: Analytics
  ...
}
```

これをコンストラクタインジェクションにしたい。

```kotlin
class MainActivity @Inject constructor(
  private val presenter: UserPresenter,
  private val analytics: Analytics
): Activity() {
  ...
}
```

しかし、この書き方はうまくいきません。なぜならActivityインスタンスはシステム側で自動的に生成されるためです。
今まではインスタンスの生成部分をユーザ側から設定することは出来ませんでした。

AppComponentFactoryというクラスがAPI28から追加されました!!

これは次のように使います。

```kotlin
@Suppress("unused")
class MainAppComponentFactory : AppComponentFactory() {
  private lateinit var application: App

  override fun instantiateActivityCompat(
    cl: ClassLoader,
    className: String,
    intent: Intent?
  ): Activity {
    if (className == MainActivity::class.java.name) {
      return application.appComponent.mainActivity
    }
    return super.instantiateActivityCompat(cl, className, intent)
  }

  override fun instantiateApplicationCompat(cl: ClassLoader, className: String): Application {
    application = super.instantiateApplicationCompat(cl, className) as App
    return application
  }
}
```

```xml
...
  <application
    android:allowBackup="true"
    android:name=".App"
    android:appComponentFactory="com.github.satoshun.example.sample.MainAppComponentFactory"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true"
    android:theme="@style/AppTheme"
    tools:replace="android:appComponentFactory">
    ...
```


