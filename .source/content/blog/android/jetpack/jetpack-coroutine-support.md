+++
date = "Sun Mar 10 10:14:21 UTC 2019"
lastmod = "Sun Mar 10 23:08:46 UTC 2019"
title = "Android: JetpackのCoroutine Supportについて"
tags = ["android", "jetpack", "coroutine", "ktx", "kotlin"]
blogimport = true
type = "post"
draft = false
+++

Jetpackのいくつかのライブラリでは、Kotlin Coroutineのサポートが入っていますが、
どのライブラリで対応が進んでいるか気になったので、軽くまとめます。使い方については言及しません。

以下、~~2019年3月10日~~ 2019年3月11日の調査結果になります。
また、これらは、supportライブラリのリポジトリから取ってきたので、現在リリースされているかどうかは不明です。

Lifecycle

```kotlin
// Lifecycleに従うCoroutineScopeの生成
val Lifecycle.coroutineScope: CoroutineScope
```

LifecycleOwner

```kotlin
// LifecycleOwnerに従うCoroutineScopeの生成
val LifecycleOwner.lifecycleScope: CoroutineScope
```

ViewModel

```kotlin
// ViewModelに従うCoroutineScopeの生成
val ViewModel.viewModelScope: CoroutineScope
```

WorkManager

```kotlin
abstract class CoroutineWorker(
  appContext: Context,
  params: WorkerParameters
) : ListenableWorker(appContext, params) {
  // suspendメソッドで定義された
  abstract suspend fun doWork(): Result
}
```

Room

```kotlin
// Dao内でsuspendメソッドが使える
@Dao
interface HogesDao {
  @Insert
  suspend fun add(hoge: Hoge)

  @Query("SELECT * FROM hoge WHERE id = :id")
  suspend fun get(id: String): Hoge

  ...
}
```

## まとめ

- Lifecycle、LifecycleOwner、ViewModelはそれらのライフサイクルに従う、CoroutineScopeの生成が出来る
- WorkManagerのdoWorkがsuspendメソッドになった
  - doWorkの中で、他のsuspendメソッドがコールできる
- RoomのDao内でsuspendメソッドが定義出来る

結構対応がされていた😃😃😃

## 補足

- LiveDataへのサポートも入るかも
  - [WIP corutine live data (Id0e47973) · Gerrit Code Review](https://android-review.googlesource.com/c/platform/frameworks/support/+/890736)
- Lifecycleに特定のstateに入った時に実行される拡張関数群が入りそう
  - [Lifecycle Dispatcher (Ib1211c0f) · Gerrit Code Review](https://android-review.googlesource.com/c/platform/frameworks/support/+/905134)
