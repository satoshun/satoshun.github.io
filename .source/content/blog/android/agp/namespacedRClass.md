+++
date = "2018-12-09T00:00:00Z"
lastmod = "Wed Dec 12 13:56:04 UTC 2018"
title = "Android: namespacedRClassフラグでRクラスを小さく保つ"
tags = ["android", "agp"]
blogimport = true
type = "post"
draft = false
+++

Android Gradle Plugin（以下AGP）3.3のalphaのどこかのタイミングで`namespacedRClass`フラグが新しく追加されたので紹介します。
本記事では`3.4.0-alpha07`で試しました。

まず現状の問題点として、ライブラリモジュールのRクラスのサイズが大きくなる課題があります。それは、ライブラリのRクラスは依存関係にあるRクラスがどんどんマージされていくためです。
それを解決するために`namespacedRClass`が追加されました。使い方は簡単で、次の記述を`gradle.properties`に追加するだけです。

```
android.namespacedRClass=true
```

では、これからこのフラグがtrueとfalseでどのようにRクラスの内容が変わるか見ていきます。
例として、appcompatに依存しているライブラリモジュールを用意します。

まずは、`namespacedRClass=false`の時のRクラスです。

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

依存関係にあるappcompatのRクラスの内容が含まれていることが分かります。

では次に、`namespacedRClass=true`の時です。

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

このモジュールで定義したリソースの内容しか含まれていないことが分かります。appcompatのRクラスは含まれていません。
ライブラリモジュールのRクラスのサイズがかなり小さくすることが出来ました!!

今後、中、大規模なAndroid開発はマルチモジュールに強く依存することになると思うので、このオプションをつけることで、デバッグ時のapkサイズを抑えることが期待できます。（リリース時はR8/Proguardを使うと思うので特に影響はない）

## 注意点

ただし、注意点として、依存関係にあるライブラリのRクラスのマージが行われないため、appcompatなどのRクラスにアクセスしたいときは、明示的にRクラスをimportをする必要があります。

```kotlin
import androidx.appcompat.R as AppCompatR
```

今までだとライブラリモジュールのRクラスからすべてのリソースにアクセスできたのですが、それができなくなります。なので、このオプションをtrueしたときは、ライブラリモジュールでRクラスのimportパスを変更する必要があります。

## まとめ

- マルチモジュール時代に適した機能だと思う
    - ライブラリサブモジュールのRクラスのサイズ大きくなる問題を解決できる
    - デバッグ時のapkサイズをやや小さく出来る
