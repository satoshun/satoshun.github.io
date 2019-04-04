+++
date = "Sun Mar 31 10:31:21 UTC 2019"
title = "TODO"
tags = ["kotlin", "coroutine"]
blogimport = true
type = "post"
draft = true
+++

Kotlin Coroutineのsuspend関数の生成コードを見て、楽しもうという記事になります！！

最初はシンプルに、少しずつ複雑なコードにしていきます。
また、生成コードは、Android Studio(Intellij)の「Show Kotlin Byte -> Decompile」で生成しています。

## 1. suspend関数の中で、他のsuspend関数をコールしない場合

```kotlin
suspend fun simple1(user: User): Int {
  println("hello")
  return 100
}
```

この関数は、suspendである必要はないのですが、そのような関数につけた場合、次のようなコードが生成されます。

```java
public final class Simple1Kt {
   @Nullable
   public static final Object simple1(@NotNull User user, @NotNull Continuation var1) {
      String var2 = "hello";
      System.out.println(var2);
      return Boxing.boxInt(100);
   }
}
```

2点、気になるコードがあります。

1. simple1関数の引数に`Continuation`が追加された
2. 返り値がObjectになった

suspendをつけると、入り口と出口、インプットとアウトプットの部分が変化します。

ここでは、変化するんやなぁ〜くらいに思っておいてください。

次に、もう少し複雑な例を見ていきます。

## 2. suspend関数の中で、他のsuspend関数をコールする

```kotlin
suspend fun simple2(user: User): Int {
  delay(1000)
  println("hello")

  return 100
}
```

delayはsuspend関数です。このメソッドは次のようなコードを生成します。`Simple2Kt$simple2$1`クラスの生成がうまくいかなかったので、少し修正しました。

```kotlin
public final class Simple2Kt {
   @Nullable
   public static final Object simple2(@NotNull User user, @NotNull Continuation var1) {
      Object $continuation;
      label28: {
         if (var1 instanceof Simple2Kt$simple2$1) {
            $continuation = (Simple2Kt$simple2$1)var1;
            if ((((Simple2Kt$simple2$1)$continuation).label & Integer.MIN_VALUE) != 0) {
               ((Simple2Kt$simple2$1)$continuation).label -= Integer.MIN_VALUE;
               break label28;
            }
         }

         $continuation = new Simple2Kt$simple2$1(var1) {
            // $FF: synthetic field
            Object result;
            int label;
            Object L$0;

            @Nullable
            public final Object invokeSuspend(@NotNull Object result) {
               this.result = result;
               this.label |= Integer.MIN_VALUE;
               return Simple2Kt.simple2((User)null, this);
            }
         };
      }

      Object var3 = ((Simple2Kt$simple2$1)$continuation).result;
      Object var5 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
      switch(((Simple2Kt$simple2$1)$continuation).label) {
      case 0:
         if (var3 instanceof Failure) {
            throw ((Failure)var3).exception;
         }

         ((Simple2Kt$simple2$1)$continuation).L$0 = user;
         ((Simple2Kt$simple2$1)$continuation).label = 1;
         if (DelayKt.delay(1000L, (Continuation)$continuation) == var5) {
            return var5;
         }
         break;
      case 1:
         user = (User)((Simple2Kt$simple2$1)$continuation).L$0;
         if (var3 instanceof Failure) {
            throw ((Failure)var3).exception;
         }
         break;
      default:
         throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
      }

      String var2 = "hello";
      System.out.println(var2);
      return Boxing.boxInt(100);
   }
}

final class Simple2Kt$simple2$1 extends ContinuationImpl {
   ...

   // $FF: synthetic field
   Object result;
   int label;
   Object L$0;

   @Nullable
   public final Object invokeSuspend(@NotNull Object result) {
      this.result = result;
      this.label |= Integer.MIN_VALUE;
      return Simple2Kt.simple2((User)null, this);
   }
}
```
