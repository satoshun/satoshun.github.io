+++
date = "Wed Jun 19 00:48:27 UTC 2019"
title = "Kotlinのwildcard importと拡張関数"
tags = ["kotlin", "codestyle"]
blogimport = true
type = "post"
draft = false
+++

Kotlinの公式?ではwildcard importを多様しているなって思っていました。

Roman Elizarovさんが、[Extension-oriented design](https://medium.com/@elizarov/extension-oriented-design-13f4f27deaee)中でちらっとメリットについて話していました。

> You might notice that our Kotlin code usually uses wildcard imports like import com.example.*. It is handy in Kotlin, because importing just a class in Kotlin is rarely enough. All the useful, convenient, utility functions are typically defined in the same package but outside of the class as extension functions.

まとめると、Javaとは違い、Kotlinでは拡張関数があります。拡張関数の多くは、クラスと同じパッケージに定義されますが、拡張関数なのでクラスの外側に配置されます。wildcard importでは外側に配置された、拡張関数もまとめてimportすることが出来るため便利です。

という理由らしいです。

## 雑談: wildcard importのデメリット

- 名前衝突が起きやすい
- どのような依存があるのかが分かりにくい
  - レビューのしずらさ
- 無駄なクラスへの依存が生まれる
