+++
date = "Tue Nov 26 00:30:46 UTC 2019"
title = "Material Components: ShapeとBottomSheetDialogとMaterialButton"
tags = ["android", "materialcomponents", "shape"]
blogimport = true
type = "post"
draft = false
thumbnail = "/blog/android/material-components/material-shape.png"
+++

Shapeがmaterial androidの1.1.0-alpha01から実装されました。

Shapeとは、こんなやつです。

<img src="/blog/android/material-components/material-shape.png" />

参照: [About shape](https://material.io/design/shape/about-shape.html)

この記事では、BottomSheetDialogとMaterialButtonを参考に、Shapeをどのように設定するかを説明します。

より詳しい説明は[公式ドキュメント/Shape.md](https://github.com/material-components/material-components-android/blob/master/docs/theming/Shape.md)を参照してください。

## まずはテーマを設定する

`Theme.MaterialComponents`を使う必要があります。

- Theme.MaterialComponents.Light
- Theme.MaterialComponents.DayNight

などをテーマに指定します。


```xml
<style name="AppTheme" parent="Theme.MaterialComponents.DayNight">
...
```

こんな感じです。

準備は終わったので、BottomSheetDialogにShapeを適用してみます。

## BottomSheetDialogにShapeを指定していく

（多分）大きく3つの指定方法があります。

- `shapeAppearanceLargeComponent`を指定する
- `bottomSheetDialogTheme`を指定する
- 独自でテーマを作り、BottomSheetDialogの引数などから与える

### shapeAppearanceLargeComponentを指定する

デフォルトだとBottomSheetDialogのスタイルとして、`@style/Widget.MaterialComponents.BottomSheet.Modal`を使うようになっています。

このスタイルの定義は、次のようになっています。

```xml
<style name="Widget.MaterialComponents.BottomSheet.Modal" parent="Widget.MaterialComponents.BottomSheet">
...

<style name="Widget.MaterialComponents.BottomSheet" parent="Widget.Design.BottomSheet.Modal">
  ...
  <item name="shapeAppearance">?attr/shapeAppearanceLargeComponent</item>
  ...
</style>
```

shapeAppearanceに、`?attr/shapeAppearanceLargeComponent`が使われています。`shapeAppearance`は、Shapeの設定を流し込む部分です。
なので、これを変えればBottomSheetDialogのShapeを変える事ができます。

たとえば、次のように設定してみます。

```xml
<style name="AppTheme" parent="Theme.MaterialComponents.DayNight">
  <!-- ?attr/shapeAppearanceLargeComponentの設定 -->
  <item name="shapeAppearanceLargeComponent">@style/ShapeAppearance.Sample.LargeComponent</item>
</style>

<style name="ShapeAppearance.Sample.LargeComponent" parent="">
  <item name="cornerFamily">cut</item>

  <!-- 各corner(top right, top left, bottom left, bottom right)のサイズを指定 -->
  <item name="cornerSize">36dp</item>
</style>
```

こんな感じになります。

<img src="/blog/android/material-components/bottom-sheet-cut.png" width="300" />

`cornerFamily`にはcut以外にも、roundedを取ることが出来ます。roundedを指定すると次のようになります。

```xml
<style name="ShapeAppearance.Sample.LargeComponent" parent="">
  <item name="cornerFamily">rounded</item>

  <item name="cornerSize">36dp</item>
</style>
```

<img src="/blog/android/material-components/bottom-sheet-rounded.png" width="300" />

`shapeAppearanceLargeComponent`を指定することは、BottomSheetDialog以外の、他のコンポーネントにも影響が出るので、注意してください。

### bottomSheetDialogThemeを指定する

`bottomSheetDialogTheme`から、BottomSheetDialogのShapeの設定を変えることも出来ます。

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

  <!-- cornerは、個別に指定することも可能 -->
  <item name="cornerSizeTopRight">12dp</item>
  <item name="cornerSizeTopLeft">12dp</item>
</style>
```

こんな感じになります。

<img src="/blog/android/material-components/bottom-sheet-bottomstyle-rounded.png" width="300" />

BottomSheetDialogはこんな感じで、StyleからShapeを指定することが出来ます。


## MaterialButtonにShapeを指定していく

（多分）大きく2つの指定方法があります。

- `shapeAppearanceSmallComponent`を指定する
- Shape用のスタイルを作り、MaterialButtonの引数、XMLの定義などから与える

### shapeAppearanceSmallComponentを指定する

`MaterialButton`は、デフォルトでは`Widget.MaterialComponents.Button`スタイルを使うようになっています。

これは、次のように定義されています。

```xml
<style name="Widget.MaterialComponents.Button" parent="Widget.AppCompat.Button">
  ...
  <item name="shapeAppearance">?attr/shapeAppearanceSmallComponent</item>
</style>
```

shapeAppearanceに、`?attr/shapeAppearanceSmallComponent`が使われています。先程と同様に、これを指定することで、Shapeの振る舞いを変えることが出来ます。

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

<img src="/blog/android/material-components/bottom-sheet-materialbutton-rounded.png" width="300" />

### XMLから直接指定する

次のように、直接XMLから指定することも出来ます。

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

<img src="/blog/android/material-components/bottom-sheet-materialbutton-cut.png" width="300" />


## まとめ

- テーマから指定するやり方と、XMLなどから直接指定するやり方がある
- テーマから指定すると全部一気に変えられるので便利😃
    - ただし、`?attr/shapeAppearanceSmallComponent`を上書きする方法だと、他のコンポーネントにも影響が出るので、慎重に
        - 例えば、Chipはデフォルトだと`shapeAppearanceSmallComponent`で定義されています
