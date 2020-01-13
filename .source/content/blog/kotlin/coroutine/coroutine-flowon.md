+++
date = "Mon Jan 13 10:41:05 UTC 2020"
title = "Kotlin: FlowのflowOnオペレータの挙動"
tags = ["kotlin", "coroutine", "flow"]
blogimport = true
type = "post"
draft = true
+++

CoroutineのFlowにはflowOnオペレータが定義されており、これを使うことで、実行コンテキストを変更することが出来ます。

この記事はRxJava2のsubscribeOnとの挙動の違いを見ていきたいと思います。

## 動かしてみる

まずは、Flowから。

```kotlin
withContext(Dispatchers.Main) {
  flowOf(1)
    .map {
      // Dispatchers.Defaultで動く
      println("${Thread.currentThread()}")
      it
    }
    .flowOn(Dispatchers.Default)
    .map {
      // Dispatchers.Mainで動く
      println("${Thread.currentThread()}")
      it
    }
    .collect {
      // Dispatchers.Mainで動く
      println("${Thread.currentThread()}")
    }
}
```

このように、それぞれ動きます。flowOnはdownstreamには影響しないためです。

次にRxJavaです。

```kotlin
Single
  .just(1)
  .map {
    // Schedulers.io上で動く
    println("${Thread.currentThread()}")
    it
  }
  .subscribeOn(Schedulers.io())
  .map {
    // Schedulers.io上で動く
    println("${Thread.currentThread()}")
    it
  }
  .subscribe({
    // Schedulers.io上で動く
    println("${Thread.currentThread()}")
  }) {}
```

RxJavaではsubscribeOnは、特に途中でスケジューラーの変更が無い限り同一のスレッドで動作します。
なので、`subscribeOn(Schedulers.io())`を設定すると、observerもSchedulers.io上で実行されます。

RxJavaと同じ感覚で実装すると、間違ってしまうので注意が必要です。

ただ、Flowのmapなどのオペレーターはsuspend関数を取るので、基本的にはスレッドを意識しなくても良いはずです。

## まとめ

- flowOnとRxJavaのsubscribeOn、動作がそれぞれ異なるので注意が必要:D
