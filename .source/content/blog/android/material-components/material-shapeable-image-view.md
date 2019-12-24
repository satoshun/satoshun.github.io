+++
date = "Tue Nov 26 00:30:46 UTC 2019"
title = "Material Components: ShapeableImageViewで丸く切り抜かれた画像を表示する"
tags = ["android", "materialcomponents", "shape", "imageview"]
blogimport = true
type = "post"
draft = true
+++

Material androidの1.2.0-alpha03にShapeableImageViewが爆誕したのでそれの紹介です。

## ShapeableImageView?

読んで字の如くで、shapeに対応したImageViewになります。shapeには丸やひし形などを設定することが可能です。

## 丸を設定してみる

スタイルから定義する場合は、次のようにやります。

```xml
<!-- スタイル定義 -->
<style name="ShapeAppearance.Example.PILL" parent="">
  <item name="cornerFamily">rounded</item>
  <item name="cornerSize">50%</item>
</style>

<!-- レイアウト -->
<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/circle"
  android:layout_width="200dp"
  android:layout_height="200dp"
  android:layout_gravity="center"
  android:layout_margin="10dp"
  android:adjustViewBounds="true"
  android:elevation="4dp"
  app:shapeAppearanceOverlay="@style/ShapeAppearance.Example.PILL"
  app:srcCompat="@drawable/img" />
```

cornerSizeを50%指定すると、ちょうど丸のshapeになります。

{{< figure src="/blog/android/material-components/shapeable-circle.png" width="300" >}}

コードからやる場合は、ShapeAppearanceModelを介して行います。

```kotlin
val model = ShapeAppearanceModel.builder().setAllCornerSizes(ShapeAppearanceModel.PILL).build()
imageView.shapeAppearanceModel = model
```

## ひし形を設定してみる & strokeをつける

スタイルから定義する場合は、次のようにやります。

```xml
<!-- スタイル定義 -->
<style name="ShapeAppearance.Example.Diamond" parent="">
  <item name="cornerFamily">cut</item>
  <item name="cornerSize">50%</item>
</style>

<!-- レイアウト -->
<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/diamond"
  android:layout_width="200dp"
  android:layout_height="200dp"
  android:layout_gravity="center"
  android:layout_margin="10dp"
  android:adjustViewBounds="true"
  android:elevation="4dp"
  app:shapeAppearanceOverlay="@style/ShapeAppearance.Example.Diamond"
  app:srcCompat="@drawable/img"
  app:strokeColor="?attr/colorSecondary"
  app:strokeWidth="2dp" />
```

先ほどと同様に、cornerSizeには50%を指定します。また、cornerFamilyにcutを指定することで、カット、ひし形を作れます。

合わせて、strokeColorとstrokeWidthを指定することでstrokeをつけることが可能です。

{{< figure src="/blog/android/material-components/shapeable-diamond.png" width="300" >}}

## まとめ

今までGlideとかPicassoとかライブラリ側でやっていた処理がこちら側に移るかも!?
