+++
date = "Sun Jan 26 02:11:55 UTC 2020"
title = "fitsSystemWindowsの話をつらつらと"
tags = ["android", "edgetoedge"]
blogimport = true
type = "post"
draft = false
+++

fitsSystemWindowsについてマスターしつつあるので、つらつらと学んだことをまとめておきます。

SystemUiVisibilityの詳細な設定については説明を割愛するのでご了承ください。

## そもそもfitsSystemWindowsとは?

Android端末には、status bar、navigation barなどのSystem UIと総称されるViewがあります。
デフォルトでは、System UIにコンテンツの要素が被ることはありません。そこには制約があります。
しかし、SystemUiVisibilityの設定を変えることで、コンテンツの要素を、System UIに裏側描くことが可能になります。

---

{{< figure src="/blog/android/ui/merge.png" width="600">}}

---

右の図がSystemUiVisibilityの設定を変更したものです。画像がstatus bar、navigation barの背後に描画されていることが分かります。

ここからが本題です。上記の画像の場合は、画像をめいいっぱいに広げて表示しても違和感がありません。しかし、AppbarLayoutといったToolbarの場合はどうなるでしょうか?

---

{{< figure src="/blog/android/ui/merge2.png" width="600">}}

---

右の図のAppbarLayoutが、status barに食い込んでしまっていることが分かります。SystemUiVisibilityの設定を変えている場合、上記の例の場合、status barの高さを考慮しなければいけないことが分かります。

このときに、fitsSystemWindowsを使うとSystem UIに被らないようにコンテンツを配置することが出来ます。

```xml
<LinearLayout android:fitsSystemWindows="true">
  <com.google.android.material.appbar.AppBarLayout />
  ...
</LinearLayout>
```

---

{{< figure src="/blog/android/ui/fits1.png" width="300">}}

---

fitsSystemWindowsをつけると、Viewのどの要素が変化するかをLayout Inspectorで確認してみます。

---

{{< figure src="/blog/android/ui/fits1-layoutinspector.png" width="300">}}

---

paddingTopと、paddingBottomに値が指定されていることが分かります。これはfitsSystemWindowsのデフォルトの振る舞いによるものです。

fitsSystemWindowsはデフォルトで、paddingTopにstatus barの高さを、paddingBottomにnavigation barの高さを設定します。それにより、コンテンツがSystem UIに被らないようになります。

---

「fitsSystemWindowsはpaddingをSystem UIの高さに応じてセットする」動作をします。

しかし、これはデフォルトの動作で、fitsSystemWindowsは動作を変更することが出来ます。

AppbarLayoutはfitsSystemWindowsのデフォルトの動作を変更しているので、AppbarLayoutにfitsSystemWindowsをつけた場合にどのように動作が変わるかを見てみます。

## AppBarLayoutのfitsSystemWindowsの解釈

fitsSystemWindowsをAppBarLayoutに設定します。

```xml
<LinearLayout>
  <com.google.android.material.appbar.AppBarLayout
    android:fitsSystemWindows="true" />
  ...
</LinearLayout>
```

---

{{< figure src="/blog/android/ui/fits1-appbarlayout.png" width="300">}}

{{< figure src="/blog/android/ui/fits1-appbarlayout-layoutinspector.png" width="300">}}

---

見た目はおかしくないのですが、paddingが変更されていません。じゃあ何が変わったかというと、heightが変わっています。

---

{{< figure src="/blog/android/ui/fits1-appbarlayout-layoutinspector2.png" width="300">}}

---

省略するのですが、fitsSystemWindowsを指定しない場合のAppbarLayoutの高さは154でした。220という数字は 154(元の高さ) + 66(status barの高さ) = 220です。

AppbarLayoutはpaddingTopを設定するのではなく、高さを調整することで見た目を調整しています。

このように、Viewによってはカスタムの振る舞いを提供しています。他には有名どころだと、CoordinatorLayout、DrawerLayoutも特別な振る舞いを提供しています。

最後にどのようにして、カスタマイズするかを説明します。

## fitsSystemWindowsのカスタマイズ

View.setOnApplyWindowInsetsListenerから、OnApplyWindowInsetsListenerを設定しておくと、デフォルトのfitsSystemWindowsではなく、OnApplyWindowInsetsListenerのほうがコールバックされます。

OnApplyWindowInsetsListenerには、WindowInsetsが渡ってきます。この中にはSystem UIのサイズが入っています。

例えば、status barの2倍のサイズのpaddingTopを設定したいときは次のようにします。

```kotlin
binding.appbar.setOnApplyWindowInsetsListener { v, insets ->
  binding.appbar.updatePadding(top = insets.systemWindowInsetTop * 2)
  insets
}
```
