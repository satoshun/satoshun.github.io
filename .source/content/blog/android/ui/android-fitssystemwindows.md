+++
date = "Sun Jan 26 04:40:11 UTC 2020"
title = "fitsSystemWindowsの話をつらつらと"
tags = ["android", "edgetoedge", "ui"]
blogimport = true
type = "post"
draft = false
+++

fitsSystemWindowsについてマスターしつつあるので、つらつらと学んだことをまとめておきます。

SystemUiVisibilityの詳細な設定については説明を割愛するのでご了承ください。

## そもそもfitsSystemWindowsとは?

Android端末には、status bar、navigation barなどのSystem UIと総称されるViewがあります。
デフォルトでは、System UIにコンテンツの要素が被ることはありません。そこには制約があります。
しかし、SystemUiVisibilityの設定を変えることで、コンテンツの要素をSystem UIの裏側描くことが可能になります。

---

{{< figure src="/blog/android/ui/merge.png" width="600">}}

---

右の図がSystemUiVisibilityの設定を変更したものです。画像がstatus bar、navigation barの背後に描画されていることが分かります。

ここからが本題です。上記の画像の場合は、画像をめいいっぱいに広げて表示しても違和感がありません。しかし、AppBarLayoutといったToolbarの場合はどうなるでしょうか?

---

{{< figure src="/blog/android/ui/merge2.png" width="600">}}

---

右の図のAppBarLayoutが、status barに食い込んでしまっていることが分かります。SystemUiVisibilityの設定を変えている場合、status barの高さを考慮する必要があることが分かります。

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

見た目が元に戻りました。fitsSystemWindowsをつけると、どのViewの要素が変化するかをLayout Inspectorで確認します。

---

{{< figure src="/blog/android/ui/fits1-layoutinspector.png" width="300">}}

---

paddingTopと、paddingBottomに値が指定されていることが分かります。これはfitsSystemWindowsのデフォルトの振る舞いによるものです。

fitsSystemWindowsはデフォルトで、paddingTopにstatus barの高さを、paddingBottomにnavigation barの高さを設定します。それにより、コンテンツがSystem UIに被らないようになります。

まとめると「fitsSystemWindowsはSystem UIの高さに応じてpaddingに値をセットする」振る舞いをします。

しかし、これはデフォルトの動作で、fitsSystemWindowsの動作を変更することが出来ます。

AppBarLayoutはfitsSystemWindowsのデフォルトの動作を変更しているViewなので、AppBarLayoutにfitsSystemWindowsをつけた場合にどのように動作するかを見てみます。

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

見た目はおかしくないのですが、paddingが変更されていません。じゃあどこが変わったかというと、heightの値が変わっています。

---

{{< figure src="/blog/android/ui/fits1-appbarlayout-layoutinspector2.png" width="300">}}

---

省略するのですが、fitsSystemWindowsを指定しない場合のAppBarLayoutの高さは154でした。上記の図の220という数字は 154(元の高さ) + 66(status barの高さ) = 220です。

AppBarLayoutはpaddingTopを設定するのではなく、高さを調整することでSystem UI上の見た目を調整しています。

このように、Viewによってはカスタムの振る舞いを提供しています。他には有名どころだと、CoordinatorLayout、DrawerLayoutも特別な振る舞いを提供しています。

最後にどのようにして、カスタムの振る舞いを実装するかを説明します。

## fitsSystemWindowsのカスタマイズ

View.setOnApplyWindowInsetsListenerから、OnApplyWindowInsetsListenerを設定しておくと、デフォルトのfitsSystemWindowsの振る舞いではなく、設定したOnApplyWindowInsetsListenerのほうがコールバックされます。

OnApplyWindowInsetsListenerには、WindowInsetsが渡ってきます。この中にはSystem UIのサイズと、WindowInsetsが消費されたかどうかを表すフラグがあります。

例えば、status barの2倍のサイズのpaddingTopを設定して、これ以降のViewにSystem WindowInsetsを渡したくないときは次のようにします。

```kotlin
binding.appbar.setOnApplyWindowInsetsListener { v, insets ->
  binding.appbar.updatePadding(top = insets.systemWindowInsetTop * 2)
  insets.consumeSystemWindowInsets()
}
```

このAPIを使うことで、fitsSystemWindowsよりもはるかに細かくSystem UIを制御することが出来ます。

CoordinatorLayoutなどのViewもこのAPIを使って細かくWindowInsetsを制御をしています。

## 個人的なハマりどころ

### fitsSystemWindowsは複数つけても意味がない

これはデフォルトの場合の挙動なのですが、fitsSystemWindowsはWindowInsetsを消費するので、一度でも使ってしまうとそれ以降のViewに対して影響を与えません。

```xml
<LinearLayou android:fitsSystemWindows="true">
  <ImageView android:fitsSystemWindows="true" />
  ...
</LinearLayout>
```

なので、この場合ImageViewにつけたfitsSystemWindowsは意味がないです。親のLinearLayoutで既に消費されているためです。

しかし、AppBarLayoutなどの特殊な振る舞いをするViewの場合は、子供のfitsSystemWindowsに意味があったりするので注意が必要です。

また、ViewによってSystem WindowInsetsを消費したり、しなかったりするのでそこも注意が必要です。DrawerLayoutは消費し、AppBarLayoutは消費しません。
既に、消費済みの場合、OnApplyWindowInsetsListenerを登録してもコールされないので注意してください。

## まとめ

- fitsSystemWindowsはデフォルトでpaddingTop、paddingBottomをセットする
- 動作をカスタムしている場合は注意が必要
  - WindowInsetsを消費したり、しなかったりする
  - しかし、大抵はいい感じにやってくれるので特に意識しなくて良いと思う
- 自分でカスタムしたい場合は、setOnApplyWindowInsetsListenerを使って頑張る
