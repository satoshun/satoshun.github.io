+++
date = "Sun Dec  8 10:32:03 UTC 2019"
title = "GitHub Actions: 最近使っているGitHub Actionsたち"
tags = ["ci", "githubactions"]
blogimport = true
type = "post"
draft = false
+++

プライベート、仕事で使っているGitHub Actionsを雑に紹介します。

## Release Drafter

[Release Drafter](https://github.com/release-drafter/release-drafter)は、マージされたPRをもとに、いい感じにリリースノートを作ってくれます。

OSSのライブラリ開発で便利なので、最近導入しました。

## Image Actions

[Image Actions](https://github.com/calibreapp/image-actions)は、PRを出したときに、画像を圧縮をチャレンジし、小さくなった場合、圧縮したファイルをコミットしてくれます。

ただし、デザイナーがすでに圧縮してくれているなら、特に何もしてくれないので、弊アプリではあまり活躍してないです。

## First Interaction

[First Interaction](https://github.com/actions/first-interaction)は、初めてPRを出した人に、定型文を出すことが出来るGitHub actionです。

「はじめてのPRありがとうございます🙏ここ見といてね」みたいなメッセージを出して使ってます。

## Assign Author/Auto Assign

[Assign Author](https://github.com/technote-space/assign-author)、[Auto Assign](https://github.com/kentaro-m/auto-assign-action)は、自分にアサインしたり、均等にレビュワーを指定する事ができます。

レビューワーを分散させたかったので、使ってます。これ系のactionは他にもあるので、いろいろ調べるのも楽しそう。

## まとめ

- GitHub Actionsはいいぞ〜
