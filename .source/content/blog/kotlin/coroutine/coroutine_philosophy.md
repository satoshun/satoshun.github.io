+++
date = "2018-12-01"
title = "Kotlin: Coroutine?"
tags = ["kotlin", "coroutine"]
blogimport = true
type = "post"
draft = true
+++

Kotlin Coroutineは様々な新しい概念を持ち込んだため、何をしていいのか悩みがちです。

https://medium.com/@elizarov/coroutines-are-not-about-multi-threading-at-all-1b2c6e97ec02

作者のRoman Elizarovさんがコメントで以下のように返信しています。

```
The main benefit of coroutines is that you can write asynchronous code without callbacks.

> Coroutineの最大の利点は非同期コードがコールバックなしで書ける点です。
```

"Callback Hell"を避けることが出来ます。

並行処理も多少は書きやすくなっていますが、難しいことには変わりありません。
