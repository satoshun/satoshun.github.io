+++
date = "Thu May 23 23:43:39 UTC 2019"
title = "ViewModel SavedState + Dagger"
tags = ["android", "jetpack", "viewmodel", "savedstate", "dagger"]
blogimport = true
type = "post"
draft = true
+++

ViewModel + SavedStateでDaggerを使う方法を考えてみました。

SavedStateを使う場合、コンストラクタに`SavedStateHandle`を渡さなければいけません。

```kotlin
class MyViewModel(
  private val state: SavedStateHandle
) : ViewModel() {
    ...
}
```

SavedStateHandleインスタンスを作るために、`SavedStateVMFactory`もしくは、`AbstractSavedStateVMFactory`を使う必要があります。

生成したいViewModelのコンストラクタの引数が、SavedStateHandleのみならSavedStateVMFactoryを使います。

```kotlin
// thisはFragmentActivity
ViewModelProvider(this, MyViewModel(this))
  .get(MyViewModel::class.java)
```

コンストラクタの引数がSavedStateHandle以外にもあるなら、`AbstractSavedStateVMFactory`を拡張します。

```kotlin
class TestViewModel(
  private val state: SavedStateHandle,
  private val name: String
) : ViewModel()

class TestViewModelFactory(
  owner: SavedStateRegistryOwner,
  defaultArgs: Bundle? = null
) : AbstractSavedStateVMFactory(owner, defaultArgs) {
  override fun <T : ViewModel> create(
    key: String, modelClass: Class<T>,
    handle: SavedStateHandle
  ): T {
    return TestViewModel(handle, "test") as T
  }
}

// 以下、生成コード
ViewModelProvider(this, TestViewModelFactory(this))
  .get(TestViewModel::class.java)
```

こんな感じになります。

今までとは違い、AbstractSavedStateVMFactoryに`SavedStateRegistryOwner`インターフェース（実質、FragmentActivity or Fragment）を渡さなければいけません。
また、初期値が欲しい場合は、defaultArgs(Bundle)も渡す必要があります。

## Daggerでどのように使うか?

以下いろいろと書いていきます。クラス名は適当です。

### 1. AbstractSavedStateVMFactoryを生成するクラスを定義する

```kotlin
class SavedStateViewModel2(
  private val dummy: Dummy,
  private val state: SavedStateHandle
) : ViewModel() {
  class Factory @Inject constructor(private val dummy: Dummy) {
    fun create(owner: FragmentActivity): AbstractSavedStateVMFactory {
      return object : AbstractSavedStateVMFactory(owner, owner.intent.extras) {
        override fun <T : ViewModel> create(
          key: String,
          modelClass: Class<T>,
          handle: SavedStateHandle
        ): T {
          return SavedStateViewModel2(dummy, handle) as T
        }
      }
    }
  }
}

// 以下、生成コード
class MainActivity : AppCompatActivity() {
  @Inject lateinit var factory: SavedStateViewModel2.Factory
  private val viewModel by viewModels<SavedStateViewModel2> { // viewModelsはktxの拡張関数
    factory.create(this)
  }
}
```

一番シンプルな方法だと思います。`AbstractSavedStateVMFactory`を作るためのFactoryを作る感じです。

### 2. 1の方法 + FragmentActivityをBinds or Providesする

```kotlin
@Binds
fun fragmentActivity(activity: MainActivity): FragmentActivity

or 

@Provides
fun fragmentActivity(activity: MainActivity): FragmentActivity = fragmentActivity

class SavedStateViewModel5(
  private val dummy: Dummy,
  private val state: SavedStateHandle
) : ViewModel() {
  class Factory @Inject constructor(
    owner: FragmentActivity,
    private val dummy: Dummy
  ) : AbstractSavedStateVMFactory(owner, owner.intent.extras) {
    override fun <T : ViewModel> create(
      key: String,
      modelClass: Class<T>,
      handle: SavedStateHandle
    ): T {
      return SavedStateViewModel5(dummy, handle) as T
    }
  }
}

// 以下、生成コード
class MainActivity : AppCompatActivity() {
  @Inject lateinit var factory: SavedStateViewModel5.Factory
  private val viewModel by viewModels<SavedStateViewModel5> {
    factory
  }
}
```

FragmentActivityがInject可能になったので直接AbstractSavedStateVMFactoryが生成可能になりました😃

### 3. AssistedInjectを使う

SavedStateHandleがDaggerで解決しにくい値なので、[square/AssistedInject](https://github.com/square/AssistedInject)を使ってみます。

```kotlin
class SavedStateViewModel3 @AssistedInject constructor(
  @Assisted private val state: SavedStateHandle,
  private val dummy: Dummy
) : ViewModel() {

  @AssistedInject.Factory
  interface Factory {
    fun create(state: SavedStateHandle): SavedStateViewModel3
  }
}

@AssistedModule
@Module(includes = [AssistedInject_SavedStateViewModel3Module::class])
interface SavedStateViewModel3Module

// 以下、生成コード
class MainActivity : AppCompatActivity() {
  @Inject lateinit var factory3: SavedStateViewModel3.Factory
  private val viewModel3 by viewModels<SavedStateViewModel3> {
    viewModelWrapper(this) { factory3.create(it) }
  }
}

// ただの便利関数
fun <T : ViewModel> viewModelWrapper(
  owner: FragmentActivity,
  body: (state: SavedStateHandle) -> T
): AbstractSavedStateVMFactory {
  return object : AbstractSavedStateVMFactory(owner, owner.intent.extras) {
    override fun <T : ViewModel> create(
      key: String,
      modelClass: Class<T>,
      handle: SavedStateHandle
    ): T {
      @Suppress("UNCHECKED_CAST")
      return body(handle) as T
    }
  }
}
```

こんな感じになります。FragmentActivityをInject可能にすれば、もう少しいい感じに書けると思います。

## まとめ

TODO