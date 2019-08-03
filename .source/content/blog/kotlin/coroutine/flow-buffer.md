+++
date = "Sat Aug  3 05:39:22 UTC 2019"
title = "Kotlin: FlowのBufferの指定について"
tags = ["kotlin", "coroutine", "flow"]
blogimport = true
type = "post"
draft = true
+++

CoroutineのFlowにはBufferを指定することができます。このBufferがどのように動くかを実際に動かしながら試してみました。

この記事は、少しFlowを触ったことあるぞ！って言う人向けになります。

## Bufferの種類

- UNLIMITED
- RENDEZVOUS
- CONFLATED
- BUFFERED
- 100など、具体的なBufferの値

の5種類の指定方法があります。

## 検証コード

```kotlin
// flowストリームの作成
val f = flow {
  repeat(100) {
    delay(1)
    emit("$it")
  }
}

// observeする
lifecycleScope.launch {
  f
//  .buffer(...) ここでBufferを指定する
    .collect {
      delay(100)
      Log.d("t4", "$it")
    }
}
```

まず、flowメソッドを使って、Flowストリームを作ります。このFlowは1つの値を送るのに`delay(1)`かかります。なので、約1ms処理に時間がかかります。
次に、作ったflowをobserveします。observe側では`delay(100)`しています。

- Consumerの処理時間 > Producerの処理時間

を擬似的に作り出しています。この状態で、Bufferを指定したらどのように挙動が変わるかを見ていきます。

### UNLIMITED

```kotlin
f
  .buffer(Channel.UNLIMITED)
  .collect {
    delay(100)
    Log.d("t4", "$it")
  }
```

Bufferのサイズが無制限なので、Consumer側で処理が終わってなくても値を送ろうとしていきます。ただ、それはflowの内部で一旦保管されて、Consumerの現在のタスクが終わったら、内部で保存したデータが1つずつ流れてきます。

### RENDEZVOUS

```kotlin
f
  .buffer(Channel.RENDEZVOUS)
  .collect {
    delay(100)
    Log.d("t4", "$it")
  }
```

RENDEZVOUSは協調的に動かすためのBufferです。Consumer側が処理出来るタイミングで、値を送ってきます。
