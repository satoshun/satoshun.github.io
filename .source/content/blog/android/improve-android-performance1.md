+++
date = "2015-08-11T00:00:00Z"
title = "Improving Android: 列挙にはenumではなくIntDef, StringDef annotationを使う"
tags = ["android"]
blogimport = true
type = "post"
+++

enumの典型的な使い方として, 月(month)や, ネットワークプロコトルなどの, 特定の種類を列挙するために使用されます.

列挙型としてのenumは, 以下のように書くことが出来ます.

```java
enum Month {
    JANUARY, FEBRUARY, MARCH, APRIL, MAY, JUNE,
    JULY, AUGUST, SEPTEMBER, OCTOBER, NOVEMBER, DECEMBER
}

/** 指定した月が何日まであるかを返す */
int getDate(Month month) {
    ////
}
```

また, enumではなく定数を使うとしたら以下のように書くことが出来ます.

```java
static final int JANUARY = 1;
static final int FEBRUARY = 2;
static final int MARCH = 3;
static final int APRIL = 4;
...
static final int NOVEMBER = 11;
static final int DECEMBER = 12;

/** 指定した月が何日まであるかを返す */
int getDate(int month) {
    ////
}
```

定数を使うバージョンだと, `getDate(int)`のため, 予期せぬ値が入ってきてしまう可能性があります. enumの場合は, `getDate(Month)`のため, type-safeを提供してくれます. これは, 大きなメリットです.

しかし, enumは, 定数を使うバージョンと比較すると, apkサイズが大きくなってしまうデメリットがあります. Androidではメモリリソースはとても貴重なため, enumを使うのは極力避けたほうが良いです.

Androidでは, IntDef(StringDef) annotationを使うことで, type-safeに定数を使うことが出来ます.

上記のMonthのコードをIntDef annotationを使い, 書き換えてみます.

```java
// Monthに代入することが出来る定数を宣言
@IntDef({JANUARY, FEBRUARY, MARCH, APRIL, ...})
@Retention(RetentionPolicy.SOURCE)
public @interface Month {};
static final int JANUARY = 1;
static final int FEBRUARY = 2;
static final int MARCH = 3;
static final int APRIL = 4;
...

int getDate(@Month int month) {
    ////
}
```

Month annotationをIntDefで定義し, `getDate(@Month int)`と書くことでtype-safeにgetDateメソッドを宣言することが出来ます.
この書き方は, 「enumのtype-safe」と「定数のパフォーマンス」を満たしています.


## まとめ

- 定数はtype-safeでないため, IntDef(StringDef) annotationを使い, type-safeを提供する
- enumはパフォーマンスに影響を与える可能性があるため, IntDef(StringDef) annotationを使うことを検討する


## 参考

- [Support Annotations](http://tools.android.com/tech-docs/support-annotations)
