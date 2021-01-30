+++
date = "Sat Jan 30 07:20:42 UTC 2021"
title = "Kotlin: RxJava -> Coroutineへの置き換えに便利なMigration.ktの紹介"
tags = ["kotlin", "coroutine", "flow"]
blogimport = true
type = "post"
draft = true
+++

Kotlin Coroutineには[Migration.kt](https://github.com/Kotlin/kotlinx.coroutines/blob/e16eb9d315cbee42bcadb438a8d62b10f65a9aa4/kotlinx-coroutines-core/common/src/flow/Migration.kt)が用意されており、RxJavaからCoroutineへの置き換えを手伝ってくれます。具体的にどのように使うかを説明します。

例として、次のRxJavaを使ったクラスがあるとします。

```kotlin
import io.reactivex.Observable

class TestRepository {
  fun getUser(): Observable<User> =
    ...
}
```

これは、次のように使っています。

```kotlin
val disposble = repository.getUser()
  .subscribeOn(Schedulers.io())
  .subscribe(
    { println(it) },
    { println(it) }
  )
```

次に、TestRepositoryクラスのRxJava部分をCoroutineのFlowに置き換えます。

```kotlin
import kotlinx.coroutines.flow.Flow

private class TestRepository {
  fun getUser(): Flow<User> =
    ...
}
```

そうすると、当然コンパイルエラーになります。なぜなら、Flow自体には、subscribeOnなどの関数は直接実装されていないためです。
しかし、Flowには拡張関数として、subscribeOnなどの関数が定義されています。なので、IDEの自動補完を使うことで、一部、コンパイルエラーを解消することが出来ます。

```kotlin
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.subscribe
import kotlinx.coroutines.flow.subscribeOn

val disposble = repository.getUser()
  .subscribeOn(Schedulers.io())
  .subscribe(
    { println(it) },
    { println(it) }
  )
```

このとき、IDEのsubscribeOn関数は、次のようなエラーメッセージを出力します。

{{< figure src="/blog/kotlin/coroutine/coroutine-subscribeon-message.png" >}}

subscribeOn拡張関数自体は使うことが出来ないんですが、Flowへの置き換えのヒントメッセージを出力してくれます。同様にsubscribe関数でもヒントメッセージを出力してくれます。

これを使うことで、RxJava -> Coroutineへの置き換えをいい感じに出来ます。

## 備考

- サポートしている関数は、[Migration.kt](https://github.com/Kotlin/kotlinx.coroutines/blob/e16eb9d315cbee42bcadb438a8d62b10f65a9aa4/kotlinx-coroutines-core/common/src/flow/Migration.kt)に定義されています

## 参考にしたページ

- [RxJava vs. Coroutines](https://blog.danlew.net/2021/01/28/rxjava-vs-coroutines/)
