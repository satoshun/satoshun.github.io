+++
date = "2018-01-07"
title = "Kotlin: 拡張関数からprotectedメソッドにアクセスする"
tags = ["android", "kotlin"]
blogimport = true
type = "post"
+++

LiveDataのonActiveメソッドで説明します。onActiveメソッドは以下で定義されています。

```java
package android.arch.lifecycle;

public abstract class LiveData<T> {
    protected void onActive() {
    }
}
```

onActiveはprotectedで定義されています。同一パッケージ内で拡張関数を定義することで、onActiveメソッドに拡張関数内からアクセスする事ができます。

```kotlin
package android.arch.lifecycle

fun <T> LiveData<T>.accessOnActive() {
  onActive() // LiveDataクラスと同一のパッケージで定義することで、protectedメソッドにアクセスできる
}
```

同一のパッケージで定義していない場合はprotectedメソッドにアクセスすることは出来ず、コンパイルエラーになります。

```kotlin
package android.arch

fun <T> LiveData<T>.accessOnActive() {
  onActive() // コンパイルエラー "Cannot access 'onActive': it is protected in 'LiveData'"
}
```

## なぜかこのような挙動になるか?

protectedメソッドは同一パッケージ内であればアクセスできるので、拡張関数を同一パッケージ内で定義することでprotectedメソッドにアクセスできるようになります。Javaでも同様のルールです。

## まとめ

protectedなメソッドに拡張関数内からアクセスするのは行儀的には良くないと思うので、奥の手段として使うのが良さそう

## 参考

- https://medium.com/@dpreussler/unit-testing-activity-lifecycle-4e740f71e68a
