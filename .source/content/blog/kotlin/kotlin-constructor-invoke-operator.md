+++
date = "Tue Mar 19 12:07:37 UTC 2019"
title = "Kotlin: ほげ"
tags = ["kotlin"]
blogimport = true
type = "post"
draft = true
+++

KotlinではJavaと異なり、コンストラクタ呼び出しの時に`new`キーワードが必要ありません。

```kotlin
class A
...
val a = A()
```

次のように関数をコンストラクタのように使うことが出来ます。

```kotlin
// Coroutine Jobの定義
@Suppress("FunctionName")
public fun Job(parent: Job? = null): Job = JobImpl(parent)
...
val job = Job()
```

また、次のようにcompanion objectを使うことも出来ます。

```kotlin
// ref: https://github.com/JakeWharton/retrofit2-kotlin-coroutines-adapter
class CoroutineCallAdapterFactory private constructor() : CallAdapter.Factory() {
  companion object {
    @JvmStatic @JvmName("create")
    operator fun invoke() = CoroutineCallAdapterFactory()
  }
}
...
val factory = CoroutineCallAdapterFactory()
```

## まとめ

- JobImplのような実装クラスを隠したいときに便利
