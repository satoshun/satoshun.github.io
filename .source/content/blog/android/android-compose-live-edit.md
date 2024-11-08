+++
date = "Fri Nov 08 10:27:21 UTC 2024"
title = "Android at Scale(with Circuit)の感想"
tags = ["android", "debugging"]
blogimport = true
type = "post"
draft = false
+++

Jetpack Composeには、実機にリアルタイムで変更内容を反映する「ライブ編集（Live Edit）」という機能があります。
ライブ編集のメイン機能はUIの調整なのですが、printなどのログ出力コードも追加可能です。ライブ編集を活用することで、Composable関数に限り、効率的にPrintデバッグが行えます。

なので、Composable関数周りでデバッグを行いたい場合は、以下の手順がおすすめです。
1. ライブ編集 + Printを用いて怪しい値をログに出力する
2. これで特定できなかったら、デバッガーを起動する

注意として、ライブ編集は開発中の機能であるため、割と新しいAS、Composeライブラリを使う必要があります。

## 参考

- [https://developer.android.com/develop/ui/compose/tooling/iterative-development](https://developer.android.com/develop/ui/compose/tooling/iterative-development?hl=ja)
