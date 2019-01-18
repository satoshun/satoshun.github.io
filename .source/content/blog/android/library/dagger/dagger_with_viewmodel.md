+++
date = "Wed Dec 19 09:59:20 UTC 2018"
lastmod = "Fri Jan 18 07:35:05 UTC 2019"
title = "Dagger + ViewModelの基本編 + 実例編"
tags = ["android", "dagger", "jetpack"]
blogimport = true
type = "post"
+++

この記事はDaggerとJetpackのViewModelをある程度知っている前提で進んでいきます😃

## 基本編

一緒にDagger + ViewModelを使うのはツラミがあります。それは、ViewModelのインスタンス生成は`ViewModelProvider`を介して行う必要があるためです。

例えば、次のコードは間違っています。

```kotlin
class MainViewModel @Inject constructor(...): ViewModel()

class MainActivity {
    @Inject lateinit var viewModel: MainViewModel

    ...
}
```

この書き方だとMainViewModelはDagger内で自動的にインスタンス生成されてしまうので、ViewModelProviderを介してくれません。よって次のように書く必要があります。

```kotlin
class MainViewModel(...): ViewModel()

@Module
class MainActivityModule {
    @Provides
    fun provideMainViewModel(...) : MainViewModel {
        // ViewModelProviderを使ってインスタンスを生成する
        return ViewModelProviders.of(...).get(MainViewModel::class.java)
    }
}

class MainActivity {
    @Inject lateinit var viewModel: MainViewModel

    ...
}
```

`@Provides`を使いインスタンス生成の方法を明示的に記述します。これで、ViewModelProviderを介してMainViewModelインスタンスを生成をすることが出来ます。

また、ViewModelを直接注入せずに、`ViewModelProvider.Factory`を注入し、ViewModelのインスタンス生成はActivity（or Fragment）に任せる方法があります。
このパターンのときは、activity-ktx（or fragment-ktx）に追加された拡張関数と組み合わせるといい感じに書けます。

```kotlin
class MainViewModel(...): ViewModel()
or
class MainViewModel @Inject constructor(...): ViewModel()

@Module
class MainActivityModule {
    @Provides
    fun provideViewModelFactory(...) : ViewModelProvider.Factory {
        // ここでMainViewModelを生成するFactoryを定義する
        return object: ViewModelProvider.Factory {
            ...
        }
    }
}

class MainActivity : AppCompatActivity() {
    // ViewModelではなく、Factoryを注入するのがポイント
    @Inject lateinit var factory: ViewModelProvider.Factory

    // MainViewModelインスタンスの生成はActivity側で行う
    // activity-ktxで定義されている拡張関数を使う
    private val viewModel: MainViewModel by viewModels { factory }

    ...
}
```

activity-ktxに定義されている`viewModels`拡張関数を使ってMainViewModelインスタンスを生成します。これでViewModelのライフサイクルを保ちつつ、Daggerで依存を解決することが出来ます。

次に、この2パターンのどちらの書き方がいいかを考えていきます。

## 実例編

`@Provides`を使うパターンと、Factoryを使うパターンは良いところ、悪いところがそれぞれあるので、好きな方を選べばいいと思います。

両アプローチともに、ソースコード、ノウハウが出ているので参考リンクを張っておきます。

---

[plaid](https://github.com/nickbutcher/plaid/blob/master/dribbble/src/main/java/io/plaidapp/dribbble/dagger/DribbbleModule.kt#L43)

- `@Provides`を使い、ViewModelを直接注入するパターン
- ViewModelごとにFactoryクラスをそれぞれ定義する必要があるので記述量が多い
- ViewModelの依存関係が解決できなかったらコンパイルエラーになる
    - Daggerのコンパイルチェックが上手く動く
- ViewModelを直接注入出来るので使い側からすると間違った使い方は出来ない（はず）

---

[Android ViewModel and FactoryProvider: good way to manage it with Dagger Multibindings](https://medium.com/@marco_cattaneo/android-viewmodel-and-factoryprovider-good-way-to-manage-it-with-dagger-2-d9e20a07084c)

- Daggerのmultibindingsを使い、ViewModel Factoryを注入するパターン
- 最初に仕組みを入れてしまえば、のちのちの記述量は少ない
- multibindingsを使っているのでランタイム時に落ちる可能性がある
    - Daggerのコンパイルチェックが上手く動かない
- ViewModelを直接注入する書き方が可能だが、その場合ViewModelProviderを介さずインスタンス生成するので正しくない
    - 間違った書き方が出来る

---

[Activity-Ktx + Dagger Example](https://github.com/satoshun-android-example/ActivityKtxDaggerExample/tree/master/app/src/main/java/com/github/satoshun/example/sample)

- Factory用のカスタムクラスを定義し、ViewModel Factoryを注入するパターン
    - 僕が作ったサンプルコードです😃　
- 最初に汎用クラスを入れてしまえば、のちのちの記述量は少ない
- ViewModelが提供されていなかったらコンパイルエラーになる
    - Daggerのコンパイルチェックが上手く動く
- ViewModelを直接注入する書き方が可能だが、その場合ViewModelProviderを介さずインスタンス生成するので正しくない
    - 間違った書き方が出来る

---

個人的には一番最後の自分のパターンを押したいところですが、上記のパターンはそれぞれメリット/デメリットがあると思うので、プロジェクトによって使い分けるのがよいと思います。

## まとめ

- 一番プロジェクトに適したパターンを適用するのが良いと思います😃
- DaggerでViewModelサポートの[Issue](https://github.com/google/dagger/issues/1271)が立っており、DaggerがViewModelをサポートする計画があります
    - なので前述のパターンはいつか過去のものとなる可能性が高いです。
