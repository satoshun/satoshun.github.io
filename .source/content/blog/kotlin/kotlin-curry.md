+++
date = "2017-12-09"
title = "Kotlinでカリー化を表現する"
tags = ["kotlin"]
blogimport = true
type = "post"
draft = false
+++

おはこんばんわ。この記事はきらぴかAdvent Calendarの2日目になります。

この記事ではKotlinでカリー化を実装する方法について解説します。カリー化の細かい定義には触れません。詳しい説明は https://ja.wikipedia.org/wiki/%E3%82%AB%E3%83%AA%E3%83%BC%E5%8C%96 を参照して下さい。


## さっそくコードから

Kotlinで汎用的なカリー化のコードをどのように書けば良いのだろうと? 調べていたらとても良いコードがありました。
ここでは3つの引数を取る関数をカリー化する例を紹介します。

```kotlin
fun <P1, P2, P3, R> ((P1, P2, P3) -> R).curried(): (P1) -> (P2) -> (P3) -> R {
	return { p1: P1 -> { p2: P2 -> { p3: P3 -> this(p1, p2, p3) } } }
}
```

(ここから一部抜粋 :pray: : https://github.com/MarioAriasC/funKTionale/blob/master/funktionale-currying/src/main/kotlin/org/funktionale/currying/namespace.kt)

このコードは3つの引数取る関数(`(P1, P2, P3) -> R`)があり、それを` (P1) -> (P2) -> (P3) -> R`に変換するものになります。たったこれだけで引数を3つ取る関数(Function3)から1つの引数取る関数(Function1)のチェインに変換しています。


## コードの説明

最初にこのコードを見たときに返り値の `(P1) -> (P2) -> (P3) -> R ` ってなんだろう? と思いました。
これは実装の動作で説明すると

```kotlin
fun hoge(a: Int, b: String, c: Float) : String {
	return a.toString() + b + c.toString()
}

val a1 = ::hoge.curried() // 関数 (P1) -> (P2) -> (P3) -> Rに変換
val a2 = a1(10) // (P1)を適用し、関数(P2) -> (P3) -> Rに変換
val a3 = a2("test") // (P2)を適用し、関数(P3) -> Rに変換
val result = a3(1f) // (P3)を適用し、値Rに変換
```

となります。分かりやすさのために括弧をつけてあげると `(P1) -> ((P2) -> ((P3) -> R))`となり 、Function1関数が連続であると評価されるので、複数の引数を取る関数を1つの引数を取る関数に変換することが出来ます。

## まとめ

- Kotlinでは関数がファーストクラスなのでこのような柔軟な書き方が可能
- https://github.com/MarioAriasC/funKTionale はKotlin + 部分適用のコードなどもあり、非常に良いライブラリ
    - Commandパターンとか楽に、汎用的に実装できる
