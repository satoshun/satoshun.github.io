+++
date = "Sun Jan 20 06:56:16 UTC 2019"
title = "R8/Proguard: Kotlinのlambda最適化について"
tags = ["android", "r8", "proguard", "kotlin"]
blogimport = true
type = "post"
+++

コードの最適化の話です。この記事ではKotlinのlambda式の最適化について紹介します。

## Kotlin lambda

Kotlinでは関数型がファーストクラスです。とても便利なのですが、ラムダを使うたびに内部的にはクラスを1つ定義するため、クラス数、メソッド数がどんどん増えていきます。

例えば、次のコードを最適化なしでコンパイルしてみます。

```kotlin
fun main() {
    lambdaTest1 { println("Kotlin lambda1") }
    lambdaTest1 { println("Kotlin lambda2") }
    lambdaTest1 { println("Kotlin lambda3") }
    ...
}

private fun lambdaTest1(body: () -> Unit) {
    ...
    body()
    ...
}
```

```java
// コンパイル後
public static final void main() {
    LambdaTestKt.lambdaTest1((Function0) LambdaTestKt$main$1.INSTANCE);
    LambdaTestKt.lambdaTest1((Function0) LambdaTestKt$main$2.INSTANCE);
    LambdaTestKt.lambdaTest1((Function0) LambdaTestKt$main$3.INSTANCE);
    ...
}

final class LambdaTestKt$main$1 extends Lambda implements Function0<Unit> {
    public static final LambdaTestKt$main$1 INSTANCE = new LambdaTestKt$main$1();

    LambdaTestKt$main$1() {
        super(0);
    }

    public final void invoke() {
        System.out.println("Kotlin lambda1");
    }
}

final class LambdaTestKt$main$2 extends Lambda implements Function0<Unit> {
    ...
}
final class LambdaTestKt$main$3 extends Lambda implements Function0<Unit> {
    ...
}
```

`LambdaTestKt$main$1`、`$2`、`$3`が生成されていることが分かります。ラムダを使う箇所を増やすと$2、$3...とクラスが増えていきます。

この問題に対し、R8ではLambdaGroupというテクニックを使い最適化をします。

R8を使って、上記のコードをコンパイルすると次のようになります。

```java
public static final void main() {
    LambdaTestKt.lambdaTest1(-$$LambdaGroup$ks$BpS7w8o_KOkdSUy0gHGt84S7irI.INSTANCE$0);
    LambdaTestKt.lambdaTest1(-$$LambdaGroup$ks$BpS7w8o_KOkdSUy0gHGt84S7irI.INSTANCE$1);
    LambdaTestKt.lambdaTest1(-$$LambdaGroup$ks$BpS7w8o_KOkdSUy0gHGt84S7irI.INSTANCE$2);
    ...
}

public final class -$$LambdaGroup$ks$BpS7w8o_KOkdSUy0gHGt84S7irI extends Lambda implements Function0 {
    public static final -$$LambdaGroup$ks$BpS7w8o_KOkdSUy0gHGt84S7irI INSTANCE$0 = new -$$LambdaGroup$ks$BpS7w8o_KOkdSUy0gHGt84S7irI(0);
    public static final -$$LambdaGroup$ks$BpS7w8o_KOkdSUy0gHGt84S7irI INSTANCE$1 = new -$$LambdaGroup$ks$BpS7w8o_KOkdSUy0gHGt84S7irI(1);
    public static final -$$LambdaGroup$ks$BpS7w8o_KOkdSUy0gHGt84S7irI INSTANCE$2 = new -$$LambdaGroup$ks$BpS7w8o_KOkdSUy0gHGt84S7irI(2);
    public final /* synthetic */ int $id$;

    public -$$LambdaGroup$ks$BpS7w8o_KOkdSUy0gHGt84S7irI(int i) {
        this.$id$ = i;
        super(0);
    }

    public final Object invoke() {
        int i = this.$id$;
        if (i == 0) {
            System.out.println("Kotlin lambda1");
            return Unit.INSTANCE;
        } else if (i == 1) {
            System.out.println("Kotlin lambda2");
            return Unit.INSTANCE;
        } else if (i == 2) {
            System.out.println("Kotlin lambda3");
            return Unit.INSTANCE;
        } else {
            throw null;
        }
    }
}
```

最適化しない場合では、それぞれのラムダに対して、専用のクラスが1つずつ定義されていました。最適化された後では1つのクラス、LambadGroupにまとまっていることが分かります。
このクラスでは、それぞれのラムダインスタンスにidを持たせることで、どのラムダかを分類することが出来ます。

この最適化により、クラス数、メソッド数を抑えることが出来ます。
