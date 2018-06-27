+++
date = "2018-06-25T00:00:00Z"
title = "Dagger-AndroidでUserScopeのようなカスタムのScopeを使い、特定のActivity間のみで同一インスタンスを使う方法"
tags = ["android", "dagger"]
blogimport = true
type = "post"
+++

[Dagger](https://github.com/google/dagger)を使い、複数のインスタンス間で同一のインスタンスを使う時は、スコープを使うことで実現できます。
Androidでは、すべてのActivityで共通のインスタンスを使うには `Singleton`スコープとAppComponentを組み合わせて使う方法がよく知られています。
しかし、__特定__のActivity間でのみ共通のインスタンスを使いたい場合にはこの方法は使えません。Singletonだと__すべて__のActivity間で共通のインスタンスが使えてしまいます。

この記事では、Dagger-Androidを使ったサンプルコードをベースに、「特定のActivity間のみで同一インスタンスを使う方法」を説明します。
また、基本的なDaggerの使い方は知っている前提で説明していきます。

サンプルコードは[こちら](https://github.com/satoshun-example/DaggerScopeExample)になります。
コードを見ると理解がより深まると思うので、ぜひご覧になってください😊

---

では説明していきます。今回のサンプルコードの目指すところは

- UserScopeを定義し、MainActivity、UserScopedActivityで同一の`UserManager`インスタンスを使用する

とします。

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
@Subcomponent
interface UserSubcomponent {
  @Subcomponent.Builder
  interface Builder {
    fun build(): UserSubcomponent
  }

  val activityInjector: DispatchingAndroidInjector<Activity>
}
```

ここでは、UserSubcomponentに`UserScope`スコープを持たせています。
このように書くことで、SubcomponentとScopeを結びつけることが出来ます。

次にAppComponentを作ります。

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

  // AppComponentとUserSubcomponentを結びつける
  val userComponentBuilder: UserSubcomponent.Builder
}
```

ここでのポイントは`AppComponent`に、`val userComponentBuilder: UserSubcomponent.Builder`を定義することです。
こうすることで、`AppComponent`に`UserSubcomponent`を結び付けることが出来ます。

これで基本的な部分の定義は完了しました。

---

次に、各ActivityをComponentに結びつけていきます。
サンプルではMainActivity、UserScopedActivityとNoUserScopedActivityの3つのActivityを定義しており、
それぞれのActivityは以下のように振る舞わせたいとします。

- MainActivity、UserScopedActivityはUserScopeに従い、インスタンスを共通で使いたい
- NoUserScopedActivityはUserScopeに従わない、コンパイルエラーにしたい

## MainActivity、UserScopedActivityをUserSubcomponentに従わせる

MainActivity、UserScopedActivityを`UserSubcomponent`に定義することで、MainActivity、UserScopedActivityを`UserScope`に従わせることが出来ます。
なぜなら、`UserSubcomponent`は`UserScope`に紐付いているためです。

```kotlin
// UserSubcomponent.kt
@UserScope
@Subcomponent(modules = [
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

## NoUserScopedActivityはUserScopeに従わない

NoUserScopedActivityを`AppComponent`に定義することで、NoUserScopedActivityで`UserScope`を使っていたらコンパイルエラーにすることが出来ます。
`AppComponent`は`UserScope`に紐付いていないためです。

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

これで、定義は完了です。実際に正しく動くかを確認してみます。
適当に`UserScope`に従う`UserManager`を定義します。

```kotlin
// UserManager.kt
@UserScope
class UserManager @Inject constructor() {
  var value = 100
}
```

これは`UserScope`に従うので、MainActivity、UserScopedActivityには期待通り同一インスタンスが`Inject`できます。

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

  //  下のコメントアウトを取るとコンパイルエラー
  //  @Inject lateinit var userManager: UserManager
  ...
}
```

NoUserScopedActivityを`UserScope`に従う形で定義してないためです。

- MainActivity、UserScopedActivityはUserScopeに従い、インスタンスを共通で使いたい
- NoUserScopedActivityはUserScopeに従わない、コンパイルエラーにしたい

が達成できました。

## 余談

そもそも、`ContributesAndroidInjector`定義時に、`UserScope`スコープを付与してあげればいいんじゃないかと思うかもしれません。

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

このやり方だと、今回のケースには不都合です。

MainActivity、UserScopedActivityは`UserScope`に従うのでコンパイルは通ります。
しかし、MainActivity、UserScopedActivityで同一インスタンスを使うことは出来ません。

何故かと言うと、`ContributesAndroidInjector`はSubcomponentを作るシンタックスシュガーのようなものですが、
`MainActivityModule`と`UserScopedActivityModule`はそれぞれ独立したSubcomponentを作るので、独立したComponent間で同一インスタンスを使うことが出来ないためです。

今回のように、`UserSubcomponent`を定義して、そのComponentをベースに所属させる必要があります。

## まとめ

- 特定のActivityのみで共通のインスタンスを使いたいときは、結構めんどう
  - 冗長な気がするので、もっといい方法があったら教えてください😋
- `ContributesAndroidInjector`がどういう動作をするのかを知っておくと、いざというときに便利
- サンプルコードは[こちら](https://github.com/satoshun-example/DaggerScopeExample)
