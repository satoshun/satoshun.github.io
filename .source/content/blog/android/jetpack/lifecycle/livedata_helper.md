+++
date = "2018-12-01"
lastmod = "2018-12-02T00:00:00Z"
title = "MutableなLiveDataを特定のクラス外から更新できなくする"
tags = ["android", "jetpack", "livedata"]
blogimport = true
type = "post"
draft = false
+++

LiveDataの値を更新したい時、`MutableLiveData`を使うのが一般的だと思います。

```kotlin
class MainViewModel {
    val hoge = MutableLiveData<Int>()
}
```

この書き方だと、外のクラスから値を更新することが可能です。

```kotlin
val viewModel = MainViewModel()

// ok
viewModel.hoge.postValue(10000)
```

外のクラスからは更新出来ないようにするためには`LiveData`に型変換する必要があります。

例えば次のように書きます。

```kotlin
class MainViewModel {
    private val _hoge = MutableLiveData<Int>()
    val hoge: LiveData<Int> = _hoge // ここでLiveDataに型変換
}
```

こうすることで、外のクラスからは`MutableLiveData`が直接見えなくなり、明示的に型変換をしない限り`LiveData`の値を更新できなくなります。

ただこの書き方はフィールドの定義が増えるのでとてもめんどくさいです。
なので、それの解決策を以下で紹介します。

## その1

まずコードをのせます。

```kotlin
abstract class ViewModel2 {
  protected fun <T> ViewModelLiveData2<T>.postValue(value: T) {
    postValue(value)
  }

  protected fun <T> ViewModelLiveData2<T>.setValue(value: T) {
    setValue(value)
  }
}
```

```java
// ViewModel2と同じパッケージに定義
public class ViewModelLiveData2<T> extends LiveData<T> {
  @Override
  protected void postValue(T value) {
    super.postValue(value);
  }

  @Override
  protected void setValue(T value) {
    super.setValue(value);
  }
}
```

が定義になります。次に使い方です。

```kotlin
class MainViewModel2 : ViewModel2() {
  val userName = ViewModelLiveData2<String>()

  fun update() {
    userName.setValue("test")
    userName.postValue("test2")
  }
}

fun main2() {
  val viewModel = MainViewModel2()

  // compile error!!
  // viewModel.userName.setValue("")

  viewModel.update()
  viewModel.userName.observeForever { }
}
```

`ViewModelLiveData2`と`ViewModel2`を作りました（名前は適当です）。

`ViewModelLiveData2`クラスで`postValue`メソッドと`setValue`メソッドをオーバーライドし、
`ViewModel2`クラスと同じパッケージに入れることで、`ViewModel2`からそれらのメソッドをコール出来るようになり、
`ViewModel2`を継承したクラスからのみ`LiveData`の値を更新できます。

`viewModel.userName.setValue("")`とクラス外から`setValue`メソッドをコールするとコンパイルエラーになります。

`protected`メソッドが同一パッケージ内からアクセスすることが出来ることを利用したコードになります。

## その2

こちらもまずコードをのせます。

```kotlin
abstract class ViewModel3 {
  protected fun <T> ViewModelLiveData3<T>.postValue(value: T) {
    internalPostValue(value)
  }

  fun <T> ViewModelLiveData3<T>.setValue(value: T) {
    internalSetValue(value)
  }
}
```

```kotlin
class ViewModelLiveData3<T> : LiveData<T>() {
  internal fun internalPostValue(value: T) {
    postValue(value)
  }

  internal fun internalSetValue(v: T) {
    value = v
  }
}
```

が定義になります。次に使い方です。

```kotlin
class MainViewModel3 : ViewModel3() {
  val userName = ViewModelLiveData3<String>()

  fun update() {
    userName.setValue("test")
    userName.postValue("test2")
  }
}

fun main3() {
  val viewModel = MainViewModel3()

  // compile error
  // viewModel.userName.setValue("")

  viewModel.update()
  viewModel.userName.observeForever { }
}
```

`ViewModelLiveData3`と`ViewModel3`を作りました（名前は適当です）。

`ViewModelLiveData3`クラスと`ViewModel3`クラスを適当なサブモジュール内で定義します。
そして、Kotlinのinternalを修飾子を使うことで、外のモジュールからは直接値を更新することができなくなります。

`viewModel.userName.setValue("")`とクラス外から`setValue`メソッドをコールしようとするとコンパイルエラーになります。

## 補足

abstract classをinterfaceにして上記のメソッドをデフォルトメソッドにすると次のように書くことが出来ます。

```kotlin
interface ViewModel2 {
  fun <T> ViewModelLiveData1<T>.setValue(value: T) {
    this.value = value
  }

  fun <T> ViewModelLiveData1<T>.postValue(value: T) {
    postValue(value)
  }
}

fun main() {
  ...

  // compile error
  // viewModel.userName.setValue("")

  // ok
  with(viewModel) {
    userName.setValue("")
  }
}
```

applyやwithを使って`ViewModel2`がreceiverになると、`setValue`メソッドがコール出来るため、外から値を更新することが出来てしまいます。

## まとめ

- おそらくLiveDataの値を更新する部分は、`ViewModel`や`Store`クラスに集中すると思うので、それらのBaseクラスで上記のメソッドを定義することで楽ができるようになると思います。
- もっと良い、楽できる書き方があればぜひ教えてください!!

今回の検証に用いた[サンプルコードはここにあります](https://github.com/satoshun-android-example/LiveDataRemoveUnderScoreExample)😃
