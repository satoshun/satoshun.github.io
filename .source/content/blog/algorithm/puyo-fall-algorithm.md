+++
date = "Sun Jan 12 03:45:23 UTC 2020"
title = "ぷよぷよアプリ開発日記: ぷよを発火させた後に、宙に浮いたぷよを下に落下させる部分のコード"
tags = ["algorithm", "puyopuyo", "android"]
blogimport = true
type = "post"
draft = true
+++

ぷよぷよは4個以上のぷよを連結させて消すことができます。その際に、上に乗っているぶよは下に落ちてきます。

{{< figure src="" width="300" >}}
{{< figure src="" width="300" >}}

まず、最初に、ぷよぷよは2次元配列で保持しています。

- `Array<Array<Puyo>>`: 現状のぷよ盤を表す。6 * 13の配列

安定したソートを使う方法で実現することにしました。

```kotlin
private fun fallPuyo(puyos: Array<Array<Puyo>>) {
  (0..5).forEach { x ->
    (0..12)
      .map { puyos[it][x] }
      .sortedBy {
        if (it == Puyo.EMPTY) 0
        else 1
      }
      .forEachIndexed { y, puyo ->
        puyos[y][x] = puyo
      }
  }
}
```
