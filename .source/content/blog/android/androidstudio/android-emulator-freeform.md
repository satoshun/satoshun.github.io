+++
date = "Sat Jun 13 08:38:48 UTC 2020"
title = "Android: エミュレータでFreeformモードを使う"
tags = ["android", "emulator"]
blogimport = true
type = "post"
draft = false
+++

おそらく [30.0.10](https://developer.android.com/studio/releases/emulator#30-0-10) から、Freeformモードなるものがエミュレータで実行できるようになっていたので、それの紹介です。

この記事では、`Android Studio 4.2 Preview1`を使っています。

## 使い方

セットアップは簡単で、`AVD Manager -> Create Virtual Device` から、 `13.5" Freeform` を指定します。

{{< figure src="/blog/android/androidstudio/freeform-create-virtual-device.png" >}}

## 動作例

Clockアプリで試してみました。次のgifアニメーション画像のように動作をします。

{{< figure src="/blog/android/androidstudio/freeform-sample.gif" >}}

アプリを立ち上げた後に、簡単に画面サイズを変更することが出来ます。

## まとめ

複数の端末サイズで確認したい時などに、便利なのかなと思いました（こなみかん）
