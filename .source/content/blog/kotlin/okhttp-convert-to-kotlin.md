+++
date = "Sun Mar 24 02:36:27 UTC 2019"
title = "OkHttp: Java to KotlinのPRを見て勉強する"
tags = ["kotlin", "okhttp", "android"]
blogimport = true
type = "post"
+++

OkHttpがKotlin化をするというISSUEが立てられました。
[Upgrade OkHttp 3 to Kotlin and call it OkHttp 4](https://github.com/square/okhttp/issues/4723)

これの是非についてはさておき。現状、いくつかのJavaコードがKotlinへと置き換わっているので、それらのレビューで気になったこと、知らなかったこと、忘れがちなことを勉強がてらまとめたいと思います。

## checkNotNullを使うかどうか

[could also be *code* no preference myself](https://github.com/square/okhttp/pull/4745#discussion_r266693078)

Kotlinの標準ライブラリに、[checkNotNull](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/check-not-null.html)があります。
これは、値がnullなら`IllegalStateException`例外を投げるものです。

以下のコードは同じ意味を持ちます。

```kotlin
val state = someState ?: throw IllegalStateException("State must be set beforehand")

val state = checkNotNull(someState) { "State must be set beforehand" }
```

ただ、`no preference myself`と言っている通り、使うかどうかはプロジェクトで分かれそうです。
事前に使うかどうかを、決めておくと揉めなく良さそうだと思いました。


## 命名はto***が慣用的

[idiomatic naming would be toUrl on the Kotlin side](https://github.com/square/okhttp/pull/4745#discussion_r266693257)

OkHttpでは、HttpUrlをURLに変換するためのメソッドとして`fun url(): URL`が定義されています。しかし、`fun toUrl(): URL`のほうがKotlinっぽいよと指摘がありました。

確かに、言われてみるとAtoBクラス変換のメソッド名は、`to***`が多い気がします。ただし、今回は下位互換を保つために、一旦この修正は入りませんでした。

## constを使う

[discussion link](https://github.com/square/okhttp/pull/4745#discussion_r266695491)

constを使うと、Compile Time Constantsとなり、付けない場合に比べ効率的に動作します。ただし、プリミティブか、String型のみに有効です。

より詳しくは[公式ドキュメント](https://kotlinlang.org/docs/reference/properties.html#compile-time-constants)で。

constを付けたほうが良いことは知っていたのですが、公式ドキュメントを読んだことがなかったので勉強になりました。

## 事前計算済みプロパティにのみ依存している場合は、事前計算済みプロパティにを使う

[scheme is a val, so this can just be a precomputed property without the get()](https://github.com/square/okhttp/pull/4745#discussion_r266702974)

isHttpsプロパティは最初、get()で定義していたんですが、isHttpsで使っているschemeプロパティがvalなので、get()が取り除かれました。

事前計算出来るプロパティは事前に計算しておく方針のようです。そのほうが、無駄なメソッドが定義されないので、良いという判断なのでしょうか？

## プロパティ + get:JvmNameはより慣用的

[property + @getJvmName("size") seems more idiomatic](https://github.com/square/okhttp/pull/4745#discussion_r266697339)

単純かつ副作用がない関数はプロパティのほうがKotlinっぽいです。さらに、`@get:JvmName`を使うことで、Javaからスムーズに使うことが出来ます、

```kotlin
fun size(): Int = encodedNames.size

@get:JvmName("size")
val size get(): Int = encodedNames.size
```

## まとめ

- valをうまく使うことが、Kotlinっぽさを出すのに大切なんだなって
- `checkNotNull`は、スコープ関数と一緒で、ある程度使い方の認識を合わせないとコードの一貫性が無くなりそう

## 今回の記事を書くために調べたコミット

- [Convert some basic types to Kotlin #4741](https://github.com/square/okhttp/pull/4741)
- [HttpUrl in Kotlin #4745](https://github.com/square/okhttp/pull/4745)
- [Kotlin Platform refactor #4749](https://github.com/square/okhttp/pull/4749)
- [Refactor exceptions and static classes to Kotlin #4751](https://github.com/square/okhttp/pull/4751)
