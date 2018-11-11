+++
date = "2018-11-11"
title = "Kotlin: Contracts + 拡張関数でより便利に"
tags = ["kotlin", "contracts"]
blogimport = true
type = "post"
+++

Kotlin 1.3.0からContractsが実装されました。
Contractsを使うことで、関数がどのような振る舞いをするか、どういう効果をもたらすかを定義（契約）することが出来ます。

例えば、`isNullOrEmpty`メソッドがfalse返すなら、
Contractsによりnullでないことが保証されます。

```kotlin
val a: String? = ...
if (!a.isNullOrEmpty()) {
    println(a.length) // !!が必要ない
}
```

Contractsがない時代だと呼び出し元で`isNullOrEmpty`がどんな振る舞いをするかを知るすべがなかったので、
`!!`をつける必要があったのですが、Contractsによりnullでないことが保証できるので、`!!`を省略できます。

`isNullOrEmpty`の実装は次のようになります。

```kotlin
@kotlin.internal.InlineOnly
public inline fun CharSequence?.isNullOrEmpty(): Boolean {
    contract {
        returns(false) implies (this@isNullOrEmpty != null)
    }

    return this == null || this.length == 0
}
```

`contract`はDSL（関数）として定義されています。
これを呼び出し、そのブロックの中でこの関数が満たす振る舞いを定義する事ができます。

`isNullOrEmpty`の場合は`returns(false) implies (this@isNullOrEmpty != null)`が契約として定義されています。

これは、`「returns(false)`: falseを返すなら `(this@isNullOrEmpty != null)`: 自分自身がnullじゃない」という意味になります。
なので呼び出し元ではfalseが返ってきたら、nullではないことが保証されるので、smartcastにより`!!`をつける必要がなくなるわけです。

なので、例えば`T.isEmpty(t: T?): Boolean`のようなメソッドがあり、ついでにnullチェックもこの関数の中でやっているようなときは、
contractを定義することでより使いやすい関数にすることが出来ます。

他の例を見てみます。スコープ関数`apply`の実装は次になります。

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}
```

`apply`関数内では、`callsInPlace(block, InvocationKind.EXACTLY_ONCE)`が契約として定義されています。
これは、`block`関数が必ず1度呼び出されることを意味します。
これにより、以下のように書くことが可能になります。

```kotlin
val a: String
hoge.apply {
    a = "hoge"
}
println(a)
```

`apply`関数は1度しか呼び出されないので`val a: String`の初期化が、`apply`関数内で正しく行われることが保証されます。
Kotlin 1.3.0以前のcontractが内時代では上記のコードはコンパイルエラーになっていたのですが、
contractにより、実行することが可能になりました。

今まで見てきたのはKotlinのスタンダートライブラリに入っていた関数ですが、カスタムで定義することも可能です。
今回は例として、`ActivityScenario.onActivity`メソッドをcontract + 拡張関数を使ってより便利にしたいと思います。

[ActivityScenario.onActivity](https://github.com/android/android-test/blob/f2f3589c9d6e2ff5740117192cb7e13bd8873a0f/core/java/androidx/test/core/app/ActivityScenario.java#L500)メソッドは、callbackを登録すると、Activityの準備ができたタイミングでcallbackが叩かれます。そして、この`onActivity`メソッドは一度しかコールされず、実行したスレッドをブロックします。なので、前述した`apply`関数と同じcontractを書くことが可能です。

以下のように拡張関数を書きます。

```kotlin
@UseExperimental(ExperimentalContracts::class)
fun <T : Activity> ActivityScenario<T>.onActivity2(block: (T) -> Unit) {
  contract {
    callsInPlace(block, InvocationKind.EXACTLY_ONCE)
  }
  onActivity {
    block(it)
  }
}


// コンパイルエラーにならない!!
val activity: Activity
scenario.onActivity2 {
    activity = it
}
println(activity)
```

`@UseExperimental(ExperimentalContracts::class)`をつけることで、ユーザ定義のcontractを定義することが出来ます。

## まとめ

- 1度しかコールされないcallbackや、関数内でnullチェックをする場合はcontractを使うと超便利になるかも
- 拡張関数を新しく定義することで、既存のメソッドをよりkotlin-friendlyなメソッドにできるかも!?

## 参考

- https://github.com/Kotlin/KEEP/blob/master/proposals/kotlin-contracts.md
