+++
date = "Sun Aug 22 12:58:47 UTC 2021"
title = "Jetpack Compose: Stable型メモ書きその1"
tags = ["android", "jetpack", "compose"]
blogimport = true
type = "post"
draft = true
+++

Stable型、非Stable型の動作が良く分からなかったので、動作を見ながら確認してみました。

この記事は、メモ程度の内容なので、特にまとまりがない内容になっております。

## recompositionされる場合、されない場合を見ていく

される

```kotlin
class A(var i: Int)

@Composable
fun Test() {
    Test(a = A(i = 10))
}

@Composable
fun Test(a: A) { ... }
```

---

される

```kotlin
class A(val i: Int)

@Composable
fun Test() {
    Test(a = A(i = 10))
}

@Composable
fun Test(a: A) { ... }
```

---

