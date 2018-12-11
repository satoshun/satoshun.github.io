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
- 失敗メッセージがわかりやすい

それぞれについて説明していきます。

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
例えばIterableなら、`containsAnyIn`や`isEmpty`などが用意されています。型ごとに一般的なテストで行うであろうアサーションが用意されており、readableにconciseに書くことが出来ます。

## 失敗メッセージがわかりやすい

こちらもbenefitにあるサンプルを取り上げます。

まずは従来にJUnitスタイルから。

```java
assertTrue(googleColors.contains(PINK));
```

この場合、失敗メッセージは特にありません。「falseを表明してる部分にtrueが来た」程度のものしかなく、原因特定するのが大変です。
失敗メッセージをカスタムすることは出来ますが、すべてのアサーションに定義するのは骨が折れます。

次にTruthスタイルです。

```java
assertThat(googleColors).contains(PINK);
```

`<[BLUE, RED, YELLOW, BLUE, GREEN, RED]> should have contained <PINK>` のようなメッセージが出ます。インスタンス情報や、こうなるべきというメッセージが含まれており原因特定がしやすくなっています。デフォルトの段階でかなり見やすい、わかりやすい失敗メッセージを出力してくれます。

以上がTruthのメリットになります。

## 補足

### Truth-Androidライブラリ

JetPackにTruth + Android用のライブラリが追加されました。これを使うことでBundle、IntentなどのAndroid固有のクラスのテストが書きやすくなります。
例えば、Intentの場合は以下のアサーションメソッドが定義されています。

```
hasComponent
hasComponentClass
hasComponentPackage
hasPackage
hasAction
hasNoAction
hasData
hasType
extras
categories
hasFlags
```

Intentの中身を確認する便利メソッドが定義されています。Truth-Androidを使うことで、よりAndroid環境でテストが書きやすくなることが期待されます。

### AssertJとの比較

これも公式ドキュメントにまとめてあります。
http://google.github.io/truth/comparison

現状の主だった差分は次のようになっています。

- Truhtはまだ1.0.0になっていないのでAPIが変わる可能性がある
- TruthはChainスタイルで書くことを想定していない
    - これは現状のTruth哲学だが、AssertJのようなChainスタイルも普及してきたので、どちらが便利かはわからない

多少の差異はあれど、AssertJとTruthはとても似ているライブラリです。どちらか一方を使っているなら、乗り換えるメリットはおそらくないだろうという旨の内容です。

## まとめ

- Truthを使うと失敗メッセージがわかりやすくなる、便利!
- AssertJを使っているなら乗り換えるメリットはないかも
