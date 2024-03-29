+++
date = "Sun Mar  6 12:12:41 UTC 2022"
title = "Jetpack Compose: debugInspectorInfoを使って、デバッグを少し楽にする"
tags = ["android", "jetpack", "compose"]
blogimport = true
type = "post"
draft = false
+++

Android Studioで、[Layout Inspector](https://developer.android.com/studio/debug/layout-inspector) を使うことで、レイアウトの配置や、実際にどのような値がセットされているかの詳細を見る事ができます。

Jetpack Composeでは、そのLayout Inspectorに情報を渡すことができます。

どのように渡すかを `Modifier.border` を例にして説明します。まず、`Modifier.border`を次のように使ってみます。

```kotlin
Surface(
    modifier = Modifier
      .border(width = 2.dp, color = Color.Cyan)
) { ... }
```

そして、Layout Inspectorで、このSurfaceの中身を見てみます。

{{< figure src="/blog/android/jetpack/compose/inspector-result.png" >}}

modifierの中に、borderがあり、詳細な情報があることが分かります。

では次に、`Modifier.border` ではどのようにして、Layout Inspectorに値をセットしているかを見ていきます。

```kotlin
fun Modifier.border(width: Dp, brush: Brush, shape: Shape): Modifier = composed(
    factory = { ... },
    inspectorInfo = debugInspectorInfo {
        name = "border"
        properties["width"] = width
        if (brush is SolidColor) {
            properties["color"] = brush.value
            value = brush.value
        } else {
            properties["brush"] = brush
        }
        properties["shape"] = shape
    }
)
```

`composed`メソッドの、inspectorInfo引数から、`debugInspectorInfo`メソッドを介してなんか色々やっていることが分かります。
`debugInspectorInfo`メソッドは、`InspectorInfo`をレシーバーとして受け取ることができ、`InspectorInfo`を介してnameとpropertiesをセットすることができます。コードを見ると、nameに"border"、propertiesに"width"などをセットしていることが分かります。
これを実行すると、上の画像のようにLayout Inspector上に情報を出力することができます。

## まとめ

Modifierを拡張するようなメソッドを書くときは、Layout Inspectorへ値を渡すことで、少しデバッグが容易になるかも。
