+++
date = "2018-12-17"
title = "Truthほげほげ"
tags = ["android", "test", "truth"]
blogimport = true
type = "post"
draft = true
+++

[Truth](http://google.github.io/truth/)はGoogleが開発をしているテストアサーションライブラリです。

従来のアサーションに比べ、大きく2つの利点があります。

- readableにアサーションが書ける
- エラーメッセージが明確

## readableにアサーションが書ける

ドキュメントのbenefitにあるサンプルを取り上げ説明します。
http://google.github.io/truth/benefits

まず従来のJUnitスタイルを使って書きます。

```java
Optional<String> middleName = user.getMiddleName();
assertFalse(middleName.isPresent());
```

assertFalse、isPresentを使っており、否定のアサーションなので少し悩みます。（個人差はあります）

これがTruthを使うと次のようになります。

```java
Optional<String> middleName = user.getMiddleName();
assertThat(middleName).isAbsent();
```

`middleName`がabsent、値が存在しないことをテストしたいんだなということが、より強く伝わります。

上記の`isAbsent`はOptionalのために用意されたアサーションです。
当然、Optional以外にも、assertThatに渡した引数に適したアサーションを使うことが出来ます。
例えばIterableなら、`containsAnyIn`や`isEmpty`などが用意されています。

