+++
date = "2018-12-10T00:00:00Z"
title = "Android: namespacedRClassフラグでRファイルをほげほげ"
tags = ["android", "agp"]
blogimport = true
type = "post"
draft = true
+++

正確な時期は知らないのですが、AGP3.3のどこかのタイミングで`namespacedRClass`フラグが使えるようになったのでその紹介です。
本記事では`3.4.0-alpha07`で試しました。

Rファイルの1つの課題として、依存関係にあるRファイルがどんどんマージされていくため、ライブラリモジュールのRファイルがfatになる問題があります。
-> 本当にそれ問題だった?

それを解決するために`namespacedRClass`が導入されました。使い方は簡単で次の記述を`gradle.properties`に追加するだけです。

```txt
android.namespacedRClass=true
```

例えばappcompatに依存しているライブラリモジュールのRファイルが次のようになります。

`namespacedRClass=false`のとき

```java
public final class R {
    private R() {}

    public static final class anim {
        private anim() {}

        public static final int abc_fade_in = 0x7f010000;
        public static final int abc_fade_out = 0x7f010001;
        public static final int abc_grow_fade_in_from_bottom = 0x7f010002;
        public static final int abc_popup_enter = 0x7f010003;
    ...
    ...
```

`namespacedRClass=true`のとき

```java
// base3ライブラリモジュールのRファイル
public final class R {
    private R() {}

    public static final class color {
        private color() {}

        public static final int red3 = 0x7f04004b;
    }
    public static final class id {
        private id() {}

        public static final int title = 0x7f0700b1;
    }
    public static final class layout {
        private layout() {}

        public static final int base3 = 0x7f09001d;
    }
    public static final class string {
        private string() {}

        public static final int base_string3 = 0x7f0b002a;
    }
}
```

falseのときは依存関係にあるRファイルの内容が含まれていることが分かります。
trueのときは、そのモジュールで定義したリソースの参照しか含まれていません。

## まとめ

- todo
