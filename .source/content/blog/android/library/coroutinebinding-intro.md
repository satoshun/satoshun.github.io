+++
date = "2018-05-19T00:00:00Z"
title = "CoroutineBindingライブラリを作りました"
tags = ["android","kotlin"]
blogimport = true
type = "post"
+++

CoroutineBindingライブラリを作ったのでその紹介です。
https://github.com/satoshun/CoroutineBinding

Android開発でCoroutineの流れが来ていると思っていて、[RxBinding](https://github.com/JakeWharton/RxBinding)のような感じで、
CoroutineフレンドリーにViewのイベントを受け取れたら便利そうだなと思い作りました。


## 使い方

例えばclickのイベントを受け取りたいとします。
CoroutineBindingでは以下のように書くことが出来ます。

```kotlin
val root = findViewById<ViewGroup>(R.id.root)
launch(UI) {
    for (click in root.clicks()) {
    Log.d("clicked", click.toString())
    }
}
```

```kotlin
fun View.clicks(capacity: Int = 0): ReceiveChannel<Unit>
```

が拡張関数で定義されており、RxBinding-kotlinのように使うことが出来ます。
他のAPIに関してもRxBindingに準拠しているため、RxBindingを使ったことがある人は自然に使えるようになっています。

## まとめ

- Coroutineを使っている人なら便利に使えると思うので、ぜひ使って下さい!
- https://github.com/satoshun/CoroutineBinding 何かあればissueや、PRを貰えると嬉しいです
