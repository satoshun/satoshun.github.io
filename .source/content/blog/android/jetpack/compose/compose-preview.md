+++
date = "Wed Jul  8 11:36:04 UTC 2020"
title = "Jetpack Compose: PreviewParameterアノテーションを使っていろいろなプレビューを作る"
tags = ["android", "jetpack", "compose"]
blogimport = true
type = "post"
draft = false
+++

この記事では、`PreviewParameter`アノテーションを使って複数のプレビューを出す方法について説明します。

## Previewアノテーションについて

Jetpack Composeでは、`@Preview`を使うことでプレビューを表示することが出来ます。

例えば、TestScreenのプレビューは次のように定義することが出来ます。

```kotlin
@Composable
fun TestScreen(
  user: User,
  count: Int
) {
  ...
}

@Preview("test screen")
@Composable
fun PreviewTestScreen() {
  TestScreen(user = User(id = "1", name = "tom"), count = 10)
}
```

ここで、いろいろなUserインスタンスでプレビューを表示したいとします。愚直にやるなら、`@Preview`を複数定義することですが、`@PreviewParameter`を使うことで少しスマートに書くことが出来ます。

具体的には、次のように書くことが出来ます。

```kotlin
class PreviewUserProvider : PreviewParameterProvider<User> {
  override val values: Sequence<User>
    get() = sequenceOf(
      User(id = "1", name = "tom"),
      User(id = "2", name = "スズキ")
    )
}

@Preview("test screen parameter")
@Composable
fun PreviewParameterTestScreen(
  @PreviewParameter(PreviewUserProvider::class) user: User
) {
  TestScreen(user = user, count = 10)
}
```

最初に、`PreviewParameterProvider`インターフェースを実装します。`PreviewParameterProvider`では、プレビューしたいインスタンス（パラメータ）を定義してあげます。
実装したクラスを `@PreviewParameter(PreviewUserProvider::class)` と指定することで、プレビューを複数出すことが出来ます。

これで、様々なパラメータでプレビューを出すことが出来ました。

## まとめ

XML時代の時は、こういうことは出来なかったと思うので、面白い機能だなと思いました😃
