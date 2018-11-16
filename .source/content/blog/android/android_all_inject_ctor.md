+++
date = "2018-11-16"
title = "Activity、Fragment、Viewにコンストラクタインジェクションする"
tags = ["android", "factory", "dagger"]
blogimport = true
type = "post"
+++

Daggerライブラリを使い、Androidフレームワークが提供するActivityなどのクラスにコンストラクタインジェクションしたい、
そんな夢をみたAndroidエンジニアは数多くいると思います。

この記事ではそんな夢を叶える方法を紹介します。

[サンプルコードはここに](https://github.com/satoshun-android-example/ConstructorInjectionExample)あります。

## FragmentFactory

Fragmentに依存関係を注入する時、普通にやると以下のコードになると思います。

```kotlin
class MainFragment : Fragment() {
  lateinit var userHandler: UserHandler
  ...
}
```

これをコンストラクタインジェクションにしたい。

```kotlin
class MainFragment @Inject constructor(
  private val userHandler: UserHandler
) : Fragment() {
  ...
}
```

`androidx.fragment:fragment:1.1.0-alpha01`から、FragmentFactoryが追加されました!!
これを使うことでコンストラクタインジェクションが可能になります。

MainFragmentインスタンスを生成する`FragmentFactory`を作成します。

```kotlin
class MainFragmentFactory @Inject constructor(
  private val fragment: Provider<MainFragment>
) : FragmentFactory() {
  override fun instantiate(
    classLoader: ClassLoader,
    className: String,
    args: Bundle?
  ): Fragment {
    if (className == MainFragment::class.java.name) {
      return fragment.get()
    }
    return super.instantiate(classLoader, className, args)
  }
}
```

`FragmentFactory.instantiate`をoverrideし、そこでMainFragmentのインスタンスを生成します。

最後に、作成した`MainFragmentFactory`をActivityの`FragmentManager`に登録します。

```kotlin
class MainActivity : AppCompatActivity() {
  @Inject lateinit var fragmentFactory: MainFragmentFactory

  override fun onCreate(savedInstanceState: Bundle?) {
    DaggerAppComponent.create().inject(this)
    supportFragmentManager.fragmentFactory = fragmentFactory

    super.onCreate(savedInstanceState)
    ...
```

`SupportFragmentManager.fragmentFactory`に登録します。
これで、Fragmentが生成されるとき`MainFragmentFactory`がフックされます。

`SupportFragmentManager.fragmentFactory`にFactoryを登録するタイミングは`super.onCreate(savedInstanceState)`の前が良いと思います。
それは`super.onCreate`のタイミングで以前のFragmentが復元されるためです。
復元されるタイミングで適切なFactoryがないとクラッシュするので、復元する前で登録する必要があります。

## LayoutInflater.Factory

次にViewです。`LayoutInflater.Factory`が定義されています。
これを使うことでカスタムのコンストラクタを持ったViewを定義することが出来ます。

```kotlin
class MainTextView(
  context: Context,
  private val userHandler: UserHandler
) : TextView(context) {
  class Factory @Inject constructor(private val userHandler: UserHandler) {
    fun create(context: Context): MainTextView {
      return MainTextView(context, userHandler)
    }
  }
}
```

カスタムのコンストラクタを持ったViewは通常の方法ではインスタンスを生成できませんが、
`LayoutInflater.Factory`を使うことで、インスタンスを生成できるようになります。

```kotlin
class MainLayoutInflaterFactory @Inject constructor(
  private val factory: MainTextView.Factory
) : LayoutInflater.Factory {
  override fun onCreateView(name: String, context: Context, attrs: AttributeSet?): View? {
    if (name == MainTextView::class.java.name) {
      return factory.create(context)
    }
    return null
  }
}
```

`LayoutInflater.Factory.onCreateView`をoverrideし、`MainTextView`インスタンスを生成します。

最後に、作成した`MainLayoutInflaterFactory`をActivityの`layoutInflater.factory`に登録します。

```kotlin
class MainActivity : AppCompatActivity() {
  private lateinit var layoutInflaterFactory: MainLayoutInflaterFactory

  override fun onCreate(savedInstanceState: Bundle?) {
    DaggerAppComponent.create().inject(this)
    layoutInflater.factory = layoutInflaterFactory

    super.onCreate(savedInstanceState)
    ...
```

Activityの`LayoutInflater.factory`に登録します。登録するタイミングはsetContentViewの前が良いと思います。

## AppComponentFactory

次にActivityです。
Activityに依存関係を注入する時、普通にやると以下のコードになると思います。

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
カスタム定義のコンストラクタだと、システム側でインスタンスを生成することが出来ません。

これを解決するべく、API28からAppComponentFactoryというクラスが追加されました!!

MainActivityインスタンスを生成する`AppComponentFactory`を作成します。

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

次にAndroidマニフェストに`MainAppComponentFactory`を登録します。

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

`AppComponentFactory.instantiateActivityCompat`をoverrideし、`MainActivity`インスタンスを生成します。

これで、カスタムのコンストラクタを持ったActivityインスタンスを生成することが出来ます!!

## まとめ

- Fragment、Viewは今からでも使い始めることができる。Daggerなどのライブラリと組み合わせると最高☆
- AppComponentFactoryはAPI28からなので...5年後くらいでしょうか😢
- [サンプルコードはここです](https://github.com/satoshun-android-example/ConstructorInjectionExample)

何か疑問点があれば、twitterやサンプルコードのISSUEなどで聞いてください😃
