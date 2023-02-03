+++
date = "Sun Jan  8 02:11:26 UTC 2023"
title = "Jetpack Compose Compiler/Runtimeを使ったプレゼンテーション層についての雑談"
tags = ["android", "architecture", "compose"]
blogimport = true
type = "post"
draft = true
+++

Jetpack Composeが普及すると共に、Jetpack ComposeをUIの構築だけではなく、プレゼンテーション層でも使う動きが出てきています。

背景として、[A Jetpack Compose by any other name](https://jakewharton.com/a-jetpack-compose-by-any-other-name/)で詳しく解説されていますが、
Jetpack Composeは Compiler/Runtime、UIの2つで構成されており、前者のCompilerは状態管理に適しています。その状態管理の部分がプレゼンテーション層でも有効に使えそうなので、徐々に広がりを見せています。
またCash App社が[molecule](https://github.com/cashapp/molecule)ライブラリを公開したことも大きな要因としてあると思います。

具体的に、Compose Compilerをプレゼンテーション層で使うメリットとしては、複数のストリームを扱うときに現れます。例えばCoroutine Flowの場合には、combineXXXメソッドなどで複数のストリームを束ねて、UiStateなどに変換する使い方が1つあるんですが、
ストリームが多くなると、とたんにエラー処理などが複雑になるという問題があります。
ここで、Compose Compilerを使うことで手続き的なコードに変換することができるため、複雑になりすぎる問題を解決することが出来ます。Kotlin Coroutineのサスペンド関数のようなメリットを受けることが出来ます。

また、まだexperimentalですが、[Circuit](https://slackhq.github.io/circuit/)ライブラリも公開されており、このライブラリではmoleculeよりもいろいろなものを提供しており、
こう書いてくださいというところまで提供されています。

まだこの段階ではデファクトな書き方が決まっていないためチャレンジングな採用になってしまうと思いますが、今後ある程度の市民権な得ると思うので、
中、大規模なプロジェクトはCompose Compilerプレゼンテーション層に適合すること検討しても良いと思っています。
