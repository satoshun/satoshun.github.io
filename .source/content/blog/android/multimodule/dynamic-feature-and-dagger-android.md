+++
date = "Tue Jan 15 13:00:56 UTC 2019"
title = "Dynamic Feature ModuleとDagger Android"
tags = ["android", "multimodule", "gradle", "dynamicmodule", "dagger"]
blogimport = true
type = "post"
draft = true
+++

[Dependency injection in a multi module project](https://medium.com/@keyboardsurfer/dependency-injection-in-a-multi-module-project-1a09511c14b7)を見て、Dynamic FeatureをAndroid Daggerで実現するにはどうするかを考えてみました。

結論からいうと、いくつかのbaseクラスを使うことで対応できそうです。

また、この記事ではDynamic Feature Module、Android Daggerをある程度知っている前提で進めていきます。
検証に用いたコードは[satoshun-android-example/DynamicFeatureDaggerExample](https://github.com/satoshun-android-example/DynamicFeatureDaggerExample)にあります😊

## 制限/前提知識

通常のAndroid DaggerはApplicationクラスでComponentを保持して、そこからSubcomponentを派生させる形になります。
ここでのポイントは、ApplicationでTopに位置するComponentを保持/作成するという点です。これはAppモジュールが全てのFeatureモジュールを知っていることを意味します。

この前提をもとに、Dynamic Featureを考えます。Dynamic FeatureではApplicationでTopに位置するComponentを保持/作成することが出来ません。なぜなら、Appモジュールは各Featureモジュールのことを知れないためです。Dynamic Moduleでは通常の構成と違い、AppとFeature Module間の依存関係が逆転します。結果、ApplicationでTopに位置するComponentを保持/作成することが出来ません。

そこでDynamic Featureでは、AppでTopのComponentを持つのはやめて、各Feature Module内でTopのComponentを保持するのが良いという結論になります。

ここまでが前提知識で、次に実現方法について説明します。

## 実現方法

Feature Subモジュールがあるとします。そして、このSubモジュールのTopがSubActivityであるとします。このSubActivityがDaggerApplicationのように振る舞うイメージです。

まずDaggerApplicationのように振る舞う`ModuleRootActivity`を定義します。

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

そして使う側のSubActivityで実装をします。

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

このようになります。`ModuleRootActivity`で、Featureモジュール内で使うComponentを保持し、各Fragmentが保持したComponentを参照します。
前述したとおり、ActivityをDaggerApplicationのように振る舞わせることでDagger Androidを実現しています。

## メモ1: 1つのFeature Module内で複数Activityがある場合

このパターンは考慮出来ていないです😂
おそらく、頑張ってApplicationクラス内でstaticで保持するか、もしくは、CoreComponentでScopedで管理するのが良いと思っています。

## メモ2: Configuration Change対応

Feature Root ComponentはActivityではなく、AACのViewModelで保持したほうが良いかも知れないです。

## メモ3: そもそもAndroid Daggerを使う必要あるのか?

Scopeをガンガン使いたい時、すでにアプリにAndroid Daggerを導入していて、Dynamic Featureに対応したい場合は使ってもいいかも。
ただ現状のPlaidのように、Android Daggerを使わないほうがコードが複雑にならないで良いと思っています。（今後心変わりする可能性は大いにあります😃）
