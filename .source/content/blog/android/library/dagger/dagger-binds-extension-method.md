+++
date = "Sun Apr 24 06:16:34 UTC 2022"
title = "Dagger2: 拡張関数を使い、@Bindsの定義をシンプルにする"
tags = ["android", "dagger"]
blogimport = true
type = "post"
draft = false
+++

Dagger2のtipsです。

Dagger2では、`Binds`アノテーションを使うことで、キャストした型をバインディングすることが出来ます。
例えば、次のように使います。

```kotlin
@Module
interface AppModule {
  @Binds fun bindHoge(impl: HogeImpl): Hoge
}
```

このときに、拡張関数を使うことで、少しシンプルに書くことが出来ます。

```kotlin
@Module
interface AppModule {
  @Binds fun HogeImpl.bindHoge(): Hoge
}
```

同様に、拡張プロパティを使うことも出来ます。

```kotlin
@Module
interface AppModule {
  @get:Binds val HogeImpl.bindHoge: Hoge
}
```

拡張関数を使うと、コード量が減り、シンプルになるのでおすすめです。
