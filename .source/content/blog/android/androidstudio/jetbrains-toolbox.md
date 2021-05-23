+++
date = "Sun May 23 03:00:22 UTC 2021"
title = "JetBrains Toolbox Appのススメ"
tags = ["android"]
blogimport = true
type = "post"
draft = false
+++

[JeBbrains Toolbox App](https://www.jetbrains.com/toolbox-app/) が便利なので、それの啓蒙ブログです。


## JetBrains Toolbox App?

JetBrains製のIDEのバージョンを管理するアプリになります。Android開発の場合だと、stable版、beta版、canary版を別々に管理することが主な使い方になります。

これで説明は終わりなのですが、設定しておくと便利な機能を紹介します。


## シェルコマンドの生成

ONにしておくと、シェルコマンドを生成をしてくれます。<br><br>

{{< figure src="/blog/android/androidstudio/jetbrains-toolbox-shell-command.png" width="480">}}<br>

次のようにコマンドラインから、プロジェクトを開くことが出来ます。

```shell
studio .
```

git clone をした後などに、上記コマンドを入力することで、プロジェクトを簡単に開くことが出来ます。


## インストール先のフォルダを `/Applications` にする

下記のツイートにあったのですが、spotlightによるインデックスがいい感じになるそうです。

https://twitter.com/kaushikgopal/status/1396210659847819264


## まとめ

JetPack Composeやらなんやらで、beta、canaryなどの開発版を使うことも多くなると思うので、入れとくと幸せになります。
