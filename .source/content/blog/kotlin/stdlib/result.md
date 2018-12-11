+++
date = "2018-12-11"
title = "Kotlin: Resultの簡単なまとめ"
tags = ["kotlin", "stdlib"]
blogimport = true
type = "post"
draft = false
+++

[Result KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/stdlib/result.md)

`Result`が1.3からstdlibに入ったので紹介をしたいと思います。

Resultは`Success T | Failure Throwable`の2状態のいずれかを表現出来ます。成功状態のときはSuccessを、失敗状態のときはFailureを内包します。

## 基本的な使い方

使い方を見ていきます。まず、Resultインスタンスの生成は、success、failureメソッドを通して行います。

```kotlin
val i: Result<Int> = Result.success(10)
val t: Result<Int> = Result.failure(IOException())
```

また、`runCatching`関数を使うことで、failする可能性があるメソッドをResult型に丸めることも出来ます。runCathinngのほうがよく使うと思います。

```kotlin
val a = runCatching { doSomeThing() }
```

Resultに対する操作は以下のようにします。

```kotlin
// successに対して操作
val i = Result.success(10)
i.getOrNull() // 10
i.isSuccess // true
i.exceptionOrNull() // null
i.map { 10 * 10 }.getOrNull() // 100
i.onSuccess { println("success") }.onFailure { println("failure") }
i.recover { 1111 }.getOrNull() // 10

// failureに対して操作
val t = Result.failure(IOException())
t.getOrNull() // null
t.isFailure // true
t.exceptionOrNull() // IOException
t.map { 10 * 10 }.getOrNull() // null
t.onSuccess { println("success") }.onFailure { println("failure") }
t.recover { 1111 }.getOrNull() // 1111
t.getOrThrow() // throw IOException
```

値successに対しての操作と、failureに対しての操作をそれぞれすることが出来ます。

また、Resultを使うことで、functionalっぽく書くことが出来ます。

```kotlin
try {
    val data = doSomethingSync()
    processData(data)
} catch(e: Throwable) {
    showErrorDialog(e)
}

-->

runCatching { doSomethingSync() }
    .onFailure { showErrorDialog(it) }
    .onSuccess { processData(it) }
```

クライアント側から見たときは、functionalっぽく書ける点がメリットだと思います。

## 補足

### Resultは戻り値として返せない

これは、Resultのユースケースから外さないための制約です。なぜなら、Resultはその関数内で処理すべき例外であり、関数の呼び出し側で処理させるべきではないためです。
例えば、ある関数の返り値が`Result<User>`となっていたときに、失敗する可能性は呼び出し側に伝わりますが、どのような失敗が起こるか分かりません。どのようにエラーを処理するかはこの関数内で完結すべきなので、返り値としてResultを指定することは出来ません。

## まとめ

簡単にまとめました。より詳細な内容は[Result KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/stdlib/result.md)を見てください😊
