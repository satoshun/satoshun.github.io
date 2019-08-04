+++
date = "Sat Aug  3 23:37:02 UTC 2019"
title = "Kotlin: FlowのBufferの指定について"
tags = ["kotlin", "coroutine", "flow"]
blogimport = true
type = "post"
draft = false
lastmod = "Sun Aug  4 03:16:20 UTC 2019"
+++

CoroutineのFlowにはBufferを指定することができます。Bufferの指定によってどのように動作が変わるかを試してみました。

この記事は、少しFlowを触ったことあるぞ！って言う人向けになります。また、検証はCoroutine 1.3.0-RCで行っております。

## Bufferの種類

Bufferの種類は5つあります。

- UNLIMITED
- RENDEZVOUS
- CONFLATED
- BUFFERED
- 50、100などの、具体的な値
  - 実質、BUFFEREDと同じ挙動をする

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
    .buffer(...) ここでBufferを指定する
    .collect {
      delay(100)
      Log.d("t4", "$it")
    }
}
```

このコードは、まず最初に、flowメソッドを使って、Flowストリームを作ります。これをこの記事ではProducer（生産者）と呼びます。このProducerは1つの値を送るのに約`delay(1)`かかります。なので、処理に約1msかかります。

次に、作ったflowをobserveします。observeする側をConsumer（消費者）と呼びます。Consumerでは`delay(100)`しているので、処理に約100msかかります。

これは、次の関係を作り出しています

- Consumerの処理時間 > Producerの処理時間

この状態で、Bufferを指定したらどのように挙動が変わるかを見ていきます。

### UNLIMITED

```kotlin
f
  .buffer(Channel.UNLIMITED)
  .collect {
    delay(100)
    Log.d("t4", "$it")
  }
```

UNLIMITEDはBufferのサイズを無制限にします。

Consumer側で処理が終わってなくても値を送ります。その値は、Consumerが現在のタスクをしていたら、flowの内部で一旦保存されます。そして、Consumerの現在のタスクが完了したら、内部で保存したデータが1つずつ流れてきます。

- Producer: Consumerの状態に関わらず、値を流してくる。Consumerの処理が終わっていないときは、内部キャッシュに保存されていく
- Consumer: すべての値を順番通りに受け取ることが出来る

### RENDEZVOUS

```kotlin
f
  .buffer(Channel.RENDEZVOUS)
  .collect {
    delay(100)
    Log.d("t4", "$it")
  }
```

RENDEZVOUSは協調的に動かすためのBufferです。

Consumer側が処理出来るタイミングで、値を送ってきます。Consumserが現在のタスクをしていたら、一時停止します。

- Producer: Consumerが空いていることを確認して値を流してくる
- Consumer: すべての値を順番通りに受け取ることが出来る

### CONFLATED

```kotlin
f
  .buffer(Channel.CONFLATED) // or conflate()
  .collect {
    delay(100)
    Log.d("t4", "$it")
  }
```

CONFLATEDはBufferに値が溜まっていたら、その値を上書きするBufferです。

最新の値は取得できますが、古い値は削除されてしまいます。

- Producer: Consumerの状態に関わらず、値を流してくる。Consumerの処理が終わっていないときは、内部キャッシュを上書きする
- Consumer: 処理に時間がかかる場合は途中の値を失う。

### BUFFERED

```kotlin
f
  .buffer(Channel.BUFFERED) // or 100などの具体的な値
  .collect {
    delay(100)
    Log.d("t4", "$it")
  }
```

BUFFEREDは最大16個までキャッシュすることが出来るBufferです。

バッファが満杯になったら、バッファの値が消費されるまで、Producerは一時停止します。

- Producer: Bufferの限界に達したら一時停止する。限界に達するまでは値を流し続ける
- Consumer: すべての値を順番通りに受け取ることが出来る

## まとめ

- UNLIMITED、BUFFERED、RENDEZVOUSはすべての値をConsumerが受け取ることが出来る
  - Producerが無限に値を作り出す場合、UNLIMITEDはメモリを枯渇させる可能性がありそう
- CONFLATEDはすべての値を受け取ることができない可能性がある
  - Consumerの処理が重くて、最新の値のみにしか興味ない場合は有効
- BUFFERED、RENDEZVOUSはConsumerの状態に応じてProducerが待つ
