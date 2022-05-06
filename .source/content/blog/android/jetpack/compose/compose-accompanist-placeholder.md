+++
date = "Fri May  6 05:23:43 UTC 2022"
title = "Jetpack Compose: コンテンツにPlaceholderを適用するライブラリ"
tags = ["android", "jetpack", "compose"]
blogimport = true
type = "post"
draft = true
+++

Jetpack Composeで、コンテンツにPlaceholderを付ける方法の紹介です。
[accompanist/placeholder](https://google.github.io/accompanist/placeholder/) を使うことで簡単に実現できます。

使い方は簡単で、 `Modifier.placeholder(...)` 拡張メソッドを適用するだけです。

```kotlin
Text(
  text = "Hoge",
  modifier = Modifier
    .placeholder(visible = true)
)
```

これでこのComposableに対して、Placeholderを適用することが出来ます。

このライブラリでは、fadeとshimmerが定義されており、それぞれPlaceholderをカスタマイズすることも出来ます。
shimmerを使うと、キラリーンみたいなPlaceholderになります。
