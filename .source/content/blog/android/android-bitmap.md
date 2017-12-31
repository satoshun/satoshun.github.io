+++
date = "2015-03-06T23:44:03Z"
title = "Android: Bitmapのメモリ管理"
tags = ["android"]
blogimport = true
type = "post"
draft = true
+++


## Bitmapのメモリ管理2.X系

2.X系では, BitmapはNativeヒープに展開されるためGCの対象になりません. なので, 使用しなくなったら, 明示的にメモリを解放(recycle)する必要があります. メモリ解放を忘れてしまうと, OutOfMemoryになってしまうため, 注意が必要です.


## 実装

以下のことを満たすように実装するのが, メモリ効率的に良いと思います

- staticなBitmapマップ(singleton)を作成し, Bitmap系はそこにキャッシュ
    - キャッシュには[LruCache](http://developer.android.com/intl/ja/reference/android/util/LruCache.html)を使う
- reference countを保持しておき, ImageView等から参照されていない場合は,
