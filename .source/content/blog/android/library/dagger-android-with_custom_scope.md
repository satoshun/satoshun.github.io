+++
date = "2018-05-22T00:00:00Z"
title = "Dagger-AndroidでUserScopeのようなカスタムのScopeを使って、同じインスタンスを使う方法"
tags = ["android", "dagger"]
blogimport = true
type = "post"
draft = true
+++

[Dagger](https://github.com/google/dagger)を使って、複数のActivityで共通のインスタンスを使う時は、スコープを使うことで実現できます。
この記事では、Dagger-Androidを使い、UserScopeスコープを定義したサンプルコードをベースに説明します。
また、基本的なDaggerの使い方は知っている前提で説明していきます。

サンプルコードは[こちら](https://github.com/satoshun-example/DaggerScopeExample)になります。

---

まず最初にUserScopeを定義します。

```kotlin
@Scope
@MustBeDocumented
@Retention(AnnotationRetention.RUNTIME)
annotation class UserScope
```

次にUserSubcomponentを作ります。

```kotlin
@UserScope
@Subcomponent(modules = [
  AndroidSupportInjectionModule::class
])
interface UserSubcomponent {
  @Subcomponent.Builder
  interface Builder {
    fun build(): UserSubcomponent
  }

  val activityInjector: DispatchingAndroidInjector<Activity>
}
```

UserSubcomponentは`UserScope`を持ちます。このように書くことで、Component(Subcomponent)とScopeを結びつけることが出来ます。

最後にAppComponentを作ります。

```kotlin
@Singleton
@Component(
    modules = [
      AndroidSupportInjectionModule::class
    ]
)
interface AppComponent : AndroidInjector<App> {
  @Component.Builder
  interface Builder {
    @BindsInstance
    fun application(application: App): Builder

    fun build(): AppComponent
  }

  override fun inject(app: App)

  val userComponentBuilder: UserSubcomponent.Builder
}
```

ここでのポイントは`AppComponent`に、`val userComponentBuilder: UserSubcomponent.Builder`を定義することです。
こうすることで、`AppComponent`に`UserSubcomponent`を結び付けることが出来ます。

```text
AppComponent <- UserSubcomponent
```

次に、各ActivityをComponentに結びつけていきます。
サンプルではMainActivity、UserScopedActivityとNoUserScopedActivityの3つのActivityを定義しており、

- MainActivity、UserScopedActivityはUserScopeに従い、インスタンスを共通で使いたい
- NoUserScopedActivityはUserScopeに従わない、コンパイルエラーにしたい

を達成したいとします。

- MainActivity、UserScopedActivityを`UserSubcomponent`に定義する
- NoUserScopedActivityを`AppComponent`に定義する

と定義すれば達成できます。

## MainActivity、UserScopedActivityをUserSubcomponentに定義する

```kotlin
// UserSubcomponent.kt
@UserScope
@Subcomponent(modules = [
  AndroidSupportInjectionModule::class,
  MainActivityModule::class,
  UserScopedActivityModule::class
])
interface UserSubcomponent {
  @Subcomponent.Builder
  interface Builder {
    fun build(): UserSubcomponent
  }

  val activityInjector: DispatchingAndroidInjector<Activity>
}

// MainActivityModule.kt
@Module
interface MainActivityModule {
  @ContributesAndroidInjector
  fun contributeMainActivity(): MainActivity
}

// UserScopedActivityModule.kt
@Module
interface UserScopedActivityModule {
  @ContributesAndroidInjector
  fun contributeUserScopedActivity(): UserScopedActivity
}
```

## NoUserScopedActivityをAppComponentに定義する

```kotlin
// AppComponent.kt
@Singleton
@Component(
    modules = [
      AndroidSupportInjectionModule::class,
      NoUserScopedActivityModule::class
    ]
)
interface AppComponent : AndroidInjector<App> {
  @Component.Builder
  interface Builder {
    @BindsInstance
    fun application(application: App): Builder

    fun build(): AppComponent
  }

  override fun inject(app: App)

  val userComponentBuilder: UserSubcomponent.Builder
}

// NoUserScopedActivityModule.kt
@Module
interface NoUserScopedActivityModule {
  @ContributesAndroidInjector
  fun contributeNoUserScopedActivity(): NoUserScopedActivity
}
```

これで、定義は完了です。
実際に正しく動くかを確認してみます。例えば`UserScope`に従う`UserManager`があるとします。

```kotlin
// UserManager.kt
@UserScope
class UserManager @Inject constructor() {
  var value = 100
}
```

これは`UserScope`に従うので、MainActivity、UserScopedActivityには`Inject`できます。

```kotlin
// MainActivity.kt
class MainActivity : AppCompatActivity() {

  // ok
  @Inject lateinit var userManager: UserManager

  ...
}
// UserScopedActivity.kt
class UserScopedActivity : AppCompatActivity() {

  // ok: MainActivityと同じインスタンスが注入される
  @Inject lateinit var userManager: UserManager

  ...
}
```

しかし、NoUserScopedActivityに`Inject`しようとするとコンパイルエラーになります。

```kotlin
// NoUserScopedActivity.kt
class NoUserScopedActivity : AppCompatActivity() {

  //  compile error if added below code
  //  @Inject lateinit var userManager: UserManager
  ...
}
```

NoUserScopedActivityを`UserScope`に従う形でComponentで定義していないためです。

- MainActivity、UserScopedActivityはUserScopeに従い、インスタンスを共通で使いたい
- NoUserScopedActivityはUserScopeに従わない、コンパイルエラーにしたい

が達成できました。

## 余談

そもそも、`ContributesAndroidInjector`定義時に、スコープを付与してあげればいいんじゃないかと思うかもしれません。

```kotlin
// MainActivityModule.kt
@Module
interface MainActivityModule {
  @UserScope
  @ContributesAndroidInjector
  fun contributeMainActivity(): MainActivity
}

// UserScopedActivityModule.kt
@Module
interface UserScopedActivityModule {
  @UserScope
  @ContributesAndroidInjector
  fun contributeUserScopedActivity(): UserScopedActivity
}
```

この方法だと、今回のケースには不都合です。

- MainActivity、UserScopedActivityは`UserScope`に従う
  - しかし、MainActivity、UserScopedActivityでインスタンスを共通で使うことは出来ない

ためです。

何故かと言うと、`ContributesAndroidInjector`はSubcomponentを作るシンタックスシュガーのようなものですが、
`MainActivityModule`と`UserScopedActivityModule`はそれぞれ独立した形でSubcomponentが作られてしまうため、インスタンスを共通で使うことが出来ません。

## まとめ

- Dagger-Androidと
