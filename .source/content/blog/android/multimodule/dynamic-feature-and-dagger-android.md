+++
date = "Wed Jan 16 12:10:36 UTC 2019"
title = "Dynamic Feature ModuleとDagger Androidサポートについて"
tags = ["android", "multimodule", "gradle", "dynamicmodule", "dagger"]
blogimport = true
type = "post"
draft = false
+++

[Dependency injection in a multi module project](https://medium.com/@keyboardsurfer/dependency-injection-in-a-multi-module-project-1a09511c14b7)を見て、Dynamic FeatureをDagger Androidで実現するにはどうするかを考えてみました。

結論からいうと、いくつかのbaseクラスを定義することで対応できそうです。

また、この記事ではDynamic Feature Module、Dagger Androidをある程度知っている前提で進めていきます。

検証に用いたコードは[satoshun-android-example/DynamicFeatureDaggerExample](https://github.com/satoshun-android-example/DynamicFeatureDaggerExample)にあります😊

## 前提知識

通常のDagger AndroidはApplicationクラスでComponentを保持して、そこからSubcomponentを派生させる形になります。
ここでのポイントは、Applficationでトップに位置するComponentを保持/作成するという点です。これはappモジュールが全てのFeatureモジュールを知っていることを意味します。

---

<img src="https://www.plantuml.com/plantuml/svg/SoWkIImgAStDuU8goIp9ILLukc_oixbB7pUkUx9lxjErCvxEtip5bPoVMv2VbvfNeX2TM53mk7dHuwOTZvkNFcxUyxXvTQn2Oh42K1XPrUEchO-RfnbYKrbSccI8gTG8Xr8ZBYukeDaAkYdvvNcwTX3TQ090DGwfUIb0Fm00" width=300>

---

この前提をもとに、Dynamic Featureを考えます。Dynamic FeatureではApplicationでトップに位置するComponentを保持/作成することが出来ません。なぜなら、appモジュールは各Featureモジュールのことを知れないためです。Dynamic Moduleでは通常のモジュール構成と違い、appとFeature Module間の依存関係が逆転します。結果、Applicationでトップに位置するComponentを保持/作成することが出来ません。

---

<img src="https://www.plantuml.com/plantuml/svg/SoWkIImgAStDuU8goIp9ILLmgSnBpCrCLd1BJImfBKfztBZkoRwvJzVEU3fxCnTNSdvkGNvUQbw9GdHYGS7ZvaMFctOyRbxwk7dFu-RLiGg9nGf0OMHLZvksFcwUPeXDPN5faY6cKYCSIesukBX0EXHqK_BBytJjm1Q1n544k1nIyrA0dW40" width=300>

---

そこでDynamic Featureでは、appモジュールでトップに位置するComponentを保持するのはやめて、各Feature Module内でそれぞれのComponentを保持するのが良いことが分かります。

ここまでが前提知識で、次にDynamic Feature + Dagger Androidの実装について説明します。

## 実装

Feature Subモジュールがあり、このSubモジュールのエントリポイント（トップに位置するクラス）としてSubActivityが定義されているとします。
実装の方針としては、このSubActivityをDaggerApplicationのように振る舞わさせることを目指します。なぜなら、このFeatureモジュールのトップに位置するクラスがSubActivityなので、これをDaggerApplicationのように扱うことができれば、Dagger Androidの世界に上手く落とし込むことが出来ると考えたからです。

では、実装を始めます。

最初に、SubActivityをDaggerApplicationのように振る舞わさせるために`ModuleRootActivity`クラスを定義します。
それに合わせて、いくつかの付随したクラスも定義しておきます。これがbaseクラス群になります。

```kotlin
abstract class ModuleRootActivity : AppCompatActivity(),
  HasModuleInjector {
  @Inject lateinit var fragmentInjector: DispatchingAndroidInjector<Fragment>

  private lateinit var injector: ModuleActivityInjector

  override fun onCreate(savedInstanceState: Bundle?) {
    injector = moduleComponent.moduleInjector
    injector.activity.inject(this)
    super.onCreate(savedInstanceState)
  }

  protected abstract val moduleComponent: ModuleActivityComponent

  override fun supportFragmentInjector(): AndroidInjector<Fragment> =
    fragmentInjector
}

class ModuleActivityInjector @Inject constructor(
  internal val activity: DispatchingAndroidInjector<Activity>
)

interface ModuleActivityComponent {
  val moduleInjector: ModuleActivityInjector
}

interface HasModuleInjector : HasSupportFragmentInjector

abstract class ModuleChildFragment : Fragment() {
  override fun onAttach(context: Context) {
    AndroidSupportInjection.inject(this)
    super.onAttach(context)
  }
}
```

次に、このbaseクラス群を使い、SubActivityとDagger Componentを実装をします。

```kotlin
@ModuleScope
@Component(
  dependencies = [CoreComponent::class], // 共通で使うComponent
  modules = [
    AndroidSupportInjectionModule::class,
    SubBuilder::class
  ]
)
internal interface SubComponent : ModuleActivityComponent {
  @Component.Builder
  interface Builder {
    fun appComponent(module: CoreComponent): Builder
    fun build(): Sub1Component
  }
}

@Module(
  includes = [SubActivityModule::class]
)
interface SubBuilder

@Module
internal interface SubActivityModule {
  @ContributesAndroidInjector(modules = [SubFragmentsModule::class])
  fun contributeSubActivity(): SubActivity
}

@Module
internal interface SubFragmentsModule {
  @ContributesAndroidInjector
  fun contributeSubFragment(): SubFragment
}

class SubActivity : ModuleRootActivity() {
  @Inject lateinit var ...

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.sub_act)
    ...
  }

  override val moduleComponent: ModuleActivityComponent
    get() = DaggerSubComponent
      .builder()
      .appComponent(App.coreComponent())
      .build()
}

class SubFragment : ModuleChildFragment() {
  @Inject lateinit var ...
}
```

`ModuleRootActivity`で、Featureモジュール内で使うComponentを保持し、各Fragmentで保持したComponentを参照することで、ActivityをDaggerApplicationのように振る舞わさせる事ができます。ComponentやModuleの定義は従来のDagger Androidの書き方とほぼ一緒です。

これで、Dynamic Featureモジュール + Dagger Androidを実現することが出来ます😃

細かい部分はサンプルを見ていただけたらと思います。[DynamicFeatureDaggerExample](https://github.com/satoshun-android-example/DynamicFeatureDaggerExample)

## メモ1: 1つのFeature Module内で複数Activityがある場合

このパターンは考慮出来ていないです😂
おそらく、頑張ってApplicationクラス内でstaticで保持するか、もしくは、CoreComponentでScopedで管理するのが良いと思っています。

## メモ2: Configuration Change対応

Feature Root ComponentはActivityではなく、AACのViewModelで保持したほうが良いかも知れないです。

## メモ3: そもそもDagger Androidを使う必要あるのか?

Scopeをガンガン使いたい時、すでにDagger Androidを導入している場合は使ってもいいかも。
ただPlaidのように、Dagger Androidを使わないほうがコードが複雑にならなそうなので、使わないほうが基本良いと思います。（今後心変わりする可能性は大いにあります）

---

では、Happy Dagger Life 😊😊😊
