+++
date = "Sun Dec 16 10:56:39 UTC 2018"
title = "Kotlin: Coroutine withContext"
tags = ["kotlin", "coroutine"]
blogimport = true
type = "post"
draft = true
+++

withContextは`CoroutineContext`を切り替えるためのメソッドで、次のAPIで定義されています。

```kotlin
public suspend fun <T> withContext(
    context: CoroutineContext,
    block: suspend CoroutineScope.() -> T
): T
```

withContextの比較対象としてasyncも載せておきます。

```kotlin
public fun <T> CoroutineScope.async(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> T
): Deferred<T>
```

- suspendなので、ここで中断する
- CoroutineContextでdispatchする先を変更する

withContextはapi側で使い、Coroutine Builderの中では使わないイメージ
- リアルワールドとCoroutineを結ぶために使うイメージ。
  - その際、リアルワールドとsuspendを組み合わせることでwithContextを呼び出すことが出来る
