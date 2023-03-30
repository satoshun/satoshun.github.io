+++
date = "Thu Mar 30 14:23:54 UTC 2023"
title = "Android: Eng modeの紹介"
tags = ["android", "build", "gradle"]
blogimport = true
type = "post"
draft = false
+++

[Netflix + Gradle, A Journey in Developer Productivity](https://speakerdeck.com/eboudrant/netflix-plus-gradle-a-journey-in-developer-productivity?slide=17)の発表で、Eng modeの解説をしていて、いいテクニックだと思ったのでそれの紹介です。

Eng modeとは、Eng modeフラグを作り、それがtrueのときに不必要な機能をoffにして、ビルドを高速化しようというテクニックです。具体的には次の機能をoff、または調整しているらしいです。

- incremental buildの最適化
- 不要なプラグイン(jacocoなど)
- Variantの削除
- app bundlesの削除

これらを調整することで、Gradleのconfiguration time、ビルドの短縮をし、開発の効率化をすることが出来ます。

より詳しくは、[スライド](https://speakerdeck.com/eboudrant/netflix-plus-gradle-a-journey-in-developer-productivity?slide=17)か、もしくは[動画](https://www.droidcon.com/2022/09/29/netflix-gradle-a-journey-in-developer-productivity-2/)を見てください。
