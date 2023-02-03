+++
date = "Fri Feb  3 06:39:59 UTC 2023"
title = "Android: Wireless debuggingでうまく繋がらない時にadb connectを使う"
tags = ["android"]
blogimport = true
type = "post"
draft = false
+++

Android Studioを再起動したり、何かいろいろとやっているとWireless debuggingが上手く繋がらないことがあります。
そういう場合にはadb connectコマンドから直接接続してあげると、上手くつながってくれることがあります。

```
> adb connect ip_address:port
```

IPアドレス、ポート番号は、端末のワイヤレスデバッグ画面から確認することができます。

{{< figure src="/blog/android/productivity/wireless-debugging.jpeg" width="300">}}

この場合、172.22.185.228:41583 がIPアドレス、ポート番号になります。

```
> adb connect 172.22.185.228:41583
```
