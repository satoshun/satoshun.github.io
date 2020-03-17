+++
date = "Tue Mar 17 09:33:04 UTC 2020"
title = "Dagger2: Dagger2.27の新しいエラーメッセージ"
tags = ["dagger", "android", "di"]
blogimport = true
type = "post"
draft = false
+++

[Dagger 2.27でエラーメッセージ](https://github.com/google/dagger/releases/tag/dagger-2.27)の改善が入っていたので、軽く試してみました。試してみた系の記事になります。

## まず有効にする

この機能はexperimentalなので、明示的に有効にする必要があります。kaptを使っているなら、次のように有効にします。

```groovy
allprojects {
  afterEvaluate {
    extensions.findByName("kapt")?.arguments {
      arg("dagger.experimentalDaggerErrorMessages", "enabled")
    }
  }
}
```

## 適当にエラーを出してみる

まずは、Providesの指定をしていないのにInjectしてみます。

```kotlin
// @Provides <-- エラーを出したいのでコメントアウト
fun provideTestObject(): TestObject = TestObject()

@Inject lateinit var testObject: TestObject
```

すると、次のようになります。

### 新しいエラーログ

<a href="/blog/android/library/dagger/dagger-missing-binding-new.png">{{< figure src="/blog/android/library/dagger/dagger-missing-binding-new.png" >}}</a>

### 古いエラーログ

<a href="/blog/android/library/dagger/dagger-missing-binding-old.png">{{< figure src="/blog/android/library/dagger/dagger-missing-binding-old.png" >}}</a>

---

次に、複数Providesの指定をして、Injectしてみます。

```kotlin
@Provides
fun provideTestObject(): TestObject = TestObject()

@Provides
fun provideTestObject2(): TestObject = TestObject()

@Inject lateinit var testObject: TestObject
```

すると、次のようになります。

### 新しいエラーログ

<a href="/blog/android/library/dagger/dagger-duplicate-binding-new.png">{{< figure src="/blog/android/library/dagger/dagger-duplicate-binding-new.png" >}}</a>

### 古いエラーログ

<a href="/blog/android/library/dagger/dagger-duplicate-binding-old.png">{{< figure src="/blog/android/library/dagger/dagger-duplicate-binding-old.png" >}}</a>

## まとめ

全部のエラーを見たわけではないので分からないですが、手元で軽くエラーを出して見た感じだと、見やすくなったかなと思います。
Daggerのエラーメッセージは難しいことで有名?だったと思うので、これで改善してくれればとても嬉しい😃
