+++
date = "Mon Mar  4 23:54:20 UTC 2019"
title = "Dagger2: ProvidesはKotlin extension methodと一緒に使うことが出来る"
tags = ["dagger", "android"]
blogimport = true
type = "post"
draft = true
+++

Dagger2のちょっとしたtipsです。

次の2メソッドは同じ振る舞いをします。

```kotlin
@Module
class MainActivityModule {
  // 普通の書き方
  @Provides fun provideMainContractView(activity: MainActivity): MainContract.View {
    return activity
  }

  // 拡張関数を使った書き方
  @Provides fun MainActivity.provideMainContractView(): MainContract.View {
    return this
  }
}
```

なぜなら、拡張関数はコンパイルされると次のように解釈されるためです。

```java
...
   @Provides
   @NotNull
   public final MainContract.View provideMainContractView(@NotNull MainActivity $receiver) {
      Intrinsics.checkParameterIsNotNull($receiver, "receiver$0");
      return (MainContract.View)$receiver;
   }
...
```

拡張関数として定義したMainActivityは`$receiver`となり、引数に入っていることが分かります。拡張関数は上記のように解釈されるため、`@Provides`と組み合わせて使うことが出来ます。

## まとめ

多分、使い所ないと思います😃😃😃
