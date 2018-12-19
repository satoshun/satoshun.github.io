+++
date = "Tue Dec 18 23:49:27 UTC 2018"
title = "Dagger + ViewModel todo"
tags = ["android", "dagger", "jetpack"]
blogimport = true
type = "post"
draft = true
+++

(この記事はDaggerとJetpackのViewModelをある程度知っている前提で進んでいきます😃)

Dagger + ViewModelを同時に使うのは少しツラミがあります。それは、ViewModelのインスタンス生成はViewModelProviderを介して行う必要があるためです。

例えば、次のコードは間違っています。

```kotlin
class MainViewModel @Inject constructor(...): ViewModel()

class MainActivity {
    @Inject lateinit var viewModel: MainViewModel

    ...
}
```

この書き方だとMainViewModelのインスタンス生成はViewModelProviderを介して行われません。なので次のように書く必要があります。

```kotlin
class MainViewModel(...): ViewModel()

@Module
class MainActivityModule {
    @Provides
    fun provideMainViewModel(...) : MainViewModel {
        return ViewModelProviders.of(...).get(MainViewModel::class.java)
    }
}

class MainActivity {
    @Inject lateinit var viewModel: MainViewModel

    ...
}
```

`@Provides`を使いインスタンス生成の方法を記述します。これで、ViewModelProviderを介してインスタンス生成をすることが出来ます。

また、ViewModelを直接注入せずに、ViewModelProvider.Factoryを注入して、ViewModelのインスタンス生成はActivity（or Fragment）に任せる方法があります。
このパターンを使うときは、activity-ktx（or fragment-ktx）に追加された拡張関数と組み合わせるといい感じに書けます。

```kotlin
class MainViewModel(...): ViewModel()
or
class MainViewModel @Inject constructor(...): ViewModel()

@Module
class MainActivityModule {
    @Provides
    fun provideViewModelFactory(...) : ViewModelProvider.Factory {
        return object: ViewModelProvider.Factory {
            ...
        }
    }
}

class MainActivity : AppCompatActivity() {
    @Inject lateinit var factory: ViewModelProvider.Factory
    private val viewModel: MainViewModel by viewModels { factory }

    ...
}
```

activity-ktxにある`viewModels`拡張関数を使ってViewModelインスタンスを生成します。これでViewModelのライフサイクルを保つことができます！

## 補足

結局どっちの書き方がいいの？って話になると思うんですが、一長一短かなと思ってます。

両方ともソースコード、ノウハウともに出ているので参考リンクを張っておきます。

[plaid](https://github.com/nickbutcher/plaid/blob/master/dribbble/src/main/java/io/plaidapp/dribbble/dagger/DribbbleModule.kt#L43)
- `@Provides`を使い、ViewModelを直接Injectするパターン
- 各ViewModelごとにFactoryクラスを定義する必要があるので記述量は多い
- コンパイル時にViewModelがProvideされていなかったらエラーになる
- ViewModelを直接Inject出来るので使い側からすると間違った使い方は出来ない（はず）

[Android ViewModel and FactoryProvider: good way to manage it with Dagger Multibindings](https://medium.com/@marco_cattaneo/android-viewmodel-and-factoryprovider-good-way-to-manage-it-with-dagger-2-d9e20a07084c)
- Daggerのmultibindingsを使うパターン
- 最初に仕組みを入れてしまえば、のちのちの記述量は少ない
- multibindingsを使っているのでランタイム時に落ちる可能性がある
- ViewModelを直接注入する書き方が可能だが、その書き方をするとViewModelProviderを介さないので正しくない。間違った書き方が出来る

[Activity-Ktx + Dagger Example](https://github.com/satoshun-android-example/ActivityKtxDaggerExample/tree/master/app/src/main/java/com/github/satoshun/example/sample)
- 汎用的なViewModelProvider.Factoryを作り、それを使うパターン
    - 僕のサンプルコードです😃
- 最初に汎用クラスを入れてしまえば、のちのちの記述量は少ない
- コンパイル時にViewModelがProvideされていなかったらエラーになる
- ViewModelを直接注入する書き方が可能だが、その書き方をするとViewModelProviderを介さないので正しくない。間違った書き方が出来る

個人的には一番最後の自分のパターンを押したいところですが、上記のパターンはそれぞれメリット/デメリットがあると思うので、プロジェクトによって使い分けるのがよいと思います。
