+++
date = "Sat Oct 24 13:19:26 UTC 2020"
title = "Android: 既存のDagger AndroidからHiltへの移行について"
tags = ["android", "dagger"]
blogimport = true
type = "post"
draft = false
+++

dagger hiltもalphaになって久しいので、既存のプロジェクトを徐々に置き換える方法を簡単なサンプルと共に説明します。

より詳細な説明は公式のドキュメント [https://dagger.dev/hilt/migration-guide](https://dagger.dev/hilt/migration-guide) を見ると良きです。

## 既存の構成

既存ではdagger androidが使われていて、MainActivityとSubActivityがそれぞれ定義されているとします。そのうち、MainActivityをhiltに移行したいと思います。

```kotlin
class App : DaggerApplication() {
  override fun androidInjector(): AndroidInjector<App> {
    return ...
  }
}

@Singleton
@Component(
  modules = [
    AndroidInjectionModule::class,
    MyActivityModule::class
  ]
)
interface AppComponent : AndroidInjector<App>

@Module(
  includes = [	...  ]
)
interface MyActivityModule {

  // 今回移行したいdaggerの定義
  @ContributesAndroidInjector(modules = [MainFragmentModule::class])
  fun contributeMainActivity(): MainActivity

  @ContributesAndroidInjector(modules = ...)
  fun contributeSubActivity(): SubActivity
}
```

まずは、AppComponentのhilt対応をします。まずhiltの依存を`build.gradle`に追加します。そして、HiltAndroidApp、EntryPoint、InstallInアノテーションを使って、次のように書き換えます。

```kotlin
@HiltAndroidApp
class App : DaggerApplication() {
  @EntryPoint
  @InstallIn(SingletonComponent::class)
  interface ApplicationInjector : AndroidInjector<App>

  override fun androidInjector(): AndroidInjector<Any> {
    return EntryPoints.get(this, ApplicationInjector::class.java)
  }
}

@InstallIn(ApplicationComponent::class)
@Module(
  includes = [
    AndroidInjectionModule::class,
    ActivityModule::class
  ]
)
interface AppComponent
```

ここでは、androidInjectorは残しておきます。これは、hilt移行前のDagger androidを使ったActivityで引き続き使われるためです。

次に、MainFragmentを置き換えます。

```kotlin
// before
class MainFragment : DaggerFragment(R.layout.main_frag)

↓↓↓

// after
@AndroidEntryPoint
class MainFragment : Fragment(R.layout.main_frag)
```

DaggerFragmentの継承、もしくはそれに類するものを除去して、AndroidEntryPointアノテーションを指定します。

次にMainActivityとMainActivityModuleを置き換えます。

```kotlin
// before
class MainActivity : DaggerAppCompatActivity(R.layout.main_act)

↓↓↓

// after
@AndroidEntryPoint
class MainActivity : AppCompatActivity(R.layout.main_act)
```

```kotlin
// before
@Module
interface MainActivityModule { ... }

↓↓↓

// after
@InstallIn(ActivityComponent::class)
@Module
interface MainActivityModule { ... }
```

これらの定義が終わったら、最初のMyActivityModuleからMainActivity周りの指定を取り除きます。

```kotlin
// before
@Module(
  includes = [	...  ]
)
interface MyActivityModule {

  // 今回移行したいdaggerの定義
  @ContributesAndroidInjector(modules = [MainFragmentModule::class])
  fun contributeMainActivity(): MainActivity

  @ContributesAndroidInjector(modules = ...)
  fun contributeSubActivity(): SubActivity
}

↓↓↓

// before
@Module(
  includes = [	...  ]
)
interface MyActivityModule {

  @ContributesAndroidInjector(modules = ...)
  fun contributeSubActivity(): SubActivity
}
```

これで完了です。シンプルな構成だったら簡単に徐々に移行できそうです。

## まとめ・その他

サンプルで、例を作ってみたのでイマイチ理解できなかった人や、具体的なコードが欲しい人は見てください。

[https://github.com/satoshun-android-example/DaggerSample/pull/2](https://github.com/satoshun-android-example/DaggerSample/pull/2)

また、独自のActivityScope、ApplicationScopeなどを定義している場合は、hilt側で定義されているScopeに置き換えるとスムーズに置き換えられると思います
