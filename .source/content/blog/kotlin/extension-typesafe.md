+++
date = "2018-11-03"
title = "拡張関数 + ジェネリック型でよりタイプセーフを得る"
tags = ["kotlin"]
blogimport = true
type = "post"
+++

Kotlinの拡張関数の話です。

以下のクラスがあったとします。

```kotlin
class A<T>(val value: T) {
  fun isNull(): Boolean {
    return value != null
  }
}

fun main() {
  val a1: A<Int> = ...
  a1.isNull()
}
```

このとき、`a1.isNull()`の結果は自明です。なぜなら`A<Int>`で宣言している時点でnonNullが確定しているためです。

```kotlin
fun main() {
  val a2: A<Int?> = ...
  a2.isNull()
}
```

このとき、`a2.isNull()`の結果は自明ではありません。
なぜなら`A<Int?>`でジェネリック型を宣言しているので、nullableな値が入ってくる可能性があるためです。

この2つの例から、`a1.isNull()`はそもそもnonNullなので`isNull()`メソッドをコールできないほうが良いのではないか？という考えが浮かびます。

Kotlinの拡張関数を使うことで達成できます。

```kotlin
class A<T>(val value: T)

// nullableのときにコールできるようにする
fun <T : Any> A<T?>.isNull(): Boolean {
  return value != null
}

fun main() {
  val a1: A<Int> = ...
  a1.isNull() // コンパイルエラー!!

  val a2: A<Int?> = ...
  a2.isNull()
}
```

`fun <T : Any> A<T?>.isNull()`と宣言することでnullable時のみコールできます。

ジェネリック型がnullable時のみコールしたい場合に有効なテクニックでした😃

## 参考

- http://blog.nightlynexus.com/kotlin-extension-members-are-great-even-for-types-you-do-control/
