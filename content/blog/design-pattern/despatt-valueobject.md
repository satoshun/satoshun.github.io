+++
date = "2015-02-22T04:53:36Z"
title = "デザインパターン: Value Object"
tags = ["design_pattern", "code"]
type = "post"
+++

Value Object(値オブジェクト)は, メンバとメソッドを持ちクラスとしての特徴を持つが, immutableであり, identityキー(RDSでいうところのautoincremental id)を持たないオブジェクトのことです. 異なるオブジェクト同士であっても値が等しければ等しいとみなされます.
immutableなので, primitive(int, floatなど)な値と同等に扱うことが出来ます.


## immutableであるメリット

immutableであるメリットとしては

- 状態を持たないので, 呼び出し順序を考慮しなくて良い
- 値が書き換わらないため, thread safeである
- プログラムの途中でオブジェクトの特徴が変わることがないため, プログラムをreadableに保つことが出来ることが多い

1, 2つ目は理解できると思うので, 「プログラムの途中でオブジェクトの特徴が変わることがないため, プログラムをreadableに保つことが出来る」をJavaのコードをあげて説明します.

```java
public static Calendar getYesterday() {
    Calendar rightnow = Calendar.getInstance(); // ここの時点ではrightnowは, 今の時間を示している
    rightnow.add(Calendar.DATE, -1); // ここの時点ではrightnowは, 昨日を示している. 変数名rightnowは相応しくない
    return rightnow;
}
```

上記コードは, 変数rightnowを定義した段階では, 相応しい変数名なのですが, `rightnow.add(Calendar.DATE, -1)`を実行した段階で, 相応しくない変数名に変わります. これは, Calendarインスタンスが, mutableなためです. このようなコードは混乱を招きます.
オブジェクトは初期化したら, セッターなどで値は変更しないほうが, コードをreadableに保つことができ, バグを防ぐことが出来ます.


## indentityキーを持たないメリット

Value Objectは, 名前の通り, Value(値)に注目しているパターンです. 量などの属性値が重要であり, identityキーには注目していないため, 除去しています. 逆に, identityキーに注目しているパターンのことを, エンティティ(Entity)と言います. identityキーが重要かどうかで, パターンが変わってくるので, 慣れが必要そうです.


# 参考

- https://www.ogis-ri.co.jp/otc/hiroba/technical/DDDEssence/chap2.html
