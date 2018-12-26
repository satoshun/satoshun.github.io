+++
date = "2018-10-14T00:00:00Z"
lastmod = "Wed Dec 26 03:06:20 UTC 2018"
title = "UnitテストでViewModelのonClearedをテストする"
tags = ["android", "jetpack", "unittest"]
blogimport = true
type = "post"
draft = false
+++

ふとAACの[ViewModel](https://developer.android.com/reference/androidx/lifecycle/ViewModel)のonClearedメソッドをテストしたくなったので、 2つのやりかたを紹介します。

環境は

```
"junit:junit:4.12"
"androidx.test:rules:1.1.0-beta02"
"androidx.test:runner:1.1.0-beta02"
"androidx.test.ext:junit:1.0.0-beta02"
"com.nhaarman:mockito-kotlin-kt1.1:1.5.0"
"org.robolectric:robolectric:4.0-beta-1"
```

になります。

また、サンプルコードは [GitHub](https://github.com/satoshun-android-example/Tests/blob/master/app/src/test/java/com/github/satoshun/example/architectures/BaseViewModelTest.kt)にあるので、参考してください😊

### 1. `ViewModelStore`を使う

`ViewModelProviders.of(activity).get(class)`からViewModelを取得したときに、取得したViewModelは[ViewModelStore](https://developer.android.com/reference/androidx/lifecycle/ViewModelStore)にキャッシュされます。このViewModelStoreはFragmentActivityから取得できるので、次のように書くことでViewModelのonClearedをテストすることが出来ます。

```kotlin
@RunWith(AndroidJUnit4::class)
class BaseViewModelTest {
  @get:Rule val activityRule = ActivityTestRule(FragmentActivity::class.java)

  @Test
  fun `dispose a coroutine when finished lifecycle of ViewModel`() {
    activityRule.activity.viewModelStore.clear() // ViewModelが開放される
  }
}
```

このテストはコード的には簡単ですが、ViewModelStoreがViewModelのライフサイクルを管理しているということを知っている、内部実装の詳細まで知っているため、テストとしてふさわしくない可能性があります。

なので、素直にonDestroyをコールするテストも書いてみます。

### 2. `Instrumentation.callActivityOnDestroy`を使う

`Instrumentation`クラスを使うことでActivityのライフサイクルをコントロールすることが出来ます。
`Instrumentation`は`InstrumentationRegistry`クラスから取得することができ、次のように書くことで、`onDestroy`をコールすることができます。

```kotlin
@Test
fun `dispose a coroutine when finished lifecycle of ViewModel 2`() {
  // onDestroyがコールされViewModelが開放される
  InstrumentationRegistry.getInstrumentation().callActivityOnDestroy(activityRule.activity)
}
```

*以下追記*

### 3. ActivityScenarioを使う

新しく追加されたActivityScenarioを使うとよりすっきりと`onDestroy`を表現することが出来ます。

```kotlin
@Test
fun `dispose a coroutine when finished lifecycle of ViewModel 3`() {
  val scenario = ActivityScenario.launch(FragmentActivity::class.java)
  ...
  // onDestroy状態になり、ViewModelのonClearedがコールされる
  scenario.moveToState(Lifecycle.State.DESTROYED)
}
```

ActivityScenarioは`moveToState`を介して、Activityのライフサイクルを操作することが出来ます。
今回はonDestroy状態にしたいので、scenario.moveToState(Lifecycle.State.DESTROYED)をコールします。

これで、Unitテストで`ViewModel.onCleared`のテストをすることが出来ます！！

---

以上です。Happy Testing😊😊😊

- [サンプルコード](https://github.com/satoshun-android-example/Tests/blob/master/app/src/test/java/com/github/satoshun/example/architectures/BaseViewModelTest.kt)
