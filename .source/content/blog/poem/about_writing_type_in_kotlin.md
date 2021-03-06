+++
date = "2018-04-16T22:50:00Z"
title = "Kotlin: ローカルで明示的に型を宣言することについて"
tags = ["poem", "android", "kotlin"]
blogimport = true
type = "post"
+++

ポエムです。

## 結論

基本的にローカルで明示的に型を宣言するのは好ましくない

## 問題

Kotlinではval(var)で変数定義することができ、型宣言を省略することが出来ます。

```kotlin
val userName = dataSource.getUserName()
```

型宣言するのと、省略するのどちらが良いのか考えると、多くの場合、型宣言をしないほうが優れていると思います。なぜかというと、型宣言が必要ということは、型宣言をしないと人間が理解できないほど複雑なコードを書いていることを意味するからです。適切なメソッド、適切なクラスが欠けている可能性があります。

Kotlinにおいて型宣言を書くことはコメントを書くことに等しいと思います。多くの場合は冗長です。

## まとめ

- ローカルで型宣言が欲しくなるコードがあったら、それは複雑すぎるコードの匂いがする
