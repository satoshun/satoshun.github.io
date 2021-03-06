+++
date = "Sat Mar  6 03:43:11 UTC 2021"
title = "Kotlin: infixのサンプルコード紹介"
tags = ["kotlin"]
blogimport = true
type = "post"
draft = false
+++

Kotlinにはinfix記法がありますが、自分で定義して使ったことがないので、どのようなケースで有用なのかを知るために、
世の中のライブラリの使用例を紹介します。有名なライブラリが中心です。

### Kotlin本体

https://github.com/JetBrains/kotlin

KotlinではPair、ビット演算などで使われています。

```kotlin
// public infix fun <A, B> A.to(that: B): Pair<A, B>

10 to "hoge"
```

```kotlin
// public inline infix fun Byte.and(other: Byte): Byte
// public inline infix fun Byte.or(other: Byte): Byte

byte1 and byte2
byte1 or byte2
```

```kotlin
// public inline infix fun CharSequence.matches(regex: Regex): Boolean

"abc" matches Regex("")
```

### Gradleスクリプト

今までのGroovyバージョンと書き心地を揃えるために、infixを使っています。

```kotlin
// infix fun PluginDependencySpec.version(version: String?): PluginDependencySpec

plugins {
  id("com.jfrog.bintray") version "0.4.1"
}
```

### kotlinx.coroutine

https://github.com/Kotlin/kotlinx.coroutines

```kotlin
// public infix fun <E, R> ReceiveChannel<E>.zip(other: ReceiveChannel<R>): ReceiveChannel<Pair<E, R>>

channel1 zip channel2
```

### koin

https://github.com/InsertKoinIO/koin

```kotlin
// infix fun <T> BeanDefinition<T>.bind(clazz: KClass<*>): BeanDefinition<T>

single<Context> { androidContext } bind Application::classe
```

### mockk

https://mockk.io/

```kotlin
// infix fun Any.invoke(name: String)
// infix fun withArguments(args: List<Any?>)
// infix fun andThen(returnValue: T)

verify { mock invoke "fn" withArguments listOf(5) }
every { doIt() } returns 6 andThen 7
```

### kotest

https://github.com/kotest/kotest

```kotlin
// infix fun String?.shouldNotHaveLength(length: Int): String?
// infix fun String?.shouldHaveMinLength(length: Int): String?

"hello" shouldNotHaveLength 3
"123" shouldHaveMinLength 1
```

## まとめ

次のようなパターンがメジャーな使い方なのかなと思います。

- Groovyなど、他の言語を意識したinfix
- テストなど、自然言語?っぽく書くためのinfix
