+++
date = "Sun Feb  2 04:51:36 UTC 2020"
title = "Dagger2: 2.26時点でのKotlinサポート状況"
tags = ["dagger", "android", "di"]
blogimport = true
type = "post"
draft = false
+++

Daggerの2.25から少しずつKotlin向けの対応が入っています。

この記事ではそれらについて紹介していきます。


## objectクラスにJvmStaticが必須ではなくなった

2.25から入った変更です。

### 前

```kotlin
@Module
object AppModule {
  @JvmStatic // JvmStaticの指定が必須だった
  @Provides
  fun provideFuga(): Fuga = Fuga()
}
```

### 今

```kotlin
@Module
object AppModule {
  @Provides
  fun provideFuga(): Fuga = Fuga()
}
```

## Qualifierを使うときに、fieldの指定がいらなくなった

これも2.25から入った変更です。

### 前

```kotlin
@Inject
@field:HogeQualifier // @field:の指定が必須だった
lateinit var namedObject: NamedObject
```

### 今

```kotlin
@Inject
@HogeQualifier
lateinit var namedObject: NamedObject
```


## companion objectにModuleをつけなくて良くなった

これは2.26から入った変更です。

### 前

```kotlin
@Module
interface MainActivityModule {
  @Module // 必須だった
  companion object {
    @JvmStatic // 必須だった
    @Provides
    fun provideTestHoge(): TestHoge = TestHoge()
  }
}
```

### 今

```kotlin
@Module
interface MainActivityModule {
  companion object {
    @Provides
    fun provideTestHoge(): TestHoge = TestHoge()
  }
}
```


## まとめ

ハマりがちだった部分が少しずつ解消されていく😊
