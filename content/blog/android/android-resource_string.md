+++
date = "2015-02-22T04:10:23Z"
title = "Android: strings.xmlのまとめ"
tags = ["android", "java"]
blogimport = true
type = "post"
+++

res/values/strings.xmlは, 文字列関連のリソースを管理するファイルです. 文字列をプログラム外で定義することで, 多国語の対応, デバッグブルドの時にサーバのURLを変更する等が, コードをいじらずに容易に行えます.

また, 意外といろいろな機能があったので, 紹介したいと思います.


## 基本的な使い方

```xml
<string name="app_name">Application</string>
```

のように記述して使います. アプリケーション側では, 下のように取得します.

```java
getString(R.string.app_name);

or

context.getString(R.string.app_name);
```

特に, 説明はいらないと思います.


## arrayの定義

strings.xmlでは単純なkey-valueだけでなく, arrayも定義することができます.

string-array要素で定義してあげます.

```xml
<string-array name="sports">
    <item>マラソン</item>
    <item>野球</item>
    <item>サッカー</item>
    <item>卓球</item>
</string-array>
```

アプリケーション側では下のように取得します.

```java
getResources().getStringArray(R.array.sports)

or

context.getResources().getStringArray(R.array.sports)
```

Contextから直接取得することが出来ないので, 一旦Resourcesを取得し, そこからarrayを取り出します.


## 値展開

strings.xmlでは, 文字列展開することが出来ます. printf formatのように使います.

```xml
<!-- %1: 引数1, $d: 数字 -->
<string name="hoge">Hello %1$d</string>
<!-- %1: 引数1, %2: 引数2, $s: 文字列 -->
<string name="hogestr">%1$s %2$d %1$s</string>
```

アプリケーション側では, 下のように指定します.

```java
getString(R.string.hoge, 100)); // Hello 100
getString(R.string.hogestr, "value", 100)); // value 100 value
```

`%1$s` は「第1引数の文字列」をここに展開しろ. `%2$d`は「第2引数の数字」をここに展開しろ. という意味になります


## plurals

あまり使ったことはないですが, pluralsという要素があります.

これは, quantity stringsと呼ばれ, 数字(量)の大きさに応じて, 文字列を変化させることが出来ます. switch分岐が出来るイメージです.

下のように記述します.

```xml
<plurals name="number">
    <item quantity="one">one</item>
    <item quantity="other">other %1$d</item>
</plurals>
```

quantityには, zero, one, two, few, many, otherが指定できます.

アプリケーション側の記述です.

```xml
getResources().getQuantityString(R.plurals.number, 1, 1)); // one
getResources().getQuantityString(R.plurals.number, 10, 100)); // other 100
```

第二引数にquantityに与える数字を指定します. 1ならone, 0ならzeroがそれぞれ対応します. 複数形の場合に文字列を変えたいときなどに有効です.


## まとめ

strings.xmlには, 意外といろいろな機能がありました. switch的な処理が書けるpluralsは頭の片隅に入れておくと良さそう.


## 参考

- [strings.xml | Android Developers](https://developer.android.com/samples/BasicNetworking/res/values/strings.html)
- [Android　複雑な文字列を xml で定義する](http://y-anz-m.blogspot.jp/2011/03/android-xml.html)
