+++
date = "2018-10-16"
title = "Kotlinで3つの関数のパラメータを省略する"
tags = ["kotlin"]
blogimport = true
type = "post"
+++

Kotlinでは拡張関数が定義されており、拡張関数を使うことで1つ関数のパラメータを省略できます。

```kotlin
fun hoge(a: String) {
  println(a)
}

->

fun String.hoge() {
  println(this)
}
```

次に、インターフェース（クラス）内で拡張関数を定義することで、さらに1つの関数のパラメータを省略できます。

```kotlin
interface User

fun hoge(a: String, b: User) {
  println(a)
  println(b)
}

->

interface User {
  fun String.hoge() {
    println(this)
    println(this@User)
  }
}
```

さらに、reified type parameterを使うことで、関数のパラメータを省略できます。

```kotlin
fun <T> hoge(a: String, b: User, c: Class<T>) {
  println(a)
  println(b)
  println(c)
}

->

class User {
  inline fun <reified T> String.hoge() {
    println(this)
    println(this@User)
    println(T::class)
  }
}
```

interfaceだと、inline関数が使えないのでクラスで定義してあります。

## まとめ

- Kotlinの拡張関数、reified便利😊
