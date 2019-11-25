+++
date = "Mon Nov 25 13:57:43 UTC 2019"
title = "Material Components: ShapeとBottmSheetDialogとMaterialButton"
tags = ["android", "materialcomponents", "shape"]
blogimport = true
type = "post"
draft = true
+++

Shapeがmaterial androidの1.1.0-alpha01から実装されました。

Shapeとは、こんなやつです。

<img src="/blog/android/material-components/material-shape.png" />

[About shape](https://material.io/design/shape/about-shape.html)

BottomSheetDialogとMaterialButtonを参考に、どのように実装出来るかについて説明します。

## まずはテーマを設定する

`Theme.MaterialComponents`を使う必要があります。

- Theme.MaterialComponents.Light
- Theme.MaterialComponents.DayNight

などをテーマに指定する必要があります。


## BottomSheetDialogにShapeを指定していく

（多分）大きく3つの指定方法があります。

- `shapeAppearanceLargeComponent`を指定する
- `bottomSheetDialogTheme`を指定する
- 独自でテーマを作り、BottomSheetDialogの引数などから与える

### shapeAppearanceLargeComponentを指定する

デフォルトだとBottomSheetDialogのスタイルとして、`@style/Widget.MaterialComponents.BottomSheet.Modal`を使うようになっています。

このスタイルの定義は次のようになっています。

```xml
<style name="Widget.MaterialComponents.BottomSheet.Modal" parent="Widget.MaterialComponents.BottomSheet">
...

<style name="Widget.MaterialComponents.BottomSheet" parent="Widget.Design.BottomSheet.Modal">
  ...
  <item name="shapeAppearance">?attr/shapeAppearanceLargeComponent</item>
  ...
</style>
```

shapeAppearanceに、`?attr/shapeAppearanceLargeComponent`が使われています。なので、これを変えればBottomSheetDialogのShapeを変える事ができます。

たとえば、次のように設定してみます。

```xml
<style name="AppTheme" parent="Theme.MaterialComponents.DayNight">
  <item name="shapeAppearanceLargeComponent">@style/ShapeAppearance.Sample.LargeComponent</item>
</style>

<style name="ShapeAppearance.Sample.LargeComponent" parent="">
  <item name="cornerFamily">cut</item>

  <item name="cornerSize">36dp</item>
</style>
```

こんな感じになります。

<img src="/blog/android/material-components/bottom-sheet-cut.png" />

`cornerFamily`にはcutまたは、roundedを取ることが出来ます。roundedを指定すると次のようになります。

```xml
<style name="ShapeAppearance.Sample.LargeComponent" parent="">
  <item name="cornerFamily">rounded</item>

  <item name="cornerSize">36dp</item>
</style>
```

<img src="/blog/android/material-components/bottom-sheet-rounded.png" />


### bottomSheetDialogThemeを指定する

`bottomSheetDialogTheme`にごりごりと設定をしていきます。

```xml
<style name="AppTheme" parent="Theme.MaterialComponents.DayNight">
  ...
  <item name="bottomSheetDialogTheme">@style/ShapeCustomBottomSheetDialog</item>
</style>

<style name="ShapeCustomBottomSheetDialog" parent="ThemeOverlay.MaterialComponents.BottomSheetDialog">
  <item name="bottomSheetStyle">@style/Widget.Sample.BottomSheetDialog</item>
</style>

<style name="Widget.Sample.BottomSheetDialog" parent="Widget.Design.BottomSheet.Modal">
  <item name="shapeAppearance">@style/Widget.Shape.Basic</item>
</style>

<style name="ShapeAppearance.Sample.Basic" parent="">
  <item name="cornerFamily">rounded</item>

  <item name="cornerSizeTopRight">12dp</item>
  <item name="cornerSizeTopLeft">12dp</item>
</style>
```

こんな感じになります。

<img src="/blog/android/material-components/bottom-sheet-bottomstyle-rounded.png" />

BottomSheetDialogはこんな感じで、Shapeを指定してあげます。


## MaterialButtonにShapeを指定していく

（多分）大きく2つの指定方法があります。

- `shapeAppearanceSmallComponent`を指定する
- 独自でテーマを作り、MaterialButtonの引数、XMLの定義などから与える

### shapeAppearanceSmallComponentを指定する

`MaterialButton`は、デフォルトでは`Widget.MaterialComponents.Button`スタイルを使うようになっています。

これは次のように定義されています。

```xml
<style name="Widget.MaterialComponents.Button" parent="Widget.AppCompat.Button">
  ...
  <item name="shapeAppearance">?attr/shapeAppearanceSmallComponent</item>
</style>
```

shapeAppearanceに、`?attr/shapeAppearanceSmallComponent`が使われています。

次のように設定してみます。

```xml
<style name="AppTheme" parent="Theme.MaterialComponents.DayNight">
  <item name="shapeAppearanceSmallComponent">@style/ShapeAppearance.Sample.SmallComponent</item>
</style>

<style name="ShapeAppearance.Sample.SmallComponent" parent="">
  <item name="cornerFamily">rounded</item>

  <item name="cornerSize">50%</item>
</style>
```

こんな感じになります。

<img src="/blog/android/material-components/bottom-sheet-materialbutton-rounded.png" />

また、直接XMLから指定することも出来ます。

```xml
<style name="ShapeAppearance.Sample.MediumComponent" parent="">
  <item name="cornerFamily">cut</item>

  <item name="cornerSize">16dp</item>
</style>

<com.google.android.material.button.MaterialButton
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:text="Show Dialog"
  app:shapeAppearance="@style/ShapeAppearance.Sample.MediumComponent" />
```

こんな感じになります。

<img src="/blog/android/material-components/bottom-sheet-materialbutton-cut.png" />


## まとめ

- テーマから指定するやり方と、XMLなどから直接指定するやり方がある
    - テーマから指定すると全部一気に変えられるので便利〜

