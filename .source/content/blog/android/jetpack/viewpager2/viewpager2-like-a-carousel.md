+++
date = "Thu Aug 29 13:29:06 UTC 2019"
title = "ViewPager2でカルーセルっぽいものを実装する"
tags = ["android", "viewpager2", "jetpack"]
blogimport = true
type = "post"
draft = false
+++

横のアイテムをチラ見せする画面をViewPager2で書けるよって話です。

最終的にこんなのが作れます。

{{< figure src="/blog/android/jetpack/viewpager2/viewpager2-carousel.gif" style="max-width:280px" >}}

では、コードを説明していきます。

## 1. 横のアイテムをチラ見せする

まず、ViewPager2をLayout内に定義します。

```xml
<androidx.viewpager2.widget.ViewPager2
  android:id="@+id/viewpager"
  android:layout_width="match_parent"
  android:layout_height="200dp"
  android:orientation="horizontal"
  />
```

次に、横のアイテムをどれだけチラ見せるかを定義します。

```xml
<dimen name="offset">32dp</dimen>
```

次に、各アイテム間にどれだけのマージンをつけるかを定義します。

```xml
<dimen name="page_margin">16dp</dimen>
```

次に、ViewPager2内で使うレイアウトのトップレベルのViewにmarginLeft、Rightをセットします。

```xml
<com.google.android.material.card.MaterialCardView
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:layout_marginLeft="48dp" // これは、offset + page_marginの値
  android:layout_marginRight="48dp"
  app:cardCornerRadius="16dp">
  ...
```

marginには前で定義したoffset、page_marginを足し合わせたものを指定します。

そして、最後に`PageTransformer`を設定します。

```kotlin
viewpager.offscreenPageLimit = 2 // これは左右のアイテムを描画するために必要

val pageMarginPx = root.context.resources.getDimensionPixelOffset(R.dimen.page_margin)
val offsetPx = root.context.resources.getDimensionPixelOffset(R.dimen.offset)
viewpager.setPageTransformer { page, position ->
  val offset = position * (2 * offsetPx + pageMarginPx)
  page.translationX = -offset
}
```

PageTransformerはスクロールに応じてコールバックされる関数になります。
この中で、`translationX`をpositionの値に応じて動かすことで、左右のアイテムのチラ見せを実現出来ます。

## 2. さらにScaleも変更する

前述したチラ見せPageTransformerに加え、Scale用のPageTransformerも作ります。

```kotlin
viewpager.setPageTransformer(CompositePageTransformer().apply {
  addTransformer { page, position ->
    val offset = position * (2 * offsetPx + pageMarginPx)
    page.translationX = -offset
  }

  addTransformer { page, position ->
    // 6は適当な値です
    val scale = 1 - (abs(position) / 6)
    page.scaleX = scale
    page.scaleY = scale
  }
})
```

`CompositePageTransformer`を使うことで、複数のPageTransformerをセットすることが出来ます。
前で作ったチラ見せPageTransformerに加え、新しくScale用のPageTransformerを定義します。

Scale用のPageTransformer内では、`scale`の値をpositionの値に応じて動かすことで、Scaleアニメーションを実現出来ます。

## まとめ

- ViewPager2便利だし、もうbetaなので積極的に使っていっても良いのでは😃
- 実際のコードは [satoshun-android/ViewPager2](https://github.com/satoshun-android-example/ViewPager2)にあります。

## 参考

- [Look Deep Into ViewPager2](https://proandroiddev.com/look-deep-into-viewpager2-13eb8e06e419)