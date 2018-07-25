+++
date = "2018-07-25T00:00:00Z"
title = "Android: ContraintLayoutでネガティブマージンを実現する"
tags = ["android", "constraintlayout"]
blogimport = true
type = "post"
+++

ConstraintLayoutはネガティブマージンに対応していないため、少しテクニックを使う必要があります。
この記事では[Space](https://developer.android.com/reference/android/widget/Space)を使ったネガティブマージンの実現について説明します。

## 例

ネガティブマージンと同等の大きさを持った`Space`を定義して、そこにConstraintを設定するだけです。

```xml
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:clipToPadding="false"
    android:padding="80dp"
    tools:context=".ui.user.main.MainActivity">

  <Space
      android:id="@+id/negative"
      android:layout_width="15dp"
      android:layout_height="15dp"
      app:layout_constraintStart_toStartOf="parent"
      app:layout_constraintTop_toTopOf="parent" />

  <ImageView
      android:id="@+id/icon"
      android:layout_width="30dp"
      android:layout_height="30dp"
      android:contentDescription="icon"
      app:layout_constraintBottom_toBottomOf="@id/negative"
      app:layout_constraintEnd_toEndOf="@id/negative"
      tools:src="@tools:sample/avatars" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

<img src="https://lh3.googleusercontent.com/LQrkHsAeTBvqlZWuw1Ccpm_dvrnCsdK7aH2rMB6HYakF_jv3q6Zf_-QBtfOo1eD0EjLBOjxy11nx2TAlZstr354XRiSXt7HUxENJXji8ZxWNxeJUTe8g9jIstSO1PRMsw7f8O5xuCakktrJLSUPLk01CI1N6OJ9MGMvsSbSDO6o4t7oeipkV9f7klqmzdxtoxESyDzoEI5SfeMOxynhmTGLwv6RnaASEOzVyL2xEW8eq63erAiu6ptdwl-_rQkxAnAIhYpSuPZjweCPBzf3GiMJi5gx_Ciz348GmoSaTIhuGM6grHd181fBGkZQkbvgg5ggrk5pLlja1N4eZA54n5rgzKs5gT36fA8K_ZFrAQI0H50cvAbesIeDVb05MZ2pP592Kwb8moTt6xp0TTFYrJf-c-MVljio-JvAKuUC0fPWPpdgMyxkJHW4AzK9l-Yo4f2-dUdfEf06J6ktMOsld0Yzsyu07Hki6MSGeBjU3bJ3R0vDi-W6SBUNmcDzJu5kC-2ItGmq4sN8fq06XJMSwVTmfDAyfHOluWHOTlL5LV3YVvYZxxgGJ6_ldqdD2HZ0ZW8wuNpfrHTxGLYCHR5EN5ouY5ILx-snrNAfQPQ9Z9Ph0U_4UeCMlODslXyRpueecgOSBJUhwc47xrQ04b-xOJz_01EWEA61x=w1452-h720-no" alt="constraintlayout-image" width="400px"/>

簡単に説明すると、`Space`に15pxを指定して、bottom, endに対してconstraintを指定することで、ネタティブマージンを達成しています。
上記の例だと、`android:layout_marginStart="-15px"`、`android:layout_marginTop="-15px"`と同等の振る舞いをしています。

## まとめ

ConstraintLayoutではネイティブでネガティブマージンに対応していないため、`Space`を使った、ややテクニカルな方法で実現するのが良いと思われます。
