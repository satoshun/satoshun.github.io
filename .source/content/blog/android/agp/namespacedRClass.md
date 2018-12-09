+++
date = "2018-12-09T00:00:00Z"
title = "Android: namespacedRClassフラグでRクラスを小さく保つ"
tags = ["android", "agp"]
blogimport = true
type = "post"
draft = false
+++

正確な時期は知らないのですが、AGP3.3のどこかのタイミングで`namespacedRClass`フラグが使えるようになったのでその紹介です。
本記事では`3.4.0-alpha07`で試しました。

Rクラスは依存関係にあるRクラスがどんどんマージされていくので、ライブラリモジュールのRクラスのサイズが大きくなる課題があります。

それを解決するために`namespacedRClass`が導入されました。使い方は簡単で、次の記述を`gradle.properties`に追加するだけです。

```txt
android.namespacedRClass=true
```

例えば、appcompatなどに依存しているライブラリモジュールのRファイルが次のようになります。

まずは、`namespacedRClass=false`の時。

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

このモジュールでは定義してない、依存関係にあるappcompatなどのRクラスの内容が含まれていることが分かります。

次に、`namespacedRClass=true`の時です。

```java
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

trueのときは、そのモジュールで定義したリソースへの参照しか含まれていなことが分かります。appcompatのRクラスの内容は含まれていません。
ライブラリモジュールのRクラスのサイズがかなり小さくなりました!!

このフラグをtrueにし、ライブラリモジュールからappcompatなどのRクラスにアクセスしたいときは、明示的にRクラスをimportをする必要があります。

```kotlin
import androidx.appcompat.R as AppCompatR
```

今までだとライブラリモジュールのRからすべてのリソースにアクセスできたのですが、それができなくなります。
今後、中、大規模なAndroid開発はマルチモジュールに強く依存することになると思うので、このオプションをつけることで、デバッグ時のapkサイズを抑えることが期待できます。（リリース時はR8/Proguardを使うと思うので特に影響はない）

## まとめ

- マルチモジュール時代に適した機能だと思う
    - ライブラリサブモジュールのRクラスのサイズ大きくなる問題を解決できる
    - デバッグ時のapkサイズをやや小さく出来る
