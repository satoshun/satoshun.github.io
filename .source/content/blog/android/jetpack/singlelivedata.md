+++
date = "2018-05-19T00:00:00Z"
title = "Android: 拡張関数でLiveDataでSingle Eventを扱う"
tags = ["android", "jetpack", "aac"]
blogimport = true
type = "post"
+++

LiveDataは最新の値をキャッシュするため、エラー値の取扱などに困ることがあります。
適当なクラスを作るのもいいのですが、拡張関数で表現することも出来るのでその紹介です。

定義は以下のようになります。

```kotlin
fun <T> oneShotLiveData(): MutableLiveData<T> {
  // skip用の初期値を入れておく
  return MutableLiveData<T>().also { it.value = null }
}

fun <T> LiveData<T>.observeOneShot(owner: LifecycleOwner, observer: ((T?) -> Unit)) {
  // 最初の値は常にskipすることで、キャッシュを無視する
  val firstIgnore = AtomicBoolean(true)
  this.observe(owner, Observer {
    if (firstIgnore.getAndSet(false)) return@Observer
    observer(it)
  })
}
```

使う時はこんな感じで使います。

```kotlin
// TestViewModel.kt
class TestViewModel: ViewModel() {
  val errorOneShot = oneShotLiveData<String>()
}

// TestActivity.kt
testViewModel = ViewModelProviders.of(this).get(TestViewModel::class.java)
testViewModel.errorOneShot.observeOneShot(activity) {
  Log.d("one", it.toString())
}
```

メリットはサブクラスを作らずに済むところです。


## 参考

- https://github.com/googlesamples/android-architecture/blob/dev-todo-mvvm-live/todoapp/app/src/main/java/com/example/android/architecture/blueprints/todoapp/SingleLiveEvent.java
- https://medium.com/@star_zero/singleliveevent-livedata-with-multi-observers-384e17c60a16
