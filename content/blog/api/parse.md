+++
date = "2015-02-22T00:00:00Z"
title = "Android: Parse API"
tags = ["android", "webapi"]
blogimport = true
type = "post"
draft = true
+++

## 目標

アプリ内からPush Notficiationを飛ばす


## 前提知識

- ClientからPushを送るときは, 設定画面から`Client push enabled`をyesにする必要がある.
- Pushを飛ばす際に条件として指定したいフィールドは, Installationテーブルに保存する必要がある
  - 他のテーブルの場合, Pushを送ることが出来ない.

## 実装
