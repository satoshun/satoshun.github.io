+++
date = "2018-11-07"
title = "KotlinのNon-Null型にnullを代入する方法"
tags = ["kotlin"]
blogimport = true
type = "post"
+++

Kotlinではnullを扱いやすくするためにNullable、Non-Nullを型で制限することが出来ます。
KotlinのNon-Null型に対してnullを代入しようとすると、代入するタイミングで例外を吐きます。
JavaとKotlinを一緒に使っていると、この例外に遭遇することがあると思います。

具体的には以下の挙動をします。

```java
public class Hoge {
    // nullを返すメソッド
    static String getName() {
        return null;
    }
}
```

```kotlin
val d: String = Hoge.getName() // ここで例外が投げられる
println(d.length) // これは実行されない
```

このコードは、`val d: String = Hoge.getName()`で`IllegalStateException`を投げます。
Non-Null型にnullを代入しようとしているからです。

次に、代入するタイミングで例外を投げなくする方法を紹介します。

具体的には以下のコードで達成できます。

```kotlin
fun <T> castNull(): T = null as T


val d: String = castNull() // ここでは例外が投げられない
println(d.length) // ここで例外が投げられる
```

なぜこのような挙動になるのかを説明します。まず `castNull()`メソッドに定義されたジェネリック型Tは`Any?`をupperに持ちます。
Kotlinはジェネリックを使い、かつNullable型をupperに持つとき、nullかどうかのチェックをしません。これはバイトコードを見れば分かります。

```text
GETSTATIC sample/SampleTestsJVMKt.a : Ljava/lang/String;
CHECKCAST java/lang/Object
```

`CHECKCAST`しかしておらず、nullかどうかのチェックをしていません。
なので、`val d: String = castNull()`ではnullチェックがされないため例外が投げられず、実際にメソッドをコールする`println(d.length)`のタイミングで例外が投げられます。

Non-Null型である`String`にnullを代入することが出来ました。

次に、`fun <T: Any> castNull(): T = null as T`と、Non-Null型をupperにした場合のバイトコードは次になります。

```text
GETSTATIC sample/SampleTestsJVMKt.a : Ljava/lang/String;
DUP
IFNONNULL L1
NEW kotlin/TypeCastException
DUP
LDC "null cannot be cast to non-null type T"
INVOKESPECIAL kotlin/TypeCastException.<init> (Ljava/lang/String;)V
```

こちらのバイトコードには、nullチェックが入っていることが分かります。nullチェックをしているため、`val d: String = castNull()`のタイミングで例外が投げられます。

Nullableな型をupperに持つ、持たないでnullチェックをするかどうかが決まる、という話でした。

この挙動は https://youtrack.jetbrains.com/issue/KT-8135 ISSUEにバグとして報告されているので、今後異なるバイトコードが生成される可能性があります。
もし、この挙動に依存するようなコードを書いている場合は注意が必要です。

実世界では、mockito-kotlinがこの挙動に依存しているので、より興味がある方はぜひ読んでください😃 [ここのあたりで使っています](https://github.com/nhaarman/mockito-kotlin/blob/2.x/mockito-kotlin/src/main/kotlin/com/nhaarman/mockitokotlin2/internal/CreateInstance.kt#L46)

## 参考

- https://github.com/nhaarman/mockito-kotlin
- https://youtrack.jetbrains.com/issue/KT-8135
