+++
date = "Tue Apr  7 00:41:08 UTC 2020"
title = "ちょっと複雑なレイアウトをConstraintLayoutで組むpart1"
tags = ["android", "constraintlayout"]
blogimport = true
type = "post"
draft = false
+++

今回の記事では次のレイアウトを、ConstraintLayoutで組みたいと思います。

---

{{< figure src="/blog/android/jetpack/constraintlayout/constraint-practical-1.png" width="400" >}}

---

ポイントは以下の3つです。

- 4つの画像を組み合わせて、1つの円形画像を作る部分
- 01/01の部分は右に、@your_idの部分はYour Nameの右に配置される
- Your Nameの部分の文字数が多いときに、...で省略される

今回は、4つの画像を組み合わせる部分で、Material ComponentのShapeableImageViewを使っています。ShapeableImageViewの説明は[ここを](/2019/12/material-shapeable-image-view/)見てください。

上記で上げたポイントを中心に説明していきます。

## 4つの画像を組み合わせて、1つの円形画像を作る部分

この部分は、Picasso、Glideなどの画像ライブラリに頼るパターンもあると思うんですが、今回はShapeableImageViewを使いたいと思います。

4つのShapeableImageViewを使い、それぞれ「左上、右上、左下、右下」の部分が丸くなるShapeを定義します。
左上を丸くするShapeは次のように定義できます。

```xml
<style name="ShapeAppearanceOverlay.Example.TopLeft" parent="">
  <item name="cornerFamilyTopLeft">rounded</item>
  <item name="cornerSizeTopLeft">100%</item>
</style>
```

右の画像が、上のShapeを指定した場合になります。

{{< figure src="/blog/android/jetpack/constraintlayout/constraint-practical-2.png" width="400" >}}

右上、左下、右下のShapeも同じように作ってあげます。そして、それら4つViewの制約を次のように作ってあげます。

```xml
<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/top_left"
  android:layout_width="30dp"
  android:layout_height="30dp"
  android:adjustViewBounds="true"
  android:scaleType="centerCrop"
  app:layout_constraintStart_toStartOf="parent"
  app:layout_constraintTop_toTopOf="parent"
  app:shapeAppearanceOverlay="@style/ShapeAppearanceOverlay.Example.TopLeft"
  tools:src="@tools:sample/avatars" />

<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/top_right"
  android:layout_width="30dp"
  android:layout_height="30dp"
  android:adjustViewBounds="true"
  android:scaleType="centerCrop"
  app:layout_constraintStart_toEndOf="@id/top_left"
  app:layout_constraintTop_toTopOf="@id/top_left"
  app:shapeAppearanceOverlay="@style/ShapeAppearanceOverlay.Example.TopRight"
  tools:src="@tools:sample/avatars" />

<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/bottom_left"
  android:layout_width="30dp"
  android:layout_height="30dp"
  android:adjustViewBounds="true"
  android:scaleType="centerCrop"
  app:layout_constraintStart_toStartOf="@id/top_left"
  app:layout_constraintTop_toBottomOf="@id/top_left"
  app:shapeAppearanceOverlay="@style/ShapeAppearanceOverlay.Example.BottomLeft"
  tools:src="@tools:sample/avatars" />

<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/bottom_right"
  android:layout_width="30dp"
  android:layout_height="30dp"
  android:adjustViewBounds="true"
  android:scaleType="centerCrop"
  app:layout_constraintStart_toEndOf="@id/bottom_left"
  app:layout_constraintTop_toTopOf="@id/bottom_left"
  app:shapeAppearanceOverlay="@style/ShapeAppearanceOverlay.Example.BottomRight"
  tools:src="@tools:sample/avatars" />
```

やっていることは単純で、4つのViewを`app:layout_constraintStart_toStartOf`などを使って適切な位置に配置しています。

---

{{< figure src="/blog/android/jetpack/constraintlayout/constraint-practical-3.png" width="400" >}}

---

これで4つの画像を組み合わせて、円形の画像を表現するところまで出来ました。

