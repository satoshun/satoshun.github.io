+++
date = "Sun Jun 21 06:49:26 UTC 2020"
title = "Android: Dagger HiltとDagger Androidの生成コードの違いについて"
tags = ["android", "dagger"]
blogimport = true
type = "post"
draft = true
+++

## 結論

- Dagger Hiltは1つのSubcomponentが作られる
- Dagger AndroidはActivityごとに専用のSubcomponentが作られる

（ドキュメントに書いてる通りです）

## 生成コードの違いについて

Dagger Hiltが爆誕したので、Dagger Androidと比較したときに、生成コードにはどのような違いがあるかを検証してみました。今回は、Activityのみに焦点を当てています。

Daggerのバージョンは2.28、Hiltのバージョンは2.28-alphaで検証しました。

## Dagger Androidの生成コード

Dagger Androidでは、`@ContributesAndroidInjector`を使うことで、ActivityのSubcomponentを自動生成してくれます。

例えば、`MainActivity`に対しては、次のようなコードが生成されます。

```java
@Module(subcomponents = MainActivityModule_ContributeMainActivity.MainActivitySubcomponent.class)
public abstract class MainActivityModule_ContributeMainActivity {
  private MainActivityModule_ContributeMainActivity() {}

  ...

  @Subcomponent
  public interface MainActivitySubcomponent extends AndroidInjector<MainActivity> {
    @Subcomponent.Factory
    interface Factory extends AndroidInjector.Factory<MainActivity> {}
  }
}
```

このSubcomponentは、`MainActivity`専用に作られているので、このSubcomponent配下では、MainActivityを直接injectすることが可能です。

```kotlin
class MainCounter @Inject constructor(private val mainActivity: MainActivity) {...}
```

このMainCounterを他のActivityに対してinjectしようとすると、エラーが出ます。これが、SubcomponentをActivity毎に独立で作るメリットです。

次に、Dagger Hiltを見てみます。

## Dagger Hiltの生成コード

Dagger Androidでは、`@AndroidEntryPoint`を使うことで、ActivityのSubcomponentを自動生成してくれます。

例えば、`MainActivity`と、`SubActivity`に対しては、次のようなコードが生成されます。

```java
@Subcomponent(
    modules = {
        FragmentCBuilderModule.class,
        ViewCBuilderModule.class,
        DefaultViewModelFactories.ActivityModule.class,
        HiltWrapper_ActivityModule.class,
        MainActivityModule.class
    }
)
@ActivityScoped
public abstract static class ActivityC implements MainActivity_GeneratedInjector,
    SubActivity_GeneratedInjector,
    ActivityComponent,
    DefaultViewModelFactories.ActivityEntryPoint,
    FragmentComponentManager.FragmentComponentBuilderEntryPoint,
    ViewComponentManager.ViewComponentBuilderEntryPoint,
    GeneratedComponent {
  @Subcomponent.Builder
  abstract interface Builder extends ActivityComponentBuilder {
  }
}
```

Dagger Androidとの大きな差異は、MainActivityとSubActivityが共通のSubcomponentを持つことです。
これは、Dagger Hiltの設計によるもので、1つの大きなSubcomponentのほうが、何かと良いだろうという事でこのようになりました。

ただ、Dagger Androidと比較したときに、1つのSubcomponentになったことで、`MainCounter`のようなクラスを作りにくくなりました。
なぜなら、Subcomponentを共通化したことにより、`MainActivity`などの専用のActivityを直接inject出来ないためです。

例えばDagger hiltでは次のコードは違法です。

```kotlin
// 違法
class MainCounter @Inject constructor(private val mainActivity: MainActivity) {...}
```

ただ、抜け道もあり、型変換を行うことで回避出来ます。

```kotlin
@Module
@InstallIn(ActivityComponent::class)
object MainActivityModule {
  @Provides
  fun provideMainActivity(activity: Activity): MainActivity =
    activity as MainActivity
}
```

型変換は悪い匂いがするので、このようなコードは最小限に抑えるのが良さそうです。

## まとめ

- Dagger Hiltは1つのSubcomponentが作られる
- Dagger AndroidはActivityごとに専用のSubcomponentが作られる

（ドキュメントに書いてる通りです）
