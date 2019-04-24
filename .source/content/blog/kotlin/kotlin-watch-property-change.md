+++
date = "Wed Apr 24 00:36:14 UTC 2019"
title = "Kotlin: プロパティの変更を検知する"
tags = ["kotlin", "android"]
blogimport = true
type = "post"
draft = true
+++

オブジェクト自身の変更ではなく、対象のオブジェクトが持つプロパティの変更を汎用的に検知する方法の紹介です。

今回紹介したいコードを次になります。

```kotlin
fun <S, A1> LiveData<S>.watch(prop1: KProperty1<S, A1>): LiveData<A1> =
  this
    .map { prop1.get(it) }
    .distinctUntilChanged()

fun <S, A1, A2> LiveData<S>.watch(
  prop1: KProperty1<S, A1>,
  prop2: KProperty1<S, A2>
): LiveData<Pair<A1, A2>> =
  this
    .map { prop1.get(it) to prop2.get(it) }
    .distinctUntilChanged()

fun <S, A1, A2, A3> LiveData<S>.watch(
  prop1: KProperty1<S, A1>,
  prop2: KProperty1<S, A2>,
  prop3: KProperty1<S, A3>
): LiveData<Triple<A1, A2, A3>> =
  this
    .map { Triple(prop1.get(it), prop2.get(it), prop3.get(it)) }
    .distinctUntilChanged()

...
```

KProperty1はプロパティの値を取得するためのインターフェースになります。
`distinctUntilChanged`はLiveData ktxに追加された便利拡張関数です。
LiveDataを使いましたが、RxJavaやCoroutineでも同じような感じで書けると思います。

---

次にサンプルコードです。

```kotlin
class Presenter(initializeUser: User = User(name = "init", age = 0)) {
  val user = MutableLiveData<User>(initializeUser)
  val watchUserName = user.watch(User::name)
  val watchUserAge = user.watch(User::age)
}

// 以下main
val presenter = Presenter()

// 監視
presenter.watchUserName.observe(this) {
  Log.d("watchUserName", it)
}
presenter.watchUserAge.observe(this) {
  Log.d("watchUserAge", it.toString())
}

// 適当にUserを更新
presenter.user.postValue(User(name = "posted1", age = 0))
```

この場合、Userのnameプロパティだけが変更されているので、`presenter.watchUserName`に登録したObserverのみが発火します。
変更が加わったプロパティだけを無事検知することができました😃

## まとめ

- 多くのデータを持ったオブジェクトの一部の変更のみ検知したい時に使うと便利かもしれない😋

---

内容におかしい点や、もっとこうしたほうがいいよって！！いうのがあればTwitterなどから教えてもらえればとても嬉しいです😊
