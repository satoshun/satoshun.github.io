+++
date = "Fri Nov 08 10:27:21 UTC 2024"
title = "Jetpack Composeライブ編集(Live Edit)でPrintデバッグをする"
tags = ["android", "debugging"]
blogimport = true
type = "post"
draft = false
+++

Jetpack Composeには、実機にリアルタイムで変更内容を反映する「ライブ編集（Live Edit）」という機能があります。
ライブ編集のメイン機能はUIの調整なのですが、printなどのログ出力コードも追加可能で、デバッグにも活用する事ができます。

なので、Composable関数周りでデバッグを行いたい場合は、以下の手順がおすすめです。
1. ライブ編集 + Printを用いて値をログに出力する
2. これで特定できなかったら、デバッガーを起動する

UIの値を確認したい時などに便利なので、ぜひ活用してみてください。

## 参考

- [https://developer.android.com/develop/ui/compose/tooling/iterative-development](https://developer.android.com/develop/ui/compose/tooling/iterative-development?hl=ja)
