+++
date = ""
title = "Android: Wireless debuggingでうまく繋がらない時にadb connectを使う"
tags = ["android"]
blogimport = true
type = "post"
draft = true
+++

Android Studioを再起動したり、何かいろいろとやっているとWireless debuggingが上手く繋がらないことがあります。
そのときにはadb connectコマンドでターミナルから直接接続してあげると、上手くつながってくれます。


```text
adb connect ip_address:port
```

IPアドレスは、端末のワイヤレスデバッグ画面から確認することができます。

{{< figure src="/blog/android/productivity/wireless-debugging.jpeg" >}}
