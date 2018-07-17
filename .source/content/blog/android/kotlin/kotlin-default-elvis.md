+++
date = "2018-07-17T00:00:00Z"
title = "Kotlin: 最初にエルビス演算子を使うときれいに書ける"
tags = ["android", "kotlin"]
blogimport = true
type = "post"
draft = true
+++

Kotlinではエルビス演算子を使うことで、左辺の値がnullだった時に処理を処理を実行することが出来ます。

```kotlin
val a: String? = ...

val b = a ?: "default" // aがnullなら"default"を代入する
```

List?を受け取る場合に、`emptyList()` をエルビスで指定することが出来ます。

```kotlin
items
    ?.map {
        ...
    }
    ?.filter {
        ...
    }
    ?: emptyList()
```

僕は、今まで最後にelvisを使っていたのですが、最初に使ったほうがきれいで読みやすいと思います。

```kotlin
(items ?: emptyList())
    .map {
        ...
    }
    .filter {
        ...
    }
```

## まとめ

メソッドチェインの長さなど考慮して、最初にelvisを使ってデフォルトの値を与えるか、
最後にelvisを使うかを意識するときれいなコードが書けそうだと思いました。
