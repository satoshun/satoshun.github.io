+++
date = "2015-03-15T00:00:00Z"
title = "読んだ: Android master of fragment"
tags = ["book", "android"]
blogimport = true
type = "post"
draft = true
+++

## ポイント

- getActivity()がNullを返すことを想定しなければいけない
  - 非同期通信処理が完了する前に
    - Activityが終了する
    - 画面を回転させる
- FragmentからのCallbackは, onAttachの時に割りあてる
- 引数ありFragmentはご法度
  - setArgumentsを使う
