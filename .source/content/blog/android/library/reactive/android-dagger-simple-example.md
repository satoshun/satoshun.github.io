+++
date = "2018-09-07T00:00:00Z"
title = "Daggerでprivate valで依存関係を取得したかった"
tags = ["android", "dagger"]
blogimport = true
type = "post"
+++

みなさんこんにちは

今回はDaggerの話をします。Dagger + Kotlinの1つ気になる点として`@Inject lateinit var` になってしまうところがあると思います。

```kotlin
class HogeActivity {
    @Inject lateinit var hoge: Hoge
}
```

これをなんとか出来ないかなと思って考えてみました。

結論から言うと最終形はこのようになります。

```kotlin
class HogeActivity {
    private val hoge: Hoge by inject()
}
```

private valになりました。これをどのように実現したかについて説明してきます。ちなみにですが、これはプロダクションに耐えれるようなコードではありません。ご了承ください。

今回は`IntoMap`を使って実装することにしました。

`IntoMap`とはその名の通りMapにバインドするためのアノテーションです。
詳しくは[ドキュメント](https://google.github.io/dagger/multibindings.html)を見てください。

サンプルコードで説明していきます。

まずはIntoMapを使い、MapへのバインドをModuleに定義していきます。

```kotlin
@Module
interface HogeModule {
    @Binds @IntoMap @ClassKey(Hoge1::class) fun bindHoge1(hoge: Hoge1): Any
    @Binds @IntoMap @ClassKey(Hoge2::class) fun bindHoge2(hoge: Hoge2): Any
}

class Hoge1 @Inject constructor()
class Hoge2 @Inject constructor()

@MustBeDocumented
@Retention(AnnotationRetention.RUNTIME)
@MapKey
annotation class ClassKey(val value: KClass<out Any>)
```

Mapには当然、valueに対応するkeyが必要になります。DaggerではMapKeyを使うことで、valueとkeyを紐づけることが出来ます。

作ったModuleをComponentに組み込みます。

```kotlin
@Component(modules = [HogeModule::class])
interface AppComponent {
    val values: Map<Class<out Any>, @JvmSuppressWildcards Provider<Any>>
}
```

そして、ApplicationでAppComponentを生成します。

```kotlin
class App : Application() {
    val values: Map<Class<out Any>, @JvmSuppressWildcards Provider<Any>> by lazy {
    DaggerAppComponent.builder().build().values
    }

    inline fun <reified T> get(): T = values[T::class.java]!!.get() as T
}
```

そして、Activity用に拡張関数を定義して、

```kotlin
inline fun <reified T> Activity.inject() = lazy { get<T>() }

```
完成です!

```kotlin
class HogeActivity : AppCompatActivity {
    private val hoge: Hoge by inject()
}
```

## まとめ

このアプローチの問題点は、というか問題点しかないんですけど、

1. ランタイムで落ちる可能性がある
    - Daggerの良さであるアノテーション時のチェックが消え去る
2. ScopeとかSubcomponentとかの対応方法が良く分かんない
    - まあこれは考えていないだけなので、もしかしたらいい方法があるかもしれない
3. いわゆるService Locatorパターンになってしまい微妙

サンプルコードは https://github.com/satoshun-android-example/SimpleDaggerExample にあります。よかったら見てください😃
