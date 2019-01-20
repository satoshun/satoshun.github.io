+++
date = "Sun Jan 20 02:16:43 UTC 2019"
title = "R8/Proguard: Class Mergingについて"
tags = ["android", "r8", "proguard"]
blogimport = true
type = "post"
+++

コードの最適化の話です。この記事ではClass Mergingについて紹介します。

## Class Merging?

その名の通り、Classをマージする最適化です。最終的なクラス数減らすことが期待出来ます。
Class Mergingには縦方向（Vertical）と、横方向（Horizontal）があります。

まずは縦方向のClass Mergingについて説明します。

### 縦方向?

縦方向とはスーパータイプの実装が1つだったときに、そのスーパータイプと実装であるサブタイプを1つにまとめる最適化です。

例えば、次の実装は最適化によって1つにまとめられます。

```kotlin
interface IVertical {
    fun show(i: Int)
}

class Vertical(
    private val a: Int
) : IVertical {
    override fun show(i: Int) {
        println("start called $i $a")
    }
}
```

--> Proguard/R8による最適化後 -->

```java
public final class Vertical {
    ...

    public final void show(int i) {
        ...
    }
}
```

`IVertical`インターフェースが見事に消されていることが分かります。

また、インターフェースではなくabstractクラスの場合はR8の場合のみ上手くマージされました。

- [R8: Vertical Merger](https://r8.googlesource.com/r8/+/master/src/main/java/com/android/tools/r8/shaking/VerticalClassMerger.java)
- [Proguard: Vertical Merger](https://github.com/facebook/proguard/blob/master/src/proguard/optimize/peephole/VerticalClassMerger.java)

次に横方向のマージを紹介します。

### 横方向?

Staticメソッドのみを持つクラスを1つにまとめるなどの最適化を行います。

例えばKotlinの複数のファイルで定義されたトップレベル関数を1つのクラスにまとめてくれます。

```kotlin
// ShowExt.kt
fun Int.show1(i: Int) {
    ...
}

// ShowExt2.kt
fun Int.show2(i: Int) {
    ...
}

--> Proguard/R8による最適化後 -->

```java
public class ShowExt2Kt {
    public static final void show1(int i) {
        ...
    }
}

    public static final void show2(int i) {
        ...
    }
}
```

1つのクラスにまとめられていることが分かります。

また、実際のAndroidプロジェクトで確認したところ、確かに拡張関数が他のクラスにマージされていました。

```
// Fuga.kt
fun Int.show1() {
    ...
}

// mapping.txtの内容
android.support.constraint.solver.widgets.Analyzer -> a.b.b.a.a.a:
    ...
    void com.github.satoshun.example.FugaKt.show1(int) -> d
```

なぜこのクラスに移動したかは分からないので、今後の宿題とします。申し訳ありません😫

## まとめ

- 横方向のマージについては内容に自信がないので、もし間違っていたり、補足があれば教えて頂けるととても嬉しいです🙏
