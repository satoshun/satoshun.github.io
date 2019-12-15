+++
date = "Sun Dec 15 07:15:43 UTC 2019"
title = "ViewModelとKotlin Coroutinesの書き方あれこれ"
tags = ["android", "kotlin", "viewmodel", "coroutine"]
blogimport = true
type = "post"
draft = false
+++

ViewModel + Kotlin Coroutineを使う場合、どんな感じでViewModelでCoroutineを表現するかについてあれこれ書いてみました。

MVVM + Repositoryを想定しており、UIに反映する部分はLiveDataを考えています。

環境は`androidx.lifecycle:lifecycle-viewmodel-ktx`は2.2.0-rc03、Coroutineは1.3.3です。

この記事は次の順序で進んでいきます。

- viewModelScopeとは?
- suspend関数をコールするとき
- Flowをコール/購読するとき

## viewModelScopeとは?

`androidx.lifecycle:lifecycle-viewmodel-ktx`ライブラリには、viewModelScope拡張関数が含まれています。定義は次の通りです。

```kotlin
/**
 * [CoroutineScope] tied to this [ViewModel].
 * This scope will be canceled when ViewModel will be cleared, i.e [ViewModel.onCleared] is called
 */
val ViewModel.viewModelScope: CoroutineScope
```

ViewModelのライフサイクルに合わせたCoroutineScopeを取得することが出来ます。
このスコープ上でCoroutineを実行すれば、ViewModelの破棄に合わせて、自動でdisposeしてくれます。

また、`viewModelScope`は、メインスレッド上で実行してくれるため、`LiveData.setValue`を使い、値を更新します。

```kotlin
val userLiveData = MutableLiveData(...)

viewModelScope.launch {
    val user = userRepository.getUser() // 適当なsuspend関数をコール

    userLiveData.setValue(user) // メインスレッド上で実行されることが保証されているのでsetValueを使う
    // userLiveData.postValues(user)
}
```

viewModelScopeを使っている場合は、postValueメソッドを使うケースは無いと思います。


## suspend関数をコールするとき

ネットワークコールなどのAPIは、suspendで表現することになると思います。
また、Retrofitでは2.6.0から、suspendでAPIを定義出来るようになりました。またRoomでもsuspend関数を使うことが可能です。

なので、Repository層での定義は次のようになります。

```kotlin
interface UserApi {
    suspend fun getUser(): User
}

class UserRepository(private val retrofitService: UserApi) {
    suspend fun getUser() : User {
        val user = retrofitService.getUser()
        ...
        return user
    }
}
```

これをViewModelからコールします。

```kotlin
viewModelScope.launch {
    val user = userRepository.getUser()
    ...
}
```

これだけだと、ネットワークの調子が悪い時などに、例外が起きてしまうので、エラーハンドリングをする必要があります。

ViewModel側でハンドリングするなら、try-catch、もしくは`runCatching`を使うのが良いと思います。

```kotlin
viewModelScope.launch {
    try {
        val user = userRepository.getUser()
        ...
    } catch (e: Exception) {
        ...
    }
}
```

```kotlin
viewModelScope.launch {
    runCatching { userRepository.getUser() }
        .onSuccess { ... }
        .onFailure { ... }
}
```

個人的には`runCatching`のほうが好きです。

また、ViewModelで例外処理をするのではなく、Repository側で適当な型で包むパターンもあると思います。例えばNetWorkResultのようなクラスがあるとします。

```kotlin
sealed class NetWorkResult<T> {
    class Success<T>(val value: T) : NetWorkResult<T>()
    class Error(val exception: Exception) : NetWorkResult<Nothing>()
    ...
}
```

このクラスをRepository側の返り値として使います。

```kotlin
class UserRepository(private val retrofitService: UserApi) {
    suspend fun getUser() : NetWorkResult<User> {
        try {
            val user = retrofitService.getUser()
            return NetWorkResult.Success(user)
        } catch (e: Exception) {
            return NetWorkResult.Error(e)
        }
    }
}
```

こうすることで、ViewModel側では、try-catchを使わなくて良くなり、try-catchの代わりにwhen式を使うことになります。

ここまでがsuspend関数の説明になります。次にFlowを返すAPIの話です。


## Flowをコール/購読するとき

Flow APIは、複数の値を流すストリームを表現することが出来ます。RxJavaで言うところのObservableとか、Flowableのようなものです。

Flowを購読するタイミングは、ViewModelのinitブロックが良いと思います。重複で購読する心配がないためです。`SavedStateHandle`を組み合わせることで、多くの場合、initブロックで初期化を行うことが出来ると思います。

```kotlin
class MyViewModel(private val state: SavedStateHandle) : ViewModel() {
    init {
        ...
    }
}
```

Flowの購読方法なんですが、`collect、collectLatest`もしくは、`launchIn`を使います。

```kotlin
class MyViewModel(...) : ViewModel() {
    init {
        viewModelScope.launch {
            repository.getFlowStream()
                .collect { ... }
        }

        repository.getFlowStream()
            .onEach { ... }
            .launchIn(viewModelScope)
    }
}
```

エラーや、完了イベントのハンドリングが必要な場合、`catch`、`onCompletion`メソッドを使います。

```kotlin
repository.getFlowStream()
    .onEach { ... }
    .catch { ... }
    .onCompletion { ... }
    .launchIn(viewModelScope)
```

個人的にはネストが少なくなるので、`launchIn`の書き方のほうが好みです。


## まとめ

- viewModelScopeメソッドを使うとViewModelの破棄に合わせてリソースを解放してくれる
- runCatchingを使うと、いい感じにエラーハンドリングが出来る
- launchInを使うと、いい感じにFlowを購読できる
- 例外が起きる可能性がある場合は、`catch`などを使ってエラーハンドリングする必要がある

Coroutineはいいぞ〜
