+++
date = "Wed Jul  3 00:17:15 UTC 2019"
title = "Android: RestrictToアノテーションのIDE上の振る舞いについて"
tags = ["android", "jetpack", "annotation"]
blogimport = true
type = "post"
draft = true
+++

`androidx.annotation:annotation`には、`RestrictTo`アノテーションクラスが定義されています。
これは、次の用途を持ちます。

> Denotes that the annotated element should only be accessed from within a specific scope (as defined by Scope).

指定したScope内のみからのアクセスを許可するアノテーションになっています。

この記事では、この`RestrictTo`アノテーションがついたクラスに様々な場所からアクセスしたときに、どのようにIDE上で警告が出るかについて見ていきます。

また、Android Studio 3.5.0-beta05で検証しました。

## Scopeの一覧

RestrictToの定義は以下のようになっています。（コメントやらなんやらは削除してます。）

```java
public @interface RestrictTo {
    Scope[] value();

    enum Scope {
        LIBRARY,
        LIBRARY_GROUP,
        LIBRARY_GROUP_PREFIX,
        @Deprecated
        GROUP_ID,
        TESTS,
        SUBCLASSES,
    }
}
```

GROUP_IDはdeprecatedとのことなので、それ以外のLIBRARY、 LIBRARY_GROUP、 LIBRARY_GROUP_PREFIX、 TESTS、 SUBCLASSESの挙動について見ていきます。

## 同一モジュール内

最初に、呼び出される側の定義です。

```kotlin
object Hoge {
  @RestrictTo(RestrictTo.Scope.LIBRARY)
  fun library() {}

  @RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
  fun libraryGroup() {}

  @RestrictTo(RestrictTo.Scope.LIBRARY_GROUP_PREFIX)
  fun libraryGroupPrefix() {}

  @RestrictTo(RestrictTo.Scope.SUBCLASSES)
  fun subclasses() {}

  @RestrictTo(RestrictTo.Scope.TESTS)
  fun tests() {}
}
```

次に呼び出し側です。

アプリケーションコードから呼び出してみます。

```kotlin
Hoge.library() // OK
Hoge.libraryGroup() // OK
Hoge.libraryGroupPrefix() // OK
Hoge.subclasses() // NG
Hoge.tests() // NG
```

SUBCLASSESと、TESTSスコープを付けたときはIDEに怒られました。サブクラスではないし、テストからの呼び出しでも無いためです。

次に、テストコードから呼び出してみます。

```kotlin
Hoge.library() // OK
Hoge.libraryGroup() // OK
Hoge.libraryGroupPrefix() // OK
Hoge.subclasses() // OK
Hoge.tests() // OK
```

SUBCLASSESスコープのときはNGだと思ったんですけど、全部OKでした。テストのときはlintって走らないんですかね？（無知）
