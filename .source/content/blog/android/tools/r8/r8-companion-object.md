+++
date = "Sun Jan 20 04:23:43 UTC 2019"
title = "R8/Proguard: KotlinのCompanion objectとobjectについて"
tags = ["android", "r8", "proguard", "kotlin"]
blogimport = true
type = "post"
+++

コードの最適化の話です。この記事ではKotlinのCompanion objectとobjectについて紹介します。

この記事は、[R8 Optimization: Staticization](https://jakewharton.com/r8-optimization-staticization/)にとても影響を受けています。

## Companion object

例えば、次のコードがあるとします。

```kotlin
class CompanionTest {
    companion object {
        fun show(i: Int) {
            ...
        }
    }
}
```

これを最適化なしで変換すると次のようになります。

```java
public final class CompanionTest {
    public static final Companion Companion = new Companion();

    public static final class Companion {
        private Companion() {
        }

        public final void show(int i) {
            ...
        }
    }
}
```

Companionインスタンスが生成されているのが分かります。ただ、このCompanion objectはインスタンス生成する必要がありません。なぜなら、インターフェースの実装などをしていないからです。

そこでR8による最適化を行うと次のようになります。

```java
public abstract class CompanionTest {
    public static final void show(int i) {
        ...
    }
}
```

無駄な内部クラス（enclosing class）が消えて、staticメソッドに変換されているのが分かります。このshowメソッドはわざわざインスタンスメソッドにする必要がないため、このような最適化が行われます。

## object

objectも同様です。

```kotlin
object ObjectTest {
    fun show(i: Int) {
        ...
    }
}
```

最適化なしだと次のようになります。

```java
public final class ObjectTest {
    public static final ObjectTest INSTANCE = new ObjectTest();

    public final void show(int i) {
        ...
    }
}
```

INSTANCEフィールドが生成されていることが分かります。

次にR8による最適化を行います。

```java
public abstract class ObjectTest {
    public static final void show(int i) {
        ...
    }
}
```

Companion objectと同様に冗長なインスタンス生成が消えています。
Proguardの場合、インスタンス生成は消えないので、R8の一歩進んだ最適化といえます。（もしかしたらProguardの設定次第でインスタンス生成をしないように出来るかもしれないです。）
