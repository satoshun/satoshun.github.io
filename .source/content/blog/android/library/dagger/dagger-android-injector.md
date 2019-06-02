+++
date = "Sun Jun  2 08:40:48 UTC 2019"
title = "Dagger2: 2.23から入ったHasAndroidInjectorについて"
tags = ["dagger", "android"]
blogimport = true
type = "post"
draft = true
+++

Dagger 2.23に新しく、`HasAndroidInjector`インターフェースが入りました。
これは、従来の`HasActivityInjector`や`HasFragmentInjector`などを置き換えるために作られました。ただし、従来と同様の動作をします。

今回は、どのように置き換えるかを説明していきたいと思います。

また、`DaggerApplication`や`DaggerActivity`などのクラスを使っていないものとします。

## 実際に置き換えていく

### AppComponent

AndroidSupportInjectionModuleを使っているなら、`AndroidInjectionModule`に置き換えます。`AndroidSupportInjectionModule`を使う必要はありません。

```kotlin
@Singleton
@dagger.Component(
  modules = [
    AndroidInjectionModule::class, // AndroidInjectionModuleを使う
    ...
  ]
)
interface AppComponent ...
```

### Application

Applicationで実装している、HasActivityInjector, HasServiceInjectorを`HasAndroidInjector`に置き換えます。

```kotlin
class App : Application(),
  // HasAndroidInjectorのみでおｋ
  HasAndroidInjector {

  // 型変数がanyになる
  @Inject lateinit var androidInjector: DispatchingAndroidInjector<Any>

  // 返り値の型変数があanyになる
  override fun androidInjector(): AndroidInjector<Any> {
    DaggerAppComponent.factory().create(this).inject(this)
    return androidInjector
  }
}
```

## Activity, Fragment

HasSupportFragmentInjectorなどのInjectorは、`HasAndroidInjector`に置き換わります。

```kotlin
class MainActivity : AppCompatActivity(),
  HasAndroidInjector {

  @Inject lateinit var androidInjector: DispatchingAndroidInjector<Any>

  override fun onCreate(savedInstanceState: Bundle?) {
    AndroidInjection.inject(this) // ここは一緒
    ...
  }

  override fun androidInjector(): AndroidInjector<Any> {
    return androidInjector
  }
}

class MainFragment : Fragment() {
  override fun onAttach(context: Context) {
    AndroidSupportInjection.inject(this) // ここは一緒
    ...
  }
}
```

となります。従来の***Injectorが汎用的な`HasAndroidInjector`まとまりました😃
大きな修正は必要なさそうです。

## なんでこんな変更が入ったか?

ronshapiroさんが以下のようなツイートをしていました。

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Dagger 2.23 turns off formatting by default so your build is faster, but has a flag to turn it back on if you&#39;d like.<br><br>It also brings bug fixes and a more flexible <a href="https://t.co/d0MeQYkvAV">https://t.co/d0MeQYkvAV</a> API that will allow for more androidx support<a href="https://t.co/vtW7gebmu2">https://t.co/vtW7gebmu2</a></p>&mdash; Ron Shapiro (@rdshapiro) <a href="https://twitter.com/rdshapiro/status/1133493726561619968?ref_src=twsrc%5Etfw">May 28, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

今後、androidxのサポートを柔軟に対応するための変更のようです😃（ViewModel対応してほしい...）

## まとめ

- HasAndroidInjectorという1つの汎用インターフェースが爆誕した
- androidxのサポート増やしてくれそう。嬉しい😃
