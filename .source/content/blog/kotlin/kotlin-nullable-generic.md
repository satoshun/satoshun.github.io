+++
date = "2018-11-05"
title = "KotlinのNonNull型にnullを代入する方法"
tags = ["kotlin"]
blogimport = true
type = "post"
+++

Kotlinではnullを扱いやすくするためにnullable、nonnullを型で制限することが出来ます。
Kotlinのnonnull型に対してnullを代入しようとすると、代入するタイミングで例外を吐きます。

具体的には以下の挙動をします。

```java
public class Hoge {
    static String getName() {
        return null;
    }
}
```

```kotlin
val d: String = Hoge.getName() // ここで例外が投げられる
println(d.length) // ここには到達しない。この式は評価されない
```

このコードは、`val d: String = Hoge.getName()`で`IllegalStateException`を投げます。
nonnull型にnullを代入しようとしているためです。

例外を投げるタイミングを代入するタイミングではなくて、評価するタイミングにする方法があります。

具体的には以下のコードで達成できます。

```kotlin
fun <T> castNull(): T = null as T


val d: String = castNull() // ここでは例外が投げられない
println(d.length) // ここで例外が投げられる
```

なぜこのような挙動になるのかを説明します。まず `castNull()`メソッドに定義されたジェネリック型Tは`Any?`をupperに持ちます。
Kotlinは、nullable型をupperに持つときnullかどうかのチェックをしません。これはバイトコードを見れば分かります。

```text
GETSTATIC sample/SampleTestsJVMKt.a : Ljava/lang/String;
CHECKCAST java/lang/Object
```

`CHECKCAST`しかしておらず、nullかどうかのチェックをしていません。

`fun <T: Any> castNull(): T = null as T`と、nonnull型をupperにした場合のバイトコードは次になります。

```text
GETSTATIC sample/SampleTestsJVMKt.a : Ljava/lang/String;
DUP
IFNONNULL L1
NEW kotlin/TypeCastException
DUP
LDC "null cannot be cast to non-null type T"
INVOKESPECIAL kotlin/TypeCastException.<init> (Ljava/lang/String;)V
```

こちらには、nullチェックが入っていることが分かります。
nullableな型をupperに持つ、持たないでnullチェックをするかどうかが決まります。

この挙動は https://youtrack.jetbrains.com/issue/KT-8135 ISSUEにバグとして報告されているので、今後異なるバイトコードが生成される可能性があるので、
この挙動に依存するようなコードを書いている場合は注意が必要です。

## 参考

- https://github.com/nhaarman/mockito-kotlin
