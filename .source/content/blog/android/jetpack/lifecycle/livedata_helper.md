+++
date = "2018-12-01T00:00:00Z"
title = "MutableなLiveDataを特定のクラス以外から更新できなくする"
tags = ["android", "jetpack", "livedata"]
blogimport = true
type = "post"
draft = true
+++

LiveDataの値を更新したい時、`MutableLiveData`を使って更新するのが一般的だと思います。

```kotlin
class MainViewModel {
    val hoge = MutableLiveData<Int>()

    fun updateValue() {
        hoge.value = 10
    }
}
```

ただこれだと、外のクラスから値を更新することが出来ます。それを防ぐために`LiveData`に型を変換したいケースがあります。

例えば次のように書きます。

```kotlin
class MainViewModel {
    private val _hoge = MutableLiveData<Int>()
    val hoge: LiveData<Int> = _hoge

    fun updateValue() {
        _hoge.value = 10
    }
}
```

このようにすることで、外のクラスからは`MutableLiveData`が見えなくなり、型変換などをしない限り、値を更新することができなくなります。

ただこの書き方はフィールドの定義が増えるのでめんどくさいです。
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

`ViewModelLiveData2`クラスで`postValue`メソッドと`setValue`メソッドをオーバーライドし、`ViewModel2`クラスと同じパッケージに入れることで、`ViewModel2`からそれらのメソッドをコールすることが出来るようになります。
`ViewModel2`にそれらのメソッドをコールするメソッドを定義することで、`ViewModel2`を継承したクラスから値を更新することができます。

`viewModel.userName.setValue("")`とクラス外から`setValue`メソッドをコールしようとするとコンパイルエラーになります。

`protected`メソッドが同一パッケージ内からアクセスすることが出来ることを利用したコードになります。

##  その2

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
  val userName = viewModelLiveData3<String>()

  fun update() {
    userName.setValue("test")
    userName.postValue("test2")
  }
}

fun main3() {
  val viewModel = MainViewModel3()
  with(viewModel) {
    viewModel.userName.postValue("10")
  }

  // compile error
//  viewModel.userName.setValue("")

  viewModel.update()
  viewModel.userName.observeForever { }
}
```

`ViewModelLiveData3`と`ViewModel3`を作りました（名前は適当です）。

今回はオーバーライドするのではなく、internalを修飾子をつけてその中で`setValue`と`postValue`をコールしています。
この2つのメソッドは`protected`でLiveData内で定義されているので継承したクラスからは呼び出すことが可能です。
そして`ViewModelLiveData3`クラスと`ViewModel3`クラスの定義を適当なサブモジュール内でします。
そうすることで、モジュール外からは、直接`ViewModelLiveData3`の値を更新できなくなります。

`viewModel.userName.setValue("")`とクラス外から`setValue`メソッドをコールしようとするとコンパイルエラーになります。

## まとめ

おそらくLiveDataの値を更新する部分は、`ViewModel`や`Store`クラスになると思うので、それのBaseクラスで上記のメソッドを定義することで楽できるようになると思います😃

今回の検証に用いた[サンプルコードはここにあります](test)😃
