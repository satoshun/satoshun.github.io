+++
date = "2015-02-22T00:00:00Z"
title = "Android: facebook API"
tags = ["android", "webapi"]
blogimport = true
type = "post"
draft = true
+++

## 目標

アプリ内でFacebook Friends APIを利用し, smallな友達ネットワークを構築する


## 前提知識

permission: `user_friends`をつけて, Loginをする.

Friend APIをプライパシーの観点から, 取得できる項目数が減少している. おそらく`Username`と`Id`しかとれない.


## 実装, 解決
