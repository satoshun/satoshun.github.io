+++
date = "Sun Apr 28 09:23:02 UTC 2019"
title = "TODO"
tags = ["android", "constraintlayout", "flow"]
blogimport = true
type = "post"
draft = true
+++

ConstraintLayoutの2.0.0 alpha 5にFlow Virtual Layoutが導入されました🎉
Flowは対象のViewを様々な方法で並べることができます。

次がメリットとしてあります。

- Viewの階層をフラットに保つことが出来る
- Flowは普通のViewのように扱う事ができる
- MotionLayoutと相性が良い

どのように書くのかを見ていきます。

---

1. 方向を決める

`android:orientation`から、horizontal or verticalを選択できます。

```xml
<androidx.constraintlayout.widget.ConstraintLayout
  android:layout_width="match_parent"
  android:layout_height="match_parent">

  <androidx.constraintlayout.helper.widget.Flow
    android:id="@+id/flow"
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:orientation="horizontal"
    android:background="@android:color/white"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />

  ...
```

このFlowはorientationにhorizontalを持ち、通常のViewのようにconstraintsを指定し配置することができます。

(Flowは普通のViewのように扱うことができます。)

2. wrapModeを決める

`flow_wrapMode`で指定することができます。

wrapModeでは、どのようにViewを並べるかを指定でき、3種類のmodeがあります。

1. none
    - 単純にsingle lineに並べる
2. chain
    - 単純に順番に配置していく。その行（列）に収まらない場合は次の行（列）に配置する
3. aligned
    - 各要素を整列するように配置していく。テーブルのようなイメージ

```xml
  <androidx.constraintlayout.helper.widget.Flow
    android:id="@+id/flow"
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:orientation="horizontal"
    android:background="@android:color/white"
    app:flow_wrapMode="chain"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

3. 対象のViewを指定する

`constraint_referenced_ids`から指定します。

```xml
<androidx.constraintlayout.widget.ConstraintLayout
  android:layout_width="match_parent"
  android:layout_height="match_parent">

  <androidx.constraintlayout.helper.widget.Flow
    android:id="@+id/flow"
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:orientation="horizontal"
    app:flow_wrapMode="chain"
    android:background="@android:color/white"
    app:constraint_referenced_ids="title1,title2,title3"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />

  <TextView
    android:id="@+id/title1"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="chain1"
    android:textColor="@android:color/black"
    android:textSize="20sp" />

  <TextView
    android:id="@+id/title2"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="chain2"
    android:textColor="@android:color/black"
    android:textSize="20sp" />

  <TextView
    android:id="@+id/title3"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="chain3"
    android:textColor="@android:color/black"
    android:textSize="50sp" />

  ...
```

この場合、「title1、title2、title3」がhorizontalに、chain指定なので順番に配置されます。

4. 細かい調整

対象のView間のマージンや、行（列）の最大数、Viewの配置場所などの細かい部分の指定ができます。

- app:flow_horizontalStyle = "spread|spread_inside|packed" (default spread)
- app:flow_verticalStyle = "spread|spread_inside|packed" (default spread)
- app:flow_horizontalBias = "float" (default 0.5)
- app:flow_verticalBias = "float" (default 0.5)
- app:flow_horizontalGap = "dimension" (default 0)
- app:flow_verticalGap = "dimension" (default 0)
- app:flow_horizontalAlign = "start|end|center" (default center)
- app:flow_verticalAlign = "top|bottom|center|baseline” (default center)
- app:flow_maxElementsWrap = "integer" (default : 0, not applied)

## MotionLayout

これは通常のMotionLayoutの使い方と一緒です。Flowの値を変更してあげればよいです。

```xml
<?xml version="1.0" encoding="utf-8"?>
<MotionScene xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:motion="http://schemas.android.com/apk/res-auto">

  <Transition
    android:id="@+id/transition"
    motion:constraintSetEnd="@+id/end"
    motion:constraintSetStart="@+id/start"
    motion:duration="1000" />

  <ConstraintSet android:id="@+id/start">
    <Constraint
      android:id="@id/flow"
      android:layout_width="0dp"
      android:layout_height="0dp"
      android:orientation="horizontal"
      motion:layout_constraintBottom_toBottomOf="parent"
      motion:layout_constraintEnd_toEndOf="parent"
      motion:layout_constraintStart_toStartOf="parent"
      motion:layout_constraintTop_toTopOf="parent" />
  </ConstraintSet>

  <ConstraintSet android:id="@+id/end">
    <Constraint
      android:id="@id/flow"
      android:layout_width="200dp"
      android:layout_height="0dp"
      android:orientation="vertical"
      motion:layout_constraintBottom_toBottomOf="parent"
      motion:layout_constraintEnd_toEndOf="parent"
      motion:layout_constraintStart_toStartOf="parent"
      motion:layout_constraintTop_toTopOf="parent" />
  </ConstraintSet>
</MotionScene>
```

こんな感じのアニメーションになります。

<img src="https://lh3.googleusercontent.com/qn9Xbhr9pqrmsJEzQ2YfhzSnJS1I9HpR_s6_UrCSxoqRHQQQ32unFB3G4ls72OuMLuIclSZ89-8=w246-h437-no" width=400>

## まとめ

- ConstraintLayout alpha5になってFlowが入った。かなり便利に使えそう
- betaはGoogle I/O前後に来るらしいので、正式版までもう少し😃

## 参考

- [ConstraintLayout 2.0](https://speakerdeck.com/camaelon/constraintlayout-2-dot-0)
- [ConstraintLayout 2.0.0 alpha 5](https://androidstudio.googleblog.com/2019/04/constraintlayout-200-alpha-5.html)
