+++
date = "Sun Jul 12 05:14:48 UTC 2020"
title = "Jetpack Compose: IOSchedをJetpack Composeで書く part1"
tags = ["android", "jetpack", "compose"]
blogimport = true
type = "post"
draft = false
+++

この記事では、Jetpack Composeを学ぶために、公式の [IoSched](https://github.com/google/iosched) のUIをJetpack Composeに書き換えていく記事になります。
完全に見た目を同一にするという所まではやらずに、大体一緒の見た目で妥協するのでご了承下さい。

Part1では、ホーム画面のAppbarを作るところまでをやります。

また、この記事のコードのライセンスはGoogle I/Oアプリと同等です。

```
Copyright 2014 Google Inc. All rights reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

## IOSchedのAppbar

まず最初に、IoSchedのスクショになります。これをComposeに置き換えていきます。

{{< figure src="/blog/android/jetpack/compose/iosched-appbar.png" width="50%" >}}

## Drawer、Menuなどを実装していく

Jetpack Composeでは、`Scaffold`関数を使うことで前述の画像のような、基本的な画面を作ることが出来ます。

```kotlin
@Composable
fun Scaffold(
    scaffoldState: ScaffoldState = remember { ScaffoldState() },
    topBar: @Composable (() -> Unit)? = null,
    bottomBar: @Composable (() -> Unit)? = null,
    floatingActionButton: @Composable (() -> Unit)? = null,
    floatingActionButtonPosition: FabPosition = FabPosition.End,
    isFloatingActionButtonDocked: Boolean = false,
    drawerContent: @Composable (() -> Unit)? = null,
    drawerShape: Shape = MaterialTheme.shapes.large,
    drawerElevation: Dp = DrawerConstants.DefaultElevation,
    backgroundColor: Color = MaterialTheme.colors.background,
    bodyContent: @Composable (InnerPadding) -> Unit
)
```

`Scaffold`関数の定義はこのようになっています。引数を見ると、topBar、bottomBarからAppbarを、floatingActionButtonからFloatingActionButton、drawerContentからDrawerの指定が出来ます。

まずは、topBar引数から、上部にAppbarを設定してみます。Appbarを構築するために`TopAppBar`が定義されているので、これを使います。

```kotlin
Scaffold(
  topBar = {
    TopAppBar(
      ...
    )
  },
  ...
)
```

`TopAppBar`関数を使うと、画面上部にAppbarをレイアウト出来ます。今回はタイトルを`title`引数から、ナビゲーションアイコンを`navigationIcon`引数から、アカウントアイコンを`actions`引数からそれぞれ設定します。


```kotlin
TopAppBar(
  title = { Text(text = "Home") },
  navigationIcon = {
    IconButton(onClick = { TODO() }) {
      Icon(Icons.Filled.Menu)
    }
  },
  actions = {
    IconButton(onClick = { TODO() }) {
      Icon(
        Icons.Outlined.AccountCircle.copy(
          defaultWidth = 36.dp,
          defaultHeight = 36.dp
        )
        tint = Color(0xFF1A73E8)
      )
    }
  },
  backgroundColor = Color.White
)
```

まずは`title`引数から見ていきます。`title`では、`Text`関数に文字列Homeを指定します。`Text`はTextViewのようなものです。文字列を表示する事が出来ます。

```kotlin
Text(text = "Home")
```

次に、`navigationIcon`引数。ここでは、メニューアイコンを出力したいので、`IconButton`関数と`Icon`関数を使います。`IconButton`はonClickからのクリックハンドリングや、レイアウトを調整してくれる機能があります。
`IconButton`関数と`Icon`関数を組み合わせることで、いい感じにアイコンを表示することが出来ます。また、Iconの引数に指定している`Icons.Filled.Menu`はJetpack Compose materialライブラリ側で定義されているアイコンです。基本的なアイコンはライブラリ側で定義されており、今回はそれを使っています。

```kotlin
IconButton(onClick = { TODO() }) {
  Icon(Icons.Filled.Menu)
}
```

次に、`actions`引数。ここでは、TopAppBar上のその他のactionを指定することが出来ます。今回は、右上にアカウントアイコンを表示したいので、`Icon`と`IconButton`を組み合わせて使います。また、IoSchedのアイコンサイズに合わせるために、サイズは36dp、色は#FF1A73E8を指定します。サイズの変更は、`defaultWidth`と`defaultHeight`から設定できます。今回は`36.dp`を設定します。次に、色の変更は`tint`からします。今回は`Color(0xFF1A73E8)`を指定し、青っぽい色に設定します。

```kotlin
IconButton(onClick = { TODO() }) {
  Icon(
    Icons.Outlined.AccountCircle.copy(
      defaultWidth = 36.dp,
      defaultHeight = 36.dp
    )
    tint = Color(0xFF1A73E8)
  )
}
```

最終的に、次のようになります。

{{< figure src="/blog/android/jetpack/compose/iosched-sample-appbar.png" width="50%" >}}

おおよそIoSchedのAppbarと同じ見た目になりました😃

## まとめ

- `Scaffold`を使うと、Appbarなどを設定することが出来る
- `Icons.Filled.Menu`などの、material iconはライブラリ側で定義されている
- `TopAppBar`、`IconButton`、`Icon`などを使うとAppbarを構築出来る

