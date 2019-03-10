+++
date = "Sun Mar 10 09:57:47 UTC 2019"
title = "Android: JetpackのCoroutine Support対応について"
tags = ["android", "jetpack", "coroutine", "ktx", "kotlin"]
blogimport = true
type = "post"
draft = true
+++

Jetpackのいくつかのライブラリでは、Kotlin Coroutineのサポートが入っていますが、
どのライブラリで対応が進んでいるか気になったので、軽くまとめます。（使い方については言及しません。）

2019年3月10日現在の調査結果になります。

Lifecycle

```kotlin
val Lifecycle.coroutineScope: CoroutineScope
```

LifecycleOwner

```kotlin
val LifecycleOwner.lifecycleScope: CoroutineScope
```

ViewModel

```kotlin
val ViewModel.viewModelScope: CoroutineScope
```

WorkManager
```kotlin
abstract class CoroutineWorker(
  appContext: Context,
  params: WorkerParameters
) : ListenableWorker(appContext, params) {
  abstract suspend fun doWork(): Result
}
```

Room
```kotlin
@Dao
interface HogesDao {
  @Insert
  suspend fun add(hoge: Hoge)

  @Query("SELECT * FROM hoge WHERE id = :id")
  suspend fun get(id: String): Hoge

  ...
}
```
