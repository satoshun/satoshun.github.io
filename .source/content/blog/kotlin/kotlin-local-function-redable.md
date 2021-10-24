+++
date = "Sun Oct 24 07:59:14 UTC 2021"
title = "Kotlin: ローカル関数を使って読みやすいコードを書きたい"
tags = ["kotlin"]
blogimport = true
type = "post"
draft = false
+++

Kotlinには、ローカルスコープで関数を作る機能があります。

```kotlin
suspend fun getUser(userId: String): User {
  suspend fun getUserFromRemote(): User {
    return ...
  }

  val user = getUserFromRemote()
  if (user.isInvalid) throw ...
  ...
}
```

---

いわゆる説明変数の代わりにローカル関数を使えば、いい感じになるケースがあるのではと思っています。

説明変数バージョン

```kotlin
fun getUser(userName: String): User {
  val nickName = userName.split(...)
  if (nickName == userName) ...
  else ...
}
```

ローカル関数バージョン

```kotlin
fun getUser(userName: String): User {
  fun getNickName(): String {
    return userName.split(...)
  }

  if (getNickName() == userName) ...
  else ...
}
```

ローカル関数を使うメリットとしては、処理がlazyになる、必要になったタイミングで処理が行われる点と、
ロジックをより適切な場所に切り出すヒントになる点だと思います。

ロジックをより適切な場所に切り出すヒントというのは、ローカル関数として定義するということは、ロジックないし、何かしらのめんどうな処理を持っていることを意味するので、
説明変数のバージョンに比べ、より適切な場所があるのではという匂いを出すことが出来ると思います。(諸説あり。主観入ってます)
