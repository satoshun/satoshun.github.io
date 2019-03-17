+++
date = "Sun Mar 17 05:17:23 UTC 2019"
title = "Kotlin: CoroutineでRxJavaのzipっぽいものを表現する"
tags = ["kotlin", "coroutine", "rxjava"]
blogimport = true
type = "post"
+++

Coroutineで非同期処理を並列に処理したいとします。例外を考慮しないなら単純にasyncで包めば良いです。

```kotlin
launch {
  val task1 = async { MainService.task1() }
  val task2 = async { MainService.task2() }

  println("${task1.await()}\n${task2.await()}")
}
```

asyncで包むことで、並列に処理をすることができます。

次に、各非同期処理が例外を吐く場合を考えてみます。その場合は、呼び出し元で`runCatching`を使います。

```kotlin
launch {
  val task1 = async { runCatching { MainService.task1() } }
  val task2 = async { runCatching { MainService.task2() } }

  val result1 = task1.await()
  val result2 = task2.await()
  println("$result1\n" + "$result2\n")
}
```

`runCatching`を使うことで、呼び出し先で例外が起こったとしても、処理を継続することが出来ます。

---

また、次のように書くことは出来ません。

```kotlin
launch {
  val task1 = async { MainService.task1() }
  val task2 = async { MainService.task2() }

  // awaitのタイミングでrunCatchingを使う
  val result1 = runCatching { task1.await() }
  val result2 = runCatching { task2.await() }
  println("$result1\n" + "$result2\n")
}
```

デフォルトの設定だと例外は伝搬し、子が倒れたら親も倒れます。

しかし、`supervisorScope`を使うと、上記の振る舞いを防ぐことができます。

```kotlin
launch {
  supervisorScope {
    val task1 = async { MainService.task1() }
    val task2 = async { MainService.task2() }

    // supervisorScopeを使っているのでこのタイミングでもおｋ
    val result1 = runCatching { task1.await() }
    val result2 = runCatching { task2.await() }
    println("$result1\n" + "$result2\n")
  }
}
```

`supervisorScope`を使うと、`Deferred.await`のタイミングで例外のハンドリングをすることが出来ます。

---

最後にキャンセルについて考えてみます。

任意の子ジョブがキャンセルされても処理を続けたいとしたら、前述のsupervisorScopeを使う必要があります。

```kotlin
launch {
  supervisorScope {
    val task1 = async { MainService.task1() }
    val task2 = async { MainService.task2() }

    val result1 = runCatching { task1.await() }
    val result2 = runCatching { task2.await() }
    println("$result1\n" + "$result2\n")
  }
}
```

このように書くことで、例えばtask1がキャンセルされたとしても、他の部分の処理を継続することが出来ます。

## まとめ

- エラー & キャンセル時にも処理を継続したいならsupervisorScope + runCatchingを使う
- エラー時のみ処理を継続したいなら、runCatchingだけで対応できる
- 正常系のみなら、asyncで包んであげるだけで良い
