+++
date = "Sat Jul 20 02:57:15 UTC 2019"
title = "GitHubのNotificationsで、androidxライブラリのリリースの通知を受け取れるリポジトリを作りました"
tags = ["product", "android"]
blogimport = true
type = "post"
+++

androidxのライブラリのリリースをGitHubのReleases onlyを使って検知するためのリポジトリを作りました。
[androidx-releases](https://github.com/androidx-releases)にそれぞれのリポジトリがあります。

例えば、Appcompatのリリースを検知したかったら、次の画像のように、[androidx-releases/appcompat](https://github.com/androidx-releases/appcompat)のNotificationsのReleases Onlyをチェックします。

<br />

<img src="/blog/product/androidx-releases-1.png" width="50%" />

<br />

この状態で、Appcompatに新しいリリースがあったら、GitHubに登録してあるメールアドレスにメールが届きます。

<br />

<img src="/blog/product/androidx-releases-2.png" width="70%" />

<br />

## まとめ

- 作ってみたものの、これが便利なのかは良く分かってないです:D
- 他のandroidxライブラリのリリースの通知を受けたいなら、[androidx-releases](https://github.com/androidx-releases)から登録を。
- 何か他に欲しいandroidxリポジトリがあったら教えてください😃