# 01/01の部分は右に、@your_idの部分はYour Nameの右に配置される

01/01の部分は右に固定なので、`app:layout_constraintEnd_toEndOf="parent"`を指定すれば良さそうです

```xml
<TextView
  android:id="@+id/date"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:layout_marginEnd="16dp"
  app:layout_constraintEnd_toEndOf="parent" />
```

次にYour Nameは一番左に、@your_idはそれの右に配置します。

```xml
<TextView
  android:id="@+id/name"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:ellipsize="end"
  android:lines="1"
  app:layout_constraintEnd_toStartOf="@id/user_id"
  app:layout_constraintStart_toEndOf="@id/top_right"
  app:layout_constraintTop_toTopOf="parent"
  tools:text="TESTTESTTES" />

<TextView
  android:id="@+id/user_id"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:layout_marginStart="6dp"
  android:layout_marginEnd="8dp"
  android:lines="1"
  app:layout_constraintBaseline_toBaselineOf="@id/name"
  app:layout_constraintEnd_toStartOf="@id/date"
  app:layout_constraintStart_toEndOf="@id/name"
  tools:text="your id" />
```

すると次のようになります。

---

{{< figure src="/blog/android/jetpack/constraintlayout/constraint-practical-4.png" width="400" >}}

---

Your Nameと、@your_idが真ん中に寄ってしまったので、`app:layout_constraintHorizontal_chainStyle`を指定して、左寄せにします。

```xml
<TextView
  android:id="@+id/name"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:ellipsize="end"
  android:lines="1"
  app:layout_constraintEnd_toStartOf="@id/user_id"
  app:layout_constraintHorizontal_bias="0"
  app:layout_constraintHorizontal_chainStyle="packed"
  app:layout_constraintStart_toEndOf="@id/top_right"
  app:layout_constraintTop_toTopOf="parent"
  tools:text="TESTTESTTES" />
```

---

{{< figure src="/blog/android/jetpack/constraintlayout/constraint-practical-5.png" width="400" >}}

---

これで、3つのTextViewを正しい位置に配置することが出来ました。

## Your Nameの部分の文字数が多いときに、...で省略される

次に、Your Nameの文字数が多い場合を考えていきます。

現状のレイアウトだと、文字数が多い場合には次のように表示されます。

---

{{< figure src="/blog/android/jetpack/constraintlayout/constraint-practical-6.png" width="400" >}}

---

wrap_contentの場合、制約を超えて枠をはみ出してしまうためです。

これを解決するために、`app:layout_constrainedWidth`を使います。これを使うと、制約を超えないwrap_contentを定義できます。

```xml
<TextView
  android:id="@+id/name"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:ellipsize="end"
  android:lines="1"
  app:layout_constrainedWidth="true"
  app:layout_constraintEnd_toStartOf="@id/user_id"
  app:layout_constraintHorizontal_bias="0"
  app:layout_constraintHorizontal_chainStyle="packed"
  app:layout_constraintStart_toEndOf="@id/top_right"
  app:layout_constraintTop_toTopOf="parent"
  tools:text="TESTTESTTES" />
```

---

{{< figure src="/blog/android/jetpack/constraintlayout/constraint-practical-7.png" width="400" >}}

---

Your Nameの部分を...にすることが出来ました。

あとの部分はそう難しくないと思うので、説明を省略させてください。[全コード](https://github.com/satoshun-android-example/ConstraintLayout/blob/master/app/src/main/res/layout/divide_into_four_item.xml)はここにあります。


## まとめ

- ShapeableImageViewは便利
- `layout_constraintHorizontal_chainStyle`、`layout_constraintHorizontal_bias`を組み合わせることで左寄せに出来る
- 制約を超えないように、wrap_contentのレイアウトを使いたい場合は、`layout_constrainedWidth`を使う
- [ソースコード](https://github.com/satoshun-android-example/ConstraintLayout/blob/master/app/src/main/res/layout/divide_into_four_item.xml)はここにあります。
