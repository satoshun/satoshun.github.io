+++
date = "Tue Apr 30 05:59:41 UTC 2019"
title = "Material Components: MotionSpecを使ってアニメーションをカスタマイズする"
tags = ["android", "materialcomponents", "motion"]
thumbnail = "https://lh3.googleusercontent.com/7iMReht6cMrOlVErQmTAHUTkcsk8GG76aQR1hwVEA_TCnOtrAgCOEoJU8SH6bhzdMcEOv6Z-pWU=w246-h437-no"
blogimport = true
type = "post"
draft = false
+++

[MotionSpec](https://developer.android.com/reference/com/google/android/material/animation/MotionSpec)はAndroid material componentsに定義されている1クラスになります。
MotionSpecを使うことで、次のアニメーション属性をカスタマイズすることができます。

- startOffset
- duration
- interpolator
- repeatCount
- repeatMode

例えば、アニメーションを長くしたいときは、durationの値を長く、アニメーションの開始時間を遅らせたいなら、startOffsetの値を長くします。

FloatingActionButtonを例に、実際にMotionSpecの値をいじってみます。

## 最初にMotionSpec用のanimator XMLを定義する

デフォルトのXMLをコピペしてきて、それをベースにカスタマイズするのが良いと思います。

FloatingActionButton用のXMLはソースコードを読んでいくと、`design_fab_show_motion_spec.xml`と`design_fab_hide_motion_spec.xml`で定義されていることが分かります。MotionSpecは、show/hide用の2種類があり、カスタマイズしたいときは両方とも変更する必要があります。

まずはshow用のMotionSpecを変更していきます。以下がデフォルトで定義されている`design_fab_show_motion_spec.xml`の中身になります。

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
    Copyright 2017 The Android Open Source Project

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
-->

<set xmlns:android="http://schemas.android.com/apk/res/android">
  <objectAnimator
      android:propertyName="opacity"
      android:startOffset="0"
      android:duration="200"
      android:interpolator="@interpolator/mtrl_linear_out_slow_in"/>
  <objectAnimator
      android:propertyName="scale"
      android:startOffset="0"
      android:duration="200"
      android:interpolator="@interpolator/mtrl_linear_out_slow_in"/>
  <objectAnimator
      android:propertyName="iconScale"
      android:startOffset="0"
      android:duration="0"
      android:interpolator="@interpolator/mtrl_fast_out_slow_in"/>
</set>
```

今回は、アニメーションを長くして、アイコンをバウンドしたいとします。
それは、durationを長くして、interpolatorに`@android:anim/bounce_interpolator`を指定すれば良いです。

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
  <objectAnimator
    android:duration="1000"
    android:interpolator="@android:anim/bounce_interpolator"
    android:propertyName="opacity"
    android:startOffset="0" />

  <objectAnimator
    android:duration="1000"
    android:interpolator="@android:anim/bounce_interpolator"
    android:propertyName="scale"
    android:startOffset="0" />

  <objectAnimator
    android:duration="1000"
    android:interpolator="@android:anim/bounce_interpolator"
    android:propertyName="iconScale"
    android:startOffset="0" />
</set>
```

これでshow用のMotionSpecが完成しました。hideでも同じようにXMLを定義してあげます。

次に、これをViewにセットします。

```kotlin
fab.showMotionSpec = MotionSpec.createFromResource(
    this,
    R.animator.fab_show_motion_spec
)
fab.hideMotionSpec = MotionSpec.createFromResource(
    this,
    R.animator.fab_hide_motion_spec
)
```

最終的な、アニメーションはこんな感じになります。

<img src="https://lh3.googleusercontent.com/7iMReht6cMrOlVErQmTAHUTkcsk8GG76aQR1hwVEA_TCnOtrAgCOEoJU8SH6bhzdMcEOv6Z-pWU=w246-h437-no" width=400 />

アニメーション時間が長くなり、ボヨンボヨンしていることが分かると思います。

## まとめ

- MotionSpecを使えば、アニメーションの長さやinterpolatorをカスタマイズすることが出来る
  - まだ、MotionSpecに対応しているViewは少ないが、今後Chipなども対応予定😃

---

内容におかしい点や、もっとこうしたほうがいいよって！！いうのがあればTwitterなどから教えてもらえればとても嬉しいです😊
