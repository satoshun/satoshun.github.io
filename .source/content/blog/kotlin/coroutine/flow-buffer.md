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
  - 実質BUFFEREDと同じ挙動をする

の5種類の指定方法があります。

以下で、それぞれの挙動について検証していきます。この検証はCoroutine 1.3.0-RCで行っております。

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

- ProducerはConsumerの状態に関わらず、値を流してくる
- Consumerはすべての値を、順番に受け取ることが出来る

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

- Producerは、Consumerが空いていることを確認して値を流してくる。
- Consumerはすべての値を、順番に受け取ることが出来る

### CONFLATED

```kotlin
f
  .buffer(Channel.CONFLATED) // or conflate()
  .collect {
    delay(100)
    Log.d("t4", "$it")
  }
```

CONFLATEDはBufferに値が溜まっていたら、その値を上書きするBufferです。最新の値は取得できますが、古い値は削除されてしまいます。

- ProducerはConsumerの状態に関わらず、値を流してくる。古い値をガンガン上書きしてくる。
- Consumerが遅い場合、途中の値を失う可能性がある

### BUFFERED

```kotlin
f
  .buffer(Channel.BUFFERED) // or 100などの具体的な値
  .collect {
    delay(100)
    Log.d("t4", "$it")
  }
```

BUFFEREDは最大16個までキャッシュすることができます。バッファが満杯になったら、バッファの値が消費されるまで、Producerは一時停止します。

- Producerは、Bufferの限界に達したら一時停止する
- Consumerは、すべての値を順番通りに受け取ることが出来る

## まとめ

- UNLIMITED、BUFFERED、RENDEZVOUSはすべての値をConsumerが受け取ることが出来る
  - Producerが無限に値を作り出す場合、UNLIMITEDはメモリを枯渇させる可能性がありそう
- CONFLATEDはすべての値を受け取ることができない可能性がある
  - Consumerの処理が重くて、最新の値のみにしか興味ない場合は有効
- BUFFERED、RENDEZVOUSはConsumerの状態に応じてProducerが待つ
