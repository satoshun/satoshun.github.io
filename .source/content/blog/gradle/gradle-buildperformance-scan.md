+++
date = "Sun Apr 10 07:10:16 UTC 2022"
title = "Android: Gradle Build Scanで、マルチモジュール環境のビルド改善"
tags = ["gradle", "android"]
blogimport = true
type = "post"
draft = false
+++

GradleにはBuild Scanという機能があり、これを使うことで、プロジェクトのビルドの全体像を知ることが出来ます。

使い方は簡単で `--scan` を指定するだけです。

```shell
./gradlew assembleDebug --scan
```

これで、GradleのBuild Scanをしてくれます。
今回はこれをヒントにして、モジュール間の依存を見直すことで、ビルドの改善をしてみます。

左タブのTimelineから、全体的なビルドの確認をすることが出来ます。

![](https://paper-attachments.dropbox.com/s_82512F6D62C51E4BD720E70C8981F518B8055502F7A6812E3E86458A5B1C7F53_1649564767788_Screen+Shot+2022-04-10+at+13.25.56.png)

Timelineでは、どのような順番でmoduleのビルドが行われたかを確認することが出来ます。
時間が掛かっているモジュール（クリティカルパス）を改善すれば、全体のビルドを改善することが期待できます。

Timelineでは、どのモジュールがクリティカルパスになっているかを検索することが出来ます。

![](https://paper-attachments.dropbox.com/s_82512F6D62C51E4BD720E70C8981F518B8055502F7A6812E3E86458A5B1C7F53_1649564981910_Screen+Shot+2022-04-10+at+13.29.34.png)

クリティカルパス検索をすると、次の画像のように、クリティカルなタスク（モジュール）に色が付きます。

![](https://paper-attachments.dropbox.com/s_82512F6D62C51E4BD720E70C8981F518B8055502F7A6812E3E86458A5B1C7F53_1649564832644_Screen+Shot+2022-04-10+at+13.27.02.png)

これで、クリティカルパスとなっているモジュールを知ることが出来ます。

次に、このクリティカルなモジュールの改善を考えます。大きく2つの方法があると思います。

1. モジュール自体のサイズを小さくする → このモジュールで管理するクラス数を減らす
2. ビルドするタイミングを早くする → 他のモジュールへの依存を減らす

1.に関しては、それはそうという感じなので、2について説明します。

例えば、A <- B <- C のような依存構成になっているときに、Bがクリティカルであるとします。
このときに、BがAに依存しないように変更することで、Bのビルドタイミングを早めることができ、全体のビルドが改善されることが期待できます。
「BがAに依存しないように変更できる」、また、「その変更によってモジュールの責務が崩れない」ことが前提となりますが、無駄な他モジュールへの依存を無くすだけでも、ビルドの改善が出来る場合があります。


## まとめ

こんな感じで、Gradle Scanを使い、ちょこちょことTimelineのクリティカルパスを改善することで、ビルド時間の改善が期待出来ます。

