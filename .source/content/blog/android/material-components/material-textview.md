+++
date = "Mon Jul 15 03:09:15 UTC 2019"
title = "Material Components: MaterialTextViewでlineHeightがTextAppearanceから指定出来るようになりました"
tags = ["android", "materialcomponents", "text"]
blogimport = true
type = "post"
draft = false
+++

従来のTextView（AppCompatTextView）では、lineHeightの指定をTextAppearanceから出来ませんでした。

それが、[material-component 1.1.0-alpha08](https://github.com/material-components/material-components-android/releases/tag/1.1.0-alpha08)にMaterialTextViewが爆誕し、lineHeightがTextAppearanceから指定出来るようになりました🎉

## 使い方

まずはstyleを定義します。

```xml
<style name="TextAppearance.LineHeight">
  <item name="lineHeight">20sp</item>
</style>
```

次に、TextViewから指定します。

```xml
<TextView
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:text="テスト"
  android:textAppearance="@style/TextAppearance.LineHeight" />
```

これで完了です😃

MaterialTextViewをXMLから直接指定してもいいのですが、AppCompatActivityを使っていれば、自動的にMaterialTextViewがinflateされるようになっています。
詳しくは[MaterialComponentsViewInflater.java](https://github.com/material-components/material-components-android/commit/d7a92485f818a63d110536d70e7040b1eadfb3aa#diff-b6219f955dbe45d238cdd3b1b43f46b8R103)をご覧下さい。

## まとめ

- material-component 1.1.0-alpha08にアップデートすると、自動的にMaterialTextViewが使われ、TextAppearanceからlineHeightが指定できる😃
