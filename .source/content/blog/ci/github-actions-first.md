+++
date = "Sat Dec  7 14:41:00 UTC 2019"
title = "GitHub Actions: 最近使っているGitHub Actionsたち"
tags = ["ci", "githubactions"]
blogimport = true
type = "post"
draft = true
+++

プライベート、仕事で使っているGitHub Actionsを雑に紹介します。

## Release Drafter

[Release Drafter](https://github.com/release-drafter/release-drafter)は、マージされたPRをもとに、いい感じにリリースノートを作ってくれます。

OSS開発で便利なので、最近導入しました。

## Image Actions

[Image Actions](https://github.com/calibreapp/image-actions)は、PRを出したときに、画像を圧縮をチャレンジし、小さくなった場合、圧縮したファイルをコミットしてくれます。

ただ、デザイナーがすでに圧縮してくれているなら、圧縮する意味はないので、弊アプリではあまり活躍してないです。

## First Interaction

[First Interaction](https://github.com/actions/first-interaction)は、初めてPRを出した人に、定型文を出すことが出来るGitHub actionです。

「はじめてのPRありがとうございます🙏」みたいなメッセージを出して使ってます。

## Assign Author

[Assign Author](https://github.com/technote-space/assign-author)、[Auto Assign](https://github.com/kentaro-m/auto-assign-action)は、自分にアサインしたり、均等にレビュワーを指定する事ができます。

レビューワーを分散させたかったので、使ってます。これ系のactionは他にもあるので、いろいろ調べるのも楽しそう。

## まとめ

- GitHub Actionsはいいぞ〜
