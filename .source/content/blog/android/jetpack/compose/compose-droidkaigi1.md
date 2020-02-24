+++
date = "Mon Feb 24 11:11:36 UTC 2020"
title = "Jetpack ComposeでDroidKaigiアプリを作ってみる part1"
tags = ["android", "jetpack", "compose"]
blogimport = true
type = "post"
draft = true
+++

最近、Composeの勉強を始めたので、手始めに[DroidKaigi2020](https://github.com/DroidKaigi/conference-app-2020)を置き換えてみたいと思います。

- まずはHome(Timeline)から置き換えていきます。データの扱い方がまるで分かってないので、最初は固定のデータを表示するに留めています
- バージョンはdev05を使ってます

Timelineはこんな感じです。

{{< figure src="/blog/android/jetpack/compose/timeline.png" width="50%" >}}

上から順番に置き換えていきます。

## Drawerの作成

横からニュッと出てくるDrawerを作るには`ModalDrawerLayout`を使います。
