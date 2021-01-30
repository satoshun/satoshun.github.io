+++
date = "Sat Jan 30 07:20:42 UTC 2021"
title = "Kotlin: RxJava -> Coroutineへの置き換えに便利なMigration.ktの紹介"
tags = ["kotlin", "coroutine", "flow"]
blogimport = true
type = "post"
draft = false
+++

Kotlin Coroutineには [Migration.kt](https://github.com/Kotlin/kotlinx.coroutines/blob/e16eb9d315cbee42bcadb438a8d62b10f65a9aa4/kotlinx-coroutines-core/common/src/flow/Migration.kt) が用意されており、RxJavaからCoroutineへの置き換えを手伝ってくれます。この記事では、どのように使うかを説明します。

---

例として、次のRxJavaを使ったクラスがあるとします。

```kotlin
import io.reactivex.Observable

class TestRepository {
  fun getUser(): Observable<User> {
    ...
  }
}
```

これは、次のように使うことが出来ます。

```kotlin
val disposble = repository.getUser()
  .subscribeOn(Schedulers.io())
  .subscribe(
    { println(it) },
    { println(it) }
  )
...
```

このコードを、Coroutineに置き換えていきます。

まず、TestRepositoryクラスのRxJava部分をCoroutineのFlowにします。

```kotlin
import kotlinx.coroutines.flow.Flow

private class TestRepository {
  fun getUser(): Flow<User> {
    ...
  }
}
```

そうすると、使っている側のコードでコンパイルエラーになります。なぜなら、Flow自体には、subscribeOnなどの関数が直接実装されていないためです。

しかし、Flowには拡張関数として、subscribeOnなどの関数が定義されています。なので、IDEの自動補完を使いimportすることで、一部、コンパイルエラーを解消することが出来ます。

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

これで、置き換え完了かというと、そうではありません。実はsubscribeOnなどの関数は定義されてはいるものの、実際には使うことは出来ません。

では、何が嬉しいかというと、subscribeOn関数は、次のようなエラーメッセージを出力します。

{{< figure src="/blog/kotlin/coroutine/coroutine-subscribeon-message.png" >}}

Flowへの置き換えのヒントメッセージを出力してくれます。この場合だと、flowOnを使うことを提案しています。
subscribe拡張関数も同様です。

この拡張関数を使うことで、RxJava -> Coroutineへの置き換えをいい感じに出来ます。

## 備考

- サポートしている関数は、[Migration.kt](https://github.com/Kotlin/kotlinx.coroutines/blob/e16eb9d315cbee42bcadb438a8d62b10f65a9aa4/kotlinx-coroutines-core/common/src/flow/Migration.kt)に定義されています

## 参考にしたページ

- [RxJava vs. Coroutines](https://blog.danlew.net/2021/01/28/rxjava-vs-coroutines/)
