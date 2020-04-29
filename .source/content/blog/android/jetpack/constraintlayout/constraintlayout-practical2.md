+++
date = "Wed Apr 29 09:56:01 UTC 2020"
title = "ちょっと複雑なレイアウトをConstraintLayoutで組んでみるpart2"
tags = ["android", "constraintlayout"]
blogimport = true
type = "post"
draft = true
+++

今回の記事では次のレイアウトを、ConstraintLayoutで組みたいと思います。

---

{{< figure src="/blog/android/jetpack/constraintlayout/constraint-practical2-1.png" width="400" >}}

---

ポイントは以下の２つです。

- 各画像は、前の画像に被っている
- 各画像は、4、3、2、1枚のいずれかの枚数を取る

上記で上げたポイントを中心に説明していきます。

## 各画像は、前の画像に被っている

View同士を被せる方法はいくつかあると思うんですが、今回は`layout_constraintStart_toStartOf`と`layout_marginStart`を使い表現します。

最初に、一番左に配置するImageViewを定義します。

```xml
<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/image1"
  android:layout_width="32dp"
  android:layout_height="32dp"
  android:layout_marginStart="20dp"
  android:layout_marginBottom="12dp"
  app:layout_constraintBottom_toBottomOf="parent"
  app:layout_constraintStart_toStartOf="parent" />
```

二番目に配置するImageViewは、このImageViewに被るように配置したいので、このViewのendに制約を合わせてしまうと上手く動かせません。
なので、このViewのstartに制約を合わせて、これだけだと全て被ってしまうので、加えてmarginStartも設定することでいい感じに表示することが出来ます。

```xml
<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/image2"
  android:layout_width="32dp"
  android:layout_height="32dp"
  android:layout_marginStart="24dp"
  android:layout_marginBottom="12dp"
  app:layout_constraintBottom_toBottomOf="parent"
  app:layout_constraintStart_toStartOf="@id/image1" />
```

三番目、四番目でも同様に制約を設定してあげます。最終形は次のようになります。

```xml
<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/image1"
  android:layout_width="32dp"
  android:layout_height="32dp"
  android:layout_marginStart="20dp"
  android:layout_marginBottom="12dp"
  app:layout_constraintBottom_toBottomOf="parent"
  app:layout_constraintStart_toStartOf="parent" />

<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/image2"
  android:layout_width="32dp"
  android:layout_height="32dp"
  android:layout_marginStart="24dp"
  android:layout_marginBottom="12dp"
  app:layout_constraintBottom_toBottomOf="parent"
  app:layout_constraintStart_toStartOf="@id/image1" />

<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/image3"
  android:layout_width="32dp"
  android:layout_height="32dp"
  android:layout_marginStart="24dp"
  android:layout_marginBottom="12dp"
  app:layout_constraintBottom_toBottomOf="parent"
  app:layout_constraintStart_toStartOf="@id/image2" />

<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/image4"
  android:layout_width="32dp"
  android:layout_height="32dp"
  android:layout_marginStart="24dp"
  android:layout_marginBottom="12dp"
  app:layout_constraintBottom_toBottomOf="parent"
  app:layout_constraintStart_toStartOf="@id/image3" />
```

一枚目、二枚目、三枚目、四枚目がそれぞれ少しずつ被っているのが分かると思います。

---

{{< figure src="/blog/android/jetpack/constraintlayout/constraint-practical2-2.png" width="400" >}}

---

## 各画像は、4、3、2、1枚のいずれかの枚数を取る

次に画像枚数の変動するパターンについて考えてみます。4つのレイアウトを個別に作ってもいいんですが、今回は1つのレイアウトで表現しようと思います。
今回のレイアウトの場合、`layout_goneMarginStart`を使うといい感じに出来ます。

具体的には、前のレイアウトに次のように`layout_goneMarginStart`を追加してあげます。

```xml
<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/image1"
  android:layout_width="32dp"
  android:layout_height="32dp"
  android:layout_marginStart="20dp"
  android:layout_marginBottom="12dp"
  app:layout_constraintBottom_toBottomOf="parent"
  app:layout_constraintStart_toStartOf="parent" />

<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/image2"
  android:layout_width="32dp"
  android:layout_height="32dp"
  android:layout_marginStart="24dp"
  android:layout_marginBottom="12dp"
  app:layout_constraintBottom_toBottomOf="parent"
  app:layout_constraintStart_toStartOf="@id/image1"
  app:layout_goneMarginStart="32dp" />

<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/image3"
  android:layout_width="32dp"
  android:layout_height="32dp"
  android:layout_marginStart="24dp"
  android:layout_marginBottom="12dp"
  app:layout_constraintBottom_toBottomOf="parent"
  app:layout_constraintStart_toStartOf="@id/image2"
  app:layout_goneMarginStart="44dp" />

<com.google.android.material.imageview.ShapeableImageView
  android:id="@+id/image4"
  android:layout_width="32dp"
  android:layout_height="32dp"
  android:layout_marginStart="24dp"
  android:layout_marginBottom="12dp"
  app:layout_constraintBottom_toBottomOf="parent"
  app:layout_constraintStart_toStartOf="@id/image3"
  app:layout_goneMarginStart="56dp" />
```

---

{{< figure src="/blog/android/jetpack/constraintlayout/constraint-practical2-3.png" width="400" >}}

---

`layout_goneMarginStart`を使うことで、Viewがない場合の配置を調整することが出来ます。

## まとめ

- View同士が被る場合には、制約に工夫が必要
- goneMarginを使うことで、制約対象のViewがない場合の位置を調整できる

フルコードは[GitHub](https://github.com/satoshun-android-example/ConstraintLayout/blob/master/app/src/main/res/layout/circle_image_sequence_item.xml)にあります。
