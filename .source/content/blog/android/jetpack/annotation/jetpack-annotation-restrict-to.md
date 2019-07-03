+++
date = "Wed Jul  3 12:30:11 UTC 2019"
lastmod = "Wed Jul  3 13:23:44 UTC 2019"
title = "Android: RestrictToアノテーションのIDE上での振る舞い"
tags = ["android", "jetpack", "annotation"]
blogimport = true
type = "post"
draft = false
+++

`androidx.annotation:annotation`には、`RestrictTo`アノテーションクラスが定義されています。
このアノテーションは次の用途を持ちます。

> Denotes that the annotated element should only be accessed from within a specific scope (as defined by Scope).

指定したScope以外からのアクセスを制限するアノテーションです。

この記事では、この`RestrictTo`アノテーションがついたクラスに様々な場所からアクセスしたときに、どのようにIDE上で警告が出るかについて見ていきます。

また、Android Studio 3.5.0-beta05で検証しました。

この記事内に出てくるRestrictTo関連のコードは以下のライセンスに従います。

```java
/*
 * Copyright (C) 2016 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
```

今回の検証に用いたコードは [satoshun/RestrictTo](https://github.com/satoshun-android-example/RestrictTo) にあります。

## Scopeの一覧

RestrictToの定義は以下のようになっています。（コメント等は削除してます。）

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

## 各スコープについて

次のように説明があります。

```java
/**
  * Restrict usage to code within the same library (e.g. the same
  * gradle group ID and artifact ID).
  */
LIBRARY,

/**
  * Restrict usage to code within the same group of libraries.
  * This corresponds to the gradle group ID.
  */
LIBRARY_GROUP,

/**
  * Restrict usage to code within packages whose groups share
  * the same library group prefix up to the last ".", so for
  * example libraries foo.bar:lib1 amd foo.baz:lib2 share
  * the prefix "foo." and so they can use each other's
  * apis that are restricted to this scope. Similarly for
  * com.foo.bar:lib1 and com.foo.baz:lib2 where they share
  * "com.foo.". Library com.bar.qux:lib3 however will not
  * be able to use the restricted api because it only
  * shares the prefix "com." and not all the way until the
  * last ".".
  */
LIBRARY_GROUP_PREFIX,

/**
  * Restrict usage to tests.
  */
TESTS,

/**
  * Restrict usage to subclasses of the enclosing class.
  * <p>
  * <strong>Note:</strong> This scope should not be used to annotate
  * packages.
  */
SUBCLASSES,
```

名前から何となく想起できると思います。LIBRARY系のスコープはやや複雑に感じました。

次から、実際の挙動を見ていきます。

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

適当にスコープをつけたメソッドを定義しただけです。

次に呼び出し側です。

まずは、アプリケーションコードから呼び出してみます。

```kotlin
Hoge.library() // OK
Hoge.libraryGroup() // OK
Hoge.libraryGroupPrefix() // OK
Hoge.subclasses() // NG
Hoge.tests() // NG
```

SUBCLASSESと、TESTSスコープを付けたときはIDEに怒られました。サブクラスではないし、テストからの呼び出しでも無いので、納得出来ます。

次に、テストコードから呼び出しになります。

```kotlin
Hoge.library() // OK
Hoge.libraryGroup() // OK
Hoge.libraryGroupPrefix() // OK
Hoge.subclasses() // OK
Hoge.tests() // OK
```

SUBCLASSESスコープのときはNGだと思ったんですけど、全部OKでした。テストのときはlintって走らないんですかね？（無知）

## 同一プロジェクト内、ライブラリモジュール

次に、lib1という名前でライブラリモジュールを作り、そこで定義したクラスに、appモジュールからアクセスしてみます。

settings.gradleは次のようになります。

```
include ':app'
include ':lib1'
```

まず、lib1に次のクラスを定義します。

```kotlin
object Hoge2 {
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

次に、appモジュールからアクセスしてみます。

```kotlin
Hoge2.library() // OK
Hoge2.libraryGroup() // OK
Hoge2.libraryGroupPrefix() // OK
Hoge2.subclasses() // NG
Hoge2.tests() // NG
```

LIBRARY、LIBRARY_GROUP、LIBRARY_GROUP_PREFIXでOKになりました。現状だと同一プロジェクト内に各モジュールが定義されている場合、これらのスコープは特に考慮されなそうでした。

## サードパーティライブラリの場合

androidx内で定義されているスコープが付いたクラスに、appモジュールからアクセスしてみます。

まずは、ライブラリモジュール内の定義から。

```java
@RestrictTo(LIBRARY)
public class ContentFrameLayout extends FrameLayout

---

public class MediaItem {
  @RestrictTo(LIBRARY_GROUP)
  public static class Builder
}

---

@RestrictTo(LIBRARY_GROUP_PREFIX)
public class DrawableWrapper extends Drawable implements Drawable.Callback
```

次に、appから呼び出してみます。

```kotlin
ContentFrameLayout(...) // NG
MediaItem.Builder().build() // NG
DrawableWrapper(...) // OK
```

LIBRARY、LIBRARY_GROUPスコープがついたクラスに、アクセスするとIDEに怒られました。これは想像つくと思います。
しかし、LIBRARY_GROUP_PREFIXは怒られませんでした。これは多分間違った挙動だと思うのでいつか直ると思います。

## まとめ

- ざっくりと挙動を見ていきました。
  - ライブラリ開発をしているときはLIBRARY**系のスコープを意識すると良さそう
  - アプリケーション開発時は、TESTS、SUBCLASSESを必要に応じてつける感じで
- LIBRARY_GROUP_PREFIXは、おそらくマルチモジュール環境で有効に使うことが可能だと思うので、今後に期待
- なぜこのような挙動になるか、などの細かい部分は実際のRestrictToクラスのコメントを見ていただけると幸いでございます
- 今回の検証に用いたコードは [satoshun/RestrictTo](https://github.com/satoshun-android-example/RestrictTo) にあります。
