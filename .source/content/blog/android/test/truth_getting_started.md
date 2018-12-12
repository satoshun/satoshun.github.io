+++
date = "Wed Dec 12 00:21:12 UTC 2018"
title = "Truthのメリット、特徴について"
tags = ["android", "test", "truth"]
blogimport = true
type = "post"
draft = false
+++

[Truth](http://google.github.io/truth/)はGoogleが開発をしているテストアサーションライブラリです。

従来のJUnitスタイルのアサーションに比べ、大きく2つの利点があります。

- readableにアサーションが書ける
- デフォルトの失敗メッセージがわかりやすい

それぞれについて説明していきます。

## readableにアサーションが書ける

ドキュメントのbenefitにあるサンプルを取り上げ説明します。
http://google.github.io/truth/benefits

まず従来のJUnitスタイルを使って書きます。

```java
Optional<String> middleName = user.getMiddleName();
assertFalse(middleName.isPresent());
```

assertFalse、isPresentを使っており、否定のアサーションなので直感的でなく理解するのに少し時間がかかります。（個人差はあります）

これがTruthを使うと次のようになります。

```java
Optional<String> middleName = user.getMiddleName();
assertThat(middleName).isAbsent();
```

assertThatはTruthに定義されているメソッドです。`middleName`がabsent、値が存在しないことをテストしていることが、JUnitスタイルより強く伝わります。

上記の`isAbsent`はOptionalのために用意されたアサーションメソッドです。assertThatに渡した引数に適したアサーションを使うことが出来ます。
例えばIterableには、`containsAnyIn`や`isEmpty`などが用意されています。型ごとに一般的なテストで行うであろうアサーションが用意されており、readableにconciseに書くことが出来ます。

## 失敗メッセージがわかりやすい

こちらもbenefitにあるサンプルを取り上げます。

まずは従来にJUnitスタイルから。

```java
assertTrue(googleColors.contains(PINK));
```

この場合、失敗メッセージは特にありません。「trueを表明してる部分にfalseが来た」程度のものしかなく、原因特定するのが大変です。
失敗メッセージをカスタムすることは出来ますが、すべてのアサーションに対して定義するのは骨が折れます。

次にTruthスタイルです。

```java
assertThat(googleColors).contains(PINK);
```

`<[BLUE, RED, YELLOW, BLUE, GREEN, RED]> should have contained <PINK>` のようなメッセージが出ます。インスタンス情報や、こうなるべきというメッセージが含まれており原因特定がしやすくなっています。デフォルトの段階でかなり見やすい、わかりやすい失敗メッセージを出力してくれます。

以上がTruthのメリットになります。

## 補足

### Truth-Androidライブラリ

JetPackにTruth + Android用のライブラリが追加されました。これを使うことでBundle、IntentなどのAndroid固有のクラスのテストが書きやすくなります。
例えば、Intentには以下のアサーションメソッドを使うことが出来ます。

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

Intentの中身を確認する便利メソッドが定義されています。Truth-Androidを使うことで、よりAndroid環境でテストが書きやすくなることが期待出来ます。

### AssertJとの比較

これも公式ドキュメントにまとめてあります。http://google.github.io/truth/comparison

現状の主だった差分は次のようになっています。

- Truhtはまだ1.0.0になっていないのでAPIが変わる可能性がある
- TruthはChainスタイルで書くことを想定していない
    - これは現状のTruth哲学だが、AssertJのようなChainスタイルも普及してきたので、どちらが便利かはわからない

多少の差異はあれど、AssertJとTruthはとても似ているライブラリです。どちらか一方を使っているなら、乗り換えるメリットはおそらくないだろうという旨の内容です。

## まとめ

- Truthを使うと失敗メッセージがわかりやすくなる、便利!
- AssertJを使っているなら乗り換えるメリットはないかも
