+++
date = "2017-12-17"
title = "Kotlin: Companion object + @JvmStaticは冗長なメソッドが作られる"
tags = ["kotlin"]
blogimport = true
type = "post"
draft = false
+++

下に説明のための簡単なKotlinのコードを載せます。

```kotlin
class Fuga {
  companion object {
    fun fuga1() {
      println("fuga1")
    }

    @JvmStatic
    fun fuga2() {
      println("fuga2")
    }
  }
}
```

これをJavaに変換すると、

```java
public final class Fuga {
   public static final Fuga.Companion Companion = new Fuga.Companion((DefaultConstructorMarker)null);

   @JvmStatic
   public static final void fuga2() {
      Companion.fuga2();
   }

   @Metadata(
      mv = {1, 1, 9},
      bv = {1, 0, 2},
      k = 1,
      d1 = {"\u0000\u0014\n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0002\b\u0002\n\u0002\u0010\u0002\n\u0002\b\u0002\b\u0086\u0003\u0018\u00002\u00020\u0001B\u0007\b\u0002¢\u0006\u0002\u0010\u0002J\u0006\u0010\u0003\u001a\u00020\u0004J\b\u0010\u0005\u001a\u00020\u0004H\u0007¨\u0006\u0006"},
      d2 = {"Lcom/github/satoshun/reactivex/cache/Fuga$Companion;", "", "()V", "fuga1", "", "fuga2", "production sources for module rx-kotlin-cache_main"}
   )
   public static final class Companion {
      public final void fuga1() {
         String var1 = "fuga1";
         System.out.println(var1);
      }

      @JvmStatic
      public final void fuga2() {
         String var1 = "fuga2";
         System.out.println(var1);
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

メソッドfuga2が2つあることが分かります。1つは実際のfuga2へのProxyになっており、JvmStaticをつけると1つ冗長なメソッドが作成されることが分かります。


## まとめ

- JvmStaticをつけると冗長なメソッドが作られることがある
