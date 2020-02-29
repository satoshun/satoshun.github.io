+++
date = "Sat Feb 29 23:32:01 UTC 2020"
title = "Jetpack Compose: Modelとstate"
tags = ["android", "jetpack", "compose"]
blogimport = true
type = "post"
draft = false
+++

Jetpack Composeのデータ監視方法について紹介します。

## state

まず、stateメソッドの紹介をします。

stateメソッドを使うと値を監視することができ、値が変更されたときに自動で再構成（recomposition）してくれます。

次のように、Viewの状態を表すのに便利に使えます。

```kotlin
@Composable
fun MyCheckbox() {
  // 初期値false
  var checked by state { false }

  Checkbox(
    checked = checked,
    onCheckedChange = {
      // stateで定義した値を更新すると、自動でUIの再構成（recomposition)が走る
      checked = it
    }
  )
}
```

`state { 初期値 }`って感じで定義して、その値を更新するとUIの再構成をしてくれます。

この例の場合、Checkboxがクリックされると、checkedの状態が変わり、Viewが再構成されます。

## Model

次にModelです。Modelはアノテーションで定義されています。
Modelアノテーションをつけたクラスのプロパティが監視対象になり、プロパティが更新されたときに自動で再構成してくれます。

例えば、クリックされたカウントを保持するModelは次のように作ります。

```kotlin
@Model
class Count(
  var count: Int = 0
)

@Composable
fun MyText() {
  // Modelの定義。引数から渡すこともある
  val count = Count()

  Ripple(bounded = false) {
    Clickable(onClick = {
      // Modelの値を更新すると、自動で再構成（recomposition)が走る
      count.count += 1
    }) {
      Container {
        // 構成、再構成が走るとtextが更新される
        Text(text = count.count.toString())
      }
    }
  }
}
```

こんな感じで使います。

Modelのプロパティ、今回の場合はcountプロパティの値が変わると、自動で再構成されます。

## まとめ

値を変更するだけで、勝手にUIを再構成してくれるので便利〜

- [state](https://developer.android.com/reference/kotlin/androidx/compose/package-summary#state(kotlin.Function2,%20kotlin.Function0))
- [Model](https://developer.android.com/reference/kotlin/androidx/compose/Model)
