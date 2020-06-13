+++
ate = "Sat Jun 13 07:51:47 UTC 2020"
title = "Android: エミュレータでFreeformモードを使う"
tags = ["android", "emulator"]
blogimport = true
type = "post"
draft = true
+++

おそらく [30.0.10](https://developer.android.com/studio/releases/emulator#30-0-10) から、Freeformモードなるものがエミュレータで実行できるようになっていたので、それの紹介です。

この記事は、`Android Studio 4.2 Preview1`を使っています。

## 使い方

セットアップは簡単で、AVD Manager -> Create Virtual Deviceから、 `13.5" Freeform` を指定します。

{{< figure src="/blog/android/androidstudio/freeform-create-virtual-device.png" >}}

## 動作

Clockアプリで試してみました。下のgif画像のように動作をします。

{{< figure src="/blog/android/androidstudio/freeform-sample.gif" >}}

## まとめ

複数の端末サイズで確認したいときに、便利なのかなと思いました（こなみかん）
