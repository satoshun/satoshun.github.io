+++
date = "2018-11-14"
title = "Activity、Fragment、Viewにコンストラクタインジェクションする"
tags = ["android", "factory", "dagger"]
blogimport = true
type = "post"
draft = true
+++

Daggerライブラリを使い、ActivityなどAndroidフレームワークが提供するクラスにコンストラクタインジェクションしたい、
そんな夢をみたAndroidエンジニアは数多くいると思います。

## FragmentFactory

Fragmentに依存関係を注入する時、普通にやると以下のコードになると思います。

```kotlin
class MainFragment : Fragment() {
  lateinit var userHandler: UserHandler
  ...
}
```

これを以下のようにし、コンストラクタインジェクションにしたい。

```kotlin
class MainFragment @Inject constructor(
  private val userHandler: UserHandler
) : Fragment() {
  ...
}
```

`androidx.fragment:fragment:1.1.0-alpha01`から、FragmentFactoryが追加されました!
これを使うことでコンストラクタインジェクションが可能になります。

次のように使います。

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

`FragmentFactory.instantiate`をoverrideし、そこでFragmentのインスタンスを生成します。

次に、`MainFragmentFactory`を`FragmentManager`に登録します。

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
これで、Fragmentを生成するときに`MainFragmentFactory`がフックされます。

`SupportFragmentManager.fragmentFactory`をコールするタイミングは`super.onCreate(savedInstanceState)`の前が良いと思います。
それは`super.onCreate`のタイミングでFragmentが復元されるためです。
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

`LayoutInflater.Factory`を使うことで、前のカスタムコンストラクタを持ったViewを生成できるようになります。

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

最後に作った`MainLayoutInflaterFactory`を登録します。

```kotlin
class MainActivity : AppCompatActivity() {
  private lateinit var layoutInflaterFactory: MainLayoutInflaterFactory

  override fun onCreate(savedInstanceState: Bundle?) {
    DaggerAppComponent.create().inject(this)
    layoutInflater.factory = layoutInflaterFactory
    super.onCreate(savedInstanceState)
    ...
```

Activityの`layoutInflater.factory`に登録します。登録するタイミングはsetContentViewの前で良いと思います。
今回は`super.onCreate(savedInstanceState)`の前で登録してますが、後でも問題ないと思います。

##　AppComponentFactory

Activityに依存関係を注入する時、普通にやると以下のコードになると思います。

```kotlin
class MainActivity : Activity() {
  @Inject lateinit var presenter: UserPresenter
  @Inject lateinit var analytics: Analytics
  ...
}
```

これを以下のようにし、コンストラクタインジェクションにしたい。

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
      return application.appComponent.mainActivity // Daggerからインスタンスを生成する
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

Androidマニフェストに登録した`MainAppComponentFactory`がActivityを生成するタイミングでフックされます。
`AppComponentFactory.instantiateActivityCompat`をoverrideし、`MainActivity`インスタンスを生成します。
これで、カスタム定義のコンストラクタを使う事ができます!!
