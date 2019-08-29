+++
date = "Thu Aug 29 00:39:47 UTC 2019"
title = "ViewPager2でカルーセルっぽいものを実装する"
tags = ["android", "viewpager2", "jetpack"]
blogimport = true
type = "post"
draft = false
+++

横のアイテムをチラ見せする画面をViewPager2で書けるよって話です。

最終的にこんなのが作れます。

<img src="/blog/android/jetpack/viewpager2/viewpager2-carousel.gif" style="max-width:280px" />

では、コードを説明していきます。

## 1. 横のアイテムをチラ見せするだけ

まずは横のアイテムをどれだけチラ見せるかを定義します。

- `<dimen name="offset">32dp</dimen>`

次に、アイテム間にどれだけのマージンをつけるかを定義します。

- `<dimen name="pageMargin">16dp</dimen>`

## 2. 横のアイテムをチラ見せしてScaleも変更する
