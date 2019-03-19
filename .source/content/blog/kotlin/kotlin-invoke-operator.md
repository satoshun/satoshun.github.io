+++
date = "Tue Mar 19 12:31:05 UTC 2019"
title = "Kotlin: コンストラクタ呼び出しっぽく関数やcompanion objectを使う"
tags = ["kotlin"]
blogimport = true
type = "post"
+++

KotlinではJavaと異なり、コンストラクタ呼び出しの時に`new`キーワードが必要ありません。

```kotlin
class A
...
val a = A()
```

よって、次のように関数をコンストラクタのように使うことが出来ます。

```kotlin
// Coroutine Jobの定義
@Suppress("FunctionName")
public fun Job(parent: Job? = null): Job = JobImpl(parent)
...
val job = Job()
```

また、次のようにcompanion object + operator invokeを使うことも出来ます。

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
