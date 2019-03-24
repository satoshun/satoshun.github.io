+++
date = "Sat Mar 23 13:15:27 UTC 2019"
title = "OkHttp"
tags = ["kotlin"]
blogimport = true
type = "post"
draft = true
+++

# HttpUrl in Kotlin

[HttpUrl in Kotlin #4745](https://github.com/square/okhttp/pull/4745)

## checkNotNullを使う

Kotlinの標準ライブラリに、[checkNotNull](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/check-not-null.html)があります。
これは、値がnullなら`IllegalStateException`例外を投げるものです。

以下のコードは同じ意味を持ちます。

```kotlin
val state = someState ?: throw IllegalStateException("State must be set beforehand")

val state = checkNotNull(someState) { "State must be set beforehand" }
```

個人的には`checkNotNull`のほうが意図が伝わりやすい気がするので好きです。

## 命名はto***が慣用的

`idiomatic naming would be toUrl on the Kotlin side`
- [link](https://github.com/square/okhttp/pull/4745#discussion_r266693257)

OkHttpでは、HttpUrlをURLに変換するためのメソッドとして`fun url(): URL`が定義されています。`fun toUrl(): URL`のほうがKotlinっぽいよとの指摘がありました。

確かに、言われてみるとAtoBクラス変換のメソッド名は、`to***`が多い気がします。ただし、今回は下位互換を保つために、この修正は入りませんでした。

## constを使う

- [link](https://github.com/square/okhttp/pull/4745#discussion_r266695491)

constを使うと、Compile Time Constantsとなり、つけない場合に比べ効率的に動作します。ただし、プリミティブか、String型のみに有効です。

より詳しくは[公式ドキュメント](https://kotlinlang.org/docs/reference/properties.html#compile-time-constants)で。

## 事前計算済みプロパティにのみ依存している場合は、事前計算済みプロパティにを使う

`scheme is a val, so this can just be a precomputed property without the get()`
- [link](https://github.com/square/okhttp/pull/4745#discussion_r266702974)

isHttpsは最初、get()で定義していたんですが、isHttps内で使っているschemeプロパティがvalなので、get()が取り除かれました。
事前計算出来るプロパティは事前に計算しておく方針のようです。

## プロパティ + get:JvmNameはより慣用的

`property + @getJvmName("size") seems more idiomatic`
- [link](https://github.com/square/okhttp/pull/4745#discussion_r266697339)

単純かつ副作用がない関数はプロパティのほうがKotlinっぽいようです。

```kotlin
fun size(): Int = encodedNames.size

@get:JvmName("size")
val size get(): Int = encodedNames.size
```

# まとめ

- valをうまく使うことが、Kotlinっぽさを出すのに大切なんだなって
- `checkNotNull`使ったことなかったけど、良さそうなので積極的に使っていきたい

# 参照コミット

- [Convert some basic types to Kotlin #4741](https://github.com/square/okhttp/pull/4741)
- [HttpUrl in Kotlin #4745](https://github.com/square/okhttp/pull/4745)
- [Kotlin Platform refactor #4749](https://github.com/square/okhttp/pull/4749)
- [Refactor exceptions and static classes to Kotlin #4751](https://github.com/square/okhttp/pull/4751)
