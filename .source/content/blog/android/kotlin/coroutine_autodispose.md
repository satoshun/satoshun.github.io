+++
date = "Sun Dec 23 09:14:43 UTC 2018"
title = "Coroutine + AutoDisposeを作ってみた"
tags = ["android", "kotlin", "coroutine", "library"]
blogimport = true
type = "post"
+++

Coroutine + AutoDisposeの実装について考えてみました。結論から言うと、`ContinuationInterceptor`を使えば上手くいきそうです。

## ContinuationInterceptorとは?

ContinuationInterceptorは次のようなインターフェースです。

```kotlin
/**
 * Marks coroutine context element that intercepts coroutine continuations.
 * The coroutines framework uses [ContinuationInterceptor.Key] to retrieve the interceptor and
 * intercepts all coroutine continuations with [interceptContinuation] invocations.
 */
@SinceKotlin("1.3")
public interface ContinuationInterceptor : CoroutineContext.Element {
  public fun <T> interceptContinuation(continuation: Continuation<T>): Continuation<T>
  public fun releaseInterceptedContinuation(continuation: Continuation<*>)
  ...
}
```

`interceptContinuation`からContinuationを受け取ることができ、Continuationは自身のCoroutineContextを持っているので、そこからJobを取得することが出来ます。それを利用することでAndroid Lifecycleと協調して動くContinuationInterceptorを実装することが出来ます。

```kotlin
class LifecycleContinuationInterceptor(
  private val lifecycle: Lifecycle
) : ContinuationInterceptor {
  override val key: CoroutineContext.Key<*>
    get() = ContinuationInterceptor

  override fun <T> interceptContinuation(continuation: Continuation<T>): Continuation<T> {
    // ContinuationからJobを取得
    val job = continuation.context[Job]
    if (job != null) {
      lifecycle.addJob(job)
    }
    return continuation
  }
}

fun LifecycleOwner.addJob(job: Job) {
  lifecycle.addJob(job)
}

fun Lifecycle.addJob(job: Job) {
  val state = this.currentState
  val event = when (state) {
      ...
  }
  val observer = LifecycleJobObserver(job, event, this)
  this.addObserver(observer)
  job.invokeOnCompletion(observer)
}

private class LifecycleJobObserver(
  private val job: Job,
  private val event: Lifecycle.Event,
  private val lifecycle: Lifecycle
) : LifecycleObserver, CompletionHandler {
  @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
  fun onEvent(owner: LifecycleOwner, event: Lifecycle.Event) {
    if (event == this.event) {
      owner.lifecycle.removeObserver(this)
      job.cancel()
    }
  }

  override fun invoke(cause: Throwable?) {
    lifecycle.removeObserver(this)
  }
}
```

これで、Android Lifecycleと協調して動くContinuationInterceptorが出来ました。

フルコードは[ここに](https://github.com/satoshun-android-example/AutoDisposeExample/blob/master/autodispose/src/main/java/com/github/satoshun/coroutine/autodispose/lifecycle/LifecycleContinuationInterceptor.kt)あります。

## 使い方

使い方は次のようになります。

```kotlin
abstract class BaseActivity : AppCompatActivity(),
  CoroutineScope {

  private val job = Job()
  override val coroutineContext get() = job +
      Dispatchers.Main +
      LifecycleContinuationInterceptor(this) // ここでInterceptorを登録
}

class MainActivity : BaseActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    // onCreateでlaunchしているので、onDestroyで自動的にキャンセルされる
    launch {
      ...
    }
  }

  override fun onResume() {
    super.onResume()

    // onResumeでlaunchしているので、onPauseで自動的にキャンセルされる
    launch {
      ...
    }
  }
}
```

となります。[Rx-AutoDispose](https://github.com/uber/AutoDispose)のように実行したタイミングに応じて、キャンセルする場所を自動的に登録してくれます!!

## まとめ

- もっと良い書き方が出来るか模索しているので、より適したAPI等を知っている人がいれば教えてくれると嬉しいです😊😊😊

サンプルコードです😃[satoshun-android-example/AutoDisposeExample](https://github.com/satoshun-android-example/AutoDisposeExample)
