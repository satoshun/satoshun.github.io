+++
date = "Sun Apr 21 23:17:56 UTC 2019"
title = "（仮）MvRx"
tags = ["android", "architecture", "mvrx"]
blogimport = true
type = "post"
draft = true
+++

[MvRx](https://github.com/airbnb/MvRx)はAirbnbが開発をしているOSSフレームワークです。

特徴としては

- Kotlinファースト
- AAC（Android Architecture Components）をベースにしている
- [Epoxy](https://github.com/airbnb/epoxy)と相性が良い
  - 一緒に使うことを推奨している
- RxJavaを使っている
- 多くの部分でReactのAPIを参考にしてる
  - State、render、Epoxyなど
