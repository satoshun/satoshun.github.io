+++
date = "Sat Oct 30 08:58:01 UTC 2021"
title = "Kotlin: functional interfaceを使うと、少し効率が良いコードが生成される"
tags = ["kotlin"]
blogimport = true
type = "post"
draft = true
+++

Kotlinでは、SAM変換を使うために、functional interfaceが用意されています。

```kotlin
// ----- normal interface -----
interface IntPredicate {
  fun accept(i: Int): Boolean
}

val isEven = object : IntPredicate {
  override fun accept(i: Int): Boolean {
    return i % 2 == 0
  }
}


// ----- use functional interface -----
fun interface IntPredicate2 {
  fun accept(i: Int): Boolean
}

val isEven = IntPredicate2 { it % 2 == 0 }
```

この functional interfaceを使うと、生成されるコードが少し効率良くなります。
具体的には、インスタンスが一度しか作られません。

具体例を上げて説明します。
次のコードはメソッドを呼び出すたびに、インスタンスが生成されます。

```kotlin
fun a() {
  val isEven = object : IntPredicate {
    override fun accept(i: Int): Boolean {
      return i % 2 == 0
    }
  }
  println(isEven.hashCode())
}
```

次のコードは、
何回メソッドを呼び出しても、同一のインスタンスが使用されます。


```kotlin
fun a() {
  val isEven = IntPredicate2 { it % 2 == 0 }
  println(isEven.hashCode())
}
```

functional interfaceを使うと、インスタンスの生成が抑制でき、少し効率が良くなります。


## 余談

状態（スコープ）を持つfunctional interfaceの場合には、同一のインスタンスを使わずに、
毎回インスタンスが生成されます。具体的には次のようなコードです。

```kotlin
fun a(value: Int) {
  val calculate = IntPredicate2 { (it + value) * 2 }
  println(calculate.hashCode())
}
```

## 参考

- https://kotlinlang.org/docs/fun-interfaces.html
